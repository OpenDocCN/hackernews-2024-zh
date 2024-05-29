<!--yml
category: 未分类
date: 2024-05-27 14:30:13
-->

# NVK is now ready for prime time

> 来源：[https://www.collabora.com/news-and-blog/news-and-events/nvk-is-now-ready-for-prime-time.html](https://www.collabora.com/news-and-blog/news-and-events/nvk-is-now-ready-for-prime-time.html)

Faith Ekstrand
February 28, 2024

Today, I'm proud to announce that NVK, the [open-source Vulkan driver for NVIDIA hardware in Mesa](https://www.collabora.com/news-and-blog/news-and-events/introducing-nvk.html), is now ready for prime time. I just landed a [merge request](https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/27832) which gets rid of the non-conformant implementation warnings and changes the Meson configuration option for NVK from `nouveau-experimental` to just `nouveau`. This will act as a signal to distros that it's now time to start shipping NVK to users. I can't speak on behalf of distros, but NVK will be part of Mesa 24.1 and you should expect to see it in either the spring or fall release of your favorite Linux distro.

Back in october, I announced that [NVK had reached Vulkan 1.0 conformance](https://www.collabora.com/news-and-blog/news-and-events/nvk-reaches-vulkan-conformance.html) on Turing hardware. As of today NVK is now a conformant Vulkan 1.3 implementation on [Turing (RTX 2000 and GTX 1600 series)](https://www.khronos.org/conformance/adopters/conformant-products/vulkan#submission_768), [Ampere (RTX 3000 series)](https://www.khronos.org/conformance/adopters/conformant-products/vulkan#submission_769), and [Ada (RTX 4000 series)](https://www.khronos.org/conformance/adopters/conformant-products/vulkan#submission_770) GPUs. Not only have we jumped forward three Vulkan versions, but the new test runs were done with the GSP firmware enabled and includes Ampere and Ada GPUs. Also, unlike the initial 1.0 run, there are no hacks this time. Every test we passed in those conformance test runs also passes on upstream Mesa.

We've also been hard at work the last few months finishing up the final bits required so that DXVK now runs out-of-the-box on upstream Mesa. Not every D3D11 game is guaranteed to work (there will be bugs) but the core requirements are all there. We are actively working on the remaining pieces to support D3D12 emulation via VKD3D-Proton. A lot is already done or in progress but there are still a few pieces missing, so don't expect D3D12 games to work just yet.

For OpenGL support, we are still expecting Zink + NVK to be the plan going forward. Not everything works yet but we are also actively triaging the remaining Zink bugs to provide OpenGL 4.6 on top of NVK via Zink. While the old nouveau OpenGL driver will continue to exist and work as well as it ever has, Zink + NVK has already surpassed the old OpenGL in terms of performance in many cases. In the long term, we expect it to offer more features and better stability as well.

Performance is still a work in progress and continues to improve regularly. A lot of titles are running at 60 FPS or better on recent GPUs. With others, we're seeing bottlenecks that we have yet to triage. If you want to know if your favorite game performs well, the best way is to just try.