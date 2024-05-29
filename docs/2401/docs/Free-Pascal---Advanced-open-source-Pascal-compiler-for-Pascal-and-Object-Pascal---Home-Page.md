<!--yml
category: 未分类
date: 2024-05-27 14:53:52
-->

# Free Pascal - Advanced open source Pascal compiler for Pascal and Object Pascal - Home Page

> 来源：[https://www.freepascal.org/](https://www.freepascal.org/)

# Introduction

## Overview

Free Pascal is a mature, versatile, open source Pascal compiler. It can target many processor architectures: Intel x86 (16 and 32 bit), AMD64/x86-64, PowerPC, PowerPC64, SPARC, SPARC64, ARM, AArch64, MIPS, Motorola 68k, AVR, and the JVM. Supported operating systems include Windows (16/32/64 bit, CE, and native NT), Linux, Mac OS X/iOS/iPhoneSimulator/Darwin, FreeBSD and other BSD flavors, DOS (16 bit, or 32 bit DPMI), OS/2, AIX, Android, Haiku, Nintendo GBA/DS/Wii, AmigaOS, MorphOS, AROS, Atari TOS, and various embedded platforms. Additionally, support for RISC-V (32/64), Xtensa, and Z80 architectures, and for the LLVM compiler infrastructure is available in the development version. Additionally, the Free Pascal team maintains a transpiler for pascal to Javascript called pas2js.

## Latest News

*   *Jan 4, 2024*

*   The creator of the Pascal Language, Niklaus Wirth, has passed away on January 1st. Free Pascal would not have existed without the work of Niklaus Wirth. We mourn a pioneer and a source of inspiration.

*   *August 8th, 2021*

*   FPC has moved to Gitlab!

    All SVN repositories have been converted to git and moved to gitlab. The Mantis bugtracker has also been converted to [gitlab](https://gitlab.com/freepascal.org/fpc/).

    You can find instructions in the [Development page](develop.var) or in the [Wiki](https://wiki.freepascal.org/FPC_git).

    Bugs can be reported [here](https://gitlab.com/groups/freepascal.org/fpc/-/issues).

*   *May 20th, 2021*

*   FPC version 3.2.2 has been released!

    This version is a point update to 3.2.0 and contains bugfixes and updated packages, some of which are high priority. In this case a new target was also backported from trunk.

    There is a list of [changes that may break backward compatibility](http://wiki.freepascal.org/User_Changes_3.2.2). You can also have a look at the [FPC 3.2.2 documentation](/docs.html).

    Downloads are available at [the download section](/download.html). Some links might be stale but will be updated in the coming days. If you have trouble using FTP due to recent browser updates, try the sourceforge mirror.

*   *June 19th, 2020**   *July 20, 2019**   *June 8, 2018*

*   Today FPC celebrates its 25th birthday !

    25 years have passed since 8 june 1993, and FPC still does not only exists, but is more alive and kicking than ever!

*   *May 28, 2018*

[Older news...](news.html)

## Current Version

Version *3.2.2* is the latest stable version of Free Pascal. Hit the [download](download.html) link and select a mirror close to you to download your copy. The development releases have version numbers *3.3.x*. See the [development](develop.html) page how to obtain the latest sources and support development.

## Features

The language syntax has excellent compatibility with TP 7.0 as well as with most versions of Delphi (classes, rtti, exceptions, ansistrings, widestrings, interfaces). A Mac Pascal mode, largely compatible with Think Pascal and MetroWerks Pascal, is also available. Furthermore Free Pascal supports function overloading, operator overloading, global properties and several other extra features.

## Requirements

**x86 architecture:**

> For the 80x86 version at least a 386 processor is required, but a 486 is recommended. The Mac OS X version requires Mac OS X 10.4 or later, with the developer tools installed.

**PowerPC architecture:**

> Any PowerPC processor will do. 16 MB of RAM is required. The Mac OS classic version is expected to work System 7.5.3 and later. The Mac OS X version requires Mac OS X 10.3 or later (can compile for 10.2.8 or later), with the developer tools installed. On other operating systems Free Pascal runs on any system that can run the operating system.

**ARM architecture**

> 16 MB of RAM is required. Runs on any ARM Linux installation.

**Sparc architecture**

> 16 MB of RAM is required. Runs on any Sparc Linux installation (solaris is experimental).

## License

The packages and runtime library come under a modified Library GNU Public License to allow the use of static libraries when creating applications. The compiler source itself comes under the GNU General Public License. The sources for both the compiler and runtime library are available; the complete compiler is written in Pascal.