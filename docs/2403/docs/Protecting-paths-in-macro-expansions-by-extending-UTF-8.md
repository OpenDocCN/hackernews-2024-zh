<!--yml
category: 未分类
date: 2024-05-27 14:38:32
-->

# Protecting paths in macro expansions by extending UTF-8

> 来源：[https://nullprogram.com/blog/2024/03/05/](https://nullprogram.com/blog/2024/03/05/)

March 05, 2024

nullprogram.com/blog/2024/03/05/

After a year I’ve finally came up with an elegant solution to a vexing [u-config](/blog/2023/01/18/) problem. The pkg-config format uses macros to generate build flags through recursive expansion. Some flags embed file system paths, but to the macro system it’s all strings. The output is also ultimately just one big string, which the receiving shell [splits into fields](https://pubs.opengroup.org/onlinepubs/9699919799/utilities/V3_chap02.html#tag_18_06_05). If a path contains spaces, or shell metacharacters, u-config must escape them so that shells treat them as part of a token. But how can u-config itself distinguish incidental spaces in paths from deliberate spaces between flags? What about other shell metacharacters in paths? My solution is to extend UTF-8 to encode metadata that survives macro expansion.

As usual, it helps to begin with a concrete example of the problem. The following is a conventional `.pc` file much like you’d find on your own system:

```
prefix=/usr
exec_prefix=${prefix}
libdir=${exec_prefix}/lib
includedir=${prefix}/include

Name: Example
Version: 1.0
Description: An example .pc file
Cflags: -I${includedir}
Libs: -L${libdir} -lexample 
```

It begins by defining the library’s installation prefix from which it derives additional paths, which are finally used in the package fields that generate build flags (`Cflags`, `Libs`). If I run u-config against this configuration:

```
$ pkg-config --cflags --libs example
-I/usr/include -L/usr/lib -lexample 
```

Typically `prefix` is populated by the library’s build system, which knows where the library is to be installed. In some situations that’s not possible, and there is no opportunity to set `prefix` to a meaningful path. In that case, pkg-config can automatically override it (`--define-prefix`) with a path relative to the `.pc` file, making the installation relocatable. This works quite well [on Windows, where it’s the default](/blog/2020/09/25/):

```
$ pkg-config --cflags --libs example
-IC:/Users/me/example/include -LC:/Users/me/example/lib -lexample 
```

This just works… *so long as the path does not contain spaces*. If so, it risks splitting into separate fields. The `.pc` format supports quoting to control how such output is escaped. Regions between quotes are escaped in the output so that they retain their spaces when field split in the shell. If a `.pc` file author is careful, they’d write it with quotes:

```
Cflags: -I"${includedir}"
Libs: -L"${libdir}" -lexample 
```

The paths are carefully placed within [quoted regions](/blog/2021/12/04/) so that they come out properly:

```
$ pkg-config --cflags example
-IC:/Program\ Files/example/include 
```

*Almost nobody writes their `.pc` files this way*! The convention is not to quote. My original solution was to implicitly wrap `prefix` in quotes on assignment, which fixes the vast majority of `.pc` files. That effectively looks like this in the “virtual” `.pc` file:

```
prefix="C:/Program Files/example"
exec_prefix=${prefix}
libdir=${exec_prefix}/lib
includedir=${prefix}/include 
```

So the important region is quoted, its spaces preserved. However, the occasional library author actively supporting Windows inevitably runs into this problem, and their system’s pkg-config implementation does not quote `prefix`. They soon figure out explicit quoting and apply it, which then undermines u-config’s implicit quoting. The quotes essentially cancel out:

```
"$includedir" -> ""C:/Program Files/example"/include" 
```

The quoted regions are inverted and nothing happens. Though this is a small minority, the libraries that do this and the ones you’re likely to use on Windows are correlated. I was stumped: How to support quoted and unquoted `.pc` files simultaneously?

### Extending UTF-8

I recently had the thought: What if somehow u-config tracked which spans of string were paths. `prefix` is initially a path span, and then track it through macro-expansion and concatenation. Soon after that I realized it’s even simpler: **Encode the spaces in a path as a value other than space**, but also a value that cannot appear in the input. Recall that [certain octets can never appear in UTF-8 text](/blog/2017/10/06/): the 8 values whose highest 5 bits are set. That would be the first octet of 5-octet, or longer, code point, but those are forbidden.

When paths enter the macro system, special characters are encoded as one of these 8 values. They’re converted back to their original ASCII values during output encoding, escaped. It doesn’t interact with the pkg-config quoting mechanism, so there’s no quote cancellation. Both quoting cases are supported equally.

For example, if space is mapped onto `\xff` (255), then:

```
in:  C:/Program Files/foo    -> C:/Program\xffFiles/foo
out: C:/Program\xffFiles/foo -> C:/Program\ Files/foo 
```

Which prints the same regardless of `${includedir}` or `"${includedir}"`. Problem solved!

That’s not the only complication. Outputs may *deliberately* include shell metacharacters, though typically these are [Makefile](/blog/2017/08/20/) fragments. For example, the default value of `${pc_top_builddir}` is `$(top_builddir)`, which `make` will later expand. While these characters are special to a shell, and certainly special to `make`, they must not be escaped.

What if a path contains these characters? The pkg-config quoting mechanism won’t help. It’s only concerned with spaces, and `$(...)` prints the same quoted nor not. As before, u-config must track provenance — whether or not such characters originated from a path.

If `$PKG_CONFIG_TOP_BUILD_DIR` is set, then `pc_top_builddir` is set to this environment variable, useful when the result isn’t processed by `make`. In this case it’s a path, and `$(...)` ought to be escaped. Even without `$` it must be quoted, because the parentheses would still invoke a subshell. But who would put parenthesis in a path? Lo and behold!

```
C:/Program Files (x86)/example 
```

Again, extending UTF-8 solves this as well: Encode `$`, `(`, and `)` in paths using three of those forbidden octets, and escape them on the way out, allowing unencoded instances to go straight through.

```
in:  C:/Program\xffFiles\xff\xfdx86\xfe/example
out: C:/Program\ Files\ \(x86\)/example 
```

This makes `pc_top_builddir` straightforward: default to a raw string, otherwise a path-encoded environment variable (note: `s8` [is a string type](/blog/2023/10/08/) and `upsert` is [a hash map](/blog/2023/09/30/)):

```
 s8 top_builddir = s8("$(top_builddir)");
    if (envvar_set) {
        top_builddir = s8pathencode(envvar, perm);
    }
    *upsert(&global, s8("pc_top_builddir"), perm) = top_builddir; 
```

For a particularly wild case, consider deliberately using a `uname -m` command substitution to construct a path, i.e. the path contains the target machine architecture (`i686`, `x86_64`, etc.):

```
Cflags: -I${prefix}/$(uname -m)/include 
```

(Not that condone such nonsense. This is merely a reality of real world `.pc` files.) With `prefix` automatically set as above, this will print:

```
-IC:/Program\ Files\ \(x86\)/example/$(uname -m)/include 
```

Path parentheses are escaped because they came from a path, but command substitution passes through because it came from the `.pc` source. Quite cool!