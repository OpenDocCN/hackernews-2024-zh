<!--yml

分类：未分类

日期：2024 年 05 月 27 日 15:11:23

-->

# 云是新的主机吗？IBM 的主机是最初的现场…|作者：Billy Newport |Medium

> 来源：[`medium.com/@billynewport/is-cloud-the-new-mainframe-e43133cac151`](https://medium.com/@billynewport/is-cloud-the-new-mainframe-e43133cac151)

# 云是新的主机吗？

IBM 的主机是最初的现场私有云。一排排集成的计算和存储，运行着一个定制的软件堆栈，安装在现场。客户喜欢它们，因为它们提供了一种集成解决方案，提供按需弹性扩展。它们被过度配置了，只需一个电话给 IBM 就可以解锁更多容量。IBM 维护硬件和广泛的热插拔功能，并且备件意味着非常高的正常运行时间和在故障发生时轻松的硬件交换。IBM 按服务使用和存储使用收费。按需付费。成本低廉，并且随着使用量的增加而增加。

如果你达到了已安装容量的限制，将现场硬件升级以获得更多容量是相当便宜和简单的。多个数据中心，复制都得到支持。

今天的云供应商只能梦想的疯狂高可用性数字。对于许多常见的故障，包括 cpu 故障的高级硬件支持，具有热插拔功能…

在其上运行的软件堆栈被很好地集成了起来。应用程序、存储、数据库等都使用管理器运行，以跟踪相对优先级和预算。

IBM 过去、现在和将来都将从其主机客户那里赚取大量利润。任何租用主机容量来运行应用程序的人基本上每月都要付费。一旦客户在主机上构建了一个应用程序或应用程序套件，那么将其迁移到另一个平台就会变得困难或昂贵，因此它是粘性的，每月的付款继续流向 IBM。如果需要新功能，最容易的地方可能就是在主机上进行。

这是一个很好的商业模式。在这个平台上构建应用程序和生态系统降低了开发成本，并且随着业务的增长，给企业提供了弹性定价。

它的问题在于，任何使用频繁的应用程序都按照服务价格支付与其服务月度使用量成比例的费用。随着应用程序使用量的增长，账单也会增长，而账单的控制在很大程度上掌握在 IBM 手中。你使用得越多，付出的费用就越多。如果一家公司在主机上运行了 100 个应用程序，那么主机的大部分月度成本将不可避免地是其中的几个应用程序。应用程序几乎总是从小开始。重点是功能而不是优化。所以，它可以快速上线，并且成本不高，因为使用量不大。但是，一小部分应用程序变得受欢迎且易于扩展。不幸的是，虽然计算是弹性的，但预算并不是…

因此，使用频繁的应用程序成为了昂贵的运营项目。即使停止开发，你仍然要为在大型机上运行它而付出昂贵的代价。通常，开发人员被指定优化昂贵的应用程序，因为它们增长以尝试优化它们以降低成本。对于大多数成功的应用程序来说，有两个阶段，构建并稳定它，然后第二阶段是管理成本，随着它的增长，你希望成本增长速度比你的使用速度慢得多。不可避免的是，客户会尝试将工作负载从大型机迁移到“更便宜”的平台，但这些项目往往成本高昂，并且失败的次数比人们意识到的要多。复制大型机提供的非功能特性也比大多数客户意识到的要昂贵得多，这从来不是一个简单的重新编译和运行的情景。甚至使用当前流行的语言和工具复制应用程序也是非常困难的。

我认为在小型计算机（AS/400 或 iSeries）和大型机上运行 ISV 应用程序的地方做得很好。今天我们称托管的 ISV 应用程序为 SaaS 平台 :) 我认为这些通常是成本效益高的。客户可以在最佳类别的基础设施上运行出色的功能堆栈，以实现一些有价值的业务目的。

# 今天的云看起来和大型机的情况几乎一样。

如果你退后一步看看。前面的部分也描述了当今的云，并预测了同样的问题。公司们急于跟风加入云。我预测许多公司将试图急于减少云支出，并发现迁移到内部可能是一项昂贵的提议，即使可能也不是一个可能的选择。

我认为云主要用于运行 SaaS，而不是近乎于金属的服务。想想 CRM/工资单/开发托管/MS Office/企业电子邮件等等，而不是数据存储/kubernetes/虚拟机托管。对于公司来说，这比仅仅托管数据或虚拟机更有价值。

如果你的公司仍然没有进行大规模的云迁移，我认为在短期/中期内你将会看起来非常聪明，而不是慢吞吞的……
