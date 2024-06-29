<!--yml

category: 未分类

date: 2024-05-27 15:01:09

-->

# GPU的竞争对手？什么是语言处理单元（LPU）

> 来源：[https://www.turingpost.com/p/fod41](https://www.turingpost.com/p/fod41)

## 下周在图灵邮报：

+   星期三，Token 1.21：**模型安全与数据隐私**

+   星期五，AI独角兽：**Scale AI**

*图灵邮报是一份由读者支持的出版物。要完全访问我们最有趣的文章和调查，请成为付费订户→*

本周，一家鲜为人知的公司，[Groq](https://groq.com/?utm_source=www.turingpost.com&utm_medium=referral&utm_campaign=fod-41-gpu-s-rival-what-is-language-processing-unit-lpu)，展示了前所未有的速度，在Groq的语言处理单元（LPU）上运行开源LLMs（如Llama-2（700亿参数）每秒超过100个标记，Mixtral每用户每秒接近500个标记。

+   “根据Groq的说法，在类似测试中，ChatGPT在典型的基于GPU的计算系统上每秒加载40-50个标记，而Bard每秒70个标记。

+   每用户每秒100个标记的背景——用户可以在一分钟多一点的时间内生成一篇4000字的文章。”

所以：**LPU是什么，它是如何工作的**，Groq（这个名字真不幸，鉴于马斯克的Grok在媒体上随处可见）又来自哪里？

记得2016年的围棋比赛吗？AlphaGo对阵世界冠军李世石并取得胜利？嗯，在比赛前大约一个月，有一场测试比赛AlphaGo输了。DeepMind的研究人员将AlphaGo移植到Tensor处理单元（TPU），然后计算机程序以大幅度的优势获胜。

认识到计算能力是AI潜力的瓶颈，这促使了Groq的诞生以及LPU的创建。这一认识最初来自[Jonathan Ross](https://www.linkedin.com/in/ross-jonathan/?utm_source=www.turingpost.com&utm_medium=referral&utm_campaign=fod-41-gpu-s-rival-what-is-language-processing-unit-lpu)，他最初开始了Google的TPU项目。他于2016年创立了Groq。

**LPU是一种特殊的计算机大脑，设计用于非常快速地处理语言任务。**与同时执行多项任务的其他计算机芯片（并行处理）不同，LPU依次处理任务（顺序处理），这非常适合理解和生成语言。可以将其想象成接力比赛，每位跑步者（芯片）将接力棒（数据）传递给下一位，使一切运行得非常快速。LPU的设计旨在克服两个LLM的瓶颈：计算密度和内存带宽。

Groq从一开始就采用了新颖的方法，**专注于软件和编译器开发**，甚至在考虑硬件之前。他们确保软件能够指导芯片如何相互通信，确保它们像工厂中的团队一样无缝协作。这使得LPU在处理语言时效率和速度都非常高，非常适合涉及理解或创建文本的AI任务。

这导致了一个高度优化的系统，不仅在速度方面远远超过传统设置，而且在成本效率和能源消耗方面表现出色。对于金融、政府和科技等行业来说，快速和准确的数据处理至关重要。

现在，请不要着急丢弃你们的GPU！虽然LPU在推理方面表现出色，轻松应用训练模型到新数据上，但GPU在训练领域仍然占据主导地位。LPU和GPU可能成为AI硬件的动态二人组，各自在其角色中表现出色。

正如Elvis Saravia所说：“*随着推理和长文本理解的突破，我们正式进入了LLM的新时代*。”

为了更好地理解架构，Groq提供了两篇论文：[2020年](https://wow.groq.com/groq-isca-paper-2020/?utm_source=www.turingpost.com&utm_medium=referral&utm_campaign=fod-41-gpu-s-rival-what-is-language-processing-unit-lpu)（*快速思考：用于加速深度学习工作负载的张量流处理器（TSP）*）和[2022年](https://wow.groq.com/isca-2022-paper/?utm_source=www.turingpost.com&utm_medium=referral&utm_campaign=fod-41-gpu-s-rival-what-is-language-processing-unit-lpu)（*用于大规模机器学习的软件定义张量流多处理器*）。由于这些论文中从未提到过“LPU”，这个术语可能是Groq最近才添加的。

+   这篇论文还涉及计算：[计算能力与人工智能治理](https://arxiv.org/pdf/2402.08797.pdf?utm_source=www.turingpost.com&utm_medium=referral&utm_campaign=fod-41-gpu-s-rival-what-is-language-processing-unit-lpu)，讨论通过计算控制管理AI发展，专注于其监管、利益和风险潜力，并建议平衡的治理方法。

+   与此同时，美国向全球第三大合同芯片制造商GlobalFoundries [授予](https://www.reuters.com/technology/us-awards-15-bln-globalfoundries-domestic-semiconductor-production-2024-02-19?utm_source=www.turingpost.com&utm_medium=referral&utm_campaign=fod-41-gpu-s-rival-what-is-language-processing-unit-lpu) 15亿美元，以增加半导体生产，增强国内供应链，在纽约和佛蒙特州进行扩展。

+   由加州大学伯克利分校人工智能研究（BAIR）发布的论文认为，“**复合人工智能系统可能是未来最大化AI成果的最佳途径**，可能是2024年AI领域最有影响力的趋势之一。”

## 来自The Usual Suspects ©的消息

### Y Combinator

+   自2009年以来，Y Combinator发布了**[创业创意征集](https://www.ycombinator.com/rfs?utm_source=www.turingpost.com&utm_medium=referral&utm_campaign=fod-41-gpu-s-rival-what-is-language-processing-unit-lpu)**，暗示了“我们希望在未来几十年内成为现实的想法，涉及到我们认为未来重要的领域”。今年的列表包含20个类别：

### 20个大名鼎鼎的公司

+   二十大科技巨头，包括Adobe、Amazon、Google、IBM、Meta、Microsoft、OpenAI和TikTok，已经同意采取“合理预防措施”，以防止人工智能在全球选举中被滥用，[参考链接](https://apnews.com/article/ai-generated-election-deepfakes-munich-accord-meta-google-microsoft-tiktok-x-c40924ffc68c94fac74fa994c520fc06?utm_source=www.turingpost.com&utm_medium=referral&utm_campaign=fod-41-gpu-s-rival-what-is-language-processing-unit-lpu)。

### OpenAI

### 亮点模型：

+   **介绍Sora**：本文介绍了Sora，这是OpenAI在视频生成技术上的突破，能够生成高保真度视频。它利用时空补丁处理不同长度和分辨率的视频，向模拟物理世界的3D一致性和长程连贯性迈出了重要一步。这代表了在创建详细模拟方面的飞跃，可用于从娱乐到虚拟测试环境等多种应用 [→阅读论文](https://openai.com/research/video-generation-models-as-world-simulators?utm_source=www.turingpost.com&utm_medium=referral&utm_campaign=fod-41-gpu-s-rival-what-is-language-processing-unit-lpu)

+   **额外阅读：**

+   **介绍V-Jepa**（Yann LeCun对先进机器智能（AMI）的愿景）：Meta的V-JEPA模型通过仅使用特征预测作为其唯一目标，革新了从视频中进行无监督学习的方式。这种方法绕过了需要预先训练的图像编码器或文本注释，而是依靠视频数据的内在动态来学习多功能视觉表示。这是对无监督视觉学习领域的重大贡献，有望在机器理解运动和外观时不需要显式指导方面取得进展 [→阅读论文](https://scontent-lga3-1.xx.fbcdn.net/v/t39.2365-6/427986745_768441298640104_1604906292521363076_n.pdf?_nc_cat=103&ccb=1-7&_nc_sid=3c67a6&_nc_ohc=Lpq5IeF5ftUAX_QVyw0&_nc_ht=scontent-lga3-1.xx&oh=00_AfCxzl-IotQM2_EGXbgWfRe66yPVffGfxGm5oSY74v1Slw&oe=65D898F1&utm_source=www.turingpost.com&utm_medium=referral&utm_campaign=fod-41-gpu-s-rival-what-is-language-processing-unit-lpu)

+   **介绍 Gemini 1.5**：Google DeepMind 的 Gemini 1.5 引入了专家混合架构，增强了模型在更广泛任务上的性能。特别是，它将上下文窗口扩展到 100 万个标记，能够深入分析大型数据集。Gemini 1.5 代表了人工智能在处理和理解广泛语境方面的重大进展，标志着多模态模型发展的一个里程碑 [→阅读论文](https://blog.google/technology/ai/google-gemini-next-generation-model-february-2024/?utm_source=www.turingpost.com&utm_medium=referral&utm_campaign=fod-41-gpu-s-rival-what-is-language-processing-unit-lpu#build-experiment)

+   **介绍 Stable Cascade**：Stability AI 的 Stable Cascade 引入了一种新的文本到图像生成框架，优先考虑效率、易于训练以及在消费级硬件上的微调。模型的分层压缩技术显著减少了训练高质量生成模型所需的资源，为人工智能社区提供了更广泛的可访问性和实验路径 [→阅读论文](https://stability.ai/news/introducing-stable-cascade?utm_source=www.turingpost.com&utm_medium=referral&utm_campaign=fod-41-gpu-s-rival-what-is-language-processing-unit-lpu)

## 为您方便分类的最新研究论文

### 语言理解与生成

+   **OpenToM**：探索评估语言模型的心理理论推理能力，解决它们理解复杂社会和心理叙事的能力。[阅读论文](https://arxiv.org/pdf/2402.06044.pdf?utm_source=www.turingpost.com&utm_medium=referral&utm_campaign=fod-41-gpu-s-rival-what-is-language-processing-unit-lpu)

+   **在一堆 10M 中寻找针**：展示了自然语言处理模型处理异常长文档的能力，推动了文档长度理解的边界。[阅读论文](https://arxiv.org/pdf/2402.10790.pdf?utm_source=www.turingpost.com&utm_medium=referral&utm_campaign=fod-41-gpu-s-rival-what-is-language-processing-unit-lpu)

+   **前提顺序在逻辑推理中的重要性**：研究了语言模型对前提顺序的敏感性，揭示了对推理任务的影响。[阅读论文](https://arxiv.org/pdf/2402.08939.pdf?utm_source=www.turingpost.com&utm_medium=referral&utm_campaign=fod-41-gpu-s-rival-what-is-language-processing-unit-lpu)

+   **无需提示的链式思维推理**：揭示了语言模型生成推理路径的固有能力，提出了一种替代显式提示的方法。[阅读论文](https://arxiv.org/pdf/2402.10200.pdf?utm_source=www.turingpost.com&utm_medium=referral&utm_campaign=fod-41-gpu-s-rival-what-is-language-processing-unit-lpu)

+   **用直接原则反馈抑制粉色大象**：解决了LLM中主题回避的挑战，提出了一种新的微调方法，以增强可控性。[阅读论文](https://arxiv.org/pdf/2402.07896.pdf?utm_source=www.turingpost.com&utm_medium=referral&utm_campaign=fod-41-gpu-s-rival-what-is-language-processing-unit-lpu)

+   **GhostWriter**：开发了一个以AI为动力的写作环境，侧重于个性化和增强用户在协作写作中的控制权。[阅读论文](https://arxiv.org/pdf/2402.08855.pdf?utm_source=www.turingpost.com&utm_medium=referral&utm_campaign=fod-41-gpu-s-rival-what-is-language-processing-unit-lpu)

### 语音和文本到语音技术

### 数学和科学推理

+   **OpenMathInstruct-1**：开发了一个用于数学指导调整的数据集，旨在提高LLM在数学推理能力方面的表现。[阅读论文](https://arxiv.org/pdf/2402.10176.pdf?utm_source=www.turingpost.com&utm_medium=referral&utm_campaign=fod-41-gpu-s-rival-what-is-language-processing-unit-lpu)

+   **InternLM-Math**：介绍了一种专门用于数学推理的语言模型，结合了各种技术，以增强数学问题解决能力。[阅读论文](https://arxiv.org/pdf/2402.06332.pdf?utm_source=www.turingpost.com&utm_medium=referral&utm_campaign=fod-41-gpu-s-rival-what-is-language-processing-unit-lpu)

+   **ChemLLM**：创建了第一个专用于化学的LLM，将结构化的化学数据转化为多样化的化学任务对话。[阅读论文](https://arxiv.org/pdf/2402.06852.pdf?utm_source=www.turingpost.com&utm_medium=referral&utm_campaign=fod-41-gpu-s-rival-what-is-language-processing-unit-lpu)

### AI中的效率和数据利用

+   **如何训练数据高效的LLM**：提出了增强LLM训练数据效率的采样方法，优化示例选择。[阅读论文](https://arxiv.org/pdf/2402.09668.pdf?utm_source=www.turingpost.com&utm_medium=referral&utm_campaign=fod-41-gpu-s-rival-what-is-language-processing-unit-lpu)

+   **FIDDLER**：引入了一种用于有效推断MoE模型的系统，利用CPU-GPU协同作业来提高资源受限环境下的性能。[阅读论文](https://arxiv.org/pdf/2402.07033.pdf?utm_source=www.turingpost.com&utm_medium=referral&utm_campaign=fod-41-gpu-s-rival-what-is-language-processing-unit-lpu)

+   **Tandem Transformers**：提出了一种用于提高LLM推理效率的架构，利用双模型系统进行更快速和准确的预测。[阅读论文](https://arxiv.org/pdf/2402.08644.pdf?utm_source=www.turingpost.com&utm_medium=referral&utm_campaign=fod-41-gpu-s-rival-what-is-language-processing-unit-lpu)

+   **走向超大规模变压器后训练量化的下一级**: 提出了一种先进的 PTQ 算法，用于在边缘设备上高效部署大型变压器模型。[阅读论文](https://arxiv.org/pdf/2402.08958.pdf?utm_source=www.turingpost.com&utm_medium=referral&utm_campaign=fod-41-gpu-s-rival-what-is-language-processing-unit-lpu)

### 多模态和视觉语言模型

### 强化学习与模型行为

+   **ODIN**: 处理 RLHF 中的奖励黑客问题，提出了一种方法来减轻 LLM 中的冗长偏见，以获得更简洁和内容聚焦的响应。[阅读论文](https://arxiv.org/pdf/2402.07319.pdf?utm_source=www.turingpost.com&utm_medium=referral&utm_campaign=fod-41-gpu-s-rival-what-is-language-processing-unit-lpu)

+   **专家混合解锁深度 RL 参数缩放**: 展示了 MoE 模块对深度 RL 网络的影响，增强了参数可伸缩性和性能。[阅读论文](https://arxiv.org/pdf/2402.08609.pdf?utm_source=www.turingpost.com&utm_medium=referral&utm_campaign=fod-41-gpu-s-rival-what-is-language-processing-unit-lpu)

### 操作系统和通用代理

### 图学习和状态空间模型

### AI 的挑战与创新

+   **一个尾巴的故事**: 探讨了合成数据对神经模型性能的影响，理论上推测了模型依赖合成数据可能导致模型崩溃的潜在风险。[阅读论文](https://arxiv.org/pdf/2402.07043.pdf?utm_source=www.turingpost.com&utm_medium=referral&utm_campaign=fod-41-gpu-s-rival-what-is-language-processing-unit-lpu)

+   **变压器可以实现长度泛化但不够健壮**: 研究变压器对更长序列的泛化能力，突显了保持健壮性能的挑战。[阅读论文](https://arxiv.org/pdf/2402.09371.pdf?utm_source=www.turingpost.com&utm_medium=referral&utm_campaign=fod-41-gpu-s-rival-what-is-language-processing-unit-lpu)

*今天成为我们的高级订阅者！在大多数情况下，* ***您可以通过公司报销此订阅！***🤍
