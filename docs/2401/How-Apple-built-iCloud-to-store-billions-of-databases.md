<!--yml

类别：未分类

日期：2024年05月27日14:51:55

-->

# 苹果如何构建iCloud来存储数十亿个数据库

> 来源：[https://read.engineerscodex.com/p/how-apple-built-icloud-to-store-billions](https://read.engineerscodex.com/p/how-apple-built-icloud-to-store-billions)

*Engineer’s Codex是一个概括实际软件工程的出版物。*

* * *

在过去的几个月中，我已经写了一些关于大型科技公司“幕后”的技术，例如[Meta的内部无服务器平台](https://engineercodex.substack.com/p/meta-xfaas-serverless-functions-explained)和[Google内部喜爱的代码审查工具](https://engineercodex.substack.com/p/how-google-takes-the-pain-out-of)。

另一方面，苹果的基础设施并不像公开。我想找出苹果是如何构建iCloud的，在本文中，我涵盖了我所知道的一切。

**苹果使用[FoundationDB](https://www.foundationdb.org/)和Cassandra来构建iCloud和CloudKit，他们的云后端服务。** 是的，标题并没有错误：苹果确实在他们的极端多租户架构中存储**数十亿个数据库**。

阅读之前，这些是**适用的教训和指南。**

[我发现论文和苹果的很多课程与[Meta的无服务器平台架构](https://read.engineerscodex.com/p/meta-xfaas-serverless-functions-explained)的课程非常接近。](https://read.engineerscodex.com/p/meta-xfaas-serverless-functions-explained)

+   两者都巧妙地使用**异步处理**，以使用户功能更加流畅。Meta使用他们的无服务器堆栈进行非用户界面功能。苹果几乎*所有*的Record Layer功能（在下面进行了详细解释）都使用异步处理，以*隐藏用户的延迟*。

+   两者都大量使用**无状态架构**，因为它们具有强大的可扩展性需求。

+   两者都为了可靠性和可用性而**逻辑隔离资源**。

+   两者都**简单处理多样化的需求**。苹果提到了如何“诱人

    为了提供‘小数据’和‘大数据’而预留和操作单独的系统。”然而，这会增加操作复杂性，而他们通过一个抽象处理所有类型的数据需求。Meta在其无服务器平台上使用相同的方式，为所有类型的功能负载提供一个抽象。

+   两者都**构建抽象层**，以改善开发人员的体验。应用程序开发人员不必担心可扩展性需求 - 这由分布式系统工程师在更深的堆栈中处理。

+   **了解你的用户。** Meta和苹果提供的每个层，API和设计决策都受到对特定技术用户的清晰了解的指导，无论是应用开发团队还是可观察性团队。

* * *

***[SWE Quiz](http://swequiz.com/)** 是一个包含 450+ 个问题的资源，可揭示您在软件领域知识中的差距，比如[数据库](https://swequiz.com/learn/databases-roadmap)、[缓存](https://swequiz.com/learn/caching-roadmap)和[网络](https://swequiz.com/learn/networking-roadmap)。其中许多问题是经过验证的概念，曾在 Meta、Apple、Airbnb、Google 等公司的面试中提出。*

*SWE Quiz 还提供[结构化路线图](https://swequiz.com/learn)，用于学习重要的软件概念。*

[获得 SWE Quiz 终身访问权限](https://swequiz.com)

* * *

[Cassandra](https://cassandra.apache.org/_/index.html) 是一种宽列 NoSQL 数据库管理系统。它最初是在 Facebook 开发，用于支持 Facebook 的收件箱搜索功能。有趣的是，Meta 本身已经用 [ZippyDB](https://engineering.fb.com/2021/08/06/core-infra/zippydb/) 替换了大部分的 Cassandra 使用。

iCloud 部分由 Cassandra 提供支持。根据 DataStax 的说法，苹果运行着世界上最大的 Cassandra 部署之一](https://news.ycombinator.com/item?id=9307563)。

[他们报告](https://twitter.com/erickramirezau/status/1578063811495477248)：

+   超过 300k 个实例/节点

+   数以百万计的 PB 数据（如果不是 EB）

+   每个集群超过 2 PB，拥有数千个集群

+   每秒数百万次查询

+   数千个应用

iCloud 中的其他 Cassandra 片段显示，管理着**数量巨大的数据**。[有](https://news.ycombinator.com/item?id=33124631#33136026) **[每个服务器上都有多个 Cassandra 节点](https://news.ycombinator.com/item?id=33124631#33136026)**，而且苹果的团队在分片和分区方面非常聪明](https://news.ycombinator.com/item?id=33124631#33136026)，这确保了 iCloud 数据的可用性接近 100%。

在 Apple，Cassandra 仍在积极改进。苹果的 Scott Andreas 上个月就[Cassandra 的未来](https://www.youtube.com/watch?v=hUxLJSFi2-A)做了一次演讲。在苹果的职位页面上，他们在招聘分布式系统工程师时经常提到 Cassandra。

**然而，[CloudKit](https://developer.apple.com/icloud/cloudkit/) + Cassandra 遇到了两个可扩展性限制，这导致他们采用了 FoundationDB。**

1.  在单个区域内，**每次只能进行一项操作**，即使正在编辑不同的记录。对于需要多个用户或设备同时处理共享数据的应用程序，这可能会有问题。

1.  在原子操作中同时更新多个记录时，更新将限制在单个 Cassandra 分区内。**这些分区有它们能处理的最大大小，随着分区大小的增加，Cassandra 的速度会变慢。**

**FoundationDB 和 Record Layer 解决了这两个问题。**

苹果对 FoundationDB 更加公开。他们[在2015年收购了 FoundationDB](https://techcrunch.com/2015/03/24/apple-acquires-durable-database-company-foundationdb/)，此后发布了各种详细介绍他们使用 FoundationDB 的论文。

[FoundationDB](https://github.com/apple/foundationdb)是一个开源的、分布式的、事务性的键值存储。它被设计用于处理大量数据，对于读/写工作负载和写入重负载都很有效。它也是[ACID 兼容](https://www.swequiz.com/learn/acid-properties)的。

苹果广泛使用**[FoundationDB Record Layer](https://github.com/FoundationDB/fdb-record-layer)**来支持 CloudKit（苹果的云后端服务）。

从他们的[GitHub](https://github.com/FoundationDB/fdb-record-layer)页面：

> 记录层是一个 Java API，在 FoundationDB 之上提供了一个基于记录的存储，（非常）粗略地相当于一个简单的关系数据库，具有以下特点：
> 
> +   **结构化类型** - 记录是根据[protobuf](https://developers.google.com/protocol-buffers/)（Protocol Buffer）消息定义和存储的。Protocol Buffers 最初由 Google 设计。
> +   
> +   **索引** - 记录层支持各种不同类型的索引，包括值索引（大多数数据库提供的类型）、排名索引和聚合索引。索引和主键可以通过 protobuf 选项或以编程方式定义。
> +   
> +   **复杂类型** - 支持复杂类型，例如列表和嵌套记录，包括定义针对这些嵌套结构的索引的能力。
> +   
> +   **查询** - 记录层并不提供查询语言，但提供了具有扫描、过滤和排序能力的查询 API，并且具有自动选择索引的查询计划器。
> +   
> +   **许多记录存储，共享模式** - 记录层提供支持许多离散记录存储实例的能力，所有这些实例都具有共享的（和不断演变的）模式。例如，与其对所有用户数据建模以存储在单个数据库中，可以为每个用户提供他们自己的记录存储，可能分片存储在不同的 FDB 集群实例中。
> +   
> +   **非常轻量级** - 记录层设计用于在大型、分布式、无状态环境中使用。打开存储并进行第一次查询之间的时间应该以毫秒计。
> +   
> +   **可扩展** - 新的索引类型和自定义索引键表达式可以动态地并入记录存储。

根据[FoundationDB Record Layer](https://www.foundationdb.org/files/record-layer-paper.pdf)的论文，他们写道：

> “[FoundationDB Record Layer被用来]为服务于**数亿用户**的应用程序提供强大的抽象。CloudKit 使用 Record Layer 来托管**数十亿个独立的数据库**，其中许多具有共同的模式。”

FoundationDB、Record Layer 和 CloudKit 的结构如下：

+   **FoundationDB**完成了所有的分布式系统和并发控制工作。

+   **记录层**作为一个关系型数据库，使得FoundationDB更易于使用。

+   **CloudKit**位于最顶层，为应用程序开发人员提供功能和API。CloudKit不是建立在记录层之上的唯一内容，还有其他层次，用于构建需要结构化存储的内容，比如JSON文档存储。

**记录层使得苹果能够支持大规模的多租户。**

实际上，这有点低估了它的价值。

记录层用于**极端的多租户，其中每个应用程序的每个用户都拥有独立的记录存储。这意味着记录层托管着数十亿个独立的数据库，这些数据库共享数千个模式。**

真是太棒了！而且令人印象深刻得多。

记录层通过两个基本的架构决策，实现了对大规模多租户的处理。

1.  该层**以无状态方式运行**，可以通过简单地添加更多这种无状态实例来轻松扩展计算资源。

    1.  这种无状态的架构简化了负载均衡器和路由器的任务，因为它们只需要关注数据的位置，而不是计算服务器的能力。此外，无状态服务器只需要在客户端之间分配更少的资源。

1.  该层使用**记录存储抽象**来有效地管理资源分配和可伸缩性。这种抽象表示了一个逻辑数据库的全部内容，包括序列化数据、索引和操作状态。

    1.  每个记录存储都被分配了一个特定的键范围，这保证了为不同租户的数据进行逻辑分离。如果需要，将租户的数据转移变得非常简单，只需要将分配的键范围重新定位到新的集群即可，因为管理和使用记录存储所需的所有信息都包含在此范围内。

**在CloudKit中，一个应用程序由一个'逻辑容器'表示，该容器遵循一个定义好的模式。**这个模式概述了启用有效数据检索和查询所需的记录类型、字段和索引。**应用程序在CloudKit中将其数据组织成'区域'**，这允许对记录进行逻辑分组，以便与客户端设备进行选择性同步。

对于每个用户，CloudKit为FoundationDB中的每个应用程序指定一个独特的**子空间**。在这个子空间内，它为用户与之交互的每个应用程序创建一个**记录存储**。实质上，CloudKit管理着大量的逻辑数据库——将用户数量乘以应用程序数量——每个数据库都包含其自己的记录、索引和元数据，总计**数十亿个数据库。**

当CloudKit从客户端设备收到请求时，它会通过**负载均衡**将此请求定向到可用的**CloudKit服务进程**。然后，该进程与特定的记录层记录存储进行交互，以满足请求。

CloudKit 将定义的应用程序模式转换为 Record Layer 中的 **元数据定义**，该元数据存储在单独的元数据存储中。这些元数据由 CloudKit 特定的系统字段增强，用于跟踪 **记录的创建、修改时间和存储记录的区域**。区域名称被添加到主键前缀，以便在每个区域内有效地访问记录。除了用户定义的索引外，CloudKit 还管理用于内部目的的 **'系统索引'**，例如通过跟踪记录大小的索引来管理存储配额。

**FoundationDB 和 Record Layer 共同解决了 4 个苹果公司的关键问题，而单独使用 Cassandra 或 FoundationDB 都无法解决。**

FoundationDB 帮助解决了用户快速访问其数据的个性化全文搜索问题。

他们的系统利用 **FoundationDB 的键顺序**，允许快速搜索文本开头（**前缀匹配**）以及更复杂的搜索（例如找到彼此接近或按特定顺序排列的单词 - **接近和短语搜索**）而不需要额外的开销。

在传统的搜索系统中，通常需要额外的后台进程来保持搜索索引的更新，但是苹果公司的系统在实时进行，这意味着数据发生变化时，搜索索引立即更新 - 无需额外步骤。

使用 FoundationDB，CloudKit 能够平稳地处理许多同时发生的更新。

以前，使用 Cassandra 时，CloudKit 依赖于一个特殊的索引来跟踪每个区域的更改，以同步设备之间的数据。当设备需要更新其数据时，它会检查此索引以查看新内容。但是这个系统有一个缺点：当多个更新同时发生时，可能会导致冲突。

但是使用 FoundationDB 时，CloudKit 使用一种特殊类型的索引来跟踪每个更改的确切顺序，而不会引起冲突。这是通过为每个更改分配唯一的“版本”来完成的，当 CloudKit 需要同步时，它会查看这些版本以确定设备错过了哪些更新。

但是，当 CloudKit 需要在不同的存储集群之间移动数据 - 也许是为了更均匀地分配负载 - 事情变得棘手，因为每个集群都有自己的版本号，这些版本号不匹配。为了解决这个问题，CloudKit 为每个用户的数据分配一个“移动计数”（称为“化身”），每次将其数据传输到新的集群时都会增加这个计数。每个记录更新都包含用户当前的“化身”编号，确保即使在移动后，CloudKit 仍然可以通过查看化身和版本号来找出正确的更新顺序。

当他们转换到这个新系统时，CloudKit 面临着处理旧数据的挑战，这些数据没有这些版本号。他们巧妙地通过使用一个特殊函数来克服这一挑战，该函数对旧更新使用先前的系统进行排序，然后再处理新的更新。这意味着不需要对应用程序进行复杂的更改，也不需要留下过时的代码。这个函数考虑了版本、版本和旧的更新计数器值，以维护记录的正确顺序。

FoundationDB 的设计是为了高并发而不是低延迟。这意味着它可以**同时处理大量任务，而不是关注单个任务的速度。**

为了充分利用这种设计，**Record Layer 大部分工作都是“异步”进行的**—它排队等待未来完成的任务，同时允许在此期间完成其他工作。这种方法有助于弥补在这些任务中可能发生的任何延迟。

然而，FoundationDB 用于与其数据库通信的工具设计为使用单个线程进行一次一次的操作进行处理网络。在早期版本中，这个设置导致系统中的交通堵塞，因为一切都在等待在这个网络线程中的轮到。**Record Layer 一直在使用这种单线程方法，这导致了瓶颈的产生。**

为了改进这一点，苹果减少了这个网络线程上的工作负载。**现在，复杂的任务似乎更快，因为系统在与数据库同时进行多个任务而不是形成队列。**这样，潜在的延迟或明显的缓慢被隐藏起来，因为系统在开始另一个任务之前不会等待一个任务完成。

在 FoundationDB 中，如果一个事务正在读取某些键，而另一个事务同时修改这些相同的键，这将导致“事务冲突”。**FoundationDB 允许通过提供控制来管理这些冲突，这些控制控制着在读取或写入时可能导致这些冲突的键集合。**

避免不必要的冲突的一种常见方法是对一系列键进行一种不会引起冲突的特殊读取，称为“快照”读取。如果这次读取发现了重要的键，事务将只标记这些特定的键可能存在的冲突，而不是整个范围。这确保了事务只受到实际影响其结果的变化的影响。

Record Layer使用这种策略来高效管理称为跳表的结构，该结构是其排名索引系统的一部分。然而，手动设置这些冲突范围可能会很棘手，并且可能会导致难以识别的错误，特别是当它们与应用程序的主逻辑混在一起时。因此，**建议基于FoundationDB构建的系统创建更高级的工具，例如自定义索引，来处理这些模式**。这种方法有助于避免将松散的冲突规则责任留给每个客户端应用程序，这可能会导致错误和不一致性。

* * *

感谢阅读！如果您对阅读完整的FoundationDB Record Layer论文感兴趣，可以在这里查看：[FoundationDB Record Layer：多租户结构化数据存储](https://www.foundationdb.org/files/record-layer-paper.pdf)。
