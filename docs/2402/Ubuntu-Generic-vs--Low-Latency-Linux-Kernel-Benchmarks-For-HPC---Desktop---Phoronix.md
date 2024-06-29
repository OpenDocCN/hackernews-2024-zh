<!--yml

类别：未分类

日期：2024-05-29 12:52:10

-->

# Ubuntu 通用 vs. 低延迟 Linux 内核基准测试 - Phoronix

> 来源：[https://www.phoronix.com/news/Ubuntu-Generic-LL-Kernel](https://www.phoronix.com/news/Ubuntu-Generic-LL-Kernel)

随着

[Ubuntu 正在考虑将其低延迟优化应用到其通用内核构建中](https://www.phoronix.com/news/Ubuntu-Low-Lat-Generic-Kernel)

为了消除维护现有的 "低延迟" 内核选项，我决定进行一些新鲜的基准测试，以研究低延迟内核与 Ubuntu Linux 系统上使用的 "通用" 默认内核之间的性能影响。

由于这些天主要是一些 Kconfig 的更改，他们在考虑将它们合并到他们的通用内核构建中。这将节省维护 "低延迟" 内核构建所需的额外构建资源、质保/测试以及用户端的简便性，并使通用内核构建更具多功能性。

![AMD EPYC 8534P](img/4c3b2ede1c9b09691ee36165378cc4e4.png)

作为权衡改变的一部分，Canonical 的内核工程师已经认识到，这可能会导致性能退化，特别是在 HPC 领域。因此，在 Ubuntu 23.10 上基于当前 Linux 6.5 版本的内核，我对其现有的 "通用" 和 "低延迟" 内核构建进行了基准测试。

此轮测试是在 AMD EPYC 8534P "Siena" 64 核服务器上进行的，使用相同配置和默认的 "通用" 内核针对其 "低延迟" 形式的同一内核版本运行相同的基准测试。重点是查看使用低延迟内核构建可能带来的性能影响，特别是对 HPC 和其他一般工作负载的影响。

共

[148 个基准测试](https://openbenchmarking.org/result/2401290-NE-UBUNTUKER76&sgm=1&swl=1)

在两种内核版本上运行测试，通用内核仅快了 1%...

在 148 个基准测试中，只有少数几个出现了可测量的差异，主要是在 Stress-NG 内核微基准测试中，这是可以预期的。在视频编码和 PostgreSQL 数据库基准测试中也有一些轻微波动。通常情况下，通用内核快的时间往往不超过 4%。

查看

[所有基准测试](https://openbenchmarking.org/result/2401290-NE-UBUNTUKER76&sgm=1&swl=1)

有兴趣的人可以查看完整内容。关于 Ubuntu 24.04 LTS 将在四月份发布，Canonical 在低延迟内核版本上的决定以及其他性能调整/增强的问题，将是非常有趣的。
