<!--yml
category: 未分类
date: 2024-05-27 14:51:56
-->

# Getting the RK3588 SoC supported upstream | Kernel Recipes 2023

> 来源：[https://kernel-recipes.org/en/2023/schedule/getting-the-rk3588-soc-supported-upstream/](https://kernel-recipes.org/en/2023/schedule/getting-the-rk3588-soc-supported-upstream/)

The Rockchip RK3588 system-on-chip (SoC) is an impressive flagship with 8 ARM cores (4x A76, 4x A55), plenty of cache, and a vast amount of peripherals. Among other things this SoC has an Arm G610 Mali GPU, multiple PCIe lanes, hardware video decoders and encoders with 8K support, a display engine supporting 8K resolution, 6TOPs Neural Processing Unit (NPU), and quite a bit more.

For several months, engineers at Collabora have been working on upstream support for this SOC. There is still much to be done, but the basics are already working. In this talk, Sebastian will present the current upstream status and share how we got from “kernel does not know this platform and cannot boot it” to “working development platform with UART + network”. Afterwards Sebastian will go through the currently ongoing work, like the GPU, HDMI output and input support. The presentation will conclude with an outlook of the open work items.

***Sébastien REICHEL***