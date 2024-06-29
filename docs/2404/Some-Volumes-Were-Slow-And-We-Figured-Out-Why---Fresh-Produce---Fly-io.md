<!--yml

类别：未分类

日期：2024-05-27 13:22:53

-->

# 有些卷慢了，我们找出了原因 - 新鲜农产品 - Fly.io

> 来源：[https://community.fly.io/t/some-volumes-were-slow-and-we-figured-out-why/19394](https://community.fly.io/t/some-volumes-were-slow-and-we-figured-out-why/19394)

## 一个 Bug 报告

Fly Volumes 是快速的。这听起来像是吹牛，但事实是，我们做出了一些权衡以获得快速的卷。我们用本地连接的 NVMe 驱动器池支持它们，这意味着它们被固定到特定的物理服务器上，并且虽然我们对它们进行了备份，但通常您希望在上层执行某些操作以复制它们。它们可能会丢失数据！但反过来，它们非常快。

因此，在本周早些时候，从经历了I/O性能问题的人那里收到报告是令人震惊的。我们让发现 I/O 问题变得容易：您可以从我们的仪表板直接点击到 `Metrics`，查看 `I/O 利用率` 百分比，它应该很低。

处理公共云基础设施运维的一个棘手问题是，任何可能出错的地方都可能出错。我们的客户以各种可能的方式使用我们的硬件。性能问题可能是我们这边的问题，也可能是应用程序陷入昂贵的紧耦合循环中。我们开始深入挖掘，但没有看到任何模式。

然后 [我们的指标集群开始拖累。](https://status.flyio.net/incidents/2sl3hm7h8n3h) 嗯，我们对系统的性能包络有信心。我们设计它来进行扩展。但我们看到运行它的 Fly Machines 正在停止运行。我们的深入挖掘增加了一些紧急性。

然后一个客户报告了同样的停滞现象。我们通过重新创建机器（先放一下，稍后会提到）与他们一起解决了这个问题。但显然有些事情不对：正如 [俗语所说](https://gutenberg.ca/ebooks/flemingi-goldfinger/flemingi-goldfinger-00-h-dir/flemingi-goldfinger-00-h.html#:~:text=Mr%20Bond%2C%20they%20have%20a%20saying%20in%20Chicago%3A%20%22Once%20is%20happenstance.%20Twice%20is%20coincidence.%20The%20third%20time%20it%27s%20enemy%20action.)，“一次是巧合。两次是偶然。三次是事件。”

### 深入挖掘

[Fly.io](http://Fly.io) 的工作方式是，在我们的物理服务器上有一个名为 `flyd` 的编排服务，使用 Firecracker 和 Cloud-Hypervisor 生成和管理 KVM 虚拟机。两者都非常轻量和超快，因此我们在任何给定的物理服务器上运行大量的虚拟机。

我们知道客户的 Fly Machines 几乎没有问题，但我们也知道有些事情不对劲。我们希望抓到一个 Fly Machine 在操作中。因此，我们设置了一个小脚本，在任何 Firecracker 进程在“不可中断睡眠”（`D`）状态超过几秒钟时向我们发出警报。这表明该进程在I/O上停滞。停顿多秒，甚至连续多次进入 `D` 状态，意味着有些不对劲。

追踪几个这些麻烦进程给了我们需要检查的候选者。Linux 让这很容易：只需 `cat /proc/$pid/stack`。这是我们看到的：

```
[<0>] blk_io_schedule+0x22/0x40
[<0>] __blkdev_direct_IO_simple+0x222/0x320
[<0>] blkdev_direct_IO+0x71/0x80
[<0>] generic_file_read_iter+0x9c/0x150 
```

这是一个正常的堆栈跟踪：一个进程在等待调度的 I/O 操作完成。但我们看过的每个进程都有完全相同的跟踪。这有点可疑：`blk_io_schedule` 应该非常快，除非有什么阻碍。这就像捕捉到一个房间里所有人同时眨眼一样。

### 显而易见的问题是

发生了什么变化？

我们的硬件没有变化。而且特定区域的这些事件之间没有关联。我们正处于一轮 Firecracker 版本更新中（具有讽刺意味地，是为了提高 I/O 性能！），但新版本已启用特性标志。我们没有改变卷的工作方式。

除了我们之外，没人意识到。

本周早些时候，我们发布了一个小功能，允许 [对交换设备进行一流支持](https://community.fly.io/t/a-new-way-to-rootfs/19196/2)。这导致将交换移出根文件系统设备，并放到一个专用的交换设备上。

关于交换的问题在于，如果您在使用它，您可能会相当用力。由于交换内存的工作方式，大多数情况下对您的应用程序来说是不可见的，它可能会变得相当混乱，因为操作系统对您的开发框架进行了虚假的内存跟踪，并用磁盘操作掩盖其痕迹。

所以：我们限制了交换设备的 I/O 带宽。这是好事。交换是可用的，以防止您的应用程序在短暂的峰值时崩溃，但它受限制，因此不会意外地造成噪声邻居问题。一切都平衡了，世界就是这样正确。

你看得出接下来会发生什么。

检查我们正在限制速率的设备很容易：

```
$ find /sys/fs/cgroup/ -name blkio.throttle.write_bps_device | xargs egrep '253'
/sys/fs/cgroup/blkio/blkio.throttle.write_bps_device:253:179 16777216
/sys/fs/cgroup/blkio/blkio.throttle.write_bps_device:253:101 16777216
/sys/fs/cgroup/blkio/blkio.throttle.write_bps_device:253:233 16777216
/sys/fs/cgroup/blkio/blkio.throttle.write_bps_device:253:250 16777216
/sys/fs/cgroup/blkio/blkio.throttle.write_bps_device:253:86 16777216
/sys/fs/cgroup/blkio/blkio.throttle.write_bps_device:253:114 16777216
... 
```

大多数 `major,minor` 对应于交换设备。在这里，“大多数”并不是您想要的答案。现在我们知道发生了什么：一些 Fly 机器由于错误的交换速率限制而受到限制。

剩下的问题是“为什么”。而对于设置速率限制的 `flyd` 代码的快速审计给出了答案：

```
writeDev.Major = int64(rdev / 256)
writeDev.Minor = int64(rdev % 256) 
```

要使用文件系统上设备的路径工作，您需要调用 `stat(2)`。`stat` 给我们一个 `dev_t`，其中包含设备的 `major,minor` 数字，您需要将其解包。很简单：主要是顶部的8位，次要是底部的8位，共16位值…

…如果是在2003年。

检查 [linux/kdev_t.h](https://github.com/torvalds/linux/blob/v5.18/include/linux/kdev_t.h#L7)：

```
#define MINORBITS	20 
```

我们的编排器可能在错误的设备上设置了 `blkio` 限制，将其限制为16MiB/s，并直接将任何 I/O 密集型应用程序推向它们的噩梦。

击中这一点的概率并不大。这解释了为什么提高的 `iowait` 时间似乎如此零星。但是在 [Fly.io](http://Fly.io) 规模上，百万事件中的一次事件会持续发生。

现在您可以看到为什么重新创建Fly Machine可以解决这个问题：它生成了一个新的交换设备和一个新的根文件系统设备，这些设备不太可能再次碰到块设备编号抽签失败的情况。

### 我们做了什么。

清除不良速率限制足够简单。

因此修复了处理`flyd`代码中的`st_rdev`的问题。

与此同时，我们正在研究更好的容器进程状态遥测。机器长时间处于不可中断状态是否值得我们采取行动，或者只是误报事件，目前还不确定，但这可能会为我们提供一些信号，帮助检测潜在问题。

我们还想直截了当地提高Firecracker的IO层速度。我不认为这会对这个 bug 有帮助，但更加确定地了解一台机器应有的预期IO性能，可能会让我们确定观察到的性能是否受到我们平台变更的影响。我们将很快详细介绍这一点。

### 简而言之。

您将应用托管在[Fly.io](http://Fly.io)，可能在过去几天看到IO异常缓慢的情况。我们做出的一项变更导致了这种情况，如果您的应用受到影响，我们感到遗憾。裸金属性能是我们平台的一个特性，我们对此非常热衷。

一如既往，如果您遇到问题或发现什么不正常的情况，应及时与我们联系。尽管我们的警报系统通过大量启发式分析海量指标数据，但您的应用是您关心的，我们希望知道您是否遇到任何问题。我们的用户帮助我们识别了这个问题，我们对您的参与表示感激。
