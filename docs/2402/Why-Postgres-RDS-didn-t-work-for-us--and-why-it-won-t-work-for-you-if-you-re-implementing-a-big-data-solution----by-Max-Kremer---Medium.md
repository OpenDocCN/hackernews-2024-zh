<!--yml

分类：未分类

日期：2024-05-27 14:33:29

-->

# 为什么对我们而言 Postgres RDS 不适用（以及如果您正在实施大数据解决方案，它为什么也不适用）| 作者 Max Kremer | Medium

> 来源：[https://medium.com/@mkremer_75412/why-postgres-rds-didnt-work-for-us-and-why-it-won-t-work-for-you-if-you-re-implementing-a-big-6c4fff5a8644](https://medium.com/@mkremer_75412/why-postgres-rds-didnt-work-for-us-and-why-it-won-t-work-for-you-if-you-re-implementing-a-big-6c4fff5a8644)

# 为什么对我们而言 Postgres RDS 不适用（以及如果您正在实施大数据解决方案，它为什么也不适用）

# 背景

我们正在构建一个通过统计归因模型来衡量营销效果的初创公司。为了使其工作，我们必须收集大量在线用户行为数据。每次点击，每次字段输入，每个操作。数据存储在 postgres 中，通过我们的用户界面，终端用户运行可以处理上述（有时非常大）数据集的仪表板和报告。需要注意的是，我们正在构建一个多租户 SAAS 应用程序，因此数据根据数百个租户进行摄取和存储。它需要可扩展性和性能。我们从 AWS RDS 托管的 PostgreSQL 开始。

# 对于 EBS 存在的大问题：吞吐量的不透明性

在 EBS 上几乎不可能准确测量吞吐量。AWS 提供的指标是延迟和 IOPS。这两者都不能提供吞吐量的衡量。当您存储一个大型时间序列表（例如 2000 万行），并且表在磁盘上的大小为 10GB，您希望扫描该表（无论出于何种原因——比如顺序扫描），那么您需要准确的吞吐量测量来理解从磁盘扫描数据将需要多长时间。

# 爆发和被限制的 IOPS —— 一个滑坡

像上面的查询会快速消耗 IO credits。我们想要更多的 IOPS，那么我们该怎么办？自然地，我们增加存储卷的大小，这样我们可以从较小的卷得到的每秒 IO 操作次数（IOPS）的基线增加到 10000 IOPS，而不是只有 3000。但随之而来的是备份成本的增加，因为您需要为整个卷付费，而不仅仅是使用的部分。成本增加，性能并没有得到很大的提升——问题就此恶化。

我们还不断提升实例的大小，以获取更多的内存用于缓存。如果可以将所有数据都放入内存中，数据库的速度会很快。此外，PG v10 还带来了更大的并行化的承诺，但遗憾的是由于 IO 限制，它给我们带来的好处非常有限。

更详细地描述了测量吞吐量的问题：

[https://www.datadoghq.com/blog/aws-ebs-latency-and-iops-the-surprising-truth/](https://www.datadoghq.com/blog/aws-ebs-latency-and-iops-the-surprising-truth/)

# Aurora

所以我们迁移到了Aurora。对于标准的postgresql，承诺的5-7倍性能提升很难忽视。然而，这些性能指标是基于pgbench（在postgresql的情况下）。如果您有OLTP系统并希望测量具有大量并行客户端的事务速度（或延迟），那么这是很好的。但对于大数据集和数据挖掘操作来说，这不是一个好的性能指标。

成本上升而不是下降。

此线程涵盖了IOPS和可计费IOPS [https://forums.aws.amazon.com/message.jspa?messageID=835303#835303](https://forums.aws.amazon.com/message.jspa?messageID=835303#835303)

再次最大的挫折是获取吞吐量的准确测量。简单地将扫描的数据量除以使用tack_io_timing花费的时间，我们发现大数据集的性能远远不及SSD速度，更接近于磁盘。

增加挫折的是与支持团队打交道的无尽时间。虽然支持人员总是非常知识渊博并且乐于帮助，但最终都归结为调整查询或调整数据库。再次回到吞吐量 — 我很乐意听到类似“考虑到您的工作负载，这项服务不是一个好的选择”。但最终我们自己找到了解决方法。

# 解决方案：自己动手做

对我们来说，解决方案相当简单：不要使用托管数据库服务，而是在EC2上自己构建基础设施。

我们使用相对低档的EC2实例来做到这一点（我们在RDS上运行了4XL xl和2xls）。

关键在于在主备复制物之间分离READ和WRITE工作负载，通过WAL-G（[https://github.com/wal-g/wal-g](https://github.com/wal-g/wal-g)）保持更新。

WRITE节点可能会很慢，它只是在摄取数据，所以EBS已经足够好了，但我们能够通过使用带有压缩的ZFS将EBS吞吐量**翻了一番**。

对于READ节点，我们有一个带有临时NVMe驱动器的设置，再次使用ZFS来提升性能。

结果不言自明：

$11K的账单降到了$2100（预计在九月，这将是我们完全摆脱任何RDS负担的第一个完整月）

原本需要数小时甚至超时的查询现在可以在**几秒钟**内完成。

我们能够使用unix实例上的命令行工具来测量实际的吞吐量（zpool iostat）

# 结论

当然，按下按钮并启动实例而无需担心备份和恢复是很容易的。但这是有成本的，在我们的情况下是货币和性能。如果您需要处理TB或GB级数据的真正速度，那么使用裸金属服务器，如果不能使用裸金属服务器，那么下一个最佳选择是带有NVMe驱动器的EC2。阅读[第二部分](/p/748c391fce51)，我将介绍如何在AWS上使用带有NVMe驱动器的实例自行构建Postgres集群的步骤。
