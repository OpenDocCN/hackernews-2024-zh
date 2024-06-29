<!--yml

category: 未分类

date: 2024-05-27 13:29:41

-->

# Snowflake releases a flagship generative AI model of its own | TechCrunch

> 来源：[https://techcrunch.com/2024/04/24/snowflake-releases-a-flagship-generative-ai-model-of-its-own/](https://techcrunch.com/2024/04/24/snowflake-releases-a-flagship-generative-ai-model-of-its-own/)

总的来说，广泛适用的生成 AI 模型曾经是游戏规则的主旋律，而且可以说现在依然如此。但是，随着各大小云服务供应商加入生成 AI 领域，我们看到一批新的模型专注于最富裕的潜在客户：企业。

举例来说：[Snowflake](https://www.snowflake.com/en/)，这家云计算公司，今天推出了Arctic LLM，一个被描述为“企业级”的生成 AI 模型。根据Apache 2.0许可证提供，Snowflake称Arctic LLM针对“企业工作负载”进行了优化，包括生成数据库代码，并且可供研究和商业使用免费。

“我认为这将是我们 — Snowflake — 和我们的客户构建企业级产品并真正开始实现人工智能的承诺和价值的基础，”CEO Sridhar Ramaswamy在新闻简报中说道。“你应该把这视为我们在生成 AI 领域的第一步，但是是一大步，而且还有更多的步骤即将到来。”

## 一个企业模型

我的同事Devin Coldewey最近写道，生成 AI 模型的浪潮似乎没有尽头。我建议你[阅读他的文章](https://techcrunch.com/2024/04/19/too-many-models/#:~:text=How%20many%20AI%20models%20is,ever%20possible%20to%20begin%20with.)，但要点是：模型是厂商推动他们的研发激发兴奋的简单方式，同时也是他们产品生态系统（例如模型托管、精细调整等）的一部分。

Arctic LLM 也不例外。[Arctic 家族中的生成 AI 模型之一](https://www.snowflake.com/blog/introducing-snowflake-arctic-embed-snowflakes-state-of-the-art-text-embedding-family-of-models/)，Arctic LLM 经过约三个月、1000个GPU和200万美元的培训，紧随Databricks的[DBRX](https://techcrunch.com/2024/03/27/databricks-spent-10m-on-a-generative-ai-model-that-still-cant-beat-gpt-4/)发布，该生成 AI 模型也被市场化为优化的企业空间。

Snowflake 在其新闻材料中直接比较了 Arctic LLM 和 DBRX，称 Arctic LLM 在编码（Snowflake 未具体说明使用哪种编程语言）和[SQL](https://en.wikipedia.org/wiki/SQL)生成的两个任务上优于 DBRX。该公司表示，Arctic LLM 在这些任务上也优于 Meta 的 Llama 2 70B（但不及较新的[Llama 3 70B](https://techcrunch.com/2024/04/18/meta-releases-llama-3-claims-its-among-the-best-open-models-available/)）和 Mistral 的 Mixtral-8x7B。

Snowflake 还声称，Arctic LLM 在流行的通用语言理解基准MMLU上实现了“领先的性能”。不过，我要注意的是，虽然MMLU声称评估生成模型在推理逻辑问题时的能力，但它包括可以通过死记硬背解决的测试，因此需要带着一定的怀疑来看待这一点。

“Arctic LLM 解决了企业部门内特定的需求，” Snowflake AI 主管 Baris Gultekin 在接受TechCrunch采访时说，“与撰写诗歌之类的通用AI应用不同，它专注于企业导向的挑战，如开发SQL合作伴侣和高质量聊天机器人。”

Arctic LLM，如DBRX和Google当前性能最佳的生成模型Gemini 1.5 Pro，是一种专家混合（MoE）架构。MoE架构基本上将数据处理任务分解为子任务，然后将其委托给更小、专门的“专家”模型。因此，虽然Arctic LLM包含4800亿个参数，但每次只激活170亿个——足以驱动128个独立的专家模型。（参数本质上定义了AI模型在解决问题上的技能，如分析和生成文本。）

Snowflake 声称，这种高效的设计使其能够以“大约相似模型成本的八分之一”训练Arctic LLM，使用的是开放公共网络数据集（包括 [RefinedWeb](https://arxiv.org/abs/2306.01116)、[C4](https://paperswithcode.com/dataset/c4)、[RedPajama](https://github.com/togethercomputer/RedPajama-Data) 和 [StarCoder](https://huggingface.co/datasets/bigcode/starcoderdata)）。

## 在任何地方运行

Snowflake 除了提供Arctic LLM之外，还提供了编码模板和培训资源列表，帮助用户完成模型的启动、运行和针对特定用例的优化过程。但是，鉴于这对大多数开发人员来说可能是昂贵和复杂的任务（调整或运行Arctic LLM需要大约八个GPU），Snowflake 还承诺将Arctic LLM提供给包括Hugging Face、Microsoft Azure、Together AI的模型托管服务和企业生成AI平台Lamini在内的多个主机上使用。

然而，问题在于：Arctic LLM 首先将在Cortex上推出，这是Snowflake用于构建基于AI和机器学习的应用和服务的平台。该公司显然将其视为运行Arctic LLM的首选方式，带有“安全性”、“治理”和可扩展性。

“我们在这里的梦想是，在一年内，提供一个API给我们的客户使用，这样业务用户可以直接与数据交流，” Ramaswamy 说道。“我们本可以选择说，‘哦，我们可以等待某个开源模型然后使用它。’相反，我们正在进行基础投资，因为我们认为这样做将为我们的客户带来更多的价值。”

因此，我在想：除了Snowflake的客户外，Arctic LLM真正是为谁准备的？

在一个充满“开放”的生成模型的景观中，这些模型几乎可以为任何目的进行微调，Arctic LLM在任何明显的方式上并没有脱颖而出。它的架构可能比其他选择带来效率提升。但我并不认为它们足够引导企业远离其他众所周知和支持良好的商业友好型生成模型（例如[GPT-4](https://techcrunch.com/tag/gpt-4/)）。

还有一个Arctic LLM不利的观点需要考虑：它相对较小的上下文。

在生成AI中，上下文窗口是指模型在生成输出（例如更多文本）之前考虑的输入数据（例如文本）。上下文窗口较小的模型往往容易忘记甚至是非常近期的对话内容，而具有较大上下文的模型通常避免了这种缺陷。

Arctic LLM的上下文在~8,000到~24,000个词之间，取决于微调的方法——远远低于Anthropic的Claude 3 Opus和Google的Gemini 1.5 Pro等模型。

Snowflake在营销中没有提到这一点，但Arctic LLM几乎可以肯定地受到与其他生成AI模型相同的限制和缺陷的困扰——即“幻觉”（即对请求的错误自信回答）。这是因为Arctic LLM以及每一个现存的生成AI模型都是一种统计概率机器——再次强调，它具有较小的上下文窗口。它根据大量示例猜测哪些数据在哪里放置最“合理”（例如在句子“I go to the market”中在“the market”之前放置“go”一词）。它将不可避免地猜测错误——这就是“幻觉”。

如Devin在他的文章中所写，直到下一次重大技术突破，增量改进都是我们在生成AI领域可以期待的全部。尽管如此，这并不会阻止像Snowflake这样的供应商将它们宣扬为伟大的成就，并将它们推广到极致。
