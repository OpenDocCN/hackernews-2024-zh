<!--yml

category: 未分类

date: 2024-05-27 15:17:04

-->

# 概览 — Flent：灵活的网络测试工具

> 来源：[`flent.org/`](https://flent.org/)

# 欢迎来到 Flent 的主页

Flent 是一个网络基准测试工具，允许您：

+   轻松地**运行网络测试**，将多个众所周知的基准测试工具组合成聚合、可重复的测试运行。

+   **探索**您的测试数据通过交互式 GUI 和丰富的绘图功能。

+   **合并和聚合**数据系列，并生成出版质量的图表。

+   **捕获**本地和远程主机的元数据，并将其存储在绘图数据中。

+   **收集次要数据系列**，如 CPU 使用情况、WiFi、qdisc 和 TCP 套接字统计，并将其与主数据集一起绘制。

+   指定批量实验运行以**完全自动化您的测试**制度。

Flent 是用 Python 编写的，将众所周知的网络基准测试工具（如[netperf](http://netperf.org)和[iperf](https://sourceforge.net/p/iperf2)）封装成聚合、可重复的测试，比如一些关于[Bufferbloat 的测试](https://www.bufferbloat.net/projects/bloat/wiki/Tests_for_Bufferbloat/)。

有一篇简短的论文（pdf）和一个[博客文章](https://blog.tohojo.dk/2017/04/the-story-of-flent-the-flexible-network-tester.html)描述了 Flent 的一些设计目标。

## 入门指南

根据下面的说明安装 Flent，然后查看快速入门部分开始使用。

欲了解更多信息，请参阅完整文档或搜索特定内容。

## 安装 Flent

安装 Flent 可以通过多种方式完成，具体取决于您的操作系统：

+   **Debian and Ubuntu:** `apt install flent`。要安装 netperf，请[启用非自由存储库](https://wiki.debian.org/SourcesList)。

+   **Fedora:** `dnf install flent`。

+   **Gentoo:** `emerge net-analyzer/flent`.

+   **Ubuntu 18.04 之前的版本：** 添加[tohojo/flent PPA](https://launchpad.net/~tohojo/+archive/ubuntu/flent)。

+   **Arch Linux：** 从[AUR](https://aur.archlinux.org/packages/flent)安装 Flent。

+   **FreeBSD：** `pkg install flent`来安装软件包或者`cd /usr/ports/net/flent`来安装端口。

+   **其他 Linux 和 OSX（使用 Macbrew）：** 从[Python Package Index](https://pypi.python.org/pypi/flent)安装：`pip install flent`

## 参与其中

参与很容易：
