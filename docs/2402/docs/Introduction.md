<!--yml
category: 未分类
date: 2024-05-27 14:37:05
-->

# Introduction

> 来源：[https://anaseto.codeberg.page/goal/chap-intro.html](https://anaseto.codeberg.page/goal/chap-intro.html)

[Home](/). Work-in-progress documentation for [Goal](https://codeberg.org/anaseto/goal). *Last update: 2024-05-25*.

# 1 Introduction

Goal is an embeddable [array programming language](https://en.wikipedia.org/wiki/Array_programming) with a bytecode interpreter, written in Go. The command line interpreter can execute scripts or run in interactive mode. For installation, see the `README.md` file in the [project’s repository](https://codeberg.org/anaseto/goal). You can also [try Goal on the browser](https://anaseto.codeberg.page/try-goal/). An EPUB version of the present documentation is available [here](https://anaseto.codeberg.page/misc/goal/goal-docs-gen.epub)

Like in most array programming languages, Goal’s builtins vectorize operations on immutable arrays and encourage a functional style for control and data transformations, supported by a simple dynamic type system with little abstraction, and mutable variables (but no mutable values).

You can read about Goal’s origins and influences in the [Origins section](chap-FAQ.html#origins) of the FAQ. If you already know about the K language, you might want to read the [Differences from K](chap-from-k.html) chapter.

Goal’s main distinctive features among array languages can be summarized as follows:

*   Dictionaries and table functionality (similar to most K dialects)

*   Atomic strings: `"ab" "bc"="bc" → 0 1`

*   Unicode-aware string primitives: `_"AB" "Π" → "ab" "π"`

*   Perl-like quoting constructs and string interpolation: `qq/$var\n or ${var}/` and `rq#raw strings#`

*   Format strings: `"%.2g"$1 4%3 → "0.33" "1.3"`

*   Dedicated regular expression syntax: `rx/\s+/`

*   Error values

*   Embeddable and extensible in Go

Goal also has support for standard features like I/O, CSV, and time handling.

Goal shines the most in common scripting tasks, like handling columnar data or text processing. Goal is also suitable for exploratory programming.

The [next chapter](chap-tutorial.html) gives a tour of Goal’s features and showcases the language in a couple of practical examples.