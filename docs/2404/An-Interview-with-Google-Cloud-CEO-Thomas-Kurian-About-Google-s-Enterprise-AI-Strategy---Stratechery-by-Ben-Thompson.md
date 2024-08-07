<!--yml

分类：未分类

date: 2024-05-27 13:06:31

-->

# 谷歌云CEO Thomas Kurian关于谷歌企业AI战略的采访 – 由本·汤普森（Ben Thompson）撰写的Stratechery

> 来源：[https://stratechery.com/2024/an-interview-with-google-cloud-ceo-thomas-kurian-about-googles-enterprise-ai-strategy/](https://stratechery.com/2024/an-interview-with-google-cloud-ceo-thomas-kurian-about-googles-enterprise-ai-strategy/)

早上好，

本周的Stratechery采访对象是谷歌云CEO[Thomas Kurian](https://twitter.com/ThomasOrTK)。Kurian于2018年加入谷歌，负责领导公司的云部门；在此之前，他在甲骨文担任产品开发总裁长达22年。

在这次采访中（在我已经发布[Gemini 1.5 and Google’s Nature](https://stratechery.com/2024/gemini-1-5-and-googles-nature/)之后的昨晚进行），我们讨论了[Kurian在本周谷歌云Next大会上的主题演讲](https://www.youtube.com/watch?v=V6DJYGn2SFk)，该演讲几乎完全专注于人工智能。因此，我们涵盖了谷歌云的战略，为什么AI提供了竞争的重置，以及公司如何在广泛的企业空间取得胜利。正如昨天的文章和这次采访所明确的那样，我确实认为谷歌在这一领域处于非常有利的位置；显然Kurian也是这么认为的。

作为提醒，所有Stratechery内容，包括采访，都可以作为播客获得；点击邮件顶部的链接将Stratechery添加到您的播客播放器中。

进入采访内容：

### 谷歌云CEO Thomas Kurian关于谷歌企业AI战略的采访

*此次采访略作编辑以提高清晰度。*

### 谷歌的云战略

**Thomas Kurian，[欢迎回到Stratechery](https://stratechery.com/2021/interviews-with-patrick-collison-brad-smith-thomas-kurian-and-matthew-prince-on-moderation-in-infrastructure/#kurian)。**

**Thomas Kurian：** 感谢邀请我。

**因此，我将让您开场发言。如果有人对您说：“嗨，Thomas。今年稍早些时候的[Google Cloud Next](https://cloud.withgoogle.com/next)大会是关于[什么](https://stratechery.com/2024/gemini-1-5-and-googles-nature/)？您在几个月前就已经进行了，您希望从主题演讲中传达给大家什么主要信息？**"

**TK:** 我们产品策略的重点是我们在主题演讲中向客户解释的内容，而产品策略非常简单。我们看到客户如何加速使用数字工具和AI来转变其核心业务。为了实现这一点，我们提供了两个重要的事项。首先，随着人们将AI从概念验证转移到其企业多个部分的生产部署中，他们希望的不仅是选择一个模型，而是一个平台，而这个平台需要具备某些特性。平台需要提供一系列服务来调整模型，将其与企业系统连接，能够委托任务给模型，衡量模型的质量，测试、部署和监控它。我们的平台提供了这些功能。

此外，我们拥有一个开放的架构，使他们能够使用这些服务，但跨不同模型范围。有些来自Google，显然是Gemini，但也来自合作伙伴。

此外，我们宣布Gemini取得了一些重大进展。我们推出了百万上下文窗口，展示了客户正在使用它的多个场景，我们对我们的图像模型[Imagen](https://imagen.research.google/)进行了重大更新，并将其整合到我们的整个产品组合中。当我说整个产品组合时，例如在Workspace中帮助人们进行写作、电子邮件、创建文档，但同样重要的是，在他们现有的工作流程中提供帮助，我们还通过引入[Google Vids](https://www.youtube.com/watch?v=4SCjXcBeW1E)，展示了使用生成AI进行故事叙述的全新体验。

同样地，我们将其引入了我们在Google Cloud平台上的应用。我们从编码开始，但我们为那些希望操作云的人们增加了操作。我们增加了数据分析的能力，并在每一步以及网络安全方面都有所加强。在每一步中，这意味着人们可以更好地与其数据交流，也意味着团队可以通过与真正的专家一起工作来加快网络安全分析的速度。

因此，作为企业，如果您打算大规模使用AI，您需要选择一个平台，而这个平台既需要是开放的，也需要是垂直优化的，因为如果您要在各种环境和系统以及流程中使用它，您需要它既具备成本效益，又具备扩展性，而我们在Google知道如何做到这一点。

因此，我们在我们的AI和其他系统中引入了许多新的进展。我们推出了一款[新的Axion处理器](https://cloud.google.com/blog/products/compute/introducing-googles-new-arm-based-cpu)，我们在传统的经典计算中与Intel引入了许多新的进展，但我们还在我们的AI系统中引入了各种新技术。TPU v5，一些新的Nvidia系统，而且最重要的是，大多数人认为这只是关于芯片，事实并非如此，重要的是你用这些芯片建立的系统，我们有很多优势的事情。一个例子是我们引入的新的[Parallelstore](https://cloud.google.com/parallelstore)，它可以在模型在训练时从磁盘加载数据时提供10倍的提升，如果您要进行大规模的训练或推理，这非常重要。

所以这些是一些进展。现在我们能够在规模极大的集群上部署这些技术，因为我们提供了各种优化用于多种不同训练和服务需求的加速器，所以我们看到了很多的采用，从成本、性能和延迟的角度来看，这使得整个过程更加高效。

**嗯，这是一个很好的概述。我觉得你预览了我想深入探讨的很多内容，但你提到了，“人们正在从概念验证转向实际产品”。这是真的吗？公司实际正在广泛推广哪些实际用例，而不是进行可能性实验？**

**TK：** 总的来说，本，我们可以将其分为四个主要类别。第一个类别是优化组织内部流程，简化内部流程。在财务领域，您希望自动化应收账款、收款和现金流预测。在人力资源方面，您希望自动化人力帮助台，并提高例如福利匹配的效率。在采购和供应链方面，例如，您希望查看所有我的供应商，他们与我的合同，并告诉我哪些供应商为我提供了赔偿和保修保护，这样我就可以将更多的业务量引导给那些提供了赔偿和保修的供应商，而减少与那些没有的供应商的业务量。例如，这些都是我们的客户正在实施的实际案例。

第二个是转变客户体验。转变客户体验，例如您如何市场推广，如何展示商品，如何进行商务以及如何进行销售和服务。一个例子是梅赛德斯-奔驰CEO [奥拉·凯伦尼乌斯](https://group.mercedes-benz.com/company/corporate-governance/board-of-management/kaellenius/) 谈到了他们如何为他们的车辆市场、销售和服务构建全新的体验。

第三点是，一些人正在将智能技术整合到他们的产品中，当我说重新构想他们的产品时，我是指重新构想他们的核心产品使用人工智能。我们有两个公司的例子，它们处于设备领域。一个是三星，另一个是Oppo，他们正在利用我们提供的各种多模态功能，通过人工智能重新构想实际设备本身。

现在有很多公司正在重新思考，如果一个模型可以改变我看待事物的方式，那我是否可以处理多模态信息。例如，在媒体领域，有人说，“如果您的模型可以读取尽可能多的信息，那么它可以将长篇的电影缩短成精彩片段吗？我能拿一场NCAA篮球锦标赛的体育录像然后说，“给我找出这个特定球员的所有精彩片段”而不必让一个人坐在那里剪辑视频，而是让它来做，我可以非常快地创建精彩片段。因此，很多人都在重新构想他们的产品提供。

最后，有一些人说，“凭借这种成本效益，我可以改变我进入全新市场的方式，例如，我可以在没有实体存在的市场上进行个性化的推广，但我可以通过在线营销和广告为客户实现更高的转化率，因为现在我可以进行高度定制的营销活动，因为制作内容的成本要低得多。”因此，大幅度地简化核心流程和后台，转变客户体验，并不仅仅是呼叫中心或聊天机器人，它实际上可以转变产品本身，改变产品的性质，并进入新的市场。

**那么，当您谈到“从概念验证到实际生产”时，或许您并没有用这个词，但人们都在说，“好吧，我们要构建这个”因为这些东西还没有出现在现实世界中。所以，是否可以说，“我们看到这可能是有价值的，现在我们参与进来了”，这就是为什么您现在强调平台选择，因为他们已经全面投入了人工智能，现在就像，“我们要在哪里构建它”？

**TK:** 我们有人在尝试实验，但我们也有人确实在实时部署和引导交通。橙子电信公司谈到了他们在线处理多少客户，Discover Financial谈到了他们的代理正在使用搜索和人工智能工具从实时政策和流程文件中获取信息。因此，确实有人正在通过这些系统运行真实流量，并确实在处理真实客户工作负载。

**您是否看到很多客户或者可能从潜在客户那里听到，AI 正在推出，如果这个词正确，那么在员工套利情况下正在进行？即个别员工正在自行使用这些工具，他们正在个人受益于增加的生产力 — 也许他们做更少的工作或者他们做更多的工作 — 公司希望更系统地捕捉到这一点。这是您看到的一个主题吗？**

**TK:** 我们看到三种情况。第一种情况是公司有，我们将尝试八九种他们称为客户旅程或用例，我们将选择我们认为具有最大回报的三种，意味着价值和价值并不总是指成本节约。例如，我们有一个每天通过我们的客户服务系统处理 100 万通电话的情况。现在每天处理 100 万通电话，如果你考虑一下，Ben，一个普通人一天大约可以处理 250 通电话，这是一个八小时的工作日内的一定量。如果你处理了一百万通，那就是很多人，所以事实是其中有几个没有得到答复，人们从未打电话是因为等待时间太长。所以在这种情况下，这不是关于成本节约，而是他们能够接触到比以前更多的客户。所以这是一个。一个部分是人们说，“我有一堆场景，我会挑选其中的三个”，在许多情况下，他们实际上正在增强他们正在做的事情或者做一些以前不能做的事情，这就是第一种情况。

第二种情况是，例如，有一家大型保险公司正在与我们合作。今天，当他们处理索赔和风险计算时，处理索赔和特别是风险计算需要很长时间，因为有成千上万页的文件，有很多来回传递的电子表格。他们将其放入 Gemini 后，能够大大加快计算速度。所以第二点是，我正在选择一个对我的组织非常有价值的用例，这是核心功能，我将实施它，因为我可以获得真正的竞争优势。在他们的情况下，事实是他们不仅可以获得更精确的风险评分，而且在响应方面也可以更快速、更准确地工作。

第三种情况是你说的那种。“嘿，我们有一群人，我们将把它交给一定数量的开发者”。例如，我们的编码工具，“他们将测试它，他们说它帮助我生成更好的单元测试，帮助我编写更高质量的代码”。Wayfair 的首席技术官正在谈论他们的经验，然后他们说，“让我们广泛应用”，所以这三种模式都在被看到。

### 一个“开放”的平台

**这是一个广义上的Google Cloud活动，今天Google Cloud的大部分业务都是Workspace和实际的云计算的结合。然而，整个演讲都是关于AI的。我是说，也许不是完全，你有几个Google Meet的更新之类的东西，但总体来说，我认为这是一个公正的描述。这对于Google的优先事项意味着什么？如果我只是一个普通的Google Cloud客户，我会说，“是的，AI很好，很酷，但我的托管数据库怎么样？”或者可能是其他什么？

**TK：** 坦率地说，Ben，我们这里有30,000名员工，而且有数百场次的会议，其中一些是关于AI的，但也有许多其他主题的会议，我认为你在主题演讲中看到的并不代表全部。

尽管我们看到，客户在选择云合作伙伴时，越来越关注的不是一个时间点，而是“谁将帮助我转型我的业务？”的问题。他们在2007年、08年、09年考虑这个问题的基础是，“如何通过构建应用程序加快速度，或者通过允许我迁移和转移工作负载来降低数据中心成本”。现在，他们以不同的方式来思考。他们把它看作是，“我能用AI来改变我的业务吗？谁有最好的平台和工具帮助我做到这一点？”。

一旦我们让他们使用AI平台和工具，它确实会牵扯到我们许多其他的服务。你需要好的数据来喂养模型，所以很多人使用我们的分析系统BigQuery来清理数据，处理数据的分段等。其他人说，“我想从操作数据库，比如AlloyDB、Spanner或我们的其他操作数据库中获取数据”。我们也有很多人说，“我想保证模型的输出安全可靠。我还想看看是否有人在分析，试图攻击我的系统”，所以我们销售我们的网络安全工具。因此，AI不仅仅是AI本身，通常是你在进行AI时需要的各种东西的集合。

**对于这种想法的解释，AI几乎作为矛头的一种方面，还有你在最后深入讨论的消息传递？你已经提到过关于“开放”的谈话，开放可以意味着很多事情，但我理解为，“你可以在任何级别访问我们所提供的任何内容”。这是否打开了一种可能性，即，“如果您已经在其他地方使用云计算，如果您在其他地方拥有大量数据，我们将使其非常容易与我们对接，因为也许您需要我们的AI工具，我们希望您带着其他一切过来，但无论如何我们都会合作”？这是否是开放框架的一种思考方式？**

**TK:** 开放包含三个元素。首先，当然是这样的。所以，如果今天您的应用程序运行在其他云上，或者如果您正在使用像Salesforce、Workday、SAP系统这样的SaaS应用程序，并且您在思考，“我能否使用谷歌的AI并将其连接到我的现有应用程序中？” — 答案是肯定的，而且您实际上不需要将任何东西转移到我们这里来实现这一点。这就是第一个元素，它使我们能够利用AI为每个客户提供服务，而不仅仅是现有的GCP客户。

第二个要点是，当我们说开放时，这意味着人们可以根据他们组织中的正确目的选择模型。例如，我们有些人说，“看，我喜欢Gemini，我将在我的客户面向功能和后台功能中使用它，但是我在这个特定的业务线上，我想为此目的开发自己的模型，因为我正在将其集成到我自己的产品体验中，并且我真的需要控制模型的权重”。在这种情况下，他们可以使用我们的开源，或者他们可以使用Mistral或其他一些开源工具。

但在过去，他们总是不得不做出选择。如果您选择了一个模型，您就必须选择一个工具集，这是没有任何意义的，因为您需要做的所有常见事情，比如调整它、接地它、将它集成到您现有的数据存储中以进行检索、增强生成等，这些都是我们平台提供的功能，这就是我们所做的第二部分。

第三个要点是，我们还可以帮助他们将这些点连接到其他服务和GCP，如果他们愿意的话，这样就能吸引新客户，让我们能够开放，让他们选择使用多种不同的模型来满足他们整个组织的需求，同时采用一种通用的方式来管理、监控和改进这些模型，并且，这也使我们能够提供一些其他的服务。

**我对整体情况很好奇，这种策略增长的重新启动是不是？在Google Cloud的结果背景下，去年Q3，[增长率从28%下降到22%](https://stratechery.com/2023/google-earnings-microsoft-earnings-ai-leverage/)，利润率收缩，这逆转了一个非常长的趋势。在我们讨论AI连接到此平台之前，去年发生了什么？那时候出了什么问题？**

**TK:** 显然，这不是一个结构性的问题，因为如果您看看我们的Q4和我们的结果，它们都显著增长，我们对未来非常有信心。在过去的几年中，如果您回顾2019年至今，我们是唯一一个云提供商，看看我们的收入和运营收入都在增长，没有任何一个，甚至那些市场份额最大的云提供商，能够像我们这样长时间地增长收入和运营收入。

这绝对不是战略的重启。我们启动了名为[multicloud](https://cloud.google.com/multicloud)的项目，即“不要被锁定在单一云服务提供商上，允许用户选择云服务提供商，选择最适合任务的云服务提供商”。因此，分析是我们表现特别突出的领域，我们在某些领域表现得非常出色，比如某些类型的传统系统迁移，我们的系统运行得非常出色，因为我们可以以不同于其他人的方式进行规模扩展，这使我们能够打开很多门。

当AI出现时，第一个周期是每个人都在考虑，“我必须选择一个模型”，而模型每三周就会变化，所以我们的观点是，“当你考虑选择模型时，你正在追逐错误的东西，你需要考虑的是平台”，因为你需要将其集成到你的异构性中，并首先考虑平台，其次是模型，并确保平台支持一系列模型的收集，因为你可以从任何人那里选择最新的模型，这就是问题的本质。

**好吧，这就是我为什么要谈到重启的问题，因为我认为，你要处理的这个想法是，你可以拥有你的多云，这在市场上的竞争地位中是有意义的，作为第三名。不过，您是否认为在所有这些关于“您需要选择平台”的讨论中，AI可以作为一个楔子，比如，“好吧，这在云的广泛重启方面是个大动作，当然，您的数据可能在AWS或Azure等等，但是如果您有一个未来的平台，您应该从我们这里开始”？也许十五年后我们会看到，所有的重心都转移到了平台的位置上？**

**TK：** 当然。我的意思是，这是人们做购买决策方式的改变，对吧？十年前，您担心的是商品化计算，您会想，“谁能给我最低的计算成本、存储成本和网络成本？”现在竞争的基础已经改变了，考虑到我们在顶端的能力，即提供平台、提供模型等等，我们拥有非常强大的位置，并且正在构建长期集成模型的产品。

举个例子，Ben，将模型集成到产品中并不像人们想象的那么简单；Gmail自2015年以来一直在这样做。每天我们运行超过5亿次操作，要做好这件事情，当合作伙伴谈到75%的生成幻灯片图像的人最终会呈现时，这是因为多年来我们非常关注如何进行集成。

所以我们在顶层玩，我们有基础设施和规模，可以从成本、性能和全球规模上做得非常好，这改变了竞争的本质。所以我们确实看到这一点，正如你所说的，这是客户选择他们的云决策的一个重置时刻。

**如果你在谈论关于模型的很多选择，并且客户在选择正确的模型时有过度投入，这可能意味着模型可能是商品，我们已经看到，自GPT-4发布以来，价格下降了大约90%。这是一个你预期继续下去的趋势吗？这是你想要推动并实际加速的事情吗？**

**TK:** 模型——无论它们是否成为商品，时间会告诉我们，现在还处在非常早期的阶段。我们要指出的是，每个月都会有新的玩家推出新模型，现有模型在多个方面都得到改进。这就像试图根据相机选择手机一样，而相机每两周都在变化，对吗？你愿意以此为基础做出选择吗？

**好吧，但如果你以此为基础，那么你可能会被锁定到操作系统上。**

**TK:** 是的，这就是为什么我们说你应该选择一个开放的平台，你应该能够使用一系列不同的模型，因为它正在改变，不要在应用程序正在改变的时候锁定到特定的操作系统上，用你的类比说。

**为什么你的平台比其他平台更开放？微软已经宣布你可以使用其他模型，而不仅仅是OpenAI的模型。亚马逊似乎在某种程度上确定了一种策略，“看，我们不承诺任何事情，你可以做任何你想做的。”为什么你觉得自己说“不，我们是开放的”，而他们不是？**

**TK:** 首先，我们平台的完整性；[Vertex拥有](https://cloud.google.com/vertex-ai)比其他平台提供更多的服务。其次，为了改进平台，你必须有自己的模型，因为在使用该模型工程化服务时会做很多事情。

我给你举个非常基础的例子。你使用一个模型，你决定[ground the answers](https://cloud.google.com/vertex-ai/generative-ai/docs/grounding/overview)。Grounding可以提高质量，但也可能引入延迟。你如何确保在grounding时不是串行后处理模型的答案以增加延迟？除非你有自己的模型，否则你甚至无法做到这一点。因此，因为我们有自己的模型，我们能够进行这些工程化的事情，但我们将其作为服务与其他模型一起提供，所以你可以使用企业grounding作为一个非常具体的例子。许多客户正在与Mistral、Llama和Anthropic一起使用它。

第二件事，我们不仅仅提供模型，而且我们实际上正在帮助第三方与我们一起接触客户。今天我与[CEO] Dario [Amodei] 共同与来自Anthropic的许多客户见面，这是一种承诺，确保我们不仅仅是提供基础设施，不仅仅是进行训练并将模型集成到Vertex中，不仅仅是使其成为一流模型，而是实际上将其与客户一起推广。

我认为这就是我们所说的开放性。另一家公司没有自己的模型，因此他们自然而然地提供了一堆模型，而另一家公司则将其模型开发外包给了第三方。

### 数据引力和市场推广策略（GTM）

**在企业背景下，你如何看待LLMs？你提到了这个基础的部分。我认为在消费者领域，就像是“哇，我可以查找答案”，但从企业角度来看，我想知道的是，人们多频繁地提出对幻觉的担忧。也许它们甚至有点夸大。在许多演示中，LLMs更多是现有数据的用户界面。这主要是它的框架吗？或者还有其他用例？我的意思是，“帮我写邮件”，但你是否主要将其视为企业用例中的数据接口？**

**TK：** 有一群人正在使用它来与他们的数据进行问答，我们称之为Open Book Q&A。可以把它看作是我有一堆数据，我想对它进行总结并问它问题，不一定是直接查看原始数据，而是总结并提问。当然，这是其中一种方式，但也有人正在自动化流程，并且自动化流程就像是将一个模型的输出，喂给另一个模型，并用它驱动一个流程。你明白我的意思吗？

**是的。**

**TK：** 他们使用函数调用，就像我给一个模型一个问题，它生成一个答案，我把它放进一个函数中，函数做一堆事情，然后它调用另一个模型，接着那个模型接手了。

想象一下当你在做软件工程或编码时，我用它作为一个例子，你可以使用模型来“审视我的代码库，看看是否存在安全漏洞”，比如我在代码库中有一个VSA漏洞模式，这相当于在你的数据上进行问答。第二点是，“嘿，如果你找到了，就修复它，并且这是我希望你遵循的修复模式，然后运行扫描看看它是否仍然存在，然后编译并部署它。当你部署它时，我想运行一个A/B测试。”我们现在看到很多人正在做的第二件事情就是自动化流程，而这个流程的复杂程度可以有很多不同的层次。

**对于实现与你的数据进行问答或执行这些过程来说，数据是否必须在GCP上是至关重要的？或者外部数据如何使用？因为我认为，这个框架和优势，比如说，与没有自己模型的竞争对手相比，他们拥有所有的数据，因此数据具有很大的引力，它将吸引人们去那里，因为模型是一个商品，他们可以做XYZ。**

**TK:** 我们支持来自四种环境的数据，包括本地数据中心、类似软件即服务的应用程序。你的客户服务数据可能在Salesforce，而Salesforce可能完全不在GCP上运行。它可能在另一个云服务提供商那里，或者可能在向量存储中。

**这个目标是不是无关紧要？还是说，更多数据在Google上运行会更好？**

**TK:** 你不必在Google上使用它，首先。其次，我们要说的重要一点是，有些基本的事情我们建议人们做，以确保他们考虑到。

一个例子是你如何处理数据的访问控制权限，因为模型本身无法处理访问控制规则。你可以在询问模型一系列问题后将其放在一个函数中，但这是后处理。有技术可以在你的数据上添加访问控制，工程师们称之为[ACLs](https://cloud.google.com/storage/docs/access-control/lists)（访问控制列表），所以我们建议他们考虑这一点，并给他们提供设计模式。

其次，我们要求他们对系统进行测试以确保系统具有可预测的延迟。因为一个模型可以发送请求，如果你的系统有一个巨大的队列，有时会快速响应，有时会花费很长时间，模型会混淆是否在等待答案，这是第二件事。

第三件我们通常要求他们解决的事情是，有工具来评估数据的质量，当我说数据的质量时，确保你不处于垃圾进，垃圾出的情况。所谓的质量，并不仅仅是指结构上的质量，确保没有空集等问题，还包括语义上的正确性，因为如果你要求的是收入，而你看的是发票收入而不是GAAP收入，你会得到一个发票收入的答案，你需要确保它是正确定义的，无论你指的是GAAP收入还是发票收入。这些都是例子，我们为客户提供蓝图来执行它。

**关于这个的决策是在哪里做出的？特别是当你谈论一个平台的背景和它的互动方式时，这是在CTO、CIO、CEO级别还是在工程领导或项目经理领导层次上做出的选择？或者这是工程师领导或项目经理领导可以说，让我们使用Vertex AI平台——你们在这方面的市场推广是怎么考虑的？**

**TK:** 通常情况下，决策是由业务单位领导和IT领导共同做出的。有时候，这种情况对公司非常重要，是CEO推动，因为这可能对公司的商业模式产生真正重大的改变。通常是业务线负责人和CIO或CTO。

**这是一个有点书呆子的问题，但我很好奇关于所有这些人工智能工作的成本如何分配。我假设 Google Cloud 自己支付其服务器的费用，但与 Google 正确的共享研发和模型开发也很多。Google Cloud 只是服务端的成本吗？我很好奇——就像我说的，这是一个非常书呆子的问题——我只是好奇你如何与 Google 正确互动，考虑到这里有这么多共享的开发。**

**TK:** 首先，我们的全球基础设施对于 Google 来说是云和其他部门共享的，你知道这一点已经是很长时间了。

**Yeah.**

**TK:** 所以从物理基础设施的角度来看，我们与所有产品单位共享基础设施，包括 Google DeepMind 团队构建模型的部门。关于成本分配的问题，我只能告诉你，我们被报告为一个部门，我们支付公平份额。

**这有点回到了商品化的问题。Google Cloud 和这些人工智能，你是想通过更优秀来获胜吗？还是说你真的可以利用基础设施的优势？当然，你应该共享资源，这是你不仅在功能上获胜，而且在成本、价格和总体成本拥有巨大优势的地方。**

**TK:** 如果可以的话，本。首先，我们之所以获胜是因为我们更优秀。每个选择我们的客户都有选择其他人的机会，我们之所以胜出是因为我们拥有出色的产品，包括在人工智能领域。其次，在规模化应用人工智能的重要因素之一是成本，但成本之外还有其他因素。还有延迟。例如，像“我需要问模型多少次才能得到正确答案？”，因为每次访问模型时，您都会传入一组标记，这会产生一定的成本。这也是客户体验的问题。你明白我的意思吗？

**Yep.**

**TK:** 在所有这些事情上，我们已经做了很多工作来解决性能、规模、延迟等问题。我的意思是，作为一个非常简单的例子，要使百万个标记工作，您需要在设计模型的方式上进行改变。这会影响到服务的库存等等。因为我们在这两个领域都拥有专业知识，所以我们能够做到这些事情。

### 基础设施和一百万标记

**这个百万上下文窗口在你讲述的故事中有多重要？我的感觉是，如果你在其周围构建了大量的基础设施，无论是RAG还是其他实施，你都可以做很多事情，但是我觉得，用Gemini 1.5，有很多全能可能性似乎可以更大程度地开放，并且有一部分，在那里，你有了合规性部分，工作声明，他们不得不将其与100页的合规性文件进行比较。我得到了一些评论，比如，“也许公司不应该有100页的合规笔记本或其他什么”，但现实是，世界就是这样。我对于这次主题演讲的看法是，那似乎是杀手级功能，似乎为一切奠定了基础。这个感觉是正确的吗？**

**TK:** 是的，有两个原因。只是为了非常清楚，本，长上下文窗口可以让你做三件重要的事情。首先，当你观看高清视频时，例如其他形式，比如想象你把一个高清视频传入，你想从最近的NCAA比赛中创造出精彩的片段，但你不想具体指定要剪辑到精彩片段中的每个属性。模型必须将其消化，因为它必须处理它，视频的表示就会变得相当密集，因为有物体，有人在移动，有动作，比如我在传球。它可能是，我的球衣上有我的名字，可能有分数，像，“他们是什么时候从24分变成26分的？他们投了三分球吗？”，所以有很多，很多维度。因此，当你可以用更多上下文进行推理时，推理就会变得更好，这是第一个，对形式尤为真实。

第二点是，如今人们不再使用模型来维护状态或记忆，这意味着他们问一个问题，下一次他们想，“嘿，它可能不记得”，所以当你能够维护更长的上下文时，你就能够维护更多的状态，因此你就能够做出更丰富和更丰富的事情，而不仅仅是和一个非常简单的接口来回交流。你懂我的意思吗？

第三件事是，肯定有复杂的场景，这是不幸的现实，有很多政策和程序手册甚至比我们展示的还要长，所以有这样的场景，我们必须能够处理。但长期来看，真正的突破在于以下。上下文长度，如果你能够从模型的能力和为模型提供服务的延迟中分离出上下文长度，那么你就能从根本上改变模型的扩展速度。

**从你的角度来看，这最终是一个基础设施问题，也是谷歌最大的优势？**

**TK:** 这是一个全球基础设施的问题，同时也是基础设施的每一层的优化，我们可以与DeepMind共同工程设计。

**对，这涉及到你自己模型的集成点。**

**TK:** 是的，然后第二件事是Google DeepMind在使用基础设施方面的专业知识。如果我可以举个例子，这有点像多年前，当Google建立其全球基础设施并收购YouTube时，他们可以立即在全球范围内扩展视频分发，而如果你回到过去，当Google收购YouTube时，视频行业有很多公司，但他们没有那种可扩展的基础设施。最终，我认为你已经看到我们的成功，因为我们可以进行全球分发。这是一个类似的概念，可以说是几年后的情况。

**是的，这符合我的思考方式，实际上对我来说，这就是那个主题演讲鼓舞人心的地方，因为它感觉上有一个方面，在任何产品开发中，你需要了解你的客户，理解他们的用例，但我认为公司需要了解自己的一部分，而我从那个主题演讲中得到的，以及我在后ChatGPT时代（我称之为现代AI时代）感到鼓舞的方式，是真正的，“我们知道我们是谁，我们擅长什么，以及我们将如何展现这种优势的方式”。我非常清楚地得到了这一点，我认为这是一个积极的迹象。**

**TK:** 谢谢。

### 挑战与机遇

**我认为Google真正失误的一个领域之一，这可能是十年前或其他什么时候，特别是在Workspace周围，是建立一个生态系统。我想我以前问过你这个问题，事实上，它有点像微软的Lite版本，“我们将做一切”，但微软非常擅长这样做，在硅谷还有许多其他最佳产品，但没有人把它们联系在一起，没有粘合剂让所有东西协同工作。这些教训或经验是否适用于这个新时代？Google如何考虑与其他初创公司或其他实体合作，以便能够将不同的东西粘合在一起？因为这与你所讲述的“看，我们通过自己做很多事情确实有真正的集成优势”有些对比。你是如何考虑这个平衡的？**

**TK:** 我们确实正在与一个生态系统合作，原因在于以下。我们一直说，我们集成东西并不意味着我们是一个封闭的系统。所以最近，由于你提到了Workspace，我们已经有了自动化工作流程的集成。人们最终希望使用协作工具，不仅仅是为了协作，而是他们实际上希望自动化工作流程。因此，我们与许多合作伙伴合作，此次活动中有来自Canva、HubSpot、DocuSign、Adobe等几家公司，我们确实做了详细的工作，使工作流程完全无缝化，你会继续看到我们做更多的工作。

**Sundar Pichai 在他的视频问候中提到，他强调了 AI 初创企业的数量，尤其是使用 Google Cloud 的 AI 独角兽。回到重启的想法，你是否认为 AI 时代是在捕捉下一代公司方面的重新开始？我的意思是，显然，AWS 在一般云计算方面有着巨大的优势，整个移动应用生态系统基本上是在 AWS 上构建的。在企业时代，你必须处理已有的东西，他们已经处理过的东西，你必须有集成。你是否认为自己把这视为一个重要的焦点，“我们将主导这个初创企业时代”？**

**TK：** 是的。顺便说一句，所有这些初创企业都被另外两家追求，事实上，90% 的独角兽和所有 AI 融资初创企业的 60%，在每种情况下都增加了十个百分点，它们是最明智的。我的意思是，坦率地说，对于他们来说，这确实是他们损益表中的最大成本。

**所以驱动因素是什么？**

**TK：** 我们基础设施的效率。

**他们没有所有的传统数据和传统的事物，所以你会觉得在没有那些传统的情况下，你在一个公平竞争的环境中，没有人可以触及你？**

**TK：** 我们不打算筹集一万亿美元来建造超级计算机，因为我们已经建造了十年的超级计算机，这是我们第五代建造这样的设备，所以我们不是第一次在工作中学习。

**Google 是，无论是企业客户还是初创企业，他们都在购买基础设施或者希望使用模型。对于 Google 在消费者空间是否有任何担忧吗？我的意思是，成为答案引擎而不是搜索引擎的一个挑战之一是你只提供一个答案，关于答案应该是什么存在很多争议。你不把这些问题交给用户解决。你是否感到任何关于 Google 成为文化战的反弹或担忧，或者 Gemini，或者其他任何可能的问题？或者这些都相对不重要？**

**TK：** 这完全不重要。

**这是一个资产吗？**

**TK：** 我们以出色的工程技能而闻名。你问了，本，谷歌的本质是什么？谷歌的核心是一家出色的工程公司。我们制造出色的产品，我们知道如何构建全球最可扩展的基础设施。随着人们选择进行人工智能，他们希望有一个曾经有过经验的伙伴，和第一次使用第三方模型并试图推介他们的产品的人不同，我们已经训练了多年的模型，并且多年来集成了我们的产品，并且在可扩展的基础设施上运行了多年。所以每家公司都有问题，我们为自己感到骄傲，客户也为和我们合作感到骄傲。

**我对你们在谷歌云方面的工作印象深刻。上次你来的时候我已经告诉过你。我对你成功地建立起与谷歌独立而不是与谷歌整合的组织印象深刻，想问问，回头看你的任期，你在多大程度上成功了？尤其是考虑到企业市场和支持需求等方面与拥有数十亿用户的消费品非常不同的地方，这其中有推拉作用是什么？**

**TK：** 我们有不同的需求，我们有共同的价值观。所以我们在很多事情上有共同的价值观，公平对待员工，追求卓越，确保我们的团队与谷歌的其他部门效果好地合作，有同情心地合作。归根结底，我们的所有模型都是由我们的团队构建的基础设施提供服务。当你使用搜索时，它就运行在我们的团队提供的基础设施上。当你使用YouTube时，它就是在我们的团队构建的网络上运作。所以我们与谷歌其他部门合作得非常好。

与此同时，在企业环境中工作的必要性要求我们有一些独特之处。举个例子，我们有许多谷歌其他部门没有的功能。他们不需要专业服务，也不需要某种类型的商业律师，我们需要这些。甚至销售团队的工作方式也与我们的销售团队不同，因为他们销售给技术部门，而他们销售给市场营销总监，因此我们的工作内容是不同的。

与此同时，我们通过努力赢得客户和发展业务，赢得了谷歌其他部门的尊重。我们感到非常自豪。

**我认为你应该。会不会有一个未来，几十年后，我们会回头看，然后说，“哦，是的，Google，这家人工智能基础设施公司，他们最开始是一个搜索引擎，信不信由你”？你们所建立的东西会不会成为这个新时代最重要的事情？**

**TK:** 对于我们来说，五年前，没有人看好我们，坦率地说，有人告诉我这一点根本不重要。看待这个问题的一种方式是，“我们在做一些重要的事情吗？”，另一种方式是，“这完全是上升期”，而我们选择了后者。今天，这确实很重要，毫无疑问。当我们看投资者社区时，当我们看客户社区时，当我们看谷歌现在的关系时，因为我们同时为许多公司的技术团队和营销团队提供服务，这确实重要。它会是最重要的事情吗？时间会告诉我们。我学会了在技术领域永远不要做预测。

**（笑）我认为你在日常工作中做了很多预测，但你不会与我分享，这完全没问题。托马斯，感谢你再次出现。我认为这是一个很好的主题演讲。关于AI有些内容我们这里不必讨论，这不是你的领域，它从商业模型的角度提出了基本的挑战，从答案的角度来看，但同时，它与云和企业的紧密结合，似乎你认识到并正在利用相同的事情。**

**TK:** 感谢你抽出时间，本。很高兴与你交谈。

* * *

本次每日更新采访也可作为播客提供。要在您的播客播放器中收听，请[访问Stratechery](https://stratechery.passport.online/member)。

每日更新是为单个收件人设计的，但偶尔转发也完全没问题！如果您想为您的团队订购多个订阅并享受团体折扣（最少5个），请直接与我联系。

感谢您的支持，并祝您有个愉快的一天！

### *相关*
