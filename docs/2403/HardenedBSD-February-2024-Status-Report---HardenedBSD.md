<!--yml

类别：未分类

日期：2024-05-27 14:32:13

-->

# HardenedBSD 2024 年 2 月状态报告 | HardenedBSD

> 来源：[https://hardenedbsd.org/article/shawn-webb/2024-03-01/hardenedbsd-february-2024-status-report](https://hardenedbsd.org/article/shawn-webb/2024-03-01/hardenedbsd-february-2024-status-report)

我大部分时间都花在了让 15-CURRENT 再次工作上。FreeBSD 引入了一个新的库，libsys，这是执行系统调用的用户空间部分的实现。在 libsys、libc、libthr 和 CSU 之间有一个复杂的交互。我花了一些时间了解这个交互，并且仍然觉得有更多要学习的地方。

HardenedBSD 15-CURRENT 大部分问题已修复。在 libsys 更改之前，我们使用 Link-Time Optimizations (LTO) 构建了 libc。使用 LTO 构建 libc 是问题的一部分，尽管不是唯一的问题。一旦所有问题解决，我将重新启用 libc 的 LTO。

FreeBSD 也引入了一个新的 pam_xdg(8) PAM 模块。这个模块有一些漏洞，在 HardenedBSD 中已修复。现在 FreeBSD 中也修复了两个空指针解引用漏洞。文件系统竞争条件和递归限制问题在 HardenedBSD 中有所缓解，但并非完全修复。

HardenedBSD 现在拥有两款 VisionFive StarFive2 64 位 RISCV SBC。我花了一些时间玩弄它们。内核启动到了 mountroot 提示符。我一直想学习硬件黑客技术，包括编写驱动程序，所以这些小型 SBC 可能非常适合这个目的。

**在端口中：**

1.  修复了 u-boot 端口

1.  dns/unbound 已更新至 1.19.1

1.  修复了 net/vnstat 端口

1.  graphics/mupdf 现在作为启用了 SafeStack 的 PIE 构建

1.  更新了 secadm 端口

展望三月份：我希望填补两个知识空白：上面提到的交互和我想回到 jemalloc 的硬化。我计划进行一些基础设施维护--例行更新。
