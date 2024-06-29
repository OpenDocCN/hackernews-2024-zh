<!--yml

类别：未分类

日期：2024年05月27日 15:00:14

-->

# 🦅 EagleX v1：在英语和多语言评估中飞越 LLaMA 7B 2T（RWKV-v5）

> 来源：[https://substack.recursal.ai/p/eaglex-17t-soaring-past-llama-7b](https://substack.recursal.ai/p/eaglex-17t-soaring-past-llama-7b)

一只鹰，飞越了一只羊驼

> 如果你在进行微调，我们建议等待本月晚些时候发布的全新 EagleX 2T 模型，除非你是出于研究目的这么做。
> 
> 这个模型是出于研究目的发布的，因为它代表了超越 LLaMA2 7B 的主要检查点，作为我们目前训练 2T 令牌及更多的一部分。

EagleX 1.7T 是我们正在训练的7.52B参数模型的早期研究发布:

我们发布了 RWKV-v5 EagleX 1.7T，[在Apache 2.0下许可](https://blog.rwkv.com/p/rwkv-joins-the-linux-foundation-as)，可以在个人或商业用途中无限制地使用。

它绝对是一个非常大胆的说法，说你已经从头开始赶上并超过了“7B”重量级的“黄金标准”，几乎每一个其他主要的开放访问模型都建立在其上（据说甚至包括 Mistral）。尤其是考虑到这是用相对较低的数据集令牌数1.7万亿（而不是2万亿令牌）完成的。

由于这是一个完全不同的模型，从头训练，所以我们会有赢和输的评估结果，我们对此是完全透明的，展示我们在平均值上如何领先于 LLaMA 7B。

与其简单地精选14个不同的我们赢得的评估，并打上一个胜利的标签，我们在 EleutherAI `[lm-eval-harness](https://github.com/EleutherAI/lm-evaluation-harness)` 中运行了我们能够做的所有基准测试，具有以下限制：

+   它必须在8x4090上在30分钟内完成（我们运行了大量的评估）

+   我们排除了所有人格/取向评估

+   评估必须能在各种不同的模型上运行，通过 lm-eval-harness

+   所有的评估都是零射击（不会5次射击一道选择题）

+   我们限制了与7B重量级内的其他模型的比较

这些导致运行60多个主要评估组，每个模型产生1000多个数据点。数据点数量如此之高，以至于我们不得不放弃标准误差偏差，只是为了确保原始 CSV 文件可以在 MacOS numbers 中加载。

把184个英语评估数据点适合放在屏幕上需要做些什么。

哇，这是一个疯狂的数据点数量要消化。让我把它分解成更易消化的部分：

所有这里展示的数据可通过此 Google 表格获得：

> 我们解释了一些评估的含义，这些你将来看到的评估结果可以牢记在心中（解开这些数字的含义！）

* * *

我们从基础开始: 困惑度。这是对测试数据集的损失值（分数越低=越好），即模型对下一个标记预测的好坏。

总的来说，随着困惑度改进，EagleX 模型在英文和多语言评估中表现优于 LLaMA2-7b，在 Falcom/LLaMA2-7b 和 Mistral 之间排名。

**为什么专家关心困惑度？**

评估总体上可能非常主观和受意见驱动，并且通常会产生不一致的结果。困惑度在某种程度上为大多数专家提供了最简短的摘要开始阅读。

EagleX保持了最佳多语言性能的领先地位，随着我们对Eagle系列模型进行的增量改进。

大多数任务都是常识推理测试，格式多样，涵盖了[包括全球23种最常用语言在内的多种语言。](https://blog.rwkv.com/i/141130059/multi-lingual-performance-details)

对于剩余的语言，我们敦促社区自行测试和评判，已经训练了超过100种语言。随着时间的推移，我们希望能够增加更多语言进行评估。

**多语言性能为何重要？**

RWKV项目及Eagle系列模型的目标是为每个人建立**包容性**AI，不论其语言如何。我们的使命是不仅为英语而建立AI模型，还为使用非英语语言的全球83%人口建立AI模型。

尽管如此，英语仍然很重要。我们将评估减少到了21个可以说是最流行的英语评估之一，如Lambada、Glue、Swag、Winogrande、TruthfulQA、MMLU：

将其缩小到我们大多数人实际关心的4个模型 - LLaMA、Mistral、EagleX和Eagle-7b - 新的EagleX模型在所有21次评估中的平均表现超过了LLaMA-2-7b，并且与Mistral相差无几。

请记住，所显示的平均值是针对所有21次评估的。

* * *

现在，让我们看看我们的模型在哪些方面超越了其他模型。

首先，最引人注目的是前6次评估，即使是我们小型的1.7T训练模型也能击败甚至是Mistral 2T++训练模型（sciq、glue、anli、mmnli、swag），涵盖了围绕基于上下文的简单常识推理或演绎逻辑的多个任务。EagleX在Wingrade和WNLI评估中也比LLaMA-2-7b表现更好，这也涉及到基于上下文的常识推理。这表明EagleX模型在主要是上下文问答（RAG）用例中是适用的，只要有正确的提示工程。

最后，在TruthfulQA方面，虽然它的表现优于LLaMA，但在我看来，这仍然表明所有模型都容易从网络中学习到常见的人类误解，看到所有模型得分如此糟糕。

（公平地说，这对大多数人类来说也很困难）

> PS：Glue/mnli的飞跃足够大，我们需要检查数据集是否存在污染。然而，我们未能找到任何污染。这一跳跃目前被归因于多个训练数据集，以及数据增强/机器重写的指导数据集，遵循了类似的结构。

在上下文中，强大的常识推理。

对多个RAG用例具有非常强的适用性

接下来：混合结果的评估集。在这里，我们有两个主要变体非常相似的评估。EagleX和LLaMA之间的结果非常接近，很难说哪个模型在这些评估中明显更好。

有趣的是，尽管logiqa可以被视为一种“常识”推理测试，但EagleX模型在6个评估（sciq、glue、anli、mmnli、swag）中的得分比其他具有更多令牌训练的模型要低得多。这可能意味着虽然在给定上下文的推理方面模型更好，但它在知识深度上不如其他模型。

* * *

这些是EagleX在与Mistral和LLaMA比较时表现最差的评估。然而，对于输给LLaMA的评估，差距并不大。但随着我们训练超过2T令牌，我们将继续跟踪这些情况。

让我们看看发生了什么糟糕的事情：数学。

算术评估的结果急剧下降，甚至比我们最初的Eagle模型还要糟糕。

发生了什么问题？

我们深入研究了用于训练的数据集，并意识到由于一个错误，我们错过了整个数学数据集（以及其他几个）。哎呀。

这强调了在训练过程中维护数据集组成的重要性。我们正在为未来的运行添加数学内容。

> 随着训练的继续，我们预计总体数学分数会重新上升，但实际上，我认为——没有人应该依赖于7B模型来进行数学（只是说说而已）。

* * *

我们并不想简单地挑选9或21个评估结果，然后宣称在LLaMA或者Mistral上获得胜利。因此，让我们放大视角，全面看待183个英文评估结果。

[您可以在此查看完整的结果](https://docs.google.com/spreadsheets/d/1PFELH3u8yQlr-bGs9D5lBYXCXqSFZw2O0vfW084jbgI/edit?usp=sharing)

尽管使用所有评估的整体平均值确实会对较大的评估集结果产生偏见（由于重复计算，例如总体和许多单独的mmlu测试），但这并不会改变EagleX、Mistral、LLaMA和原始Eagle模型之间的排名。

然而，这些结果对于更小的洞察力非常有用，例如

EagleX模型在美国历史方面输给了LLaMA2，但在世界历史中胜出。考虑到我们采取的更广泛的方法来构建数据集，这是有道理的，这种方法比起以美国为中心的方法更加包容和全球化。

详细的见解将由我们的数据集团队用于迭代和改进我们未来的数据集。

模型的回答方式，反映了它学习过的数据集经验。

模型消耗多少资源，反映了它的架构。

我们所做的最大变化之一是为当前的1T令牌更改数据集，现在使用了一个更清洁的、经过筛选的数据集，**在确保使用可许可的内容源时非常慎重考虑**。

事实上，模型比计划时间更早地跨越了llama2线，这表明该架构在训练中更加高效，或者数据集质量的改进对模型性能有重大影响。

以下是使用的数据集的摘要，其公开发布将在当前2T训练完成后的下个月进行。

```
## 15% Code 

Contains code/programming related topics
- the-stack
- codeparrot
- devopedia
- mdn

## 15% Multi lang

Generally multi-lang webtext
- sea-lion (Singapore)
- madlad
- culturax
- multi lang wiki

## The giant soup

Creative content
- fandom (only sites with permissive licenses, and low spam)
- scp-foundation

Wikipedia
- Various Permissively licensed wikis.
- wikipedia

Papers:
- Mainly arxiv (Permissive Licenses) and pes2o

Books:
All the books contained in out train sets are public domains books.
- gutenberg, 
- standardebooks

Webtext
- webtext
- refinedweb (Note: This chunk made the model worse, we recommend against refinedweb in future trains)
- slimpajama
- europarl
- eurlex.
- stackexchange

Various
- aya (multilang convo)
- some system prompt, instruct
- long list of sub 100B training datasets on HF
- rewritten text !!! (splicing in, to replicate the rewritten web paper)
```

* * *

我们的可扩展性比变压器架构高出100倍以上。

变压器之所以成为AI中最突出的架构，不是因为它是最好的，而是因为它是首个成功扩展到数十亿参数训练的架构。

至今

RWKV架构与变压器模型的CUDA计算成本-二次与线性的确实存在巨大的差距！

* * *

总体而言，这款模型的发布标志着我们许多人在Recursal AI的商业团队以及RWKV团队内部开源团队中的重要里程碑和转变。

+   作为我们的主要计算提供者，Recursal AI团队与AWS合作进行了第一个重大训练

+   该模型采用Apache 2许可证发布

+   完全训练的2T模型将在Linux Foundation下的RWKV集团发布

+   首个非变压器架构通过LLaMA2评估

+   至今最强大的线性变压器

+   证明你可以在多语言和英语性能上都非常强大

* * *

与原始Eagle 7B的公告类似，以下是模型训练的修订目标

+   [2024年4月] 完成了2T Eagle 7B模型

+   [2024年3月至5月] 我们的v6“Finch”系列模型训练

+   [2024年6月] v6 MoE模型，为GPT 3.5级性能

> 免责声明：所有日期均为近似值，并且严重依赖于我们赞助商/计算提供者/投资者的计算可用性

如果你想了解更多关于RWKV开源项目的信息，请访问

如果你今天想尝试这款模型，可以在我们的平台上[recursal.ai](https://recursal.ai)进行，这是托管、运行和创建Eagle系列RWKV模型的最佳地点。
