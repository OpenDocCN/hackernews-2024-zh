<!--yml

类别：未分类

日期：2024-05-27 15:23:29

-->

# Jepsen：RavenDB 6.0.2

> 来源：[`jepsen.io/analyses/ravendb-6.0.2`](https://jepsen.io/analyses/ravendb-6.0.2)

# 背景

[RavenDB](https://ravendb.net)是一个分布式文档数据库，[反复宣传](https://ravendb.net/why-ravendb)其支持 ACID 事务。它旨在处理 OLTP 工作负载，并提供各种[ETL 路径](https://ravendb.net/docs/article-page/6.0/csharp/server/ongoing-tasks/etl/basics)以将数据导出到其他系统。其事务性 API 围绕着一个[会话](https://ravendb.net/docs/article-page/6.0/csharp/client-api/session/what-is-a-session-and-how-does-it-work)句柄展开，“代表特定数据库上的单个业务交易。”用户创建会话，执行读取和写入等操作，最后调用`session.saveChanges()`以提交其写入为原子单位。

RavenDB 可以通过自动故障转移在一组节点之间[复制数据](https://ravendb.net/why-ravendb/high-availability)。分片是[正在进行中](https://issues.hibernatingrhinos.com/issue/RavenDB-8115)或[在 6.0 中已准备就绪](https://ravendb.net/features/clusters/sharding)，这取决于您正在阅读文档的部分。RavenDB 包括与自制[查询语言](https://ravendb.net/docs/article-page/6.0/csharp/client-api/session/querying/how-to-query)包装的次要索引，[文档的多个修订版本](https://ravendb.net/docs/article-page/6.0/csharp/document-extensions/revisions/overview)，[时间序列](https://ravendb.net/docs/article-page/6.0/csharp/document-extensions/timeseries/overview)数据类型和基于[CRDT](https://crdt.tech/)的[计数器](https://ravendb.net/why-ravendb/multi-model)。在本文中，我们将重点介绍 RavenDB 的事务键值操作。

## 复制

根据 RavenDB 的[高可用性](https://ravendb.net/why-ravendb/high-availability)页面，数据库接受集群中所有节点上的写入和读取。它使用[Raft](https://raft.github.io/)共识算法，理论上应允许 RavenDB 提供高达[强一致性](https://jepsen.io/consistency/models/strict-serializable)的一致性模型。然而，该页面继续说操作员可以“轻松设置拓扑结构，其中端点在网络中断的情况下可以独立运行”。在一篇博客文章中，首席执行官 Oren Eini 重复了这一说法：

> 如果一个节点位于网络连接中断的地方，该节点可以继续离线操作，本地接收数据。一旦连接恢复，节点将获取其处理的数据并在整个集群中复制。

这将使 ACID 事务变得不可能。ACID 中的“I”指的是“隔离”：事务必须看起来是独立执行的，不受其他事务干扰。这个属性被形式化为[可串行化](https://jepsen.io/consistency/models/serializable)：等价于一些完全有序的、非并发的事务执行。我们知道完全可用的系统[无法提供](https://jepsen.io/consistency)可串行化甚至[快照隔离](https://jepsen.io/consistency/models/snapshot-isolation)。RavenDB 可能提供[因果一致性](https://jepsen.io/consistency/models/causal)或[读取已提交](https://jepsen.io/consistency/models/read-committed)，但更强的一致性模型在理论上是被限制的。

[高可用性的第二页](https://ravendb.net/features/clusters/high-availability)解释了 RavenDB 中存在两个层次，而 Raft 仅用于集群元数据：

> +   首先，集群层由一种叫做 Raft 的一致性协议管理。在 CAP 定理中它是 CP（一致性和分区容忍）的。
> +   
> +   第二层，即数据库层，是 AP 的（即使有分区也始终可用，并且最终一致），并且由不同节点上的数据库之间的八卦协议处理，形成多主网状结构，并在彼此之间复制数据。
> +   
> RavenDB 利用不同的层次来达到不同的目的。在集群层面，一致性协议确保运算者可以放心知道他们的命令被接受并执行。在数据库层面，你知道 RavenDB 永远不会丢失写入数据，并且始终保持你的数据安全。

这也让人困惑。AP 系统以可用性而闻名，而不是安全性；在 AP 寄存器中，丢失更新是一个被充分理解的问题。RavenDB 声称提供带有 ACID 保证的事务。然而，这些事务显然是通过一个最终一致、完全可用的复制系统路由的。有[数据库将](https://docs.datomic.com/pro/getting-started/brief-overview.html)一个（例如）顺序事务协调器与一个最终一致的数据存储耦合在一起，以提供可串行化，但从这份文档中并不清楚 RavenDB 如何将集群和数据库层链接在一起以确保安全。

[RavenDB 集群设计章节内部](https://ravendb.net/learn/inside-ravendb-book/reader/4.0/6-ravendb-clusters)确认 Raft 仅用于集群元数据。允许在每个节点上写入，并且完全可用：

> RavenDB 在数据库内部使用多主复制，并且总是能够接受写入。
> 
> 换句话说，即使大多数集群宕机，只要有一个节点可用，我们仍然可以处理读取和写入。

另一方面，RavenDB 的[NoSQL 中的 ACID 事务](https://ravendb.net/articles/acid-transactions-in-nosql-ravendb-vs-mongodb)文章则声称相反：

> 与单节点版本一样，RavenDB 只需一轮 Raft 共识即可提交事务。

如果 Raft*参与*交易，RavenDB 可以提供高强度串行化，但交易不能完全可用。实际上，RavenDB 的[集群文档](https://ravendb.net/docs/article-page/6.0/Csharp/server/clustering/cluster-transactions)澄清了实际上有两条不同的事务路径。默认模式称为*单节点*事务，允许冲突“当两个客户端尝试在两个不同的数据库节点上修改相同的一组文档时。” *集群范围事务*使用 Raft 来防止冲突，允许事务“偏向一致性而不是可用性。”要执行集群范围事务，必须[设置](https://ravendb.net/docs/article-page/6.0/java/client-api/session/saving-changes#transaction-mode---cluster-wide)`TransactionMode = CLUSTER_WIDE`。

这些事务路径保证了哪些安全性质？为此，我们需要详细考虑 RavenDB 的 ACID 声明。

## ACID

RavenDB 的[主页](https://ravendb.net/why-ravendb)显眼地宣传“跨多个文档和整个集群的 ACID 数据库事务。”它的[ACID 数据库事务](https://ravendb.net/why-ravendb/acid-transactions)页面解释说，没有事务的数据库“不算得上是数据库。”它自豪地宣称 RavenDB“保证 ACID 而不牺牲性能”，并指出由于其分布式 ACID 保证，“开发人员无需处理部分数据传输的众多场景和数据存储的复杂性。”

RavenDB 的[ACID Transactions in NoSQL](https://ravendb.net/articles/acid-transactions-in-nosql-ravendb-vs-mongodb)文章解释说 RavenDB“从 1.0 版本开始就能够进行多文档事务”：

> 由于它是针对这一点进行了优化，甚至没有必要提供非 ACID 选项。任何数据库操作的组合都可以合并为 ACID 事务。作为用户，您无需自己实现 ACID 保证，而且可以根据自己的需求自由设计文档……
> 
> RavenDB 的设计旨在每个事务仅进行一次服务器往返。 RavenDB 的会话对象版本跟踪一系列命令，将它们收集为一个批处理，并在调用方法`session.saveChanges()`时将它们全部发送到服务器中进行一次往返。

再次，[RavenDB 声称](https://ravendb.net/articles/acid-cluster-distributed-nonrelational-database) 自己是“在非关系上下文中提供 ACID 的先驱数据库。2010 年，RavenDB 在多个文档上提供了 ACID 一致性。” 但是，这些保证仅在单个节点上保持：不同节点上的并发客户端可能违反隔离。在 2020 年秋季发布的 RavenDB 4.0 中，引入了整个集群的事务路径，将事务“从在多个文档上执行 ACID 到在整个集群上执行 ACID。”

[Transaction FAQ](https://ravendb.net/docs/article-page/6.0/csharp/client-api/faq/transaction-support) 表示“对文档执行的所有操作都是完全 ACID”，但立即自相矛盾地表示“在单个事务中，所有操作都在快照隔离下运行”。[DBDB](https://dbdb.io/db/ravendb) 认为这意味着 RavenDB 默认提供快照隔离。在 [2020 年的网络研讨会](https://www.youtube.com/watch?v=5ZXBR3croMA&t=39m) 中，CEO Oren Eini 确认了这一立场：“RavenDB 默认使用快照隔离，事务在同一节点上发生的操作之间实际上会观察到 Serializable。”

有迹象表明 RavenDB 可能提供的东西比快照隔离要弱。埋藏在 *Inside RavenDB* 书中，在文档建模章节中，有一部分关于[并发控制](https://ravendb.net/learn/inside-ravendb-book/reader/4.0/3-document-modeling#concurrency-control)。该部分解释了 RavenDB（至少在 4.0 版本中）不执行并发控制，而是默认使用最后写入获胜的冲突解决方案。

> 如果两个请求同时尝试修改同一文档会发生什么？这取决于您究竟要 RavenDB 做什么。如果您什么都没做，RavenDB 将依次执行这两个修改，最后一个会胜出。没有办法控制哪个会是最后的。请注意，两个操作都会执行。

这与 ACID 隔离的相反。隔离的事务似乎是顺序执行的，而不是并发执行的。这也与快照隔离的声明相矛盾：最后写入获胜的注册表允许各种各样的异常情况，这些异常情况在快照隔离下将被禁止，包括[丢失更新](https://dzone.com/articles/conflict-resolution-using-last-write-wins-vs-crdts)。然而，这本书已经过时了两个主要版本；可能不适用于 6.0.2。

那么集群范围的事务呢？[Cluster Transactions](https://ravendb.net/docs/article-page/6.0/java/server/clustering/cluster-transactions#concurrent-cluster-wide-and-single-node-transactions) 页面似乎是最终的答案。“并发的集群范围事务被保证看起来像是一个接一个地运行（`serializable` 隔离级别）。”

从中，Jepsen 推断 RavenDB 的默认事务设置应该默认确保快照隔离，并在单节点系统中确保可串行性。集群范围的事务应该在全局范围内确保可串行性。

# 测试设计

在 2020 年，RavenDB [编写了他们自己的 Jepsen 测试](https://github.com/ml054/jepsen/) 并在 [网络研讨会](https://www.youtube.com/watch?v=5ZXBR3croMA&) 中宣布，根据该测试，“一切正常”。他们的测试检查了针对单个文档的单个 [读取和写入](https://github.com/ml054/jepsen/blob/08960da98ab9bf7959fce014c63b146b903cbe6c/ravendb/src/jepsen/ravendb.clj#L164-L196) 的线性可达性。它没有评估多操作或多文档事务。

我们为运行在单个 Debian Bookworm 节点上的 RavenDB 6.0.2 [设计了一个新的测试工具](https://github.com/jepsen-io/ravendb/tree/4431310402f334ffdb18a5a7ec819316847b642b)。我们的测试使用了 RavenDB 的 JVM 客户端库版本为 5.0.4\. 我们没有评估多节点集群或任何类型的故障。

我们编写了一个使用 [Elle](https://github.com/jepsen-io/elle) 验证事务隔离的单个列表追加工作负载。这个工作负载在列表上执行事务，每个列表由唯一的整数 ID 标识。每个事务由对这些列表的读取和/或追加唯一整数组成。测试中的每个工作线程 [打开一个单独的 DocumentStore](https://github.com/jepsen-io/ravendb/blob/8315d6053bf022203a24db74812d2fdfe89b56b7/src/jepsen/ravendb/client.clj#L26-L31C15) 连接到同一节点。每个事务创建一个新的 [session](https://github.com/jepsen-io/ravendb/blob/8315d6053bf022203a24db74812d2fdfe89b56b7/src/jepsen/ravendb/client.clj#L54-L62)，执行 [读取和/或追加](https://github.com/jepsen-io/ravendb/blob/8315d6053bf022203a24db74812d2fdfe89b56b7/src/jepsen/ravendb/append.clj#L25-L41)，然后调用 [`session.saveChanges()`](https://github.com/jepsen-io/ravendb/blob/8315d6053bf022203a24db74812d2fdfe89b56b7/src/jepsen/ravendb/client.clj#L90) 进行提交。读取被编码为对 `session.load(java.util.Map, id)` 的 [单个调用](https://github.com/jepsen-io/ravendb/blob/8315d6053bf022203a24db74812d2fdfe89b56b7/src/jepsen/ravendb/append.clj#L26-L28)。追加调用 `session.load` 读取当前值，将其整数元素添加到列表末尾，然后调用 [`session.store(map, key)`](https://github.com/jepsen-io/ravendb/blob/8315d6053bf022203a24db74812d2fdfe89b56b7/src/jepsen/ravendb/append.clj#L40)。

RavenDB 提供了一些用于调整事务安全性的旋钮：`transactionMode` 和 `optimisticConcurrency`。我们使用默认设置运行了我们的测试（单节点事务，无乐观并发），使用单节点事务和乐观并

# 结果

我们在所有三种事务模式中发现了令人惊讶的安全错误。

## 单节点事务丢失更新 (#17927)

RavenDB 默认使用 `transactionMode = SINGLE_NODE` 和 `optimisticConcurrency = false` 执行事务。人们可能会认为 `SINGLE_NODE` 事务在单节点集群上是安全的。然而，我们发现默认设置导致 RavenDB 在没有故障的单节点集群中不断丢失更新。

例如，在这个 [五秒测试运行](https://github.com/ravendb/ravendb/files/13794868/20231229T102201.960-0600.zip) 中，我们执行了 12,886 个事务，涉及 975 个键。其中有 81 个键表现出可证明的丢失更新。以下是涉及键 830 的两个已提交事务：

```
[[:r 830 [1 2]]
 [:append 824 11]
 [:append 807 14]
 [:append 830 3]]

[[:r 830 [1 2]]
 [:r 831 nil]
 [:r 831 nil]
 [:append 830 4]]}]}
```

这两个事务都读取了键 830 的值作为列表 `[1, 2]`。两者都继续向键 830 添加了一个值：第一个事务添加了 `3`，而第二个事务添加了 `4`。两者都没有看到对方的效果。在一个快照隔离系统中，*先提交者胜出* 规则要求这两个事务中的一个必须中止。然而，RavenDB 却允许了这两个事务的提交。这就是丢失更新异常的定义。

在一个孤立的事务系统中，只会对列表追加元素，对单个列表的每个观察版本必须是该列表的最长版本的前缀。然而，这次测试中有 454 个键违反了这个前缀属性，展示了 *不兼容的顺序*。例如，以下是对键 116 的所有读取：

| 3.01 | 1 | [1 2] |
| --- | --- | --- |
| 3.01 | 1 | [1 2] |
| 3.01 | 0 | [1 2] |
| 3.01 | 1 | [1 2 3] |
| 3.01 | 0 | [1 2 4] |
| 3.01 | 1 | [1 2 4 5] |
| 3.01 | 0 | [1 2 4] |
| 3.02 | 1 | [1 2 4 6] |
| 3.02 | 1 | [1 2 4 6] |
| 3.02 | 1 | [1 2 4 6 9] |
| 3.02 | 1 | [1 2 4 6 9 10] |
| 3.02 | 1 | [1 2 4 6 9 10 11 15] |
| 3.02 | 0 | [1 2 4 6 9 10 11 15] |

测试进行到第三秒多一点时，进程 1 观察到键 116 的状态为 `[1 2 3]`。然而，随后进程 0 立即读取到 `[1 2 4]`，并且值为 `3` 的写入再也没有出现过。接着进程 1 观察到 `[1 2 4 5]`。这个写入 `5` 被 `6` 替换后再也没有被看到过。

我们的丢失更新检查器是保守的：只有当两个事务读取同一键的相同版本 *并且* 都对其进行写入时，它才会推断出异常。然而，我们的 `append` 操作是通过读取值，然后写回更改来执行的。这意味着这些事务包含对检查器来说实际上是不可见的读取。这些不兼容的顺序情况很可能也代表了丢失更新。

总计，这次测试中有 975 个键中的 481 个展示出了丢失更新或不兼容的顺序。这些现象是被串行化、快照隔离和可重复读所禁止的。我们已将此报告为 RavenDB 的问题追踪器中的 [issue 17927](https://github.com/ravendb/ravendb/issues/17927)。

## 乐观并发下的断裂读（#17929）

启用了*[乐观并发](https://ravendb.net/docs/article-page/6.0/csharp/client-api/session/configuration/how-to-enable-optimistic-concurrency)*功能后，RavenDB 承诺“在客户端接收并修改文档后，当服务器端修改了文档时，将生成并发异常（并中止当前事务中的所有修改）”。在单节点运行时，该设置似乎确实可以防止丢失更新。但是，它允许出现断裂读取，以及各种 G-Single、G-nonadjacent 和 G2-item 的变体。同样，这些异常发生在健康的单节点系统中。

例如，考虑一下这个[十秒钟的测试运行](https://s3.amazonaws.com/jepsen.io/analyses/ravendb-6.0.2/20231229T140050.249-0600.zip)，其中每个事务都启用了乐观并发。我们的检查器发现了数百个类似这样的异常：

在这个图表中，顶部事务*T*[1]向键 271 追加了`3`，然后向键 279 追加了`2`。底部事务*T*[2]读取了键 271，但什么也没找到，然后向键 276 追加了 6，并最终读取了键 279 的值为`[2]`。因为*T*[2]未能观察*T*[1]对键 271 的追加，我们有一个读-写反依赖，表示为`rw`。因为*T*[2]观察到*T*[1]对键 279 的追加，我们有一个写-读依赖，表示为`wr`。简而言之，*T*[2]观察到了*T*[1]的一些影响，但不是全部。

这个异常被称为*断裂读取*，在读取原子、更新原子、因果、前缀、并行快照隔离、快照隔离、可重复读和可串行的情况下是被禁止的。RavenDB 的[事务常见问题](https://ravendb.net/docs/article-page/6.0/csharp/client-api/faq/transaction-support)承诺了快照隔离：“即使您访问多个文档，您也将得到它们在请求开始时的所有状态。”在所有 Jepsen 熟悉的快照隔离数据库中，快照跨越多个读取。而在 RavenDB 中，似乎每个读取都可以观察到不同的状态。我们已经将此问题报告给了 RavenDB 的[问题编号 #17929](https://github.com/ravendb/ravendb/issues/17929)。

## 使用集群范围事务的断裂读取（#17928）

集群范围的事务[应该是可串行化的](https://ravendb.net/docs/article-page/6.0/csharp/server/clustering/cluster-transactions)。然而，我们发现，即使是健康的单节点集群，每个事务都使用了`CLUSTER_WIDE`模式，也经常出现了断裂读取，以及 G-single、G-nonadjacent、G2-item 等情况。考虑一下这个[五秒钟的测试运行](https://s3.amazonaws.com/jepsen.io/analyses/ravendb-6.0.2/20231229T145350.284-0600.zip)，其中包含数百个可串行性违规。这是其中一个异常：

在这里，底部事务 *T*[2] 向键 146 添加了`6`，向键 149 添加了`1`。顶部事务 *T*[1] 未观察到 *T*[2] 对键 149 的添加，但观察到了其对键 146 的添加。这是另一个破碎读取的例子。与以前一样，这种行为似乎被 RavenDB 的文档所禁止，以及所有高于读取原子性的一致性模型。

我们已将此报告给 RavenDB 作为[问题＃17928](https://github.com/ravendb/ravendb/issues/17928)。

| 17927 | 单节点事务丢失更新 | None | 未解决 |
| --- | --- | --- | --- |
| 17929 | 乐观并发下的破碎读取 | None | 未解决 |
| 17928 | 带有集群范围事务的破碎读取 | None | 未解决 |

# 讨论

RavenDB 各种声称提供“完全 ACID”事务、串行化，或者至少是快照隔离。所有这些声明似乎都是错误的。RavenDB 6.0.2 的默认设置允许丢失更新。即使是集群范围事务也出现了破碎读取：这是快照隔离下严重禁止的异常情况，也是几个较弱模型中的一种。这些行为甚至在健康的、单节点、单片系统中也会发生，在这些系统中所有访问都通过主键进行。

RavenDB 的最强安全设置违反了读取原子性。因此它无法满足更新原子性、因果关系、前缀、并行快照隔离、快照隔离、可重复读、或者可序列化。RavenDB 可能提供读取提交或者单调原子视图，但是没有进行更严格的测试，Jepsen 不愿做出这种断言。

鉴于 RavenDB 一再强调安全性，其弱默认行为令人惊讶。正如首席执行官 Oren Eini 在关于 MongoDB 的事务安全设置的[评论](https://www.youtube.com/watch?v=5ZXBR3croMA&t=850s)中所述：

> 默认值很重要。它们相当重要。为什么？因为如果你选择了不好的值，你肯定会在基准性能测试中得到一些很好的数字。但是你在生产环境中会遇到一些[安全性]问题。然后有这样一个经典的回答：“哦，你应该读过文档并使用适当的配置。”

人们不禁要问：如果 ACID 特性对于 RavenDB 的用户如此重要，为什么默认设置允许丢失更新，即使在单键操作上也是如此？用户是否意识到他们的更新可能会被静默丢弃？有多少人会小心使用集群范围事务，以避免丢失更新违反安全性？他们是否知道即使是集群范围事务也允许破碎读取？

此报告是对 RavenDB 行为的一次粗略调查——这绝不是全面的。我们始终建议 Jepsen 对安全验证采取实验性方法：我们可以证明存在错误，但不能证明不存在错误。RavenDB 中可能还存在其他异常。

## RavenDB 是否具有事务？

RavenDB 的[集群事务文档](https://ravendb.net/docs/article-page/6.0/csharp/client-api/session/cluster-transaction/overview)的第一句话显得非常清楚：

> 会话代表一个单一的业务事务。

RavenDB 的[会话文档](https://ravendb.net/docs/article-page/6.0/csharp/client-api/session/what-is-a-session-and-how-does-it-work)的第一句话也强调：

> 从文档存储获取的会话是表示特定数据库上的单个业务事务的工作单元。

…接着说：

> 发送到`SaveChanges()`的批处理操作将在事务中完成。换句话说，要么所有更改都作为单一原子事务保存，要么都不保存。因此，一旦`SaveChanges`成功返回，就保证所有更改都已持久保存到数据库中。

RavenDB 会话明显不打算像典型数据库中的会话那样工作，后者（粗略地说）[与客户端连接一一对应](https://www.postgresql.org/docs/current/tutorial-arch.html)。它们带有默认限制[30 个网络请求](https://ravendb.net/docs/article-page/5.4/java/client-api/session/what-is-a-session-and-how-does-it-work#remarks)；典型的数据库会话是无限制的。它们缓冲写入；Jepsen 不知道有任何其他数据库的会话会这样做。它们缓存读取；大多数会话不这样做。它们包含像防止丢失更新这样的并发控制机制；Jepsen 不知道有任何其他数据库在会话级别这样做。这些都是大多数数据库会称为事务的特点。

RavenDB 的文章[NoSQL 中的 ACID 事务？RavenDB vs MongoDB](https://ravendb.net/articles/acid-transactions-in-nosql-ravendb-vs-mongodb)强调：RavenDB 已经支持了十多年的任何组合的数据库操作的 ACID 事务。看起来 RavenDB 会话的用途就是这个角色！

然而，在[对问题 17927 的回应](https://github.com/ravendb/ravendb/issues/17927#issuecomment-1872912239)中，Eini（又名 Ayende Rahien）解释说，会话实际上*不是*事务：

> 重要的是，RavenDB 并不试图在整个会话上提供事务语义，而是在各个请求上提供事务。

在[对问题#17928 的回应](https://github.com/ravendb/ravendb/issues/17928#issuecomment-1872916841)中，Eini 确认：

> RavenDB 中的事务是一个请求 - 因此上述的 TX1 和 TX2 实际上并不是单一事务，相反，它们每个代表了 3 个独立的事务。

这是一个引人注目的观点：事务的目的通常是为了在多个请求之间提供隔离。此外，RavenDB 的乐观并发和集群范围的事务机制明显旨在提供从会话的读取到其写入的事务隔离。此外，Eini [直接将 RavenDB 会话与 MongoDB 事务进行了比较](https://www.youtube.com/watch?v=5ZXBR3croMA&t=23m40s)（后者提供典型的交互式事务语义），并声称与 MongoDB 不同，RavenDB 会话实际上满足了快照隔离。然而根据 Eini 的评论，RavenDB *根本没有交互式事务*。

反复宣传“ACID 事务”涵盖“任意组合的数据库操作”，告诉用户“会话代表一个单一的业务事务”，将 RavenDB 会话与其他数据库中的交互式事务进行比较，提供跨越整个会话范围的并发控制机制，并最终期望用户意识到会话根本不是事务——事务实际上仅限于单个 HTTP 请求——这是令人难以置信的。

Jepsen 努力在其营销和文档的背景下评估数据库。尽管 RavenDB 的首席执行官 [现在表示](https://github.com/ravendb/ravendb/issues/17928#issuecomment-1874064897) “我们不支持跨越多个单独 HTTP 请求的事务”，但 RavenDB 的文档和营销给人的印象是会话应该是一个事务。Jepsen 已经与几位软件工程师就对这些声明的解释进行了咨询，并相信典型的数据库用户会得出相同的结论：RavenDB 会话就是事务。我们在本报告中继续这种解释。

## 建议

RavenDB 用户应该意识到 RavenDB 事务在任何有意义的情况下都不是 ACID 的。即使在单节点、单分片部署中也是如此。默认情况下允许丢失更新：您应该预期一些写操作将被静默丢弃。最强的安全设置允许断裂读取：您可能会观察到另一个事务的一些效果，但不是全部。您可能会看到自己“写入”另一个事务的中间。RavenDB 宣传的两个隔离级别——快照隔离和串行化——似乎无法实现。

那些假设 RavenDB 提供交互式 ACID 事务（甚至是快照隔离）的用户应该仔细重新评估他们的事务，以确保在存在这些异常情况下它们是安全的。考虑编写简单的测试来验证应用程序不变量在并发执行下是否得以保持：本报告中的问题很容易再现。

Jepsen 建议 RavenDB 从其营销材料和文档中删除“ACID”、“Serializable”和“Snapshot Isolation”的声明。相反，RavenDB 应该对安全属性进行具体、准确和内部一致的声明。例如，RavenDB 可以说“事务默认提供读已提交，加上事务范围内的内部一致性：一旦事务读取一个键，随后对该键的读取和写入都会观察到最初读取的状态，以及该特定事务的写入效果。事务默认允许丢失更新。启用集群范围的事务可以防止丢失更新，但仍允许分段读取”，等等。

RavenDB 的文档非常令人困惑。它反复声称提供[ACID 事务](https://ravendb.net/why-ravendb/acid-transactions)，这意味着可串行化。有具体的声明称 RavenDB 确保[可串行化](https://ravendb.net/docs/article-page/6.0/csharp/server/clustering/cluster-transactions#case-1-multiple-concurrent-cluster-transactions)或[快照隔离](https://www.youtube.com/watch?v=5ZXBR3croMA)。然而，文档还说 RavenDB 的数据库层是基于 Last Write Wins 的[AP 系统](https://ravendb.net/learn/inside-ravendb-book/reader/4.0/6-ravendb-clusters)，营销材料声称孤立的节点可以[独立运行](https://ravendb.net/why-ravendb/high-availability)。这是不可能的：完全可用的系统[无法提供](https://jepsen.io/consistency)可串行化或快照隔离。

有些系统（如[Riak](https://riak.com/index.html)和[Cassandra](https://cassandra.apache.org/_/index.html)）允许客户端执行完全可用但一致性较弱的操作，或者具有更强保证的大多数可用操作，如线性一致性。如果 RavenDB 打算构建支持两种模式的系统，他们应该在整个营销和文档中清楚地区分这些模式。它们具有完全不同的可用性、延迟和安全特性。反复声称 RavenDB 提供 ACID “[而不牺牲性能](https://ravendb.net/why-ravendb/acid-transactions)”是[可证明不可能的](https://lamport.azurewebsites.net/pubs/lower-bound.pdf)，应该重新撰写以清楚解释所涉及的权衡。

ACID 事务对于 RavenDB 来说显然非常重要。因此，令人担忧的是，RavenDB 的文档和 GitHub 评论在“事务”的定义上存在根本性分歧。根据一种解释，RavenDB 提供了交互式事务，由会话 API 表示，提供相对较弱的隔离性——显然不是 ACID。根据另一种解释，RavenDB 根本没有交互式事务。相反，它提供了一种微事务，例如在单个网络请求中写入多个文档。在这种情况下，会话提供了在微事务之间延伸的各种弱一致性约束。

为了解决这种混乱，RavenDB 应该选择一个“事务”的单一定义，并坚持下去。事务和会话之间的等价性或差异应该得到清晰解释，并且这些术语在市场营销和文档中应该一致使用。RavenDB 应该提供关于每个单元边界的指导：在单个事务中何时执行多次`load`调用？`store`呢？一个单一事务能否同时包含`load`和`store`？事务和会话的一致性属性应该得到明确定义。事务是否可串行化？会话是否确保单调原子视图？会话何时排除丢失更新，何时允许它发生？最重要的是，如果事实上会话根本不是事务，请不要告诉用户它们“代表单个业务事务”。

最后，如果 RavenDB 事务真的只意味着涵盖一个单一网络请求，考虑完全使用不同的术语，并避免与确实具有交互式事务的数据库进行比较。一些数据库称这些为“微型”或“微”事务，这提供了它们有限范围的明显提示。

## 未来的工作

这项工作仅评估了没有故障的单节点 RavenDB 集群。未来的研究可以扩展到跨多个节点的测试，并引入网络、进程和磁盘故障。我们仅处理了键值操作，并没有评估 RavenDB 的二级索引。这些索引被描述为最终一致，这就引发了有关谓词读取完整性的问题。RavenDB 还提供了使用 Javascript 或内置补丁操作库的[服务器端事务](https://ravendb.net/docs/article-page/6.0/csharp/client-api/operations/patching/single-document)。这些可能具有与我们在本报告中使用的交互式事务不同的安全特性。最后，跨 shard 事务是一个众所周知的难题，值得仔细测试。

*Jepsen 感谢[Irene Kannyo](https://www.irenekannyo.com/)对本文的宝贵编辑支持。同时感谢 C. Scott Andreas、Taber Bain、Silvia Botros、Coda Hale、Ben Linsay、Kelly Shortridge、Nathan Taylor、Zach Tellman 和 Leif Walsh 对本手稿早期版本的评论。本工作是根据[Jepsen 的道德政策](https://jepsen.io/ethics)独立完成的，没有获得任何补偿。*
