<!--yml

category: 未分类

date: 2024-05-27 13:03:45

-->

# The threat to open source comes from within

> 来源：[https://newsletter.goodtechthings.com/p/the-threat-to-open-source-comes-from](https://newsletter.goodtechthings.com/p/the-threat-to-open-source-comes-from)

一个伟大的文明不会被外部征服，直到它在内部毁灭。

-Will Durant

The viability of open-source software was challenged twice over the past couple of weeks. One attack came from an outside adversary, the other from within the community itself. While the external threat triggered wider consternation, the internal threat seems to me far more dangerous.

*NB: while I have created and contributed to various open-source projects over the years, I am certainly not an expert on OSS governance. Today’s edition of Good Tech Things is not intended to be “thought leadership” or whatever, just a tool for me to organize my own thoughts.*

如果没有人听说过开源软件，而你今天写了一篇本科论文，提议以此方式开发驱动世界的软件，你会得到F减。

**Thesis sentence of your undergrad paper**: *Random people will anonymously create much of the world’s critically important software in their spare time, companies will make billions of dollars by using it for free, and this situation will remain shelf-stable for decades.*

**红笔批注的教员**：*这毫无意义。是什么驱使任何自重的软件开发者参与这样的计划？你怎么知道生成的代码会有多好？*

-   **你本科论文的论点1**：*痴迷的书呆子们会为了确保代码尽可能好而互相撕裂，因为他们无法自拔。*

-   **红笔批注的教员**：*这听起来不像是一个可持续的状态。*

**Point 2 of your paper**: *It works for Wikipedia.*

-   **红笔批注的教员**：*在学术环境中，你不能引用维基百科。F减。*

Anyway, open-source—like Wikipedia and the BGP protocol—is one of those networked-collaboration things that makes no sense on paper, but in practice is more resilient than any centrally-managed system could ever be. Not because of some brilliant technical innovation, but because it’s an intensely *human* process. It works thanks to the pride and determination and sheer bloody-mindedness of beautiful weirdos all over the world.

The XZ thing put that messy, human resilience on full display. Here are [the facts](https://www.wired.com/story/xz-backdoor-everything-you-need-to-know/) as I understand them:

+   有一个名为XZ的开源数据压缩库，它出现在流行的Linux发行版中，如Debian、Fedora和Ubuntu。

+   XZ’s maintainer, Lasse Collins, was [evidently somewhat burned out](https://robmensching.com/blog/posts/2024/03/30/a-microcosm-of-the-interactions-in-open-source-projects/).

+   一个 [可能是国家支持的](https://x.com/hackerfantastic/status/1773864354439417983) 恶意行为者开始通过GitHub的问题和评论骚扰Collins，使用多个身份要求他更加关注该项目。

+   与此同时，恶意行为者使用另一个身份（“Jia Tan”），在数月时间内逐渐获得Collins的信任，最终说服他让Jia Tan作为XZ的维护者帮忙解决所有的抱怨。

+   Collins对帮助感激不尽，允许Jia Tan向项目提交代码。

+   Jia Tan在XZ中放入的一些代码是一个后门，会使对手获得EVERY SERVER RUNNING THE COMPROMISED VERSION OF XZ的SSH访问权限。

只有当微软员工和 [认证的传奇](https://x.com/vxunderground/status/1774071339671794134) Andres Freund 在运行Postgres基准测试时注意到SSH调用消耗了额外的CPU，并将其追溯到XZ时，后门才被发现，并发出警报。

当社交媒体上 [全都开始慌乱](https://x.com/eigenrobot/status/1773960746826649758)，每个人都在想象，如果说的那个后门幸存下来足够长时间，供有国家支持的对手在全球每台Linux服务器上获得SSH权限，会发生什么。

这正是你那位手持红笔的教练担心的场景。孤立的、无偿的开源维护者如何抵御那些有耐心运行多年社会工程活动的技术娴熟、资源雄厚的国家行为者？可能几乎没有机会。

*但这忽略了关键点。* 正如Mark Atwood所说，[攻击并非因为XZ是开源软件而企图](https://x.com/_Mark_Atwood/status/1774935413834367445)，[攻击](https://x.com/_Mark_Atwood/status/1774935413834367445) *[失败](https://x.com/_Mark_Atwood/status/1774935413834367445)* [正是因为它是开源软件](https://x.com/_Mark_Atwood/status/1774935413834367445)。

你知道当国家行为者想要攻击封闭源软件时会做什么吗？他们只需[在软件公司受雇](https://www.wsj.com/politics/national-security/u-s-charges-chinese-national-with-stealing-ai-secrets-from-google-5c66524a)，然后通过减少的审查来实施他们的卑鄙手段。阳光是最好的消毒剂，而开源则是太阳能驱动的。

XZ的攻击显然暴露了开源软件的长期问题：维护者的过度使用，[调整资金激励的难度](https://x.com/lorenc_dan/status/1775285656383639778)，等等。但总体而言，我认为这是开源展示其最佳功能的一个美好例子：围剿问题，社会化解决方案，然后在几天内向其他项目传播[更好的实践](https://github.com/chainguard-dev/bincapz)。

在我看来，对XZ事件的回应中最不给力的部分实际上来自GitHub本身，他们[暂停了项目页面](https://boehs.org/node/everything-i-know-about-the-xz-backdoor)，这让社区更难以重建攻击的范围和时间线。

这种做法似乎在指向真正的问题所在。

*再次说明，我既不是律师，也不是开源政策专家，所以如果我有什么理解错误，请随时告诉我。*

外部威胁，正如XZ事件所展示的，似乎对更广泛的开源社区产生了激励作用。在这类压力下，开源软件仅变得更加强大。

现在我最担心的事情正好相反：一种冷冻效应。这种效应在过去十多年里悄然向开源软件社区靠近，如同一块冰川。

让我稍微绕个弯，进行一个快速的历史课。这很重要，我保证。

+   **2000年晚期/2010年初期：** **开源作为损失领先者。** 像MongoDB和Elastic这样的公司通过免费提供他们的软件来赢得开发者的喜爱，对于开源软件，他们有一个相当不错的商业模式：他们还销售作为服务的托管版本，以及商业支持等。

+   **2010年中期：大多数的主机。** 猜猜谁在卖作为服务的托管软件和商业支持方面真的很厉害？云服务提供商！突然之间AWS有了Elasticsearch服务，Redis服务等等。他们的网络效应吸引了更多客户资金进入云服务提供商，绕过实际创建开源软件的公司。开源软件公司对此越来越感到沮丧。

+   **2010年晚期至2020年初：终结许可证。** 开源软件公司用他们唯一的武器反击：他们软件的许可条款。MongoDB切换到服务器端公共许可证（SSPL），Elastic和[最近](https://redis.com/blog/what-redis-license-change-means-for-our-managed-service-providers/)Redis也是如此。Hashicorp为其项目如Terraform和Vault选择了商业源代码许可证（BSL或BUSL）。新的条款不再那么“开源”，而是“源代码可用”：它们专门设计用于阻止其他公司（咳咳，AWS）通过托管他们的软件来赚钱。

+   **2020年中期：我们分叉吧。** 开源社区并没有请求发生任何事情，（在某种程度上还被云服务提供商[支持](https://aws.amazon.com/blogs/opensource/why-aws-supports-valkey/)），他们唯一能够做的是：分叉项目并在原始开源许可条款下构建平行版本。Terraform孵化了[OpenTofu](https://opentofu.org/)，由Linux基金会支持；Redis正在获得名为[Valkey](https://github.com/valkey-io/valkey)的分支。

+   **当前局势：** 每个人都在彼此生气。

在这种背景下，[一篇非常好奇的文章](https://www.infoworld.com/article/3714980/opentofu-may-be-showing-us-the-wrong-way-to-fork.html)上周由MongoDB的副总裁Matt Asay发布。该文章通过暗示而非直接指控，表明OpenTofu项目正在从新的BSL许可证版的Terraform中复制代码回到他们的开源分支中。

我应该明确一点，我喜欢并尊重Matt；多年来，他和我在很多事情上都有共识。与几乎所有参与此讨论的人不同，他还接受过律师培训。因此，我不愿仅仅因为他为一家源代码可用软件公司工作就轻视他的担忧。

但我也不能把这件事轻描淡写。Hashicorp显然支持Matt；他们已经向OpenTofu的维护者们发送了一封[正式的制止信函](https://www.linkedin.com/posts/opentofuorg_opentofu-project-was-recently-made-aware-activity-7182147077496344576-jsDQ/)。至少还有一位OpenTofu的批评者[认为](https://www.linkedin.com/pulse/opentofu-independent-work-copyright-infringement-mark-tinderholt-6usqe/?trackingId=cQINUaOXTHKuYVNS%2BkSv1w%3D%3D)，涉及的代码确实看起来非常像Hashicorp的新功能。整个情况变得更加奇怪，因为OpenTofu的新文件中包含了Hashicorp的版权，Matt觉得这很可疑，但OpenTofu的维护者称他们[出于谨慎而包含进来](https://twitter.com/Yantrio/status/1775581822589559195)，因为他们正在大规模移动代码。再次强调，这里并没有涉及很多律师。

在争议的另一面，有许多愤怒的OpenTofu支持者说这是[一场打击](https://twitter.com/bcantrill/status/1775962870762844212)，[当然代码看起来相似](https://twitter.com/adamhjk/status/1776264677301027057)，因为只有一种合理的实现方式等等。回顾我们的历史课程，值得注意的是，一些最[响亮的捍卫者](https://x.com/_Mark_Atwood/status/1775923047478034672)OpenTofu的人都在AWS工作。

我无法知道OpenTofu是否窃取了Hashicorp的代码。Joe Duffy认为，作为一个其目标本质上是跟上Terraform路线图的项目，[无论怎样都处于一个困难境地](https://x.com/funcOfJoe/status/1776479584613171288)。OpenTofu表示他们将很快发布一些东西来澄清他们的清白，但我怀疑他们不会说服任何不已经支持他们的人。

我想我能看到几种可能的走向：

1.  OpenTofu清楚地展示他们是如何独立设计出有争议的代码的，而Hashicorp的律师则后退。如果是这种情况，我期待并希望Matt会撤回他的文章并道歉。

1.  OpenTofu的解释引发了更多问题，而不是回答了多少问题，Hashicorp决定与Linux基金会进行一场恶劣的法律斗争。

1.  没有令人信服的证据证明任何一方是对的，所以没有人退缩，但也不值得打官司，所以每个人都只是彼此抱怨，心怀怨恨，准备下次再战。

我觉得这是一个双输的局面。无论结果如何，像这样的案件——我预计随着像OpenTofu和Valkey这样的更多开源软件项目的启动，“影子”源代码版本将变得更加普遍——对每个人都有冷却效应。企业，这些年来一直在努力信任开源，现在有了一个新理由担心开源软件可能使他们面临像Hashicorp这样的公司的法律行动。开发者犹豫是否要分叉可能某一天会变成开源的项目。贡献者不想因为有人以恶意主张他们抄袭代码而被解雇或被起诉。如果他们因此而疲惫不堪，消失无踪，对开源解决方案变得不太热衷，你能责怪他们吗？

如果我是一个思想领袖，我会发出一些声音宏大的呼吁，要求公司“做得更好”，或者要求开源软件基金会“重新审视他们的治理结构”，或者其他什么。但这是一个不能用陈词滥调解决的僵局。云公司基本上把开源视为可以利用的东西；开源软件公司则认为这与可持续商业模式不兼容。那些不属于任何一方的贡献者正在被践踏。最终，随着生态系统的分裂，每个人都将失去。整个情况实在令人悲哀。

最终，开源软件这种不太可能的人格和激励的奇特融合，推动了21世纪科技创新引擎的平衡，只有当社区认为维持这种平衡比它值得的麻烦更多时，它才会被破坏。这是我们所有人都应该关注的威胁。

* * *

Pluralsight的技术技能日即将在4月25日举行。我将与Faye Ellis进行一场炉边谈话，回顾云简历挑战的4年历程；你可以在[这里免费注册](https://www.pluralsight.com/techskillsday)参加这场对话以及许多其他技术技能的提升讲座。

* * *

我想这或多或少总结了这场讨论：

还在这里发布新的漫画作品[这里](https://www.goodtechthings.com/)！
