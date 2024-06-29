<!--yml

类别：未分类

日期：2024-05-27 13:38:25

-->

# Nix 社区的领导危机 [LWN.net]

> 来源：[https://lwn.net/SubscriberLink/970824/0d89c6d83efad1e0/](https://lwn.net/SubscriberLink/970824/0d89c6d83efad1e0/)

| **你知道吗……？**LWN.net 是由订阅者支持的出版物；我们依赖订阅者来维持整个运营。请通过[购买订阅](/subscribe/)来支持并让 LWN 继续存在。 |
| --- |

作者：**Daroc Alden**

2024年4月29日

2024年4月21日，一群匿名作者和非匿名签署者发布了[一封长篇公开信](https://save-nix-together.org/)给[Nix](https://nixos.org/)社区和Nix的创始人Eelco Dolstra，要求他从项目中辞职。他们声称项目领导存在持续问题，主要集中在他的行动据称削弱了被授权执行各种调解和治理任务的人。自公开后，该信件已获得超过100个签名。

#### 决策权

Nix项目由[ NixOS基金会](https://github.com/NixOS/foundation)管理，这是一个负责项目财务和法律责任的非营利组织。该基金会由一个拥有五名投票成员的董事会管理，由Dolstra主持。董事会没有任期限制，可以自行选举成员和主席。根据[董事会团队页面](https://nixos.org/community/teams/foundation-board/)，其责任包括处理“管理、法律和财务任务”、赞助和捐赠、“社区事件和努力”的资金，以及在社区冲突发生时充当仲裁者。特别地，董事会“不负责技术领导、决策或方向”，也不必处理所有决策。董事会负责为团队“自我组织提供框架”，包括“分发团队工作所需的证书和权限”的职责。

公开信中有几项相关投诉，但最核心的一项是他们声称Dolstra多次强迫董事会和其他社区团队成员推翻他们的决定：

> 例如，在董事会关于赞助政策进行数月讨论后，已形成一项允许社区否决NixCon赞助商的政策共识。Eelco（以及同时的Graham [Christensen]）在[开放董事会电话会议中的45分钟之后](https://discourse.nixos.org/t/open-board-call-2024-03-20/41898/)出现，并开始重新讨论我们是否需要限制赞助的问题，尽管除了他以外的所有人都已经同意了这一点。

克里斯滕森是 [Determinate Systems](https://determinate.systems/) 的联合创始人，与 Dolstra 合作。该信列举了其他例子，例如 Dolstra 阻止长期贡献者成为代码审阅者，或者阻止经过 RFC 过程的构建系统更改。信件得出结论，认为 Dolstra 实质上是在利用作为项目创始人的社会权力，以推翻本应该合作决策的决定。简言之，Dolstra 正如项目的"<q>事实上的终身慈善独裁者（BDFL）</q>"，尽管 NixOS 基金会的章程并未授予任何人此种权力。信中称这导致"<q>一种无权但要负责任的文化</q>"，侵蚀了贡献者继续参与项目的意愿。特别是提到了管理团队，说他们"<q>担心自己的权威会被 Eelco 直接或通过基金会间接削弱</q>"。

当被要求评论信中提出的问题时，NixOS 管理团队的成员作出了回应：

> 总体而言，我们认为这封信相当准确地描述了管理团队的情况。我们已经在紧急模式下运作了超过半年的时间。我们的团队保留率创下历史新低，我们几乎无法跟上新成员的招募步伐，因为老成员们都在离职。现在管理团队只剩下四个人，包括两个希望在找到替代者后离开的人，不包括上周离职的另一名管理员。
> 
> 就管理团队感到无力的程度而言，这主要是因为来自某些社区成员的严重对抗或者不稳定社区的风险，而不是因为实际缺乏权力。这大部分反映了社区内部更深层的文化冲突，而非直接与基金会董事会相关。

尽管对问题的根源有些不同意，他们继续承认 Dolstra 阻碍了多次改善情况的尝试，并表示理解许多社区成员的投诉。团队还称这个情况本身是一个“涉及许多人的深层次结构性和文化性问题”。

Pierre Bourdon，Nix 的长期贡献者，在 [Mastodon](https://mastodon.delroth.net/@delroth/112310645064859357) 上发布了他在 NixOS 工作经历的帖子，[表示](https://mastodon.delroth.net/@delroth/112310661634038271)虽然他不同意公开信的语气和方式，但关于 Dolstra 领导能力的事实陈述与他的经验相符。

#### 利益冲突

这封信还指控了多个利益冲突，主要涉及到 Dolstra 的雇主 Determinate Systems。[安都瑞尔](https://zh.wikipedia.org/wiki/Anduril_Industries)，一家使用 NixOS 的军事承包商，多次试图成为 NixCon 的赞助商，这在三月二十日董事会会议记录中反映出社区的不满。信中称，即使在许多核心贡献者表示反对后，Dolstra 仍然极力推动安都瑞尔成为赞助商。最终，由于社区的压力，安都瑞尔被取消了 [NixCon 2023](https://2023.nixcon.org/) 的赞助资格。[感谢 Martin Weinelt 指出 [安都瑞尔最终赞助了 NixCon 2024](https://2024-na.nixcon.org/#sponsors)。]

4月10日，董事会秘书**Théophane Hufschmitt**分享了关于董事会的[最新赞助政策](https://github.com/NixOS/foundation/blob/bb7af14ae242ab87c8312bb4122b2a3cd184462d/policies/sponsorship_policy.md)的[更新](https://discourse.nixos.org/t/nixos-foundation-event-sponsorship-policy/43110)。Hufschmitt表示对处理该情况的方式表示歉意，并承诺"<q>我们将优先考虑在我们的决策过程中的透明性、包容性和响应能力</q>。"

同一天，**Samuel Dionne-Riel**[表示](https://twitter.com/samueldr/status/1778259760707448853)，Dolstra拒绝澄清他是否与Anduril有关系，并询问了Determinate Systems的联合创始人**Christensen**：“<q>DetSys是否与Anduril有关系或曾有关系？</q>”**Christensen**[回答道](https://twitter.com/grhmc/status/1778386025007460682)：“<q>你知道这类问题基本上是不可能回答的，因为保密协议是存在的。</q>”

这并非是Dolstra第一次似乎避免披露潜在利益冲突；信函声称他将自己作为Determinate Systems的创始人的身份保密了几个月（Dolstra后来否认了这一说法），而这在Determinate Systems向客户做出的一些技术承诺的背景下尤为令人担忧。该公司为Nix生产了自己的安装程序，该公司承诺将为一些Nix功能提供稳定支持。信函指出：“<q>尽管如此，当雇佣了*CppNix的首席开发者*并且对团队的[决策]自治权没有任何交代时，这样做是值得质疑的。</q>”这些担忧并非全然是理论性的；自2023年9月版本2.18以来，主要的Nix安装程序在各种方面一直[存在问题](https://github.com/NixOS/nix/issues/10109)。

#### 呼吁行动

信的最后一节呼吁 Dolstra 辞去董事会职务，并建议他在至少六个月内完全脱离项目，以便让董事会其他成员有时间改革项目的治理结构。

> 这份文件应被视为矿井中的警报，*许多*人多年来一直有同感，并不能详尽地覆盖社区中的所有问题，但我们希望这足以证明需要采取行动。

信以建议如果 Dolstra 不辞职，签署者们将转向并支持项目的分支结尾。我联系了几位签署者，询问他们是否愿意提供更多关于他们为何认为这样一封信是必要步骤的评论。Haydon Welsh 回复道：

> 我签署了这封信，因为很明显其他团队成员都厌倦了 Eelco，所以我认为如果这是他们唯一重燃项目热情的希望，那是正确的。没有开源项目应该因为一个人而死亡或被硬分叉，这会破坏开源存在的许多意义。

Kiara Grouwstra 对此事有更强烈的感受：

> 尽管我希望 NixOS 董事会成员进行全面轮换，以及改变其目标和结构，以更好地包容社区，包括边缘化的观点，但我的朋友说服我，激烈的回应不会得到版主的喜欢，并与我分享了关于 Eelco 角色的草案公开信，我选择目前选择最容易的方法。

4月27日，Xe Iaso 在 [博客文章](https://xeiaso.net/blog/2024/much-ado-about-nothing/) 中表达了自己对这一事件的看法，称在这一点上分叉既是不可避免的，也是注定要失败的。即使信中呼吁的行动确实发生，这种困境已经对 Nix 社区产生了影响。4月21日，Nixpkgs 贡献者 Kamila Borowska [辞去了项目职务](https://github.com/NixOS/nixpkgs/pull/305826)。4月25日，Mario Rodas，贡献了超过 250 个软件包的人，也 [步其后尘](https://github.com/NixOS/nixpkgs/pull/306702)。总共有 24 位维护者 [离开了](https://github.com/NixOS/nixpkgs/issues?q=label%3A%228.has%3A+maintainer+removal%22)。

#### Dolstra 的回应

4月26日，Dolstra 在 [回应](https://determinate.systems/posts/on-community-in-nix/) 信中发表了他的看法。他表示董事会的角色是处理财务和法律事务，而不是管理 Nix 社区。他还声称自己在最近几年对 Nixpkgs 和 NixOS 的参与非常少。Dolstra 继续说道：“我只是五人 Nix 团队中的一员，并没有比其他人在确定团队方向上更多的正式权威。”尽管如此，这并没有直接反驳信中关于 Dolstra 超出其正式授权范围的指控。

Dolstra还重申了他的立场，认为NixCon不应拒绝Anduril的赞助，他说：“<q>我认为，作为开源软件开发者，我们无权决定谁的观点有效，谁的观点无效，也无权因此允许或禁止项目或会议的参与。</q>” 不过，Dolstra明确驳斥了一个观点，即他在决策系统的参与一直是秘密的：“<q>我的角色、参与以及对决策系统所做的良好工作的关注自公司成立以来一直是公开的。</q>” 他接着表示，决策系统试图对社区产生过大影响的说法是“<q>完全错误的</q>”。

他在回应中邀请社区成员，那些在Nix社区感到不受欢迎的人可以转而在决策系统工作：

> 我致力于创建一个让每个人感受到被看见、被听到和被重视的社区，我不会让毫无根据的指责影响这一重要工作。我鼓励所有阅读此文并感觉自己没有被听到或感到被排斥的人加入决策系统社区，因为我们将继续努力使Nix尽可能易于使用且产生最大的影响。

目前很难预测Nix社区将会走向何方，以及任何分支的最终命运。目前，Dolstra仍然是董事会主席 —— 在来自信函签署者的压力下，他似乎不太可能放弃这个职位。

* * *

（

[登录](https://lwn.net/Login/?target=/Articles/970824/)

[发表评论](https://lwn.net/Login/?target=/Articles/970824/)
