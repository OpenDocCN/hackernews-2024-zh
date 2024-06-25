<!--yml

类别：未分类

日期：2024-05-27 15:02:33

-->

# 机器学习对软件工程师来说仍然太难了 | Nyckel

> 来源：[https://www.nyckel.com/blog/machine-learning-difficulties/](https://www.nyckel.com/blog/machine-learning-difficulties/)

对于软件工程师来说，开发机器学习功能最困难的事情 *应该* 是找到干净而具有代表性的基准数据，但实际情况往往并非如此。如果你已经有了一个来源于你的应用程序的高质量数据（也许是因为它已经被你的应用程序收集起来），那么你还将面临一些障碍：

## 学习机器学习

ML 的概念很难理解。这个行业仍然主要处于研究阶段，正在开始向使系统和学习资源可被普通开发者消费的方向发展。书籍和类似[FastAI](https://www.fast.ai/)的库就是这一运动的例子，但要学会做最基本的事情仍需要几天或几周的时间。

要做一些基本的事情，比如[图像分类](/blog/image-classification/)，你必须： 

+   理解张量、损失函数、迁移学习、逻辑回归、网络微调、超参数搜索、过拟合、主动学习、正则化和量化等概念。

+   熟悉一个或多个 ML 库，如 PyTorch、Tensorflow、FastAI 或 scikit-learn。这比熟悉普通编程库更难，因为概念和范式与程序员所习惯的非常不同。

+   从研究和工业界寻找最先进的图像深度神经网络。继续寻找新的最先进网络以跟上行业的改进。

+   确保深度网络是在适当的数据语料库上进行预训练的。从头开始训练网络并不是一个好主意，而且预训练使用的数据类型真的很重要。

## 软件

你将需要软件来进行数据探索和管理，做一些事情，比如：

+   可视化关键数据统计，比如类别的相对频率

+   寻找相似/不同的数据样本

+   挖掘稀有类别的实例

+   调试数据标签错误

+   实施“主动学习” - 专注于标记高价值样本，以减少需要标记的数据量

你还需要软件来跟踪各种实验和结果模型的准确性，并在将模型投入生产后监控其性能。

有些公司解决了其中的一部分问题 - 比如[水族馆学习](https://www.aquariumlearning.com/)处理数据管理，[Weights and Biases](https://wandb.ai/site)用于实验跟踪。你需要评估这些公司，并组合一个涉及多个供应商和系统的解决方案。另外，正如你可能从上面的数据管理列表中看到的那样，诸如主动学习和发现标签错误之类的事情需要与正在开发的 ML 模型进行接口。

## 基础设施和 MLOps

一旦你掌握了机器学习和软件部分，你将需要云基础设施方面的专业知识，以便：

+   评估各种硬件选择，以寻找训练和推理的成本性能平衡。

+   设置一个快速、弹性和成本效益的训练流水线。在进行主动学习期间提供交互式和响应式体验时，扩展基础设施的速度是重要的。

+   在您的数据附近设置低延迟、弹性、高可用性和成本效益的推理。 

+   监控云系统和软件的性能、可用性和利用率。

+   必须有一种方法为训练标注成千上万个样本。例如，我们的[回收工具](/pretrained-classifiers/recycling-identifier/)涉及对1万个样本进行标记，标记它们的具体材料，并将其与该材料是否可回收进行交叉引用。

这些关注点通常被称为“MLOps”。有各种公司，比如[Valohai](https://valohai.com/)和[Determined](https://www.determined.ai/)，可以为您解决其中的一些或所有问题。您将需要评估它们，并找出它们如何适应您的整体系统。

## 系统维护

如果你足够幸运能够走到这一步，你将面对一个由多个组件组合在一起的大型系统，你需要操作、维护和调整。维护不仅适用于软件和基础设施，还适用于必须紧跟最新的机器学习技术以获得竞争性性能的机器学习部分。

## 这意味着什么

就这些问题而言，它们构成了开发机器学习功能的一道严峻的入门障碍。是的，如果激励足够诱人，你会克服这个障碍。但我们从一些相当大的公司的机器学习团队那里听说，即使是他们也不会尝试一些项目，因为收益不足以弥补成本。甚至回答“*机器学习能否解决我的问题？*”这个问题都需要你克服上述一半的挑战。想象一下所有被错过的机会和未解决的问题，尤其是在创新型的小型公司和初创企业。

## 未来的形态

软件的故事是关于降低这些障碍，从而引导有创造力的开发者解决以前甚至未曾考虑过的问题。Nyckel诞生于我们尝试使用机器学习解决一个相对简单的问题（用户生成文本的定制策划）时的挫败感，并且我们希望延续这个故事。