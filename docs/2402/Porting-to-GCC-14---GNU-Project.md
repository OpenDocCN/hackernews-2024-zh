<!--yml
category: 未分类
date: 2024-05-27 15:00:10
-->

# Porting to GCC 14 - GNU Project

> 来源：[https://gcc.gnu.org/gcc-14/porting_to.html#c](https://gcc.gnu.org/gcc-14/porting_to.html#c)

# Porting to GCC 14

The GCC 14 release series differs from previous GCC releases in [a number of ways](changes.html). Some of these are a result of bug fixing, and some old behaviors have been intentionally changed to support new standards, or relaxed in standards-conforming ways to facilitate compilation or run-time performance.

Some of these changes are user visible and can cause grief when porting to GCC 14\. This document is an effort to identify common issues and provide solutions. Let us know if you have suggestions for improvements!

## C language issues

### Certain warnings are now errors

> Function prototyping was added, first to help enforce type checking on both the types of the arguments passed to the function, and also to check the assignment compatibility of the function return type.Standard C: The ANSI Draft Grows Up. PC Magazine, September 13, 1988, page 117.

The initial ISO C standard and its 1999 revision removed support for many C language features that were widely known as sources of application bugs due to accidental misuse. For backwards compatibility, GCC 13 and earlier diagnosed use of these features as warnings only. Although these warnings have been enabled by default for many releases, experience shows that these warnings are easily ignored, resulting in difficult to diagnose bugs. In GCC 14, these issues are now reported as errors, and no output file is created, providing clearer feedback to programmers that something is wrong.

Most components of the GNU project have already been fixed for compatibility, but not all of them have had releases since fixes were applied. Programs that bundle parts of [gnulib](https://www.gnu.org/software/gnulib/) or [autoconf-archive](https://www.gnu.org/software/autoconf-archive/) may need to update these sources files from their upstream versions.

Several GNU/Linux distributions have shared patches from their porting efforts on relevant upstream mailing lists and bug trackers, but of course many programs that exhibit these historic C compatibility issues are dormant today.

#### Implicit `int` types (`-Werror=implicit-int`)

In most cases, simply adding the missing `int` keyword addresses the error. For example, a flag variable like

```
  static initialized;

```

becomes:

```
  static  initialized;

```

If the return type of a function is omitted, the correct type to add can be `int` or `void`, depending on whether the function returns a value or not.

In some cases the previously implied `int` type was not correct. This mostly happens in old-style function definitions, when argument types are not declared outside the parameter list. Using the correct type may be required to avoid int-conversion errors (see below). In the example below, the type of `s` should be `const char *`, not `int`:

```
  write_string (fd, s)
  {
    write (fd, s, strlen (s));
  }

```

The corrected standard C source code might look like this (still disregarding error handling and short writes):

```

  write_string ( fd, s)
  {
    write (fd, s, strlen (s));
  }

```

#### Implicit function declarations (`-Werror=implicit-function-declaration`)

It is no longer possible to call a function that has not been declared. In general, the solution is to include a header file with an appropriate function prototype. Note that GCC will perform further type checks based on the function prototype, which can reveal further type errors that require additional changes.

For well-known functions declared in standard headers, GCC provides fix-it hints with the appropriate `#include` directives:

```
error: implicit declaration of function ‘strlen’ [-Wimplicit-function-declaration]
    5 |   return strlen (s);
      |          ^~~~~~
note: include ‘<string.h>’ or provide a declaration of ‘strlen’
  +++ |+#include <string.h>
    1 |

```

On GNU systems, headers described in standards (such as the C standard, or POSIX) may require the definition of certain macros at the start of the compilation before all required function declarations are made available. See [Feature Test Macros](https://sourceware.org/glibc/manual/latest/html_node/Feature-Test-Macros.html) in the GNU C Library manual for details.

Some programs are built with `-std=c11` or similar `-std` options that do not contain the string `gnu`, but still use POSIX or other extensions in standard C headers such as `<stdio.h>`. The GNU C library automatically suppresses these extensions in standard C mode, which can result in implicit function declarations. To address this, the `-std=c11` option can be dropped, `-std=gnu11` can be used instead, or `-std=c11 -D_DEFAULT_SOURCE` can be used re-enable common extensions. Alternatively, projects using Autoconf could enable `AC_USE_SYSTEM_EXTENSIONS`.

If undeclared functions from the same project are called and there is no suitable shared header file yet, you should add a declaration to a header file that is included by both the callers and the source file containing the function definition. This ensures that GCC checks that the prototype matches the function definition. GCC can perform type checks across translation units when building with options such as `-flto -Werror=lto-type-mismatch`, which can help with adding appropriate prototypes.

In some rare cases, adding a prototype may change ABI in inappropriate ways. In these situations, it is necessary to declare a function without a prototype:

```
  void no_prototype ();

```

However, this is an obsolete C feature that will change meaning in C23 (declaring a function with a prototype and accepting no arguments, similar to C++). Usually, a prototype with the default argument promotions applied can be used instead.

When building library code on GNU systems, [it was possible to call undefined (not just undeclared) functions](https://sourceware.org/binutils/docs-2.42/ld/Options.html#index---allow-shlib-undefined) and still run other code in the library, particularly if ELF lazy binding was used. Only executing the undefined function call would result in a lazy binding error and program crash.

#### Typos in function prototypes (`-Werror=declaration-missing-parameter-type`)

Incorrectly spelled type names in function declarations are treated as errors in more cases, under a new `-Wdeclaration-missing-parameter-type` warning. The second line in the following example is now treated as an error (previously this resulted in a warning not controlled by a specific `-W…` option):

```
  typedef int *int_array;
  int first_element (intarray);

```

The fix is to correct the spelling mistake:

```
  typedef int *int_array;
  int first_element ();

```

GCC will type-check function arguments after that, potentially requiring further changes. (Previously, the function declaration was treated as not having a prototype.)

#### Incorrect uses of the `return` statement (`-Werror=return-mismatch`)

GCC no longer accepts `return` statements with expressions in functions which are declared to return `void`, or `return` statements without expressions for functions returning a non-`void` type.

To address this, remove the incorrect expression (or turn it into a statement expression immediately prior to the `return` statements if the expression has side effects), or add a dummy return value, as appropriate. If there is no suitable dummy return value, further changes may be needed to implement appropriate error handling.

Previously, these mismatches were diagnosed as a `-Wreturn-type` warning. This warning still exists, and is not treated as an error by default. It now covers remaining potential correctness issues, such as reaching the closing brace `}` of function that does not return `void`.

By default, GCC still accepts returning an expression of type `void` from within a function that itself returns `void`, as a GNU extension that matches standard C++ rules in this area.

#### Using pointers as integers and vice versa (`-Werror=int-conversion`)

GCC no longer treats integer types and pointer types as equivalent in assignments (including implied assignments of function arguments and return values), and instead fails the compilation with a type error.

It makes sense to address missing `int` types, implicit function declarations, and incorrect `return` statement usage prior to tackling these `int`-conversion issues. Some of them will be caused by missing types treated as `int`, and the default `int` return type of implicitly declared functions.

To fix the remaining `int`-conversions issues, add casts to an appropriate pointer or integer type. On GNU systems, the standard types `intptr_t` and `uintptr_t` (defined in `<stdint.h>`) are always large enough to store all pointer values. If you need a generic pointer type, consider using `void *`.

In some cases, it may be more appropriate to pass the address of an integer variable instead of a cast of the variable's value.

#### Type checking on pointer types (`-Werror=incompatible-pointer-types`)

GCC no longer allows implicitly casting all pointer types to all other pointer types. This behavior is now restricted to the `void *` type and its qualified variations.

To fix compilation errors resulting from that, you can add the appropriate casts, and maybe consider using `void *` in more places (particularly for old programs that predate the introduction of `void *` into the C language).

Programs that do not carefully track pointer types are likely to contain aliasing violations, so consider building with `-fno-strict-aliasing`. (Whether casts are written manually or performed by GCC automatically does not make a difference in terms of strict aliasing violations.)

A frequent source of incompatible function pointer types involves callback functions that have more specific argument types (or less specific return types) than the function pointer they are assigned to. For example, old code which attempts to sort an array of strings might look like this:

```
#include <stddef.h>
#include <stdlib.h>
#include <string.h>

int
compare (char **a, char **b)
{
  return strcmp (*a, *b);
}

void
sort (char **array, size_t length)
{
  qsort (array, length, sizeof (*array), compare);
}

```

To address this, the callback function should be defined with the correct type, and the arguments should be cast as appropriate before they are used (as calling a function through a function pointer of an incorrect type is undefined):

```
int
compare (, )
{

  return strcmp (*a, *b);
}

```

A similar issue can arise with object pointer types. Consider a function that is declared with an argument of type `void **`, and you want to pass the address of a variable of type `int *`:

```
extern int *pointer;
extern void operate (int command, void **);

operate (0, &pointer);

```

In these cases, it may be appropriate to make a copy of the pointer with the correct `void *` type:

```
extern int *pointer;
extern void operate (int command, void **);

operate (0, &pointer);

```

As mentioned initially, adding the cast here would not eliminate any strict aliasing violation in the implementation of the `operate` function. Of course in general, introducing such additional copies may alter program behavior.

Some programming styles rely on implicit casts between related object pointer types to implement C++-style `struct` inheritance. It may be possible to avoid writing manual casts with the `-fplan9-extensions` option and unnamed initial `struct` fields for the base type in derived `struct`s.

Some programs use a concrete function pointer type such as `void (*) (void)` as a generic function pointer type (similar to `void *` for object pointer types), and rely on implicit casts to and from this type. The reason for that is that C does not offer a generic function pointer type, and standard C disallows casts between function pointer types and object pointer types. On most targets, GCC supports implicit conversion between `void *` and function pointer types. However, for a portable solution, the concrete function pointer type needs to be used, together with explicit casts.

#### Impact on Autoconf and build environment probing in general

Most [Autoconf](https://www.gnu.org/software/autoconf/) probes that check for build system features are written in such a way that they trigger a compiler error if a feature is missing. The new errors may cause GCC to fail compiling a probe when it worked before, unrelated to the actual objective of the probe. These failed probes tend to disable program features together with their tests, resulting in silently dropping features.

In cases where this is a concern, generated `config.log`, `config.h` and other source code files can be compared using `diff`, to ensure there are no unexpected differences.

This phenomenon also impacts similar procedures that are part of CMake, Meson, and various build tools for C extension modules of scripting languages.

Autoconf has supported C99 compilers since at least version 2.69 in its generic, core probes. (Specific functionality probes may have issues addressed in more recent versions.) Versions before 2.69 may have generic probes (for example for standard C support) that rely on C features that were removed in C99 and thus fail with GCC 14.

#### Turning errors back into warnings

Sources that cannot be ported to standard C can be compiled with `-fpermissive`, `-std=gnu89`, or `-std=c89`. Despite their names, the latter two options turn on support for pre-standard C constructs, too. With the `-fpermissive` option, programs can use C99 inlining semantics and features that were removed from C99\. Alternatively, individual errors can be downgraded to warnings using the relevant `-Wno-error=…` option, or disabled completely with `-Wno-…`. For example, `-Wno-error=incompatible-pointer-types` turns off most type checking for pointer assignments.

Some build systems do not pass the `CFLAGS` variable to all parts of the builds, and may require setting `CC` to something like `gcc -fpermissive` instead. If the build system does not support whitespace in the `CC` variable, a wrapper script like this may be required:

```
#!/bin/sh
exec /usr/bin/gcc -fpermissive "$@"

```

#### Accommodating C code generators

C code generators that cannot be updated to generate valid standard C can emit [`#pragma GCC diagnostic warning`](/onlinedocs/gcc-14.1.0/gcc/Diagnostic-Pragmas.html) directives to turn these errors back into warnings:

```
#if defined __GNUC__ && __GNUC__ >= 14
#pragma GCC diagnostic warning "-Wimplicit-function-declaration"
#pragma GCC diagnostic warning "-Wincompatible-pointer-types"
#pragma GCC diagnostic warning "-Wint-conversion"
#pragma GCC diagnostic warning "-Wreturn-mismatch"
#endif

```

Not included here are `-Wimplicit-int` and `-Wdeclaration-missing-parameter-type` because they should be straightforward to address in a code generator.

#### Future directions

This section concerns potential future changes related to language features from the C standard and other backwards compatibility hazards. These plans may change and are mentioned here only to give guidance which source-level changes to prioritize for future compatibility.

It is unclear at which point GCC can enable the C23 `bool` keyword by default (making the `bool` type available without including `#include <stdbool.h>`). Many programs define their own `bool` types, sometimes with a different size than the built-in `_Bool` type. A further complication is that even if the sizes are the same, a custom `bool` typically does not have trap representations, while `_Bool` and the new `bool` type do. This means that there can be subtle compatibility issues, particularly when processing untrusted, not necessarily well-formed input data.

GCC is unlikely to warn about function declarations that are not prototypes by default. This means that there is no stringent reason to turn

```
void do_something ();

```

into

```
void do_something (void);

```

except for diagnosing extraneous ignored arguments as errors. A future version of GCC will likely warn about calls to functions without a prototype which specify such extraneous arguments (`do_something (1)`, for example). Eventually, GCC will diagnose such calls as errors because they are constraint violations in C23.

GCC will probably continue to support old-style function definitions even once C23 is used as the default language dialect.

## C++ language issues

Some C++ Standard Library headers have been changed to no longer include other headers that were being used internally by the library. As such, C++ programs that used standard library components without including the right headers will no longer compile.

The following headers are used less widely in libstdc++ and may need to be included explicitly when compiling with GCC 14:

*   `<algorithm>` (for `std::copy_n`, `std::find_if`, `std::lower_bound`, `std::remove`, `std::reverse`, `std::sort` etc.)
*   `<cstdint>` (for `std::int8_t`, `std::int32_t` etc.)

### Pragma GCC target now affects preprocessor symbols

The behavior of pragma `GCC target` and specifically how it affects ISA macros has changed in GCC 14\. Before the `GCC target` pragma defined and undefined corresponding ISA macros in C when using the integrated preprocessor during compilation but not when the preprocessor was invoked as a separate step or when using the `-save-temps` option. In C++ the ISA macro definitions were performed in a way which did not have any actual effect.

In GCC 14 C++ behaves like C with integrated preprocessing in earlier versions. Moreover, in both languages ISA macros are defined and undefined as expected when preprocessing separately from compilation.

This can lead to different behavior, especially in C++. For example, a part of the C++ snippet below will be (silently) compiled for an incorrect instruction set by GCC 14.

```
  #if ! __AVX2__
  #pragma GCC push_options
  #pragma GCC target("avx2")
  #endif

  /* Code to be compiled for AVX2\. */

  /* With GCC 14, __AVX2__ here will always be defined and pop_options
  never invoked. */
  #if ! __AVX2__
  #pragma GCC pop_options
  #endif

  /* With GCC 14, all following functions will be compiled for AVX2
  which was not intended. */

```

The fix in this case is to remember whether `pop_options` needs to be performed in a new user-defined macro.