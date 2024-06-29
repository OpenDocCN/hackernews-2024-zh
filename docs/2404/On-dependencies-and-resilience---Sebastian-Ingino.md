<!--yml

category: 未分类

date: 2024-05-27 13:06:58

-->

# 论依赖性和韧性 - Sebastian Ingino

> 来源：[https://ingino.me/ideas/on-dependencies/](https://ingino.me/ideas/on-dependencies/)

我在构建基于[Astro](https://astro.build)的新网站时，一直在思考依赖性问题。Astro在没有额外包的情况下，使用了总计400个依赖项，约占122 MB。Andrei Kashcha创建了一个[美观的可视化工具](https://npm.anvaka.com/#/)，您可以在其中查看[Astro](https://npm.anvaka.com/#/view/2d/astro)和所有其他包的关系图。我的网站加上所有额外包后，拥有467个依赖项，总计超过500 MB，这简直是疯狂。即使像11ty这样的“轻量级”项目也有213个依赖项。

然而，我不想讨论那些比我聪明得多的人已经谈论过的JS生态系统依赖问题^([1](#user-content-fn-1))。我也不会直接讨论软件膨胀问题^([2](#user-content-fn-2))或供应链安全问题^([3](#user-content-fn-3))。相反，我想探讨的是软件开发文化向更多依赖性转变的理论，特别是在Web开发领域，这种转变正在创造更多依赖闭源软件或API完全功能的产品和公司，这可能会带来未来的负面影响。

# 来自游戏行业的热爱

去年九月，[Unity](https://unity3d.com)宣布^([4](#user-content-fn-4))对开发者推出新的每次下载费用，将从2024年1月开始实施。对于*个人*计划（*Plus*将被整合到*个人*中），过去12个月内收入超过20万美元并且至少有20万次安装的游戏将符合条件；对于更昂贵的*专业版*和*企业版*计划，则分别是100万美元和100万次安装。作为定价变更的交换条件，*个人*计划将免费。*个人*计划的安装费用为每次$0.20，而*专业版*和*企业版*则从每次$0.15和$0.125开始，随着每月安装次数的增加而递减。

Unity之前对每个计划收取订阅费用，此举立即引起了强烈愤怒，主要担忧是该系统可能被滥用以惩罚开发者。然而，最让我震惊的不仅仅是这一举措纯粹出于利润和股票驱动，还有游戏工作室如此迅速地放弃Unity**尽管**后来大部分变更被撤销。一些工作室甚至将Unity描述为“操作风险”^([5](#user-content-fn-5))。

独立开发者受到的冲击最大，因为如此快速地切换软件极其困难，尤其是从库到游戏编辑器再到分发，甚至到编程语言的所有工具都可能发生变化。在这种情况下，Unity确实对其与开发者的声誉造成了无法挽回的损害，但这引发了一个问题：如果他们更加隐秘地做了什么会怎样？如果他们逐渐提高价格而几乎不通知开发者会怎样？尽管“沸蛙”类比是错误的^([6](#user-content-fn-6))，如果水慢慢变热到足够缓慢，开发者或工作室是否不会感到太大的压力去切换引擎，从而允许Unity获取越来越多的利润？

我认为Unity的行动应该提醒不仅是整个公司而是个人开发者，他们需要*意识到*他们存在的闭源依赖关系，并保持一个在需要时可以切换的计划。对于较大的工作室而言，Unity问题是一个较容易解决的问题——他们可以创建内部引擎——但它对较小的工作室和独立开发者造成了最大的伤害。然而，任何规模的任何产品的创作者，在我看来，从第一天起都需要考虑它依赖的内容，无论是YouTube的视频收入还是AWS的托管服务，这些依赖关系有多可信，以及如何在条款变更时作出响应。

# 一个开放的门紧闭了。

即使是开源也不是“安全”的，正如[HashiCorp](https://hashicorp.com)转向^([7](#user-content-fn-7))从MPL 2.0^([8](#user-content-fn-8))到BSL^([9](#user-content-fn-9))所展示的那样，这种变更使源代码保持可用，同时阻止了某些商业使用。在这种情况下，HashiCorp的BSL阻止了商业竞争对手在没有许可证的情况下使用他们的软件。这种转变有两个问题：1\. HashiCorp决定谁是“商业竞争对手”，2\.一些开源开发者担心在他们的工作中使用HashiCorp软件可能会影响他们的许可证，并迫使他们采用HashiCorp的BSL。

对于中大型公司而言，他们在内部使用HashiCorp软件，这可能需要进行法律讨论，甚至进行审计，以确保他们对HashiCorp软件（主要是[Vault](https://www.vaultproject.io)和[Terraform](https://www.terraform.io)）的使用不会与HashiCorp竞争^([10](#user-content-fn-10))。而其他公司，特别是新兴的DevOps初创公司，他们可能无法在没有昂贵许可证的情况下使用这些产品，这正是HashiCorp此次更改试图瞄准的确切产品段。与MongoDB在2018年转向SSPL^([11](#user-content-fn-11))或CockroachDB在2019年转向BSL^([12](#user-content-fn-12))^([13](#user-content-fn-13))相比，HashiCorp的这一变更似乎是开源历史上更为极端的许可证变更之一。

自从Linux基金会赞助了[OpenTofu](https://opentofu.org)，这个永久开源的Terraform分支，保证许可证在未来的任何时候都不会更改。虽然Linux基金会和其他许多开源基金会（如Apache软件基金会）为他们所有的项目提供后勤支持和开源许可证的保证，但很少有小的依赖项落入他们的保护范围。任何规模的现代公司都需要检查他们的依赖项来源和管理者，以确保他们不会面临未来许可证变更的风险。

有公司所有权的开源项目似乎是最冒险的，审查保持软件开源或最终更改其许可证的激励是至关重要的。作为一个强大的开源贡献者，谷歌不太可能撤下，例如[Angular](https://angular.io)的copyleft^([14](#user-content-fn-14))。另一方面，[Automattic](https://automattic.com)的产品如[WooCommerce](https://woo.com)是开源代码产品——未来限制许可证没有任何阻碍。在我看来，最安全的依赖关系是由基金会运行的，然后是有多个公司赞助的，然后是历史上值得信赖的公司。即使是个人运行的开源软件在许可证变更方面也比单一公司所有者更安全。尽管存在这些风险，开源软件几乎总是比封闭源代码更好，因为即使在许可证变更的情况下，社区也可以进行分支，就像OpenTofu一样。

# 处理供应商

例如，[Stripe](https://stripe.com)在过去90天内的可用时间为99.999%^([15](#user-content-fn-15))。我在Stripe时，[Slack](https://slack.com)在这十二周内**两次**下线^([16](#user-content-fn-16))，当时令人沮丧，但他们在上个季度保持了99.98%的正常运行时间^([17](#user-content-fn-17))。尽管很少出现故障，Slack的故障通常会严重影响办公生产力和内部通信，如果没有其他即时通讯工具是个麻烦。幸运的是，大多数公司也有电子邮件，Slack只是临时的不太专业的替代品。

但是，如果你的产品或业务依赖关系中的一个像[Square](https://squareup.com)去年九月那样出现故障会怎么样？Square全球停机近两天，导致10亿美元的销售损失，成为客户转换供应商或自主开发的警钟^([18](#user-content-fn-18))。[AWS](https://aws.amazon.com)同年六月短暂停机，影响了其US-EAST-1区域的运营^([19](#user-content-fn-19))。[Cloudflare](https://cloudflare.com)在去年十月底全面停机约半小时，之后不到一周的时间内出现了一次较小但持续时间更长的停机^([20](#user-content-fn-20))。

我认为我们每天看到的所有故障都可以得出一些明显的教训。虽然这可能是昂贵或非常困难的，但在大多数情况下，为所有规模的公司维护一个次要提供商是最安全的选择。然而，这可能过于复杂，特别是在使用高度集成的 API 时。因此，重要的是在这些 API 的顶部保持一致的抽象，以便即使没有自动回退系统，更换底层软件也应该更容易。

此外，我认为软件设计师和开发人员应该考虑*失效安全*与*失效保护* - 即，在紧急情况下是否应该解锁某些内容或反之。大多数软件都是失效安全的：如果许可证管理器无法连接主机，您将无法登录；如果支付系统故障，您将无法购买产品；如果身份验证提供者未响应，您将无法登录。这通常是最佳默认选项 - 通常情况下，您不希望有人关闭 WiFi 以跳过许可验证或者通过 DDoS 攻击 API 以获取免费商品。然而，在某些情况下，编写失效安全的代码可能是您的业务更好的选择。比如说，如果您运行一个通过 API 进行订阅的网站，并且该 API 停止运行，那么订阅者应该继续保持订阅状态吗？我会说是。这是一个相对简单的案例研究，但它作为每个无法控制的依赖关系的一个简单案例。

最后，中型到大型公司（甚至可能是较小的公司）应定期进行混沌测试和故障测试。Cloudflare 多日的停机是由他们俄勒冈州一个主要数据中心的电力问题引起的，本应该通过高可用多数据中心集群进行预防^([21](#user-content-fn-21))。

> 特别是，处理日志并驱动我们的分析的两个关键服务 - Kafka 和 ClickHouse - 仅在 PDX-04 可用，但有依赖于它们的服务在高可用集群中运行。这些依赖关系不应该那么紧密，应该更优雅地失败，并且我们应该早点察觉到它们。

虽然 Cloudflare 进行了相当强大的故障测试，但他们并没有关闭整个数据中心 - 只是部分数据中心，因此没有发现问题。你应该[阅读整篇博文](https://blog.cloudflare.com/post-mortem-on-cloudflare-control-plane-and-analytics-outage/)，因为它详细说明了 Cloudflare 将来将如何使用严格的测试和预防措施，但我认为它仍然是所有公司都面临的一个例子。AWS 现在允许您模拟区域故障^([22](#user-content-fn-22))，我希望更多的供应商能够跟上这个问题的发展。

# GPT-4 / Claude / Llama / 5 分钟前的新模型

让我开始思考这个问题的是“AI”产品的增长，它们只是简单地包装了[OpenAI 的 API](https://openai.com/blog/openai-api)，提供专有的嵌入或作为额外服务的提示工程。所有之前的例子都是关于部分依赖，即使依赖变化，顶层的大部分工作也可以被挽救。我担心许多基于 GPT 的应用正在使自己对 OpenAI 过度依赖，为 OpenAI 创造了大量的议价权，并为产品带来了很多风险。此外，由于许多这些应用只是添加了像 PDF 这样的新嵌入方法，这使得 OpenAI 像他们去年九月那样轻而易举地进行 *sherlocking*^([23](#user-content-fn-23))。

在一天结束时，我认为我们每个依赖只需要问两个问题：

如果它今天消失了对我的业务有什么影响？这种情况发生的可能性有多大？

第一个问题有一堆相关问题 - 其中一些我已经讨论过：如果出了问题我们有没有应对计划；切换需要多长时间；能否自动化这个过程；失败的优雅程度如何；等等？第二个问题也有其他问题：谁是这个依赖的背后人物；他们的记录如何（运行时间，行为等）；类似依赖做了什么；等等？不过，我认为这些简单的问题足以理解基本的危险，并希望能启动内部努力来减少潜在影响。你甚至可以把它画成一张图表：

<svg xmlns="http://www.w3.org/2000/svg" aria-roledescription="quadrantChart" role="graphics-document document" viewBox="0 0 500 500" id="mermaid-0"><g class="main"><g class="quadrants"><g class="quadrant"><text transform="translate(379, 147) rotate(0)" text-anchor="middle" dominant-baseline="middle" font-size="16" fill="#111111" y="0" x="0">🚨 尽快移开</text></g><g class="quadrant"><text transform="translate(147, 147) rotate(0)" text-anchor="middle" dominant-baseline="middle" font-size="16" fill="#0c0c0c" y="0" x="0">解耦和抽象</text></g><g class="quadrant"><text transform="translate(147, 379) rotate(0)" text-anchor="middle" dominant-baseline="middle" font-size="16" fill="#070707" y="0" x="0">目前安全</text></g><g class="quadrant"><text transform="translate(379, 379) rotate(0)" text-anchor="middle" dominant-baseline="middle" font-size="16" fill="#020202" y="0" x="0">自动回退</text></g></g><g class="labels"><g class="label"><text transform="translate(147, 5) rotate(0)" text-anchor="middle" dominant-baseline="hanging" font-size="16" fill="#111111" y="0" x="0">低影响</text></g><g class="label"><text transform="translate(379, 5) rotate(0)" text-anchor="middle" dominant-baseline="hanging" font-size="16" fill="#111111" y="0" x="0">高影响</text></g><g class="label"><text transform="translate(5, 379) rotate(-90)" text-anchor="middle" dominant-baseline="hanging" font-size="16" fill="#111111" y="0" x="0">低可能性</text></g><g class="label"><text transform="translate(5, 147) rotate(-90)" text-anchor="middle" dominant-baseline="hanging" font-size="16" fill="#111111" y="0" x="0">高可能性</text></g></g></g></svg>

许多公司正在从直接依赖 OpenAI 转向其他提供商，如[Hugging Face](https://huggingface.co)^([25](#user-content-fn-25))。我甚至记得在旧金山湾区的高速公路上看到过一则广告，尽管我找不到公司名字，但是它推广的产品可以让你在多个语言模型提供商之间切换，以提高产品的稳定性。这个产品的讽刺之处在于现在它们自己也成为了一个依赖。

# 解决方案？其实并没有。

我们不会很快摆脱这个相互依存的世界。除非你是世界上最大的公司，并且已经完全垂直整合（苹果的头等愿望），否则总会有依赖关系，而这是好事。有像[Twilio](http://twilio.com)这样的公司负责通讯，以及像[Mapbox](https://mapbox.com)这样的公司负责地理信息系统，这样的公司可以实现规模经济并抽象复杂性，这对一些大公司来说可能过于困难。有像[React](https://react.dev)和[Hadoop](https://hadoop.apache.org)这样的开源软件，它们也抽象了复杂性并经常拥有强大的社区，而且都是免费的。

尽管你尽了最大的努力，你总是会有依赖关系。如果你是一家产品公司，即使你的物流是内部完成的，天气也会成为一个依赖项。对于大多数公司来说，整个互联网甚至是一个重要的依赖项。在某个时候，我们不能为了那些超出我们控制范围的事情而担心，我们只能为最有可能发生的情况做计划。我希望这篇博客文章能在你的产品团队和公司内引发关于如何构建弹性软件的讨论，因为如果出了什么问题，这可能决定你的产品的成败。

有什么想法吗？[给我留下反馈！](/contact)

* * *
