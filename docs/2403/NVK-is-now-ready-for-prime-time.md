<!--yml

分类：未分类

日期：2024-05-27 14:30:13

-->

# NVK 现在已经准备好投入实际使用

> 来源：[https://www.collabora.com/news-and-blog/news-and-events/nvk-is-now-ready-for-prime-time.html](https://www.collabora.com/news-and-blog/news-and-events/nvk-is-now-ready-for-prime-time.html)

Faith Ekstrand

2024年2月28日

NVK，即[NVIDIA 硬件在 Mesa 上的开源 Vulkan 驱动](https://www.collabora.com/news-and-blog/news-and-events/introducing-nvk.html)，现在正式发布。我刚刚提交了一个[合并请求](https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/27832)，消除了不符合规范的实现警告，并将 NVK 的 Meson 配置选项从`nouveau-experimental`修改为`nouveau`。这将作为一个信号传递给发行版，表明现在是向用户提供 NVK 的时候了。我不能代表发行版发言，但 NVK 将成为 Mesa 24.1 的一部分，您应该期待它出现在您喜爱的 Linux 发行版的春季或秋季发布中。

去年十月，我宣布[NVK 已达到 Vulkan 1.0 的一致性](https://www.collabora.com/news-and-blog/news-and-events/nvk-reaches-vulkan-conformance.html)，适用于 Turing 硬件。截至今日，NVK 现在在[Turing（RTX 2000 和 GTX 1600 系列）](https://www.khronos.org/conformance/adopters/conformant-products/vulkan#submission_768)，[Ampere（RTX 3000 系列）](https://www.khronos.org/conformance/adopters/conformant-products/vulkan#submission_769)和[Ada（RTX 4000 系列）](https://www.khronos.org/conformance/adopters/conformant-products/vulkan#submission_770) GPU 上实现了 Vulkan 1.3 的一致性。我们不仅向前跃进了三个 Vulkan 版本，而且这次的新测试是在启用了 GSP 固件的情况下进行的，包括 Ampere 和 Ada GPU。与最初的 1.0 版本不同的是，这次测试没有任何破解。我们在这些一致性测试中通过的每个测试现在也在上游 Mesa 上通过。

过去几个月，我们也在努力完成所需的最后部分，以便 DXVK 现在可以在上游 Mesa 上直接运行。并非每个 D3D11 游戏都保证可以正常运行（可能会有 bug），但核心要求都已满足。我们正在积极地处理剩余的问题，以支持通过 VKD3D-Proton 进行 D3D12 模拟。目前已经完成或正在进行的工作很多，但仍有少数部分尚未完成，所以暂时不要期望 D3D12 游戏可以正常运行。

关于 OpenGL 支持，我们仍然期待 Zink + NVK 继续推进。目前并非所有功能都已完全实现，但我们也在积极地解决剩余的 Zink 问题，以通过 Zink 在 NVK 上提供 OpenGL 4.6 支持。虽然旧的 nouveau OpenGL 驱动程序仍将继续存在并正常工作，但 Zink + NVK 在许多情况下已经超越了旧的 OpenGL 驱动程序的性能。从长远来看，我们期望它能够提供更多功能和更好的稳定性。

性能仍在不断改进和提升中。许多标题在最新的GPU上以60 FPS或更高的帧率运行。而其他一些，则出现了我们尚未解决的瓶颈问题。如果你想知道你喜欢的游戏表现如何，最好的方法就是亲自试一试。
