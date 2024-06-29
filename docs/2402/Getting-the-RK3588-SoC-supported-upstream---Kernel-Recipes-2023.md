<!--yml

类别：未分类

日期：2024-05-27 14:51:56

-->

# 将 RK3588 SoC 上游支持 | Kernel Recipes 2023

> 来源：[https://kernel-recipes.org/en/2023/schedule/getting-the-rk3588-soc-supported-upstream/](https://kernel-recipes.org/en/2023/schedule/getting-the-rk3588-soc-supported-upstream/)

Rockchip RK3588 系统芯片（SoC）是一款令人印象深刻的旗舰产品，配备了 8 个 ARM 核心（4x A76，4x A55），大量缓存和广泛的外设。该 SoC 还包括 Arm G610 Mali GPU、多个 PCIe 通道、支持 8K 的硬件视频解码器和编码器、支持 8K 分辨率的显示引擎、6TOPs 神经处理单元（NPU）等。目前工作包括：

几个月来，Collabora 的工程师们一直在为这款 SoC 进行上游支持的工作。虽然还有很多工作要做，但基础功能已经实现。在这次演讲中，Sebastian 将介绍当前的上游状态，并分享我们如何从“内核不认识这个平台，无法引导”到“具备 UART + 网络的工作开发平台”。随后，Sebastian 将详细介绍目前正在进行的工作，如 GPU、HDMI 输出和输入支持。演示将以未来的工作事项展望作为结尾。

***Sébastien REICHEL***
