<!--yml

category: 未分类

date: 2024-05-27 15:03:41

-->

# Gemma：Google推出全新的最先进开放模型

> 来源：[https://blog.google/technology/developers/gemma-open-models/](https://blog.google/technology/developers/gemma-open-models/)

## 设计上负责

Gemma设计时我们将[AI原则](https://ai.google.dev/responsible?utm_source=agd&utm_medium=referral&utm_campaign=explore-responsible&utm_content)置于前沿。作为确保Gemma预训练模型安全可靠的一部分，我们使用自动化技术从训练集中过滤掉某些个人信息和其他敏感数据。此外，我们利用来自人类反馈的广泛微调和强化学习（RLHF）来使我们的指导调整模型与负责任的行为保持一致。为了了解和降低Gemma模型的风险概要，我们进行了包括手动红队测试、自动对抗性测试以及评估模型在危险活动中的能力的健全评估。这些评估在我们的[模型卡片](https://www.kaggle.com/models/google/gemma)中有详细描述。

我们还发布了一个新的[负责任生成AI工具包](https://ai.google.dev/responsible?utm_source=agd&utm_medium=referral&utm_campaign=explore-responsible&utm_content)，与Gemma一起帮助开发者和研究人员优先构建安全和负责任的AI应用。该工具包包括：

+   **安全分类：** 我们提供一种[新方法论](https://codelabs.developers.google.com/codelabs/responsible-ai/agile-classifiers)，用于仅需少量示例构建强大的安全分类器。

+   **调试：** 一个[调试工具](https://codelabs.developers.google.com/codelabs/responsible-ai/lit-gemma)可帮助您调查Gemma的行为并解决潜在问题。

+   -   **指导：** 您可以访问基于Google在开发和部署大型语言模型方面的经验的模型构建者最佳实践。

## 在各种框架、工具和硬件上优化

您可以根据自己的数据对Gemma模型进行微调，以适应特定的应用需求，如摘要或检索增强生成（RAG）。Gemma支持各种工具和系统：

+   -   **多框架工具：** 携带您喜爱的框架，提供多框架Keras 3.0、原生PyTorch、JAX和Hugging Face Transformers的推断和微调参考实现。

+   -   **跨设备兼容性：** Gemma模型适用于包括笔记本电脑、台式机、物联网、移动设备和云在内的流行设备类型，实现广泛可访问的AI功能。

+   **尖端硬件平台：** 我们与NVIDIA合作，优化Gemma以适应从数据中心到云端到本地RTX AI PC的NVIDIA GPU，确保行业领先的性能和与尖端技术的集成。

+   **优化适用于Google Cloud：** Vertex AI提供广泛的MLOps工具集，具有多种调整选项和内置推理优化的一键部署功能。您可以使用完全托管的Vertex AI工具或自管理的GKE进行高级定制，包括在GPU、TPU和CPU上从任何平台部署到成本高效的基础设施。

## 免费的研发学分

Gemma是为推动AI创新的开放开发者和研究者社区打造的。您可以通过Kaggle的免费访问、Colab笔记本的免费层以及首次使用Google Cloud的$300学分开始使用Gemma。研究人员还可以申请高达共$500,000的[Google Cloud学分](https://docs.google.com/forms/d/e/1FAIpQLSe0grG6mRFW6dNF3Rb1h_YvKqUp2GaXiglZBgA2Os5iTLWlcg/viewform)来加速他们的项目。

## 入门指南

您可以在[ai.google.dev/gemma](http://ai.google.dev/gemma)了解更多关于Gemma的内容并访问快速入门指南。

随着我们继续扩展Gemma模型系列，我们期待为不同应用引入新的变体。请关注接下来几周的活动和机会，与Gemma一起连接、学习和构建。

我们期待看到您创造的成果！
