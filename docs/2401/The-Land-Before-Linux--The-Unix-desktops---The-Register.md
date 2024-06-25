<!--yml

类别：未分类

日期：2024-05-27 15:17:19

-->

# 《Linux 诞生之前：Unix 桌面》• The Register

> 来源：[`www.theregister.com/2024/01/27/opinion_column/`](https://www.theregister.com/2024/01/27/opinion_column/)

观点今天，由于 Android 和 ChromeOS，Linux 是一个重要的最终用户操作系统。但在 Linux 之前，有重要的 Unix 桌面，尽管其中大多数从未被注意到。

早在 1993 年，我主持了 PC Magazine 的一个关于 Unix 桌面的 [特别评测](https://books.google.com/books?id=jMKfH6i9OcYC&pg=PA220&dq=Vaughan-Nichols&hl=en&sa=X&ved=2ahUKEwi8pdG5gveDAxXhlmoFHSYsDdcQ6AF6BAgMEAI#v=onepage&q=Vaughan-Nichols&f=false)。是的，没错，在我成为 Linux 桌面用户之前，我是一名 Unix 用户。事实上，自从 1979 年 [2BSD Unix](https://opensource.fandom.com/wiki/Berkeley_Software_Distribution) 出现以来，我一直是一名 Unix 粉丝。但是到了 1993 年，出现了许多 Unix 桌面，我说服了杂志让我来试驾它们。

我和我的团队审查了来自 Consensys、Dell、Interactive Unix、SCO、Univel、Sun 和 NeXT 的 Unix 发行版。我们还看了但没有审查来自 UHC、Microport 和其他公司的 Unix。我敢保证很多人从未听说过它们。

Linux 呢？是的，它已经存在了，而且我当时已经在使用它了。但是当时最先进的 Linux 发行版是 [Softlanding Linux System (SLS)](https://archiveos.org/sls/)，我无法说服我的编辑——甚至是我自己——它是可以审查的。而我将会审查的第一个版本，[Slackware](http://www.slackware.com/)，至今仍与我们同在——但那时还有好几个月的时间。

今天，只有 Dell 仍然存在，而且显然不是因为它的 System V Release 4（SVR4）Unix 发行版。然而，其中一个早期的 Unix 桌面至今仍然健在，并且在约 [四分之一的桌面](https://www.statista.com/statistics/218089/global-market-share-of-windows-7/) 上运行。

当然，那个操作系统就是 macOS X，直接源自 [NeXT 的 NeXTSTEP](https://www.zdnet.com/article/steve-jobs-the-next-years/)。你可以说，基于多线程、[多处理器微内核操作系统 Mach](https://developer.apple.com/library/archive/documentation/Darwin/Conceptual/KernelProgramming/Mach/Mach.html)、[BSD Unix](https://docs.freebsd.org/en/articles/explaining-bsd/) 和 [开源 Darwin](https://github.com/apple/darwin-xnu)，macOS 是所有 Unix 操作系统中最成功的。

那时候看起来确实不是这样。并不是 Windows 比 Unix 更好。在 1993 年，Unix 的竞争对手，如果可以这么说的话，是 Windows 3.1 和 NT 3.1。

[NT](https://www.theregister.com/2023/12/19/windows_nt_30_years_on/)，尤其是在那个时候，是一个糟糕的服务器操作系统的一个笑话。NT 只有在 Windows NT 3.5 发布之后才开始变得重要。

Windows 胜过 Unix 的原因有很多。其中一个最重要的原因是微软确保硬件和软件供应商要么配合微软，要么[无法获得 Windows 或 Microsoft Office 的访问权限](http://www.practical-tech.com/business/b020298.htm)。

那时这是一笔巨大的交易。今天，我们认为 Mac 与 Windows PC 竞争对手或者比其更优秀。但那时不是这样的。史蒂夫·乔布斯已经被解雇，在苹果的 1993 年年度报告中，公司报告称其[净收入](https://www.nytimes.com/1993/10/15/business/company-reports-a-small-profit-for-apple-computer.html)下降了 97%。

但是，尽管历史上[不光彩的商业交易](https://www.theregister.com/2000/04/04/judge_finds_against_ms/)对微软的成功至关重要，但微软并不需要作弊来取胜。Unix 公司正在自掘坟墓。

你看，虽然有许多尝试为 Unix 创建软件开发标准，但它们过于泛泛，无济于事——例如[便携操作系统接口（POSIX）](https://www.techtarget.com/whatis/definition/POSIX-Portable-Operating-System-Interface)——或者陷入了开放系统基金会和 Unix 国际之间的商业联盟之争，后者被称为[Unix 战争](https://klarasystems.com/articles/unix-wars-the-battle-for-standards/)。

当 Unix 公司忙于相互撕裂时，微软笑傲银行。核心问题在于 Unix 公司无法达成软件标准。独立软件供应商（ISV）必须为每个 Unix 平台编写应用程序。这些平台的桌面市场份额微乎其微。对于程序员来说，为 SCO OpenDesktop（又称 OpenDeathtrap）编写一个应用程序版本，为 NeXTStep 编写另一个版本，再为 SunOS 编写一个版本，根本没有商业意义。

那听起来耳熟吗？Linux 桌面依然存在这样的问题，这也是我对[Linux 容器化桌面应用](https://www.theregister.com/2023/06/09/will_flatpak_and_snap_replace/)，如红帽的 Flatpak 和 Canonical 的 Snap，大力支持的原因。

当两方终于在 1996 年加入[开放集团](https://www.opengroup.org/membership/forums/platform/unix)并达成和平时，为时已晚。Unix 在传统桌面上被排挤出局，工作站几乎成了 Sun Microsystems 的专利。

那么，Linux 是如何取得胜利的呢？嗯，它相对于 Unix 发行版有两个主要优势。第一个是它是开源的。在开源的优势中，优秀的代码生存，糟糕的代码消亡。特别是我认为 Linux 使用 GNU 通用公共许可证（GPL）功不可没。

毕竟，如果成功所需的只是开源代码，我们都会运行纯 BSD 操作系统，如[FreeBSD](https://www.freebsd.org/)、[DragonflyBSD](https://www.dragonflybsd.org/)和[GhostBSD](https://ghostbsd.org/)。相反，虽然 BSD Unix 系统仍然很重要，但它们没有像 Linux 那样的市场份额。

今天的 Linux 基金会开源供应链安全总监大卫·惠勒（David Wheeler）解释说，这是因为 BSD 许可证一直很麻烦，因为每隔几年就会有人说，“[嘿，让我们基于这个 BSD 代码创办一家公司！](https://lwn.net/Articles/197875/)”...他们将*BSD 代码引入，并吸引了一些最优秀的 BSD 开发人员，然后编写了一个专有的派生产品。但是作为专有供应商，他们的分支变得昂贵且难以自行维护，最终公司倒闭。反复循环。

“与此同时，GPL 在法律上对主要商业公司强制执行了一项财团……[所有人]都在做出贡献，并且因为其他人也被法律要求这样做而感到安全。它基本上创造了一个合作的‘安全’区域。”

Linux 的另一个致命优势是有林纳斯·托瓦兹（Linus Torvalds）。有了托瓦兹作为 Linux 的唯一领导者，它避免了旧 Unix 的内斗陷阱。

这不仅仅是因为托瓦兹是一位天才开发者。托瓦兹幽默的头衔可能是终身仁慈独裁者，但多年来，[托瓦兹学会了与他人良好地合作和共事](https://www.theregister.com/2018/10/22/linus_torvalds_back/)。

传闻托瓦兹有点刻薄，而且他确实不容易对付，但我参加过许多[Linux Plumbers 会议](https://lpc.events/)。在那里，我看到他和顶级 Linux 内核开发人员互相合作，毫无戏剧性。如今的 Linux 是一个集体的努力。

如果 Linux 只有托瓦兹一个人，我会担心操作系统的未来。林纳斯是一个了不起的人和一位优秀的程序员，但如果这就是 Linux 成功的全部，我们距离它的终结只有一步之遥。

相反，Linux 发行商和开发人员已经吸取了他们的 Unix 历史教训。

他们意识到，要想成功，不仅需要开源，还需要开放标准和共识来制作一个成功的桌面操作系统。

我们可能永远也不会见到传说中的“Linux 桌面年”，但是由于 Android 和 Chrome OS 的出现，Linux 已经成为一款顶级的终端用户操作系统。虽然花了很长时间，但是 Unix，通过 Linux，终于成为了一款顶级的终端用户操作系统。®
