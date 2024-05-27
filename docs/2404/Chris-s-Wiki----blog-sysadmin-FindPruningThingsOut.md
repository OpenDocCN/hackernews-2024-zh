<!--yml
category: 未分类
date: 2024-05-27 13:33:39
-->

# Chris's Wiki :: blog/sysadmin/FindPruningThingsOut

> 来源：[https://utcc.utoronto.ca/~cks/space/blog/sysadmin/FindPruningThingsOut](https://utcc.utoronto.ca/~cks/space/blog/sysadmin/FindPruningThingsOut)

Suppose that you need to scan your filesystems and pass some files with specific names, ownerships, or whatever, except that you want to exclude scanning under /tmp and /var/tmp (as illustrative examples). Perhaps also you're feeding the file names to a shell script, especially in a pipeline, which means that you'd like to screen out directory and file names that have (common) problem characters in them, like spaces.

(If you can use Bash for your shell script, the latter problem can be dealt with because you can get Bash to read NUL-terminated lines that can be produced by 'find ... -print0'.)

Excluding things from '`find`' results is done with find's -prune action, which is a little bit tricky to use when you want to exclude absolute paths (well okay it's a little bit tricky in general; see [this SO question and answers](https://stackoverflow.com/questions/1489277/how-to-use-prune-option-of-find-in-sh)). To start with, you're going to want to generate a list of filesystems and then scan them by absolute path:

> ```
> FSES="$(... something ...)"
> for fs in $FSES; do
>     find "$fs" -xdev [... magic ...]
> done
> 
> ```

Starting with an absolute path to the filesystem (instead of cd'ing into the root of the filesystem and doing '`find . -xdev [...]`' means that we can now use absolute paths in find's -path argument instead of ones relative to the filesystem root:

> ```
> find "$fs" -xdev '(' -path /tmp -o -path /var/tmp ')' -prune -o ....
> 
> ```

With absolute paths, we don't have to worry about what if /var or /tmp (or /var/tmp) are separate filesystems, instead of being directories on the root filesystem. Although it's hard to work out without experimentation, -xdev and -prune combine the way we want.

(If we're running '`find`' on a filesystem that doesn't contain either /tmp or /var/tmp, we'll waste a bit of CPU time having '`find`' evaluate those -path arguments all the time despite it never being possible for them to match. This is unimportant when compared to having a simpler, less error prone script.)

If we want to exclude paths with spaces in them, this is easily done with '`-name "* *"`'. If we want to get all whitespace, we need [GNU Find](https://man7.org/linux/man-pages/man1/find.1.html) and its '-regex' argument, documented best in ["Regular Expressions" in the info documentation](https://www.gnu.org/software/findutils/manual/html_mono/find.html#Regular-Expressions). Because we want to use [a character class](https://www.gnu.org/software/grep/manual/html_node/Character-Classes-and-Bracket-Expressions.html) to match whitespace, we need to use one of the regular expression types that include this, so:

> ```
> find "$fs" -regextype grep ... -regex '.*[[:space:]].*' ...
> 
> ```

On the whole, '`find`' is an awkward tool to use for this sort of filtering. Unfortunately it's sometimes what we turn to because our other options involve things like writing programs that consume and filter NUL-terminated file paths.

(And having '`find`' skip entire directory trees is more efficient than letting it descend into them, print all their file paths, and then filtering the file paths out later.)

PS: One of the little annoyances of Unix for system administrators is that so many things in a stock Unix environment fall apart the moment people start putting odd characters in file names, unless you take extreme care and use unusual tools. This often affects sysadmins because we frequently have to deal with other people's almost arbitrary choices of file and directory names, and we may be dealing with actively malicious attackers for extra concern.

### Sidebar: Reading null-terminated lines in Bash

[Bash's version of the '`read`' builtin](https://www.gnu.org/software/bash/manual/bash.html#index-read) supports a '-d' argument that can be used to read NUL-terminated lines:

> ```
> while IFS= read -r -d '' line; do
>   [ ... use "$line" ... ]
> done
> 
> ```

You still have to properly quote `"$line"` in every single use, especially as you're doing this because you expect your lines to (or filenames) to sometimes contain troublesome characters. You should definitely use [Shellcheck](http://www.shellcheck.net/) and pay close attention to its warnings ([they're good for you](/~cks/space/blog/programming/ShellcheckGoodForMe)).