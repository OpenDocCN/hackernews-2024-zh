<!--yml

类别：未分类

日期：2024年5月27日 14:56:27

-->

# 任天堂摧毁Yuzu后如何影响模拟器界 - The Verge

> 来源：[https://www.theverge.com/24098640/nintendo-emulator-yuzu-lawsuit-switch-aftermath](https://www.theverge.com/24098640/nintendo-emulator-yuzu-lawsuit-switch-aftermath)

[2024年3月4日，任天堂将Yuzu开发者告上法庭](/2024/3/4/24090357/nintendo-yuzu-emulator-lawsuit-settlement)，这不仅仅是对一种无需任天堂Switch就能玩Switch游戏的主要方式的攻击，更是对任何建造视频游戏模拟器的人的警告。

现在已有七位开发者退出了项目、关闭了它们，或者完全离开了模拟器领域。其中留下的许多人正在团结一致，变得更加低调和谨慎，试图不要在自己背后打上标记。五位开发者拒绝与*The Verge*交谈，告诉我他们不想引起注意。甚至有一个人在我们开始后试图删除对我的问题的答案，突然害怕吸引媒体的关注。

除了Yuzu，模拟器界在一个星期内失去了这些：

+   任天堂3DS的Citra模拟器已经消失。

+   任天堂Game Boy Advance和Game Boy Color的Pizza Boy模拟器已经消失。

+   任天堂DS的Drastic模拟器目前是免费的，并将被移除。

+   Yuzu和Citra的首席开发者已经退出了模拟器开发。

+   Strato的首席开发者，一款Switch模拟器，也已经退出了模拟器开发。

+   Dynarmic，用于加速包括Yuzu在内的多个模拟器的开发，已经突然结束了。

+   Ryujinx的一位贡献者已经退出了这个项目。

+   PS2模拟器AetherSX2最终消失了（与此无关；[开发已于一年前暂停](https://www.reddit.com/r/EmulationOnAndroid/comments/1193rxk/aethersx2_is_dead_and_we_are_the_reason/)）

并非所有人都如此害怕。其他四个模拟器团队告诉我，他们乐观地认为任天堂不会挑战他们，他们的法律立场很坚定，而Yuzu可能是一个异常罪证确凿的案例。一个从事模拟器开发已长达十年的老手告诉我，每个人都稍微担心了一点。

但是当我指出任天堂在法庭上不必证明任何事情时，他们都承认他们没有钱请律师。他们说，如果这家日本游戏巨头找上门来，他们可能会被迫妥协，就像Yuzu一样。“我会做我该做的事情，”这四个人中最有信心的告诉我。“我想要反抗...但与此同时，我知道我们存在是因为我们不挑衅任天堂。”

[这里有一个新的梗](https://www.reddit.com/r/yuzu/comments/1bbaq4s/the_real_current_situation/)，其中 Yuzu 被比作神话中的九头蛇：砍掉一头，两个头就会长出来。这在Yuzu（以及3DS模拟器Citra）的多个分支在其前身消亡后迅速出现方面部分是真实的：Suyu、Sudachi、Lemonade 和 Lime 是一些公开的名称。但他们并没有对任天堂竖中指：他们将任天堂的诉讼看作是如何**不**惹恼公司的指南。

在其法律投诉中，任天堂声称 Yuzu 正在“大规模地促进盗版行为”，并向用户提供了关于如何“使用非法拷贝的任天堂Switch游戏”来“让其运行”的“详细指导”，等等。好吧，不再有指南，说了与我交谈的Switch模拟器开发人员。

他们还说他们正在剔除一些使得可以轻松玩盗版游戏的Yuzu的部分。[正如*Ars Technica*报道的](https://arstechnica.com/gaming/2024/03/heres-how-the-makers-of-the-suyu-switch-emulator-plan-to-avoid-getting-sued/)，一个名为Suyu的分支版本将要求您在解密并播放任天堂游戏之前从您的Switch上带来固件、title.keys 和 prod.keys。在此之前只有一个是技术上必需的。（不要在意大多数人没有[容易被黑客攻击的第一代Switch](/2018/4/24/17276214/nintendo-switch-hack-jailbreak-security-homebrew-apps-games)，并且可能会从网络上下载这些东西。）

另一个分支的开发者告诉我，他计划做类似的事情，让用户“自行解决”，确保代码不会自动生成任何密钥。

*一个Suyu的管理员为我们总结了一下。*

大多数我交谈过的开发者也试图澄清他们并没有从任天堂那里获利。一个最初将早期访问版本锁在捐赠页面背后的人已经停止这样做，改为在GitHub上公开提供。另一个项目的负责人告诉我，永远不会有任何内容需要付费，目前也“绝对没有捐赠”。当我询问Dolphin模拟器时，去年面对任天堂的[轻微挑战](/2023/6/1/23745772/valve-nintendo-dolphin-emulator-steam-emails)时，我被告知它将[公开展示其微不足道的非营利预算](https://opencollective.com/dolphin-emu)，供任何人审查。

但我不知道这些步骤是否足以阻止任天堂再次行使其权力，特别是在模拟任天堂Switch这一主要赚钱机制时。关于这整个情况需要了解的一件事情：赌注比以往任何时候都高。模拟器通常落后于新的游戏机代数，因为数字化复制一个游戏机的功能和学习其机密可能需要大量计算资源和时间。但对于Switch而言，情况并非如此。

不仅黑客在原版 Switch 发布不到一年后发现了 [前所未有的漏洞](https://medium.com/@SoyLatteChen/inside-fus%C3%A9e-gel%C3%A9e-the-unpatchable-entrypoint-for-nintendo-switch-hacking-26f42026ada0)，模拟器界还设法开发了比原版 Switch 在其有限寿命内表现*更好*的软件。他们通过在类似 Steam Deck 和一些手机这样的 Switch 风格便携设备上运行的能力，挑战了任天堂的地盘。YouTuber 发布了模拟 Switch 游戏的教程，其中一些视频据称受到了任天堂的版权索赔威胁；甚至 Valve ([暂时](https://kotaku.com/yuzu-nintendo-switch-emulator-steam-deck-dock-valve-1849632836)) 在官方 Steam Deck 视频中展示了 Yuzu 模拟器。

但技术的发展速度也可能使得任天堂更容易挑战模拟器，几位开发者承认。虽然任天堂直到 Wii 时期才开始正式引入加密，但据说 Switch 有五层不同的加密层，而任天堂对 Yuzu 的投诉主要是指控该模拟器鼓励用户绕过这些保护。

这是一个法院尚未测试的论点，且至少有一个重要原因认为，如果尝试的话，任天堂可能不会赢得胜利。《数字千年版权法案》第 1201 条*确实*允许人们为“独立创建的计算机程序与其他程序的互操作性”而有限制地绕过复制保护，而海豚模拟器团队 [公开主张](https://dolphin-emu.org/blog/2023/07/20/what-happened-to-dolphin-on-steam/) 这甚至可以延伸到共享解密密钥。

然而，许多早期模拟器从未面临过这种挑战，再次强调，没有开发者有精力去搞清楚后果。“我们很幸运，我们使用的系统更简单，它们没有先进的加密技术，”一位专注于早期任天堂游戏主机的开发者说道。

另一种说法是，最安全的模拟器是为如此古老的游戏主机或以如此高水平仿真的主机设计的，它们甚至不需要用户提供受版权保护的 BIOS。例如，索尼 PSP 模拟器 PPSSPP [可以通过这样的方式自我辩护](https://www.ppsspp.org/docs/faq/#:~:text=Do%20I%20need%20a%20BIOS%20file%20to%20run%20PPSSPP%2C%20like%20I%20would%20with%20PSX/PS1%20and%20PS2%20emulators%3F)，它模拟一切，包括 BIOS 和操作系统，使用自己的代码。

但我与他们交谈的时间越长，越清楚地看到，模拟器的信徒们对 Yuzu 倒台感到略有动摇。没有人完全相信 Yuzu 做错了什么。

模拟器俱乐部的第一条规则：不要讨论盗版。

“Yuzu Discord非常谨慎，不提及盗版 — 他们甚至扫描bug报告的日志，以检查人们是否在使用自制的游戏副本”，一个分支贡献者告诉我。

“他们的头上有一把隐喻性的枪”，另一位称，对Yuzu承认的胡说八道持怀疑态度（链接：/2024/3/4/24090357/nintendo-yuzu-emulator-lawsuit-settlement#:~:text=also%20says%20Yuzu%20is%20%E2%80%9Cprimarily%20designed%20to%20circumvent%20and%20play%20Nintendo%20Switch%20games.%E2%80%9D），因为它“主要是为了绕过和玩任天堂Switch游戏”，因此违法。

我与之交谈的开发者中没有人面临巨大的损失。这不会影响他们的生计，也不会增加他们的负担。但他们仍然依赖于像任天堂这样的公司的恩惠。

如果您掌握有关仿真威胁或Yuzu实际情况的内幕消息，请通过sean@theverge.com与我联系。

大公司有时确实会从允许仿真发展中获益。由于仿真器，您今天可以在任天堂Switch和PlayStation 5上下载经典游戏。据说任天堂已从[其内部仿真团队](https://www.nerd.nintendo.com/)中招募人员，而索尼肯定也是，至少聘请了PlayStation 2模拟器PCSX2的一名开发者，以将PS2游戏带到PS4（链接：https://www.timeextension.com/features/meet-the-company-bringing-classic-games-to-switch-ps5-and-xbox-by-mistake）。

如今，他的公司Implicit Conversions与索尼合作，将PS1和PSP游戏移植到PS5，可通过仿真[下载索尼游戏](https://www.implicitconversions.com/#:~:text=identification%20purposes%20only.-,Shipped%20Titles,-About%20Us)如《Twisted Metal》和《Syphon Filter》。这是一个乐观的理由：也许任天堂会看到留下其余场景的至少一些好处。
