<!--yml

category: 未分类

date: 2024-05-29 12:46:43

-->

# 在 Linux 上，Win32 是唯一稳定的 ABI

> 来源：[https://blog.hiler.eu/win32-the-only-stable-abi/](https://blog.hiler.eu/win32-the-only-stable-abi/)

# 在 Linux 上，Win32 是唯一稳定的 ABI

2022-08-15，由 ivyl

## TL;DR

+   Glibc 2.36 发布后，使用 EAC EOS 的所有游戏都不再可玩。

+   有用户进行了二分法，找到了[有问题的提交](https://github.com/ValveSoftware/Proton/issues/6051#issuecomment-1206888814)。

+   我已经提交了[一个 Glibc bug](https://sourceware.org/bugzilla/show_bug.cgi?id=29456)，希望能快速恢复。

+   结果还有其他问题也出现了。

+   很可能会一直保持这种情况。

+   在 Linux 上，Win32（通过 Wine + 其他工具）是唯一稳定的 ABI :-P

## `DT_HASH` 和 `DT_GNU_HASH`

在 ELF 格式中，提供符号哈希表的两种方法。`DT_HASH` 和 `DT_GNU_HASH`。第一种是[SYSV 通用 ABI](http://www.sco.com/developers/gabi/latest/ch5.dynamic.html#hash) 的一部分，有着良好的文档记录，并标记为强制性。`DT_GNU_HASH` 是一种更新、更小、更快的替代品，它[没有被文档化](https://sourceware.org/legacy-ml/binutils/2006-10/msg00377.html)，在几个地方实现（Glibc、GNU ld、Musl、LLVM、mold 等），并且是[几篇](https://flapenguin.me/elf-dt-gnu-hash) [博客](http://deroko.phearless.org/dt_gnu_hash.txt) [文章](https://blogs.oracle.com/solaris/post/gnu-hash-elf-sections) 的主题。事实证明，它也没有提供与 `DT_HASH` 相同的功能。要获取符号数，您必须[完全解析它](https://groups.google.com/g/generic-abi/c/th5919osPAQ/m/hRL78ooqBAAJ)。

值得注意的是，除非您正在构建 `ELFOSABI_NONE` 二进制文件，否则您[无需遵循通用 ABI](https://groups.google.com/g/generic-abi/c/th5919osPAQ/m/CWQyAu36AwAJ)。Glibc 使用 `ELFOSABI_GNU`，因此技术上不包括 `DT_HASH` 是可以的。

Glibc 强制长时间要求同时保留两个部分以维持兼容性。它[最近已经被取消](https://sourceware.org/git/?p=glibc.git;a=commitdiff;h=e47de5cb2d4dbecb58f569ed241e8e95c568f03c)，而构建现在依赖于“默认设置”。

## 回归问题

这个问题最初是由 rolling-release 发行版用户注意到的，当时[使用 EAC EOS 的游戏停止工作](https://github.com/ValveSoftware/Proton/issues/6051)。还有一个开源项目出现了回退 - 一个叫做[libstrangle](https://bugs.gentoo.org/863863)的帧率限制器。非 EAC 游戏[Shovel Knight 也出了问题](https://github.com/ValveSoftware/Proton/issues/6051#issuecomment-1212748397)。

这些只是在 Glibc 2.36 发布后，更 bleeding-edge 的发行版用户发现的几个破坏。我怀疑一旦 2.36 成为主流，会有更多游戏（和其他软件）出现问题。

## ABI 还是非 ABI

我建议您阅读所有链接的讨论，并对每个问题的责任形成自己的看法。

就我个人而言，我赞同[Linus的观点](https://youtu.be/5PmHRSeA2c8?t=509)，即改变ABI是可以接受的，只要没有人注意到，但一旦被注意到，那么它就是一个退步。一旦成为人们依赖的东西，它就变成了一个特性。

不幸的是，并非一切都可以轻松修改以适应或重新编译。

将问题转嫁给发行版也是问题的一部分，并且[增加了分裂性](https://gitlab.com/torkel104/libstrangle/-/issues/59)。

我知道[`DT_GNU_HASH`已经存在了16年了](https://sourceware.org/pipermail/libc-alpha/2022-August/141304.html)，大多数发行版已经默认使用它，并且只使用它。在这16年里，是Glibc提供了兼容性，并为所有人覆盖了默认设置，从来没有出现过容易发现的弃用警告。期望每一个ELF消费者跟上格式未经记录的发展也是不现实的。

## 为什么`--hash-style=gnu`是默认值？

有点偶然吗？GCC的默认行为是允许链接器做任何它们想做的事情。GNU链接器的默认值是“both”，

```
ld --help | grep hash-style
  --hash-style=STYLE          Set hash style to sysv/gnu/both.  Default: both 
```

而[mold默认为“sysv”](https://github.com/rui314/mold/blob/415673a49e24b1f4189000b9eb4fb1a36a697093/elf/mold.h#L1475)。那么为什么大多数东西都使用`--hash-style=gnu`？在很多发行版中，gcc -v 声称

```
Configured with: /build/gcc/src/gcc/configure ... --with-linker-hash-style=gnu ... 
```

这似乎是在DT_GNU_HASH还年轻得多的时代的遗留物，ld的默认值仍然是sysv。发行版希望获得承诺的加速，因此他们强制执行了这一点并且坚持了下来。

这似乎正在扩散 - clang进行了发行版检测，并[似乎在模仿](https://github.com/llvm/llvm-project/blob/b405407/clang/lib/Driver/ToolChains/Linux.cpp#L241-L244)发行版使用的默认gcc。

## 最后的思考

我认为整个情况说明了为Linux创建本地游戏的挑战性。很难责怪开发者将目标对准Windows并依赖Wine和朋友们。这只是更加稳定，更不容易出现问题和继续保持问题。
