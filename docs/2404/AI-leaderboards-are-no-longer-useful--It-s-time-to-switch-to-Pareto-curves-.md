<!--yml

类别：未分类

日期：2024-05-27 13:40:45

-->

# AI 排行榜已不再实用。现在是时候转向帕累托曲线了。

> 来源：[https://www.aisnakeoil.com/p/ai-leaderboards-are-no-longer-useful](https://www.aisnakeoil.com/p/ai-leaderboards-are-no-longer-useful)

*作者：Sayash Kapoor，[Benedikt Stroebl](https://citp.princeton.edu/citp-people/benedikt-strobl/)，Arvind Narayanan*

哪种AI系统最适合生成代码？令人惊讶的是，目前还没有一个很好的方法来回答这类问题。

基于[HumanEval](https://paperswithcode.com/sota/code-generation-on-humaneval)，用于代码生成的广泛使用的基准，目前公开可用的最准确系统是[LDB](https://arxiv.org/abs/2402.16906)（LLM调试器的简称）。但有个问题。包括LDB在内的最准确的生成式AI系统往往是代理，它们反复调用像GPT-4这样的语言模型。这意味着它们的运行成本可能比模型本身高出数个数量级（模型本身已经非常昂贵）。如果我们为了提高2%的准确性而增加100倍的成本，这真的算是更好的选择吗？

在本文中，我们认为：

+   没有控制成本的AI代理准确性测量是没有用的。

+   帕累托曲线可以帮助可视化准确性与成本的权衡。

+   当前最先进的代理架构复杂且昂贵，但在某些情况下，其准确性不如成本低50倍的极其简单的基线代理。

+   例如参数计数之类的成本代理是误导性的，如果目标是识别适合特定任务的最佳系统，则应直接测量美元成本。

+   发布的代理评估由于缺乏标准化和某些情况下有问题的、未记录的评估方法而难以重现。

LLM（大型语言模型）是随机的。简单地多次调用模型并输出最常见的答案可以提高准确性。

在某些任务上，似乎可以无限量地提高推理计算量以提高准确性。Google Deepmind的[AlphaCode](https://arxiv.org/pdf/2203.07814.pdf)，通过改进自动编码评估的准确性，显示出即使调用LLM数百万次，这一趋势仍然存在。

*尽管对底层模型进行了一百万次调用（不同的曲线代表不同的参数计数），但[AlphaCode](https://arxiv.org/pdf/2203.07814.pdf)在编码任务上的准确性仍在不断提高。准确性是通过模型生成的前十个答案中多少次正确来衡量的。*

因此，对代理进行有用的评估必须问：成本是多少？如果我们不进行成本控制的比较，将鼓励研究人员开发极其昂贵的代理，仅仅是为了声称它们在排行榜上名列前茅。

实际上，当我们评估去年提出的解决编码任务的代理时，我们发现可视化成本与准确性之间的权衡产生了令人惊讶的洞见。

我们重新评估了三个声称在HumanEval排行榜上占据前几位的代理的准确性：[LDB](https://arxiv.org/pdf/2402.16906v3.pdf)、[LATS](https://arxiv.org/pdf/2310.04406v2.pdf) 和 [Reflexion](https://arxiv.org/pdf/2303.11366.pdf)。我们还评估了运行这些代理所需的成本和时间要求。

这些代理依赖于运行模型生成的代码，如果未能通过问题描述中提供的测试用例，则尝试调试代码，查看代码生成过程中的替代路径，或者在生成另一种解决方案之前“反思”为何模型输出不正确。

另外，我们计算了几个简单基准模型的准确性、成本和运行时间：

+   **GPT-3.5** 和 **GPT-4** 模型（零预测；无代理架构）

+   **重试：** 如果未能通过问题描述中提供的测试用例，我们会重复以零温度设置调用模型，最多五次。重试是有意义的，因为即使在温度为零的情况下，LLM [不是确定性的](https://community.openai.com/t/run-same-query-many-times-different-results/140588#:~:text=Apr%202023-,OpenAI%20models%20are%20non%2Ddeterministic%2C%20meaning%20that%20identical%20inputs%20can%20yield,amount%20of%20variability%20may%20remain%20due%20to%20GPU%20floating%20point%20math.,-Solution)。

+   **温升：** 这与重试策略相同，但我们会逐步增加每次运行时底层模型的温度，从0增加到0.5。这增加了模型的随机性，并且我们希望增加至少一个重试成功的可能性。

+   **升级：** 我们从廉价模型（Llama-3 8B）开始，如果遇到测试用例失败，则升级到更昂贵的模型（GPT-3.5、Llama-3 70B、GPT-4）。

令人惊讶的是，我们不知道有任何论文将他们提出的代理架构与后三种简单基准模型进行比较。

我们最引人注目的结果是，**HumanEval 的代理架构尽管成本更高，却不如我们更简单的基准模型表现出色**。事实上，这些代理在成本上有显著差异：对于准确度相近的情况，成本可能相差近两个数量级！然而，运行这些代理的成本并未在任何论文的主要度量标准中报告。

*我们的简单基线相对于现有的代理架构提供了巴雷托改进。我们每个代理运行五次，并报告在164个HumanEval问题上的平均准确性和平均总成本。对于LDB的结果，括号中有两个模型/代理，它们指示生成代码的语言模型或代理，接着是用于调试代码的语言模型。如果只有一个，它指示用于生成代码和调试代码的同一模型。请注意，y轴显示从0.7到1；具有完整轴（0到1）和误差条的数字，以及关于我们的实证结果的鲁棒性检查和其他详细信息，包含在附录中。*

我们的结果指出了另一个潜在问题：有关代理的实用性的论文到目前为止未能测试简单代理基线是否能够达到类似的准确性。这导致人工智能研究人员普遍认为，像规划、反思和调试这样的复杂思想是准确性提升的原因。事实上，Lipton 和 Steinhardt 注意到人工智能文献中一种趋势，即未能识别实证增益的来源。

因此，研究人员通常会选择巴雷托曲线的不同轴，例如参数数量。

根据我们的研究结果，关于调试、反思和其他“系统2”方法是否对代码生成有用的问题仍然存在。这可能对比HumanEval中所代表的更难的编程任务有所帮助。目前，关于系统2方法的过度乐观情绪加剧了我们在下文中报告的可再现性和标准化缺乏问题。

乍一看，报告美元成本令人震惊。它打破了我们视为理所当然的基准的许多特性：即测量不会随时间改变（而成本往往会下降），以及不同模型在一个公平竞争的场地上竞争（而一些开发者可能会从规模经济中受益，导致较低的推理成本）。

报告成本的缺点是真实存在的，但我们将在下文中描述如何减轻这些缺点。更重要的是，我们认为使用参数数量等属性作为成本的替代品是一个错误，并不能解决其本意的问题。要理解为什么，我们需要引入一个概念上的区别。

AI 评估至少有两个不同的目的。模型开发者和AI研究人员用它们来确定哪些训练数据和架构的变化可以提高准确性。我们称之为[model evaluation](https://arxiv.org/pdf/2211.09110)。而下游开发者，如使用AI构建面向消费者产品的程序员，则用评估来决定在其产品中使用哪些AI系统。我们称之为[downstream evaluation](https://www.arthur.ai/product/bench)。

模型评估和下游评估之间的差异被低估了。这导致了如何考虑AI运行成本的混淆。

模型评估是研究人员感兴趣的科学问题。因此，出于上述原因，避开货币成本是有道理的。相反，控制计算量是一个合理的方法：如果我们对用于训练模型的计算量进行归一化，我们就可以理解像架构变更或数据组成变更这样的因素是否负责改进，而不是更多的计算量。值得注意的是，Nathan Lambert [提出](https://www.interconnects.ai/p/compute-efficient-open-llms)，过去一年中的许多准确性提升（例如Meta的Llama 2）仅仅是使用更多计算资源的结果。

另一方面，下游评估是一个工程问题，有助于决策采购。在这里，成本是真正的关注点。成本测量的缺点实际上并不是缺点；它们恰恰是所需的。推理成本随时间降低，这对下游开发者至关重要。评估停留在时间上冻结是不必要且适得其反的。

在这种情况下，作为成本替代品的指标（如活跃参数数量或使用的计算量）是误导性的。例如，Mistral [发布](https://mistral.ai/news/mixtral-8x22b/)了下面的数据，说明为什么开发者应该选择其而不是竞争对手。

*将活跃参数作为成本的替代品是误导性的。来源：[Mistral](https://mistral.ai/news/mixtral-8x22b/)*

在这张图中，活跃参数的数量是成本的一个不良替代品。在[Anyscale](https://docs.endpoints.anyscale.com/pricing/)上，Mixtral 8x7B的成本是Llama 2 13B的两倍，但Mistral的图表显示它们的成本几乎相同，因为他们只考虑活跃参数的数量。当开发者在使用API时，并不关心活跃参数的数量，他们只关心成本与准确性的比较。Mistral选择了“活跃参数”作为替代指标，可能是因为这样可以使他们的模型看起来比Meta的Llama和Cohere的Command R+等密集模型更好。如果我们开始使用成本的替代指标，每个模型开发者都可以选择一个让他们的模型看起来更好的替代指标。

对成本评估的一些障碍仍然存在。不同的供应商可能对[同一款](https://docs.endpoints.anyscale.com/pricing/) [模型](https://www.together.ai/pricing) 收费不同，API调用的成本可能会[一夜之间](https://openai.com/blog/new-models-and-developer-products-announced-at-devday)改变，并且成本可能会根据模型开发者的决定而变化，比如[批量API](https://twitter.com/OpenAIDevs/status/1779922566091522492?ref_src=twsrc%5Etfw%7Ctwcamp%5Etweetembed%7Ctwterm%5E1779922566091522492%7Ctwgr%5E100b538f87ce81b828a23634fa12d127c126b5ba%7Ctwcon%5Es1_c10&ref_url=https%3A%2F%2Fwww.theverge.com%2F2024%2F4%2F15%2F24131401%2Fopenai-will-give-you-a-50-percent-discount-for-off-peak-gpt-use) 调用可能会有不同的收费方式。这些缺点部分可以通过使评估结果可定制来解决，使用调整运行模型成本的机制，即提供用户选择调整输入和输出代币成本的选项，以重新计算成本和准确性之间的权衡。反过来，代理的下游评估应该包括输入/输出代币计数以及美元成本，以便将来的任何人可以立即使用当前价格重新计算成本。

但最终，尽管有这些障碍，良好的测量需要建模于感兴趣的基础构造。对于下游评估来说，这个基础构造就是成本。所有其他的替代指标都是缺乏的。

在我们的评估过程中，我们发现代理评估的可复制性和标准化存在许多缺陷。

+   **我们无法重现LATS和LDB代理在HumanEval上的结果。** 特别是在LDB（Reflexion，GPT-3.5）的所有5次运行中，最大准确率为91.5%，远低于论文中报告的95.1%。 LATS在所有五次运行中的最大准确率同样较低，为91.5%，而不是94.4%。

+   类似地，LDB 论文中基准 GPT-4 模型的准确率远低于我们对论文代码的再现（75.0% vs. 五次运行的平均值 89.6%）。事实上，根据论文所述，GPT-3.5 和 GPT-4 模型表现非常相似（73.9% vs. 75.0%）。弱基线可能会让人误以为改进归因于代理架构的量是多少。

+   LATS 代理仅在 HumanEval 基准提供的[子集](https://github.com/andyz245/LanguageAgentTreeSearch/blob/554886901183a9908183d2cb104c3088c493650a/programming/mcts.py#L138C1-L138C29)上进行了评估。这夸大了它们的准确性数字，因为某个特定 HumanEval 问题的代码可能是不正确的，但如果它仅通过了该问题的部分测试用例，则仍然可以标记为正确。在我们的分析中，这导致了准确率（五次运行的平均值）的差异达到了 3%，这解释了我们发现的准确率与论文报告之间差异的一个重要部分。此外，论文或 GitHub 存储库中未报告的许多实现细节，如超参数值，请参见[附录](https://www.cs.princeton.edu/~sayashk/papers/ai-leaderboards-appendix.pdf)获取详细信息。

+   据我们所知，这篇文章是首次在 HumanEval 上测试准确率最高的四个代理（Retry、Warming、LDB（GPT-4）和 LDB（GPT-4 + Reflexion））。

+   **Reflexion、LDB 和 LATS 都使用 HumanEval 的不同子集。** 在原始版本的 HumanEval 中，有三个（共 164 个）编码问题缺乏示例测试。由于这些代理需要示例测试来调试或重新运行它们的解决方案，Reflexion 移除了那三个缺少示例测试的问题。LATS 移除了这三个问题，再加上另一个问题，理由未报告。LDB 为原始基准中缺失的三个问题添加了示例测试。**这三篇论文中没有一篇报告了这一点。** LATS 论文错误地声称：“我们在实验中使用了所有 164 个问题。” 在我们的分析中，我们对 LDB 提供的基准版本进行了所有评估，因为它为所有问题提供了示例测试。

+   LDB 论文声称使用 GPT-3.5 生成代码使用 Reflexion：“对于 Reflexion，我们选择基于 GPT-3.5 的版本，并利用官方 Github 存储库中发布的相应生成的程序。”然而，他们从 Reflexion 存储库中使用的[生成程序](https://github.com/noahshinn/reflexion/blob/d15acda1c81d464d9a81648d7f29fb951e326c70/programming_runs/root/reflexion_humaneval_py_pass_at_1/reflexion_humaneval_py_pass_at_1.jsonl)依赖于 GPT-4 进行代码生成，而不是 GPT-3.5。

这些经验结果中的缺陷也导致了广泛讨论中对AI代理准确性的解读错误。例如，安德鲁·吴最近的[文章](https://www.deeplearning.ai/the-batch/issue-241/)声称使用GPT-3.5的代理可以胜过GPT-4。特别是，他声称：

> *[对于HumanEval，] GPT-3.5（零样本）准确率为48.1%。GPT-4（零样本）则提升至67.0%。然而，通过融合迭代代理工作流程来实现GPT-3.5到GPT-4的改进效果，确实让人惊叹。确实，包裹在代理循环中，GPT-3.5 的准确率可高达95.1%。*

尽管这一声明引起了很多关注，但是这是不正确的。该声明（“GPT-3.5包裹在代理工作流中可达到95.1%的准确率”）似乎是关于LDB代理的。HumanEval的[Papers With Code排行榜](https://paperswithcode.com/sota/code-generation-on-humaneval)也做出了同样的声明。然而，正如我们之前讨论的，对于LDB，仅使用GPT-3.5来查找错误。代码生成使用的是GPT-4（或使用GPT-4的反思代理）。不幸的是，论文中的错误导致了广泛的AI社区对代理的过度乐观。

吴的文章还犯了常见的错误，即重复论文中的结果而没有验证它们或考虑到提示和模型版本的变化。例如，GPT-3.5（48.1%）和GPT-4（67.0%）的零样本准确率数字似乎是从2023年3月的[GPT-4技术报告](https://arxiv.org/abs/2303.08774)复制而来。然而，自发布以来，模型已经更新多次。确实，在我们的比较中，当我们使用LDB论文提供的提示时，基础模型的表现要比吴的文章中声称的数字好得多（GPT-3.5：73.9%，GPT-4：89.6%）。因此，这篇文章大大高估了可以归因于代理架构改进的提升。

评估框架如斯坦福大学的[HELM](https://crfm.stanford.edu/helm/lite/latest/)和EleutherAI的[LM评估工具包](https://www.eleuther.ai/projects/large-language-model-evaluation)试图修复模型评估中类似的缺陷，通过提供标准化的评估结果。我们正在努力解决代理评估的标准化和可重现性问题，特别是从代理评估的下游视角来看。

最后，下游开发人员应牢记，HumanEval或任何其他标准化基准只是特定下游应用中出现任务的粗略代理。要理解代理在实践中的表现，有必要在来自感兴趣领域的[自定义数据集](https://arxiv.org/abs/2404.12272)上评估它们 — 或者更好地，在生产环境中对不同代理进行A/B测试。

+   [Zaharia等人](https://bair.berkeley.edu/blog/2024/02/18/compound-ai-systems/)观察到，AI基准测试中最先进的准确性通常由复合系统实现。如果代理的采用继续，将更有必要将成本和准确性可视化为帕累托曲线。

+   [Santhanam等人](https://arxiv.org/pdf/2212.01340.pdf)指出，在信息检索基准评估中评估成本与准确性同样重要。

+   [Ozrmazabal等人](https://arxiv.org/html/2404.12387v1)强调了在MMLU上各种模型（但不是代理）的[准确性与每个输出令牌成本的权衡](https://arxiv.org/html/2404.12387v1#:~:text=Figure%201%3A,different%20LLM%20APIs.)。尽管输出令牌的成本可能不是总体成本的良好指标，考虑到不同模型的输入令牌成本以及输出长度的差异，报告这些权衡总比不报告好。

+   [伯克利函数调用排行榜](https://gorilla.cs.berkeley.edu/leaderboard.html)包括语言模型在函数调用评估中的各种指标，包括成本和延迟。

+   [Xie等人](https://arxiv.org/pdf/2404.07972.pdf)开发了OSWorld，一个用于评估计算机环境中代理的基准。在他们的[GitHub存储库](https://github.com/xlang-ai/OSWorld)中（尽管不在论文中），他们提供了在其基准上运行各种多模态代理的大致成本估算。

+   不出所料，成本与准确性的权衡主要来自[下游开发者](https://x.com/swyx/status/1772799201023557697)，他们[使用AI](https://x.com/jaredpalmer/status/1783899239140986884)。

+   在先前的讲话中，我们讨论了LLM评估中的[三个主要陷阱](https://www.cs.princeton.edu/~arvindn/talks/evaluating_llms_minefield/)：提示敏感性、构造有效性和污染。当前的研究主要与此无关：提示敏感性对于代理评估不是问题（因为允许代理定义自己的提示）；下游开发者可以通过在定制数据集上进行评估来解决污染和构造有效性问题。

重现我们分析的代码可在[这里](https://github.com/benediktstroebl/agent-eval)找到。[附录](https://www.cs.princeton.edu/~sayashk/papers/ai-leaderboards-appendix.pdf)包含了我们设置和结果的更多细节。

我们感谢Rishi Bommasani、Rumman Chowdhury、Percy Liang、Shayne Longpre、Yifan Mai、Nitya Nadgir、Matt Salganik、Hailey Schoelkopf、Zachary Siegel和Venia Veselovsky参与讨论并提供了对我们分析有帮助的意见。我们感谢Cunxiang Wang和Ruoxi Ning对我们关于NovelQA基准的问题的及时回应。

我们对本文中所涉及论文的作者表示感谢，感谢他们快速的回应并分享他们的代码，这些使得首次可能进行此类再现分析。特别感谢王子龙（LDB）、周安迪（LATS）和卡尔希克·纳拉西姆汉（Reflexion），他们在本博客文章初稿中给予了反馈。
