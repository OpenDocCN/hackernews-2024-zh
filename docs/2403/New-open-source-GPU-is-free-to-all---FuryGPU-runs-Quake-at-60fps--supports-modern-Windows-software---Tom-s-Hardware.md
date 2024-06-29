<!--yml

类别：未分类

日期：2024-05-29 12:44:55

-->

# 新的开源 GPU 对所有人免费 - FuryGPU 在 Quake 中以 60fps 运行，支持现代 Windows 软件 | Tom's Hardware

> 来源：[https://www.tomshardware.com/pc-components/gpus/new-open-source-gpu-is-free-to-all-supports-modern-windows-software-stack-runs-on-an-fpga-with-custom-pcb](https://www.tomshardware.com/pc-components/gpus/new-open-source-gpu-is-free-to-all-supports-modern-windows-software-stack-runs-on-an-fpga-with-custom-pcb)

一个开源的完全定制 GPU 在经过四年的秘密开发后现世。[FuryGPU](https://www.furygpu.com/) 是游戏软件开发者迪伦·巴里独自完成的一个项目，他表示这是他业余时间完成的一个极其复杂的硬件和软件项目。FuryGPU 基于 Xilinx FPGA 设计，目前的原型 PCIe 显卡在《Quake Timedemo》中可以达到约 44fps。在受到[本·伊特](https://eater.net/8bit)从零开始构建可编程 8 位计算机项目的启发后，巴里开始了 FuryGPU 的工作。

正如您在本文中看到的图片，FuryGPU 看起来非常像大约 20 年前的典型 PC 显卡，通过配备 [DisplayPort 和 HDMI](https://www.tomshardware.com/features/displayport-vs-hdmi-better-for-gaming) 输出进行现代化。然而，该项目远不止于硬件，巴里承认，该显卡设计中最令人痛苦的部分是创建 Windows 驱动程序。

## 硬件，从单板机到显卡

巴里开始实现从零开始构建 GPU 的梦想，之后他拿起了搭载 FPGA 的 Arty Z7 开发板进行了一些初步开发和测试。随后，[Xilinx](https://www.tomshardware.com/news/amd-xilinx-acqusition-completed) Kria System-on-Modules (SoMs) 的推出为项目注入了新的活力，这些模块结合了“非常便宜的 [Zynq UltraScale+](https://www.xilinx.com/products/silicon-devices/soc/zynq-ultrascale-mpsoc.html) FPGA、大量的 DSP 单元和（相对而言）大量的 LUT 和 FF，以及特别重要的硬化 PCIe 核心”，巴里兴奋地说道。

要从这个单板机到我们在 2024 年看到的 FuryGPU PCIe 插卡设计，巴里学会了 SystemVerilog 硬件描述和硬件验证语言，以及 KiCAD EDA / 电子 CAD 软件套件。他说，即使有 FPGA 电路集成到 SoM 中，设计今天我们看到的 FuryGPU 带有 4 路 PCIe 的原理图也需要极大的努力。现在，是时候将 FuryGPU 插入他的测试设备中，编写驱动程序并测试游戏了。

(图片来源：迪伦·巴里 - FuryGPU)

## Windows 驱动程序和 Quake 以 60fps 运行

巴里描述为“最痛苦”的整个项目过程是为 FuryGPU 创建 Windows 驱动程序，尽管他过去 14 年在游戏开发行业的软件渲染方面有丰富的经验。

最初，FuryGPU 制造者的雄心是组装一个简单的旋转立方体演示，以展示GPU的工作。然而，随着项目的发展，以可玩的帧率玩传奇PC游戏 Quake 开始成为新目标。

Barrie 解释说，在准备好 Windows 驱动程序之后，他编写了一个自定义的图形API来与GPU通信，为显示和音频编写了Windows内核驱动程序，现在已经有“一个可以以稳定的60帧每秒渲染Quake的完全功能图形硬件”。

获取 Tom's Hardware 的最佳新闻和深度评论，直接到您的收件箱。

我们嵌入了 Barrie 的 Quake 时间演示视频捕捉，展示 FuryGPU 大约一个月前在这个基准测试中可以达到44fps 的性能，分辨率为720p。开发者表示，有明显的优化机会可以使 Quake “运行得更快”，因为他看到了一些明显的瓶颈，他将为优化工作着手。

FuryGPU Windows 驱动程序支持视频和音频输出（图片来源：Dylan Barrie - FuryGPU）。

FuryGPU 将开源。在周三的 [黑客新闻](https://news.ycombinator.com/item?id=39836745) 帖子中，Barrie 写道：“我计划在某个时候开源整个堆栈（PCB原理图/布局，所有的HDL，Windows WDDM驱动程序，API运行时驱动程序，以及用于使用该API的Quake端口），但是存在一些法律问题。”由于他从事相关的职业，他希望确保这些工作不会违反他的工作合同或许可等问题。对于那些特别感兴趣的人，同一主题中还包含有关FuryGPU项目的相当多的额外细节。

在 FuryGPU 网站上，有一篇文章专门讨论了GPU的纹理单元，供希望深入了解其架构的人参考。

总结我们对这个对我们来说新颖的项目的报道，值得解释一下 FuryGPU 项目的预期范围。显然，这是一个制造商项目，类似于面包板CPU，但 FuryGPU 的性能如此出色，以至于一些人可能会误以为它是一个严肃的新 [GPU架构](https://www.tomshardware.com/news/amd-rdna-3-gpu-architecture-deep-dive-the-ryzen-moment-for-gpus)。Barrie 在前述的黑客新闻主题中明确指出这并不是事实（使用 PfhorSlayer 的笔名）。FuryGPU 制造者断言：“这只是一个玩具，这不会改变GPU领域或与任何商业参与者竞争。”

尽管 FuryGPU（或其后代）可能永远不会出现在 [最佳显卡](https://www.tomshardware.com/reviews/best-gpus,4380.html) 的排行榜上，但我们将对 FuryGPU 的发展非常感兴趣。现在该项目已经公开，公众关注度和专家合作者的加入可能会加速已有计划的实施。
