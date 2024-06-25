<!--yml

类别：未分类

日期：2024-05-27 14:52:37

-->

# Linux 开发人员提高 6% 的文件系统性能——称其“实际上只需五分钟”的工作 | 汤姆硬件

> 来源：[https://www.tomshardware.com/software/linux/linux-dev-delivers-6-file-system-performance-increase-says-it-was-literally-a-5-min-job](https://www.tomshardware.com/software/linux/linux-dev-delivers-6-file-system-performance-increase-says-it-was-literally-a-5-min-job)

一位 Linux 开发者对缓存算法进行了一些修改，声称能够提高 I/O 操作的性能达到 6%。IO_uring 的创造者，同时也是自称为 [Linux](https://www.tomshardware.com/monitors/linux-is-the-only-os-to-support-diagonal-pc-monitor-mode-dev-champions-the-case-for-22-degree-rotation-computing) 内核 IO 爱好者的 Jens Axboe，在拖延多年后决定实现这些代码修改，但他 [承认](https://twitter.com/axboe/status/1747016366891442220) 这些修改“实际上只是一个 5 分钟的工作”（引自 [Phoronix](https://www.phoronix.com/news/Linux-Caching-Time-Block-IO)）。

Axboe 的补丁似乎通过减少对 I/O 系统的查询次数来实现性能提升。在他的 [RFC 补丁说明](https://lore.kernel.org/linux-block/20240115215840.54432-1-axboe@kernel.dk/) 中，Axboe 写道，许多代码“在查询时间方面非常敏感。” 尽管已经有一些代码来减少这种情况，但是新的补丁集，被 Axboe 描述为微不足道的， “简单地在 struct blk_plug 中缓存当前时间，前提是任何问题侧的时间查询都能通过它获得足够的精度。” 开发者推理道：“实际上没有人真正需要纳秒级别的时间戳。” 在这里，我们又有另一个例子，一些巧妙的思考在一个长期存在的技术中产生了可量化的效益。

## 一个人的五分钟工作为所有人（Linux 用户）带来了 6% 的 I/O 收益

在 Axboe 的测试中，与补丁前后进行的 IOPS 测试观察到了 6% 的改善。有趣的是，[异步 I/O 接口](https://unixism.net/loti/what_is_io_uring.html) 的开发者暗示 Linux 用户在真实世界中可能会看到更大的收益。这是因为在 Axboe 的测试系统中，“他甚至没有启用大多数代价高昂的块层项，这些项通常在发行版中都能找到，并且会进一步增加问题侧的时间调用次数。” 换句话说，那些使用更臃肿的 Linux 供应商内核的用户可能会从 Axboe 的新补丁中获得更多好处。

Phoronix 认为 RFC 补丁有很大的机会在今年晚些时候与 Linux 6.9 一起准备好并上游。无论何时到来，免费获得额外性能都是很棒的，尤其是存储往往是常见的系统瓶颈。与此同时，如果您正在寻找更快或更大容量的存储，不妨查看我们的[最佳 SSD 存储](https://www.tomshardware.com/reviews/best-ssds,3891.html)指南，涵盖了从预算 SATA 驱动器到最新的 M.2 PCIe SSD 驱动器的所有内容。

获取 Tom's Hardware 最佳新闻和深度评论，直接发送到您的收件箱。
