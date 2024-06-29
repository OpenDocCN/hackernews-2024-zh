<!--yml

category: 未分类

日期：2024-05-27 14:58:12

-->

# 通过 Generational ZGC 控制暂停时间 | 由 Netflix 技术博客提供 | Netflix TechBlog

> 来源：[https://netflixtechblog.com/bending-pause-times-to-your-will-with-generational-zgc-256629c9386b?gi=f77216038c95](https://netflixtechblog.com/bending-pause-times-to-your-will-with-generational-zgc-256629c9386b?gi=f77216038c95)

# 通过 Generational ZGC 控制暂停时间

*在 Z 垃圾收集器中世代的惊人和不那么惊人的好处。*

作者：Danny Thomas，JVM 生态系统团队

JDK 的最新长期支持版本为[Z 垃圾收集器](https://docs.oracle.com/en/java/javase/21/gctuning/z-garbage-collector.html)提供了世代支持。由于并发垃圾收集的显著好处，Netflix 已经默认从 G1 切换到 JDK 21 及更高版本的 Generational ZGC。

我们超过一半的关键流媒体视频服务现在都在运行 JDK 21，并采用 Generational ZGC，现在是时候谈谈我们的经验和我们所见到的好处了。如果您对 Netflix 如何使用 Java 感兴趣，Paul Bakker 的演讲[Netflix 真正使用 Java 的方式](https://www.infoq.com/presentations/netflix-java/) 是一个很好的起点。

# 降低尾延迟

在我们的 GRPC 和 [DGS 框架](https://netflix.github.io/dgs/)服务中，GC 暂停是尾延迟的重要来源。这在我们的 GRPC 客户端和服务器中尤为明显，由于由于超时而导致的请求取消与重试、对冲和回退等可靠性功能交互。每个这些错误都是一个取消的请求，导致一个重试，因此这种减少进一步降低了总体服务流量。

每秒的错误率。上周的白色相对于当前取消率的紫色，因为 ZGC 在 11 月 16 日在一个服务集群上启用

通过消除暂停的干扰，我们还能够识别端到端的实际延迟来源，这些延迟原本会被干扰掩盖，因为最大的暂停时间异常可能非常显著：

相同服务集群的按原因分类的最大 GC 暂停时间。是的，那些 ZGC 的暂停通常都在一毫秒以下。

# 效率

即使在我们的评估中看到了非常有前景的结果之后，我们仍然预计采用 ZGC 将是一种权衡：由于存储和加载障碍、在线程本地握手中执行的工作以及 GC 与应用程序竞争资源，会导致一些应用吞吐量的下降。我们认为这是可以接受的权衡，因为避免暂停带来的好处将超过这些开销。

实际上，对于我们的服务和架构，我们发现并不存在这样的权衡。对于给定的 CPU 利用率目标，与 G1 相比，ZGC 在平均延迟和 P99 延迟上都改善了，并且在 CPU 利用率上表现相等或更好。

我们在许多服务中看到的请求率、请求模式、响应时间和分配率的一致性确实有助于ZGC，但我们发现它同样能够处理不那么一致的工作负载（当然也有例外；稍后详述）。

# 操作上的简单性

服务所有者经常与我们联系，询问过高的暂停时间并寻求帮助进行调优。我们有几个框架定期刷新大量的堆上数据以提高效率，这些定期刷新的堆上数据往往会让G1感到措手不及，导致暂停时间超出默认暂停时间目标。

这些长期存在于堆上的数据是我们之前不采用非代际ZGC的主要原因。在我们评估的最坏情况下，非代际ZGC的CPU利用率比同样工作负载下的G1高出36%。对于代际ZGC来说，这几乎提高了近10%。

所有流媒体视频服务的一半都使用我们的[Hollow](https://hollow.how/)库用于堆上元数据。消除暂停作为一个问题使我们能够[移除数组池化缓解措施](https://github.com/Netflix/hollow/commit/4f21ab593543bb622d9ccea2f8e6295eae5e8080)，为分配释放数百兆字节内存。

操作上的简单性也源于ZGC的启发式算法和默认设置。我们不需要进行显式调整即可实现这些结果。分配停顿是罕见的，通常与分配速率异常波动同时发生，并且比我们在G1中看到的平均暂停时间更短。

# 内存开销

在堆大小小于32G时，我们预计由于[彩色指针需要64位对象指针](https://youtu.be/YyXjC68l8mw?t=816)，失去了[压缩引用](https://shipilev.net/jvm/anatomy-quarks/23-compressed-references/)将是选择垃圾收集器的一个重要因素。

我们发现，尽管这对于停顿式GC来说是一个重要的考虑因素，但对于ZGC来说却不是，即使在小堆上，分配速率的增加也被效率和操作改进所摊销。我们感谢Oracle的Erik Österlund解释了彩色指针在并发垃圾收集器中的不太直观的好处，这使我们对ZGC的评估比最初计划的更广泛。

在大多数情况下，ZGC也能够始终为应用程序提供更多的可用内存：

每个GC周期后，与上述相同服务集群的已用与可用堆容量

ZGC的固定开销是堆大小的3%，需要比G1更多的本地内存。除了少数几个情况外，我们没有必要降低最大堆大小以腾出更多的余地，而这些情况是具有高于平均水平的本地内存需求的服务。

只有在主要集合中才会使用 ZGC 进行引用处理。我们特别关注了直接字节缓冲区的释放，但到目前为止我们并没有看到任何影响。这种引用处理的差异导致了一个[JSON线程转储支持的性能问题](https://bugs.openjdk.org/browse/JDK-8321178)，但这是一个不寻常的情况，是由一个框架意外为每个请求创建了一个未使用的ExecutorService实例引起的。

# 透明大页

即使你没有使用 ZGC，你可能应该使用大页，而且[透明大页](https://shipilev.net/jvm/anatomy-quarks/2-transparent-huge-pages/)是使用它们最方便的方式。

ZGC使用共享内存作为堆，许多Linux发行版会将shmem_enabled配置为*never*，这会悄悄地阻止ZGC使用-XX:+UseTransparentHugePages进行大页的使用。

这里有一个服务，除了将shmem_enabled从never改为advise外，没有进行其他更改，这显著减少了CPU利用率：

从4k到2m的页的部署移动。不必理会这个差距，这是我们不可变的部署过程临时使集群容量翻倍。

我们的默认配置：

+   将堆的最小和最大值设置为相等

+   配置-XX:+UseTransparentHugePages -XX:+AlwaysPreTouch

+   使用以下transparent_hugepage配置：

```
echo madvise | sudo tee /sys/kernel/mm/transparent_hugepage/enabled
echo advise | sudo tee /sys/kernel/mm/transparent_hugepage/shmem_enabled
echo defer | sudo tee /sys/kernel/mm/transparent_hugepage/defrag
echo 1 | sudo tee /sys/kernel/mm/transparent_hugepage/khugepaged/defrag
```

# 有哪些工作负载不太合适？

没有最好的垃圾收集器。每个垃圾收集器在垃圾收集器的目标确定时都会在收集吞吐量、应用程序延迟和资源利用之间进行权衡。

那些与ZGC相比性能更好的工作负载通常更加注重吞吐量，具有非常波动的分配速率，并且长时间运行的任务会在不可预测的时期持有对象。

一个值得注意的例子是一个服务，它的分配速率非常波动，同时有大量长寿命对象，这恰好非常适合于G1的暂停时间目标和老年代区域收集启发式。它使G1能够在ZGC无法做到的GC周期中避免无效的工作。

默认切换到ZGC提供了一次完美的机会，让应用程序所有者思考他们对垃圾收集器的选择。几种批处理/预计算情况一直默认使用G1，而他们本可以从并行收集器中获得更好的吞吐量。在一个大型预计算工作负载中，与G1相比，我们看到了应用吞吐量的6-8%的改善，批处理时间缩短了1小时。

# 亲自尝试一下！

如果我们不去质疑，我们对假设和期望的盲目追随可能导致我们错过了这十年来对我们操作默认值做出的最有影响的变化之一。我们鼓励你亲自尝试分代ZGC。这可能会像它让我们感到惊讶一样让你感到惊讶。
