<!--yml

category: 未分类

date: 2024-05-27 14:32:35

-->

# FOSDEM 2024上的开源人工智能 [LWN.net]

> 来源：[https://lwn.net/Articles/961868/](https://lwn.net/Articles/961868/)

| **请考虑订阅LWN**订阅是LWN.net的生命线。如果您喜欢这篇内容并希望看到更多类似内容，您的订阅将有助于确保LWN继续发展。请访问[此页面](/subscribe/)以加入并支持LWN的持续运营。 |
| --- |

February 15, 2024

本文由Koen Vervloesem贡献

* * *

[FOSDEM](/Archives/ConferenceByYear/#2024-FOSDEM)

在布鲁塞尔举办的[FOSDEM 2024](https://fosdem.org/2024/)上，[人工智能与机器学习开发室](https://fosdem.org/2024/schedule/track/ai_ml/)举办了多场关于开源人工智能模型的讲座。讨论了开源人工智能的定义，许可证中的“伦理”限制，以及开放数据集的重要性，特别是对非英语语言的影响。该开发室为领域当前状况提供了概述。

人工智能模型是一种程序，经过数据集训练以识别模式、模仿学习数据并在输出中做出某些类型的自主决策。尤其是大型语言模型（LLM），这些广泛的神经网络能够生成类似人类文本，成为FOSDEM的重要议题。此报告基于讲座的直播内容，由于流感，今年我不幸未能亲临FOSDEM。

典型地，一个LLM包含多达数千亿的“权重”，这些浮点数也称为“参数”。开发大型语言模型的公司不倾向于将其模型和运行所需的代码作为开源释放，因为训练这些模型需要大量的计算资源和财务投入。然而，这并不妨碍各种组织开发开源LLM。去年，LWN曾审视过[开源语言模型](/Articles/931853/)。

#### License restrictions

尼哈里卡·辛格尔，[自由软件基金会欧洲](https://fsfe.org)（FSFE）项目经理，谈到了通过许可证对AI模型施加道德限制的趋势。辛格尔提供了几个类似的限制实例，涉及到行业、行为或商业实践。其中一个是[Hippocratic License](https://firstdonoharm.dev)，限制许可证持有人执行许多被认为有害的行为，基于各种“《国际协议和权威关于基本人权规范的协议》”。还有[Llama 2 v2使用政策](https://ai.meta.com/llama/use-policy/)，禁止将LLM用于暴力或恐怖活动，以及“任何其他犯罪活动”。同样，BigScience的[OpenRAIL-M许可证](https://bigscience.huggingface.co/blog/bigscience-openrail-m)对将模型用于各种有害活动施加了限制。

根据辛格尔的说法，这些额外的限制有严重的影响：“它们在使用和再利用模型方面设立了障碍，这也使得调整和改进模型变得更加困难。”她认为，为了保持AI的“开放性”，AI模型的许可证必须与自由软件许可证兼容，而这些限制却并非如此。她总结道，许可证不能替代法规：“遵守道德规则的限制性做法不应该出现在许可证中：这些属于法规的范畴。”

#### 开源AI的定义

[开源倡议](https://opensource.org)（OSI）的执行董事斯特凡诺·马富利（Stefano Maffulli）描述了OSI定义开源AI的努力。2022年，OSI开始联系研究人员、其他“开放”组织、技术公司和民权组织，询问他们对开源AI系统的想法。

作为一般原则，马富利认为[GNU宣言](https://www.gnu.org/gnu/manifesto.html)的黄金法则应适用于AI：“如果我喜欢一个AI系统，我必须可以自由地与其他人分享它。”要将AI系统归类为开源，它需要授予我们适用于开源软件的四个基本自由的自由：使用、研究、修改和分享。

> 我们需要能够出于任何目的使用系统，而无需请求许可。我们需要能够研究系统的工作方式并检查其组件。我们需要能够修改系统以更改其建议、预测或决策，以适应我们的需求。我们需要能够分享带或不带修改的系统，以供任何目的使用。

根据马富里的说法，在这种情况下提出一个相关的问题是："什么是修改AI系统的首选形式？"为了得到这个问题的答案，OSI已经创建了小型工作组来分析一些流行的AI系统。"我们将从[Llama 2](https://llama.meta.com)和[Pythia](https://www.eleuther.ai/papers-blog/pythia-a-suite-for-analyzing-large-language-modelsacross-training-and-scaling)开始，这两个LLM。之后，我们将重复相同的练习，涉及[BLOOM](https://bigscience.huggingface.co/blog/bloom)、[OpenCV](https://opencv.org)、[Mistral](https://docs.mistral.ai)、[Phi-2](https://www.microsoft.com/en-us/research/blog/phi-2-the-surprising-power-of-small-language-models/)和[OLMo](https://allenai.org/olmo)。对于每个AI系统，工作组将确定确保四个基本自由的要求。例如，理解为什么在给定输入时会得到特定输出，是研究AI系统的必要条件。

2024年，OSI将根据每两周一次的虚拟公开市民厅会议发布新的[开源AI定义草案](https://opensource.org/deepdive/drafts/)。"我们的目标是在十月底之前发布1.0版"，马富里说道。欢迎所有人参与OSI的[公共论坛](https://discuss.opensource.org)上有关草案的讨论。

根据马富里的说法，涉及开源人工智能时，不能存在模糊地带：AI系统要么是开源的，要么不是。然而，在大型语言模型领域内，许多参与者滥用"开源"这一术语。例如，最受欢迎的"开源"LLM之一是Meta的Llama 2。去年，Meta的杨·勒孔在Twitter上[宣布这一模型](https://twitter.com/ylecun/status/1681336284453781505)时写道："\<q>这是个大事：Llama-v2是开源的，其授权许可允许商业使用！\</q>"。然而，[Llama 2的许可证](https://github.com/facebookresearch/llama/blob/main/LICENSE)对其商业使用施加了基于活跃用户数量的限制。该许可证还禁止使用Meta的模型改进其他LLM。这两个限制与OSI的开源定义不符。

#### 开放数据集

法国软件公司[Linagora](https://www.linagora.com)的研究工程师Julie Hunter讨论了[构建开源语言模型](https://fosdem.org/2024/schedule/event/fosdem-2024-2591-building-open-source-language-models/)。根据Hunter的说法，Meta开发的LLMs，以及[MosaicML](https://www.mosaicml.com)和[Technology Innovation Institute](https://www.tii.ae/)的Falcon模型，被称为“开放权重模型”：神经网络的权重是公开的。这使得可以选择如何运行模型，并且可以通过调整权重进行额外训练来微调模型。然而，权重并不能解释为什么某些情况有效或无效。“没有访问模型训练数据的权限，这将导致很多只能靠猜测”，Hunter表示。

近年来推动了开放训练数据的发展，因此很多数据集被添加到了像[Hugging Face](https://huggingface.co)运营的网站上。“任何人都可以在这些数据集上训练他们的新LLM”，Hunter说。“然而，许多这些数据集存在一些问题。它们通常是从网上抓取的，包含个人信息、毒性语言和低质量的句子。此外，它们主要是英文。”

[OpenLLM France](https://www.openllm-france.fr) 联盟旨在为法语构建开源AI模型和技术。对于其首个模型Claire，主要目标是创建一个带有可追溯许可的法语数据集。[Claire French Dialogue Dataset](https://arxiv.org/abs/2311.16840)是一个包含来自法语戏剧剧本、议会讨论等文字的语料库，总词量达到1.4亿。这个数据集[Claire-Dialogue-French-0.1](https://huggingface.co/datasets/OpenLLM-France/Claire-Dialogue-French-0.1)，主要使用[知识共享署名-非商业性使用-相同方式共享4.0国际许可](https://creativecommons.org/licenses/by-nc-sa/4.0/)，部分文字使用其他（可追溯）许可。

这个数据集被用来调整一个开放权重模型，[Falcon-7B](https://huggingface.co/tiiuae/falcon-7b)。“这种方法的主要目的是评估一个好的数据集对模型性能的影响”，Hunter说道。Linagora的总经理Michel-Marie Maudet补充道，公司基于微软研究的论文“[仅需教科书](https://arxiv.org/abs/2306.11644)”的想法，开发了基于小而高质量数据语料库的语言模型。他继续说道：

> 数据集的质量比数量更为重要。一个小而高质量的语料库将会产生一个紧凑、专业化的模型，能够更好地控制其响应的可解释性和可靠性。这同时也加快了训练速度，使得我们可以持续进行更新。

在 2023 年 10 月，模型 [Claire-7B-0.1](https://huggingface.co/OpenLLM-France/Claire-7B-0.1) 在 Hugging Face 上发布。模型的 [训练代码](https://github.com/OpenLLM-France/Lit-Claire) 也已经在 AGPLv3 下公开。

#### 超越英语

OpenLLM France 目前正在开发一个完全开源的语言模型 Lucie，计划于 2024 年 4 月发布。Maudet 解释道：“这个模型是使用 100% 开源的法语、英语、德语、西班牙语和意大利语文本数据集进行训练的，还包括一些计算机代码。” 数据集包括法国国家图书馆的档案和开放获取的学术出版物。

Maudet 的 [演讲](https://fosdem.org/2024/schedule/event/fosdem-2024-2629-from-openllm-france-to-openllm-europe-paving-the-way-to-sovereign-and-open-source-ai/) 揭示了关于 OpenLLM France 及其使命的一些细节。这个社区始于 2023 年 7 月，拥有超过 450 位活跃成员，包括学术机构和公司。为什么需要一个以法国为重点的 LLM 联盟？Maudet 解释道，自 2018 年以来，超过十亿参数的 LLM 的地理分布显示，近 70% 在北美制造，而仅有 7.5% 在欧洲。分析 Llama 2 的语言分布数据更加令人沮丧：“尽管英语占据了数据的将近 90%，但德语和法语等欧洲语言仅占数据的 0.17% 和 0.16%。” 由于欧洲语言在数据集中的代表性不足，像 Llama 2 这样的模型在这些语言中表现不佳。

欧洲其他地区也有类似的倡议，致力于构建欧洲语言的开源 LLM，例如德国的 [LAION](https://laion.ai) 和 [openGPT-X](https://opengpt-x.de/en/)，以及意大利的 [Fauno](https://github.com/RSTLess-research/Fauno-Italian-LLM)。在 FOSDEM 上，Maudet 宣布 OpenLLM France 正在更名为 [OpenLLM Europe](http://openllm-europe.org)（尽管该网站尚未启用）。“我们的使命是为每一种欧洲语言开发一个开源 LLM。”

#### 结论

组织将他们的 AI 系统称为“开源”，即使其许可证与四个基本自由相矛盾，这表明我们确实需要对开源 AI 进行明确定义。希望到 2024 年底 OSI 的定义能够帮助阻止带有各种良意但有害的伦理限制的许可证的传播。此外，对于像 OpenLLM Europe 这样的联盟来说，吸引足够的成员来构建强大的非英语开源 LLM 将是有益的。

* * *

（

[登录](https://lwn.net/Login/?target=/Articles/961868/)

以发布评论）
