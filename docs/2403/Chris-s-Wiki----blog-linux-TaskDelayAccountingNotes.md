<!--yml

category: 未分类

date: 2024-05-29 12:41:51

-->

# Chris's Wiki :: blog/linux/TaskDelayAccountingNotes

> 来源：[https://utcc.utoronto.ca/~cks/space/blog/linux/TaskDelayAccountingNotes](https://utcc.utoronto.ca/~cks/space/blog/linux/TaskDelayAccountingNotes)

如果您在典型的 Linux 系统上运行足够新的版本的 [iotop](https://man7.org/linux/man-pages/man8/iotop.8.html)，它可能会以如下方式提醒您：

> CONFIG_TASK_DELAY_ACCT 和 [kernel.task_delayacct](https://docs.kernel.org/admin-guide/sysctl/kernel.html#task-delayacct) sysctl 未在内核中启用，无法确定 SWAPIN 和 IO %。

您可能会想知道是否应该启用这个 sysctl，以及您关心程度如何，以及为什么它在第一次默认情况下被禁用。

此 sysctl 启用了[(Task) Delay accounting](https://docs.kernel.org/accounting/delay-accounting.html)，它会跟踪诸如进程等待 CPU 或等待其 IO 完成的时间，按任务计算（在 Linux 中基本上是 '线程'）。一般的系统信息会在诸如 'iowait%' 和 [pressure stall information](/~cks/space/blog/linux/PSINumbersAndMeanings) 中提供整体测量，但这些是聚合数据；您可能对了解特定进程延迟或等待 IO 的情况感兴趣。

（另外，[整体系统 iowait% 是一种保守的测量](/~cks/space/blog/linux/LinuxMultiCPUIowait)，并不能完全准确地反映进程等待 IO 的情况。您可以获取每个 cgroup 的压力停滞信息，在某些情况下可以接近每个进程的数据。）

在特定于 [iotop](https://man7.org/linux/man-pages/man8/iotop.8.html) 的上下文中，您可能会错过的主要内容是 'IO %'，即特定进程等待 IO 的时间百分比。任务延迟计费可以提供关于每个进程（或任务）运行队列延迟的信息，但我不知道是否有类似 iotop 的工具能提供这样的信息。在内核源代码中有一个程序，[tools/accounting/getdelays.c](https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/tree/tools/accounting/getdelays.c)，它会一次性地转储原始信息（在某些版本中，还会为您计算平均值，这可能是有益的）。您理论上可以获取到的当前任务延迟计费信息在 [include/uapi/linux/taskstats.h](https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/tree/include/uapi/linux/taskstats.h) 的注释中有文档记录，或者在 [文档中的这个版本](https://docs.kernel.org/accounting/taskstats-struct.html)。您可能还想查看 [include/linux/delayacct.h](https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/tree/include/linux/delayacct.h)，我认为它是跟踪这些信息的内核内部版本。

（你可能需要从你的内核源树中获取 getdelays.c 的版本，因为当前版本可能与你的内核不兼容。这通常会导致编译错误，至少是显而易见的错误。）

如何自己获取这些信息，有点在 [任务统计界面](https://docs.kernel.org/accounting/taskstats.html) 中提到，但实际上，你会想要阅读 getdelays.c 的源代码或者 [iotop](https://repo.or.cz/iotop.git) 的 Python 源代码。如果你特别想追踪一个任务在进行IO延迟时花费了多长时间，/proc/<pid>/stat 中也有一个字段；参见 [proc(5)](https://man7.org/linux/man-pages/man5/proc.5.html)，第42个字段是 delayacct_blkio_ticks。就我从内核源码中了解的情况而言，这与 [netlink 接口](https://docs.kernel.org/accounting/taskstats.html) 提供的信息是一样的，尽管它只有等待“块”（文件系统）IO的总时间，并没有块IO操作的计数。

理论上可以在每个 cgroup 基础上请求任务延迟会计（如我在关于 Linux 平均负载来自哪里的之前条目中看到的），但实际上，这仅适用于 [cgroup v1](https://docs.kernel.org/admin-guide/cgroup-v1/index.html)。这种（任务）延迟会计从未添加到 [cgroup v2](https://docs.kernel.org/admin-guide/cgroup-v2.html)，这可能表明整个功能有些被忽视。我找不到太多关于为什么延迟会计在 2021 年被更改为默认关闭的原因。[做出此更改的提交](https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=e4042ad492357fa995921376462b04a025dd53b6) 似乎暗示它默认关闭的假设是因为它没有被广泛使用。还请参阅 [此内核邮件列表消息](https://lore.kernel.org/all/20210505111525.308018373@infradead.org/T/) 和 [此 Reddit 线程](https://old.reddit.com/r/linuxquestions/comments/1b6bijd/downsides_to_kerneltask_delayacct/)。

现在我已经发现了 kernel.task_delayacct 并进行了一些尝试，我认为它对于我们诊断问题是足够有用的，因此我们将默认启用它，直到我们遇到问题（无论是性能还是其他方面）。可能我会用一个 /etc/sysctl.d/ 的文件来完成这个操作，因为我认为这样可以在启动早期覆盖大多数感兴趣的进程。

（正如某处所述，如果通过 sysctl 打开延迟会计，显然只覆盖了在更改 sysctl 后启动的进程。在更改之前启动的进程没有延迟会计信息，或者可能只有“CPU”延迟会计信息。其中一个这样的进程是 init，PID 1，它将始终在设置 sysctl 之前启动。）

PS：每个任务的IO延迟确实包括NFS IO，[就像iowait一样](/~cks/space/blog/linux/NFSIOShowsInIowait)，如果你有NFS客户端，这可能会使情况更有趣。有时候很明显哪些程序受到慢速NFS服务器的影响，但有时候却不明显。
