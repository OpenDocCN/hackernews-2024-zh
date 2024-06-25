<!--yml

类别：未分类

日期：2024 年 05 月 27 日 15:19:48

-->

# 🦅 Eagle 7B：跨越 100 多种语言使用 1 万亿标记的变压器（RWKV-v5）。

> 来源：[`blog.rwkv.com/p/eagle-7b-soaring-past-transformers`](https://blog.rwkv.com/p/eagle-7b-soaring-past-transformers)

一只鹰，飞过一个看起来像变压器的机器人。

Eagle 7B 是一个 7.52B 参数模型，它：

我们发布了 RWKV-v5 Eagle 7B，[在 Linux 基金会下以 Apache 2.0 许可证授权](https://blog.rwkv.com/p/rwkv-joins-the-linux-foundation-as)，可以个人或商业用途无限制地使用

我们对以下基准测试进行了多语言性能测试：[xLAMBDA](https://github.com/EleutherAI/lm-evaluation-harness?tab=readme-ov-file#advanced-usage-tips)，[xStoryCloze](https://huggingface.co/datasets/Muennighoff/xstory_cloze)，[xWinograd](https://huggingface.co/datasets/Muennighoff/xwinograd)，[xCopa](https://huggingface.co/datasets/xcopa)

在总共 23 种语言中

这些基准大部分涵盖了常识推理，使用各自的语言。并且显示了 RWKV v4 到 v5 架构的多语言性能的巨大整体跃升。以及 v2 世界数据集。

还应该注意到，由于上述涵盖了大约前 23 种语言，所以缺乏多语言基准测试。

这使得直接评估剩余的 75 多种语言的模型语言性能变得困难，超过了总计 100 多种受过培训的语言。这是我们希望在未来的模型中改进的一个缺点。

英语表现是在 12 个独立的基准测试中进行的，涵盖常识推理和世界知识。

我们再次看到从 RWKV v4 到 v5 架构的巨大整体跃升。以及 v2 世界数据集。

在 v4 之前输给了 MPT-7b，即 1T token 层中的顶级模型。

v5 开始在基准测试中争夺，某些情况下甚至在某些基准测试中（LAMBADA，StoryCloze16，WinoGrande，HeadQA_en，Sciq）超过 Falcon，甚至羊驼 2。

另外，v5 的性能开始与预期的变压器性能水平保持一致，考虑到其给定的近似标记训练计数。

随着狂风-7B 保持其领先地位，据说其训练了 2 到 7 万亿个标记。

我们期望随着我们额外训练 1T token，缩小差距，越过羊驼 2 线，并希望达到狂风线。

或者，作为一个基础模型，它轻微调整了（真的很小的指令集混合），我们渴望看到各种社区和指令调整的变体如何。

* * *

一个显着的观察是，我们的检查点在接近 3000 亿标记点附近，显示出与 pythia-6.9b 相似的性能。

这与我们以前在 RWKV-v4 架构上进行的堆实验一致，即线性变压器像 RWKV 一样按照相同的标记计数进行性能级别的类似规模。

如果是这样，它是否重复了问题。如果确切的架构，对于模型评估性能，数据是否比较重要？

RWKV 基于架构与变压器模型的 CUDA 计算成本——这真是一个二次 vs 线性的比较！

如果是真的，也许我们应该寻求更高效、可扩展的架构，以提高可访问性，降低人人都能承受得起的 AI 成本，并[减少对环境的影响。](https://blog.rwkv.com/p/the-worlds-greenest-ai-model-rwkvs)

* * *

我们接收到的 RWKV 多语言方法的一条常见反馈是

对于大多数部分，我们都同意这两点。

但我们没有计划改变这一点，因为我们正在为全世界构建 AI——这不仅仅是一个英语世界。

[2023 年，全球只有 17% 的人口讲英语](https://preply.com/en/blog/english-language-statistics/#:~:text=Current%20research%20suggests%20that%20the,widely%20spoken%20language%20in%202022%3F)

（13 亿人口）

世界地图显示流利讲英语的地区和人口分布（来源：[维基百科](https://en.wikipedia.org/wiki/List_of_countries_by_English-speaking_population)）

但是，通过确保支持世界前 25 大语言及以上，我们可以覆盖大约[40 亿人口，约占全球人口的 50%](https://en.wikipedia.org/wiki/List_of_languages_by_number_of_native_speakers#Top_languages_by_population)

有缺陷的地图，突出显示鹰语言模型将完全或部分支持的区域——目标是能够充满信心地将整个地图涂成绿色

这与团队的共同目标非常一致，即让 AI 支持每个人，不仅仅是通过让其在低端硬件上便宜且可负担，而是通过支持他们的语言。

随着时间的推移，我们打算扩大多语言数据集，支持更广泛的语言种类，并逐步将覆盖范围扩大到全球 100%——以确保没有任何一种语言被落下。

我们社区中的一个主要例子是[印尼 NLP Discord 群](https://discord.gg/dy9YWXjV)，该群会从 RWKV 系列基础模型中微调印尼语言模型。

使他们能够在廉价可负担的基础上（即单节点）构建强大的语言特定模型，而不需要进行五十万美元的预训练。

* * *

此版本标志着迄今为止最强的线性变压器（在评估基准方面）的发布。

虽然它可能没有成功通过 LLaMA2 和 Mistral。它提供了以下强有力的证据。

+   RWKV-v5 模型架构的规模与变压器性能类似，具有相似的令牌数

+   您可以以极低的推理成本实现接近 LLaMA2 的性能水平

+   同时支持多语言水平的性能

我们计划继续推进

+   [2024 年 2 月] 更新的 RWKV v5：鹰论文，我们将深入探讨自 v4 以来的架构变化，以及模型基准和评估

+   [Feb 2024] 另外 1T token 在训练中（总共 2T），以与 LLaMA2 7B 模型进行直接比较

+   [Mar 2024] 基于 v5 鹰 2T 模型的 An MoE 模型

+   [Mar 2024] RWKV-v6：“雀鸟” 1.5B，3B 世界模型

> 免责声明：所有日期均为近似值，并且受到我们赞助商/供应商计算资源的严重影响

了解更多有关 RWKV 项目的信息，请访问

* * *

我们非常感激并要感谢以下关键团体：

与各种开发人员一起，致力于发展 [RWKV 相关项目](https://wiki.rwkv.com) 的人员。
