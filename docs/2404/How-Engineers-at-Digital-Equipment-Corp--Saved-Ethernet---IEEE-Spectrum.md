<!--yml

category: 未分类

date: 2024-05-27 13:01:06

-->

# 数字设备公司工程师如何拯救以太网 - IEEE Spectrum

> 来源：[https://spectrum.ieee.org/how-dec-engineers-saved-ethernet](https://spectrum.ieee.org/how-dec-engineers-saved-ethernet)

我享受阅读有关以太网50周年纪念的杂志文章，包括[*The Institute*](https://spectrum.ieee.org/ethernet-ieee-milestone)的一篇。以太网由计算机科学家[罗伯特·梅特卡夫](https://ethw.org/Robert_M._Metcalfe)和[大卫·博格斯](https://ethw.org/David_Boggs)发明，具有极大的影响力。作为IEEE会士，梅特卡夫因其工作而于1996年获得[IEEE荣誉奖](https://corporate-awards.ieee.org/recipients/ieee-medal-of-honor-recipients/)，并于2022年获得[图灵奖](https://amturing.acm.org/)，颁发者为[计算机协会](https://www.acm.org/)。但这个故事还有更多不为人知的内容。

在1980年代和1990年代初，我在[Digital Equipment Corp.](https://en.wikipedia.org/wiki/Digital_Equipment_Corporation)（数字设备公司）位于马萨诸塞州的网络高级开发小组中担任领导。我是局域网技术发展和标准化竞争激烈时期的第一手见证者。

DEC, [Intel](https://www.google.com/aclk?sa=l&ai=DChcSEwi2hOvzgu-EAxWZH60GHXwkAToYABAAGgJwdg&ase=2&gclid=Cj0KCQjw-r-vBhC-ARIsAGgUO2DU3b9FN2WnvoVFOiwwrcDDz5q7PPS2MiSG6DRs-u-xJc9j5_yBWkcaAn2LEALw_wcB&ei=dHLwZY3bOdDB0PEP7_WwiAU&sig=AOD64_3OM2YVYFwaHUS5T6_5SPLa3qL80Q&q&sqi=2&nis=4&adurl&ved=2ahUKEwiN5ePzgu-EAxXQIDQIHe86DFEQ0Qx6BAgIEAE), and [Xerox](https://www.xerox.com/en-us/about)努力获益于以太网在1970年代的推出。但在1980年代，其他局域网技术作为竞争对手出现。主要竞争者包括由[IBM](https://www.ibm.com/about)推广的[令牌环](https://www.geeksforgeeks.org/efficiency-of-token-ring/)和[令牌总线](https://www.sciencedirect.com/topics/computer-science/token-bus)。 （今天，以太网和两种基于令牌的技术都是[IEEE 802](https://www.ieee802.org/)标准族的一部分。）

所有这些局域网都有一些共同的基本部分。其中之一是48位媒体访问控制（MAC）地址，这是在计算机网络端口制造过程中分配的唯一编号。MAC地址仅在局域网内部使用，但对其操作至关重要。通常，除了网络上的通用计算机外，它们至少有一台特殊用途的计算机：路由器，其主要工作是代表局域网上的所有其他计算机发送数据至互联网，并从互联网接收数据。

在一个几十年前的网络概念模型中，局域网本身（电缆和低层硬件）被称为第二层，或数据链路层。路由器主要处理另一种地址：在LAN内外都使用的网络地址。许多读者可能听说过*互联网协议*和*IP地址*这些术语。除了一些例外，数据包中的IP地址（网络地址）足以确保该数据包通过由服务提供商和承运商操作的一系列其他路由器在互联网上任何地方传递。路由器及其执行的操作被称为第三层，或网络层。

在令牌环局域网中，屏蔽双绞线铜线将每台计算机连接到其上游和下游邻居，形成一个无限环结构。每台计算机将其上游邻居的数据转发给其下游邻居，但只有在从上游邻居接收到短数据包（令牌）后才能将自己的数据发送到网络中。如果没有数据需要传输，它只是将令牌传递给其下游邻居，如此循环。

在令牌总线局域网中，一根同轴电缆连接所有网络计算机，但是布线并不控制计算机传递令牌的顺序。计算机们就传递令牌的顺序达成一致，形成一个无限的虚拟环，数据和令牌在其中循环传播。

与此同时，以太网已经成为使用一种称为*带冲突检测的载波侦听多路访问*方法管理传输的同轴电缆连接的代名词。在CSMA/CD方法中，想要传输数据包的计算机首先监听是否有其他计算机在传输。如果没有，则计算机发送其数据包，同时监听以确定其数据包是否与其他计算机的数据包发生碰撞。由于计算机之间的信号传播不是即时的，碰撞可能会发生。在碰撞的情况下，发送计算机会延迟重新发送其数据包，延迟包含随机部分和根据碰撞次数指数增长的部分。

需要检测碰撞涉及数据速率、物理长度和最小数据包大小之间的权衡。将数据速率增加一个数量级意味着要么减少物理长度，要么将最小数据包大小大致增加相同的因子。以太网的设计者明智地选择在这些权衡中找到了一个甜点：每秒10兆比特和长度为1500米。

## 来自光纤的威胁

与此同时，一些公司联合（包括我的雇主DEC）正在开发一种新的ANSI局域网标准：[光纤分布数据接口（FDDI）](https://www.techtarget.com/searchnetworking/definition/FDDI)。FDDI方法使用令牌总线协议的变体通过光纤传输数据，承诺的速度为100 Mb/s，远远快于以太网的10 Mb/s。

一系列技术出版物发布了对不同工作负载下竞争LAN技术的吞吐量和延迟的分析。鉴于结果和预期从更快的处理器、RAM和非易失性存储中得到更大网络性能需求，以太网的有限性能成为一个严重问题。

对于创建更高速度LAN的最佳选择，FDDI似乎比以太网更有希望，尽管FDDI使用了昂贵的组件和复杂的技术，特别是对于故障恢复。但是，所有共享介质访问协议都存在一个或多个不可取的特性或性能限制，这要归因于在共享电缆或光纤中实现共享时涉及的复杂性。

## 解决方案出现

我觉得，与FDDI或以太网的更快版本相比，开发一种执行存储转发交换的LAN技术会更好。

1983年的一个晚上，就在下班回家前，我拜访了我的团队成员、首席工程师之一[马克·肯普夫](https://www.linkedin.com/in/mark-kempf-4a2668b7)的办公室。马克是我曾经合作过的最优秀的工程师之一，他设计了流行且盈利丰厚的DECServer 100终端服务器，该服务器使用了DEC公司企业架构组的布鲁斯·曼创建的局域传输（LAT）协议。终端服务器将只有RS-232串行端口的一组哑终端连接到具有以太网端口的计算机系统。

我告诉马克我的想法，即使用存储转发交换来提高LAN性能。

第二天早晨，他带着一个关于学习桥的想法（也称为第2层交换机或简称交换机）来到我的办公室。这个桥接设备将连接两个以太网LAN。通过监听每个LAN上的所有流量，该设备将学习到两个以太网上计算机的MAC地址（记住哪台计算机连接在哪个以太网上），然后根据目标MAC地址有选择地转发适当的数据包。对于两个网络上的计算机来说，不需要知道它们的数据会在扩展LAN上走哪条路径；对它们来说，这个桥接设备是透明的。

这个桥接设备需要每秒接收和处理约30,000个数据包（每个以太网15,000个数据包），并决定是否转发每一个数据包。虽然30,000个数据包每秒的需求接近当时最好的微处理器技术——Motorola 68000的极限，但马克确信他可以使用仅包括一些现成的组件来建造一个双以太网桥接设备，包括使用可编程阵列逻辑（PAL）设备设计的专用硬件引擎和专用静态RAM来查找48位MAC地址。

马克的贡献并未得到广泛认可。一个例外是乔治·瓦格里斯所著的教材[*网络算法学*](https://www.amazon.com/Network-Algorithmics-Interdisciplinary-Designing-Networking/dp/0120884771)。

在一个错误配置的网络中——即存在桥梁在以太网中形成环路的网络中——数据包可能会永久循环。我们有信心能够找到一种方法来防止这种情况发生。在紧急情况下，产品可以在没有安全功能的情况下交付。显然，双端口设备只是一个起点。多端口设备可以随后推出，尽管它们将需要定制组件。

我将我们的想法呈现给了三级管理层，希望能够获得建造 Mark 设想的学习桥的批准。在当天结束之前，我们得到了绿灯，并且理解如果原型成功，将会推出产品。

## 桥梁的开发

我在 DEC 的直属经理 Tony Lauck 的挑战下，要求几位工程师和架构师解决错误配置网络中数据包环路的问题。几天内，我们提出了几种潜在的解决方案。在 Tony 管理的团队中，架构师 [Radia Perlman](https://en.wikipedia.org/wiki/Radia_Perlman) 提出了一种明确的胜者：生成树协议。

Perlman 的方法中，各个桥梁会相互检测，并根据指定的标准选择一个根桥梁，然后计算最小生成树。在这种情况下，最小生成树是一个数学结构，描述了如何在不形成环路的情况下有效连接局域网和桥梁。最小生成树随后用于将任何可能造成环路的桥梁置于备份模式。作为一个附带的好处，它在桥梁故障的情况下提供了自动恢复。

一个被 Digital Equipment Corp. 在 1986 年发布的 LANBridge 100 的逻辑模块，由 Alan Kirby 发布。

Mark 设计了硬件和对时敏感的低级代码，而软件工程师 Bob Shelly 则编写了剩余的程序。1986 年，DEC 将这项技术作为 LANBridge 100 推出，产品代码为 DEBET-AA。

不久之后，DEC 开发了 DEBET-RC 版本，支持桥梁之间 3 公里的光纤跨越。一些 DEBET-RC 的手册可以在 [Bitsavers 网站](https://bitsavers.org/) 找到。

Mark 的想法并没有取代以太网——这正是它的精彩之处。通过允许现有的 CSMA/CD 同轴基础以太网之间的存储-转发交换，桥梁使得现有局域网的升级变得简单。由于任何碰撞不会传播到桥梁之外，连接两个以太网与桥梁将立即将单个以太网电缆的长度限制加倍。更重要的是，将彼此通信频繁的计算机放置在同一以太网电缆上，将使该流量隔离到该电缆，同时桥梁仍然允许与其他以太网电缆上的计算机进行通信。

这降低了两根电缆上的流量，提高了容量，同时减少了碰撞的频率。推而广之，这最终意味着每台计算机都有自己的以太网电缆，多端口桥梁将它们全部连接起来。

这导致了逐渐从CSMA/CD在同轴电缆上的迁移，到如今普遍存在的个体计算机之间和专用交换机端口之间的铜缆和光纤链接。

链路的速度不再受碰撞检测的限制。随着时间的推移，这种变化彻底改变了人们对以太网的看法。

如果相关的数据包头部足够相似，一个桥甚至可以拥有不同LAN类型的端口。

我们的团队随后开发了GIGAswitch，这是一种支持以太网和FDDI的多端口设备。

存在性能越来越高的桥梁使得开发新的共享介质局域网访问协议的人失去了动力。 FDDI后来在更快的以太网版本面前逐渐退出市场。

当然，桥梁技术也并非没有争议。一些工程师仍然认为第二层交换是个坏主意，而你只需更快的第三层路由器就能在局域网之间传输数据包。然而，在当时，IP并未在网络层获胜，DECNet、IBM的SNA和其他网络协议争相主导。第二层的交换可以适用于任何网络协议。

马克在1986年为该设备获得了[美国专利](https://patents.google.com/patent/US4597078A/en)。 DEC提议无偿许可任何公司使用这项技术。

这导致了IEEE的标准化工作。已经建立的网络公司和初创公司开始采用并努力改进交换技术。其他增强措施包括交换机专用的ASIC、虚拟局域网和更快更便宜的物理介质及相关电子设备的发展，这些稳步促进了以太网的长寿和流行。

以太网的持久价值不在于CSMA/CD或其原始的同轴介质，而在于它为协议设计者提供的易于理解和功能齐全的服务。

如今许多家庭网络中的交换机直接继承了这一创新。现代数据中心有许多交换机，个体端口的速度在40到800吉比特每秒之间。仅数据中心交换机市场每年的收入就超过100亿美元。

我的DEC经理劳克曾说，一个架构的价值可以通过其在技术世代中的使用次数来衡量。按照这个标准，以太网已经极其成功。第二层交换技术也是如此。

如果没有马克发明学习桥，以太网会发生什么变化，没有人知道。也许其他人会想出这个主意。但以太网也可能会慢慢消亡。

对我来说，马克挽救了以太网。
