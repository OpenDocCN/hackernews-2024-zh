<!--yml

category: 未分类

date: 2024-05-27 12:54:12

-->

# AI与Web：理解和管理机器学习模型对Web的影响

> 来源：[https://www.w3.org/reports/ai-web-impact/](https://www.w3.org/reports/ai-web-impact/)

## 摘要

本文提出了对[AI系统](#dfn-artificial-intelligence-system)（尤其是基于[机器学习模型](#dfn-model)的系统）对Web系统性影响的分析，并探讨了Web标准化在管理该影响中可能发挥的作用。

## 本文状态

本文旨在捕捉[W3C团队](https://www.w3.org/staff/)对人工智能系统对Web发展的当前和预期影响的共享理解，并确定World Wide Web Consortium社区已经或应该开始的探索，以管理该影响。本文不代表W3C成员的任何共识，也不是标准化文件。

本文由Dominique Hazaël-Massieux（[dom@w3.org](mailto:dom@w3.org)）撰写，并得到W3C团队其他成员的重要贡献。

本文首要目标是帮助结构化讨论，探讨在标准化层面可能需要的措施，以使AI（特别是[机器学习模型](#dfn-model)）的系统影响变得更少有害或更易管理。本文肯定是不完整的，有时可能是错误的 - 我们正在收集意见和反馈，优先在2024年6月30日之前在[GitHub](https://github.com/w3c/ai-web-impact/issues)上进行。

根据收到的反馈，可能的下一步包括更深入的利益相关者访谈、专门的W3C研讨会或制定标准化路线图。

## 目录

1.  [摘要](#abstract)

1.  [本文状态](#sotd)

1.  [<bdi class="secno">1\.</bdi> 执行摘要](#executive-summary)

1.  [<bdi class="secno">2\.</bdi> 引言](#introduction)

    1.  [<bdi class="secno">2.1</bdi> 术语](#terminology)

1.  [<bdi class="secno">3\.</bdi> AI系统与Web的交叉点](#intersections-between-ai-systems-and-the-web)

1.  [<bdi class="secno">4\.</bdi> 伦理与社会影响](#ethics-and-societal-impact)

    1.  [<bdi class="secno">4.1</bdi> 尊重自治与透明度](#respecting-autonomy-and-transparency)

        1.  [<bdi class="secno">4.1.1</bdi> AI生成内容的透明度](#transparency-on-ai-generated-content)

        1.  [<bdi class="secno">4.1.2</bdi> AI中介服务的透明度](#transparency-on-ai-mediated-services)

    1.  [<bdi class="secno">4.2</bdi> 隐私权和数据控制权](#right-to-privacy-and-data-control)

    1.  [<bdi class="secno">4.3</bdi> 安全与保障](#safety-and-security)

    1.  [<bdi class="secno">4.4</bdi> 可持续性](#sustainability)

    1.  [<bdi class="secno">4.5</bdi> 平衡内容创作者的激励与消费者的权利](#balancing-content-creators-incentives-and-consumers-rights)

        1.  [<bdi class="secno">4.5.1\. <span class="secno">4.5.1</span></bdi>] 与搜索引擎的比较](#comparison-with-search-engines)

1.  [<bdi class="secno">5\. <span class="secno">5</span></bdi>] 集成AI系统对互操作性的影响](#interop)

1.  [<bdi class="secno">A\. <span class="secno">A</span></bdi>] 参考资料](#references)

    1.  [<bdi class="secno">A.1\. <span class="secno">A.1</span></bdi>] 信息性参考文献](#informative-references)

[机器学习模型](#dfn-model)支持一代新的AI系统的生成。这些模型通常通过对大量的网络内容进行[t全球化训练](#dfn-training)，并通过网络接口大规模部署，并可生成前所未有的速度和成本下可靠的在线内容。

考虑到这些交叉点的范围和规模，这一波 AI 系统[对企业化智能](#dfn-artificial-intelligence-system)对网络及其生态系统中生长的一些平衡状态可能产生潜在的系统性影响。

本文档通过伦理、社会和技术影响回顾这些交叉点，并点出标准化、指南和互操作性可以帮助管理这些变化的几个关键领域：

我们正在[寻求输入](https://github.com/w3c/ai-web-impact/issues)来自社区对有助于推进这些问题的提案，以及本文档尚未识别的其他主题。

数十年来计算机科学领域的人工智能发展，引发了一系列系统产生影响并在网络上已经启动，且可以预期进一步重塑大量依赖网络健康特性的共同期望。

为了帮助塑造 W3C 社区（及其可能与其他Web相关标准组织）关于这些转变的对话，本文档收集了W3C 团队关于'人工智能'，具体是在[机器学习模型](#dfn-model)领域（包括大型语言模型和其他所谓的生成 AI 模型），以及网络作为一个系统之间的交集的当前共享理解，并在这一空间的持续开发的背景下的共享内容。我们的目标之一是在AI领域进一步发展时，弄清楚可能需要进行的其他探索。

当前面的这一理解可能不完整或有时完全不正确；我们希望能通过发布这部文件和征求社区的意见，不断优化这一共享理解，帮助构建一个路径以增加这一交汇处的正面影响并减少潜在的负面影响。

"人工智能"一词涵盖了广泛的算法、技术和技术。[[ISO/IEC-22989](#bib-iso/iec-22989 "人工智能概念与术语")] 将 <dfn id="dfn-artificial-intelligence" tabindex="0" aria-haspopup="dialog" data-dfn-type="dfn">人工智能</dfn> 定义为 "研究和开发[AI系统](#dfn-artificial-intelligence-system)的机制和应用"，其中 <dfn data-lt="Artificial Intelligence system|AI system" data-plurals="ai systems|artificial intelligence systems" id="dfn-artificial-intelligence-system" tabindex="0" aria-haspopup="dialog" data-dfn-type="dfn">AI系统</dfn> 是 "一个工程化系统，为给定的人类定义目标生成内容、预测、建议或决策等输出"。在撰写本文档的2024年初，Web生态系统关于人工智能的讨论主要集中在基于 <dfn id="dfn-machine-learning" tabindex="0" aria-haspopup="dialog" data-dfn-type="dfn">机器学习</dfn> 的系统上（"通过计算技术优化模型参数的过程，使模型行为反映数据或经验"）及其软件表现形式， <dfn data-lt="model|Machine Learning model" data-plurals="machine learning models" id="dfn-model" tabindex="0" aria-haspopup="dialog" data-dfn-type="dfn">机器学习模型</dfn> （"根据输入数据或信息生成推断或预测的数学构造"）。

虽然我们承认人工智能的更广泛含义及其与许多其他Web和W3C相关活动（例如语义Web和链接数据）的交集，但本文档有意仅关注这些[机器学习模型](#dfn-model)对Web带来的当前影响。我们进一步承认，本文档在编写期间部分是对该领域期望膨胀和投资周期的响应。这种情况凸显了需要一个框架来结构化这一讨论。

由于集中于[机器学习](#dfn-machine-learning)，本文档通过操作[机器学习模型](#dfn-model)的两个主要阶段来分析AI的影响： <dfn id="dfn-training" tabindex="0" aria-haspopup="dialog" data-dfn-type="dfn">训练</dfn>（"基于训练数据，使用机器学习算法确定或改进机器学习模型参数的过程"）和 <dfn data-lt="run|running|inference" id="dfn-run" tabindex="0" aria-haspopup="dialog" data-dfn-type="dfn">推断</dfn>（利用这些模型实际产生预期结果），我们也常称之为运行模型。

Web扮演的一个重要角色是作为内容创作者的平台，使得他们可以大规模地向内容消费者展示他们的内容。AI直接关联到平台的这两个方面：

+   在许多情况下，模型是基于从Web抓取的内容进行训练的；这些内容的规模和结构（由底层标准实现）的结合使其成为训练数据的宝贵来源，支持了最近AI发展中一些最显著的成果，如大型语言模型或图像/视频生成器；

+   相反地，许多这些AI模型可以用来在前所未有的规模上生成内容，而Web的覆盖范围则使得这些内容可以无缝地部署到平台上数十亿用户身上。

当更具体地看待Web的浏览器介入部分时，这部分仍然主要是客户端/服务器架构，AI模型可以在服务器端或客户端上[运行](#dfn-run)（并在某种程度上，目前也可以在两者之间以[混合方式运行](https://github.com/webmachinelearning/proposals/issues/5)）。在客户端，它们可以由浏览器提供和操作（可以是用户请求，也可以是应用程序请求），或者完全由客户端应用程序自身提供。

值得注意的是，随着[AI系统](#dfn-artificial-intelligence-system)的迅速采用，它们与Web的交集注定会发展，并可能触发新的系统性影响；例如，结合了实时从Web加载的内容和机器学习模型的新兴[AI系统](#dfn-artificial-intelligence-system)可能会深入重新审视Web浏览器在消费或搜索内容方面的角色和用户体验。

[W3C技术架构组伦理Web原则](https://www.w3.org/TR/ethical-web-principles/) [[ethical-web-principles](#bib-ethical-web-principles "伦理Web原则")]包括确保“[Web不应对社会造成伤害](https://www.w3.org/TR/ethical-web-principles/#noharm)”。

正如上文所述，Web已经成为某些最近人工智能发展中的关键推动者，人工智能的使用和影响通过Web的分发而被放大。这促使W3C社区作为Web的管理者，理解这种组合可能带来的潜在危害，并确定可能的缓解措施。

[网络机器学习伦理原则](https://www.w3.org/TR/webmachinelearning-ethics/) [[webmachinelearning-ethics](#bib-webmachinelearning-ethics "网络机器学习伦理原则")]起源于[网络机器学习工作组](https://www.w3.org/groups/wg/webmachinelearning)，将联合UNESCO的[关于人工智能伦理的建议](https://unesdoc.unesco.org/ark:/48223/pf0000380455) [[UNESCO-AI](#bib-unesco-ai "关于人工智能伦理的建议")]的价值和原则与来自伦理网络原则的Web特定原则结合起来，识别出4个价值观和11个原则，这些原则应该遵循Web上的机器学习整合，并有助于构建这份文件的结构。

最近的[AI系统](#dfn-artificial-intelligence-system)能够在文本、图形、音频和视频的部分或完全创建中提供（至少在表面上）可信的质量，并且数量超过了人类的开发能力。这不仅为内容创作者提供了机会和风险，更重要的是，在大量可信但可能或故意错误的生成内容中，它为内容消费者的系统性风险创造了不再能够区分或发现权威或精心策划的内容的环境。

需求直接压力至最终用户，因为他们个别消耗内容，但也适用于最终用户依赖的代理：通常，搜索引擎可能会受益于对纯AI生成内容的透明度。有些讽刺的是，用于训练AI模型的爬虫也可能需要这样的信号，因为在[训练](#dfn-training)模型的输出上训练模型可能会产生意外和不受欢迎的结果。

我们不知道任何可以保证（例如，通过加密）某一给定内容片段是或不是由[AI系统](#dfn-artificial-intelligence-system)（部分或完全）生成的解决方案。不幸的是，这种空白留下了关于误导和垃圾信息的系统性风险，这应该引起对Web作为内容分发平台以及整个社会健康的严重关注。

在这一领域，标准的一个合理角色将至少是促进**内容的标注**，以指示它是否是**计算机生成的过程**的结果。虽然这些标签不太可能通过技术手段来强制执行，但它们可能会通过由[AI系统](#dfn-artificial-intelligence-system)自动添加（可能会在一定程度上增加移除它们的成本），并可能作为监管背景下的钩子而广泛采用。

在这一领域已经出现了多个提议，这些提议可能会因更多的可见性、讨论和最终的可扩展部署而受益：

可以探索的一个领域是 Web 浏览器在展示标签或内容来源信息方面可能扮演的角色，例如嵌入内容如图像或视频。目前这可以由发布者的网站或独立验证者来做，但将此功能集成到浏览器中可能会让用户更方便地访问信息，而且独立于任何特定发布者或网站，在这些地方可能查看到同一内容。

在 Web 上未经分类或部分分类内容训练的模型很可能包含个人可识别信息（PII）。对于那些用户选择与服务提供商共享的数据（无论是否公开），情况也是如此。这些模型通常可以被制造成检索并共享那些信息的工具，这违背了那些个人信息被收集的隐私期望，并且可能违反多个司法管辖区的隐私法规。更糟糕的是，它们为新类型的攻击创造了风险（见[<bdi class="secno">4.3</bdi> 安全性](#safety-and-security)）。

虽然在内容创建的背景下讨论的排除规则可能在第一个情况中部分有所帮助，但对第二个情况无济于事。这个问题领域很可能会接受严格的监管和法律审查。

从技术标准化的角度来看，除了标记内容之外，用户数据被重新用于模型 [训练](#dfn-training) 及其引发的一些反对意见可能会为去中心化架构带来新的动力（无论是用户还是服务提供商），从而减少对用户数据的集中控制（正如最近广泛采用的活动流示例所展示的）。

这种模式的一个特别相关实例是所谓的**个人数据存储**：它们为用户提供了更精细控制其数据的方式，通过更清晰地分离数据存储和数据处理角色（在传统的云基础设施中，通常由单一实体处理）。

这个话题最近通过 [提议的 SOLID 工作组宪章](https://lists.w3.org/Archives/Public/public-new-work/2023Sep/0007.html) 在 2023 年末在 W3C 上浮出水面（W3C 社区已经认识到其重要性，但尚无共识）。

允许在个人数据不上传到服务器的情况下[运行](#dfn-run)模型是浏览器[Web神经网络API](https://www.w3.org/TR/webnn/) [[WEBNN](#bib-webnn "Web Neural Network API")]背后的关键动机之一，它在已经由[WebAssembly](https://www.w3.org/groups/wg/wasm/) [[WASM-CORE-2](#bib-wasm-core-2 "WebAssembly Core Specification")]和[WebGPU](https://www.w3.org/groups/wg/gpu/) [[WEBGPU](#bib-webgpu "WebGPU")]提供的计算能力基础上，为在浏览器内（因此在最终用户设备上）高效地[运行](#dfn-run)模型提供了额外的机器学习特定优化。

一些[机器学习模型](#dfn-model)已显著降低了生成可信文本、音频和视频（实时或录制）的成本。这会显著增加钓鱼等各种欺诈行为的能力，并因此在建立在线互动的信任方面提高了更高的障碍。如果用户不再在其数字化介质互动中感到安全，网络将无法再发挥其作为这些互动平台的角色。

一些最大和最显著的[机器学习模型](#dfn-model)已知或被假设是通过从网络上抓取的材料进行训练的，而没有其创建者或出版者的明确同意。

从这种情况中产生的争议正在通过版权法的视角进行辩论（有些情况下甚至进行仲裁）。

我们没有权力确定各种版权立法是否以及如何对特定使用产生影响。除了法律考虑之外，版权制度在创建者和消费者之间创造了（相对）共享的理解，即默认情况下，内容不能在没有创建者同意的情况下重新分发、混合、适应或基于此构建。这种共享理解使得许多内容能够在网络上公开分发。它还允许创作者考虑各种货币化选项（订阅、按次观看、广告），基于消费者将始终到达其页面的假设。

一些[AI系统](#dfn-artificial-intelligence-system)结合了（1）自动化的大规模消费Web内容和（2）规模化生产内容的方式，这些内容并未承认或以其他方式补偿其训练来源的内容。

虽然其中一些紧张局势并不新鲜（如下所述），基于机器学习的系统正准备颠覆现有的平衡。除非找到新的可持续平衡，否则这将使网络面临以下不良后果：

+   明显减少开放分发的内容（这可能会对人口较少富裕的部分产生不成比例的影响）

+   一个不那么吸引人的内容分发平台

较少直接的风险可能源自旨在帮助重新平衡情况的版权法律的变化，但这些变化可能会减少内容消费者的权利，从而削弱 Web 作为一个内容分发关键价值主张的平台的价值。

围绕从 Web 大规模爬取内容再利用而产生的张力中，有些是长期存在的，这是因为搜索引擎在 Web 中的核心作用。确实，搜索引擎通过其检索和组织 Web 上内容的能力提供（并吸收）价值，并且它们在实现这些结果时严重依赖于标准化的基础设施。

搜索引擎和内容提供者之间出现的更或少隐含的契约是，搜索引擎可以检索、解析和部分显示提供者的内容，作为给予它们更多可见性和流量的交换。在 Web 运作方式中，进一步的假设已经被编码，即任何公开提供 Web 上内容的人默认接受这种契约，通过 [`robots.txt` 指令](https://www.rfc-editor.org/rfc/rfc9309.html) [[RFC9309](#bib-rfc9309 "Robots Exclusion Protocol")] 实现了一种选择退出的机制。

随着时间的推移，除了链接到与用户查询匹配的网站外，搜索引擎还整合了更多直接从目标网站上展示内容的方式：通过丰富片段（通常由使用 schema.org 元数据实现）或通过嵌入预览（例如，[AMP 项目](https://amp.dev/)的实现）。这些变化往往伴随着有时挑战性的讨论，围绕着在增加爬取内容的可见性和减少终端用户访问源网站的动机之间的平衡（例如，因为他们可能已经从搜索结果页面获得了足够的信息）。

在一定数量的情况下，[AI 系统](#dfn-artificial-intelligence-system)被用作用户传统上使用搜索引擎的替代或补充（事实上，它们越来越多地集成到搜索引擎界面中）。因此，探讨从搜索引擎和内容创建者之间平衡需求的进化过程中学到的教训，在培训[机器学习模型](#dfn-model)的爬虫上的讨论似乎是有用的。

在进行比较时，还需要注意显著的差异：

+   内容创作者从搜索引擎爬虫那里期望的隐含契约——即它们将为他们的内容带来曝光——在集成到[人工智能系统](#dfn-artificial-intelligence-system)中并没有系统化的等效物；虽然一些这样的系统正在获得指向其用于特定[推断](#dfn-run)的训练数据来源的能力，但这并不是这些系统的普遍特征，也不明显能够系统地应用（例如，为生成的图像链接回源是否有意义？）；即使可能如此，暴露的源数量也可能少于典型的搜索引擎结果页面，并且用户点击链接的激励可能大大降低。

+   `robots.txt` 指令允许根据用户代理为特定爬虫设定特定规则；尽管在处理（无论好坏）少数知名搜索引擎爬虫时这是可行的，但期望内容创建者维护快速扩展的爬虫数量的潜在允许和阻止列表似乎不太可能实现可持续的结果。

鉴于在[人工智能系统](#dfn-artificial-intelligence-system)背景下，爬行的“交换条件”可能存在不同的期望，显然，从Web早期（robots.txt于1994年设计）继承的无需许可模式是否能满足确保Web内容发布的长期可持续性并不明显合适（这本身也可能符合AI爬虫的长期利益）。

总体而言，在这个领域可能有助的一条探索路径是找出能帮助**内容生产者和AI爬虫找到可接受条件，理想情况下能够广泛适用于所有方**的解决方案。

几个团体和个人一直在探索如何使内容发布者能够表达他们愿意为[训练](#dfn-training) [机器学习模型](#dfn-model)使用其内容：

[W3C对Web的愿景](https://www.w3.org/TR/w3c-vision/#vision-web) [[w3c-vision](#bib-w3c-vision "W3C愿景")]的关键部分是确保Web围绕互操作性原则发展：即W3C制定的Web标准技术，确保它们在产品中的实施和部署方式是相同的，为用户提供更大的选择，并促进内容的长期可行性。

当跨操作依赖的算法是确定性的时，确保互操作性就是描述该算法的细节和清晰度，并对产品进行充分测试，以验证它们能够达到预期的结果。更多[算法规范](https://www.w3.org/TR/design-principles/#algorithms)的采用和通过[Web 平台测试项目](https://web-platform-tests.org/)进行彻底自动化测试，主要是为了提供一个强大的互操作平台。

正如上文所讨论的，[机器学习模型](#dfn-model)已经开始进入标准化Web API。这给我们的互操作性目标带来了两个挑战：

+   [机器学习模型](#dfn-model)大多数情况下不是按照一系列算法步骤来构建或描述的。如果期望某个标准化行为最好由[机器学习模型](#dfn-model)来实现，那么该行为应该如何指定？如何测试以足够验证在使用不同模型的产品之间的**可互操作结果**？这对浏览器的指纹表面有何影响？

+   一些重要的[机器学习模型](#dfn-model)并非确定性的；如果或当某些这些**非确定性模型**在标准化API中公开，这种一致性问题就不再仅限于两个使用两种不同模型的产品，因为给定的输入将不再产生预定的输出。目前尚不清楚如何准备基于非确定性模型的互操作行为，这可能引发关于这些模型是否应作为互操作实现的一部分以及如何接受的问题。

这些挑战的一个可能后果是，在可以被有意义地实现互操作和标准化的范围中，可能会减少功能数量，因为越来越多的功能受到[机器学习模型](#dfn-model)的调解（类似于[假设增加Web应用程序能力对需要标准化协议的影响](https://datatracker.ietf.org/doc/html/draft-tschofenig-post-standardization-02)的影响）。在这种情况下，例如围绕基于AI编解码器的讨论可能会指向互操作性景观可能的显著变化。

[↑](#title)
