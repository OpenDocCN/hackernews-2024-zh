<!--yml

category: 未分类

date: 2024-05-27 14:33:01

-->

# StripedHyena：下一代生成AI的新架构？

> 来源：[https://the-decoder.com/stripedhyena-a-new-architecture-for-next-generation-generative-ai/](https://the-decoder.com/stripedhyena-a-new-architecture-for-next-generation-generative-ai/)

**GPT-4和其他模型依赖transformers。StripedHyena为研究人员提供了一种替代广泛使用的架构。**

Together AI团队通过StripedHyena呈现了一系列拥有70亿参数的语言模型。它的特别之处在于：StripedHyena使用了一组新的AI架构，旨在提高训练和推理性能，与广泛使用的transformer架构相比，例如[GPT-4](https://the-decoder.com/open-ai-gpt-4-announcement/)中使用的架构。

发布包括StripedHyena-Hessian-7B（SH 7B），一个基础模型，和StripedHyena-Nous-7B（SH-N 7B），一个聊天模型。这些模型设计得更快、更节省内存，并能处理长达128,000令牌的非常长的上下文。来自HazyResearch、hessian.AI、Nous Research、MILA、HuggingFace和德国人工智能研究中心（DFKI）的研究人员参与其中。

## StripedHyena：transformers的高效替代品

根据Together AI的说法，StripedHyena是第一个可以与最优秀的开源transformers竞争的替代模型。基础模型在OpenLLM排行榜任务上表现出与[Llama-2](https://the-decoder.com/how-to-get-started-with-metas-llama-2-guide/)、Yi和[Mistral 7B](https://the-decoder.com/new-open-source-llm-mistral-7b-outperforms-larger-meta-llama-models/)可比较的性能，并在长文本摘要任务上表现优于它们。

广告

THE DECODER Newsletter

将最重要的AI新闻直接发送到您的收件箱。

✓ 每周更新

✓ 免费

✓ 随时取消

广告

THE DECODER Newsletter

将最重要的AI新闻直接发送到您的收件箱。

✓ 每周更新

✓ 免费

✓ 随时取消

StripedHyena模型的核心组件是状态空间模型（SSM）层。传统上，[SSMs](https://kevinkotze.github.io/ts-4-state-space/)用于建模复杂的序列和时间序列数据。它们特别适用于需要建模时间依赖性的任务。然而，在过去的两年中，研究人员已经开发出了更好的方法来利用SSMs进行语言和其他领域的序列建模。原因是它们需要较少的计算能力。

结果是：StripedHyena在32,000、64,000和128,000令牌的序列端到端训练中比传统transformers分别快30%，50%和100%以上。

StripedHyena 模型的主要目标是推动架构设计超越变压器的边界。未来，研究人员计划调查更大的模型，具有更长的上下文支持、多模态支持、进一步的性能优化，并将 StripedHyena 集成到检索管道中，以充分利用更长的上下文。
