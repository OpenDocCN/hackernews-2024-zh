<!--yml

category: 未分类

date: 2024-05-29 13:25:04

-->

# Forgejo 完全脱离了Gitea [LWN.net]

> 来源：[https://lwn.net/SubscriberLink/963095/85db4f864c060d22/](https://lwn.net/SubscriberLink/963095/85db4f864c060d22/)

| **LWN 订阅者的好处**[订阅 LWN](/subscribe/) 的主要好处是帮助我们继续发布，但除此之外，订阅者可以立即访问所有网站内容并获得许多额外的网站功能。请今天就注册！ |
| --- |

作者：**乔·布罗克迈尔**（Joe Brockmeier）

2024年2月23日

开源世界中的"[forges](https://en.wikipedia.org/wiki/Forge_(software))"领域正变得更加分散。[Forgejo](https://forgejo.org/)项目是一个软件开发平台，它在2022年末作为[Gitea](https://about.gitea.com/)的"<q>软件的</q>"分支开始。2024年2月16日，Forgejo [宣布](https://forgejo.org/2024-02-forking-forward/)其意图成为Gitea的"<q>硬分支</q>"，以继续其社区控制开发和"<q>解放软件开发免于专有工具的枷锁</q>"的目标。在专有工具的阴影长时间笼罩开源开发的世界中，这些话语是受欢迎的——如果项目能够交付的话。

Gitea曾经（至今）是一个流行的开源Git托管和协作平台。Gitea本身是[Gogs](https://gogs.io/)的一个分支，起源于2016年12月，因对Gogs的"<q>单一维护者管理模式</q>"感到沮丧，选择了"<q>更开放和更快速的开发模式</q>"。最初在其 `[CONTRIBUTING.md](https://github.com/go-gitea/gitea/blob/release/v1.8/CONTRIBUTING.md)` 文件的早期版本中，项目的治理规定了项目将有三个"所有者"负责保持"<q>健康的</q>"开发。实际上，所有者的责任和权利并不明确，但他们将由Gitea的维护者年度选举产生，他们将通过简单的投票模式决定哪些贡献被接受，并决定谁将担任所有者角色。

当 Gitea 的域名和商标被两位 Gitea 的 "所有者"（Lunny Xiao 和 Matti Ranta）悄悄转移到了一个名为 Gitea Ltd. 的营利性公司时，维护者社区并没有参与投票。[公告](https://blog.gitea.com/open-source-sustainment/)称此举是为了通过允许无法通过赞助或代码贡献向开源项目回馈的公司直接向 Gitea Ltd. 支付来实现 "<q>确保项目的长期成功</q>" 的目标。此外，该公司将 "<q>作为项目的管理者</q>"，项目 "<q>当然仍然是开源的，是一个社区项目</q>"。到目前为止，该公司似乎在保持关于 Gitea 本身仍然开源方面做到了这一点。它推出了一个托管云服务，提供了自托管版本中没有的功能，但 Gitea 的自托管版本仍然是开源的。公告中提到的 "<q>增强版</q>" 企业版尚未出现。

不出所料，Gitea 社区的成员们提出了反对意见。他们于 2022 年 10 月 25 日发布了一封[公开信](https://gitea-open-letter.coding.social/)（随后在公开信上方进行了更新），并要求 Gitea 的所有者们改为成立一个非营利组织来持有域名和商标。一旦[明确表示](https://blog.gitea.com/a-message-from-lunny-on-gitea-ltd.-and-the-gitea-project/)不会发生这种情况，社区宣布将创建[软分叉](https://gitea-open-letter.coding.social/updates/october-31th/)的 Gitea，以 "<q>保持 Gitea 社区的团结</q>"。Forgejo 项目于 2022 年 12 月 15 日在 Codeberg e.V. 的主持下[宣布启动](https://blog.codeberg.org/codeberg-launches-forgejo.html)，Codeberg e.V. 是总部位于德国柏林的非营利组织，提供 [Codeberg](https://codeberg.org/) 协作平台。

Forgejo 这个名字来自于[Esperanto](https://codeberg.org/forgejo/governance/src/branch/main/README.md)中锻造的意思。通过之前的经验教训，社区安排由 Codeberg e.V. 持有 Forgejo 的域名，并且项目有详细的文档，涵盖了项目治理的各个方面，如决策、品牌、合并拉取请求、使命、团队以及资金处理方式。虽然 Gitea 在开发中[仍使用 GitHub](https://github.com/go-gitea/gitea/issues/1029)，但 Forgejo 已经在 Codeberg 平台上开发，并成为 Codeberg 的基础软件。

#### 为什么现在？

自 2022 年 12 月推出以来，Forgejo 一直在 Gitea 开发分支上作为[一组提交](https://codeberg.org/forgejo/forgejo/milestones?state=closed&q=cleanup)进行开发——在功能分支上进行工作，并定期在 Gitea 上进行变基。今年年初，Forgejo 贡献者 Earl Warren [发起了讨论](https://codeberg.org/forgejo/discussions/issues/96)，希望改变 Forgejo 的开发工作流程，并“<q>变得独立，并从 Gitea 代码库中精选，直到差异变得太大而无法再这样做</q>”。Warren 认为现有的工作流已经不再合理，因为 Gitea “<q>合并了非常大的未经测试的代码（功能或重构），这些代码导致了对用户影响较大的重大回归</q>”。他还提到，简化的工作流将提高稳定性，并使贡献者更容易管理单一开发分支而不是多个功能分支。Warren 还声称，Gitea 已经“<q>变成了开放核心</q>”，并对其“<q>安全行为</q>”表示关注，但没有详细说明。

在讨论之后，Warren 在 1 月 14 日提出了[一项提案](https://codeberg.org/forgejo/governance/issues/58)，计划在 Forgejo 的下一个重要版本发布后或者 3 月 1 日之前，“<q>以较早者为准</q>”改变 Forgejo 的工作流程。在提案中，他还辩称 Forgejo 拥有独立运作的资金、资源和治理，并且这一改变“<q>不会妨碍不存在的合作</q>”。尽管 Forgejo 曾吸收 Gitea 的工作，Warren 写道，Gitea 的贡献者“<q>从未从 Forgejo 中精选过提交</q>”。至少，Warren 基于 Forgejo 的工作在 [Gitea](https://github.com/go-gitea/gitea/issues?q=earl-warren) 提交了一些贡献。

硬分叉与软分叉究竟意味着什么？在其 2 月 16 日的公告中，Forgejo 项目解释说，软分叉意味着 Forgejo 包含“<q>所有 Gitea 的内容，无论好坏</q>”，因此用户可以将 Gitea 迁移到 Forgejo 作为直接替代。这些日子尚未结束，但尾声已近。计划从 Gitea 迁移到 Forgejo 的用户应尽早行动，“<q>因为最终，Forgejo 将不再是直接替代品</q>”。Forgejo 打算至少在现有情况下与 Gitea API 兼容。公告称，任何对 API 的更改将经过仔细评估，并且 Forgejo 将尽量保持与 Gitea 的兼容性——尽管贡献者可以自行决定是否实施新的 API。公告指出，Forgejo 的部分内容已经开始分化，例如文档、本地化和[Forgejo Actions](https://forgejo.org/docs/v1.21/user/actions/)持续集成系统。

#### 现在怎么办？

Gitea Ltd没有直接回应Forgejo的声明。然而，它确实在Forgejo计划中公开的“硬分叉”声明前一天（即2月14日），为Free Software Foundation Europe (FSFE)的“[我爱自由软件日](https://fsfe.org/activities/ilovefs/)”发布了一篇[博客文章](https://blog.gitea.com/i-love-fs-day/)。该文章指出几个组织“使用Gitea构建自由或其他开源软件”，包括FSFE、Blender Foundation、openSUSE等。

这两个项目似乎有足够的动力继续前进，尽管可以推测，用户、贡献者和客户是否有足够的兴趣来无限期地支持这两个项目。例如，Gogs自Gitea分支以来似乎并没有繁荣发展。GitHub上的Gogs存储库显示有最近的活动，但截至撰写本文时，Gogs的[最后一个发布](https://github.com/gogs/gogs/releases/tag/v0.13.0)是在2023年2月25日，几乎一年前。从其主页链接的演示出现错误，主页上的通讯链接指向404页面，而非有关该项目的当前新闻。

考虑到Forgejo得到了Codeberg e.V.的支持（和使用），似乎有望长期保持活力。作为一个平台，Codeberg已经吸引了可观的追随者，拥有超过117,000个项目和95,000名用户。虽然可观，但与GitHub数百万项目和用户相比只是一个小样本。除非从Gitea大规模迁移到Forgejo，否则似乎Gitea也将以大致相同的速度继续发展。Gitea曾在今年一月宣布自己在GitHub上获得了40,000颗星的荣誉，这一指标据称显示了Gitea在“全球开发者社区中的重要性”，以及“全球开发者社区对Gitea的信任和信心”。在援引寻求取代的平台上取得成功，确实有一些讽刺意味。

理论上，使用[ForgeFed协议](https://forgefed.org/)的网站可以通过简化软件锻造之间的协作来帮助取代专有的替代品。如果真正实施的话。ForgeFed协议是允许软件锻造无需用户在每个锻造上创建帐户而相互交互的[ActivityPub](https://www.w3.org/TR/activitypub/)扩展。在关于2023年初联邦化的[帖子](https://forgejo.org/2023-01-10-answering-forgejo-federation-questions/)中，Forgejo项目描绘了一个挑战GitHub的开放生态系统的美好画面。

> [GitHub](https://github.com/)是封闭花园的典范。我们希望创建类似的东西，但在一个开放的生态系统中，所有内容都使用ForgeFed协议，这样你就不会被你使用的锻造或代码协作软件所困扰。

在实践中，联合体可能需要几年时间才能被实施或采纳。该文章暗示联合体可能会包含在1.19版本中，并声称Forgejo打算向上游贡献联合体至Gitea。然而，联合体并未出现在1.19版本中，而Forgejo上追踪联合体实施的[问题](https://codeberg.org/forgejo/forgejo/issues/59)显示还有很多工作要做。ForgeFed网站仅列出一个名为[Vervis](https://vervis.peers.community/browse)的项目，它已实现工作，但该项目似乎还不准备好投入实际使用。

Gitea和Forgejo试图解决同一个问题——提供一个自托管的开源软件开发平台，该平台可以提供开发者所需和想要的所有功能。Gitea选择放弃真正的社区治理，试图通过提供商业支持来维持开发，并愿意在专有平台上进行工作并接受非开放的提议。Forgejo选择了另一条道路，拒绝使用专有工具，试图建立一个社区主导的、非盈利的方式来维持自身。

这两条道路都面临着许多挑战和障碍。Gitea和Forgejo可能会竞争同样的用户、贡献者和心智份额。在一年或三年后回头看，看看它们的成功程度和它们对将开源开发者从专有平台转移的影响，将会很有趣。

* * *

(

[登录](https://lwn.net/Login/?target=/Articles/963095/)

用于发布评论)
