<!--yml

category: 未分类

date: 2024-05-29 13:19:30

-->

# 吉姆·凯勒批评Nvidia的CUDA和x86 — 'CUDA是沼泽，不是护城河。x86也是个沼泽' | 汤姆硬件

> 来源：[https://www.tomshardware.com/tech-industry/artificial-intelligence/jim-keller-criticizes-nvidias-cuda-and-x86-cudas-a-swamp-not-a-moat-x86-was-a-swamp-too](https://www.tomshardware.com/tech-industry/artificial-intelligence/jim-keller-criticizes-nvidias-cuda-and-x86-cudas-a-swamp-not-a-moat-x86-was-a-swamp-too)

[吉姆·凯勒](https://www.tomshardware.com/tech-industry/artificial-intelligence/jim-keller-responds-to-sam-altmans-plan-to-raise-dollar7-billion-to-make-ai-chips)，一位传奇处理器架构师，曾参与设计x86、Arm、MISC和RISC-V处理器，本周末批评了[Nvidia的CUDA](https://www.tomshardware.com/reviews/nvidia-cuda-gpu,1954.html)架构和软件堆栈，并将其类比为他所称的沼泽x86。他指出，即使是Nvidia自身也有多个依赖开源框架以提升性能的专用软件包。

"CUDA是一个沼泽，不是护城河，" 凯勒在[一篇X的帖子中](https://twitter.com/jimkxa/status/1758943525662769498)写道。"x86也是个沼泽。[…] CUDA并不美丽。它是一步步堆砌而成的。"

事实上，就像x86一样，CUDA在保持软硬件向后兼容性的同时逐步添加功能。这使得Nvidia的平台完整且向后兼容，但影响了性能并增加了程序开发的难度。与此同时，许多开源软件开发框架比CUDA更有效率。

"基本上没有人写CUDA," 凯勒在一篇后续帖子中写道。"如果你确实写了CUDA，它可能不快。[…] 有个很好的理由会有Triton、Tensor RT、Neon和Mojo。"

即使是Nvidia自身也有一些不完全依赖CUDA的工具。例如，Triton推断服务器是Nvidia的一个开源工具，简化了大规模部署AI模型的过程，支持TensorFlow、PyTorch和ONNX等框架。Triton还提供诸如模型版本控制、多模型服务和并发模型执行等功能，以优化GPU和CPU资源的利用。

Nvidia的[TensorRT](https://www.tomshardware.com/news/nvidia-boosts-ai-performance-with-tensorrt)是一个高性能的深度学习推断优化器和运行时库，加速[Nvidia GPU](https://www.tomshardware.com/reviews/gpu-hierarchy,4388.html)上的深度学习推断。TensorRT接收来自各种框架（如TensorFlow和PyTorch）的训练模型，并优化它们以进行部署，减少延迟并提高实时应用（如图像分类、物体检测和自然语言处理）的吞吐量。

但是尽管像[Arm](https://www.tomshardware.com/news/intel-foundry-and-arm-to-collaborate-on-18nm-mobile-socs)、CUDA和[x86](https://www.tomshardware.com/features/zhaoxin-kx-u6780a-x86-cpu-tested)这样的架构可能因其相对缓慢的演进、强制的向后兼容性和臃肿性而被认为是沼泽，但这些平台也没有像[GPGPU](https://www.tomshardware.com/reviews/nvidia-cuda-gpu,1954-4.html)那样碎片化得那么严重，这可能并不是一件坏事。

获取Tom's Hardware的最新资讯和深度评测，直接发送到您的收件箱。

不清楚吉姆·凯勒对AMD的[ROCm](https://www.tomshardware.com/news/amd-enables-rocm-and-pytorch-on-radeon-rx-7900-xtx)和英特尔的[OneAPI](https://www.tomshardware.com/news/intel-bringing-oneapi-to-gaming-rendering-toolkit-xe-graphics)有何看法，但显然，尽管他花了多年时间设计x86架构，他并不喜欢其未来前景。他的声明还暗示，尽管他曾在包括苹果、英特尔、AMD、Broadcom（现在的[Tenstorrent](https://www.tomshardware.com/news/tenstorrent-shares-roadmap-of-ultra-high-performance-risc-v-cpus-and-ai-accelerators)）在内的一些全球最大的芯片制造商工作过，我们可能不会很快在Nvidia的名单上看到他的名字。
