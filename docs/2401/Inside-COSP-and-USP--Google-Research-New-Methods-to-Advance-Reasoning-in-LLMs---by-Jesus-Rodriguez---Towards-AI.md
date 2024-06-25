<!--yml

category: 未分类

日期：2024-05-27 15:08:59

-->

# 在 COSP 和 USP 内部：Google 研究新方法以推进LLM中的推理 | 作者：Jesus Rodriguez | 向AI迈进

> 来源：[https://pub.towardsai.net/inside-cosp-and-usp-google-research-new-methods-to-advance-reasoning-in-llms-07338b323dfd](https://pub.towardsai.net/inside-cosp-and-usp-google-research-new-methods-to-advance-reasoning-in-llms-07338b323dfd)

# 在 COSP 和 USP 内部：Google 研究新方法以推进LLM中的推理

## 通过自适应提示，这两种新方法增强了LLM中的常识推理能力。

使用 DALL-E 3 创建

> 我最近开始了一份以人工智能为重点的教育性通讯，已经拥有超过160,000名订阅者。TheSequence 是一份无BS（即没有炒作，没有新闻等）的面向机器学习的通讯，只需5分钟阅读。我们的目标是让您了解机器学习项目、研究论文和概念的最新情况。请通过下面的链接尝试订阅：

提示生成的演进是基于LLM的应用的关键构建块之一。诸如推理或微调等任务高度依赖于具有强大提示数据集。技术，如少样本设置，已经显著减少了需要大量数据来针对特定任务微调模型的必要性。然而，在涵盖广泛任务数组的情况下，特别是在通用模型涵盖的情况下，制作示例提示仍然存在挑战。即使生成一小部分演示也可能是一项艰巨的任务。这对于总结冗长文章或回答需要专业领域知识的查询（如医学问题回答）等任务尤为真实。

在这种情况下，拥有强大的零-shot性能的模型会挽救局面，消除手动提示生成的需求。然而，值得注意的是，零-shot性能往往不那么强大，因为语言模型在没有具体指导的情况下运行，留下了偶尔出错的余地。

最近，Google 研究引入了两种推进LLM中零-shot自适应提示的技术。第一种方法被称为“基于一致性的自适应提示（COSP）”，在一篇[最近的ACL 2023研究论文](https://aclanthology.org/2023.findings-acl.216/)中概述。COSP通过利用未标记样本和模型自身的预测来解决生成合适提示的困境，从而在保留零-shot提示的优势的同时弥合了零-shot和少-shot之间的性能差距。

在一个平行发展的过程中，“[通用自适应提示（USP）](https://arxiv.org/abs/2305.14926)”，正如即将发表的 EMNLP 2023 论文所述，将这一概念扩展到了各种自然语言理解和生成任务中，展示了其在各个领域的有效性。

# COSP 和 USP 详解

COSP和USP背后的核心思想是利用模型的零-shot输出作为提示自身的演示。挑战在于选择可靠的自动生成演示，因为错误的演示可能具有破坏性。为了解决这一挑战，COSP利用了这样一个观察结果，即自信和一致的模型预测更有可能是正确的。这种置信度测量仅基于模型的预测，不需要标记数据。高置信度预测及其相应的输入被视为伪演示。

在此基础上，通过自一致性评估来估计模型对其输出的信心，作为正确性的一个标尺。为了生成一系列可能的原因和答案，使用零-shot思维链提示多次查询模型，其随机性由“温度”超参数控制。然后计算答案的熵以量化不确定性。具有高自一致性和更大模型确定性的答案被认为是可靠的并被选中。

总而言之，COSP和USP遵循类似的方法论：

· 输入未标记的问题到模型中，以获取多个原因和答案。

· 突出显示最常见的答案，并测量其在多个模型输出中的一致性。

· 惩罚重复，并在所选演示中促进多样性。

· 将伪演示串联成测试问题，并再次查询模型以获取最终预测答案。

图片来源：Google研究

虽然COSP主要关注具有明确正确答案的问答任务，但USP将这种方法推广到其他NLP任务，包括分类、短格式生成和长格式生成，并相应地调整置信度测量技术。在USP下，Google研究将其方法论扩展到更广泛的自然语言处理任务领域：

**· 分类（CLS）：** 在这个类别中，问题涉及根据神经网络输出logits确定每个类别的概率。Google研究采用这种方法来衡量不确定性，而无需通过计算logit分布的熵进行多次采样。

**· 短格式生成（SFG）：** 问题类似于问答，如果需要，可以使用与COSP中使用的类似过程，但不包括生成理由的步骤。

**· 长格式生成（LFG）：** 诸如摘要和翻译之类的任务通常涉及开放性问题，即使在模型确信时也会产生非相同的输出。在这些情况下，Google研究采用重叠度量，计算相同查询的不同输出之间的平均成对ROUGE分数。

图片来源：Google研究

这些创新方法代表了AI提示领域的重大进步，使模型能够有效地自行提示并增强其在各种自然语言任务中的性能。

# 结果

谷歌研究评估了COSP和USP在不同基准测试中的表现。就一致性自适应提示（COSP）而言，谷歌研究最初集中于一组六个算术和常识推理问题。他们将COSP与0-shot-CoT方法进行了基准测试，使用自一致性跨所有基线来确保公平的计算资源比较。在三种不同的大型语言模型（LLM）上，结果无疑显示，零-shot COSP优于标准的零-shot基线。

图片来源：谷歌研究

通过通用自适应提示（USP），谷歌研究采取了更为广泛的方法，将分析范围扩大到涵盖25多个分类任务、短文生成和长文生成任务。此外，他们使用最先进的PaLM 2模型来处理庞大的BIG-Bench Hard任务套件，这是LLM在人类表现方面以前曾经困难的领域。与他们的COSP发现惊人一致，谷歌研究表明，USP始终优于基准方法，并在与使用黄金示例提示时保持竞争力。

图片来源：谷歌研究

通过对置信度和正确性之间关系的调查，谷歌研究明显致力于理解USP的机制。他们的发现证实了一个关键观察结果，即USP主要选择自信的预测，这些预测往往在考虑的所有类型的任务中产生优越的结果，如附图所示。这加强了USP在增强语言模型在各种自然语言理解和生成任务中的性能方面的效力。

图片来源：谷歌研究

COSP和USP都代表着探索提示生成的重要领域，以增强LLM的常识推理能力。