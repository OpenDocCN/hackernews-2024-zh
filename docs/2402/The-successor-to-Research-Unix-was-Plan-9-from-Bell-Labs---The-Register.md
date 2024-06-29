<!--yml

category: 未分类

date: 2024-05-27 15:05:58

-->

# 研究 Unix 的后继者是来自贝尔实验室的 Plan 9 • The Register

> 来源：[https://www.theregister.com/2024/02/21/successor_to_unix_plan_9/](https://www.theregister.com/2024/02/21/successor_to_unix_plan_9/)

FOSDEM 2024 要向前进，就必须放下过去。在上世纪90年代，这意味着不兼容，但现在不再是这样了。

本文是基于 *The Reg* FOSS desk 在 [FOSDEM 2024](https://fosdem.org/2024/schedule/event/fosdem-2024-3095-one-way-forward-finding-a-path-to-what-comes-after-unix/) 上的演讲的第三篇文章。

这个短系列的第一部分讨论了 [软件臃肿的问题](https://www.theregister.com/2024/02/12/drowning_in_code/)。第二部分讨论了 [UNIX 的历史](https://www.theregister.com/2024/02/16/what_is_unix/)，以及在商业供应商采纳早期版本并将其推向市场后，贝尔实验室如何继续其开发。

这种开发模式的一个结果是第八版 Unix 对行业影响不大，第九版和第十版也鲜有显著引用。尽管如此，工作仍在继续，但紧随第十版之后的不再被称为 "Unix"。取而代之的是 "来自贝尔实验室的 Plan 9"。这一次，我们想探讨为什么 Plan 9 最初没有被称为 Unix，以及如何利用21世纪的硬件和软件特性来解决这个问题。

*The Reg* FOSS desk 多次 [关注 Plan 9](https://www.theregister.com/Print/2013/11/01/25_alternative_pc_operating_systems/)，包括对更近期的 [9Front 最活跃的分支进行了审视](https://www.theregister.com/2022/11/02/plan_9_fork_9front/)。去年，我们 [讨论了 9Front 的重要性](https://www.theregister.com/2023/12/01/9front_humanbiologics/)，特别是将其与现代 Linux 的规模进行比较。

Plan 9 将导致 Unix 的概念重新思考，以适应上世纪90年代。与任何大的变革一样，这样做会产生后果，并非所有后果都是正面的。

### 好的部分

Plan 9 在某种程度上是 Unix 和 C 核心概念的第二次实现，但是重新考虑了面向网络图形工作站的世界。它汲取了80年代末计算机界的许多潮流理念，既有学术理论，也有当时计算机行业的思想，并通过两位伟大导师 [肯尼斯·汤普逊](https://www.theregister.com/2023/03/17/ken_thompson_is_a_maccie/) 和 [丹尼斯·里奇](https://www.theregister.com/2011/10/13/dennis_ritchie_obituary/)（[及其学生们](https://www.theregister.com/2011/05/05/google_go/?page=2)）的疲惫眼光重新解释了它们 —— 他们可以被认为是设计上的天才，看到他们以前的好想法被误解和曲解。

在 Plan 9 中，网络是前置的。Unix 之所以不是这样，有很多合理的理由 – 它在设计和建设时正值 [局域网](https://www.cisco.com/c/en/us/products/switches/what-is-a-lan-local-area-network.html) 的发明时期。[UNIX 第四版](https://gunkies.org/wiki/UNIX_Fourth_Edition)，第一个用 C 语言编写的版本，发布于 1973 年 – 同年也是 [以太网的第一个版本](https://www.theregister.com/2023/06/30/ethernet_50th_birthday/)。

Plan 9 将网络功能置于设计的核心。而 Unix 后来被用作独立工作站的最常见操作系统，Plan 9 则是为计算机集群设计的，一些是图形桌面，一些是共享服务器。

Plan 9 接受了“一切皆为文件”的理念，并使其变为现实。在 Unix 中，这实际上只是一个空洞的营销口号。许多东西，比如进程 ID 的树，不是文件系统的一部分。在某些版本中，它们可以通过附加的额外部分看到，比如在 [`/proc`](https://eschrock.dtrace.org/2004/06/25/a-brief-history-of-proc/) 内存中的[伪文件系统](https://superuser.com/questions/1198292/what-is-a-pseudo-file-system-in-linux) – 但在 macOS 中你不会找到 `/proc`。

在 [Rio](https://wiki.xxiivv.com/site/rio.html) 下，当前的 Plan 9 窗口系统（取代了 [较旧的 8½ 系统](http://doc.cat-v.org/plan_9/4th_edition/papers/812/)），屏幕上的窗口被表示为文件系统中的目录，它们的内容是文件。网络上的其他 Plan 9 机器在你的文件系统中可见 – 当然要受访问权限的限制，并且你可以在自己的文件系统中直接看到其中一些文件系统的内容。

因为一切都是文件，将窗口显示在另一台机器上可以如同创建一个目录并填充一些文件那样简单。你可以在其他计算机上启动程序，但在你自己的计算机上显示结果，而不需要任何 X11 或者任何可见的网络。

这意味着所有关于 `telnet`、`rsh`、`ssh` 以及 [X forwarding](https://goteleport.com/blog/x11-forwarding/) 等的 Unix 风格的东西都…消失了。这让 X11 看起来非常复杂，也让 Wayland 看起来像是由微软发明的。

由于一切都通过文件系统进行，这消除了微内核设计中最大的复杂问题之一 – 进程间通信。Plan 9 并不是一个微内核，它更多地使微内核的定义特性在某些方面变得不那么重要。

Plan 9 不仅仅是一个替代内核或者其上的各层。它还是一个集群系统、一个网络文件系统，以及一个容器管理系统。从功能块的角度来看，我们在谈论取代 Linux，还有 Ceph 或者 Gluster 或者其他类似的东西，以及所有的虚拟化程序、所有的容器系统，以及 Kubernetes 等等。

由于它被构想为处理所有这些问题的一个工具，因此它比在 Linux 之上堆叠的复杂性的几十亿字节要简单得多，这种复杂性使得 Linux 能够执行这些操作。

即便如此，Plan 9 却非常小。去年十一月我写作时，我和一些来自 9front 社区的人谈过，其中一位给了我一些数据。

（这包括所有应用程序、演示和游戏，包括 Doom。）

至于复杂性：Plan 9 内核有 *38* 个系统调用。

我没有找到 Linux 的确切数字，但我有一些估计。截至 2016 年，内核 4.7 有大约 335 个。到了 2017 年，这个数字增加到 341。我尝试估算内核 6.8 的数字，我认为在所有架构上大约有 520 个。

举个例子，Kubernetes 单独就有大约两百万行代码。这是将一个集群管理层添加到系统中，而不是将其嵌入到设计中的代价。

### 对文件系统的简要说明

如果听起来有点太理论化，那么当一切都在文件系统中时，其中一个问题当然是决定一切的位置在哪里。

2011 年，我[写过关于容器的文章](https://www.theregister.com//Print/2011/07/18/brief_history_of_virtualisation_part_3)，并称其为系统管理员的梦想成真。那正是 Docker 推出的同时。

容器就像是增强版的 `chroot`。在给定进程中，所有文件路径都从一个新的根目录开始，而由于在 Unix 中几乎所有东西都是相对于根目录的，改变这一点会将该进程（以及任何子进程）与操作系统的其余部分隔离开来。

现在在 Linux 上有几种形式的容器化，但[其中一些](https://www.theregister.com/2016/07/05/containers_state_of_the_art/)使用了 Linux 内核的[`cgroups`特性](https://www.theregister.com/2014/05/23/google_containerization_two_billion/)，这是来自 Google 的。它通过将它们置于不同的命名空间中来将容器内的进程互相隔离。

Unix 的问题在于它并不只有一个全局命名空间，因为当它设计时，这些概念还比较新而且不成熟。Unix 中不属于单一命名空间的概念包括用户 ID、组 ID、进程 ID，以及很多并未在或通过单一全局文件系统中表示的东西。

这就是为什么[cgroups 命名空间](https://man7.org/linux/man-pages/man7/cgroup_namespaces.7.html)存在的原因 – 为了将这些命名空间进一步分离，以实现更完整的隔离。问题在于它是后来添加的功能，因此就像任何依赖 `/proc` 的东西在 macOS 上无法工作一样 – [或者在 OpenBSD 上](https://stackoverflow.com/questions/20465780/process-information-in-openbsd) – 依赖 cgroups 命名空间的东西在 FreeBSD 上也无法工作。

### 回到 Plan

命名空间是Plan 9设计的[核心部分](http://doc.cat-v.org/plan_9/4th_edition/papers/n)，Plan 9下的每个单独进程都有其自己的命名空间。因为OS的各个部分之间的通信更多地通过文件系统进行。因此，每个进程都有自己对文件系统的视图。

因此，实际上，每个进程都在一个容器中……注意，在一个[1990年宣布的项目](https://dl.acm.org/doi/10.1145/155848.155861)中，在FreeBSD 4.0中的[第一个版本](https://lists.freebsd.org/pipermail/freebsd-announce/2000-March/000529.html)发布前十年。

### 坏的部分：它失败了

如果 Plan 9 如此聪明，为什么我们不都在使用它呢？嗯，有一些原因很容易列举。

首先，就像 Unix 本身在开始时一样，它是一个实验性的操作系统。在早期，它们并不是用于生产使用的，Plan 9 是研究操作系统概念的工具。直到第二版在[1995年](https://web.archive.org/web/20080706034735/http://9fans.net/archive/1995/07/16)才获得商业使用许可。

其次，在20世纪初，Plan 9并不是开源的。但在2000年，第三版在其自定义许可协议下[发布为FOSS](https://www.theregister.com/2000/06/09/plan_9_goes_open_source/)，然后在2014年它被[重新许可为GPL](https://www.theregister.com/2014/02/14/plan_9_moves_to_gnu_space/)。

第三，Plan 9 非常清晰、极简的概念设计带来了一些惩罚。由于没有对“真实”文件系统的“真正”基础视图，这意味着 Plan 9 简单地不做很多事情。没有符号链接，也没有硬链接。

最初，它的`mv`命令简单地[复制文件然后删除原始文件](https://groups.google.com/g/comp.os.plan9/c/1ghzz_q1NlE)。这是通过[9P协议](http://9p.cat-v.org/documentation/)，使得内核与本地文件系统通信的方式与其与其他机器上的文件系统通信的方式相同的代价。

(是的，这可能会慢，听起来非常限制，但是这只秃鹫已经在他的职业生涯中度过了五年，当时第一个带有硬链接的微软操作系统 Windows NT 3.1 发布了。直到 MS-DOS 5 在 1991 年发布，DOS 才没有`MOVE`命令，且在 Windows 95 的“快捷方式”之前，没有任何 Microsoft 操作系统有类似的符号链接。)

Plan 9 是不友好的、不宽容的，使用起来很困难。我不是某种狂热的 Plan 9 支持者。我自己并不经常使用它，虽然我在虚拟机中保留了一个 9front 的副本，多年来，我偶尔会拿来新版本玩耍。

[9front](http://9front.org/) 相比于 AT&T 版本已经进步了很长一段路。像我这样的非专家可以安装并尝试它，这比我在原始版本中能做的更多。但正如[FQA页面所说](http://fqa.9front.org/fqa0.html)：

Plan 9 有一个图形用户界面，但它非常奇怪。但它确实是一个非常奇怪的操作系统。

问题是，当我开始使用Linux时——大约是在1995年的[Slackware 3.0](https://www.linux.co.cr/distributions/review/1995/0930.html)，使用的内核是1.0版本——Linux非常基础，非常不友好，实际上用处不大。自那以后，Linux走过了非常长的路程，即使它的[桌面渗透率仍然很低](https://www.theregister.com/2023/07/18/linux_desktop_debate/)。

致命的是，Plan 9与任何Unix都有足够的不同，以至于与Unix不兼容。它的创建者们有足够的理由没有把它称为Unix第十一版。它在设计的基本部分上做了根本性的改变，这意味着无法将现有的源代码带过来并重新编译。

举一个微不足道的例子，Plan 9的C版本[禁止嵌套的`#include`指令](http://doc.cat-v.org/plan_9/4th_edition/papers/comp)。大多数非平凡的C程序包含大量文件，而这些文件都包含大量的`#include`语句。在Plan 9的C中，只有程序中的顶层C文件可以`#include`其他文件，而且头文件根本不允许`#include`。这意味着程序员需要做更多的工作，但是[这样做可以使编译速度更快](https://go.dev/talks/2012/splash.article)。

### 它很好，但不够好

为了取代一个现有的、根深蒂固的产品——在本例中是传统的Unix——一个新的继任产品必须显著优于其前身，使得切换的成本值得，即使这个成本只是时间和精力，而不是金钱。我们不知道这个规则有没有一个简洁的术语，但可能有一个，类似于克莱顿·克里斯滕森的[革命性创新](https://en.wikipedia.org/wiki/Disruptive_innovation)。用一个旧的OS/2营销口号来概括，Plan 9是“比Unix更好的Unix”——但它还不够好。

另一个小的旁注是，Plan 9并不是原始Unix系列的终结。研究Unix有十个版本，然后是一个如此不同的继任者，它不再是Unix了。那就是Plan 9，在某种意义上，它是“UNIX 2.0”。

Plan 9本身也有一个非常不同的继任者。它被称为[Inferno](https://www.vitanuova.com/inferno/)，某种意义上可以说是“UNIX 3.0”。今天文章已经够长了，我就不在这里详细讨论它了，但这确实很有趣，值得深入探索。

9front非常聪明和令人印象深刻。它也很奇怪和难以理解，但是当Plan 9首次商业发布时，Linux也是如此……现在看看吧。Linux驱动着ChromeOS和Android，这意味着除了无数不可见的服务器外，数十亿人每天都在使用Linux，即使他们从未听说过它。

即使对于那些了解“操作系统”并且关心他们使用哪一个的少数人来说，大多数“Linux发行版”现在也相当容易。

但Plan 9不是Unix，不能运行Unix程序。你不能简单地像对Windows、macOS和所有BSDs做的那样，把Firefox或LibreOffice移植到Plan 9上。

有一些工具可以帮助处理这样的事情，但它们有点简陋，因为那些重视Plan 9的人要么不想要这样的东西，要么在其他计算机上进行这样的操作。

例如，有一个Plan 9 Linux模拟器，类似于[FreeBSD的linuxulator](https://wiki.freebsd.org/Linuxulator)。Plan 9的称为[Linuxemu](https://web.archive.org/web/20231119190241/http://9p.io/wiki/plan9/Linux_emulation/index.html))。它只支持32位，但直到几年前，[Solaris LX zones](https://docs.oracle.com/cd/E19455-01/817-1592/gepea/index.html)也是如此——然后Joyent为[SmartOS实现](https://wiki.smartos.org/lx-branded-zones/)现代化了它们。

有一个用于移植源代码的Linux兼容性层，称为[APE](http://doc.cat-v.org/plan_9/4th_edition/papers/ape)。还有一个X11服务器，[Equis](https://web.archive.org/web/20230927210129/https://9p.io/wiki/plan9/X11_installation/index.html)。

更重要的是，9front有一个称为VMX的[hypervisor](http://man.9front.org/1/vmx)，使用Intel和AMD芯片的内置虚拟化。甚至有一个[安装Void Linux的指南](https://wiki.9front.org/vmx)。虚拟机只能有一个核心，但它确实存在并且可以工作，尽管文档说，VMX可能会崩溃您的内核。

所以这是我的想法。

现在是2024年。我们现在都有64位的机器，而x86-32正在迅速消失。大多数发行版不再支持它；甚至最后的一些坚持者之一，Debian，也计划在下一个版本中[停止支持](https://www.theregister.com/2023/12/19/debian_to_drop_x86_32/)。

现代PC具有大量内存和大量存储空间。在Linux领域，有许多像[Amazon's Firecracker](https://www.theregister.com/2018/11/27/aws_sets_firecracker/)这样的“微型虚拟机”。最近，它还[增加了对FreeBSD客户端的支持](https://www.theregister.com/2023/08/29/freebsd_boots_in_25ms/)。还有其他的，比如[Intel Clear Containers](https://www.theregister.com/2015/05/21/intel_wants_containers_to_be_alone_together_naturally/)，它们合并到[Kata Containers](https://www.theregister.com/2018/05/23/vmcontainer_chimera_kata_containers_emerges_from_lab/)等。

微型虚拟机可以在几毫秒内启动一个客户端，并且它们用于["无服务器"](https://www.theregister.com/Tag/Serverless/)云计算。还有一个谎言，但我们不要深入讨论。

这个想法是小型虚拟机（**microVMs**），它们与容器一样快速和短暂。您启动它们，通常只运行一个程序，程序完成后，虚拟机就退出。

如果在9front上运行微型虚拟机是可能的，你原则上可以启动一个Linux应用程序，使用它，然后再退出，而在现代计算机上几乎不会注意到从启动虚拟机开始的延迟。

这也减少了对 Linux 模拟器的需求。这是一项复杂的任务，需要不断维护以跟踪当前 Linux 内核的变化目标。所以就此放手吧。

这里有一个 X 服务器。如果它已经在运行，或者有一个依赖项，新的微虚拟机可以连接到它，从而在 Rio 中无缝运行。我怀疑 Equis 需要一些关爱和维护，在最初可能会很不稳定，但这项技术的组成部分已经存在。

这方面已经有了一个模型。[Qubes](https://www.qubes-os.org/) 已经存在[十多年](https://www.theregister.com/2012/09/05/qubes_secure_os_released/)，并且正在[积极开发中](https://www.theregister.com/2022/02/09/qubes_vm_upgrade/)。所有用户程序都在专用的虚拟机中运行。在 Qubes 的情况下，这是为了增加安全性。Qubes 基于 Fedora 和 Xen hypervisor，但这里重要的是概念而非实现。

这不必专注于图形应用程序。只要微虚拟机中的应用程序从文件系统获取输入并将其输出到文件系统，这就可能使 Linux 应用程序能够与本机 Plan 9 应用程序在某种程度上实现互操作，而不会使 Plan 9 充斥着现代 Linux 系统的复杂性。混乱的东西被整齐地盒装起来。

这里存在潜力，逐步将一些工具从 Unix 带到 Plan 9 的方式在 Plan 9 构建时是不可能的。第一阶段：在 VM 中隔离应用程序。第二阶段：将最有用的较小、较简单的应用程序带到 APE 或类似的地方。第三阶段：可能将基本的东西重写为本机 Plan 9 应用程序。（对于控制台应用程序，像 [ncurses](https://tldp.org/HOWTO/NCURSES-Programming-HOWTO/intro.html) 这样的本机版本会使生活变得轻松许多。）

随着时间的推移，这样的转变无疑意味着 Plan 9 本身逐渐变得更大更复杂……但是已经有了 Plan 9 的后继者，即 [Inferno OS](https://github.com/inferno-os/inferno-os)。正如[他人之前观察到的](https://yotam.net/posts/linux-namespaces-are-a-poor-mans-plan9-namespaces/)：

这只是众多选择之一。但这是通向更清洁、更少臃肿未来的潜在路径。这很快就会有后续，原始的 FOSDEM 演讲的最后部分将讨论将 Linux 精简为专用的虚拟机操作系统，剥离所有裸机功能。®
