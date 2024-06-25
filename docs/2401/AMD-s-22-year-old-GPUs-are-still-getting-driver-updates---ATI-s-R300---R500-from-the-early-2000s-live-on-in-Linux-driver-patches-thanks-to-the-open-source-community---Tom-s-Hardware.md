<!--yml

category: 未分类

date: 2024-05-27 14:35:29

-->

# AMD的22年前的GPU仍在获得驱动程序更新 —— ATI的R300 - R500从2000年代初期起在Linux驱动程序补丁中继续存在，得益于开源社区 | 汤姆硬件

> 来源：[https://www.tomshardware.com/pc-components/gpus/amd-22-year-old-gpus-are-still-getting-driver-updates-atis-r300-r500-from-the-early-2000s-live-on-in-linux-driver-patches-thanks-to-the-open-source-community](https://www.tomshardware.com/pc-components/gpus/amd-22-year-old-gpus-are-still-getting-driver-updates-atis-r300-r500-from-the-early-2000s-live-on-in-linux-driver-patches-thanks-to-the-open-source-community)

[Phoronix报道](https://www.phoronix.com/news/ATI-R300g-2024-Action)说，针对[ATI](https://www.tomshardware.com/tag/ati)的古老的R300到R500系列Radeon GPU的驱动程序更新正在今年实现到Linux中。尽管它们的年龄很大，但开源社区正在使用开源[驱动程序](https://www.tomshardware.com/tag/drivers)，使这些GPU继续在现代Linux操作系统上运行。

Linux驱动程序更新将NIR降低相关的大部分后端降低放入其中，这与顶点着色器有关。这个驱动程序更新将在Mesa 24.0中的本季度推出，这意味着任何仍在使用R300-R500系列Radeon GPU的人将在今年晚些时候获得它。

*"这个MR将大部分剩余的后端降低到NIR中。具体来说，ftrunc，fcsel（适当时）和flrp。后端降低路径被移除。这是进行更多后端清理的先决条件，例如，我准备好了一个MR来摆脱顶点着色器的后端DCE。" -- Pavel Ondračka*

ATI R300于2002年首次亮相，作为[ATI Radeon 9700 PRO](https://www.tomshardware.com/reviews/ati-radeon-9700-pro,505-3.html)的一部分。GPU采用AGP接口（与PCIe相比较古老），150纳米工艺，1.1亿晶体管，325MHz核心时钟，256MB内存，以及19.8GB/s的内存带宽。

在推出时，Radeon 9700 Pro是当时最快的GPU，几乎在所有工作负载中都击败了Nvidia的对手GeForce 4 Ti 4600。它还是当时最先进的GPU，是第一个完全支持[DirectX](https://www.tomshardware.com/tag/directx) 9的GPU。

随后推出的R400和R500系列GPU大多是对R300 GPU架构的优化，配备了更多的像素流水线、顶点着色器引擎和速度更快、功能更强大的内存配置。

令人惊讶的是，仍然有人在使用这些显卡，更不用说在现代Linux操作系统中支持它们了，正如开源社区所做的那样。显然，这些GPU在现代操作系统上无法做太多事情，除了显示窗口和文本之外。但仍然很酷，一个来自2002年的GPU能够在现代操作系统上运行。

Get Tom's Hardware's best news and in-depth reviews, straight to your inbox.
