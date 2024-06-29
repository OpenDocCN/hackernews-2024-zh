<!--yml

类别：未分类

日期：2024-05-27 13:34:04

-->

# rfc-evidence/rfc_evidences_experiences.md at master · nrdxp/rfc-evidence · GitHub

> 来源：[https://github.com/nrdxp/rfc-evidence/blob/master/rfc_evidences_experiences.md](https://github.com/nrdxp/rfc-evidence/blob/master/rfc_evidences_experiences.md)

*NixOS 的一部分 [RFC 175](https://github.com/NixOS/rfcs/pull/175)*

> 注意：RFC的作者在发布后几乎立即被禁止。然而，你可以在这个公开的Matrix房间讨论它：[https://matrix.to/#/#rfc-175-all-together:matrix.org](https://matrix.to/#/#rfc-175-all-together:matrix.org)

### 证据和经验附录

[](#evidences-and-experiences-appendix)

此附录详细描述了一些经历，突显了NixOS社区管理文化的困境，从撰写本RFC的两位作者独特的视角来看。它提供了*具体的理由*，正如前述的*备忘录*所示，以展示采纳这样一份文件的必要性。然而，值得注意的是，这份文件并不详尽，主要限于最近的过去。

回顾过去几个月的事件，包括2023年11月Srid的禁令，必须在最近和持续存在的争议背景下审视这一RFC。在2024年初的第二次Anduril赞助争议期间，我们清楚地看到社区的某个子群体正在极力倾向于将“边缘化神话”作为获取特权的战略手段。

与此同时，这个团体对RFC 98的失败仍然感到不满，他们使用了*法西斯主义神话*以及对*忍耐的悖论*（最初由卡尔·波普尔提出，意图完全不同）的滥用延伸，将他们的对手描绘为恶意行为者，甚至是彻头彻尾的邪恶。当然，他们自己尽一切努力显得有理有据地行使权力，毫不负责任，并且明显感到他们自己有特权去这么做。^([1](#user-content-fn-1-46f7588a22422546136b3839ea28499a))

在最近的事件转折中，一群人支持匿名删除的文件，并开始攻击整个Nix项目的创始人`Eelco Dolstra`。**请注意，RFC 98的主要作者`Irene Knapp`也是这份文件的签署者之一：**

[https://save-nix-together.org/](https://save-nix-together.org/)

当前的管理团队大力支持这一最新举措。正如我们在这里清楚地看到的，贡献者`Hexa`，他本人也是Nix社区管理团队的成员，正在通过在 lobste.rs 上发布这条信息来向公众传达这一消息：

[https://lobste.rs/s/0qvtim/open_letter_nixos_foundation](https://lobste.rs/s/0qvtim/open_letter_nixos_foundation) [*镜像*](https://archive.is/GWkKA)

**请再次注意，本文撰写时，RFC 98 的主要作者**`Irene Knapp`**是 lobste.rs 的管理团队成员之一**：[https://lobste.rs/~Irene](https://lobste.rs/~Irene)

`Eelco Dolstra` 被描绘为一位不负责任的项目负责人，必须被驱逐，以便"社区"接管项目。但究竟是谁构成了"社区"，谁代表他们？在最近关于 GitHub 上发现的讨论中，一位用户提到了他认为的管理团队关于决定 Nix 相关事件赞助政策的事实上的权力夺取：

[NixOS/foundation#128 (comment)](https://github.com/NixOS/foundation/pull/128#issuecomment-2035356976) [*屏幕截图*](/nrdxp/rfc-evidence/blob/master/assets/issuecomment-2035356976.png)

我们目前正在目睹一个更大规模的事实上的权力夺取，一切都公开进行，并且再次涉及到管理。我们对这种公然试图将现有开放社区转变为极端不宽容的极左认同论意识形态游乐场的企图深感担忧。**管理团队在其中起着核心作用，因为他们似乎在执行并保护他们的不宽容行为**。这正在危及所有希望分享 Nix 生态系统带来好处的温和和正派的人们。

匿名文件的作者显然意识到他们自己企图的性质。在他们文件的一个部分结尾，用"同理心"的措辞，他们要求对任何批评进行预防性审查：

> 最后，我们承认，即使是由于明显的不端行为引起的 Eelco 的角色变化，也会被解读为一群左翼人士接管该项目。尽管作者的政治倾向是左倾的，但这不是关于 Eelco 的政治观点，而是关于他的行动，以及他的行动如何在文化上塑造了 Nix 项目，并且他的行动不符合项目的利益。我们意识到这种解读将导致显著的管理负荷，因为某些被禁止的社区成员进行了社区外的集结活动，以及内部社区的辩论。**我们请求基金会在采取行动时明确声明，这种行为是不可接受的**。

指责他人多年来一直采用相同策略，效果显著，却对团结进行指责的虚伪，以及怎样？而任何极左边主义者都会承认：技术是政治的，文化也是如此。因此，在他们看来，这必然主要是关于政治的，而且从这个意义上讲，他们所有（相当极端的）行动在他们自己眼中都是正当的，并且基于他们自己的权威。现在每个人都知道，极左边主义者不相信政治中立性！事实上，他们在许多场合已经明确表明了这一点。因此，“对此的否认必须被视为战略转移”。而我们可以清楚地看到，尽管“内部辩论不可接受”因为它增加了“调节负担”，“调节团队成员Hexa本人”却在公开发布具有争议性的信件，加剧了风险，为此类分裂性辩论创造了更多的激励。他们随后愿意利用这场辩论来合法化向他们分配更多权力，以保持社区的“安全性”。“分裂的代理人正在抱怨他们自己制造的分裂。”

请花点时间欣赏所有这一切的超现实荒谬性：

1.  在关于赞助政策的最新讨论中，有人在GitHub上指责正在进行的权力夺取 [NixOS/foundation#128 (comment)](https://github.com/NixOS/foundation/pull/128#issuecomment-2035356976) [*screen*](/nrdxp/rfc-evidence/blob/master/assets/issuecomment-2035356976.png)

1.  NixOS基金会董事会观察员`RaitoBezarius`在GitHub上否认有权力夺取，并谈到“权力夺取的幻想。” [NixOS/foundation#128 (comment)](https://github.com/NixOS/foundation/pull/128#issuecomment-2037592426) [*screen*](/nrdxp/rfc-evidence/blob/master/assets/issuecomment-2037592426.png)

1.  几天后，同一位`RaitoBezarius`在Mastodon上推广一篇帖子，其中有人说：所有法西斯主义者都将被驱逐 [https://hachyderm.io/@leftpaddotpy/112248186696362113](https://hachyderm.io/@leftpaddotpy/112248186696362113) [*screen*](/nrdxp/rfc-evidence/blob/master/assets/112248186696362113.png)

1.  该帖子的作者利用法西斯主义的神话来实现排斥性目的，指望调节团队成员`piegames`在Nix社区内推行这种极左边主义政策：“**请与piegames交流，他们在促成此事方面非常投入，这将有助于我们未来争取更好的董事会组成**”（目标是基金会董事会成员，其中最重要的是`Eelco Dolstra`） [https://hachyderm.io/@leftpaddotpy/112248495631909404](https://hachyderm.io/@leftpaddotpy/112248495631909404)

1.  一封公开信的出现呼吁驱逐并剥夺该项目创始人从他的终身工作中的权力，声称“Eelco应该辞去他在项目中所有正式角色，并至少休息6个月，**以便社区可以在实践中接管项目所有权。**” [https://save-nix-together.org/](https://save-nix-together.org/)

1.  **Hexa** 模板小组成员公开发布了这封信，而信件的作者们希望在事先压制了对他们权力控制的批评，因为这可能会增加“调节负担。” [https://lobste.rs/s/0qvtim/open_letter_nixos_foundation](https://lobste.rs/s/0qvtim/open_letter_nixos_foundation) [*mirror*](https://archive.is/GWkKA)

1.  很可能我们将来会看到关于关心巨魔和类似事物的投诉，当人们开始发表批评意见时，特定的Mastodon实例上的人们抱怨法西斯主义者和其他什么的。他们同时告诉别人他们有多么疲倦，希望他们的时间得到尊重。

荒诞剧的这一循环需要结束。地狱般的身份认同游戏的朋友们需要被驯服。**实际的调节 - 谦逊的美德 - 需要取代身份认同的分裂。**

在所有这些事情中，模板团队发挥着至关重要的作用，这一点我们已经可以看到。因此，其改革是最重要的一步。如果不是在这一点上明显滥用其权力以谋求政治利益，这一切都不可能发生。当然，他们会否认或者像他们多次那样转移注意力，这就是为什么我们不得不无奈地编制这份证明文件的原因。

无论如何，作为唯一有权决定谁属于“社区”的机构，我们认为防止其倾向身份认同排斥意识形态是至关重要的，因为它在最近几个月里可悲地越来越多地这样做。

虽然失败的RFC 98已经试图在公开场合使用法西斯主义神话作为排斥中庸声音的工具，我们现在可以看到同样的策略以更分散的方式和更隐秘的方式操作，试图损害该项目的核心。

对于不符合这一身份封闭严格政治评估方案的贡献者（可悲的是，它非常热衷且一点也不懒惰），下面收集的例子可以说明：对我们来说，这看起来非常像个人诽谤、恐吓和排斥。

我们充分意识到，由于我们最近过去的经历构成这一说法，我们提出的每个理由都可能在各种方面受到质疑。然而，我们认为这种展示事物看起来值得被他人看到。

愿所有人自行辨别。

辨别的自由绝不可牺牲。

### -   当前的审查文化并不公平，因为它不遵循所有人能够理解或解释的理由。

[](#1-current-moderation-culture-is-not-equitable-in-that-it-does-not-abide-by-reasons-accessible-or-explicable-to-all)

#### 1.1 没有明确的一般性理由指导审查决策。简而言之：这是一团糟

[](#11-there-are-no-discernible-general-reasons-guiding-moderation-decisions-tldr-its-a-giant-mess)

看起来，目前对于区分正确和错误的审查决定没有透明的标准。没有人确切知道审查团队基于何种决定。因此，无人能够明确预期何为允许何为不允许、何时何地以及为何。所有这些都可以即兴决定，没有任何可靠的标准。

因此，目前尚不清楚审查团队在其决策过程中实际上是受到何种目标的指导。对于新手来说，这一切都是不透明的。

尽管审查团队被认为是通过RFC 102来维护NixOS基金会的使命声明，并且审查团队最迟在2023年11月采纳了行为准则，人们仍然想知道团队如何与这些文件相关联？

在贡献者`Infinisil`发起的最近辩论中，版主`rhendric`澄清了审查团队与他们自己采纳的行为准则之间的关系。与`Infinisil`的想法相反，行为准则并非审查团队用来指导决策的文件，而是给出了以下解释：

> [...] 有一个行为准则，合理地描述了我们在审查过程中的目标。[...] **如果您想要的透明性之一是在我们采取行动时引用哪部分行为准则被违反，我们不会这样操作。** 我们采取最不强硬的审查行动，以履行RFC 102要求我们实施的NixOS基金会声明，并且我们在这样做时互相指导。
> 
> (周一，2024年3月11日，1:24 CET，来源：[https://matrix.to/#/!uzGGsQXWXsuwIqJImY:nixos.org/$syzY0wQ9wM7c1usw-XxrmZpB5xjzNQlp7hy6rFCaAKM?via=nixos.org&via=matrix.org&via=nixos.dev](https://matrix.to/#/!uzGGsQXWXsuwIqJImY:nixos.org/%24syzY0wQ9wM7c1usw-XxrmZpB5xjzNQlp7hy6rFCaAKM?via=nixos.org&via=matrix.org&via=nixos.dev) [*screen*](/nrdxp/rfc-evidence/blob/master/assets/syzY0wQ9wM7c1usw-XxrmZpB5xjzNQlp7hy6rFCaAKM.png))

***令人惊讶的是，行为准则似乎与实际决定毫无明显关系。决策并不以任何方式基于准则的规定。这引发了一个问题：准则的存在意义何在？而在这一点上，它如何“合理描述”任何内容？***

本文的作者，作为他们所信奉的自由派人士，坚信没有被禁止的就是允许的。否则会削弱在开放环境中贡献者必要的自发性。这将使他们在任何时刻都担心不公正的惩罚，负担过重。我们需要事先了解什么是被禁止的。这种理解显然是缺失的。

这在具体情况中的表现可从2023年11月Srid被禁止的情况看出。当要求他从Discourse页面移除链接到[https://srid.ca/unwoke](https://srid.ca/unwoke)时，他要求管理团队制定一般规则，禁止在社区平台公开链接类似声明。这是一个非常合理的要求，但管理团队并未丝毫遵守，甚至似乎未予考虑。

我们从这个解释中得出什么结论？

**管理团队*拒绝遵守一般原则来指导他们的管理决策*，在我们看来这违反了公平的基本原则。所有人都有权事先知道互动规则是什么。这些规则需要对所有人平等适用。**

#### 1.2 具体原因从未被提供，而只是从他人那里要求。tldr：一旦你开始查看，这就是双重标准。

[](#12-concrete-reasons-are-never-provided-but-only-ever-demanded-from-others-tldr-its-double-standards-once-you-start-looking)

除了一般原则的缺失之外，人们可能倾向于说**一般原则被具体案例决定所取代，但案例只以抽象的术语解释，从不与具体证据相关联。**

现在我们将看到，这导致了**一种具有讽刺意味的双重标准，使管理决策实际上无法问责，同时对受其影响的人难以理解。**

在2023年11月，Srid被禁止之后，一些人公开表达了对这一禁令完全无理的担忧。有人公开怀疑，Srid的禁令是出于政治意识形态的动机，而管理团队声称这是因为他过于“对抗性”。

在这种背景下，长期贡献者`Shea Levy`在GitHub上问贡献者兼管理团队负责人`Ryan Mullgian`，是否Srid的禁令是因为他在Discourse个人资料中链接了某些政治观点：

> @NixOS/moderation：@srid声称他被禁止是因为在他的Discourse资料中链接到了[https://srid.ca/unwoke](https://srid.ca/unwoke)。这正确吗？[https://github.com/NixOS/moderation/commit/e9d67b7efa03e6e9bc0bf2d03c6344d60c103395#commitcomment-132517786](https://github.com/NixOS/moderation/commit/e9d67b7efa03e6e9bc0bf2d03c6344d60c103395#commitcomment-132517786) [*screen*](/nrdxp/rfc-evidence/blob/master/assets/commitcomment-132517786.png)

对此，`Ryan Mulligan`给出了以下回答：

> Srid 多次将讨论偏离到一般政治/哲学辩论中。因为含有煽动性言论，多篇帖子已被隐藏/删除。多位社区成员报告了他们对 Srid 的不适感。在这种情况下，他的 Discourse 个人资料被视为又一种挑衅。作为回应，Srid 收到了警告。我们要求他停止如此对抗，移除个人资料上指向该页面的链接，并且今后不再链接它，等等。他部分地遵守了，但拒绝更改或删除该链接，后来又通过在 Discourse 上发布公开投票进一步升级，讨论这个审查问题。我们正是试图阻止这种投票行为。他因发布投票而被暂停，这是考虑到所有这些情况的结果。
> 
> [https://github.com/NixOS/moderation/commit/e9d67b7efa03e6e9bc0bf2d03c6344d60c103395#commitcomment-132624102](https://github.com/NixOS/moderation/commit/e9d67b7efa03e6e9bc0bf2d03c6344d60c103395#commitcomment-132624102) [*screen*](/nrdxp/rfc-evidence/blob/master/assets/commitcomment-132624102.png)

注意，这个回答没有为 Srid 的禁令提供任何具体理由。它只涉及无法验证的概括性内容。没有人可以查阅 Srid 偏离讨论的多次事件，也没有人可以看到隐藏或删除的帖子来判断它们是否具有煽动性或无害，也没有人知道社区成员可能感到不适的原因是由于 Srid 的实际行为还是因为他们自己坚持攻击性身份认同意识形态，将 Srid 视为意识形态上的敌人。

然而，每个人都可以验证，Srid 致力于尽可能地*无害*，并因此制定了自己的行为准则。这是一个奇怪的对比。

显然不足的回应导致贡献者 `David Arnold (blaggacao)` - 令我们惋惜的是，他也因此被审查团队禁止，他曾帮助创建 RFC 102 的团队 - 开启了**关于审查责任的帖子。**^([2](#user-content-fn-2-46f7588a22422546136b3839ea28499a)) 这个帖子因决定的争议性质而变得更加情绪化，几位明显支持左倾身份认同意识形态的人变得激进。审查团队决定将其关闭，从 Discourse 中删除，并稍后大幅审查它，删除了原始帖子的 13 篇*即使它已被从公共搜索结果中解除列表并因此隐藏*。然而，在这个特定案例中，原始版本已被保留。^([3](#user-content-fn-3-46f7588a22422546136b3839ea28499a))

在该帖子的最后，至今未变的消息中，同一审查团队成员 Ryan Mulligan 要求批评者验证：

> 如果发现你的私人关切未得到处理，那么你可以考虑**发布一些有具体论据支持的礼貌公开帖子，而不是一些模糊的担忧。**
> 
> 论坛管理员Ryan Mulligan [https://discourse.nixos.org/t/moderation-team-accountability-issues/35482/71](https://discourse.nixos.org/t/moderation-team-accountability-issues/35482/71) [*镜像*](https://archive.is/ynqiM)

**向你宣扬抽象！但只向我宣扬具体原因！**

我们应该从中得出什么结论？对他人不提供具体解释，但如果他人打算追责你，则要求最具体的解释。

我们相当明显地认为，这为各种双重标准奠定了良好基础！这也使问责几乎成为不可能。因为除非你知道决定的依据是什么，否则你怎么能调查它是否正确执行？

我们认为这种做法不能继续下去。管理决策的不透明和武断不仅破坏了信任，也未尊重社区了解影响其成员的重大行动背后理念的权利。

### 2) 管理团队似乎在社区成员之间激起敌意。简而言之：有些人比其他人更平等

#### 2.1) 管理团队激起可能对他人有害的态度。简而言之：这不好！

[](#21-管理团队激起可能对他人有害的态度-简而言之，这不好)

鉴于力图不被称为权力夺取之初所提及，我们将探讨管理团队如何促成一种特定类型的激进活动主义，从而在贡献者之间播下敌意。

这一点确实如此，当我们**得到这样的印象时，存在一种压制特定类型政治言论的努力**（参见对Srid的禁令，没有接受一般规则），同时启用其他类型，即使这些类型令人非常不安。

由于过去几个月中有几个人提出完全禁止Nix社区的政治言论的问题，这种情况可能看起来比现在更可取，因此管理团队曾在某个时候澄清了他们在这个问题上的立场。

这一澄清很重要。对于限制政治言论的更严肃尝试之一，由贡献者`Jon Ringer`提出了禁止不可接受政治活动的PR：

> 为了帮助将来避免争议，我仍希望看到[NixOS/.github#16](https://github.com/NixOS/.github/pull/16)或类似内容被添加到行为准则。只需明确预期，这些事物最好在Nix平台之外解决。
> 
> 贡献者 Jon Ringer，2024年3月12日，NixOS治理室在Matrix上的[链接](https://matrix.to/#/!VyoUhyWvlhSpFWWxHL:matrix.org/%24W6QrMhOJbtwjxL3-yquzfuxchLJHUWqFzfjeCh9fc0U?via=nixos.org&via=matrix.org&via=nixos.dev) [*屏幕截图*](/nrdxp/rfc-evidence/blob/master/assets/FC6KAUDoSdZXyBiMX39ShZlxkGgkh1u1tDLEvDQqkDs.png)

管理团队对于更广泛地限制政治言论的建议并未积极跟进，而是表现得有些轻视。在这个PR的背景下，管理团队给出了以下解释：

> [...] 我们可能不会对政治活动实施禁令。**活动的语气并不一定是鉴别的因素。** 我们不会允许违背我们“提供一个故意欢迎和安全的环境，让所有人感到受到重视和关心”的使命的活动。
> 
> 管理员 rhendric，2024年3月12日，NixOS治理室在Matrix上的[链接](https://matrix.to/#/!VyoUhyWvlhSpFWWxHL:matrix.org/%24FC6KAUDoSdZXyBiMX39ShZlxkGgkh1u1tDLEvDQqkDs?via=nixos.org&via=matrix.org&via=nixos.dev) [*屏幕截图*](/nrdxp/rfc-evidence/blob/master/assets/FC6KAUDoSdZXyBiMX39ShZlxkGgkh1u1tDLEvDQqkDs.png)

注意，***活动的语气并不是管理团队的鉴别因素***。这可能已经表明，***某些类型的活动主义被允许在社区中变得不文明***，而在我们的估计中，它们被允许变得不文明到足以对其他社区成员进行口头攻击并给他们带来重大情感压力的程度。

这一点特别明显体现在一系列事件中，这些事件都与一个政治活跃的社区成员有关，为了最小化匿名化，我们将他称为`J`。对我们来说，`J`似乎与管理团队有一种特殊的关系，有人可能会认为这构成一种偏袒。

在最近几个月中，自2023年11月禁止`Srid`并采用行为准则以来，我们观察到了以下事件：

+   第一个Anduril赞助的辩护者在2023年欧洲NixCon的讨论中被`J`在NixOS基金会Matrix室称为“谋杀机器的辩护者”，在语言上将他们类比于真正辩护谋杀的人。我们认为这样的措辞在专业环境中是不恰当的。

    正如我们将看到的那样，这并不是一个孤立的事件。 值得注意的是，这些被诽谤的人被认为“事实上不是创造安全环境的人群”，因此这种攻击是在使用*边缘化的神话*对抗一个想象的敌人的论据中进行的 - 注意，这是普通社区成员！ - 被诽谤。 [https://matrix.to/#/!CJXQiUGqNPcFonEdME:nixos.org/%24GNOHzZx-f_7Fkkgg4f4auQpTNa9tB4aa33oHQxzArwg?via=nixos.org&via=matrix.org&via=nixos.dev](https://matrix.to/#/!CJXQiUGqNPcFonEdME:nixos.org/%24GNOHzZx-f_7Fkkgg4f4auQpTNa9tB4aa33oHQxzArwg?via=nixos.org&via=matrix.org&via=nixos.dev) [*屏幕截图*](/nrdxp/rfc-evidence/blob/master/assets/GNOHzZx-f_7Fkkgg4f4auQpTNa9tB4aa33oHQxzArwg.png)

+   在Discourse上已经提到的隐藏主题中，由`David Arnold (blaggacao)`开启，`J`声称`Srid`的一个未知帖子是一个“大oted screed，这恰恰是由那些（确实地）打算谋杀我们边缘化人群的人写的事情，或者希望其他人这样做。” 从而将`Srid`比作那些确实对少数族裔有敌意的人。**这种说法确实确立了一种模式：*在一个真正的行为与一种真正令人发指的行为之间创造修辞上的相似，倾向于损害Nix社区贡献者的公众形象。*** [https://srid.ca/nixos-mod/accountability-thread](https://srid.ca/nixos-mod/accountability-thread)

+   在2023年末，nixpkgs的一位长期贡献者因试图提出关于`J`行为的批评性问题而受到`J`的言语攻击。 `J`声称这种批评性尝试等同于“明显的滥用行为”，并在这种情况下“未经他们同意对人们的情绪状态进行实验”，即对`J`的情绪状态进行实验。 我们应该注意，在我们看来，`J`所给出的这些描述甚至不能接近对情况的准确描述。

    然而，引起我们注意的是，就像上述情况一样，`J`再次将其他社区成员的行为修辞地比作一种特别可怕的行为，这一次是将其比作一位科学家违反他们的意愿对人们进行不道德实验。显然，这种事情从未发生过。**然而，这又一次产生了前述修辞模式的另一个实例。** 毋庸置疑，论坛管理员团队对此行为没有发表公开声明。 [https://matrix.to/#/!VyoUhyWvlhSpFWWxHL:matrix.org/%24Y-ndXmAfGUXPPfOD_YvOE59pj8gQzp1kI4tBdXQR13w?via=nixos.org&via=matrix.org&via=nixos.dev](https://matrix.to/#/!VyoUhyWvlhSpFWWxHL:matrix.org/%24Y-ndXmAfGUXPPfOD_YvOE59pj8gQzp1kI4tBdXQR13w?via=nixos.org&via=matrix.org&via=nixos.dev) [*屏幕截图*](/nrdxp/rfc-evidence/blob/master/assets/Y-ndXmAfGUXPPfOD_YvOE59pj8gQzp1kI4tBdXQR13w.png)

+   对于管理团队来说，`J`声称多年来一直存在骚扰行为，据称是针对他和类似他的人。我们没有找到任何这种情况属实的迹象。对我们来说，这种说法显得过于夸张且有些牵强。

+   当首次贡献者`Jon Ringer`和稍后的贡献者`David Arnold`（`blaggacao`）试图在2023年末介入上述情况时，`Jon Ringer`首先被`J`口头攻击，而`David Arnold`试图斡旋导致他被管理团队无端封禁。我们注意到`David Arnold`展示的行为风格，是我们在具有健康和开放沟通文化的社区中可以预期的。由于这些上下文中的一些相关消息已经被删除，因此它们不再容易在Matrix上找到。

不久之后，`J`决定给人们提供私人政治课可能是个好主意。

在`J`过去行为意识到的情况下，本RFC的两位作者之一决定提出几个澄清问题关于`J`的提议。这是为了防止其他人受到伤害，因为似乎已经形成了这种可能性的一种已建立的模式。

这些问题发布后，管理团队感到有责任在讨论区做出官方声明。由于这主要涉及管理团队，姓名显然已被删除：

> 作为社区，我们最近在讨论这个想法时很难不过度激动。**我对J的态度感到鼓舞**，并且我希望这个实验能成功。考虑到这个话题的波动性，我想请所有有兴趣让J担任导师的人**定期反思自己的情绪状态**，并且倾向于在需要时花些时间冷静下来。[...] **尊重的言论仍然是NixOS空间的基本期望。**
> 
> Moderator rhendric的官方声明 [https://discourse.nixos.org/t/open-offer-politics-in-tech-sessions/40097/3](https://discourse.nixos.org/t/open-offer-politics-in-tech-sessions/40097/3) [*mirror*](https://archive.is/wip/JU8Tk)

我们发现这个声明令人非常不安。鉴于我们从`J`这里得到的缺乏尊重和文明，以及管理团队对此显然的认识，这些要求的接收者是谁？尊重只是针对某些人的期望，而不是其他人吗？文明只是对社区的某个子集提出的要求吗？

如果版主团队不愿意从事所谓的“语气管理”，我们注意到当版主`Hexa`在GitHub上干预贡献者`amjoseph`时，这种语气管理并没有问题，导致`amjoseph`因对治理方法的担忧而退出项目。他的声明如下：“@amjoseph-nixpkgs **你需要减轻你的提交信息的语气**，并开始与其他人沟通，持续的合作真的不会以任何其他方式进行。” [NixOS/nixpkgs#282220 (comment)](https://github.com/NixOS/nixpkgs/pull/282220#issuecomment-1902634102) [*屏幕截图*](/nrdxp/rfc-evidence/blob/master/assets/issuecomment-1902634102.png)

对高效贡献者进行语气管理是否可接受，但对于似乎有口头攻击历史的政治教训者不可接受呢？

但或许更重要的是，我们应该如何理解**版主团队是如何被J的态度所*鼓励*的**？那时候，`J`已经确立了相当多的口头攻击其他社区成员的态度模式。**社区成员，版主团队选择在公众场合不保护他们免受这些攻击。**

如果这还不够，`J`的个人网站上还有更多令人不安的信息，表明这种态度模式比过去几个月的事件所显示的要老得多。由于这个网站是公开可访问的，而且版主们可能对此已经了解，我们认为他们有责任检查他们信任的人，以改善社区。

在撰写本文时（2024/19/04），`J`的个人网站包含了以下关于自己的声明：

> 万一你还没有意识到，我是一名社会工程师。全职。24/7。我在几乎每一次对话中都使用操纵和其他心理战术。我会试图改变你的思维方式，并尝试解决你可能甚至没有意识到的问题。

并且稍后：

> 唯一我会升级情况到更严重事态（比如曝光个人信息）的情况是当我已经和你谈过，并且你在伤害他人的过程中积极拒绝倾听时。很少有人达到这个阶段。

我们不在这里链接这个网站，因为阅读它可能有些令人不安，然而找到这些声明却很容易。

然而，我们真正想提出的问题是：**为什么版主团队官方认为对待政治言论的态度对公共记录中有这种模式的人是鼓励的？**请记住，这些都不包含任何机密或私人信息。这些都是任何负责任的人在几分钟内可以轻松找到的东西。

对我们来说，这仅在试图利用管理团队的权力来加强Nix社区内的身份认同主义意识形态的尝试中才有意义。最近的事件似乎证实了这一点。

**亲爱的读者，你希望成为全职社会工程的对象，然后，在你抵制到可能被社会工程师判断为有害的程度时，可能会遭受到doxing的对象吗？**

**在此期间，管理团队告诉社区，自己承认自己全天候从事社会工程？**

我们不这样认为。

#### 2.2）身份认同主义活动分子如何看待其他社区成员。tldr：你们都在他们之下！

作为最后一点，我们希望分享一下这背后可能存在的政治态度。在最近的另一场争议中，也被管理团队关闭并且隐藏起来，用户`J`明确表明了他如何看待与我们技术社区其他成员沟通的态度：

> **这里没有所谓的“与同行的讨论”。**

他否认其他人是他的同行，这也是他们作为社区成员基本平等的否定，他自己则认为在政治问题上拥有特殊的能力，这种能力对其他人不负责，这对我们来说似乎是问题的核心。

> **如果你没有资格，那么你应该把这些考虑交给有资格的人来做，而不是试图把自己或你的观点强加进去。**

不再征求其他人的意见。**当涉及政治分歧时，只有一方的声音应该被听到；另一方则应该保持沉默。**要求无声服从，不分享自己的意见，几乎不能更明确地表达。

**对我们来说，这些声明是侵略性左翼身份认同主义的典型标志，通过言语攻击他人以建立统治的等级制度。**

让我们再次直接问你，亲爱的读者：当你的目标是将这个社区，你是其中的一员，引领到更加平静的水域时，你会在这些态度中找到鼓励吗？在知道这一系列事件的其他人眼中，选择这个人来向他人传授政治课程会被视为鼓励吗？

你会对那些选择支持这些使者的权威人士的能力感到信任吗？我们把这个判断留给你。

对我们来说，在这里找不到任何鼓励的东西。

对我们来说，这不是“有意欢迎和安全的环境，让所有人感到受到重视和关心”。

#### 3）重新审视Srid的禁令及其后果。tldr：没有理由，这全是“管理”团队的个人行为。

[]（＃3-重新访问srids-ban-and-its-aftermath-tldr-there-is-no-justification-its-all-personal-for-the-moderation-team）

让我们最终来到一个特别有趣的话题：Srid 的禁令。或者说，它的未知后果。由于禁令本身及其背后的情况最好的文档实际上由 Srid 自己提供，我们发现这是一个非常客观和冷静的陈述，考虑到他是权力滥用的受害者。^([4](#user-content-fn-4-46f7588a22422546136b3839ea28499a))

我们已经提到管理团队不愿或不能公开陈述实际导致禁令的具体原因。这导致这两位作者之一开始试图找出 Srid 被禁止的真正原因，并探讨如何以建设性方式解决。

这次旨在建设性改进的尝试最终以管理团队（特别是版主 `Hexa`）对被禁止的贡献者 `Srid` 进行了一种显然是报复性的抨击，严重违反了私下沟通中的信任。

在 Srid 被禁后的 12 月，我们启动了与管理团队的私下沟通，因为版主 `Ryan Mulligan` 向整个社区做出了以下承诺：

> 如果他愿意以建设性方式参与，我们将继续与 Srid 进行沟通，并有机会让他复职。
> 
> [https://github.com/NixOS/moderation/commit/e9d67b7efa03e6e9bc0bf2d03c6344d60c103395#commitcomment-132624102](https://github.com/NixOS/moderation/commit/e9d67b7efa03e6e9bc0bf2d03c6344d60c103395#commitcomment-132624102) [*screen*](/nrdxp/rfc-evidence/blob/master/assets/commitcomment-132624102.png)

所以，管理团队履行了他们的承诺，他们与 **Srid** 的沟通应该是私下进行的，并由这份 RFC 的两位作者之一陪同。这样做是为了防止再次发生任何双方升级的情况。由于 Ryan 曾声称 Srid 在“多次”“搅乱讨论”，因此似乎很重要确保不会再次发生类似的升级。

由于管理团队坚持让 `Srid` 首先尝试建立沟通，因此鼓励 Srid 这样做，事实上他也确实这样做了，发了一封简短的电子邮件询问他重返 Nix 社区的可能条款。那是在 2023 年 12 月 12 日。然后什么也没发生。没有回应，只是沉默。所以我们等待着。每两周，`Srid` 都会向管理团队发送新的消息，跟进他的请求。

我们终于在 2024 年 1 月 21 日（周日）收到了管理团队的回应，这距离最初的短信写作仅仅过去了不到 6 周。回应的性质让我们感到非常失望。版主 `Hexa` 凭借非常牵强的理由拒绝了 `Srid` 重返 Nix 社区的请求。

然而，就在发送此电子邮件的几小时前，`Hexa` 本人已对 `Srid` 在 lobste.rs 上发布的一个无害帖子进行了干预，`Srid` 在帖子中展示了他正在制作的 Nix 教程。

没有任何挑衅，并且完全滥用作为 NixOS 审查团队成员授予的权威，该权威仅限于 Nix 社区平台，显然不打算伤害此类平台之外的人，官方声音表达如下声明：

> 我是 NixOS 审查团队的成员，请见[https://nixos.org/community/teams/moderation](https://nixos.org/community/teams/moderation)。`Srid` 已被禁止进入我们的社区（[https://github.com/NixOS/moderation/commit/e9d67b7efa03e6e9bc0bf2d03c6344d60c103395](https://github.com/NixOS/moderation/commit/e9d67b7efa03e6e9bc0bf2d03c6344d60c103395)），现在试图将流量引导到他们的第三方通信平台。
> 
> `Srid` 在私下交流仍在进行时，lobste.rs 上的版主 `Hexa` 在没有任何挑衅的情况下公开干预，详情见[https://archive.is/Z2BU3](https://archive.is/Z2BU3) & [https://archive.is/75A7j](https://archive.is/75A7j)。

由于这种干预，lobste.rs 上的一位版主基于非常可疑的理由决定对 `Srid` 进行封禁。很显然，两个团队之间的个人关系是 lobste.rs 对 `Srid` 实施封禁的原因。我们最初注意到，《RFC 98》的主要作者 `Irene Knapp` 是 lobste.rs 的审查团队成员。然而，这些都属于推测范围，目前无法证实。

确定的是，社区的审查工作结束于该社区的边界。显然越过了这一边界，在私下交流仍在进行时给 `Srid` 带来了更多伤害。

**私下与审查团队沟通的尝试似乎已被滥用，成为我们只能视为个人报复行为的行动。至少我们没有看到这一举措的任何可能以诚信为基础的解释。显然，此举从未向任何相关人员解释过。**

**我们可以说，整个团队的早期声明，由 `Ryan Mulligan` 代表整个团队，似乎是对整个社区的谎言，旨在转移注意力而不是真正建设性地解决 `Srid` 的前进道路。我们从未收到任何建设性沟通的迹象。**

未来我们只能警告任何其他人：只要现有的等级制度还在，不要对私下沟通解决方案的声明抱有任何信任。我们的经验表明，私下沟通被首要目的是试图让当前的审查层免受公众审查和问责的影响。

**以目前的制度和个人设置来看，审查团队是不可信的私下沟通对象。**

为了确保这种情况迅速且实质性地得到改善，我们已经在我们的 RFC 中共同创建了两个提案。

#### 5) 以积极的态度结束：我们希望的是

[](#5-ending-on-a-positive-note-what-we-would-wish-for)

在经历了这么多负面经历之后以积极的态度结束可能是个好主意，我们认为。

其他人应该理解我们希望发生什么，而不是我们经历过的事情。我们的提议并非旨在执行任何形式的个人报复，就像我们自己曾经目睹的那种。相反，它旨在确保将来不再可能出现这种行为。

特别是，我们期望：

+   以文明互动为要求，适用于所有人。我们已经看到了不文明语言的例子。我们希望用文明来取代这种行为。

+   一个适用于所有人的规范标准。这并不意味着对个体困境的不灵活认知，但意味着不基于政治立场或假定的身份差异来确定规范地位。

+   结束内部技术项目中认同主义活动的分裂激化。这种行为及其所涉及的语言已经在整体社会层面造成了足够的伤害。

+   最后，也许是最重要的是：**我们希望克服目前的局面，实施更加公平的管理方案。但我们不希望对当前或过去的任何管理团队成员产生怨恨。**

我们认为这是一个制度性问题而非个体问题。这是因为这个机构的建立方式及其有机发展导致了这种局面，并非人们固有的特性。机构的不良建立结合认同陷阱的影响使得这种不良行为成为可能。让我们不要对这种不良行为过度谴责，而是看到变革的必要性。

让我们改革管理团队的制度，确保这样的事件不再发生。
