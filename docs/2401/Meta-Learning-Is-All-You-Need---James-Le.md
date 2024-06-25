<!--yml

类别：未分类

日期：2024-05-27 15:03:08

-->

# 元学习就是你所需要的一切 —— 詹姆斯·勒

> 来源：[`jameskle.com/writes/meta-learning-is-all-you-need`](https://jameskle.com/writes/meta-learning-is-all-you-need)

在过去几十年中，由于计算能力的提升、非结构化数据的丰富以及算法解决方案的进步，神经网络在机器学习社区中具有极高的影响力。然而，对于研究人员来说，要在数据稀缺且模型准确度/速度要求严格的真实世界环境中完全使用神经网络仍然还有很长的路要走。

**元学习**，也被称为*学习如何学习*，最近已经成为一个潜在的学习范式，它可以从一个任务中学习信息，并将该信息有效地推广到未见过的任务中。在这段隔离时间里，我开始观看由杰出的[切尔西·芬恩](https://ai.stanford.edu/~cbfinn/)教授的[斯坦福大学 CS 330 深度多任务与元学习课程](http://cs330.stanford.edu/)的讲座。作为对她讲座的一种礼貌，这篇博客尝试回答以下关键问题：

1.  我们为什么需要元学习？

1.  元学习的数学原理是如何工作的？

1.  设计元学习算法的不同方法有哪些？

**注意：** *本文的内容主要基于 CS330 的* [*问题定义讲座 1*](https://www.youtube.com/watch?v=0rZtSwNOTQo&list=PLoROMvodv4rMC6zfYmnD7UG3LVvwaITY5&index=1)*,* [*监督式和黑盒元学习讲座 2*](https://www.youtube.com/watch?v=6stKGH6zI8g&list=PLoROMvodv4rMC6zfYmnD7UG3LVvwaITY5&index=2)*,* [*基于优化的元学习讲座 3*](https://www.youtube.com/watch?v=v7otSgpTc0Q&list=PLoROMvodv4rMC6zfYmnD7UG3LVvwaITY5&index=3)*,* 和* [*通过度量学习进行少样本学习讲座 4*](https://www.youtube.com/watch?v=bc-6tzTyYcM&list=PLoROMvodv4rMC6zfYmnD7UG3LVvwaITY5&index=4)*。它们都是公开可访问的。*

## **1 - 元学习的动机**

由于过去十年算法、数据和计算能力的进步，深度神经网络已经使我们能够非常好地处理非结构化数据（例如图像、文本、音频、视频等），而无需手工设计特征。经验研究表明，如果我们为神经网络提供大量且多样化的输入，它们可以非常好地推广。例如，[变压器](https://arxiv.org/abs/1706.03762)和[GPT-2](https://openai.com/blog/better-language-models/)去年在自然语言处理研究社区掀起了风浪，因为它们在各种任务中的广泛适用性。

然而，在使用神经网络的真实世界设置中存在一个问题，即：

+   **大型数据集不可用**：这个问题在许多领域都很常见，从罕见疾病的分类到罕见语言的翻译都是如此。在这些场景中，对每个任务从头开始学习显然是不可行的。

+   **数据呈长尾分布**：这个问题很容易打破标准的机器学习范式。例如，在自动驾驶汽车设置中，自主车辆可以很好地训练以处理常见情况，但在不常见情况下（如人们横穿马路、动物穿越、交通线不起作用等），它经常遇到困难，而人类可以轻松处理。这可能导致非常糟糕的结果，比如[几年前亚利桑那州优步的事故](https://en.wikipedia.org/wiki/Death_of_Elaine_Herzberg)。

+   **我们希望能够快速学习新任务的一些知识，而不必从头开始训练我们的模型：**人类可以通过利用我们的先前经验轻松做到这一点。例如，如果我了解一点西班牙语，那么学习意大利语就不应该太困难，因为这两种语言在语言学上相似。

在这篇文章中，我想对元学习进行初步概述，这是一种学习框架，可以帮助我们的神经网络在上述设置中变得更加有效。在这种设置中，我们希望我们的网络更加熟练地学习新任务-假设它可以访问以前任务的数据。

从历史上看，有一些论文在这个方向上进行了思考。

+   [1992 年](https://www.researchgate.net/publication/2389122_On_the_Optimization_of_a_Synaptic_Learning_Rule)，[Bengio 等人](https://www.researchgate.net/publication/2389122_On_the_Optimization_of_a_Synaptic_Learning_Rule)研究了一种可以解决新任务的学习规则的可能性。

+   [1997 年](https://link.springer.com/article/10.1023/A:1007379606734)，[Rich Caruana](https://link.springer.com/article/10.1023/A:1007379606734)撰写了一份关于**多任务学习**的调查报告，这是元学习的一种变体。他解释了如何通过在模型之间共享表示来并行学习任务，并提出了一种多任务归纳转移概念，该概念使用反向传播来处理额外的任务。

+   [1998 年](https://papers.nips.cc/paper/1034-is-learning-the-n-th-thing-any-easier-than-learning-the-first.pdf)，[Sebastian Thrun](https://papers.nips.cc/paper/1034-is-learning-the-n-th-thing-any-easier-than-learning-the-first.pdf)探讨了**终身学习**的问题，灵感来自人类利用来自相关学习任务的经验以概括新任务的能力。
