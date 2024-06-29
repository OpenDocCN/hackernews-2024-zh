<!--yml

category: 未分类

date: 2024-05-27 12:48:18

-->

# 一个 awk 实现 | 一种相对紧凑的 awk 编程语言实现

> 来源：[https://www.raygard.net/awkdoc/](https://www.raygard.net/awkdoc/)

<main>

# [](#an-awk-implementation)一个 awk 实现

Rob Landley 的[toybox](https://landley.net/toybox/) [项目](https://github.com/landley/toybox)提供了各种类似于[busybox](https://busybox.net/)的 Linux 命令行工具。我写了一个紧凑但相当完整的 `awk` 实现，旨在与 toybox 集成，但也可以独立构建。

这个实现被命名为 `wak`，因为所有好的 `awk` 名称都被占用了。但在 toybox 中使用时，它只是 `awk` 或 `toybox awk`。

`wak` 是用 C99 编写的。它主要使用 C 标准库，也就是 `libc`，但还需要一些 POSIX 支持，主要用于正则表达式。

它不是公共领域，但有 Landley 非常宽松的[0BSD](https://landley.net/toybox/license.html)许可证。

这些页面描述了我的 awk 实现，但并不完整。

这个实现位于[github](https://github.com/raygard/wak/)。

</main>
