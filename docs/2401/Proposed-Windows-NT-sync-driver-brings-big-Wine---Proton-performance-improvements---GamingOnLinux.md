<!--yml

分类：未分类

日期：2024-05-27 15:09:19

-->

# 提议的 Windows NT 同步驱动程序带来了 Wine / Proton 的大幅性能改进 | GamingOnLinux

> 来源：[https://www.gamingonlinux.com/2024/01/proposed-windows-nt-sync-driver-brings-big-wine-proton-performance-improvements/](https://www.gamingonlinux.com/2024/01/proposed-windows-nt-sync-driver-brings-big-wine-proton-performance-improvements/)

CodeWeavers 的 Zeb Figura [提出了](https://lore.kernel.org/lkml/20240124004028.16826-1-zfigura@codeweavers.com/) 一个 Windows NT 同步原语驱动程序，以帮助在 Linux 上使用 Wine / Proton 运行的针对 Windows 的游戏和应用的性能。

根据发布到 Linux 内核邮件列表的 RFC（请求评论）的内容，Figura 指出：“此补丁系列引入了一个新的 char 杂项驱动程序，/dev/ntsync，用于实现 Windows NT 同步原语。”

然后对此进行了一些背景介绍：“Wine 项目在用户空间模拟 Windows API。其中一个特定的部分是

该 API，即 NT 同步原语，过去是通过 RPC 实现到专用的“内核”进程的。但是，最近的应用程序更加强烈地使用了这些 API，RPC 的开销已经成为一个瓶颈。

NT 同步 API 太复杂，无法在现有原语的基础上实现而不损失正确性。某些操作，如 NtPulseEvent() 或 NtWaitForMultipleObjects() 的“等待全部”模式，需要直接控制底层的等待队列，并且在用户空间为 Wine 实现足够强大的等待队列是不可能的。因此，该提议的驱动程序直接在 Linux 内核中实现了有问题的接口。”

Zeb Figura 在 2023 年的 Linux Plumbers Conference 上做了一次关于此事的演讲，您可以在下面看到：

我们可能会看到类似这样的性能差异？差异取决于您的硬件和您要运行的内容，但是多个用户提供了一些 FPS 的示例：

*测试的硬件和软件没有注明。*

一些令人印象深刻的数字，但是又再次取决于硬件，但即便如此，任何性能提升都将受欢迎，因此希望在进行可能需要的任何调整后，这个补丁系列得到一些良好的评论和批准。看起来我们可能会在 Linux 和 Steam Deck 上通过 Wine 和 Proton 看到一个不错的未来性能提升。

*来源：[Phoronix](https://www.phoronix.com/news/Windows-NT-Sync-RFC-Linux)*

文章来自[GamingOnLinux.com.](https://www.gamingonlinux.com/)
