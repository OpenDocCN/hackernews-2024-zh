<!--yml

category: 未分类

date: 2024-05-27 15:04:12

-->

# Nvidia发布了Blackwell B200 GPU，这是用于AI的“世界上最强大的芯片” - The Verge

> 来源：[https://www.theverge.com/2024/3/18/24105157/nvidia-blackwell-gpu-b200-ai](https://www.theverge.com/2024/3/18/24105157/nvidia-blackwell-gpu-b200-ai)

Nvidia的必备H100 AI芯片使其成为[一家价值数万亿美元的公司](/2024/2/23/24080975/nvidia-ai-chips-h100-h200-market-capitalization)，可能超过了Alphabet和Amazon，并且竞争对手一直在[努力追赶](/2024/2/1/24058186/ai-chips-meta-microsoft-google-nvidia)。但也许Nvidia即将通过新的Blackwell B200 GPU和GB200“超级芯片”扩大其领先优势。

*Nvidia CEO黄仁勋在GTC直播中展示他的新GPU（左）和右侧的H100。*

图片：Nvidia

Nvidia表示，新的B200 GPU从其2080亿个晶体管中提供高达20 *petaflops*的FP4运算能力。此外，它表示，一个结合了两个GPU和单个Grace CPU的GB200，可以在LLM推理工作负载中提供30倍的性能，同时可能更加高效。Nvidia称其“可以将成本和能源消耗降低多达25倍”比H100，尽管成本尚存在疑问 — [Nvidia的CEO暗示](/2024/3/19/24106141/nvidia-ceo-says-his-blackwell-b200-ai-gpu-will-cost-30k-40k-a-pop) 每个GPU可能在30,000到40,000美元之间。

Nvidia声称，以前，训练一个1.8万亿参数模型需要8,000个Hopper GPU和15兆瓦的电力。如今，Nvidia的CEO表示，2,000个Blackwell GPU只需4兆瓦即可完成。

在一个具有1750亿个参数的GPT-3 LLM基准测试中，Nvidia表示GB200的性能比H100略高七倍，并且Nvidia表示它提供四倍的训练速度。

*这就是一台GB200的外观。两个GPU，一个CPU，一个主板。*

图片：Nvidia

Nvidia告诉记者，其中一个关键改进是第二代变压器引擎，通过每个神经元使用四位而不是八位（因此，我之前提到的20 petaflops的FP4）。第二个关键区别仅在连接大量这些GPU时出现：一款下一代NVLink开关，可让576个GPU相互通信，具有每秒1.8TB的双向带宽。

这要求Nvidia建造了一整套全新的网络交换芯片，拥有50亿晶体管和一些自身的板载计算能力：Nvidia称其为3.6 teraflops的FP8。

*Nvidia表示，它正在Blackwell中添加FP4和FP6。*

图片：Nvidia

Nvidia表示，以前，仅16个GPU的集群会花费60％的时间进行通信，实际计算仅占40％。

当然，Nvidia 指望公司购买大量这些 GPU，并将它们打包成更大的设计，例如 GB200 NVL72，该设计将 36 个 CPU 和 72 个 GPU 集成到单个液冷机架中，总共提供 720 petaflops 的 AI 训练性能或者 1,440 petaflops（即 1.4 exaflops）的推断性能。内部有将近两英里长的电缆，有 5,000 条单独的电缆。

GB200 NVL72。

图片：Nvidia

机架中的每个托盘都含有两个 GB200 芯片或两个 NVLink 开关，每个机架分别有 18 个前者和 9 个后者。总体而言，Nvidia 表示这样的一个机架可以支持一个 27 万亿参数模型。传闻中，GPT-4 的参数约为 1.7 万亿。

公司表示，亚马逊、谷歌、微软和甲骨文都已经计划在其云服务中提供 NVL72 机架，尽管目前尚不清楚他们打算购买多少。

当然，Nvidia 也乐意为公司提供其余的解决方案。这里是 DGX GB200 的 DGX Superpod，将八个系统合并为一个，总共包括 288 个 CPU、576 个 GPU、240TB 内存和 11.5 exaflops 的 FP4 计算能力。

Nvidia 表示其系统可以扩展到数万个 GB200 超芯片，使用新的 Quantum-X800 InfiniBand（最多可达 144 个连接）或 Spectrum-X800 以太网（最多可达 64 个连接）进行 800Gbps 网络连接。

今天我们不指望能听到关于新游戏 GPU 的任何消息，因为这则消息来自 Nvidia 的 GPU 技术大会，通常几乎完全专注于 GPU 计算和人工智能，而不是游戏。但是 Blackwell GPU 架构很可能也会驱动未来的 RTX 50 系列桌面图形卡。[（来源）](https://videocardz.com/newz/nvidia-blackwell-gb203-gpu-to-feature-256-bit-bus-gb205-with-192-bit-claims-leaker)

**更新，3 月 19 日：** Nvidia CEO 估计新 GPU 的成本可能高达每个 40K 美元。
