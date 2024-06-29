<!--yml

category: 未分类

date: 2024-05-27 12:54:12

-->

# Yi-34B、Llama 2以及LLM训练中的常见实践：对《纽约时报》的事实检验 | EleutherAI博客

> 来源：[https://blog.eleuther.ai/nyt-yi-34b-response/](https://blog.eleuther.ai/nyt-yi-34b-response/)

在2024年2月21日，《纽约时报》发表了[“中国加快主导人工智能的步伐：伴随着一个扭曲：它依赖于美国技术。”](https://www.nytimes.com/2024/02/21/technology/china-united-states-artificial-intelligence.html) 作者声称，中国初创公司01.AI的最新大型语言模型Yi-34B在本质上依赖于Meta的Llama 2：

> 有一个扭曲之处：01.AI系统中的一些技术来自Llama。李先生的创业公司随后利用Meta的技术，用新数据训练其系统，使其更加强大。

这一评估基于对[cited Hugging Face issue](https://huggingface.co/01-ai/Yi-34B/discussions/11)的误读。尽管我们对中美人工智能竞争的整体状态没有任何主张，但我们想解释为什么01.AI所做的事情并不特别，并且完全在常见的机器学习实践范围内。

简而言之，所有现代大型语言模型（LLMs）都是由相同的算法构建模块制成的。Llama 2与原始的2017年Transformer之间的架构差异并非由Meta发明，并且所有这些都是公开的，因为计算机科学中的开放获取出版是常态。因此，尽管Yi-34B采用了Llama 2的架构，Meta的模型并没有使01.AI获得任何先前无法访问的创新。

在2023年11月，一位Hugging Face用户要求将Yi-34B的两个组件重命名，以使该模型默认兼容Llama 2的第三方代码库。训练数据是LLM之间差异的主要原因，而在这方面，Meta没有披露任何有用的细节；01.AI开发并描述了他们自己的英中数据集。Yi-34B与Llama 2之间的相似性并不支持中国人工智能公司仅仅依赖于美国开放模型的论点，因为所有LLM都具有极其相似的架构。此外，我们注意到Llama 2中的大部分架构创新来自于实现已知并在研究文献中描述的算法。

### [Hugging Face讨论的非编程人员概述](#an-overview-of-the-hugging-face-discussion-for-non-coders)

就像其他软件中一样，语言模型开发者为软件组件指定名称，以便以后引用。例如，神经网络的第一层通常被命名为 `layer_one`。重要的是，实际名称没有语义内容，它只是一个符号，用于指代软件的特定部分。然而，这些名称对于开发者之间的互操作性非常重要。如果两个人训练神经网络，但一个称第一层为 `layer_one`，另一个称其为 `layer_1`，那么第三方代码要同时与它们交互将非常不方便，因为第三方代码需要能够按名称引用组件。

当 01.AI 将 Yi-34B 上传到 Hugging Face Hub 时，他们使用了与 Meta 不同的命名约定。这意味着旨在与 Llama 2 交互的第三方代码无法与 01.AI 的模型配合使用。每个第三方开发者可以通过调整其代码来解决此问题，但在所有第三方开发者中，这将累计产生大量工作量。HF 提出问题的目的是要求 01.AI 重命名这些组件，以避免这种额外工作量。

在开源发布中，这类兼容性问题很常见，但并不表明 01.AI 有恶意意图或依赖 Llama 来训练他们的模型。EleutherAI 在我们的早期模型发布中也遇到过这种情况，现在我们已将兼容性检查作为标准的预发布审查的一部分。

然而，一些[媒体](https://technode.com/2023/11/15/kai-fu-lees-ai-large-language-model-yi-used-metas-llama-architecture-without-namechecking-its-source/)在11月份夸大了 Hugging Face 的问题，声称 01.AI 故意隐瞒了其主导模型与 Llama 2 的关联。这些夸张的说法在机器学习社区中并未引起多少关注。

### 所有 LLMs 的相似性[#](#how-all-llms-are-similar)

在高层次上，目前每个现有的大型语言模型都是使用相同的基本技术和方法进行训练的。开发者从网络上的大量文本数据集开始，将其分成称为 "tokens" 的单个类似词的组件。然后，通过训练一个深度学习架构（几乎总是基于注意力机制的 Transformer，如由 Vaswani 等人于 2017 年提出的）来适应数据集，通过优化损失函数来预测数据集中接下来最有可能出现的下一个 token。这个过程被称为 "预训练"。

自从Google Brain在2017年引入Transformer以来，这种基本的配方和其中使用的构建模块基本上没有根本性的变化，稍作调整后成为今天的由OpenAI在[GPT-1](https://s3-us-west-2.amazonaws.com/openai-assets/research-covers/language-unsupervised/language_understanding_paper.pdf)和[GPT-2](https://d4mucfpksywv.cloudfront.net/better-language-models/language_models_are_unsupervised_multitask_learners.pdf)中采用的从左到右语言模型。

例如，Meta的Llama 2系列模型的神经网络架构只是在2017年原始Transformer或GPT-2或[GPT-3](https://arxiv.org/abs/2005.14165)中采用了几个区别的变化：

[SwiGLU激活函数](https://arxiv.org/abs/2002.05202)是对变压器架构的轻微改动，据观察能提供轻微的质量改进。它是由Google Brain的Noam Shazeer于2020年引入的，并随后被Google的PaLM模型采用。现在**事实上**被大多数新语言模型使用，尽管不是普遍。

[旋转位置嵌入（RoPE）](https://arxiv.org/abs/2104.09864)是由2021年由珠宜科技公司的一群中国研究人员引入的方法，允许语言模型更清晰地跟踪文本中两个不同标记的相对“位置”。这一技术后来在西方研究社区中部分由[EleutherAI](https://blog.eleuther.ai/rotary-embeddings/)推广，并因其简单性和提供的几个实用的生活质量改进而成为当前绝大多数LLM的标准。

[“Pre-norm”架构](https://arxiv.org/abs/2002.04745)是变压器模型操作的重新排序，使得它在数学上更加清洁可靠地训练。这一修正由包括中国学术界和微软亚洲研究院的合作提出。作为一个有趣的历史插曲，这一调整最初是由Vaswani等人（2017年）在原始变压器论文中发现并使用的，但报告不一致。

- [多查询注意力](https://arxiv.org/abs/1911.02150)和[分组查询注意力](https://arxiv.org/abs/2305.13245)都是由Google提出的。多查询注意力（MQA）是一种提高同时运行多个输入到语言模型中以供下游使用效率的改进技术。它最早由Google Brain的Noam Shazeer在2019年提出，并被[Google的PaLM模型](https://arxiv.org/abs/2204.02311)大规模采用。分组查询注意力（GQA）是相对较新的提议，于2023年由Google Research提出，作为多查询注意力的扩展，并保留了其优势。分组查询注意力在Llama 2中的应用已帮助其成为许多最近语言模型架构的新基准。

尽管这并不完全削弱Meta在训练Llama 2方面的成就，但重要的是要注意，这些进展都不是由Meta发明的，并且其中许多技术已经存在了几年。这些技术在Llama 2创建和发布时已被视为事实上的最佳实践。

我们希望这说明了今天的LLM之间的共同点，其中许多追溯到几年前的原始Transformer论文和第一个基于Transformer的模型。

### 那么Yi-34B与Llama 2有何不同？[#](#so-what-makes-yi-34b-different-from-llama-2)

虽然Llama 2模型的权重以及它们的架构细节是可用的（这对其他开发人员或用户在其代码中运行模型是必要的），但有必要看看Llama 2创建过程中未披露的组成部分，这表明了真正的差异化因素。

Llama 2的[技术报告](https://arxiv.org/abs/2307.09288)仅对其数据集的确切内容做了以下说明：

> 我们的训练语料库包括来自公开来源的新数据混合，不包括 Meta 的产品或服务数据。我们努力删除了来自某些已知包含大量私人信息的网站的数据。

关于数据来源的进一步细节未提供。这是因为，由于当前的语言模型大多使用类似的架构，Meta 的关键竞争优势来自他们的训练数据集。

此外，尽管Meta提供了一个[文件](https://github.com/meta-llama/llama/blob/main/llama/model.py)来定义Llama架构，可以用来加载他们的模型，但他们也**没有**公开用于从头开始训练Llama模型的基础设施或代码库。要实际训练自己的语言模型，需要组建自己的数据集并在自己的计算集群上构建这种训练基础设施，这进一步需要具备高性能计算（HPC）专业知识的工程团队。释放Llama 2模型权重并不会使其他团队能够免费开发竞争对手，特别是考虑到像SwiGLU或GQA这样的进展已经通过开放获取的学术出版物公开知晓。此外，Llama 2还为Meta带来了明显的好处：Hugging Face的讨论将Llama定位为开放权重LLM软件的当前标准，因此不使用其架构和命名约定将是一个不利因素。

在《纽约时报》文章发布两周后，01.AI发布了[Yi技术报告](https://arxiv.org/abs/2403.04652)，提到了这些因素。或许是作为对负面媒体报道的回应，他们明确表示“标准”架构是足够的，而要花费更多精力来筛选和处理高质量的数据集。对于Yi-34B来说，这包括确保在中文表现良好，并且他们不得不从头开始构建这个数据处理流水线以及模型预训练基础设施。因此，01.AI必须解决与其他语言模型训练公司面临的相同问题，并不像“建立在”Llama 2之上，而是建立在其他从未发布的语言模型（例如谷歌的PaLM）之上。

### 结论[#](#conclusion)

我们希望这篇文章能说明**Yi-34B**如何融入语言模型训练的更大趋势。我们想强调《纽约时报》的作者们并不是疏忽大意，错误完全可以理解，因为要解释这个[Hugging Face问题](https://huggingface.co/01-ai/Yi-34B/discussions/11)，显然需要大量关于模型开发和当前开放AI生态系统的背景知识。我们不能指望记者们具备这种机器学习水平。我们在这里的目标是教育，因为我们认为公众对AI的讨论必须以事实为基础。在这种情况下，01.AI公司的创始人在指出公司遵循了标准行业实践时是完全正确的。
