<!--yml

category: 未分类

日期：2024年5月27日 14:25:38

-->

# 对 PostgreSQL 在 2024 年的思考 | 乔纳森·卡茨

> 来源：[https://jkatz05.com/post/postgres/postgresql-2024/](https://jkatz05.com/post/postgres/postgresql-2024/)

# 对 PostgreSQL 在 2024 年的思考

*Tue, Jan 2, 2024 *18分钟读完**

**我经常听到也自己问的一个问题是“PostgreSQL会走向何方？”这是一个深刻的问题：它不仅限于核心数据库引擎的工作，而是包括社区中的所有事物，其中也包括相关的开源项目和活动以及社区发展。尽管 PostgreSQL 很受欢迎，已经第四次被选为[DB Engine的“年度数据库管理系统”](https://db-engines.com/en/blog_post/106)，但是有时放慢脚步，思考一下 PostgreSQL 在未来会是什么样子，是一个好主意。尽管这可能不会立即导致改变，但它确实有助于为社区中正在进行的所有工作提供背景。**

**新的一年是一个很好的机会，问一下“PostgreSQL会走向何方？”这是我一直在思考的一个问题。所以在我们进入2024年之际，我想分享一些我对PostgreSQL未来发展的看法。这并不意味着是一个路线图，而仅仅是我个人对PostgreSQL未来走向的思考。**

## **PostgreSQL 功能的发展**

**在[PGCon 2023开发者会议](https://wiki.postgresql.org/wiki/PgCon_2023_Developer_Meeting)上，我提出了一个名为[“PostgreSQL用户面临的重大挑战是什么？”](https://wiki.postgresql.org/wiki/PgCon_2023_Developer_Meeting#What_are_the_big_challenges_for_our_users.3F_What_are_the_big_challenges_for_us_to_solve.3F)的话题。这个话题的目的是探讨常见的用户需求，了解数据库工作负载的方向，以确定我们是否将PostgreSQL建设成与数据库工作负载相一致。根据许多对话和观察，我提出了三个广泛的功能桶来进行研究：**

+   ****可用性****

+   ****性能****

+   ****开发者功能****

**这些都是 2024 年及以后的持续工作领域，但 PostgreSQL 绝对可以在未来一年中采取一些步骤，在所有这些领域中进行改进。下面我将详细介绍这些功能组的更多细节。**

### **可用性**

**持续改善PostgreSQL集群的可用性是当前和潜在的PostgreSQL用户最常提到的第一、第二和第三“特性”。我没有夸张：虽然重新启动PostgreSQL几乎是瞬间完成的，但在极端情况下可能会花费太多时间（虽然这些情况很极端）。此外，锁定导致写入阻塞的操作可能被视为“停机时间”。**

**虽然大多数 PostgreSQL 用户目前可以实现他们的持续运行时间要求，但是有一类具有关键运行时间要求的工作负载，我们可以通过额外的开发工作更好地支持 PostgreSQL。本节（以及博客文章）的大部分内容都集中在一个功能领域，持续的改进将使 PostgreSQL 可以部署在更多具有这些要求的环境中。**

#### **逻辑复制如何帮助实现活动-活动、蓝/绿、零停机升级和其他工作流程**

**对于现有的 PostgreSQL 用户和希望迁移到 PostgreSQL 的用户来说，围绕可用性的特性是最重要的需求。通常，这集中在[高可用性](https://zh.wikipedia.org/wiki/%E9%AB%98%E5%8F%AF%E7%94%A8%E6%80%A7)或者在计划（更新）或未计划（故障）中断期间继续访问数据库（特别是读/写访问）的能力。PostgreSQL 提供了许多功能来支持高可用性，包括流复制。然而，要最大化高可用性仍然需要使用额外的服务或类似[Patroni](https://github.com/zalando/patroni)这样的实用程序来实现许多持续运行时间目标。**

**我与许多用户交谈时发现，他们对 PostgreSQL 提供的可用性感到满意：它适用于他们大多数的用例。然而，我注意到了一种新兴的趋势，即在 PostgreSQL 上需要更高可用性的工作负载，其中 15-30 秒的离线窗口不够用。这既适用于计划中的中断（例如次要版本升级、主要版本升级）又适用于未计划中的中断。我甚至与那些需要只能中断 1 秒的工作负载的用户进行了交谈 - 虽然我最初持怀疑态度，但当我了解到工作负载的内容时，我确实认为 1 秒对于他们来说是一个合理的要求！**

**对于 PostgreSQL 的一个关键特性将继续提高可用性，那就是[逻辑复制](https://www.postgresql.org/docs/current/logical-replication.html)。逻辑复制允许从数据库实时流式传输更改到任何能够理解 PostgreSQL 逻辑复制协议的系统中。PostgreSQL 中的逻辑复制[已经存在一段时间了](/post/postgres/postgres-10-tribute/)，但是[最近的发布](https://www.postgresql.org/about/news/postgresql-16-released-2715/)添加了重要的增强功能，可以更好地支持可用性用例，包括功能和性能特性。**

**与物理（或二进制）复制相比的一个优势是，你可以使用逻辑复制将更改从 PostgreSQL 15 流式传输到 PostgreSQL 16 系统，作为主要版本升级的一部分。 这可以帮助减少执行主要版本升级所需的停机时间（这是一个示例，[Instacart如何使用逻辑复制实现了零停机的主要版本升级](https://www.instacart.com/company/how-its-made/zero-downtime-postgresql-cutovers/)），但是在 PostgreSQL 中仍需要进行一些工作以改进这种用法以及其他高可用性用例。 其他功能将有助于解锁更无缝地支持 [蓝绿部署](https://en.wikipedia.org/wiki/Blue%E2%80%93green_deployment) 的方式。**

**逻辑复制还可以作为高可用性机制的一部分使用。 一种技术，“主动-主动复制”，允许多个数据库同时接受写入并在它们之间复制更改。 该技术通常用于具有“不超过 1s 的不可用性”要求的系统：如果写入数据库不可用，则应用程序可以将其数据库流量切换到另一个写入数据库，而无需等待其被提升。 尽管这听起来很理想，但构建和管理主动-主动系统非常复杂：它会影响应用程序设计，需要你拥有写入冲突管理和解决策略，并且需要仔细的容错监控以帮助确保数据完整性（例如“冲突风暴”）和复制健康状况（例如如果一个实例无法在几个小时内复制更改会发生什么？）。**

**但是，主要版本升级和主动-主动情况确实为我们如何继续改进 PostgreSQL 中的逻辑复制提供了一个路线图。 [阿米特·卡皮拉](https://amitkapila16.blogspot.com/)，他领导了许多逻辑复制功能的工作，我今年开发了一个演讲，名为 [在 PostgreSQL 中实现主动-主动复制的旅程](https://www.postgresql.eu/events/pgconfeu2023/sessions/session/4783/slides/434/pgconfeu2023_active_active.pdf) （[我们共同呈现的一个版本的视频](https://www.youtube.com/watch?v=jPp4XIY4XRw)），谈到了为什么解决这些用例很重要，PostgreSQL 逻辑复制的当前技术水平，以及我们需要做什么工作来使 PostgreSQL 更好地支持这些用例。 好消息是：截至 PostgreSQL 16，我们已经具有支持主动-主动，蓝绿部署和零停机主要版本升级的大多数基础模块 - 即使它们不在核心中，也有 PostgreSQL 扩展可以提供此功能（披露：我参与了一种这样的扩展，[`pgactive`](https://aws.amazon.com/blogs/database/using-pgactive-active-active-replication-extension-for-postgresql-on-amazon-rds-for-postgresql/)）。**

**在2024年，有多个努力来帮助弥补这些功能差距。针对PostgreSQL 17（通常的免责声明，可能不包括其中），我们重点关注确保逻辑复制可以与关键工作流程一起使用，例如[`pg_upgrade`](https://www.postgresql.org/docs/current/pgupgrade.html)和[高可用性系统](https://commitfest.postgresql.org/46/4423/)，并致力于支持附加更改的复制（例如[sequences](https://commitfest.postgresql.org/46/3823/)）。此外，我们必须继续在逻辑复制中支持更多命令（例如[DDL](https://commitfest.postgresql.org/46/3595/)），继续改善性能（更多并行支持，工作器优化），并添加简化逻辑复制管理的功能（节点同步/重新同步）。**

**所有这些努力将使得在更多需要高可用性的工作负载中使用PostgreSQL成为可能，并简化用户在生产环境中实施新更改的方式。虽然在增强PostgreSQL中的逻辑复制方面还有更多工作要做，但看起来在2024年我们将获得更多功能，这些功能将帮助用户在关键环境中运行PostgreSQL。**

#### **解锁锁定的状态**

**另一个需要考虑的可用性领域是围绕模式维护操作（即[DDL](https://en.wikipedia.org/wiki/Data_definition_language)语句），例如[`ALTER TABLE`](https://www.postgresql.org/docs/current/sql-altertable.html)会在表上获取一个[`ACCESS EXCLUSIVE`](https://www.postgresql.org/docs/current/explicit-locking.html#LOCKING-TABLES)锁，这将阻塞该表上的所有其他写操作。对于许多用户来说，即使只是对其数据的一个子集而言，这等同于不可用。随着其他关系数据库包括此功能的支持，PostgreSQL对非阻塞/在线模式维护操作的完整支持变得更加明显。**

**有各种实用程序和扩展可以让你运行非阻塞模式的模式更新，但更方便，而且可能更高效的方式是在PostgreSQL中本地支持更多的非阻塞模式变化。基于设计，我们可能已经有了构建这一功能的基础，但这需要一些时间。虽然我不知道有没有积极的实施工作，但我认为在2024年，我们需要在使用户能够运行大多数（如果不是全部）DDL命令而不会阻塞写入方面取得更多进展。**

### **性能**

**性能是一个“最近你为我做了什么”的特性：我们总是可以更快！好消息是，PostgreSQL以垂直扩展或在向单个实例提供更多硬件资源时可以扩展的声誉。虽然有用例需要水平扩展读写，但我们需要继续确保PostgreSQL可以随着计算和内存资源的增长而继续扩展。**

这里有一个更加“实际”的表述：有一个亚马逊EC2实例，拥有[448 vCPU和24TB的RAM](https://aws.amazon.com/ec2/instance-types/high-memory/) - PostgreSQL能否充分利用单个实例上的所有资源？观察当前和即将使用的硬件可以为PostgreSQL用户设定一个合适的目标，以便我们持续提高PostgreSQL的性能。

进入2024年，已经有多个努力在帮助继续垂直扩展PostgreSQL的可能性。其中最大的努力之一，也是一个持续多年的项目，是支持PostgreSQL中的直接IO（DIO）和异步IO（AIO）。有关详情，我将转交给Andres Freund在[PGConf.EU](https://www.pgconf.eu/)上关于[将AIO添加到PostgreSQL的状态](https://anarazel.de/talks/2023-12-14-pgconf-eu-path-to-aio/path-to-aio.pdf)的幻灯片介绍，但看起来在2024年，我们将更接近完全支持AIO。

我对另一个努力很感兴趣，即[并行恢复](https://wiki.postgresql.org/wiki/Parallel_Recovery)。拥有大量写入工作负载的PostgreSQL用户往往会延迟[检查点](https://www.postgresql.org/docs/current/sql-checkpoint.html)以推迟I/O工作负载。如果PostgreSQL发生崩溃并且一段时间内没有发生检查点，这对于繁忙的系统可能会产生问题。当PostgreSQL重新启动时，它进入“崩溃恢复”阶段，在此期间它会重放自上次检查点以来的每一次更改，以达到一致状态。在崩溃恢复期间，PostgreSQL无法接受读取或写入操作，这意味着它不可用。这对于繁忙的系统来说是有问题的：虽然PostgreSQL可以接受并发写入，但它只能用一个进程重放更改。如果在上次检查点后的一个小时发生系统崩溃，系统可能需要几个小时才能达到一致状态，而在此期间系统将处于离线状态！

克服这个限制的一种方式是支持“[并行恢复](https://wiki.postgresql.org/wiki/Parallel_Recovery)”，或者能够并行重放更改。在[PGCon 2023](https://www.pgcon.org/)上，铃木浩一（Koichi Suzuki）详细介绍了PostgreSQL如何支持并行恢复。这不仅适用于崩溃恢复，还涉及PostgreSQL如何重放任何WAL更改（例如时间点恢复）。虽然这是一个非常具有挑战性的问题，但支持并行恢复有助于PostgreSQL继续垂直扩展，因为用户可以进一步优化大量写入工作负载，并减轻系统崩溃导致无法及时恢复上线的风险。

**这不是与性能相关的功能的详尽清单。在PostgreSQL服务器性能方面还有许多努力，包括索引优化、锁定改进、利用硬件加速等。除此之外，还有一些关于客户端的工作，如驱动程序和连接池器，可以为应用程序与PostgreSQL的交互带来额外的性能提升。从社区在2024年正在努力的方向来看，我相信我们将继续看到PostgreSQL各个领域的性能提升。**

### **开发人员功能**

**我认为“开发人员功能”是一个相当广泛的范畴，涉及用户如何围绕PostgreSQL构建和开发其应用程序的方式。这包括SQL语法、函数、[过程化语言支持](https://wiki.postgresql.org/wiki/PL_Matrix)和其他能够帮助用户构建应用程序并从其他数据库系统过渡的功能。一个这样创新的例子是在PostgreSQL 14中添加的[`multirange`](https://www.postgresql.org/docs/current/rangetypes.html)数据类型，让用户可以将非连续的范围进行分组。这在排程等方面有着许多实际用途，个人而言，它让我将[P L/pgSQL代码](https://www.crunchydata.com/blog/better-range-types-in-postgres-14-turning-100-lines-of-sql-into-3)从数百行简化为大约三行。开发人员功能也是一种追踪PostgreSQL如何支持新兴工作负载（如[JSON或向量](/post/postgres/vectors-json-postgresql/)）的方法。**

**目前，在PostgreSQL开发人员功能方面，很多创新都发生在扩展中，这是PostgreSQL可扩展模型的优势。在服务器本身，有一些领域PostgreSQL的开发人员功能发布的速度正在落后。例如，PostgreSQL是[第一个支持JSON作为可查询数据类型的关系型数据库](/post/postgres/vectors-json-postgresql/)，但在实现SQL/JSON标准中指定的语法和功能方面一直滞后。PostgreSQL 16发布了几个SQL/JSON语法功能，2024年还有多个计划，将包括更多的SQL/JSON规范。**

**话虽如此，我们应该投资于在PostgreSQL中添加那些无法通过扩展添加的开发人员功能，例如SQL标准功能。我建议将重点放在已经在其他数据库中可用的功能上，比如更多的SQL/JSON标准（例如`JSON_TABLE`）、系统版本表（用于审计和“闪回”/双时间轴查询以查看特定时间点的数据）和模块支持（用于“打包”存储过程）。**

**此外，除了之前提到的可用性和性能方面的关注，我们还应该继续简化用户如何从其他数据库迁移到PostgreSQL的过程。作为我的日常工作的一部分，我有机会阅读了大量关于从商业数据库迁移到PostgreSQL的迁移策略的内容，尽管我们仍有充分机会简化这个过程，同时增强PostgreSQL的功能。这包括其他数据库中可用的功能（例如全局临时表、全局分区索引、[自主事务](https://www.postgresql.org/message-id/f7470d5a-3cf1-4919-8404-5c4d91341a9f@tantorlabs.com)）以及在PL/pgSQL中添加更多功能和性能优化（批量数据处理函数、[模式变量](https://commitfest.postgresql.org/46/1608/)、[缓存函数元数据](https://commitfest.postgresql.org/46/4684/)）。所有这些事情都可以改善PostgreSQL开发人员的体验，同时使来自其他关系数据库的用户更容易接受PostgreSQL。**

**最后，我们需要看看如何继续支持来自人工智能/机器学习数据的新兴工作负载，特别是矢量存储和搜索。在 [PGCon](https://www.pgcon.org/) 2023年会上，虽然人们希望在PostgreSQL本身中看到原生的矢量支持，但共识是在像 [pgvector](https://github.com/pgvector/pgvector) 这样的扩展中实现功能将让我们更快地支持这些工作负载（而这个 [策略似乎已经奏效](/post/postgres/pgvector-overview-0.5.0/) ，在 [矢量数据上取得了很好的性能结果](https://aws.amazon.com/blogs/database/accelerate-hnsw-indexing-and-searching-with-pgvector-on-amazon-rds-for-postgresql/)）。然而，鉴于[矢量工作负载的许多特性](https://www.postgresql.eu/events/pgconfeu2023/sessions/session/4592/slides/435/pgconfeu2023_vectors.pdf)，我们可以对PostgreSQL进行一些增强，包括针对[处于活动查询路径中的TOAST’d数据的计划优化](https://www.postgresql.org/message-id/ad8a178f-bbe7-d89d-b407-2f0fede93144@postgresql.org)，以及探索如何更好地支持在`ORDER BY`子句中发生大量筛选步骤的查询。**

**我确实认为我们可以在2024年在所有这些领域取得很大进展，并继续直接向PostgreSQL添加功能，使其更容易构建应用程序，即使我们在PostgreSQL周围看到扩展方面的功能激增。**

### **但安全性呢？**

**我确实想快速讨论一下安全功能。 PostgreSQL确实以在注重安全的环境中支持工作负载而享有良好的声誉，但总还有更多工作要做。 在过去的几年中，PostgreSQL添加了对[透明数据加密](https://wiki.postgresql.org/wiki/Transparent_Data_Encryption)（TDE）的本机支持引起了很多关注，但我们还可以在其他领域继续创新。 这包括添加对其他认证方法或机制的支持（OIDC是最大的要求之一），并探索联邦授权模型的可能性，以允许PostgreSQL继承来自其他系统的权限。 尽管这在今天是具有挑战性的，但我建议我们看看如何支持数据库级别的TDE。 我会把这个讨论简短，因为今天有满足这些功能的要求的方法，但我们当然可以继续努力支持完全本地的支持。**

**现在让我们看看2024年PostgreSQL可以取得进展的其他领域。**

## **扩展**

**PostgreSQL被设计为可扩展的：您可以在不分叉的情况下向PostgreSQL添加功能。 这包括新的数据类型、索引方法、与其他数据库系统配合工作的方法、使其更容易管理PostgreSQL功能的实用程序、[附加编程语言](https://wiki.postgresql.org/wiki/PL_Matrix)，甚至[让你编写自己的扩展的扩展](https://github.com/aws/pg_tle)。 人们已经围绕特定的PostgreSQL扩展（例如[PostGIS](https://postgis.net/)）建立了开源社区和公司，PostgreSQL扩展使得支持各种工作负载（地理空间、时间序列、分析、人工智能）从一个数据库中成为可能。 有[成千上万可用的PostgreSQL扩展](https://gist.github.com/joelonsql/e5aa27f8cc9bd22b8999b7de8aee9d47)，它们确实是PostgreSQL的“力量倍增器”，可以在让用户快速为他们的数据库构建功能的同时推动显著的采用！**

**所有这一切的副作用是我们现在看到了“扩展[过度](https://en.wikipedia.org/wiki/Urban_sprawl)”。 我怎么知道要使用哪个扩展？扩展的支持水平是多少？我怎么知道扩展将继续得到积极维护？我如何帮助对扩展做出贡献？甚至“我从哪里下载扩展”已经成为一个大问题：虽然postgresql.org有一个[不完整的扩展列表](https://www.postgresql.org/download/products/6-postgresql-extensions/)，并且[社区包](https://www.postgresql.org/download/)维护了一组扩展，但现在有多个可用的PostgreSQL扩展存储库（[PGXN](https://pgxn.org/)，[dbdev](https://database.dev/)，[Trunk](https://pgt.dev/)），以及[pgxman](https://pgxman.com/)。**

**PostgreSQL 社区的一大优势是其分布广泛，但我们可以更方便地帮助用户在这一范围内做出明智的选择来管理他们的数据。我认为2024年是一个将更多核心资源投入到如何表示 PostgreSQL 扩展的机会，并帮助用户了解何时使用某些扩展及其开发成熟度水平，并帮助扩展构建者获得治理和维护资源。**

## **社区建设**

**我希望在2024年的思考中围绕社区建设展开。自从我开始参与 PostgreSQL 贡献者社区以来，社区已经显著增长，并且社区在[认可贡献者](https://www.postgresql.org/community/contributors/)方面做得更好，不仅仅是在代码库上（需要指出这里仍有改进空间）。但我们可以做得更好，我想特别强调三个方面：指导和[DEI](https://en.wikipedia.org/wiki/Diversity,_equity,_and_inclusion)（多元化、公平和包容性），以及透明度，这将有助于项目的各个方面。**

**在[2023年 PGCon 开发者会议](https://wiki.postgresql.org/wiki/PgCon_2023_Developer_Meeting#What_are_the_big_challenges_for_our_users.3F_What_are_the_big_challenges_for_us_to_solve.3F)期间，[Melanie Plageman](https://mastodon.social/@melanieplageman/) 对成为 PostgreSQL 的新贡献者的经历以及加速过程中的挑战进行了非常详细的分析。Melanie 指出了许多问题：学习如何贡献到 PostgreSQL（开始接触代码库、在邮件列表上沟通）的启动时间，将补丁提交到可提交状态并且有提交者对其感兴趣所需的努力，以及可能出于善意给出的指导（通过审查补丁开始！）实际上可能比编写代码更具挑战性，以及反馈的传递方式。**

**关于最后一点，如何提供反馈，我想提一下 Robert Haas 的一篇[卓越博客文章](https://rhaas.blogspot.com/2023/12/praise-criticism-and-dialogue.html)，该文章专门探讨了在[提供反馈时表达赞美的力量](https://rhaas.blogspot.com/2023/12/praise-criticism-and-dialogue.html) - 这些事情确实会产生影响，这是一个良好的提醒，即使在我们进行批评时，我们也应该给予支持。**

**回到 Melanie 的论点，导师制是整个社区可以做得更好的事情。就我个人而言，我承认我在项目倡导领域的导师制，包括帮助更多的人为[网络基础设施](https://www.postgresql.org/developer/related-projects/)和[发布流程](https://www.postgresql.org/about/press/presskit16/)做出贡献方面做得不够好。这并不意味着 PostgreSQL 缺乏导师制 - 我可以数出社区中众多的导师 - 但在如何帮助人们开始贡献并找到能够指导他们前进的导师方面，我们可以做得更好。**

**2024年将成为建立更好导师制流程的一个契机，我们计划在2024年5月在温哥华举办[PGConf.dev 2024](https://2024.pgconf.dev/)，届时将测试一些这些想法。**

**（一些历史：在[PGConf.dev](https://www.pgconf.dev/)之前，[PGCon](https://www.pgcon.org/) 是 PostgreSQL 贡献者聚集讨论未来发展周期战略项目的活动。PGCon 由 Dan Langille 于2007年至2023年组织，并在经过多年的组织后，他已经准备好将努力扩展给一群人，并帮助建立了[PGConf.dev](https://www.pgconf.dev/)）。**

**[PGConf.dev](https://www.pgconf.dev/) 是一个为想要为 PostgreSQL 做出贡献的人举办的会议，涵盖了围绕 PostgreSQL 开发的主题（包括核心服务器和所有围绕 PostgreSQL 的开源项目，例如扩展和驱动程序）、社区建设和开源思想领导力。PGConf.dev 的一个重要部分致力于导师制，并计划包括围绕如何为 PostgreSQL 做出贡献的研讨会。如果你正在寻找为 PostgreSQL 做贡献的方式，我强烈建议参加或者[提交演讲](https://2024.pgconf.dev/cfp/)！**

**这引出了 PostgreSQL 社区在[DEI（多元化、公平、包容）](https://en.wikipedia.org/wiki/Diversity,_equity,_and_inclusion)方面可以改进的问题。我强烈建议阅读[卡伦·杰克斯](https://karenjex.blogspot.com/)和[拉提蒂娅·阿夫罗特](https://mydbanotebook.org/)的 PGConf.eu 2023 演讲[试图在肯的Mojo Dojo Casa House里做芭比](https://www.postgresql.eu/events/pgconfeu2023/schedule/session/4913-trying-to-be-barbie-in-kens-mojo-dojo-casa-house/)幻灯片和观看视频（一旦可用），因为这是一个关于我们如何继续使 PostgreSQL 社区更具包容性的深入演示。社区在这方面已经取得了进展（卡伦和拉提蒂娅指出了帮助此事的举措），但我们仍有待改进，我们应积极主动地努力解决反馈问题，以确保为 PostgreSQL 做贡献是一种受欢迎的体验。我们都可以采取一些行动，例如在不当行为（例如性别歧视）发生时立即予以指出，并提供为何不恰当的指导。**

最后，透明度也很重要。这在开源项目中可能看起来有点奇怪，因为，嗯，它是开放的。但是有一些治理问题并不是公开讨论的，了解决策是如何形成的很有帮助。[PostgreSQL行为准则委员会](https://www.postgresql.org/about/policies/coc_committee/)提供了一个很好的例子，展示了社区如何公开讨论需要敏感性的问题。每年，行为准则委员会都会发布一份工作报告（[这是2022年的报告](https://www.postgresql.org/about/policies/coc/reports/2022/)），包括案例的高层描述和整体统计数据。这是一个我们可以在许多参与可能因敏感性而需要保密的任务的PostgreSQL团队中复制的做法。

## 结论：这原本应该是一篇较短的帖子。

当我最初开始写这篇文章时，我以为它会是一篇简洁的帖子，我会在几个小时内完成它。几天后&mldr;

严肃地说，PostgreSQL处于一个良好的状态。它仍然很受欢迎，其可靠性、健壮性和性能的声誉仍然良好。但我们仍然可以做得更好，而好消息是社区正在积极努力在各个方面改进。

虽然这些是关于PostgreSQL在2024年及以后可能做的事情的思考，但PostgreSQL今天已经做了很多事情。事实上，像“PostgreSQL将何去何从”这样的问题确实给了我们一个机会，让我们回顾一下PostgreSQL在过去几年取得的所有进展，同时展望未来！
