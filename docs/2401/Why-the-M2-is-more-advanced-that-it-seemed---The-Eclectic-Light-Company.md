<!--yml

类别：未分类

日期：2024-05-27 14:48:45

-->

# M2 为什么比看起来更先进 - The Eclectic Light Company

> 来源：[`eclecticlight.co/2024/01/15/why-the-m2-is-more-advanced-that-it-seemed/`](https://eclecticlight.co/2024/01/15/why-the-m2-is-more-advanced-that-it-seemed/)

当苹果在 18 个月前的 WWDC 上推出了其 M2 芯片，即 2022 年 6 月，几乎每个人都认为它是渐进式的，比其前身 M1 快得多，但在功能上几乎没有真正的变化。在这篇文章中，紧随上周关于 [AI 支持变化](https://eclecticlight.co/2024/01/13/how-m1-macs-may-lag-behind/) 的文章之后，我深入探讨了苹果硅片内部，以发现当时我们是否低估了 M2。线索来自芯片内部 CPU 核心支持的指令集。

在比较芯片时，重点放在性能上，尤其是基准测试，这些测试相当于运动员的短跑表现，它们告诉您芯片能够多快地运行今天的任务。如果您打算将 Mac 保留超过一年，您还应该对其芯片在运行未来任务时的性能感兴趣，或者对于我们的田径运动员来说，他们是否也足够擅长田赛项目以成为优秀的十项全能运动员，他们的能力。

#### 自 M1 以来的指令变化

对于 CPU 来说，功能主要由它们运行的指令决定。今天，您可能对您的 Mac 芯片是否能够快速执行光线追踪不感兴趣，但是在几年后，M3 中 GPU 中的硬件加速光线追踪可能会产生重大影响。虽然 Apple 添加了大量自己的硬件，包括 GPU、神经引擎和其传奇的矩阵协处理器 AMX，但 CPU 核心的功能仍然是其芯片执行的许多任务的核心。这些由 Arm 在其指令集架构（ISA）中定义，并授权给苹果，在目前超过 5,000 页的手册中有详细记录。

幸运的是，Arm 将其 ISA 分为版本。M1 芯片中的 CPU 核心使用 ARMv8.5-A，而 M2 和 M3 芯片中的 CPU 核心使用 ARMv8.6-A，Arm 友好地解释了它们在 [此变更列表](https://community.arm.com/arm-community-blogs/b/architectures-and-processors-blog/posts/arm-architecture-developments-armv8-6-a) 中的主要区别 2019 年 ARMv8.6-A：

+   通用矩阵乘法（AI 和其他）

+   bfloat16 数据类型和算术指令（AI 和其他）

+   用于虚拟化的更细粒度陷阱（虚拟化）

+   用于虚拟化的等待事件陷阱（虚拟化）

+   高精度时间（1 GHz，通用）

+   扩展指针验证（安全）。

在这些中，对 bfloat16 和 General Matrix Multiply 的支持可能对用户产生最大影响。

尽管基于 M1 芯片的 Mac 在 ARMv8.6-A 推出一年后直到 2020 年 11 月才发布，但设计和开发的前导时间使得从 2019 年以来没有时间来整合 bfloat16 首次出现后的变化。

#### **bfloat16**

正如我之前 [解释过的](https://eclecticlight.co/2024/01/13/how-m1-macs-may-lag-behind/)，我们在这里处理的是三种不同的浮点数格式，每种格式都使用一个 *符号*（+ 或 -）、一个 *长度决定其精度的分数* 和一个 *决定其能表示的整体范围的指数*。在 bfloat16 引入之前，AI 和其他一些计算的选择仅限于两种格式：

+   *float32*（单精度），范围约为 +/- 1.2 x 10^-38 到 3.4 x 10³⁸，占用 32 位。

+   *float16*（半精度），范围约为 +/- 6.1 x 10^-5 到 65,504，占用 16 位。

在几乎所有需要不需要双精度（float64）的 AI 和其他应用中，都广泛使用 float32。

bfloat16 添加了一个中间值，在 [与 float32 相同的范围](https://en.wikipedia.org/wiki/Bfloat16_floating-point_format) 内，约为 +/- 1.2 x 10^-38 到 3.4 x 10³⁸，但仅占用一半的空间，精度较低。它设计用于与 float32 进行简单转换，因为其符号和指数保持不变，只需扩展或截断分数（尾数），具体取决于转换的方向。在 float32 和 float16 之间的转换更为复杂，最重要的是，由于 float16 允许的范围远小于 float32，超出其范围的数字将失去其数值。这意味着任何大于 65,504 的浮点数在许多应用程序中都存在严重限制。

一个长度为 float32 数字一半的数字格式不仅在存储大量数据时很重要，而且对操作性能有重要影响。通常使用“单指令，多数据”（SIMD）技术加速这些操作，其中 [一个寄存器被打包](https://eclecticlight.co/2021/08/23/code-in-arm-assembly-lanes-and-loads-in-neon/) 为两个或更多值，然后核心并行执行这些指令。在 M 系列芯片的 CPU 核心中，通常使用 128 位寄存器进行这些操作，可以容纳四个 float32 值或八个 bfloat16 值。对于涉及数千个算术操作的任务，将寄存器的值增加一倍几乎可以使吞吐量翻倍，正如 [在 Arm 处理器上的测试报告](https://community.arm.com/arm-community-blogs/b/ai-and-ml-blog/posts/bfloat16-processing-for-neural-networks-on-armv8_2d00_a) 中所述。

在接受其降低精度的应用程序中，bfloat16 格式因此提供了与 float32 相同的范围、与 float32 简单快速的转换、占用一半存储空间，并在 SIMD 执行中提供最多两倍的性能。

#### 但我的 Mac 没有训练 AI 模型

当谷歌的 AI 研究人员[首次声称](https://cloud.google.com/blog/products/ai-machine-learning/bfloat16-the-secret-to-high-performance-on-cloud-tpus) bfloat16 是“高性能的秘密”时，他们当然是在指 AI 模型的开发者，而不是普通用户。该文章发表于 2019 年 8 月，距苹果宣布 M1 不到一年，解释了为什么 M1 中的硬件都不支持 bfloat16，并且 ARMv8.6-A 直到 ARM 才添加了支持。

Arm 和苹果都认识到，在设备上尽可能多地执行 AI 训练的重要性，而不是在云中执行。这一点由独立的 Arm 的 Hellen Norman [进行了阐述](https://community.arm.com/arm-community-blogs/b/architectures-and-processors-blog/posts/ai-vs-ml-whats-the-difference) ，与苹果无关。

想象一下，我们可以设计一些本地训练任务，以改善 macOS 中常见功能并不需要太多的想象力。我们中的许多人都会禁用拼写检查，因为它似乎无法识别我们应该使用类似的单词，比如 *their, there* 和 *they’re*。如果建议的更正是基于语法、用法和上下文，那不是更好吗？尽管这方面已经开始改善，而且 Sonoma 的自动完成也变得更智能了，但还有很大的改进空间。这在某种程度上至少部分取决于您的 Mac 是否使用了设备内的训练来学习您的写作风格。

在本文开头，我解释了这不是关于当前任务的性能，而是关于我们的应用程序和 macOS 将来将要执行的任务的能力，当中有一些任务目前是由专门的或专用的系统来完成的。

CPU 核心并非 Apple 芯片中支持 AI 的唯一硬件：根据任务的不同，macOS 可能会使用它们的 GPU、苹果的专用神经引擎，或者是其传奇的 AMX。考虑到 M1 中的这些部分是在与 CPU 核心相同的时间尺度上设计和开发的，它们似乎不太可能具有 bfloat16 支持，而苹果在 Sonoma 中仅刚刚[为 Metal Performance Shaders 添加了该数字类型](https://developer.apple.com/documentation/metalperformanceshaders/mpsdatatype/bfloat16?changes=l_8_6)。

#### 我的 Mac 是否支持它？

如果您对您的苹果芯片 Mac 是否在其 CPU 核心中支持 bfloat16 感到怀疑，那么有一种简单的方法可以检查。在终端中，运行以下命令

`sysctl -A > ~/Documents/sysctloutput.text`

其中最后一个路径是一个新的文本文件，用于接收命令的输出。

如果该文件包含以下行

`hw.optional.arm.FEAT_BF16: 1`

如果该数字为 0，则表示不支持。苹果提供了解码大多数`hw.optional.arm`特性的信息[此处](https://developer.apple.com/documentation/kernel/1387446-sysctlbyname/determining_instruction_set_characteristics)。

在硬件中没有 bfloat16 支持并不是世界末日，也不意味着您的 M1 Mac 已经过时。然而，这意味着在未来几年里，随着越来越多和更重的人工智能被推出，其中一些功能在其上运行将明显更慢。不过，也请为英特尔 Mac 想一想，因为无论它们的 CPU 有多快，或者有多少个核心，它们永远不会有任何等效的硬件支持用于人工智能。
