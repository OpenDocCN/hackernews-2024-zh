<!--yml
category: 未分类
date: 2024-05-29 12:34:23
-->

# When Postgres is not enough: performance evaluation of PostgreSQL vs. Distributed DBMSs | by Evgeniy Ivanov | YDB.tech blog

> 来源：[https://blog.ydb.tech/when-postgres-is-not-enough-performance-evaluation-of-postgresql-vs-distributed-dbmss-23bf39db2d31?gi=bdf24da23039](https://blog.ydb.tech/when-postgres-is-not-enough-performance-evaluation-of-postgresql-vs-distributed-dbmss-23bf39db2d31?gi=bdf24da23039)

# When Postgres is not enough: performance evaluation of PostgreSQL vs. Distributed DBMSs

**Disclaimer:** this post was written by a YDB developer, [Evgenii Ivanov](https://twitter.com/eivanov89). However, the research presented is the result of our joint effort and close collaboration with [Evgeny Efimkin](https://linkedin.com/in/evgeny-efimkin-4061a893), an expert in PostgreSQL who doesn’t work on YDB.

It is well known that PostgreSQL is both extremely efficient and feature-rich. The vibrant PostgreSQL community has reshaped the Database Management Systems (DBMS) industry landscape, defining what modern DBMS should offer for application developers and users. At the same time, it’s not a secret that PostgreSQL scales only vertically, thus its performance is limited by the capabilities of a single server.

Many insightful publications compare monolithic and distributed database architectures (for example, read this [post](https://franckpachot.medium.com/monolithic-vs-distributed-sql-f07af959e1a4) on the topic). However, often these publications are purely theoretical and lack specific performance numbers. In contrast, this post presents an empirical study based on [TPC-C](https://www.tpc.org/tpcc/), the industry-standard On-Line Transaction Processing (OLTP) benchmark.

The benchmarking approach in this post is straightforward. We use three servers equipped with 128 CPU cores, 512 GiB RAM, and 4 NVMe disks to set up a database designed to withstand a single server outage and maintain a dataset size of at least 1 TiB. We then execute the TPC-C benchmark for 12 hours. Also, we demonstrate why even this duration may not suffice.

Given that no universal, bulletproof PostgreSQL configuration exists, we explore various options. Starting with the most performant and least reliable configuration without Write Ahead Log (WAL) and replication, we gradually move towards less performant but more reliable setups, including two synchronous replicas. Throughout this process, we monitor and share PostgreSQL hardware metrics under load, identifying bottlenecks and fine-tuning the system. Ultimately, we compare PostgreSQL’s TPC-C results with those of well-known open-source distributed databases: [CockroachDB](https://www.cockroachlabs.com/) and [YDB](https://ydb.tech/).

Because of the extensive nature of this post, we begin with the results before delving into the geeky stuff. But first, let’s briefly recap what TPC-C is.

# TPC-C

TPC-C is “the only objective comparison for evaluating OLTP performance”, — CockroachDB [quote](https://www.cockroachlabs.com/docs/v23.1/performance#tpc-c).

“The TPC-C has been one of the more popular OLTP benchmarks for a couple of decades, and it is commonly used to determine a system’s cost or power efficiency. It’s useful for determining the common, day-to-day performance of an OLTP database”, — PingCAP [quote](https://www.pingcap.com/blog/why-benchmarking-distributed-databases-is-so-hard/).

[YugabyteDB](https://www.yugabyte.com/) has also stated, that TPC-C is the most objective performance measurement of OLTP systems, but we can’t find the exact quote.

Our previous [post](/ydb-meets-tpc-c-distributed-transactions-performance-now-revealed-42f1ed44bd73) has a “TPC-C 101” section as well as a description of our TPC-C implementation for YDB. You can find more information about our Benchbase based (pun intended) TPC-C implementation for PostgreSQL in this [post](/how-we-switched-to-java-21-virtual-threads-and-got-deadlock-in-tpc-c-for-postgresql-cca2fe08d70b). Since that time our implementation for PostgreSQL has moved from the [c3p0](https://www.mchange.com/projects/c3p0/) to the [HikariCP](https://github.com/brettwooldridge/HikariCP). Below we gently recap the TPC-C metrics, which should be enough to understand the results.

The goal of TPC-C benchmark is to achieve the highest number of new order transactions per minute (metric called *tpmC*) within given latency restrictions (for example, 90% of new order transactions must be below 5 seconds). The benchmark is designed in a way, that the maximum number of tpmC per warehouse is limited: to get more tpmC, you have to service more warehouses and each warehouse is approximately 100 MiB of data. In this post for simplicity we interchangeably use the number of warehouses and tpmC: it means, that the achieved tpmC is very close to the maximum possible tpmC for the specified number of warehouses.

Also, keep in mind, that TPC-C is a standard that describes the benchmark without any official implementation. Neither our TPC-C implementations for YDB and PostgreSQL (both based on the [Benchbase](https://github.com/cmu-db/benchbase)), nor the one from CockroachDB are officially ratified by the [TPC](https://www.tpc.org/) organization, but all closely follow the TPC-C v5.11.0 [specification](https://www.tpc.org/tpc_documents_current_versions/current_specifications5.asp). It means that the results provided in this post are not officially recognized TPC results and are not comparable with other TPC-C test results published on the TPC website.

# Setup

We conduct our experiments on a bare metal cluster with three machines. Each machine has the following configuration:

*   Two 32-core processors Intel Xeon Gold 6338 2.00GHz with hyper-threading turned on (128 logical cores).
*   512 GiB of RAM.
*   Four NVMe Intel-SSDPE2KE032T80 disks.
*   50 Gbps network.
*   huge pages in the case of Postgres, and transparent huge pages in the case of distributed DBMS.
*   Ubuntu 20.04.3 LTS.
*   Linux kernel version 5.4.161–26.3.

# Postgres setup

The used PostgreSQL version is 16.2 (Ubuntu 16.2–1.pgdg20.04+1). For the final Postgres setup, we configured two RAID 0 devices, each containing two NVMe disks. One RAID 0 is used for the WAL, another one is for the data. Also, we configured Linux as follows:

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

RAID 0 setup:

```
sudo mdadm --create /dev/md/md_db  --assume-clean -l0 --raid-devices=2 /dev/nvme1n1p2 /dev/nvme0n1p2
sudo mdadm --create /dev/md/md_wal  --assume-clean -l0 --raid-devices=2 /dev/nvme2n1p2 /dev/nvme3n1p2
```

We format the RAID devices with the following command:

```
mkfs.ext4 -F -m 0 -E lazy_itable_init=0,lazy_journal_init=0,discard
```

And then mount it with `noatime` option.

[Here](https://github.com/ydb-platform/postgres_vs_distributed/tree/main/postgres) you can find the full configuration files. There are 5 TPC-C clients, each uses 60 sessions to connect.

# YDB setup

We use YDB [24.1.10](https://storage.yandexcloud.net/binaries.ydb.tech/release/24.1.10/ydbd-24.1.10-linux-amd64.tar.gz) (testing version). There is 1 storage node (32 cores taskset) and 6 compute nodes (each one in its own 16 cores taskset) per machine. You can find the full configuration [here](https://github.com/ydb-platform/postgres_vs_distributed/tree/main/ydb), which is close to the default one. In total, there are up to 3000 connections to the YDB from the 3 TPC-C clients.

# CockroachDB setup

The used CockroachDB version is 23.1.10\. We failed to run TPC-C on newer 23.2.2, because of this [issue](https://github.com/cockroachdb/cockroach/issues/119924). Although the issue persists in version 23.1.10, the CockroachDB team has indicated that, in this version, the problem arises due to cluster overload.

We run per NVMe CockroachDB instance in a 32 cores taskset, 37.5 GiB cache and 37.5 GiB SQL memory. We apply [this](https://www.cockroachlabs.com/docs/v23.1/performance-benchmarking-with-tpcc-medium#step-3-configure-the-cluster) recommended configuration. [Here](https://github.com/ydb-platform/postgres_vs_distributed/blob/main/cockroach/cluster_config.py) is the configuration used by our [install](https://github.com/ydb-platform/benchhelpers/tree/main/db_installers/cockroach/) scripts. Note, that we don’t use partitioning, which is an enterprise feature. We avoid this because there are scenarios when it’s not possible to partition data like in TPC-C.

# Results

Recall, that the TPC-C benchmark aims to achieve the highest possible tpmC score while remaining within specified transaction latency boundaries for transactions. As demonstrated, PostgreSQL wins attaining 1.05 times more tpmC running 18K warehouses with an efficiency of 92.37% than YDB running 16K warehouses with 98.57% efficiency. Meanwhile, YDB holds a 1.29 times advantage over CockroachDB running 12K warehouses with 97.8% efficiency. However, PostgreSQL exhibits significantly higher latency, though it remains within the TPC-C specified limits, compared to its distributed counterparts. Let’s take a closer look at the new order latencies (additional latency details are provided at the end of this post):

You might know that there’s typically a trade-off between throughput (tpmC) and latency: generally, higher tpmC leads to increased latencies. With 16K warehouses, YDB achieves its highest possible tpmC, and transaction latencies remain within TPC-C requirements, but in some production scenarios, these latencies might be considered too high. Therefore, we also provide results for YDB operating with 13K warehouses with 99.24% efficiency, where latencies are significantly reduced and comparable to those of CockroachDB running 12K warehouses, while still achieving a slightly higher tpmC than CockroachDB.

Below are the new order latencies in seconds during the benchmark execution as reported by the TPC-C client:

The latencies are not “steady”. Each spike corresponds to the beginning of a checkpoint, which seems sessions to hang while waiting for `IPC: SyncRep`. Despite our efforts, we were unable to resolve this issue. For PostgreSQL, we observed noticeably better latencies only when below approximately 10K warehouses. We tried to triage the Postgres latencies. In the next section, we describe our extensive investigation into tuning PostgreSQL.

Comparing efficiency, PostgreSQL utilizes only about 15% of the available CPU on the leader machine. As we understand (and demonstrate in the next section), replication is the only bottleneck. This suggests PostgreSQL could achieve comparable results on servers with fewer CPU cores than ours. In contrast, distributed databases are less CPU efficient, with CPU usage nearing 100%. However, they also demonstrate better latencies, which can be considered a CPU-latency trade-off.

The shown results demonstrate PostgreSQL’s exceptional efficiency. However, we had anticipated a more substantial lead for PostgreSQL based on the belief that distributed DBMSs primarily excel when the cluster incorporates a substantially larger number of machines. In a sense, we have debunked this myth.

However, note that both distributed DBMSs discussed in this post can achieve over 1M tpmC on commodity hardware just by adding more machines to the cluster to handle the increased load. Below, we present ongoing results from a scalability test for YDB. We incrementally added more machines (9, 18, and 36) while maintaining the same configuration. The result comes from a testing version (upcoming YDB v24), though we anticipate similar performance in the YDB v23 branch. While these results might be suboptimal, they illustrate this point clearly:

Above we outlined that a DBMS should be fault-tolerant to a single server outage. Depending on the configuration it could be equivalent to a data center outage. Furthermore, in the event of a machine failure, the client load should be immediately redirected to the remaining servers to avoid operational downtime. For certain usage scenarios, this might represent a particularly stringent fault model. Below, we introduce ‘fault intolerant’ PostgreSQL configurations that achieve unprecedented high performance.

# Fault-intolerant, but extremely performant PostgreSQL configurations

Configuring PostgreSQL has essentially become a dedicated field of engineering. The main challenge with configuration is that you never know if your config is good enough until you have saturated hardware. Specifically, it’s crucial to determine whether further enhancements are achievable through additional PostgreSQL or Linux tuning. Another complication lies in the enormous number of configuration options ranging from I/O and concurrency to SQL planning and execution.

To streamline our configuration exploration, we began with unlogged tables and no replication (a “NoWAL” configuration). This approach maximizes commit speed by eliminating WAL overhead and potential network bottlenecks. Such a configuration is ideal for assessing SQL execution and the efficiency of I/O operations related to database data.

Next, we reintroduced WAL but adjusted it to optimize I/O distribution over time, creating infrequent checkpoints and enlarging WAL size. This ‘HugeWAL’ setup, while not suited for production due to the prolonged recovery times from crashes (potentially extending to tens of minutes), serves as a valuable configuration for testing.

Finally, we incorporated two synchronous replicas (‘R1’), allowing a switch to an alternate server in case of leader failure. Across all three configurations, we utilized three NVMe disks in RAID 0 for the database and a single NVMe disk for WAL storage. Let’s examine the outcomes:

During our tests, we observed significant unused CPU and network resources, with the disk also having additional write capacity. The findings are highly enlightening:

1.  The ‘NoWAL’ configuration reveals potential limitations due to either PostgreSQL SQL execution or the latency of the RAID 0 device handling database data. CPU usage stands at just 40 cores, with PostgreSQL writing at 1.5–2 GB/s (200–250K writes/s) and reading at 2.3 GB/s (275K reads, with a read unit of 8 KiB).
2.  A comparison of ‘HugeWAL’ against ‘NoWAL’ suggests that the observed performance degradation is the cost of utilizing WAL. PostgreSQL writes 800 MB/s of WAL data (approximately 6.7K writes/s, with a write unit of 128 KiB, which is the hardware limit of our NVMe disk).
3.  The significant performance drop in the ‘R1’ setup is attributed solely to replication overhead, not to WAL or SQL execution.

Achieving 35K warehouses (450K tpmC) is outstanding, yet the ‘NoWAL’ setup lacks reliability. Reaching 29K warehouses with ‘HugeWAL’ is also remarkable, and that’s what we would expect from the “R1” configuration. That is why we began investigating the slow replication.

Our initial finding is that replicas write 1.5–2 times more WAL than the leader node, for which we currently lack a precise explanation. Specifically, the leader writes at 250–450 MB/s (averaging 4K writes/s), while each replica writes at 500–800 MB/s. Notably, in the ‘HugeWAL’ configuration, writing was exactly 800 MB/s and performance was satisfactory.

The “WAL thing” on replicas gave us a hint to focus our investigation on this aspect. We employed the [procstat](https://github.com/FritsHoogland/procstat) tool to obtain real-time disk behavior data. We must acknowledge that procstat is an incredibly handy and user-friendly tool. Below are screenshots of WAL disk information obtained from procstat.”

The leader:

The replica:

It’s easy to see when exactly Postgres writes checkpoints. But the really interesting thing is that not only the follower writes more, but the latency of writes is 10–30 times higher (several milliseconds vs. hundreds of microseconds on the leader). Because of this, we made an assumption, that replicas WAL is the bottleneck.

To mitigate this issue, we experimented with the following adjustments:

```
bgwriter_delay = 20
bgwriter_lru_maxpages = 4192
```

```
wal_buffers = 256MB
wal_writer_delay = 400commit_delay = 200checkpoint_completion_target = 0.99
checkpoint_flush_after = 2MB
```

It didn’t help and we opted for a different approach to disk configuration: using two disks (instead of three) in RAID 0 for the database and another two disks in RAID 0 for WAL. Moreover, to expedite commits, we altered the `synchronous_commit` setting from `on` to `remote_write`. This change means that commits are not guaranteed to be durable on standby nodes in the event of an OS crash. However, it is also supposed to decrease commit latency on the leader and enhance overall system performance. We considered this compromise worthwhile.

This revised setup and configuration allowed us to successfully conduct a TPC-C benchmark over 2 hours, achieving 20K warehouses. However, extending the test to 12 hours revealed limitations: the system functioned for 5 hours before WAL space was exhausted on its replica. For the next 3 hours of downtime, replicas were struggling with cleaning and applying their WAL simultaneously. Adjusting the warehouse count down to 17K enabled the system to maintain operations throughout the 12-hour duration. Interestingly, post-benchmark, the replay lag was almost 3 hours, matching the time required to apply it:

```
tpcc=# SELECT pid,application_name,client_addr,client_hostname,state,sync_state,replay_lag FROM pg_stat_replication;
   pid   | application_name |           client_addr            | client_hostname |   state   | sync_state |   replay_lag
---------+------------------+----------------------------------+-----------------+-----------+------------+-----------------
 1250777 | s1               | 2a02:6b8:c34:14:0:5a59:eb1f:2ca6 |                 | streaming | sync       | 02:56:10.114747
 1250824 | s2               | 2a02:6b8:c34:14:0:5a59:eb1f:2956 |                 | streaming | sync       | 02:52:11.340846
```

This indicates that the replicas do not serve as effective hot standbys. In the event of the master’s failure, replicas would need a significant amount of time to apply the accumulated WAL, potentially resulting in hours of downtime. Moreover, if the lag continuously increases, there’s a risk of exhausting disk space allocated for WAL. Such a scenario would halt the primary’s ability to process client requests, leading to additional downtime, which, based on our experience, could extend for several hours.

We adjusted the `synchronous_commit` setting to `remote_apply` to address these issues. With this final configuration, PostgreSQL is capable of supporting 18K warehouses. Now, `replay_lag` has become negligible, which allows followers to serve as hot standbys. In this configuration, we observed the following resource consumption on the leader:

*   The leader consumes around 20 CPU cores.
*   The leader writes WAL at a rate of 400 MB/s (6K writes/s) and on average 600 MB/s of DB data (80K writes/s).
*   The leader reads data from the database at 700 MB/s.
*   Peak link utilization for the leader is around 9 Gbps.

And resource consumption by each of the followers:

*   Each follower writes approximately 800 MB/s of WAL data and about 500 MB/s of DB data.
*   Peak link utilization for each follower is 4 Gbps.

# Conclusions

There is an infinite number of hardware setups, PostgreSQL configurations, and different production workloads. In this post, we have explored PostgreSQL limitations on a particular hardware setup using a specific TPC-C workload. If you follow [our blog](https://blog.ydb.tech/), you might remember that this is our usual hardware for performance research, and like many DBMS developers, we are fond of TPC-C.

We must acknowledge that PostgreSQL is indeed extremely CPU efficient, even exceeding our expectations. “Fault intolerant” PostgreSQL setups demonstrate a fantastic performance, making PostgreSQL an excellent choice when you don’t need to survive a server (or data center) failure and when downtime is allowed. It’s as obvious as saying the sky is blue and the water is wet.

However, to be fault-tolerant and to store your data reliably, you should have multiple machines running your DBMS. That’s a must, if “failure is not an option” for your business. We were surprised to discover that PostgreSQL replication poses such a significant bottleneck and affects the achieved tpmC so dramatically. Furthermore, major latency spikes could pose challenges in certain production scenarios. We believe there might be a solution to this issue we aren’t aware of, so if you know of any effective strategies or “magic switches” we haven’t yet tried, please let us know in the comments below.

If PostgreSQL is not enough for your needs, we offer a relief. Contrary to popular belief, our study reveals that distributed databases excel even in modest three-machine clusters. Moreover, using YDB as an example, we’ve shown that distributed DBMS offers the opportunity to scale it easily, enabling you to focus your efforts on application development rather than database management nuances.

# Acknowledgements

This research is the result of our joint effort and close collaboration with our colleague [Evgeny Efimkin](https://linkedin.com/in/evgeny-efimkin-4061a893), who is an expert in PostgreSQL.

A special note of appreciation is extended to [Andrey Borodin](https://twitter.com/x4mmmmmm), an active contributor to PostgreSQL and the developer behind [Odyssey](https://github.com/yandex/odyssey), a scalable PostgreSQL connection pooler. Andrey provided strong support throughout our journey.

[Frits Hoogland](https://twitter.com/fritshoogland) provided invaluable feedback and shared a great tool with us, [procstat](https://github.com/FritsHoogland/procstat).

# Appendix: transaction latencies