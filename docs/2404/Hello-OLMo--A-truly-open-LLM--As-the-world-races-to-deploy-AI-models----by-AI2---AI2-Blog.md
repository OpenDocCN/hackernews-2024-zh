<!--yml

category: 未分类

date: 2024-05-27 13:00:49

-->

# 你好，OLMo：一款真正开放的LLM。随着世界竞相部署AI模型... | 作者：AI2 | AI2 博客

> 来源：[https://blog.allenai.org/hello-olmo-a-truly-open-llm-43f7e7359222?gi=760105621962](https://blog.allenai.org/hello-olmo-a-truly-open-llm-43f7e7359222?gi=760105621962)

# **你好，OLMo：一款真正开放的LLM**

随着世界竞相部署既有效又安全的AI模型，对开放大语言模型（LLM）的需求激增。开放和封闭AI模型的广泛采用意味着AI能力已经超越了我们理解它们如何被创造的能力。发布OLMo框架将为行业提供了解AI模型内部运作的机会。

今天，[The Allen Institute for AI (AI2)](https://allenai.org/olmo)发布了[OLMo 7B](https://huggingface.co/allenai/OLMo-7B)，一款真正开放、先进的大语言模型，同时发布了预训练数据和训练代码。这使研究人员和开发人员能够共同使用最佳和开放的模型来推进语言模型科学。

> “开放基础模型在推动生成AI周围的创新和发展方面起到了关键作用，”Meta的首席AI科学家Yann LeCun说道。“开源社区带来的充满活力的社区是构建AI未来的最快、最有效的方式。”

OLMo和其框架旨在帮助研究人员训练和实验大型语言模型。它们可直接在[Hugging Face](https://huggingface.co/allenai/OLMo-7B)和[GitHub](https://github.com/allenai/OLMo)上下载。这项工作在某种程度上通过与[哈佛大学肯普纳自然与人工智能研究所](https://www.harvard.edu/kempner-institute/)和包括[AMD](https://www.amd.com/en.html)、[CSC](https://www.csc.fi/en/home)（[Lumi超级计算机](https://www.lumi-supercomputer.eu/)）、[华盛顿大学保罗·艾伦计算机科学与工程学院](https://www.cs.washington.edu/)以及[Databricks](https://www.databricks.com/)的合作实现。

该框架提供了一套完全开放的AI开发工具，包括：

+   **完整的预训练数据：** 该模型基于AI2的[Dolma](/dolma-3-trillion-tokens-open-llm-corpus-9a0ff4b8da64)数据集，包括用于语言模型预训练的三万亿标记的开放语料库，包括生成训练数据的代码。

+   **训练代码和模型权重：** OLMo 框架包括四种 7B 规模模型变体的完整模型权重，每个模型至少训练了 2T 个标记。推理代码、训练指标和训练日志均已提供。

+   **评估：** 我们发布了开发中使用的评估套件，每个模型每1000步提供500多个检查点，以及Catwalk项目下的评估代码。

> “我对将OLMo交到AI研究人员手中感到非常激动，”微软首席科学家、AI2科学顾问委员会的创始成员埃里克·霍维茨说道。“这一新提供延续了Allen AI提供有价值的开放模型、工具和数据的传统，这些已经在全球社区的AI领域推动了许多进步。”

## **真正开放的模型：**

通过将OLMo及其训练数据完全提供给公众，AI2迈出了构建世界上最好的开放语言模型的重要一步。在未来几个月中，AI2将继续对OLMo进行迭代，并将不同的模型大小、模态、数据集和功能引入OLMo家族。

> “如今许多语言模型发布时都缺乏透明度。没有访问训练数据，研究人员就无法科学地理解模型的工作原理。这相当于没有临床试验进行药物发现，或者在没有望远镜的情况下研究太阳系。” [汉娜·哈吉什尔齐](https://homes.cs.washington.edu/~hannaneh/)说道，她是OLMo项目负责人，AI2的NLP研究高级总监，也是华盛顿大学艾伦学院的教授。“通过我们的新框架，研究人员最终将能够研究LLM的科学，这对于构建下一代安全可信赖的AI至关重要。”

有了OLMo，AI研究人员和开发人员将体验到：

+   **更精确：** 有了对模型背后的训练数据的全面了解，研究人员将能够更快地工作，不再需要依赖对模型表现的定性假设，而可以进行科学测试。

+   **更少的碳排放：** 目前，一次训练运行相当于[九个美国家庭一年的排放量](https://www.epa.gov/energy/greenhouse-gas-equivalencies-calculator)。通过开放完整的训练和评估生态系统，它大大减少了开发中的冗余，这对于AI的脱碳至关重要。

+   **持久的结果：** 将模型及其数据集保持开放，而不是隐藏在API后面，使研究人员能够从以前的模型和工作中学习和构建。

“在OLMo的帮助下，开放*实际上*意味着‘开放’，AI研究界的每个人都将可以访问模型创建的所有方面，包括训练代码、评估方法、数据等等。” [诺亚·史密斯](https://nasmith.github.io/) 说道，他是OLMo项目负责人，AI2的NLP研究高级总监，也是华盛顿大学艾伦学院的教授。“AI曾经是一个以积极的研究社区为中心的开放领域，但随着模型的增长，变得更加昂贵，并且开始变成商业产品，AI工作开始在幕后进行。通过OLMo，我们希望抵制这种趋势，并赋予研究社区以更好地以科学方式理解和参与语言模型，从而产生对每个人都有益的更负责任的AI技术。”

“AI2在自然语言处理方面拥有深厚的专业知识，结合AMD高性能计算引擎，由AMD EPYC™ CPU和AMD Instinct™加速器驱动的LUMI超级计算机上开发的OLMo模型为真正扩展AI实验和创新，推动行业发展提供了独特的机会。这一新的开放框架将为全球AI研究社区提供可信赖的资源和平台，直接参与和开发语言模型。” — Ian Ferreria，AMD AI解决方案高级总监

“我们很高兴能通过提供来自LUMI超级计算机的计算能力以及我们的专业知识，为这一重要倡议做出贡献。像LUMI这样的公共超级计算机在开放和透明的AI基础设施中发挥着至关重要的作用。” Dr. Pekka Manninen，CSC科学技术总监

LUMI超级计算机位于芬兰，由CSC托管，由EuroHPC联合企业和10个欧洲国家拥有。LUMI是欧洲最快的超级计算机，以完全无碳运营而闻名，并在支持开发OLMo所需的预训练工作中发挥了关键作用。

“Databricks很高兴与艾伦人工智能研究所合作发布他们的OLMo开源模型和框架。OLMo设定了开放标准的新标准。学术界、工业界和更广泛的社区的每个人都将从访问模型以及包括数据、代码和中间检查点在内的所有培训细节中受益良多。我特别自豪地宣布，这个模型是在Databricks的Mosaic AI模型训练平台上开发的。正如所有伟大的开源发布一样，现在这些工具和艺术品已经交到社区手中，最好的时代即将到来。” — Jonathan Frankle，Databricks首席科学家（神经网络）

## 了解更多

[OLMo技术博客入门指南](/olmo-open-language-model-87ccfc95f580)

[OLMo 7B技术报告](https://allenai.org/olmo/olmo-paper.pdf)

[获取OLMo 7B](https://huggingface.co/allenai/OLMo-7B)

有关OLMo框架和艾伦人工智能研究所的更多信息，请访问[这里](https://allenai.org/olmo)。
