<!--yml
category: 未分类
date: 2024-05-27 13:30:00
-->

# PyPy v7.3.16 release | PyPy

> 来源：[https://www.pypy.org/posts/2024/04/pypy-v7316-release.html](https://www.pypy.org/posts/2024/04/pypy-v7316-release.html)

## PyPy v7.3.16: release of python 2.7, 3.9, and 3.10

The PyPy team is proud to release version 7.3.16 of PyPy.

This release includes security fixes from upstream CPython, and bugfixes to the garbage collector, described in a [gc bug-hunt blog post](https://www.pypy.org/posts/2024/03/fixing-bug-incremental-gc.html).

The release includes three different interpreters:

> *   PyPy2.7, which is an interpreter supporting the syntax and the features of Python 2.7 including the stdlib for CPython 2.7.18+ (the `+` is for backported security updates)
>     
>     
> *   PyPy3.9, which is an interpreter supporting the syntax and the features of Python 3.9, including the stdlib for CPython 3.9.19.
>     
>     
> *   PyPy3.10, which is an interpreter supporting the syntax and the features of Python 3.10, including the stdlib for CPython 3.10.14.

The interpreters are based on much the same codebase, thus the multiple release. This is a micro release, all APIs are compatible with the other 7.3 releases. It follows after 7.3.15 release on Jan 15, 2024

We recommend updating. You can find links to download the v7.3.16 releases here:

> [https://pypy.org/download.html](https://pypy.org/download.html)

We would like to thank our donors for the continued support of the PyPy project. If PyPy is not quite good enough for your needs, we are available for [direct consulting](https://www.pypy.org/pypy-sponsors.html) work. If PyPy is helping you out, we would love to hear about it and encourage submissions to our [blog](https://pypy.org/blog) via a pull request to [https://github.com/pypy/pypy.org](https://github.com/pypy/pypy.org)

We would also like to thank our contributors and encourage new people to join the project. PyPy has many layers and we need help with all of them: bug fixes, [PyPy](index.html) and [RPython](https://rpython.readthedocs.org) documentation improvements, or general [help](project-ideas.html) with making RPython's JIT even better.

If you are a python library maintainer and use C-extensions, please consider making a [HPy](https://hpyproject.org/) / [CFFI](https://cffi.readthedocs.io) / [cppyy](https://cppyy.readthedocs.io) version of your library that would be performant on PyPy. In any case, both [cibuildwheel](https://github.com/joerick/cibuildwheel) and the [multibuild system](https://github.com/matthew-brett/multibuild) support building wheels for PyPy.

### What is PyPy?

PyPy is a Python interpreter, a drop-in replacement for CPython It's fast ([PyPy and CPython 3.7.4](https://speed.pypy.org) performance comparison) due to its integrated tracing JIT compiler.

We also welcome developers of other [dynamic languages](https://rpython.readthedocs.io/en/latest/examples.html) to see what RPython can do for them.

We provide binary builds for:

> *   **x86** machines on most common operating systems (Linux 32/64 bits, Mac OS 64 bits, Windows 64 bits)
>     
>     
> *   64-bit **ARM** machines running Linux (`aarch64`).
>     
>     
> *   Apple **M1 arm64** machines (`macos_arm64`).
>     
>     
> *   **s390x** running Linux

PyPy support Windows 32-bit, Linux PPC64 big- and little-endian, and Linux ARM 32 bit, but does not release binaries. Please reach out to us if you wish to sponsor binary releases for those platforms. Downstream packagers provide binary builds for debian, Fedora, conda, OpenBSD, FreeBSD, Gentoo, and more.

### What else is new?

For more information about the 7.3.16 release, see the [full changelog](https://doc.pypy.org/en/latest/release-v7.3.16.html#changelog).

Please update, and continue to help us make pypy better.

Cheers, The PyPy Team