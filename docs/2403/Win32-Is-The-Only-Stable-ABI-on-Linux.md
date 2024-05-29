<!--yml
category: 未分类
date: 2024-05-29 12:46:43
-->

# Win32 Is The Only Stable ABI on Linux

> 来源：[https://blog.hiler.eu/win32-the-only-stable-abi/](https://blog.hiler.eu/win32-the-only-stable-abi/)

# Win32 Is The Only Stable ABI on Linux

2022-08-15, by ivyl

## TL;DR

*   Glibc 2.36 was released and all the games using EAC EOS are no longer playable.
*   A user did bisect and found [the offending commit](https://github.com/ValveSoftware/Proton/issues/6051#issuecomment-1206888814).
*   I’ve filed [a Glibc bug](https://sourceware.org/bugzilla/show_bug.cgi?id=29456) hoping for a quick revert.
*   Turns out other things are also broken.
*   Most likely it will stay this way.
*   Win32 (via Wine + friends) is the only stable ABI on Linux :-P

## `DT_HASH` and `DT_GNU_HASH`

In the ELF format there are two ways of providing a hash table of symbols. `DT_HASH` and `DT_GNU_HASH`. The first one is part of [SYSV generic ABI](http://www.sco.com/developers/gabi/latest/ch5.dynamic.html#hash) and is well documented and marked as mandatory. `DT_GNU_HASH` is a newer, smaller, and faster replacement that is [not documented](https://sourceware.org/legacy-ml/binutils/2006-10/msg00377.html), implemented in a few places (Glibc, GNU ld, Musl, LLVM, mold, etc.) and a subject of a [couple](https://flapenguin.me/elf-dt-gnu-hash) of [blog](http://deroko.phearless.org/dt_gnu_hash.txt) posts that I’ve [found](https://blogs.oracle.com/solaris/post/gnu-hash-elf-sections). Turns out it also doesn’t provide the same functionality as `DT_HASH`. To get the number of symbols you have to [completely parse it](https://groups.google.com/g/generic-abi/c/th5919osPAQ/m/hRL78ooqBAAJ).

It’s worth noting that unless you are building `ELFOSABI_NONE` binaries you are [not forced to follow the generic ABI](https://groups.google.com/g/generic-abi/c/th5919osPAQ/m/CWQyAu36AwAJ). Glibc uses `ELFOSABI_GNU` so technically not including `DT_HASH` is fine.

Glibc was forcing having both sections for a very long time claiming to maintain compatibility. It was [recently dropped](https://sourceware.org/git/?p=glibc.git;a=commitdiff;h=e47de5cb2d4dbecb58f569ed241e8e95c568f03c) and the build instead relies on “the defaults”.

## The Regression(s)

This problem was first noticed by rolling-release distros users when [games using EAC EOS stopped working](https://github.com/ValveSoftware/Proton/issues/6051). There’s also an open source project that has regressed - a frame rate limiter called [libstrangle](https://bugs.gentoo.org/863863). The non-EAC game [Shovel Knight is broken as well](https://github.com/ValveSoftware/Proton/issues/6051#issuecomment-1212748397).

Those are only a few breakages users of the more bleeding-edge distros found shortly after the Glibc 2.36 release. I suspect there will be more broken games (and other software) once 2.36 will hit the mainstream.

## To ABI Or Not To ABI

I suggest you read through all the linked discussions and develop your own opinion on who is responsible for what exactly.

Personally I share [Linus’ opinion](https://youtu.be/5PmHRSeA2c8?t=509) that changing ABI is okay as long as no one has noticed, but once it gets noticed - then it’s a regression. Once it’s a thing people depend on, it becomes a feature.

Sadly not everything can be easily modified to accommodate or recompiled.

Shifting the problem downstream onto the distributions is also problematic and [adds to fragmentation](https://gitlab.com/torkel104/libstrangle/-/issues/59).

I know that [`DT_GNU_HASH` is a thing for 16 years already](https://sourceware.org/pipermail/libc-alpha/2022-August/141304.html) and most distributions have switched to using it and only it by default. For those 16 years, it was Glibc who provided the compatibility and overrode the defaults for everyone and there never were any easy-to-spot deprecation warnings. It’s also unrealistic to expect every ELF consumer to keep up with undocumented developments of the format.

## Why `--hash-style=gnu` is The Default?

Kinda by accident? GCC’s default behavior is to allow linkers to do whatever they want. The GNU Linker defaults to “both”

```
ld --help | grep hash-style
  --hash-style=STYLE          Set hash style to sysv/gnu/both.  Default: both 
```

while [mold defaults to “sysv”](https://github.com/rui314/mold/blob/415673a49e24b1f4189000b9eb4fb1a36a697093/elf/mold.h#L1475). So why most things are built with –hash-style=gnu? On a lot of distributions gcc -v claims that

```
Configured with: /build/gcc/src/gcc/configure ... --with-linker-hash-style=gnu ... 
```

which seems to be a leftover from the days long gone where `DT_GNU_HASH` was so young that ld’s default was still sysv. Distributions wanted to have the promised speedup so they have just forced this and it stuck.

This seems to spread further - clang does distro detection and [seems to be mimicking](https://github.com/llvm/llvm-project/blob/b405407/clang/lib/Driver/ToolChains/Linux.cpp#L241-L244) whatever the distro is doing with their default gcc.

## Final Thoughts

I think this whole situation shows why creating native games for Linux is challenging. It’s hard to blame developers for targeting Windows and relying on Wine + friends. It’s just much more stable and much less likely to break and stay broken.