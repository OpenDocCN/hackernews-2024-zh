<!--yml

category: 未分类

date: 2024-05-27 14:42:58

-->

# 关于非技术性电子游戏作弊缓解措施

> 来源：[https://dustri.org/b/on-non-technical-video-games-cheat-mitigations.html](https://dustri.org/b/on-non-technical-video-games-cheat-mitigations.html)

作弊行为如同电子游戏一样古老，而且将一直存在。今天反作弊市场有一些知名的参与者：[BattlEye](https://en.wikipedia.org/wiki/BattlEye)、[Valve's VAC](https://en.wikipedia.org/wiki/Valve_Anti-Cheat)、[PunkBuster](https://en.wikipedia.org/wiki/PunkBuster)、[Epic's EAC](https://easy.ac/en-us/)、[Blizzard's Warden](https://wowpedia.fandom.com/wiki/Warden_(software))、[Riot's Vanguard](https://support-valorant.riotgames.com/hc/en-us/articles/360046160933-What-is-Vanguard-)、[Activision's Ricochet](https://callofduty.com/en/warzone/ricochet)……还有一些内部的。

为了在这场竞争中保持竞争力，双方都在采取越来越具侵入性的技术隐私侵犯措施：流式传输虚拟化的 shellcode、硬件指纹和锁定、[栈行走](https://secret.club/2020/01/05/battleye-stack-walking.html)、类似启动组件的内核驱动程序、[TPM](https://en.wikipedia.org/wiki/Trusted_Platform_Module)/安全启动/[HVCI](https://learn.microsoft.com/en-us/windows-hardware/drivers/bringup/device-guard-and-credential-guard)/[IOMMU](https://en.wikipedia.org/wiki/Input%E2%80%93output_memory_management_unit)/[VBS](https://learn.microsoft.com/en-us/windows-hardware/design/device-experiences/oem-vbs)/……[花招](https://support-valorant.riotgames.com/hc/en-us/articles/22291331362067-Vanguard-Restrictions)、检测/使用[虚拟机管理程序](https://secret.club/2020/04/13/how-anti-cheats-detect-system-emulation.html)、[可疑材料的外泄](https://secret.club/2020/03/31/battleye-developer-tracking.html)、外部[DMA](https://en.wikipedia.org/wiki/Direct_memory_access)硬件，或其他[更奇异的事情](./paper-notes-reversing-anti-cheats-detection-generation-cycle-with-configurable-hallucinations.html)。

然而，反作弊仍然被常规性地规避，尽管方式不太公开，但私人和封闭社区的作弊依然繁荣，因为这本质上是一场失败的游戏。由于游戏和反作弊都是软件，它们当然充满了[滑稽](https://vice.com/en/article/d7y5wj/street-fighter-v-rootkit)缺陷，导致了[愚蠢](https://unknowncheats.me/forum/anti-cheat-bypass/614682-eac-dll-loading-method-eac-forcer.html)[规避](https://unknowncheats.me/forum/anti-cheat-bypass/503052-easy-anti-cheat-kernel-packet-fucker.html)。

但这不是这篇博客文章的主题。如今，作弊被视为更大问题的一部分：滥用和毒性。作弊不仅仅是因为在道德上有问题，而是因为它扰乱了游戏的本意。毒性和滥用行为导致完全相同的结果：由于作弊/滥用/毒性问题而不愿意玩的游戏将会减少玩家数量，评价不佳，等等，不会赚钱。我相信我们可以对我们社会的当前状态做出类比，但我岔开了话题。

对于本文，我们将作弊和滥用/毒性视为一个术语*滥用*下的单一问题。现在，由于滥用不仅仅是一个技术问题，而且是一个社会问题，它不能仅通过技术解决方案来解决，因此让我们看看游戏开发者正在想出哪些非技术缓解措施来遏制这个问题。

最明显的缓解措施是让作弊变得昂贵，从金钱上来看。为一款游戏支付60欧元是一项巨大的投资，尤其是如果每次被封禁都要重新购买。当然，这并不适用于免费游戏，但可以通过拥有一个化妆品生态系统来模拟，无论是付费购买还是通过努力获得。玩视频游戏时的另一个昂贵的事情是硬件，而封禁可以与硬件绑定。

## 全球措施

在这个层面上的**主要**缓解措施是声誉系统。它们基于最了解游戏应该如何有趣和公平进行的人：玩家。比赛结束后，他们被鼓励对比赛的公平性进行投票，不仅是在比赛层面上，而且直接在玩家层面上：“鲍勃真的很关心其他人”，“鲍勃是一个团队合作者”，等等。对于负面行为，举报不必等到比赛结束，玩家可以举报作弊、在文本/语音聊天中发表冒犯性言论、[捣乱](https://en.wikipedia.org/wiki/Griefer)、逃避排队、[使用低等级账号](https://www.urbandictionary.com/define.php?term=smurfing)等等。当然，诽谤性的举报会受到惩罚。

同伴压力也是一个好的杠杆，不仅要针对作弊者采取行动，还要针对从作弊中获益的人，比如常规队友。

[漏洞赏金计划](https://en.wikipedia.org/wiki/Bug_bounty_program)现在很普遍，因此现在有[一些](https://hackerone.com/riot)在奖励反作弊绕过/利用方面的奖励。奖励目前有点便宜，但随着计划的成熟，可能会上涨。积极的影响有多重：

1.  它增加了报告问题以修复它们的动力：现在玩家发现漏洞/利用可以获得一些现金作为发现的奖励。

1.  随着更多的滥用向量被消灭，奖励价格将上升，也许报告漏洞比将其出售给作弊提供者更有利可图。这并不罕见，谷歌的[内核CTF比赛](https://google.github.io/security-research/kernelctf/rules.html)支付的比Zerodium高两倍。

1.  如果漏洞赏金计划得到正确管理，报告问题将获得一定金额的概率会高于在未知时间内使用漏洞作弊直到被修复。

1.  这可能会增加寻找问题并愿意报告问题的人数。

社区管理员还可以定期发布有关封禁潮、防作弊措施、举报等的更新，以明确表示滥用行为是正在被解决的事情，对玩家来说是一场危险的赌博。我想我曾经看到一些人花时间证明一些正在直播的作弊者实际上是游戏早期版本的预先录制的镜头，因为其中一些游戏细节在此期间已经更新。

## 账户级别措施

一些游戏商店，如[Steam](https://zh.wikipedia.org/wiki/Steam) ，有一个账户级别的“作弊者”标记，这意味着如果有人因作弊而被某个游戏封禁，其他游戏也可以知道。但更重要的是，[成就](https://zh.wikipedia.org/wiki/%E6%88%90%E5%B0%B1_(%E8%A7%86%E9%A2%91%E6%B8%B8%E6%88%8F))和化妆品也与一个账户相关联，正如之前提到的，这些都是非零的时间和/或金钱投资。被封禁意味着失去它们。当然，这只会吓阻投机的作弊者，因为人们可以简单地创建其他账户来作弊，但这可以通过纯技术手段更加困难。

大多数*竞技*在线游戏都有排名和休闲游戏模式，前者只有在在后者中花费一定时间后才能访问。这意味着每次被封禁都必须重新开始，或者[付钱给别人来做](https://zh.wikipedia.org/wiki/%E5%8A%A9%E5%8A%BF%E5%BC%80%E5%8F%91)。一些工作室甚至让玩家通过更多的步骤才能玩游戏，比如要求[MFA](https://zh.wikipedia.org/wiki/%E5%A4%9A%E5%9B%A0%E7%B4%A0%E8%BA%AB%E4%BB%BD%E9%AA%8C%E8%AF%81)，或者与被标记为教程的[机器人](https://zh.wikipedia.org/wiki/%E8%A7%86%E9%A2%91%E6%B8%B8%E6%88%8F%E6%9C%BA%E5%99%A8%E4%BA%BA)对战几场比赛后才能与其他人一起玩。当然，这需要保持一种微妙的平衡来烦扰滥用者但不是合法玩家。

## 玩家级别措施

非技术措施的目标不是为了使滥用变得不可能，而是为了让其不值得。此外，对[边缘之主](https://zh.wikipedia.org/wiki/%E8%BE%B9%E7%BC%98%E4%B9%8B%E4%B8%BB)采取即时永久封禁似乎有点过分，因此采取大量措施对付滥用者是有道理的：有人可能希望允许人们纠正他们的行为，将他们孤立以冷静下来等等。这可能包括文本警告、暂时封禁、从当前游戏中踢出、聊天/语音静音、失去参与排名比赛的资格、减少获得的经验点数，等等。

玩家因各种原因滥用行为，但我认为大多数人之所以这样做是因为好玩。因此，破坏他们的乐趣是遏制这种行为的好方法。一个简单的方法是让他们一起玩，通过按声誉分组玩家，或者在服务器上明确禁用技术反作弊措施。但还有更具创意的措施，比如[禁用他们的降落伞](https://www.callofduty.com/en/blog/2023/11/call-of-duty-ricochet-anti-cheat-modern-warfare-III-progress-report)，将他们的伤害输出降低到荒谬的水平，夺走他们的武器，[让其他合法玩家对他们不可见](https://www.callofduty.com/blog/2023/06/call-of-duty-ricochet-anti-cheat-season-04-update)，随机删除他们的输入，[幻觉](./paper-notes-reversing-anti-cheats-detection-generation-cycle-with-configurable-hallucinations.html)，... 虽然这比简单地将他们分组需要更多的工程时间，但它有几个高价值的回报：

+   允许游戏开发者花更多时间收集有关作弊行为在技术层面上是如何运作的数据，

+   减少作弊者对游戏的影响使得能够大大推迟封禁他们而不对其他玩家产生太大影响，从而使作弊者更难准确找到作弊被检测的原因和方式。

+   这绝对是滑稽的。

## 举例

+   它使用 BattlEye，在 2022 年底到 2023 年初每月封禁了约 [5000](https://ubisoft.com/en-us/game/rainbow-six/siege/news-updates/2g7hT2NNuOqrj35RfgsFxN/anticheat-status-update-march-2023) 个账户，这是很多，但也显示它并不能阻止作弊行为。

+   这款游戏售价 [$8](https://store.steampowered.com/app/359550/Tom_Clancys_Rainbow_Six_Siege/)，但如果你想要获得所有的操作员，需要 $70。也可以通过游戏解锁操作员，这需要数百小时的时间。

+   要进行排位赛，需要达到 [50 级](https://ubisoft.com/en-gb/game/rainbow-six/siege/news-updates/4hShcX2HZTG2ttIi3IIN9Y/matchmaking-rating)，这需要大约 50 小时左右。

+   这款游戏有丰富的化妆品生态系统，可以 [以高昂的价格购买](https://store.ubisoft.com/us/dlc-type-skins-cosmetics)，也可以通过玩游戏辛苦获得，但如果账户被封禁，则会失去这些化妆品。

+   如果玩家被举报为故意的友军伤害，则会受到相应的伤害。

+   它正在开发一个相当复杂的 [声誉系统](https://ubisoft.com/en-gb/game/rainbow-six/siege/news-updates/22JLMFeayzuamhb7YKbAjm/reputation-system-activation-more)，其中行为“积极”的人会得到奖励（更多的经验值，化妆品，...），而那些行为“消极”的人可能会被阻止玩*排位*，获得较少的经验值，...

其开发者甚至发表了一系列关于所谓“游戏健康”的 [精彩博文](https://playvalorant.com/en-us/news/tags/game-health-series/)。

+   这款游戏是免费游玩的，但附带*大量*的[装饰品](https://valorantstrike.com/valorant-store/)。

+   作弊者会被永久封禁，但是从中受益的人也可能会被禁止6个月。

+   玩家加入游戏并[懒得获得经验点数](https://playvalorant.com/en-gb/news/dev/valorant-behavior-detection-and-penalty-updates/)，除了拖累他们的队友什么都不做，将会[受到惩罚](https://playvalorant.com/en-us/news/dev/valorant-systems-health-series-afk/)。

+   鼓励玩家举报有毒行为，并且不要参与，因为参与也可能会受到惩罚。

+   玩家使用[特定词语](https://support-valorant.riotgames.com/hc/en-us/articles/360044791253-Inappropriate-In-Game-Names)，无论是在聊天中还是作为用户名，都将被标记为有毒。

+   处罚有各种各样的大小、形状和持续时间，允许根据行为进行微调：警告、语音/聊天限制、减少经验值获取、减少排名评级、增加排队等待时间、排名游戏禁令、全局禁令。

+   Valorant[发布了](https://playvalorant.com/en-us/news/dev/valorant-systems-health-series-smurf-detection/)他们减少二次账号使用的方法；承认虽然拥有多个账号来二次账号/交易/逃避封禁/... 是不可取的，但有些人正在使用它们与排名更高/更低的朋友一起玩。因此，虽然他们采取了措施来检测和减少多账号，但他们也放宽了玩家一起玩的最大排名差异，这显著减少了备用账户的使用量，但也没有在可衡量的方式上改变比赛的公平性。

## 结论

这一切都很好，但是它有效吗？根据来自[彩虹六号围攻](https://www.ubisoft.com/en-us/game/rainbow-six/siege/player-protection)的数据：[Valorant](https://playvalorant.com/en-us/news/tags/game-health-series/)，[Call of Duty: Modern Warfare 2](https://www.callofduty.com/blog/2023/06/call-of-duty-ricochet-anti-cheat-season-04-update)，…… 这些措施的确效果相当不错，并且可能比仅采取技术措施提供更好的结果。它们还更便宜，因为远离有害行为并不像直接禁止他们那样会减少玩家数量。很高兴看到视频游戏行业意识到作弊和滥用/毒性可以通过类似的非技术手段解决，并且这两种方法是互补的。这与其他地方形成了鲜明对比，在其他地方技术解决方案被视为唯一可能的补救措施，尤其是在我们机器学习无所不在的时代。

## 来源和资源
