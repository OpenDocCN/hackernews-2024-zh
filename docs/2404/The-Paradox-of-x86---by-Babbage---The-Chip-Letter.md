<!--yml

category: 未分类

date: 2024-05-27 13:18:49

-->

# x86的悖论 - 由巴比奇 - The Chip Letter

> 来源：[https://thechipletter.substack.com/p/the-paradox-of-x86](https://thechipletter.substack.com/p/the-paradox-of-x86)

英格兰苏塞克斯的博迪亚姆城堡及其护城河。无论从哪个方向看，那条护城河看起来都难以跨越！ *由 WyrdLight.com，CC BY-SA 3.0, https://commons.wikimedia.org/w/index.php?curid=7910287*

*“护城河”是一个熟悉的商业概念。它是一种保护你免受竞争对手侵害的屏障。每个人都希望他们的企业受到强大护城河的保护。*

*现实生活中的物理护城河也可以阻止防御者离开其环绕的堡垒保护。但呆在护城河里也是有风险的。它让潜在的攻击者在外面积聚力量，直到他们准备突破护城河。*

*在本文中，我们将看一个实例，展示一个商业护城河导致了相同的现象。这是关于英特尔x86指令集架构（ISA）的故事。*

*指令集架构（ISA），即微处理器能理解和运行的“指令代码”集合，可能看起来是一个古怪的技术细节。但正如我们将要发现的那样，它已经是英特尔业务的关键部分近五十年，并且可以说直接影响了公司目前的问题根源。*

*这些问题对英特尔及其延伸至美国的经济和地缘政治后果巨大。这反过来导致美国政府需要向英特尔、台积电和三星提供数十亿美元的补贴。*

* * *

上周，英特尔宣布其新定义并分离的“英特尔晶圆厂”业务的财务结果，引起了广泛关注。英特尔晶圆厂是英特尔的一部分，既为其自家产品制造芯片，也希望越来越多地为外部客户提供芯片。

剧透警告，情况并不好。一点都不好。

这导致英特尔晶圆在2023年亏损了70亿美元。

就在这一切似乎够糟糕的时候，英特尔如今承认，在约十年前就失去了其10纳米制造工艺的领导地位。

英特尔期望（希望）通过其Intel 18A工艺再次超越台积电，但仅在落后台湾公司近十年后才能做到。

英特尔在经历了多年的领导地位后，是如何陷入这种境地的？公司可能归咎于“结构上更具竞争力的环境”，但这并非全貌。

正如英特尔所承认的那样，其旧的制造模式依赖于专有的知识产权和流程，专注于仅提供内部产品，并通过销售这些产品来获得制造的货币化。

正如

[强调了](https://www.semianalysis.com/p/rebuilding-intel-foundry-vs-idm-decades)，这种模式允许英特尔在其制造和设计流程中积累和隐藏大量的低效率。

这些低效现象现在已经暴露出来，英特尔重新发现，摩尔定律的核心实际上是关于经济（Intel的头条如下）。

我在二月份的《摩尔对摩尔》（Moore on Moore）中写道，关于经济对摩尔定律的重要性。引用英特尔创始人戈登·摩尔：

> 摩尔定律实际上是关于经济的。我的预测是关于半导体行业未来发展方向的，我发现这个行业最好通过其潜在的经济原理来理解。
> 
> *戈登·摩尔*

现在，英特尔面临着艰难的挑战。它需要重拾其工艺实力，在新的晶圆厂投入数百亿美元，并为英特尔Foundry建立客户基础。

如需了解更多有关英特尔的计划和挑战，请参阅本文末尾的进一步阅读内容。

不过，本文的其余部分将深入探讨英特尔陷入困境的历史和起源。我认为，它的起源可以追溯得更早，源于2007年发生（或者说，从英特尔的角度看，没有发生）的两件事情。

[分享](https://thechipletter.substack.com/p/the-paradox-of-x86?utm_source=substack&utm_medium=email&utm_content=share&action=share)

这些问题都源于同一个根本原因，这也恰恰是英特尔最大的优势之一，是保护英特尔最重要业务超过四十年的护城河：x86指令集体系结构（ISA）。

ISA是处理器可以执行的机器指令集合。

当我们说‘x86指令集体系结构（ISA）’时，实际上指的是从1976年起（因此得名x86）以英特尔8086为首的一系列处理器的ISA集合，直至今天的英特尔和AMD最新的处理器设计。正如我们在“万亿美元的权宜之计”中看到的，8086及其后继者本来只打算维持几年。

8086甚至不是一个全新的设计。正如英特尔的发布宣传所述，8086是早期8位8080微处理器的演变，后者又是英特尔首款8位微处理器8008的演变，后者实现了1970年DataPoint 2200终端的体系结构。

x86的起源可以追溯到一个非常古老的设计，甚至不是由英特尔开发的！

近50年来，从16位8086发布以来，x86 ISA已经有了巨大的进化。今天的x86代码是64位的，并且增加了数百条新指令，增强了其功能。这些设计一直保持着高度的‘向后兼容性’。用户可以在新设计上运行他们现有的软件，即使x86 ISA已经演变并得到增强。

在过去的五十年里，x86 已经取得了巨大的成功。数十亿个处理器使用 x86 ISA 发货，为 Intel 创造了超过一万亿美元的收入。部分原因是因为消费者和企业需要购买兼容 x86 的个人电脑才能运行首先是 MS-DOS 然后是 Windows 软件。最近，由于 Intel 的制造实力得到了大量个人电脑销售的支持，x86 设计在服务器上变得主导，使该公司能够创建比竞争对手更强大的设计。

x86 ISA 在过去四十年间充当了英特尔业务的“护城河”，因此从零开始创建竞争性的 x86 设计变得困难，随着时间的推移变得更加困难。

这并不总是不可能的，AMD、NEC、Via 和其他公司过去曾这样做过。多年来，它们大多数已经退出，无法与 Intel 竞争。然而，随着 x86 变得更加复杂，AMD 现在是唯一一家留下来直接与 Intel 竞争的公司。

> 阅读更多关于 NEC 试图通过其自己版本的 x86 与 Intel 竞争的内容在‘Intel vs NEC：V20 微码的案例’中。

* * *

“x86 护城河”听起来像是一个巨大的资产，那么为什么会导致 Intel 的近期问题？

让我们回到 Intel Foundry。说 Intel Foundry 的成功对 Intel 的未来至关重要简直是轻描淡写。简单来说，Intel 正在建设大量昂贵的半导体制造厂。它将用它们来制造自己的 x86 产品。但要填满这些厂房，并最终使它们在财务上可行，它需要大量的第三方客户，几乎没有一个客户会在他们的设计中使用 x86。

Intel 在二月份举行了一场关于 Foundry 的盛大演讲。查看本文末尾的链接以获取完整事件录像。

演讲中有一个时刻格外引人注目。Arm 的 CEO Rene Haas 的出现。

如读者所知，Arm 不生产芯片。相反，它将其知识产权授权给其他公司，以便将其构建到他们自己的 SoC（系统芯片）中。该知识产权最初是使用 Arm 自己的 ISA（S）的处理器设计，但现在扩展到向第三方授权 ISA 自身。授权 Arm ISA 并在其自己的 CPU 设计中使用该 ISA 的主要公司包括 Apple、Broadcom、Qualcomm 和 Nvidia。

> 读者可以在以下帖子中了解 Arm 的故事起源于英国剑桥的 Acorn 计算机，经过“Arm Barn”并进入 Nokia 3G 手机。

今天，使用 Arm IP-based 产品的 Arm 客户完全主导了智能手机 SoC 市场。它们与 Intel 在服务器上和越来越多地在桌面上竞争。今天的 Mac 中使用的 Apple Silicon 使用 64 位 Arm ISA，因为 Apple 已经远离了 x86。

> 更多关于苹果跨越 x86 护城河并完成向基于 Arm 的 Apple Silicon 的转变的内容：

美国三大“超级扩展者”之一的亚马逊，现在加入了微软和最近加入的谷歌，通过推出 [Axion](https://cloud.google.com/blog/products/compute/introducing-googles-new-arm-based-cpu)，现在都在建造他们自己的基于 Arm 的服务器 CPU。

> 更多关于 Arm 如何进入亚马逊网络服务的信息：

* * *

Arm 是一家相对较小的公司，2023 年收入为 280 亿美元，而 Intel 的收入为 540 亿美元，但使用 Arm IP 的客户的收入超过了 1.5 万亿美元（是的，这是超过一万五千亿美元的收入）。

**在这里邀请 Haas 登台可以比作放下吊桥让侵略军的主谋进来喝一杯礼貌的茶。**

但现在 Intel 需要 Arm。如果 Intel 制造的 SoC 中没有 Arm 的 CPU 设计，那么它的铸造业务和新建造的半导体工厂将会面临困难。欢迎 Haas 登上 Intel 的舞台，延续了两家公司去年 [宣布的](https://www.theregister.com/2023/04/12/arm_and_intel_fab_deal/) 合作伙伴关系。

这带我们回到 2007 年 Intel 所未能实现的两件事情中的第一件。首先让我们为 Intel 在 2007 年的业务提供一些背景。

[分享](https://thechipletter.substack.com/p/the-paradox-of-x86?utm_source=substack&utm_medium=email&utm_content=share&action=share)

2006 年对 Intel 来说是一个胜利的一年。在 Itanium 和 Netburst 的失误后，它通过低功耗 x86 [Core](https://en.wikipedia.org/wiki/Intel_Core_(microarchitecture)) 架构的回归，在桌面上击败了一度复苏的 AMD，并继续向主导服务器市场迈进。

当 Apple 从 PowerPC 切换到 Intel 的 CPU 时，Intel 获得了一个备受关注的新客户。

Apple 的史蒂夫·乔布斯宣布 Mac 从 PowerPC 到 Intel 的过渡

但 Apple 在 2007 年成为 Intel 的第一个重大失误的场所。

Intel 放弃了制造 Apple 新 iPhone 的 SoC，该手机于 2007 年初推出。而是 Apple 使用了三星制造的 SoC 中的 Arm 设计处理器。

Intel 意识到错误已经太迟。超过一千亿美元花在了试图在智能手机 SoC 市场上站稳脚跟的终极徒劳的尝试上。Intel 最终在 2016 年 [放弃了](https://www.theverge.com/2016/5/3/11576216/intel-atom-smartphone-quit)，将市场让给了三星和台积电生产的 Arm 处理器。

今天，Apple 是 TSMC 最大的客户，智能手机占据台湾公司超过 40% 的收入。

* * *

Intel 当时的 CEO，已故的保罗·奥泰利尼（Paul Otellini），在他从 Intel 退休前不久，在 [大西洋](https://www.theatlantic.com/technology/archive/2013/05/paul-otellinis-intel-can-the-company-that-built-the-future-survive-it/275825/) 的采访中，对 Intel 错过 iPhone 表达了直言不讳的歉意：

> "你必须记住的一件事是，在iPhone推出之前，没有人知道iPhone会做什么... 最终，他们对一款芯片感兴趣，他们愿意支付特定的价格，而不多支付一分钱，而这个价格低于我们预测的成本。我看不到这点。这不是你可以通过增加产量来弥补的事情之一。事后看来，预测的成本是错误的，而产量是任何人预想的100倍。"

因此，iPhone的‘错失’只是一个简单的预测错误？而这个决定纯粹是由这些预测所指示的财务结果驱动的？事实上，如果是这种情况，我认为这将是非常值得注意的，我们将在稍后讨论这些原因。

无论如何，这是一个巨大的失去机会。2007年，英特尔显然是工艺领导者。

英特尔在2007年发货了其首款45纳米芯片。而iPhone 3GS则在2009年发布，使用三星制造的SoC，采用90纳米工艺。

失去iPhone和智能手机将数千亿美元的业务赠予了三星和台积电，这些业务对它们追赶并最终超越英特尔至关重要。

* * *

2007年第二次错失机会，是因为没有直接回应CUDA，Nvidia的通用计算GPU软件生态系统。

英特尔在独立GPU方面历史悠久，但并不密集。

集成英特尔i740 GPU的显卡 - 作者：Konstantin Lanzet - 自有收藏，GFDL，https://commons.wikimedia.org/w/index.php?curid=8400155

*本文的其余部分供高级订阅者阅读。请点击下面的按钮进行升级。*
