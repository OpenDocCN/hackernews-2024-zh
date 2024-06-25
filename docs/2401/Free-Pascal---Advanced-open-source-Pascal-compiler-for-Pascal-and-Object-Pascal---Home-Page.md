<!--yml

类别：未分类

日期：2024 年 05 月 27 日 14:53:52

-->

# 自由帕斯卡 - 高级开源帕斯卡编译器 - 主页

> 来源：[`www.freepascal.org/`](https://www.freepascal.org/)

# 简介

## 概述

Free Pascal 是一个成熟、多功能、开源的帕斯卡编译器。它可以针对许多处理器架构：Intel x86（16 位和 32 位）、AMD64/x86-64、PowerPC、PowerPC64、SPARC、SPARC64、ARM、AArch64、MIPS、Motorola 68k、AVR 和 JVM。支持的操作系统包括 Windows（16/32/64 位、CE 和本机 NT）、Linux、Mac OS X/iOS/iPhoneSimulator/Darwin、FreeBSD 和其他 BSD 变体、DOS（16 位或 32 位 DPMI）、OS/2、AIX、Android、Haiku、Nintendo GBA/DS/Wii、AmigaOS、MorphOS、AROS、Atari TOS 和各种嵌入式平台。此外，开发版本还提供对 RISC-V（32/64）、Xtensa 和 Z80 架构以及 LLVM 编译器基础设施的支持。此外，Free Pascal 团队维护了一个将 Pascal 转换为 Javascript 的转译器，名为 pas2js。

## 最新消息

+   *2024 年 1 月 4 日*

+   帕斯卡语言的创造者 Niklaus Wirth 在 1 月 1 日去世。没有 Niklaus Wirth 的工作，Free Pascal 将不会存在。我们悼念一位先驱和灵感之源。

+   *2021 年 8 月 8 日*

+   FPC 已经迁移到 Gitlab！

    所有 SVN 存储库都已转换为 git 并移至 gitlab。Mantis bugtracker 也已转换为 [gitlab](https://gitlab.com/freepascal.org/fpc/)。

    您可以在开发页面或[Wiki](https://wiki.freepascal.org/FPC_git)中找到说明。

    Bug 可以在[这里](https://gitlab.com/groups/freepascal.org/fpc/-/issues)报告。

+   *2021 年 5 月 20 日*

+   FPC 版本 3.2.2 已发布！

    此版本是对 3.2.0 的点更新，包含了修复错误和更新包，其中一些是高优先级的。在这种情况下，还从 trunk 回溯了一个新的目标。

    这里有一份[可能破坏向后兼容性的更改列表](http://wiki.freepascal.org/User_Changes_3.2.2)。您也可以查看 FPC 3.2.2 文档。

    下载可在下载部分找到。一些链接可能已经过时，但将在未来几天内更新。如果您由于最近的浏览器更新而无法使用 FTP，请尝试 SourceForge 镜像。

+   *2020 年 6 月 19 日**   *2019 年 7 月 20 日**   *2018 年 6 月 8 日*

+   今天 FPC 庆祝它的 25 岁生日！

    自 1993 年 6 月 8 日以来已经过去了 25 年，而 FPC 不仅仍然存在，而且比以往任何时候都更加充满活力！

+   *2018 年 5 月 28 日*

更早的新闻...

## 当前版本

版本*3.2.2*是自由帕斯卡最新的稳定版本。点击下载链接，选择一个靠近您的镜像下载您的副本。开发版本的版本号为*3.3.x*。请查看开发页面以了解如何获取最新源代码并支持开发。

## 特点

语言语法与 TP 7.0 以及大多数 Delphi 版本（类、rtti、异常、ansistrings、widestrings、接口）具有良好的兼容性。还提供了与 Think Pascal 和 MetroWerks Pascal 大部分兼容的 Mac Pascal 模式。此外，Free Pascal 支持函数重载、操作符重载、全局属性和其他几个额外功能。

## 要求

**x86 架构：**

> 对于 80x86 版本，至少需要 386 处理器，但建议使用 486 处理器。Mac OS X 版本需要 Mac OS X 10.4 或更高版本，并安装了开发工具。

**PowerPC 架构：**

> 任何 PowerPC 处理器都可以。需要 16 MB 的 RAM。Mac OS 经典版本预计能在 System 7.5.3 及更高版本上运行。Mac OS X 版本需要 Mac OS X 10.3 或更高版本（可以编译为 10.2.8 或更高版本），并安装了开发工具。在其他操作系统上，Free Pascal 可以在能够运行该操作系统的任何系统上运行。

**ARM 架构**

> 需要 16 MB 的 RAM。可以在任何 ARM Linux 安装上运行。

**Sparc 架构**

> 需要 16 MB 的 RAM。可以在任何 Sparc Linux 安装上运行（solaris 是实验性的）。

## 许可证

包和运行库使用修改后的 GNU Library 公共许可证，允许在创建应用程序时使用静态库。编译器源代码本身采用 GNU 通用公共许可证。编译器和运行库的源代码都是可用的；完整的编译器是用 Pascal 编写的。
