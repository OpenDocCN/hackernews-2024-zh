<!--yml

分类：未分类

日期：2024-05-29 12:34:23

-->

# 当Postgres不够用：PostgreSQL与分布式DBMS性能评估 | Evgeniy Ivanov | YDB.tech博客

> 来源：[https://blog.ydb.tech/when-postgres-is-not-enough-performance-evaluation-of-postgresql-vs-distributed-dbmss-23bf39db2d31?gi=bdf24da23039](https://blog.ydb.tech/when-postgres-is-not-enough-performance-evaluation-of-postgresql-vs-distributed-dbmss-23bf39db2d31?gi=bdf24da23039)

# 当Postgres不够用：PostgreSQL与分布式DBMS的性能评估

**免责声明：**此篇文章由YDB开发人员[Evgenii Ivanov](https://twitter.com/eivanov89)撰写。然而，所呈现的研究成果源于我们的共同努力和与[Evgeny Efimkin](https://linkedin.com/in/evgeny-efimkin-4061a893)的紧密合作，他是PostgreSQL领域的专家，但并不在YDB工作。

众所周知，PostgreSQL既高效又功能丰富。充满活力的PostgreSQL社区已经重塑了数据库管理系统（DBMS）行业的格局，定义了现代DBMS应为应用程序开发者和用户提供的功能。同时，众所周知，PostgreSQL只能垂直扩展，因此其性能受限于单个服务器的能力。

许多富有洞见的出版物比较单体和分布式数据库架构（例如，阅读这篇关于该主题的[文章](https://franckpachot.medium.com/monolithic-vs-distributed-sql-f07af959e1a4)）。然而，这些出版物通常是纯理论性的，缺乏具体的性能数据。相比之下，本文提出了基于[TPC-C](https://www.tpc.org/tpcc/)的经验研究，这是行业标准的在线事务处理（OLTP）基准测试。

本文的基准测试方法非常直接。我们使用了三台服务器，配备了128个CPU核心，512 GiB RAM和4个NVMe磁盘，设置了一个数据库，设计用于抵抗单个服务器故障并保持至少1 TiB的数据集大小。然后，我们执行了12小时的TPC-C基准测试。此外，我们还展示了为什么即使这样的持续时间可能仍然不够。

鉴于不存在通用且绝对可靠的PostgreSQL配置，我们探索了各种选项。从性能最高但可靠性最低的配置开始，没有预写日志（WAL）和复制，逐步过渡到性能较低但更可靠的设置，包括两个同步副本。在整个过程中，我们在负载下监控并分享PostgreSQL的硬件指标，识别瓶颈并对系统进行微调。最终，我们将PostgreSQL的TPC-C结果与众所周知的开源分布式数据库——[CockroachDB](https://www.cockroachlabs.com/)和[YDB](https://ydb.tech/)进行了比较。

由于本文内容广泛，我们将先介绍结果，再深入讨论技术细节。首先，让我们简要回顾一下TPC-C是什么。

# TPC-C

TPC-C是“评估OLTP性能的唯一客观比较”，— CockroachDB [引用](https://www.cockroachlabs.com/docs/v23.1/performance#tpc-c)。

“TPC-C已经是几十年来最受欢迎的OLTP基准之一，通常用于确定系统的成本或功率效率。它对于确定OLTP数据库的日常性能非常有用”，— PingCAP [引用](https://www.pingcap.com/blog/why-benchmarking-distributed-databases-is-so-hard/)。

[YugabyteDB](https://www.yugabyte.com/)还表示，TPC-C是评估OLTP系统性能的最客观的性能测量方法，但我们找不到确切的引用。

我们先前的[文章](/ydb-meets-tpc-c-distributed-transactions-performance-now-revealed-42f1ed44bd73)还有一个“TPC-C 101”部分以及我们为YDB实现的TPC-C描述。您可以在这篇[文章](/how-we-switched-to-java-21-virtual-threads-and-got-deadlock-in-tpc-c-for-postgresql-cca2fe08d70b)中找到更多关于我们基于Benchbase的PostgreSQL TPC-C实现的信息。自那时以来，我们的PostgreSQL实现已从[c3p0](https://www.mchange.com/projects/c3p0/)迁移到[HikariCP](https://github.com/brettwooldridge/HikariCP)。以下我们简要回顾了TPC-C指标，这应该足以理解结果。

TPC-C基准的目标是在给定的延迟限制下（例如，新订单交易的90%必须低于5秒），每分钟完成最高数量的新订单交易（称为*tpmC*）。该基准的设计方式是，每个仓库的最大tpmC数量是有限的：要获得更多的tpmC，您必须服务更多的仓库，每个仓库大约有100 MiB的数据。为了简化，我们在本文中可以交替使用仓库数量和tpmC的术语：这意味着实现的tpmC非常接近指定仓库数量的最大可能tpmC。

此外，请注意，TPC-C是一种标准，描述了基准测试，没有任何官方实施。我们为YDB和PostgreSQL都基于[Benchbase](https://github.com/cmu-db/benchbase)实现的TPC-C，并且CockroachDB的实现也都未被[TPC](https://www.tpc.org/)组织官方批准，但所有实现都紧密遵循TPC-C v5.11.0 [规范](https://www.tpc.org/tpc_documents_current_versions/current_specifications5.asp)。这意味着本文提供的结果不是官方认可的TPC结果，不能与TPC网站上发布的其他TPC-C测试结果进行比较。

# 设置

我们在一个裸机集群上进行实验，共有三台机器。每台机器的配置如下：

+   两个32核处理器Intel Xeon Gold 6338 2.00GHz，启用超线程（128逻辑核）。

+   512 GiB内存。

+   四个NVMe Intel-SSDPE2KE032T80硬盘。

+   50 Gbps网络。

+   在Postgres情况下，使用大页，而在分布式DBMS情况下，使用透明大页。

+   Ubuntu 20.04.3 LTS。

+   Linux内核版本为5.4.161–26.3。

# Postgres 设置

使用的PostgreSQL版本是16.2（Ubuntu 16.2–1.pgdg20.04+1）。对于最终的Postgres设置，我们配置了两个包含两个NVMe磁盘的RAID 0设备。一个RAID 0用于WAL，另一个用于数据。此外，我们将Linux配置如下：

```
# we have a write intensive workload:
# background ratio is decreased, dirty ratio is increased
sudo sysctl -w vm.dirty_background_ratio=5
sudo sysctl -w vm.dirty_ratio=40
```

```
# Based on our config, see [https://www.postgresql.org/docs/current/kernel-resources.html#LINUX-HUGE-PAGES](https://www.postgresql.org/docs/current/kernel-resources.html#LINUX-HUGE-PAGES)
# sudo -u postgres /usr/lib/postgresql/16/bin/postgres \
#                  -D /opt/postgres/postgres/16/main \
#                  -c config_file=/etc/postgresql/16/main/postgresql.conf \
#                  -C shared_memory_size_in_huge_pages
sudo sysctl -w vm.nr_hugepages=67129# disable overcommit
sudo sysctl -w vm.overcommit_memory=2# max GHz
sudo tuned-adm profile throughput-performance
```

RAID 0 设置：

```
sudo mdadm --create /dev/md/md_db  --assume-clean -l0 --raid-devices=2 /dev/nvme1n1p2 /dev/nvme0n1p2
sudo mdadm --create /dev/md/md_wal  --assume-clean -l0 --raid-devices=2 /dev/nvme2n1p2 /dev/nvme3n1p2
```

我们使用以下命令格式化RAID设备：

```
mkfs.ext4 -F -m 0 -E lazy_itable_init=0,lazy_journal_init=0,discard
```

然后使用`noatime`选项挂载它。

[这里](https://github.com/ydb-platform/postgres_vs_distributed/tree/main/postgres)您可以找到完整的配置文件。有5个TPC-C客户端，每个客户端使用60个会话进行连接。

# YDB 设置

我们使用的是YDB [24.1.10](https://storage.yandexcloud.net/binaries.ydb.tech/release/24.1.10/ydbd-24.1.10-linux-amd64.tar.gz)（测试版本）。每台机器有1个存储节点（32核心任务集）和6个计算节点（每个节点各自在其16核心任务集中）。你可以在[这里](https://github.com/ydb-platform/postgres_vs_distributed/tree/main/ydb)找到完整的配置，该配置接近默认配置。总共，有高达3000个来自3个TPC-C客户端的YDB连接。

# CockroachDB 设置

使用的CockroachDB版本是23.1.10。由于这个[问题](https://github.com/cockroachdb/cockroach/issues/119924)，我们未能在更新的23.2.2版本上运行TPC-C。尽管在23.1.10版本中问题仍然存在，但CockroachDB团队表示，此版本中问题是由于集群超载引起的。

我们在每个NVMe上运行一个CockroachDB实例，在32核心任务集中，有37.5 GiB缓存和37.5 GiB SQL内存。我们应用了[此](https://www.cockroachlabs.com/docs/v23.1/performance-benchmarking-with-tpcc-medium#step-3-configure-the-cluster)推荐配置。[这里](https://github.com/ydb-platform/postgres_vs_distributed/blob/main/cockroach/cluster_config.py)是我们的配置文件，由我们的[安装](https://github.com/ydb-platform/benchhelpers/tree/main/db_installers/cockroach/)脚本使用。请注意，我们不使用分区，这是一种企业功能。我们避免这样做是因为在TPC-C中有时无法像分区数据那样分区。

# 结果

请注意，TPC-C基准旨在在事务的指定事务延迟边界内实现尽可能高的tpmC得分。如演示，PostgreSQL通过在18K仓库上运行并获得92.37%的效率，比运行16K仓库且效率为98.57%的YDB多达1.05倍。同时，与其分布式对手相比，PostgreSQL表现出显著更高的延迟，尽管它仍保持在TPC-C指定的限制内。让我们更详细地看一下新订单延迟（本文末提供了附加延迟详细信息）：

你可能知道吞吐量（tpmC）和延迟之间通常存在折衷：一般来说，更高的tpmC会导致增加的延迟。使用16K个仓库，YDB实现了其可能的最高tpmC，并且交易延迟保持在TPC-C要求范围内，但在某些生产场景中，这些延迟可能被认为太高了。因此，我们还提供了使用13K个仓库时YDB的效率为99.24%的结果，其中延迟显著降低，并且与CockroachDB运行12K个仓库的延迟相当，同时实现了比CockroachDB略高的tpmC。

下面是TPC-C客户端在基准执行期间报告的新订单延迟（以秒为单位）：

这些延迟并不是“稳定的”。每个高峰对应于一个检查点的开始，这似乎会使会话在等待`IPC: SyncRep`时挂起。尽管我们的努力，我们未能解决这个问题。对于PostgreSQL，在大约10K个仓库以下，我们观察到明显更好的延迟。在下一节中，我们描述了我们对PostgreSQL调优的广泛调查。

比较效率，PostgreSQL仅利用领导机器上约15%的可用CPU。正如我们理解的（并在下一节中展示），复制是唯一的瓶颈。这表明PostgreSQL在拥有更少CPU核心的服务器上也可以达到可比较的结果。相比之下，分布式数据库的CPU利用效率较低，CPU使用率接近100%。然而，它们也展示了更好的延迟，可以视为CPU延迟的折衷。

展示的结果显示了PostgreSQL的异常效率。然而，我们预计PostgreSQL会基于分布式数据库管理系统主要在集群中包含大量更多的机器时表现出更为显著的优势。从某种意义上说，我们已经驳斥了这一谬论。

但是，请注意，本文讨论的这两个分布式数据库管理系统只需通过向集群中添加更多机器来处理增加的负载，即可在普通硬件上实现超过1M tpmC。下面，我们展示了YDB的可伸缩性测试结果。我们逐步添加了更多机器（9、18和36台），同时保持相同的配置。结果来自测试版本（即将推出的YDB v24），尽管我们预计在YDB v23分支中也会有类似的表现。虽然这些结果可能不是最佳的，但它们清楚地说明了这一点：

我们上面概述了DBMS应该对单个服务器故障具有容错能力。根据配置不同，它可能等同于数据中心的故障。此外，在机器故障的情况下，客户端负载应立即重定向到剩余的服务器，以避免操作停机时间。对于某些使用场景，这可能代表一种特别严格的故障模型。接下来，我们介绍了实现前所未有的高性能的‘不容错’PostgreSQL配置。

# 不容错，但极具性能的PostgreSQL配置

配置PostgreSQL实质上已成为一个专门的工程领域。配置的主要挑战在于，你永远不知道你的配置是否足够好，直到你达到硬件饱和。具体来说，关键是确定是否可以通过额外的PostgreSQL或Linux调整来实现进一步的增强。另一个复杂性在于庞大的配置选项数量，从I/O和并发到SQL规划和执行都有涉及。

为了简化我们的配置探索，我们从未记录表和无复制（“NoWAL”配置）开始。这种方法通过消除WAL开销和潜在的网络瓶颈来最大化提交速度。这种配置非常适合评估与数据库数据相关的SQL执行和I/O操作的效率。

接下来，我们重新引入了WAL，但调整了它以优化随时间的I/O分布，创建不频繁的检查点并扩大了WAL的大小。这种‘HugeWAL’设置虽然不适合生产环境，因为从崩溃中恢复可能需要数十分钟，但作为测试的宝贵配置。

最后，我们引入了两个同步副本（‘R1’），允许在主服务器故障时切换到备用服务器。在这三种配置中，我们使用了三个NVMe硬盘进行数据库的RAID 0配置，以及一个NVMe硬盘用于WAL存储。让我们来看看结果：

在我们的测试中，我们观察到显著的未使用CPU和网络资源，同时磁盘还有额外的写入容量。这些发现非常启发人：

1.  ‘NoWAL’配置显示出由于PostgreSQL SQL执行或RAID 0设备处理数据库数据的延迟而存在潜在的限制。CPU使用率仅为40核，PostgreSQL写入速度为1.5–2 GB/s（200–250K次写入/秒），读取速度为2.3 GB/s（275K次读取，读取单元为8 KiB）。

1.  与‘NoWAL’比较表明，观察到的性能下降是利用WAL的代价。PostgreSQL每秒写入800 MB的WAL数据（大约6.7K次写入/秒，写入单元为128 KiB，这是我们NVMe硬盘的硬件限制）。

1.  ‘R1’设置中显著的性能下降完全归因于复制开销，而不是WAL或SQL执行。

实现 35K 仓库（450K tpmC）是非常优秀的，然而‘NoWAL’设置缺乏可靠性。使用 ‘HugeWAL’ 达到 29K 仓库同样令人印象深刻，这也是我们从“R1”配置中所期望的。这就是为什么我们开始调查慢复制的原因。

我们的初步发现是，副本写入的 WAL 比领导节点多 1.5–2 倍，目前我们还缺乏一个精确的解释。具体而言，领导者以 250–450 MB/s 的速度写入（平均每秒 4K 次写入），而每个副本写入的速度为 500–800 MB/s。值得注意的是，在‘HugeWAL’配置下，写入速度恰好为 800 MB/s，性能令人满意。

“WAL 问题” 在副本上为我们提供了一个线索，让我们将调查重点放在这个方面。我们使用了 [procstat](https://github.com/FritsHoogland/procstat) 工具来获取实时磁盘行为数据。我们必须承认，procstat 是一个非常方便和用户友好的工具。以下是从 procstat 获取的 WAL 磁盘信息的屏幕截图。”

领导者：

复制品：

很容易看到 PostgreSQL 何时写入检查点。但真正有趣的是，不仅跟随者写入更多，而且写入的延迟比领导者高 10–30 倍（数毫秒 vs. 数百微秒）。由于这个原因，我们做了一个假设，即副本的 WAL 是瓶颈。

为了缓解这个问题，我们尝试了以下调整：

```
bgwriter_delay = 20
bgwriter_lru_maxpages = 4192
```

```
wal_buffers = 256MB
wal_writer_delay = 400commit_delay = 200checkpoint_completion_target = 0.99
checkpoint_flush_after = 2MB
```

这并没有帮助，因此我们选择了不同的磁盘配置方法：在数据库中使用两个（而不是三个）RAID 0 磁盘，并在 WAL 中使用另外两个 RAID 0 磁盘。此外，为了加快提交速度，我们将 `synchronous_commit` 设置从 `on` 更改为 `remote_write`。这一变更意味着在操作系统崩溃时，提交在备用节点上不再保证持久性。然而，它也被认为能够减少领导者的提交延迟并增强整体系统性能。我们认为这种妥协是值得的。

这一修订后的设置和配置使我们成功地在 2 小时内完成了 TPC-C 基准测试，实现了 20K 仓库。然而，将测试延长到 12 小时后显现出了一些限制：系统在副本上的 WAL 空间耗尽后运行了 5 小时。在接下来的 3 小时内，副本在同时清理和应用其 WAL 方面遇到了困难。将仓库数量调整至 17K 使得系统能够在整个 12 小时的时间内保持运行。有趣的是，基准测试后，回放延迟几乎达到了 3 小时，与应用所需的时间相匹配：

```
tpcc=# SELECT pid,application_name,client_addr,client_hostname,state,sync_state,replay_lag FROM pg_stat_replication;
   pid   | application_name |           client_addr            | client_hostname |   state   | sync_state |   replay_lag
---------+------------------+----------------------------------+-----------------+-----------+------------+-----------------
 1250777 | s1               | 2a02:6b8:c34:14:0:5a59:eb1f:2ca6 |                 | streaming | sync       | 02:56:10.114747
 1250824 | s2               | 2a02:6b8:c34:14:0:5a59:eb1f:2956 |                 | streaming | sync       | 02:52:11.340846
```

这表明复制品不起到有效的热备份作用。在主服务器故障的情况下，复制品需要大量时间来应用累积的WAL，可能导致长达几小时的停机时间。此外，如果滞后持续增加，有可能耗尽分配给WAL的磁盘空间。这种情况将阻止主要服务器处理客户请求的能力，导致额外的停机时间，根据我们的经验，这可能会延长几个小时。

我们调整了`synchronous_commit`设置为`remote_apply`来解决这些问题。在这种最终配置下，PostgreSQL能够支持18K个仓库。现在，`replay_lag`已经变得可以忽略不计，这使得跟随者可以作为热备份。在这种配置下，我们观察到领导者的以下资源消耗情况：

+   领导者消耗大约20个CPU核心。

+   领导者以每秒400 MB的速度写入WAL（6K次写入/秒），平均每秒600 MB的数据库数据（80K次写入/秒）。

+   领导者以700 MB/s的速度从数据库读取数据。

+   领导者的峰值链路利用率约为9 Gbps。

每个跟随者的资源消耗情况：

+   每个跟随者大约写入800 MB/s的WAL数据和约500 MB/s的数据库数据。

+   每个跟随者的峰值链路利用率为4 Gbps。

# 结论

存在无限数量的硬件设置、PostgreSQL配置和不同的生产工作负载。在本文中，我们探讨了在特定硬件设置和特定TPC-C工作负载下的PostgreSQL限制。如果您关注我们的[博客](https://blog.ydb.tech/)，您可能还记得这是我们通常用于性能研究的硬件，像许多DBMS开发人员一样，我们喜欢TPC-C。

我们必须承认，PostgreSQL确实非常高效，甚至超出了我们的预期。在“容错不允许”的PostgreSQL设置下，展示了出色的性能，使其成为在不需要应对服务器（或数据中心）故障且可以接受停机的情况下的出色选择。这就像说天空是蓝色的，水是湿的一样显而易见。

然而，为了具有容错能力并可靠地存储您的数据，您应该有多台运行您的DBMS的机器。如果对于您的业务，“失败不是一个选择”，这是必须的。我们惊讶地发现，PostgreSQL复制导致了如此显著的瓶颈，并且对实现的tpmC产生了如此大的影响。此外，主要的延迟峰值可能在某些生产场景中造成挑战。我们相信可能有解决此问题的方案，我们还不知道，因此如果您知道任何有效的策略或者我们尚未尝试过的“魔法开关”，请在下面的评论中告诉我们。

如果 PostgreSQL 无法满足您的需求，我们提供一种解决方案。与普遍看法相反，我们的研究表明，即使在小型三机集群中，分布式数据库也能表现出色。此外，以 YDB 为例，我们展示了分布式数据库管理系统易于扩展，让您能够专注于应用开发，而非数据库管理的细节。

# 致谢

这项研究是我们与同事 [Evgeny Efimkin](https://linkedin.com/in/evgeny-efimkin-4061a893) 紧密合作的成果，他是 PostgreSQL 的专家。

特别感谢 [Andrey Borodin](https://twitter.com/x4mmmmmm)，他是 PostgreSQL 的活跃贡献者，也是 [Odyssey](https://github.com/yandex/odyssey)（一个可扩展的 PostgreSQL 连接池）的开发者。在我们的研究过程中，Andrey 给予了强大的支持。

[Frits Hoogland](https://twitter.com/fritshoogland) 给予了宝贵的反馈，并与我们分享了一个很棒的工具，[procstat](https://github.com/FritsHoogland/procstat)。

# 附录：事务延迟
