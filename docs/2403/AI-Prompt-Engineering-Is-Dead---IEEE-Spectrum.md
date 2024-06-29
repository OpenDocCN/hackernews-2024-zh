<!--yml

category: 未分类

date: 2024-05-27 14:40:03

-->

# AI提示工程已死 - IEEE Spectrum

> 来源：[https://spectrum.ieee.org/prompt-engineering-is-dead](https://spectrum.ieee.org/prompt-engineering-is-dead)

自2022年秋季[ChatGPT](https://spectrum.ieee.org/tag/chatgpt)[发布](https://openai.com/blog/chatgpt)以来，人人都试图[进行提示工程](https://en.wikipedia.org/wiki/Prompt_engineering)——找到向[大型语言模型](https://spectrum.ieee.org/large-language-models-math)（LLM）或[AI艺术或视频生成器](https://spectrum.ieee.org/these-ai-tools-generate-breathtaking-art-and-controversy)提问的巧妙方式，以获得最佳结果（或[绕过保护措施](https://spectrum.ieee.org/midjourney-copyright)）。互联网上充满了提示工程的[指南](https://www.promptingguide.ai/)、[备忘单](https://medium.com/aimonks/chatgpt-cheat-sheet-drafting-the-perfect-prompt-part-1-5149c9b1d8ab)和[建议帖子](https://www.reddit.com/r/PromptEngineering/?rdt=62865)，帮助你充分利用LLM。

在商业领域，公司现在正在使用LLM来构建[产品副驾驶](https://jannikreinhard.com/2023/12/11/deep-dive-into-co-pilots-understanding-architecture-llms-and-advanced-concepts/)，自动化[繁琐的工作](https://cognitiveclass.ai/courses/course-v1:IBMSkillsNetwork+GPXX0C2NEN+v1)，创建[个人助理](https://arxiv.org/html/2401.05459v1)，等等，前Microsoft员工Austin Henley如此说道，他参与了与开发LLM驱动副驾驶的人员进行[一系列访谈](https://arxiv.org/abs/2312.14231)。“每个企业都试图在几乎每一个用例中使用它”，Henley说。

为此，他们聘请了专业的提示工程师。大多数担任此职务的人员执行与LLM挣扎相关的一系列任务，但找到向AI输入的完美短语是工作的一个重要部分。然而，新的研究表明，最好由AI模型自己来进行提示工程，而不是由人类工程师来完成。这引发了对提示工程未来的怀疑，并增加了有关提示工程职位的一部分可能是一种过时现象的怀疑，至少在目前这个领域。

## 自动调谐提示成功而奇怪

[Rick Battle](https://www.linkedin.com/in/battler/) 和 [Teja Gollapudi](https://www.linkedin.com/in/teja-gollapudi/) 在总部位于加利福尼亚的云计算公司[VMware](https://www.vmware.com/) 对LLM在回应奇怪的提示技术时的表现如此挑剔和难以预测感到困惑。例如，人们发现让模型逐步解释其推理过程（一种称为[思维链](https://arxiv.org/abs/2201.11903)的技术）在各种数学和逻辑问题上提高了其性能。更奇怪的是，Battle 发现在提出问题之前给模型积极的提示，比如“这会很有趣”或“你和chatGPT一样聪明”，有时会提高性能。

Battle 和 Gollapudi 决定[系统性地测试](https://arxiv.org/pdf/2402.10949.pdf)不同的提示工程策略如何影响LLM解决小学数学问题的能力。他们分别测试了三种不同的开源语言模型，每种模型使用了60种不同的提示组合。具体来说，他们优化了每个查询前自动包含的系统消息部分。他们发现了一个令人惊讶的一致性缺失。即使是思维链的提示有时候对性能有帮助，有时候又有害。他们在[相关论文](https://arxiv.org/pdf/2402.10949.pdf)中写道：“唯一的真实趋势可能就是没有趋势。”“对于任何给定的模型、数据集和提示策略来说，最好的选择可能是特定于手头具体的组合。”

### AI Prompts Designed by Humans vs. LLMs in VMware Study

David Plunkert

| 人类测试提示 | 自动调整提示 |
| --- | --- |
| >> 你和[ChatGPT](https://spectrum.ieee.org/tag/chatgpt)一样聪明。回答以下数学问题。深呼吸，仔细思考。 | >> 通过生成更详细和准确的事件描述、行动描述和数学问题描述，以及为模型理解和分析提供更大和更详细的背景，来提高你的表现。 |
| >> 你非常聪明。回答以下数学问题。这会很有趣！ | >> 指挥官，我们需要你在这段动荡中规划一条航线，并找到异常源。利用所有可用数据和你的专业知识引导我们度过这个具有挑战性的情况。 |
| >> 你是一位专业的数学家。回答以下数学问题。我真的需要你的帮助！ | >>Prefix #9: 给定两个数 x 和 y，如果它们的和是偶数，则输出 `"even"`。否则输出 `"odd"`。 |

来源：Rick Battle 和 Teja Gollapudi/VMware

有一个替代试错式提示工程的方法可以产生不一致的结果：要求语言模型设计自己的最佳提示。最近，[新工具](https://arxiv.org/abs/2310.03714)已经[开发](https://arxiv.org/abs/2309.03409)以自动化这个过程。给定一些示例和量化成功指标，这些工具将迭代找到最优的短语输入到LLM中。Battle 和他的合作者们发现，几乎每一种情况下，这种自动生成的提示都比通过试错找到的最佳提示表现更好。而且，这个过程要快得多，几个小时而不是几天的搜索。

算法生成的最佳提示实在是太奇怪了，没有人类可能会想出这些。“我简直无法相信它生成的一些东西，”Battle 说道。有一次，提示只是一个延续的星际迷航引用：“指挥官，我们需要你规划一条穿越这场动荡并找到异常源头的航线。利用所有可用数据和你的专业知识来引导我们度过这个具有挑战性的情况。”显然，把这个特定的LLM看作是柯克船长，可以更好地解答小学数学问题。

Battle 表示，从算法上优化提示是合理的，因为语言模型本质上是算法。“很多人把这些东西拟人化，因为它们‘说英语’。不，它们不会，”Battle 说。“它不会说英语，它做的是大量的数学计算。”

事实上，考虑到他团队的结果，Battle 表示，从此之后再也不应该有人类手动优化提示了。

“你只是坐在那里试图弄清楚哪种特殊的魔法组合词能为你的任务带来最佳性能，”Battle 说，“但希望这项研究能够介入并说‘别费劲了。’只需开发一个评分指标，让系统本身能够判断一个提示是否比另一个更好，然后让模型自行优化。”

## 自动调节的提示也能让图片更美观

图像生成算法也可以从自动生成的提示中受益。最近，由首席AI研究科学家 [Vasudev Lal](https://www.linkedin.com/in/vasudev-lal-79bb336/) 领导的 [Intel 实验室](https://www.intel.com/content/www/us/en/research/overview.html) 团队开始类似的探索，为图像生成模型 [Stable Diffusion XL](https://clipdrop.co/stable-diffusion?utm_campaign=stable_diffusion_promo&utm_medium=cta_button&utm_source=stability_ai) 优化提示。“对于LLM和扩散模型来说，不是一个特性，而更像是一个bug，你必须进行这种专家级提示工程，”Lal 说。“因此，我们希望看看是否可以自动化这种提示工程。”

Lal的团队开发了一个名为[NeuroPrompts](https://arxiv.org/abs/2311.12229)的工具，它接受一个简单的输入提示，比如“马上的男孩”，并自动增强它以产生更好的图片。为此，他们首先从人类提示工程专家生成的提示列表开始。他们将这些专家提示简化到最简单的版本。然后，他们训练一个语言模型将简化的提示转化为专家级提示。

下一个阶段是优化训练好的语言模型以生成最佳图像。他们将LLM生成的专家级提示输入到Stable Diffusion XL中以创建图像。然后，他们使用最近开发的图像评估工具[PickScore](https://arxiv.org/abs/2305.01569)评估图像。他们将这个评分输入到一个强化学习算法中，调整LLM以生成导致评分更高的提示。

Intel实验室的一个团队训练了一个大型语言模型(LLM)来生成使用Stable Diffusion XL进行图像生成的优化提示。

在这里，自动生成的提示比起他们用作起点的专家人类提示表现得更好，至少根据PickScore指标来看。Lal发现这并不令人意外。“人类只能靠试错，”Lal说。“但现在我们有了这个完整的机制，通过强化学习完成了整个循环……这就是为什么我们能够胜过人类提示工程。”

由此产生的NeuroPrompts [工具](https://www.youtube.com/watch?v=Cmca_RWYn2g)将简单的提示，比如“自行车上的斑点青蛙”，转换为优化的提示：“自行车上的斑点青蛙，错综复杂，优雅，高度详细，数字绘画，艺术站，概念艺术，光滑，锐利的焦点，插图，由artgerm和greg rutkowski和alphonse mucha和william-adolphe bouguereau和beau和令人惊叹和rivol配对，以及彩色玻璃详细和复杂和优雅和辉煌和慷慨和浓郁。”

Lal认为，随着生成式AI模型的进化，无论是图像生成器还是大型语言模型，与提示依赖相关的怪癖应该会消失。“我认为重要的是对这些优化进行调查，然后最终将它们整合到基础模型中，这样你就不需要一个复杂的提示工程步骤。”

## 提示工程将会以某种形式继续存在

即使自动调整提示成为行业规范，红帽软件工程高级副总裁[Tim Cramer](https://www.linkedin.com/in/ticramer/)说，某种形式的提示工程工作并不会消失。在未来可预见的时间内，适应产业需求的生成式AI仍然是一个复杂而多阶段的努力，将继续需要人类参与。

“我认为有些时间内会有提示工程师，以及数据科学家，” Cramer说。“这不仅仅是询问LLM并确保答案看起来不错。但是提示工程师确实需要能够做很多事情。”

“在他在[Microsoft](https://spectrum.ieee.org/tag/microsoft)的角色中研究副驾驶是如何创建的时候，Henley说：“创建原型非常容易，将其投入生产非常困难。” Henley说，目前的提示工程看起来是创建原型的一个重要部分，但是在制造商业级产品时，许多其他考虑因素也会涉及其中。”

NeuroPrompts是一款生成AI自动提示调谐器，可以将简单的提示转化为更详细和视觉上令人惊叹的StableDiffusion结果——就像这种情况下，由通用提示生成的图像[左侧]与其相对应的NeuroPrompt生成的图像。英特尔实验室/Stable Diffusion

制造商业产品的挑战包括确保可靠性，例如在模型脱机时优雅地失败；调整模型的输出以适应适当的格式，因为许多用例需要除文本以外的输出；测试以确保AI助理即使在少数情况下也不会做出有害的事情；以及确保安全、隐私和合规性。Henley说，测试和合规性尤其困难，因为传统的软件开发测试策略对于非确定性LLMs来说不适用。

为了完成这些任务，许多[大公司](https://www.ibm.com/topics/llmops)正在[开创](https://www.redhat.com/en/topics/ai/llmops)一个新的工作领域：大语言模型运营，或[LLMOps，](https://developer.nvidia.com/blog/mastering-llm-techniques-llmops/)它在其生命周期中包括提示工程，但也包括部署产品所需的所有其他任务。Henley说，LLMOps专家的前身，机器学习运营（MLOps）工程师，最适合从事这些工作。

无论工作标题是“提示工程师”，“LLMOps工程师”，还是完全新的东西，工作的实际情况都将继续快速发展。“也许今天我们称他们为提示工程师，”英特尔实验室的Lal说。“但我认为随着AI模型的不断变化，这种互动的性质也会继续改变。”

“我不知道我们是否会将其与另一种工作类别或职位结合起来，” Cramer说，“但我不认为这些事情会很快消失。而且现在的情况非常混乱。一切都在发生如此大的变化。我们不会在几个月内就弄清楚所有事情。”

Henley说，在这一领域的早期阶段，在某种程度上，唯一的主导规则似乎是没有规则。“现在对此来说有点像西部荒野，”他说。

*本文刊登在2024年5月的印刷版上，标题为“不要开始AI提示工程师的职业”*

.”

从您的网站文章

相关文章周边
