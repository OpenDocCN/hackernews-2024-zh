<!--yml

category: 未分类

date: 2024-05-27 14:50:05

-->

# [Arm 架构二十年的工作经历](https://marcin.juszkiewicz.com.pl/2024/02/12/twenty-years-of-my-work-with-arm-architecture/) – Marcin Juszkiewicz

> 来源：[https://marcin.juszkiewicz.com.pl/2024/02/12/twenty-years-of-my-work-with-arm-architecture/](https://marcin.juszkiewicz.com.pl/2024/02/12/twenty-years-of-my-work-with-arm-architecture/)

二十年前，我购买了 Sharp Zaurus SL-5500 Linux PDA。内置 StrongARM SA1110。当时我并不知道它会以怎样的方式结束…

### 爱好

起初这只是一个爱好。发现与之前运行 PalmOS 的设备相比，我能做什么是很有趣的。SharpROM，OpenZaurus 3.2，OpenZaurus 3.3-pre1 等等。新应用程序，任务栏插件…

之后我想要构建一些东西。找到了 OpenZaurus 的工具链，尝试了一下，然后寻求帮助。结果得到了“忘掉这个工具链吧，我们有一个新玩具：OpenEmbedded”。

我深陷其中…

我花了一些时间来熟悉它（问了很多初学者问题）。然后尝试了很多次才成功获得可引导的映像。结果发现我的映像是第一个能在 SL-5500 设备上正确引导的。所以我收到了 OpenEmbedded 的写权限，并带有“现在你可以合并所有内容”的消息。

#### OpenZaurus

我在 OpenZaurus 发行版上工作了三年。在我空闲时间里。从用户开始，成为开发人员，最终成为 [发行经理](/2006/03/18/openzaurus-354-released/)。在这些年里，我从 SL-5500 迁移到了 c760（由用户捐赠）。在这段时间里我的桌上有几个其他 Zaurus 型号的设备。

那些是有趣的岁月。我学到了很多关于 Linux 文件系统结构，打包工作原理，所有编译时与安装时依赖性的区别，处理升级等内容。以及如何与其他开发人员和用户合作。还有如何避开那些该被忽略的人。

### 嵌入式 Linux 开发者

曾经，与 Arm 架构的工作从一个爱好转变为日常工作。

OpenedHand 的 Matthew Allum 向我提出了全职合同，所以 [我离开了我的日常工作](/2006/12/15/job-change/)（作为 PHP 程序员），开始致力于 Poky Linux 及其 OpenEmbedded 版本的工作。

新设备、新的有趣挑战、新的同事和客户。适用于开发的大型开发板（那些有许多有趣连接器的），各种设备的原型…

在 OpenedHand 之后，还有其他公司对我的服务感兴趣。如 Bug Labs 的基于 Java 的系统在 Poky Linux 上，Vernier 的学校掌上电脑等。依然是嵌入式 Linux。

### 受邀演讲者

我的第一个会议是 FOSDEM 2007\. 后来还参加了几个其他会议，但我更多时候是作为与会者而不是演讲者。

2009 年的转折点是 —— Andrea Gallo 来自 ST-Ericsson 邀请我在 [他们的研讨会](/2009/10/19/st-ericsson-community-workshop-2009/) 中在格勒诺布尔做一个演讲。我的演讲是关于在 Poky Linux 和 OpenEmbedded 中支持他们的开发板 NHK-15。

第二天是 [ELCE，我在那里发表了演讲](/2009/10/27/elc-e-2009/)，名为“使用 OpenEmbedded 进行黑客活动”。那也是我第一次见到 Leif Lindholm 和 Jon Masters 的会议。

### Linaro

2010 年来临，我已经在嵌入式 Linux 工作中获得了三年多的付费经验。[Canonical 询问并签署了文件。](/2010/04/06/another-job-change/) 然后是另一份文件，我成为了最早的 20 个在 Linaro 工作的人之一。致力于改进 Linux 上的 Arm 支持。

那个时候，与 Arm 相关的工作意味着嵌入式工作，只是使用主要发行版而不是嵌入式发行版。所有那些特定设备的内核、镜像等。

我为 Debian/Ubuntu 做了交叉编译工具链，使用 OpenEmbedded 进行 AArch64 的开发和其他几项任务。

### Red Hat

2013 年，我结束了与 Canonical 的合同，[离开了 Linaro](/2013/05/31/my-time-at-linaro-is-over/)，并且[加入了 Red Hat](/2013/08/01/new-job-senior-software-engineer-red-hat/)。并见识了 Arm 世界的另一面。

我没有做开发板或 SBC 系统。这个项目是让 Red Hat Enterprise Linux 发行版在 64 位 Arm 服务器上运行。甚至在这些服务器还不存在的时候……

那是一个有趣的时光。在非常慢的系统仿真器中进行本地构建需要数小时/数天。原型服务器速度更快，但也非常不稳定。

无论如何，我们交付了。[RHEL 7.2 提供了 AArch64 支持。](/2015/11/20/red-hat-enterprise-linux-server-for-arm-7-2-development-preview-released/) 无数的软件包在 RHEL 和 Fedora 中得到了修复。

### Linaro 再次

2016 年，[我再次加入了 Linaro](/2016/04/08/back-linaro-org/)（这次是作为 Red Hat 的工程师）。为数据中心和云服务的 Arm 服务器工作。

虚拟化、OpenStack、大数据、Arm 服务器仿真等。

### 致谢

在这段旅程中帮助我的人有很多。

首先是 Anna Wagner-Juszkiewicz — 没有她的耐心和促使我创建自己的公司，我不可能走到今天。

然后是 OpenEmbedded 团队的成员：

+   Philip Balister

+   Phil Blundell

+   Florian Boor

+   Holger Freyther

+   Koen Kooi

+   Christopher Larson

+   Mickey Lauer

+   Richard Purdie

+   Holger Schurig

+   以及无数的贡献者

与工作变动有关的一些人：

+   Cliff Brake（我第一份自由职业 OE 工作）

+   Tim Bird（CELF 是我的第一个客户）

+   Matthew Allum（全职合同，所以我离开了网页开发）

+   Christian Reis（Canonical/Linaro 工作）

+   Jon Masters（“来吧，和我们一起在 Red Hat 工作”）

Linaro 的朋友们：

+   Fathi Boudra

+   Andrea Gallo

+   Gema Gomez

+   Lee Jones

+   Leif Lindholm

+   Sahaj Sarup

+   Ebba Simpson

+   Riku Voipio

+   Linus Walleij

+   Wookey

当然，我无法列出每个人。在这二十年间，我遇到了许多人，有些成为了朋友，有些则成为了对手。有些面孔/名字我记得（还有更多我应该记得的）。还有些人认出了我，而我却认不出他们（抱歉朋友们）。

### 名人堂架子

有一个计划来展示我过去的几个设备：

+   Sharp Zaurus SL-5500（一切的开始）

+   Atmel AT91SAM9263EK（我拥有的第一个开发板）

+   Applied Micro Mustang（第一个 AArch64 服务器）

而谁知道，也许还有其他硬件，但首先需要获取它。有些已经被回收利用，有些不得不被销毁。

### 未来的计划？

我计划继续在Arm架构上工作。它仍然给我带来乐趣。

而谁知道，也许有一天，符合标准的基于AArch64的系统将取代我的x86-64台式机。所有那些UEFI、ACPI、SBSA、SBBR、SystemReady的概念，在一台正常工作的硬件中。只是不用花上成千上万欧元去购买它。
