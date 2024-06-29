<!--yml

类别：未分类

日期：2024-05-27 14:46:30

-->

# 韩国研究人员以新的神经AI芯片对Nvidia进行了功耗比较 —— 声称功耗减少了625倍，体积减少了41倍 | Tom's Hardware

> 来源：[https://www.tomshardware.com/tech-industry/artificial-intelligence/korean-researchers-power-shame-nvidia-with-new-neural-ai-chip-claim-625-times-less-power-41-times-smaller](https://www.tomshardware.com/tech-industry/artificial-intelligence/korean-researchers-power-shame-nvidia-with-new-neural-ai-chip-claim-625-times-less-power-41-times-smaller)

来自韩国科学技术院(KAIST)的科学家团队在最近的2024年国际固态电路会议(ISSCC)上详细介绍了他们的“Complementary-Transformer”人工智能芯片。新的C-Transformer芯片被称为世界上第一款能够进行大型语言模型(LLM)处理的超低功耗AI加速器芯片。

在一篇[新闻稿](http://koreabizwire.com/kaist-develops-next-generation-ultra-low-power-llm-accelerator/274663)中，研究人员对Nvidia进行了功耗比较，声称C-Transformer的功耗是绿队A100张量核心GPU的625倍少，体积也小了41倍。它还揭示，三星制造的芯片主要来源于精细的[神经形态计算](https://www.tomshardware.com/news/intel-loihi-chip-neuromorphic-computing,35537.html)技术。

(图片来源：KAIST)

尽管我们被告知，KAIST的C-Transformer芯片可以执行与Nvidia的[A100 GPUs](https://www.tomshardware.com/news/nvidia-ampere-A100-gpu-7nm)相同的LLM处理任务，但我们提供的所有新闻和会议材料都没有提供任何直接的性能比较数据。这是一个显著的统计数字，由于其缺失而显眼，愤世嫉俗的人可能会推测，性能比较对C-Transformer不利。

上述画廊有一个“芯片照片”和处理器规格的摘要。您可以看到，C-Transformer目前采用三星的28纳米工艺制造，芯片面积为20.25平方毫米。它以最大频率运行在200 MHz，功耗低于500毫瓦。在最佳情况下，它可以达到3.41 TOPS。从表面上看，这比Nvidia A100 PCIe卡声称的624 TOPS要慢183倍（但KAIST芯片声称功耗减少了625倍）。然而，我们更希望看到某种基准性能比较，而不是看每个平台声称的TOPS。

C-Transformer芯片的架构非常有趣，由三个主要功能特征块组成。首先，有一个具有混合乘积累加单元（HMAU）的同质DNN-Transformer / 脉冲变换器核心（HDSC），以有效处理动态变化的分布能量。其次，我们有一个输出脉冲推测单元（OSSU），用于减少脉冲域处理的延迟和计算量。第三，研究人员实现了一个具有扩展符号压缩（ESC）的隐式权重生成单元（IWGU），以减少外部存储器访问（EMA）能耗。

据KAIST新闻稿解释，C-Transformer芯片并非仅仅将一些现成的神经形态处理作为其“特殊酱料”来压缩LLM的大参数。此前，神经形态计算技术在[LLMs](https://www.tomshardware.com/pc-components/cpus/intels-npu-acceleration-library-goes-open-source)中的使用精度还不够，KAIST称。然而，研究团队表示，“已经成功改进了技术的精度，使其达到了[深度神经网络] DNNs的水平”。

尽管由于与行业标准[AI加速器](https://www.tomshardware.com/news/rtx-4090-blowers-stripped-for-ai)没有直接比较，对这款首个C-Transformer芯片的性能存在不确定性，但很难质疑它将成为移动计算的一个吸引人选项的说法。同样令人鼓舞的是，研究人员已经通过三星测试芯片和广泛的GPT-2测试取得了如此进展。

获取Tom's Hardware最佳新闻和深度评论，直接发送到您的收件箱。
