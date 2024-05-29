<!--yml
category: 未分类
date: 2024-05-29 13:23:03
-->

# Shell, Awk, and Make Should Be Combined

> 来源：[https://www.oilshell.org/blog/2016/11/13.html](https://www.oilshell.org/blog/2016/11/13.html)

| [blog](/blog/) | [oilshell.org](/)

## Shell, Awk, and Make Should Be Combined

2016-11-13

[The shell](/cross-ref.html?tag=shell#shell) has been a part of Unix since its first release in 1971, while [Awk](/cross-ref.html?tag=awk#awk) and [Make](/cross-ref.html?tag=make#make) were additions to Unix, both released in 1977 and developed at Bell Labs.

These three tools were conceived as domain-specific languages with **distinct roles**:

*   Shell is about sequential and parallel *composition of processes*, including pipelines.
*   Awk is about *streaming computation* over rows of data. It has regular expressions and associative arrays, which presaged Perl, Python, Ruby, JavaScript, and the like.
*   Make is about *data-driven, incremental, and parallel computation*, also using Unix processes.

Not only do they survive four decades later, they're as popular as ever:

*   A major new feature of Windows 10 is that [it can run a 27-year-old program: bash](https://blogs.windows.com/buildingapps/2016/03/30/run-bash-on-ubuntu-on-windows/#o8Hjme8wQwqvj80U.97). This is apparently an improvement on the dev environment of an OS that has itself been around for three decades.
*   Despite scores of "Make replacements" that exist, it's the most common build tool you'll encounter in a Unix environment. Makefile tutorials are still being written in 2016\. An excellent [book on GNU Make](https://www.amazon.com/GNU-Make-Book-Graham-Cumming/dp/1593276494) was published in 2015.
*   Awk is less popular due to the adoption of its successors, but it's still deeply embedded in the build processes of [Linux, FreeBSD](http://lists.landley.net/pipermail/toybox-landley.net/2016-July/008499.html), and other foundational software.

The success of these tools has led to an common anti-pattern: they grew into [ALGOL-like languages](/cross-ref.html?tag=algol-like#algol-like) with a **quirky syntax** constrained by backward compatibility.

Tomorrow, I'll show this with example code in three languages. For now, let me support this argument with some observations:

*   It's common to see all three languages in the **same source tree** (e.g. Linux, FreeBSD), which is a tax on holistic understanding.

*   Not only are they in the same tree, but they're often **embedded** within each other. When you see `awk` in a `Makefile`, you're seeing **three** intertwined languages, because `make` is a **macro language** that passes command strings literally to `/bin/sh`.

Quoting is difficult when composing two languages, let alone three. Here's a sample from my `~/src` dir:

```
$ grep AWK */Makefile
busybox-1.24.2/Makefile:        ($(OBJDUMP) -h $< | $(AWK) '/^ +[0-9]/{print $$4 " 0 " $$2}'; $(NM) $<) | sort > $@
cityhash-1.1.1/Makefile:  $(AWK) 'BEGIN { files["."] = "" } { files[$$2] = files[$$2] " " $$1; \
dash-0.5.8/Makefile:      $(AWK) '{ files[$$0] = 1; nonempty = 1; } \
glibc-2.23/Makefile:    AWK='$(AWK)' scripts/check-local-headers.sh \
re2c-0.16/Makefile:  $(AWK) 'BEGIN { files["."] = "" } { files[$$2] = files[$$2] " " $$1; \

```

*   Make and shell have unfortunate **syntactic conflicts**. For example: `$@` means the *output file* of a rule in Make, but it means the *arguments array* in shell.

*   Eighty percent of lines in a typical `Makefile` are **literally shell**, or variable assignments that are easily expressed in shell. It's easy to imagine extensions to shell syntax to support Make's semantics.

*   Make and shell both [simulate arrays with delimited strings](06.html), leading to quoting problems.

As far as awk:

*   `bash` and `zsh` add **associative arrays** and **regex support** to shell, which makes them semantically close to `awk`.

*   Awk only has floating point **arithmetic**, and bash only has integer arithmetic. It's easy to imagine a language with both.

*   Awk and shell are similar in an interesting way: they lack **garbage collection**. This leads to arrays that can neither be nested nor returned from functions.

What all three languages have in common:

## Conclusion

Despite their origins, the three languages are now more similar than different — at least semantically. We'll see tomorrow that their syntaxes are wildly and needlessly different.

And while I'm wary of discussing plans without code, I should say that I want `oil` to subsume `awk` and `make` as well. I expect that this theme will consume many more blog posts.