<!--yml

类别：未分类

日期：2024-05-29 12:50:15

-->

# 蠢萌的 RCU 技巧：内核对 RCU 的读取强度如何？- Paul E. McKenney's Journal — LiveJournal

> 来源：[https://paulmck.livejournal.com/67547.html](https://paulmck.livejournal.com/67547.html)

RCU 是一种专门的同步机制，通常用于读者更多的场景（

`rcu_read_lock()`

，

`rcu_read_unlock()`

，

`rcu_dereference()`

等）比更新者（

`synchronize_rcu()`

，

`call_rcu()`

，

`rcu_assign_pointer()`

，等等）。但 Linux 内核是否真的更多地使用了 RCU 的读取端原语而不是更新端原语？

一种确定这一点的方法是使用类似 ftrace 的工具记录对这些函数的所有调用。这种方法有效，但是跟踪消息可能会丢失，尤其是在频繁调用的函数上。另外，倒出跟踪缓冲区可能会扰乱系统。另一种方法是修改内核源代码以计算这些函数调用的次数，并以一种对缓存友好的方式转储到用户空间。这种方法有效，但我有点懒。还有一种方法是向跟踪专家寻求建议。

最后，这就是我实际做的，因为我碰巧询问的跟踪人员是 Andrii Nakryiko，我对 BPF 总体和

`bpftrace`

命令特别是。如果你手头没有 Andrii，你可以通过 Brendan Gregg 的《BPF 性能工具》附录 A 和附录 B 来完成相当不错的工作。当然，你需要安装

`bpftrace`

本身，在许多 Linux 发行版上相当简单。

## Linux 内核 RCU 读取强度

曾经使用过的人

`sed`

和

`awk`

有一点的启动是因为你可以调用

`bpftrace`

带有

`-e`

参数和一系列跟踪点/程序对，其中一个程序是

`bpftrace`

用大括号括起来的代码。此代码被编译、验证，并作为内核模块加载到运行中的内核中。代码执行完成后，结果会直接打印在这里给你看

`stdout`

。例如：

> ```
> bpftrace -e 'kprobe:__rcu_read_lock { @rcu_reader = count(); } kprobe:rcu_gp_fqs_loop { @gp = count(); } interval:s:10 { exit(); }'
> 
> ```

这个命令使用

`kprobe`

可以附加程序到

`__rcu_read_lock()`

函数并附加一个非常相似的程序到

`rcu_gp_fqs_loop()`

函数，这个函数恰好在每个 RCU 优雅期间被调用一次。这两个程序都计算调用次数，带有

`@gp`

是

`bpftrace`

“变量”累积计数，和

`count()`

以一种对缓存友好的方式进行计数的函数。最后

`interval:s:10`

实际上附加了一个程序到定时器，这样最后一个程序将每 10 秒执行一次（“

`s:10`

”）。除了这个程序调用了

`exit()`

函数终止这个

`bpftrace`

程序在第一个 10 秒时间间隔结束时执行。终止时，

`bpftrace`

在空闲系统上输出以下内容：

> ```
> Attaching 3 probes...
> 
> @gp: 977
> @rcu_reader: 6435368
> 
> ```

换句话说，这段时间内大约有一千个宽限期和六百多万个RCU读者，读取到宽限期比率超过六千。这确实可以被视为读取密集型。

但是如果系统很忙呢？很大程度上取决于系统的忙碌程度以及具体的忙碌方式，但让我们使用那个老生常谈的方法，内核构建（但使用

`nice`

命令以避免延迟

`bpftrace`

）。我们也把

`bpftrace`

将脚本命名为创造性地命名的文件

`rcu1.bpf`

如下所示：

> ```
> kprobe:__rcu_read_lock
> {
>         @rcu_reader = count();
> }
> 
> kprobe:rcu_gp_fqs_loop
> {
>         @gp = count();
> }
> 
> interval:s:10
> {
>         exit();
> }
> 
> ```

这允许命令

`bpftrace rcu1.bpf`

产生以下输出：

> ```
> Attaching 3 probes...
> 
> @gp: 274
> @rcu_reader: 78211260
> 
> ```

空闲系统在十秒内大约有一千个宽限期，而忙碌系统只有274个。另一方面，忙碌系统有78百万个RCU读取侧关键部分，是空闲系统的十倍以上。忙碌系统每个宽限期有超过四分之一百万个RCU读取侧关键部分，这显然是严重的读取密集型。

RCU 努力使相同的宽限期计算覆盖多个请求。因为

`synchronize_rcu()`

调用

`call_rcu()`

，我们可以使用

`call_rcu()`

调用作为每个宽限期更新的粗略代理，即对宽限期请求的请求数量。（调用越多

`synchronize_rcu_expedited()`

和

`kfree_rcu()`

，这个代理的越粗糙。

我们可以使

`bpftrace`

通过将相同的操作分配给一组跟踪点来使脚本更加简洁，如下所示

`rcu2.bpf`

在这里显示的文件：

```

kprobe:__rcu_read_lock,
kprobe:call_rcu,
kprobe:rcu_gp_fqs_loop
{
        @[func] = count();
}

interval:s:10
{
        exit();
}

```

有了这个文件，

`bpftrace rcu2.bpf`

在内核构建中间产生以下输出：

```

Attaching 4 probes...

@[rcu_gp_fqs_loop]: 128
@[call_rcu]: 195721
@[__rcu_read_lock]: 21985946

```

这些结果看起来与之前的内核构建结果大不相同，确证了您对内核构建作为可重复基准测试的适用性的任何怀疑。尽管如此，每个宽限期大约有180K个RCU读取侧关键部分，这仍然是严重的读取密集型。此外，还有将近2K

`call_rcu()`

每个RCU宽限期的调用，这意味着RCU能够将给定宽限期的开销几乎降至零。

## Linux内核RCU宽限期延迟

以下

`bpftrace`

程序制作宽限期延迟的直方图，即从调用到

`rcu_gp_init()`

到从

`rcu_gp_cleanup()`

：

```

kprobe:rcu_gp_init {
        @start = nsecs;
}

kretprobe:rcu_gp_cleanup {
        if (@start) {
                @gplat = hist((nsecs - @start)/1000000);
        }
}

interval:s:10 {
        printf("Internal grace-period latency, milliseconds:\n");
        exit();
}

```

在

`kretprobe`

将程序附加到

`rcu_gp_cleanup()`

。这

`hist()`

函数计算一个对数尺度的直方图。检查

`@start`

变量在这个脚本通常在宽限期中开始的情况下避免了此变量的时间开头值。（尝试在没有该检查的情况下！）

输出如下：

```

Attaching 3 probes...
Internal grace-period latency, milliseconds:

@gplat: 
[2, 4)               259 |@@@@@@@@@@@@@@@@@@@@@@                              |
[4, 8)               591 |@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@|
[8, 16)              137 |@@@@@@@@@@@@                                        |
[16, 32)               3 |                                                    |
[32, 64)               5 |                                                    |

@start: 95694642573968

```

大多数宽限期在4到8毫秒之间完成，剩余大部分在2到4毫秒之间完成，然后在8到16毫秒之间，但也有少数拖延者需要64毫秒。最后

`@start`

行显示

`bpftrace`

仅仅输出所有变量。您可以使用

`delete(@start)`

函数来防止打印

`@start`

，但请注意下一次调用

`rcu_gp_init()`

将重新创建它。

知道RCU宽限期的内部延迟很好，但大多数内核用户更关心

`synchronize_rcu()`

函数，需要等待当前宽限期完成并等待回调调用。我们可以使用以下命令来测量此函数的延迟

`bpftrace`

脚本：

```

kprobe:synchronize_rcu {
        @start[tid] = nsecs;
}

kretprobe:synchronize_rcu {
        if (@start[tid]) {
                @srlat = hist((nsecs - @start[tid])/1000000);
                delete(@start[tid]);
        }
}

interval:s:10 {
        printf("synchronize_rcu() latency, milliseconds:\n");
        exit();
}

```

该

`tid`

变量包含当前运行任务的ID，这使得此脚本能够将给定的返回与

`synchronize_rcu()`

使用对应的调用来

`tid`

作为

`@start`

变量。

如您所预期，生成的直方图偏向于稍长的延迟，尽管没有仍在运行的部分：

```

Attaching 3 probes...
synchronize_rcu() latency, milliseconds:

@srlat: 
[4, 8)                 9 |@@@@@@@@@@@@@@@                                     |
[8, 16)               31 |@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@|
[16, 32)              31 |@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@|

@start[4075307]: 96560784497352

```

此外，我们看到不是一个而是两个值

`@start`

。该

`delete`

语句会摆脱旧的条目，但任何新的调用

`synchronize_rcu()`

将创建更多这些。

## Linux内核加速RCU宽限期延迟

Linux内核有时会执行

`synchronize_rcu_expedited()`

以获得更快的宽限期，并且以下命令会进一步导致

`synchronize_rcu()`

以像

`synchronize_rcu_expedited()`

：

> ```
> echo 1 >  /sys/kernel/rcu_expedited
> 
> ```

在双套接字系统上执行此操作，有80个硬件线程可能不明智，但你只能活一次！

不管是不是明智，以下

`bpftrace`

脚本测量

`synchronize_rcu_expedited()`

延迟，但以微秒为单位而非毫秒：

> ```
> kprobe:synchronize_rcu_expedited {
>         @start[tid] = nsecs;
> }
> 
> kretprobe:synchronize_rcu_expedited {
>         if (@start[tid]) {
>                 @srelat = hist((nsecs - @start[tid])/1000);
>                 delete(@start[tid]);
>         }
> }
> 
> interval:s:10 {
>         printf("synchronize_rcu() latency, microseconds:\n");
>         exit();
> }
> 
> ```

此脚本与内核构建并发运行的输出如下：

> ```
> Attaching 3 probes...
> synchronize_rcu() latency, microseconds:
> 
> @srelat: 
> [128, 256)            57 |@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@|
> [256, 512)            14 |@@@@@@@@@@@@                                        |
> [512, 1K)              1 |                                                    |
> [1K, 2K)               2 |@                                                   |
> [2K, 4K)               7 |@@@@@@                                              |
> [4K, 8K)               2 |@                                                   |
> [8K, 16K)              3 |@@                                                  |
> 
> @start[4140285]: 97489845318700
> 
> ```

大多数

`synchronize_rcu_expedited()`

调用在几百微秒内完成，但有几个在十毫秒左右的落后者。

但是线性直方图呢？这就是

`lhist()`

函数是用来做什么的，增加了最小值、最大值和桶大小参数：

> ```
> kprobe:synchronize_rcu_expedited {
>         @start[tid] = nsecs;
> }
> 
> kretprobe:synchronize_rcu_expedited {
>         if (@start[tid]) {
>                 @srelat = lhist((nsecs - @start[tid])/1000, 0, 1000, 100);
>                 delete(@start[tid]);
>         }
> }
> 
> interval:s:10 {
>         printf("synchronize_rcu() latency, microseconds:\n");
>         exit();
> }
> 
> ```

在后台使用通常的内核构建运行此命令：

> ```
> Attaching 3 probes...
> synchronize_rcu() latency, microseconds:
> 
> @srelat: 
> [100, 200)            26 |@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@|
> [200, 300)            13 |@@@@@@@@@@@@@@@@@@@@@@@@@@                          |
> [300, 400)             5 |@@@@@@@@@@                                          |
> [400, 500)             1 |@@                                                  |
> [500, 600)             0 |                                                    |
> [600, 700)             2 |@@@@                                                |
> [700, 800)             0 |                                                    |
> [800, 900)             1 |@@                                                  |
> [900, 1000)            1 |@@                                                  |
> [1000, ...)           18 |@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@                |
> 
> @start[4184562]: 98032023641157
> 
> ```

最后一个桶是溢出桶，包含超过一毫秒限制的测量。

上述直方图几乎没有空桶，但这主要是因为18

`synchronize_rcu_expedited()`

溢出了一毫秒限制的实例被合并成单一的

`[1000, ...)`

溢出桶。有时这正是所需的，但有时丢失最大延迟可能成为问题。这可以通过以下方式处理

`bpftrace`

程序：

> ```
> kprobe:synchronize_rcu_expedited {
>         @start[tid] = nsecs;
> }
> 
> kretprobe:synchronize_rcu_expedited {
>         if (@start[tid]) {
>                 @srelat[(nsecs - @start[tid])/100000*100] = count();
>                 delete(@start[tid]);
>         }       
> }       
> 
> interval:s:10 {
>         printf("synchronize_rcu() latency, microseconds:\n");
>         exit();
> }
> 
> ```

考虑到通常的内核构建后台负载，这会产生以下输出：

> ```
> Attaching 3 probes...
> synchronize_rcu() latency, microseconds:
> 
> @srelat[1600]: 1
> @srelat[500]: 1
> @srelat[1000]: 1
> @srelat[700]: 1
> @srelat[1100]: 1
> @srelat[2300]: 1
> @srelat[300]: 1
> @srelat[400]: 2
> @srelat[600]: 3
> @srelat[200]: 4
> @srelat[100]: 20
> @start[763214]: 17487881311831
> 
> ```

这有点难以阅读，但可以应用简单的脚本来生成类似这样的输出：

> ```
> 100: 20
> 200: 4
> 300: 1
> 400: 2
> 500: 1
> 600: 3
> 700: 1
> 1000: 1
> 1100: 1
> 1600: 1
> 
> ```

尽管存在异常值，例如最后一个条目，对应的调用需要花费介于1.6到1.7毫秒之间。

## 总结

这

`bpftrace`

`command` 可用于快速轻松地编写内核中编译的脚本程序，可用于测量和监视各种事物。本文集中讨论了RCU的几个方面，但是在 Brendan Gregg 的“BPF Performance Tools” 书中可以找到更多的材料。
