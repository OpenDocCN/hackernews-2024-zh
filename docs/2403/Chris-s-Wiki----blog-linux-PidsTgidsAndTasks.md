<!--yml

category: 未分类

date: 2024-05-29 12:38:10

-->

# Chris's Wiki :: blog/linux/PidsTgidsAndTasks

> 来源：[https://utcc.utoronto.ca/~cks/space/blog/linux/PidsTgidsAndTasks](https://utcc.utoronto.ca/~cks/space/blog/linux/PidsTgidsAndTasks)

在开始时，Unix只有进程，进程具有进程ID（PID），生活很简单。然后人们添加了（内核支持的）线程，因此进程可以是多线程的。当您添加线程时，需要为它们提供一些用户可见的标识符。有许多选项可以确定此标识符及其工作方式（以及线程在内核内部的工作方式）。Linux的选择是，线程只是进程（与其他进程共享更多），因此它们的标识符是一个进程ID，从与独立常规进程相同的全局进程ID空间中分配。这在程序和其他工具中对“进程ID”意味着一些歧义（包括对我而言）。

曾经称为“进程ID”的真正名称，也就是“一个进程及其所有线程”的整体实体的*TGID*（线程或任务组ID）。进程的TGID是主线程的PID；单线程程序的TGID与其PID相同。您可以在/proc/<PID>/status的“Tgid:”和“Pid:”字段中看到这一点。尽管某些地方会将“pid”和“tid”（例如[`proc(5)`](https://man7.org/linux/man-pages/man5/proc.5.html)的某些部分）区分开来，但这两种类型都是从相同的数字范围分配的，因为它们都是“PIDs”。如果我只给出一个“PID”而没有进一步的细节，就无法知道它是进程的PID还是任务的PID。

在每个/proc/<PID>目录中，都有一个“tasks”子目录；这包含属于线程组（即具有相同TGID）的所有*任务*（线程）的PID。所有PID都有一个/proc/<PID>目录，但是为了方便，像“ls /proc”只列出进程的PID（您可以将其视为TGID）。内核在请求/proc目录内容时不会返回其他任务的/proc/<PID>目录，尽管如果直接访问它们，可以使用它们（也可以通过/proc/<PID>/tasks访问或发现它们）。我不确定/proc/<PID>目录中哪些信息是特定于任务本身，哪些是TGID中所有任务的总体信息。[`proc(5)`](https://man7.org/linux/man-pages/man5/proc.5.html)手册有时谈论进程，有时谈论任务，但我不确定这是否全面。

(当您实际上查看的是TGID时，大部分时间您希望获取的是TGID中所有线程的总体信息。如果/proc/<PID>始终仅向您提供任务信息，即使对于“进程”PID/TGID，多线程程序在报告诸如CPU使用情况等方面可能会显示混乱低数字，除非您通过额外的方式总结/proc/<PID>/tasks/*的信息。)

通常情况下，各种工具将返回整个进程的 PID（TGID），而不是多线程进程中随机任务的 PID。例如，'pidof <thing>' 就是这样工作的。根据特定进程的工作方式，这可能是或不是程序的 '主线程'（一些多线程程序会将其初始线程放置在一边，然后在稍后创建的其他线程上进行主要工作），有时程序甚至不会有这样的概念（我认为 Go 程序大部分情况下不会有，因为它们根据需要将 goroutine 多路复用到实际线程上）。

如果一个工具或系统让你选择使用 'PID' 还是 'TGID'，则意味着你可以选择单个线程（任务）或整个进程来操作。你选择哪一个取决于你的操作，但如果你像[请求任务延迟信息](/~cks/space/blog/linux/LoadAverageWhereFrom)这样的操作，使用 TGID 可能更符合你的预期（因为它提供的是整个进程的信息，而不是特定线程的信息）。如果一个程序只讨论 PIDs，那么默认情况下它可能会操作或者提供整个进程的信息，尽管如果你给它一个任务内的 PID（而不是 TGID 的 PID），你可能会得到特定于该任务的信息。

在内核上下文中，例如[eBPF 程序](/~cks/space/blog/linux/EbpfExporterNotes)，我认为你几乎总是想通过 PID 跟踪事物，而不是 TGID。是 PIDs 完成诸如[体验运行队列调度延迟](/~cks/space/blog/linux/SystemResponseLatencyMetrics)、进行系统调用和引发阻塞 IO 延迟的任务，而不是 TGIDs。但是，如果你在选择要报告的内容、监控等方面，你很可能希望匹配 TGID 而不是 PID，这样你就可以报告多线程程序中的所有任务，而不仅仅是其中的一个（除非你专门关注任务/线程，而不是 '一个进程'）。

（我写下这些部分是为了在我的脑海中搞清楚，因为最近在与[eBPF 程序](/~cks/space/blog/linux/EbpfExporterNotes)一起工作时有些困惑。）
