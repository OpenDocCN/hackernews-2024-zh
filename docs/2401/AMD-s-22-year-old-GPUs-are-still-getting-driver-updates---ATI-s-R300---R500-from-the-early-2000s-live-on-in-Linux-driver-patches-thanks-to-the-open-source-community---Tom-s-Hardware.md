<!--yml
category: 未分类
date: 2024-05-27 14:35:29
-->

# AMD's 22-year-old GPUs are still getting driver updates — ATI's R300 - R500 from the early 2000s live on in Linux driver patches thanks to the open-source community | Tom's Hardware

> 来源：[https://www.tomshardware.com/pc-components/gpus/amd-22-year-old-gpus-are-still-getting-driver-updates-atis-r300-r500-from-the-early-2000s-live-on-in-linux-driver-patches-thanks-to-the-open-source-community](https://www.tomshardware.com/pc-components/gpus/amd-22-year-old-gpus-are-still-getting-driver-updates-atis-r300-r500-from-the-early-2000s-live-on-in-linux-driver-patches-thanks-to-the-open-source-community)

[Phoronix reports](https://www.phoronix.com/news/ATI-R300g-2024-Action) that driver updates for [ATI](https://www.tomshardware.com/tag/ati)'s ancient R300 through R500 series Radeon GPUs are being implemented into Linux this year. Despite their old age, the open-source community is keeping these GPUs alive with open-source [drivers](https://www.tomshardware.com/tag/drivers), enabling them to continue to run on modern Linux operating systems.

The Linux driver update makes a change to NIR lowering, which is related to the vertex shaders. The driver update will be available this quarter in Mesa 24.0, meaning anyone still using an R300-R500 series Radeon GPU will have it later this year.

*"This MR moves the most of the remaining backend lowering into NIR. Specifically, ftrunc, fcsel (when suitable) and flrp. The backend lowering paths are removed. This is a prerequisite for more backend cleanups, for example I have a MR ready to get rid of backend DCE for vertex shaders." -- Pavel Ondračka*

The ATI R300 debuted in 2002 as part of the [ATI Radeon 9700 PRO.](https://www.tomshardware.com/reviews/ati-radeon-9700-pro,505-3.html) The GPU featured an AGP interface (an ancient competitor to PCIe), 150nm process, 110 million transistors, 325MHz core clock, 256MB of memory, and 19.8GB/s of memory bandwidth.

When it launched, the Radeon 9700 Pro was the fastest GPU of its time, beating Nvidia's counterpart, the GeForce 4 Ti 4600, in almost every workload. It was also the most technologically advanced GPU at the time, being the first GPU to support [DirectX](https://www.tomshardware.com/tag/directx) 9 in its entirety.

The R400 and R500 series GPUs that would follow afterward were mostly optimizations of the R300's GPU architecture, sporting far more pixel pipelines, vertex shader engines, and faster, more capable memory configurations.

It is amazing that anyone is still running one of these cards, let alone supporting them in modern Linux operating systems, as the open-source community has done. Obviously, these GPUs can't do much on modern operating systems, other than display windows and text. But it is cool, nevertheless, that a GPU from 2002 can run a modern operating system at all.

Get Tom's Hardware's best news and in-depth reviews, straight to your inbox.