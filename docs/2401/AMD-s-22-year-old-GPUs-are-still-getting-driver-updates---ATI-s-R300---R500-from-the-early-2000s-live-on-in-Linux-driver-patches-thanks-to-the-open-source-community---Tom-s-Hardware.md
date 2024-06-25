<!--yml

category: 未分类

date: 2024-05-27 14:35:29

-->

# AMD 的 22 年前的 GPU 仍在获得驱动程序更新 —— ATI 的 R300 - R500 从 2000 年代初期起在 Linux 驱动程序补丁中继续存在，得益于开源社区 | 汤姆硬件

> 来源：[`www.tomshardware.com/pc-components/gpus/amd-22-year-old-gpus-are-still-getting-driver-updates-atis-r300-r500-from-the-early-2000s-live-on-in-linux-driver-patches-thanks-to-the-open-source-community`](https://www.tomshardware.com/pc-components/gpus/amd-22-year-old-gpus-are-still-getting-driver-updates-atis-r300-r500-from-the-early-2000s-live-on-in-linux-driver-patches-thanks-to-the-open-source-community)

[Phoronix 报道](https://www.phoronix.com/news/ATI-R300g-2024-Action)说，针对[ATI](https://www.tomshardware.com/tag/ati)的古老的 R300 到 R500 系列 Radeon GPU 的驱动程序更新正在今年实现到 Linux 中。尽管它们的年龄很大，但开源社区正在使用开源[驱动程序](https://www.tomshardware.com/tag/drivers)，使这些 GPU 继续在现代 Linux 操作系统上运行。

Linux 驱动程序更新将 NIR 降低相关的大部分后端降低放入其中，这与顶点着色器有关。这个驱动程序更新将在 Mesa 24.0 中的本季度推出，这意味着任何仍在使用 R300-R500 系列 Radeon GPU 的人将在今年晚些时候获得它。

*"这个 MR 将大部分剩余的后端降低到 NIR 中。具体来说，ftrunc，fcsel（适当时）和 flrp。后端降低路径被移除。这是进行更多后端清理的先决条件，例如，我准备好了一个 MR 来摆脱顶点着色器的后端 DCE。" -- Pavel Ondračka*

ATI R300 于 2002 年首次亮相，作为[ATI Radeon 9700 PRO](https://www.tomshardware.com/reviews/ati-radeon-9700-pro,505-3.html)的一部分。GPU 采用 AGP 接口（与 PCIe 相比较古老），150 纳米工艺，1.1 亿晶体管，325MHz 核心时钟，256MB 内存，以及 19.8GB/s 的内存带宽。

在推出时，Radeon 9700 Pro 是当时最快的 GPU，几乎在所有工作负载中都击败了 Nvidia 的对手 GeForce 4 Ti 4600。它还是当时最先进的 GPU，是第一个完全支持[DirectX](https://www.tomshardware.com/tag/directx) 9 的 GPU。

随后推出的 R400 和 R500 系列 GPU 大多是对 R300 GPU 架构的优化，配备了更多的像素流水线、顶点着色器引擎和速度更快、功能更强大的内存配置。

令人惊讶的是，仍然有人在使用这些显卡，更不用说在现代 Linux 操作系统中支持它们了，正如开源社区所做的那样。显然，这些 GPU 在现代操作系统上无法做太多事情，除了显示窗口和文本之外。但仍然很酷，一个来自 2002 年的 GPU 能够在现代操作系统上运行。

Get Tom's Hardware's best news and in-depth reviews, straight to your inbox.
