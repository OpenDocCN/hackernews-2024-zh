<!--yml

category: 未分类

date: 2024-05-29 13:29:53

-->

# StarCoder 2是一款在大多数GPU上运行的代码生成AI | TechCrunch

> 来源：[https://techcrunch.com/2024/02/28/starcoder-2-is-a-code-generating-ai-that-runs-on-most-gpus/](https://techcrunch.com/2024/02/28/starcoder-2-is-a-code-generating-ai-that-runs-on-most-gpus/)

开发人员正在采用由AI驱动的代码生成器——例如[GitHub Copilot](https://techcrunch.com/tag/github-copilot/)和[Amazon CodeWhisperer](https://techcrunch.com/2022/06/23/amazon-launches-codewhisperer-its-ai-pair-programming-tool/)，以及Meta的开放访问模型[Code Llama](https://techcrunch.com/2023/08/24/meta-releases-code-llama-a-code-generating-ai-model/)——以惊人的速度。但是这些工具还远非理想。许多是收费的。其他一些是免费的，但只能在排除在普通商业环境中使用的许可证下使用。

针对市场需求，几年前AI初创公司Hugging Face与工作流自动化平台ServiceNow合作创建了[StarCoder](https://techcrunch.com/2023/05/04/hugging-face-and-servicenow-release-a-free-code-generating-model/)，这是一个开源代码生成器，其许可证比其他一些存在的模型更少限制性。原始版本于去年初上线，此后一直在开发其后续产品StarCoder 2。

StarCoder 2并不是一个单一的代码生成模型，而是一个系列。今天发布，它有三个变体，其中前两个可以在大多数现代消费级GPU上运行：

+   ServiceNow培训的一个30亿参数（3B）模型

+   Hugging Face培训的70亿参数（7B）模型

+   Nvidia培训的一个150亿参数（15B）模型，是StarCoder项目的最新支持者

（请注意，“参数”是模型从训练数据中学到的部分，基本上定义了模型在解决问题时的技能，本例中是生成代码。）

与大多数其他代码生成器一样，StarCoder 2可以建议完成未完成的代码行的方法，以及在自然语言中被询问时总结和检索代码片段。与原始StarCoder相比，使用的数据量增加了4倍（67.5TB对比6.4TB），StarCoder 2以较低的运营成本提供了Hugging Face、ServiceNow和Nvidia称之为“显著”改进的性能。

StarCoder 2可以使用像Nvidia A100这样的GPU“在几小时内”进行微调，以创建诸如聊天机器人和个人编码助手之类的应用程序。并且，由于它是在比原始StarCoder更大更多样化的数据集上训练的（大约619种编程语言），StarCoder 2可以做出更准确、具有上下文意识的预测——至少在假设情况下。

“StarCoder 2 是专门为需要快速构建应用程序的开发人员而创建的，” ServiceNow 的StarCoder 2 开发团队负责人Harm de Vries 在接受 TechCrunch 采访时表示。“开发者可以利用StarCoder 2 的能力使编码更加高效，而不会牺牲速度或质量。”

现在，我敢说并不是每个开发者都同意德弗里斯关于速度和质量方面的观点。代码生成器承诺简化某些编码任务——但是代价是什么？

最近一项斯坦福的[研究](https://techcrunch.com/2022/12/28/code-generating-ai-can-introduce-security-vulnerabilities-study-finds/)发现，使用代码生成系统的工程师更有可能在他们开发的应用程序中引入安全漏洞。另外，来自网络安全公司Sonatype的[调查](https://www.sonatype.com/hubfs/The%20Risks%20and%20Rewards%20of%20Generative%20AI%20in%20Software%20Development.pdf)显示，大多数开发人员对于代码生成器生成的代码的生产方式缺乏了解和代码泛滥问题感到担忧。

StarCoder 2 的许可证可能对某些人构成障碍。

StarCoder 2 的许可证采用了 BigCode Open RAIL-M 1.0，旨在通过对模型许可人和下游用户施加“轻触”限制来促进负责任的使用。虽然不像许多其他许可证那样限制严格，RAIL-M 在某种程度上并不真正“开放”，因为它不允许[开发者](https://about.gitlab.com/blog/2023/07/25/rail-m-is-an-imperfectly-good-start-for-ai-model-licenses/)将 StarCoder 2 用于*每一种*可能的应用（例如医疗建议应用严格禁止）。一些评论员表示，RAIL-M 的要求可能过于模糊，以至于无法有效遵守，而且RAIL-M可能会与欧盟的AI法案等相关法规产生冲突。

针对上述批评，Hugging Face 的一位发言人通过电子邮件声明说：“许可证经过精心设计，旨在最大限度地遵守当前的法律和法规。”

暂且不谈这些，StarCoder 2 是否真的比其他现有的代码生成器——无论是免费还是付费的——更优秀呢？

根据基准测试，StarCoder 2 似乎比 Code Llama 的某个版本，Code Llama 33B，更高效。Hugging Face 表示，StarCoder 2 15B 在某些代码完成任务的子集上与 Code Llama 33B 的速度相当，但快了两倍。不清楚是哪些任务；Hugging Face 没有具体说明。

StarCoder 2作为一个开源模型集合，还有一个优势，即可以在本地部署并“学习”开发人员的源代码或代码库——这对于对暴露代码给云托管AI感到担忧的开发人员和公司来说是一个有吸引力的前景。在2023年的一项[调查](https://chainstoreage.com/survey-companies-have-mixed-feelings-about-generative-ai)中，来自Portal26和CensusWide的85%的企业表示，由于隐私和安全风险（例如员工分享敏感信息或供应商在专有数据上进行训练），他们对采用类似代码生成器的GenAI感到谨慎。

Hugging Face、ServiceNow和Nvidia还表明，StarCoder 2比其竞争对手更加道德化，并且在法律上的风险较低。

所有GenAI模型都会复制——换句话说，它们会输出它们所训练的数据的镜像副本。可以想象为什么这可能会让开发人员陷入麻烦。尽管有过滤器和其他额外的保障措施，但在受版权保护的代码上训练的生成器可能会无意中推荐受版权保护的代码，并且未能标记它们。

包括GitHub、微软（GitHub的母公司）和亚马逊在内的一些供应商已经[承诺](https://techcrunch.com/2023/10/06/some-gen-ai-vendors-say-theyll-defend-customers-from-ip-lawsuits-others-not-so-much/)在代码生成器客户被指控侵犯版权的情况下提供法律保护。但不同供应商之间的覆盖范围各不相同，并且通常仅限于企业客户。

与依靠受版权保护的代码（例如GitHub Copilot等）训练的代码生成器不同，StarCoder 2仅使用了来自软件遗产（提供代码归档服务的非营利组织）许可的数据进行训练。在StarCoder 2的训练之前，负责StarCoder 2路线图大部分的跨组织团队[BigCode](https://techcrunch.com/2022/09/26/hugging-face-and-servicenow-launch-bigcode-a-project-to-open-source-code-generating-ai-systems/)给予了代码所有者选择退出训练集的机会。

与原始的StarCoder一样，StarCoder 2的训练数据对开发人员来说是可以随意分叉、复制或审计的。

Hugging Face的机器学习工程师兼BigCode的联合主管Leandro von Werra指出，尽管最近出现了许多开放式代码生成器，但很少有伴随其背后训练数据以及它们如何训练的信息。

“从科学的角度来看，一个问题是训练结果不可复制，但也作为数据生产者（例如将其代码上传到GitHub的人），你不知道你的数据是否被使用以及如何使用”，von Werra在一次采访中表示。“StarCoder 2通过在整个从预训练数据抓取到训练本身的整个过程中都是完全透明的方式来解决这个问题”。

话虽如此，StarCoder 2 并非完美。正如其他代码生成器一样，它容易受到偏见的影响。德·弗里斯指出，它可能生成反映有关性别和种族刻板印象的代码元素。由于 StarCoder 2 主要训练于英语评论、Python 和 Java 代码，因此在其他语言和像 Fortran 和 Haskell 这样的“低资源”代码上的表现较弱。

然而，冯·韦拉坚称这是朝正确方向迈出的一步。

“我们坚信，与 AI 模型建立信任和责任的关键在于完整的模型管道的透明性和可审计性，包括训练数据和训练配方。” 他说道，“StarCoder 2 展示了完全开放模型如何提供竞争力的性能。”

你可能会想——正如本文作者一样——Hugging Face、ServiceNow 和 Nvidia 有什么动机投资像 StarCoder 2 这样的项目。毕竟，他们都是企业——训练模型并非便宜。

据我所知，这是一种经过验证的策略：在开源发布的基础上建立有偿服务，以培育善意。

ServiceNow 已经利用 StarCoder 创造了 Now LLM，这是一个专为 ServiceNow 工作流模式、使用案例和流程精调的代码生成产品。Hugging Face 则提供模型实施咨询计划，并在其平台上提供 StarCoder 2 模型的托管版本。Nvidia 也通过 API 和 Web 前端提供 StarCoder 2。

对于专注于无费离线体验的开发人员来说，StarCoder 2——模型、源代码及更多内容——可从项目的GitHub页面下载。
