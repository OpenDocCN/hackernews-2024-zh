<!--yml

类别：未分类

日期：2024-05-27 14:36:12

-->

# Meta切断第三方访问Facebook群组，使开发者和客户陷入混乱 | TechCrunch

> 来源：[https://techcrunch.com/2024/02/05/meta-cuts-off-third-party-access-to-facebook-groups-leaving-developers-and-customers-in-disarray/](https://techcrunch.com/2024/02/05/meta-cuts-off-third-party-access-to-facebook-groups-leaving-developers-and-customers-in-disarray/)

最近的突然宣布，Meta将很快关闭其Facebook群组API，使一些企业和社交媒体营销人员感到困惑。

在2024年1月23日，Meta[宣布发布](https://developers.facebook.com/blog/post/2024/01/23/introducing-facebook-graph-and-marketing-api-v19/)其Facebook Graph API v19.0，其中包括公司将淘汰其现有的Facebook群组API的消息。Meta表示，后者被开发者和企业用来安排在Facebook群组中发布帖子，将在90天内移除。这还包括与API相关的所有权限和可审查功能，它还指出。

Meta解释说，API的一个主要用例是一个功能，允许开发者在Facebook群组中[私密回复](https://developers.facebook.com/docs/messenger-platform/discovery/private-replies/)。例如，一家小型企业希望向在其Facebook群组中发帖或评论的人发送单一消息，可以通过API发送消息。然而，Meta表示，新的v19.0 API中的另一个变化将使这一功能成为可能，而不需要使用群组API。

但开发者告诉TechCrunch，关闭API将导致那些为希望安排和自动化其社交媒体帖子的客户提供解决方案的公司遇到问题。例如，VipeCloud的CEO亚当·彼得森解释说，该公司提供一套工具来安排社交媒体帖子，API的关闭将对他的业务产生“显著影响”，因为大约8%的总收入将受到影响。他指出，他的公司服务于大约5000个Facebook账户，主要是女性企业家的账户。

这些客户依靠VipeCloud对Facebook的API的访问，既可以公开发布到他们的Facebook页面，也可以私密地发布到群组以与他们的团队沟通。他说，这些小企业将私密群组用作一种类似Slack的替代工具。

“我们的每一个客户都在感到恐慌，”Peterson说。

其他使用群组API的客户可能依赖于由业务代理商安排的自动化，其中一些将因API的关闭而受到不成比例的影响。

Peterson解释说，客户经常依赖机构来处理他们发布的各个方面，比如团队建设或团队激励。“这些机构，这是他们的全部业务。这是他们的生计。”他补充道。

这一举措也影响了 VipeCloud 的竞争对手，通常是非风险投资公司，他们构建特定市场服务，收入可能在数百万到低两位数百万美元之间。

“有些其他公司——它们将被淘汰，” Peterson 指出。“即使我们与它们竞争，这也不是一件愉快的事情。你宁愿靠服务、产品或其他方式取胜，” 他继续说道。“这是实时的平台风险。”

一家名为 [PostMyParty](https://postmyparty.com/) 的公司，帮助社交销售和其他人安排和自动化在线聚会，表示 API 的关闭将使该公司破产。

“我将失去七年的工作和超过 10,000 名客户，” 所有者 Daniel Burge 告诉 TechCrunch。“数百万美元的损失。更不用说对所有依赖我们软件的客户的影响了，” 他补充道。

Burge 表示，PostMyParty 被小微企业使用，包括在 Facebook 群组中进行在线健身营的健身教练、从家工作的妈妈从事社交销售以及具有教练组或客户组的其他人。

企业家指出，这不是 Meta 第一次这样做。

“几年前，[Meta](https://wiki.example.org/meta)突然终止了它们的事件 API，完全没有提前通知，” Burge 说道。“某天我们发现一切都崩溃了，我们收到了成千上万个来自客户的支持请求，那时几乎摧毁了我们的业务。”

更重要的是，开发人员告诉我们，Meta 关闭 API 的动机尚不清楚。一方面，可能是因为 Facebook 群组不产生广告收入，关闭 API 将使开发人员无法绕过这一点。但 Meta 并没有澄清这是否是情况。相反，Meta 的博客文章只提到了新的 v.19.0 API 将解决的一个用例。

Maurice W. Evans，一名 Meta 认证的社区管理员，认为这一举措将对小企业、开发人员和数字市场构成挑战，但也代表着“Meta 运营理念的重大转变”。

“取消第三方访问 Facebook 群组可能会显著改变数字景观，为社区管理员和企业提供了障碍和机会。作为 Meta 认证的社区管理员，我亲眼见证了这些工具对促进充满活力、积极参与的在线社区的价值。这一变化强调了我们战略中适应性和创新的需求，” Evans 告诉 TechCrunch。

在社交媒体的其他地方，网站设计公司 Archer Web Design [称 API 关闭的消息“毁灭性](https://twitter.com/ArcherWebsites/status/1752687220639650066)”并表示“企业和社交媒体营销人员将会回到石器时代！”，他们在 X（曾是 Twitter）上发表的帖子中写道。

在 Meta 的开发者论坛上，[一位开发者表示](https://developers.facebook.com/community/threads/2076398286093257/) 他们对公司的公告感到“相当震惊”，指出他们的应用依赖于 Groups API，当关闭发生时基本上将无法继续工作。

其他人对 Meta 没有清楚地解释未来是否会使用页面访问令牌发布组中的帖子感到沮丧，因为公告的措辞似乎只与发布私人回复的部分相关，而不适用于整个群组的发布。例如，Burge 怀疑整个事情可能只是某种消息传递错误 —— 就像 Meta 可能忘记包含说明其新解决方案的部分一样。

然而，有人担心 Meta 最近关闭了其[开发者漏洞门户](https://developers.facebook.com/support/bugs/)，表明其正在降低开发者利益的优先级。

截至撰写本文时，Facebook 的代表们尚未在其论坛上回复开发者的评论，使所有人都感到一片茫然。

[另一位开发者在论坛中感叹](https://developers.facebook.com/community/threads/584177870580474/)，“这影响了我的正在进行的项目和即将启动的项目。我不知道该怎么办。”

*本故事正在发展中。已经要求 Meta 进行评论，我们将在了解更多信息后更新。*
