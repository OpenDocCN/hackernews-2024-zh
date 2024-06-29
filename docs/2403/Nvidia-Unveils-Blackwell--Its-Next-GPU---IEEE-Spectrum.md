<!--yml

category: 未分类

日期：2024-05-29 12:31:34

-->

# Nvidia揭示布莱克维尔（Blackwell），其下一代GPU - IEEE Spectrum

> 来源：[https://spectrum.ieee.org/nvidia-blackwell](https://spectrum.ieee.org/nvidia-blackwell)

今天在Nvidia的开发者大会上，[GTC 2024](https://www.nvidia.com/gtc/)，公司揭示了其下一代GPU，[B200](https://nvidianews.nvidia.com/news/nvidia-blackwell-platform-arrives-to-power-a-new-era-of-computing)。B200能够提供四倍的训练性能，高达30倍的推理性能，并且比其前身霍珀H100 GPU的能效提高了25倍。基于新的布莱克维尔架构，该GPU可以与公司的Grace CPU结合，形成新一代DGX SuperPOD计算机，能够以新的低精度数字格式进行高达11.5亿亿浮点运算（exaflops）的AI计算。

“布莱克维尔是一种新型的AI超级芯片，” Nvidia高性能计算和超大规模副总裁[Ian Buck](https://www.linkedin.com/in/ian-buck-19201315/)说。Nvidia将GPU架构命名为数学家[戴维·哈罗德·布莱克维尔（David Harold Blackwell）](https://en.wikipedia.org/wiki/David_Blackwell)，他是美国国家科学院首位黑人成员。

B200由大约1600平方毫米的处理器组成，分布在两个硅片上，通过每秒10 TB的连接在同一封装中连接，因此它们表现得像单个2080亿晶体管芯片。这些硅片使用[TSMC的N4P芯片技术](https://pr.tsmc.com/english/news/2874)制造，与用于制造霍珀架构GPU（如H100）的N4技术相比，性能提升了6%。

与霍珀芯片一样，B200被高带宽内存环绕，这对于减少大型AI模型的延迟和能耗至关重要。B200的内存是最新型号，HBM3e，总容量为192 GB（相比第二代霍珀芯片H200的141 GB有所增加）。此外，内存带宽从[H200](https://www.nvidia.com/en-us/data-center/h200/)的4.8 TB/s提升至8 TB/s。

## 较小的数字，更快的芯片

芯片制造技术在制造布莱克维尔（Blackwell）时发挥了一定作用，但GPU对晶体管的使用才是真正的区别所在。在去年的IEEE热门芯片大会上，Nvidia首席科学家比尔·达利（Bill Dally）向计算机科学家解释了[Nvidia的人工智能成功](https://spectrum.ieee.org/nvidia-gpu)，他表示，大部分成功来自于在AI计算中使用越来越少的位数来表示数字。布莱克维尔（Blackwell）继续这一趋势。

它的前身架构 Hopper 是 Nvidia 所称的[变压器引擎](https://spectrum.ieee.org/nvidias-next-gpu-shows-that-transformers-are-transforming-ai)的第一个实例。这是一个系统，它检查神经网络的每一层，并确定是否可以使用更低精度的数字进行计算。具体来说，Hopper 可以使用最小为 8 位的浮点数格式。较小的数字计算速度更快、更节能，需要更少的内存和内存带宽，并且进行这些数学运算所需的逻辑占用的硅空间更少。

“通过 Blackwell，我们迈出了一大步”，Buck 表示。这种新架构具有可以使用仅 4 位浮点数进行矩阵运算的单元。更重要的是，它可以决定将它们部署在每个神经网络层的部分上，而不仅仅是整个层，像 Hopper 那样。“能够达到如此精细的粒度本身就是一个奇迹”，Buck 表示。

## NVLink 和其他功能

Nvidia 关于 Blackwell 的其他架构洞见包括它包含一个专用的“引擎”，致力于 GPU 的可靠性、可用性和可维护性。据 Nvidia 称，它使用基于 AI 的系统进行诊断和预测可靠性问题，旨在提高运行时间，帮助大型 AI 系统连续运行数周，这通常是训练大语言模型所需的时间。

Nvidia 还包括用于帮助保护 AI 模型安全以及解压数据以加快数据库查询和数据分析速度的系统。

最后，Blackwell 集成了 Nvidia 第五代计算机互联技术 NVLink，现在在 GPU 之间双向提供 1.8TB/s 的带宽，并允许最多 576 个 GPU 之间进行高速通信。Hopper 版本的 NVLink 只能达到该带宽的一半。

## SuperPOD 和其他计算机

NVLink 的带宽对[从 Blackwell 构建大规模计算机](https://nvidianews.nvidia.com/news/nvidia-blackwell-dgx-generative-ai-supercomputing)至关重要，能够处理万亿参数的神经网络模型。

基本计算单元被称为 DGX GB200。每个单元包括 36 个 GB200 超级芯片。这些模块包含一个 Grace CPU 和两个 Blackwell GPU，全部通过 NVLink 连接在一起。

Grace Blackwell 超级芯片是同一模块中的两个 Blackwell GPU 和一个 Grace CPU。Nvidia

另外，八个 DGX GB200 可以通过 NVLINK 进一步连接，形成一个称为 DGX SuperPOD 的 576-GPU 超级计算机。Nvidia 表示，这样的计算机可以使用 4 位精度计算达到 11.5 exaflops 的计算能力。使用公司的[Quantum Infiniband](https://www.nvidia.com/en-us/networking/quantum2/)网络技术，也可以构建数万个 GPU 的系统。

公司表示，预计今年晚些时候将推出SuperPOD和其他英伟达计算机。与此同时，芯片代工厂TSMC和电子设计自动化公司Synopsys分别宣布，它们将把英伟达的反向光刻工具cuLitho [投入生产](https://spectrum.ieee.org/inverse-lithography)。最后，英伟达宣布了一个[面向人形机器人的新基础模型](https://spectrum.ieee.org/nvidia-gr00t-ros)，名为GR00T。
