<!--yml

类别：未分类

日期：2024-05-27 14:51:05

-->

# 我们如何切换到 Java 21 虚拟线程并在 PostgreSQL 的 TPC-C 中遇到死锁 | 作者：Evgeniy Ivanov | YDB.tech 博客

> 来源：[`blog.ydb.tech/how-we-switched-to-java-21-virtual-threads-and-got-deadlock-in-tpc-c-for-postgresql-cca2fe08d70b`](https://blog.ydb.tech/how-we-switched-to-java-21-virtual-threads-and-got-deadlock-in-tpc-c-for-postgresql-cca2fe08d70b)

# **我们如何切换到 Java 21 虚拟线程并在 PostgreSQL 的 TPC-C 中遇到死锁**

Java 21 的餐馆哲学家有问题

在我们之前的文章中讨论了 TPC-C 的一些缺点，这些缺点来自于[Benchbase](https://github.com/cmu-db/benchbase)项目的原始实现（尽管这个项目仍然很棒）。其中一个缺点是由于生成了太多物理线程而导致的并发限制，我们通过切换到 Java 21 虚拟线程来解决了这个问题。后来我们发现，和往常一样，没有免费的午餐。在这篇文章中，我们提供了一个案例研究，介绍了我们如何在 TPC-C 中遇到了虚拟线程在 PostgreSQL 中的死锁问题，即使没有餐馆哲学家问题。

这篇文章可能对考虑切换到虚拟线程的 Java 开发人员有所帮助。我们首先复习了一些基本背景，然后强调了虚拟线程背后的一个重要问题：死锁可能是不可预测的，因为它们可能发生在你使用的库的深处。幸运的是，调试是直接了当的，我们解释了如何在发生死锁时找到它们。

# 为什么在 YDB 博客中谈论 PostgreSQL

PostgreSQL 是一个以其高性能、丰富的功能集、高级 SQL 兼容性水平和充满活力和支持性社区而闻名的开源数据库管理系统。直到你考虑到水平扩展性和容错性为止，它都很棒。然后，你会得到基于 PostgreSQL 的第三方解决方案，如 Citus，它们实现了分片的 PostgreSQL。拥有一只大象可能很有趣。成为一群大象的驭象人是一个挑战，特别是如果你想让这些大象维护多个一致的副本并执行具有可序列化隔离的分布式事务。

与此相反，[YDB](https://ydb.tech/)是一个按其原始设计分布式数据库管理系统。 YDB 的分布式事务是一流的公民，并且默认情况下以可序列化隔离级别运行。现在，我们正在积极地向 PostgreSQL 兼容性迈进，因为我们看到 PostgreSQL 用户中有很强的需求，希望他们的现有应用程序能够自动扩展和容错。这就是为什么我们维护[TPC-C for PostgreSQL](https://github.com/ydb-platform/tpcc)（我们希望很快将其合并到上游 Benchbase 中）。

# 一个非常简短的背景和动机

让我们回顾一些基本概念：并发性、并行执行以及异步与同步请求的区别。

并发意味着任务同时执行，无论是并行还是顺序执行。例如，你可能在编辑器中编写代码并与同事在 Slack 上聊天。你同时进行这些任务，但不是并行执行。或者你可能带着狗散步并和朋友打电话。同样，你同时进行这两项任务，但这次是并行执行。

现在，考虑当您的应用程序想要向数据库发出请求时的情况。请求通过网络发送，由数据库提供服务，然后回复发送回您的应用程序。请注意，网络往返可能是请求中最昂贵的部分，并且可能需要几毫秒。在等待回复期间，应用程序端可以做些什么？

1\. 请求可能是同步的，即它将阻塞调用线程。这种方法很容易编写代码：第 1 行是请求；第 2 行，您可以处理响应：

```
String userName = get_username_from_db(userId);
System.out.printf("Hello, %s!", userName);
```

2\. 请求可能是异步的。您的线程不会被阻塞，并继续执行，而请求会并行处理：

```
CompletableFuture<String> userNameFuture = get_username_from_db(userId);

// Note, that this is kind of callback, it's not executed "right here",
// even more, at some point it will be executed in parallel with your thread.
// In real life scenarios, you will have to use mutual exclusion.
userNameFuture.thenAccept(userName -> {
    System.out.println("Hello, %s!", userName);
});
execute_something_else();
userNameFuture.get(); // wait for the completion of your request
```

无论哪种情况，都有两个并发任务：您的线程正在等待来自数据库的回复，而数据库正在处理请求。同步代码非常简单易读。但是，如果您需要同时向数据库发出数千个请求怎么办？您将不得不为每个请求生成一个线程。在 Linux 中生成线程很便宜，尽管生成太多线程背后存在着严重的担忧：

1.  每个线程都需要一个堆栈。您无法分配比系统的页面大小更少的内存，通常为约 4 KiB，除非您使用大页面，其中默认页面大小为 2 MiB。

1.  存在着 Linux 调度器。如果有重置按钮，您可以尝试生成 100,000 个准备执行的线程。

这就是为什么直到 Java 21，没有办法编写具有高并发性的同步代码：你无法生成许多线程。并发（创意打趣），Go 语言革新了这一点：goroutines 提供了非常轻量级的并发，因此您可以*高效*地编写同步代码。我们推荐观看 Dmitry Vyukov 关于 Go 调度器的[演讲](https://youtu.be/-K11rY57K7k?si=XT8mkXL2ypmVj4ig)。Java 21 引入了类似 goroutines 的虚拟线程。请记住，goroutines 和虚拟线程不是新发明，而是对用户级线程的一种再次实现。

现在，你可以理解原始 Benchbase TPC-C 实现中同步数据库请求的问题了。如果你的数据库能够处理高负载，你必须运行许多 TPC-C 仓库，生成许多线程。使用物理线程，我们无法运行超过 30,000 个终端线程，而使用虚拟线程，我们可以轻松拥有数十万个终端虚拟线程。

# 死锁变得简单了

想象一下，您已经有了多线程 Java 代码。添加一个选项来使用虚拟线程非常容易，并且可能非常有益。通过简单地将标准线程创建替换为新的虚拟线程构建器，您的应用程序可以处理数千个并发任务，而无需与物理线程相关的开销。以下是我们 TPC-C 实现的一个[示例](https://github.com/ydb-platform/tpcc/blob/c8474bc7cf40456de0fe4c7fb060d867dd985ede/src/main/java/com/oltpbenchmark/ThreadBench.java#L65-L69)：

```
if (useRealThreads) {
    thread = new Thread(worker);
} else {
    thread = Thread.ofVirtual().unstarted(worker);
}
```

就是这样；现在，您正在使用虚拟线程。在幕后，Java 虚拟机创建了一个`载体线程`池，用于执行您的`虚拟线程`。这种过渡看起来是无缝的，直到您的应用程序意外地冻结。

我们的 PostgreSQL TPC-C 实现利用了[c3p0](https://www.mchange.com/projects/c3p0/)进行连接池管理。[TPC-C 标准](https://www.tpc.org/tpcc/)规定每个终端必须拥有自己的连接。然而，在许多实际场景中，这并不实际，因此我们提供了一个选项来限制数据库连接数。

终端数量远远大于可用连接数量。因此，一些终端必须等待会话变为可用，即由另一个终端释放。

当我们启动 TPC-C 运行时，应用程序卡住了。幸运的是，调试这种情况很简单：

1.  使用`jstack <PID>`捕获线程堆栈。

1.  使用`jcmd <PID> Thread.dump_to_file -format=text jcmd.dump.1`创建当前状态的更详细的转储，其中包括有关载体线程和虚拟线程的信息。

经过调查，我们发现一些等待会话的虚拟线程已经将其载体线程固定住了。以下是其中一个虚拟线程的堆栈：

```
#7284 "TPCCWorker<7185>" virtual
      java.base/java.lang.Object.wait0(Native Method)
      java.base/java.lang.Object.wait(Object.java:366)
      com.mchange.v2.resourcepool.BasicResourcePool.awaitAvailable(BasicResourcePool.java:1503)
      com.mchange.v2.resourcepool.BasicResourcePool.prelimCheckoutResource(BasicResourcePool.java:644)
      com.mchange.v2.resourcepool.BasicResourcePool.checkoutResource(BasicResourcePool.java:554)
      com.mchange.v2.c3p0.impl.C3P0PooledConnectionPool.checkoutAndMarkConnectionInUse(C3P0PooledConnectionPool.java:758)
      com.mchange.v2.c3p0.impl.C3P0PooledConnectionPool.checkoutPooledConnection(C3P0PooledConnectionPool.java:685)
      com.mchange.v2.c3p0.impl.AbstractPoolBackedDataSource.getConnection(AbstractPoolBackedDataSource.java:140)
      com.oltpbenchmark.api.BenchmarkModule.makeConnection(BenchmarkModule.java:108)
      com.oltpbenchmark.api.Worker.doWork(Worker.java:428)
      com.oltpbenchmark.api.Worker.run(Worker.java:304)
      java.base/java.lang.VirtualThread.run(VirtualThread.java:309)
```

以及其载体线程的堆栈：

```
"ForkJoinPool-1-worker-254" #50326 [32859] daemon prio=5 os_prio=0 cpu=12.39ms elapsed=489.99s tid=0x00007f3810003140  [0x00007f37abafe000]
   Carrying virtual thread #7284
        at jdk.internal.vm.Continuation.run(java.base@21.0.1/Continuation.java:251)
        at java.lang.VirtualThread.runContinuation(java.base@21.0.1/VirtualThread.java:221)
        at java.lang.VirtualThread$$Lambda/0x00007f3c2424e410.run(java.base@21.0.1/Unknown Source)
        at java.util.concurrent.ForkJoinTask$RunnableExecuteAction.exec(java.base@21.0.1/ForkJoinTask.java:1423)
        at java.util.concurrent.ForkJoinTask.doExec(java.base@21.0.1/ForkJoinTask.java:387)
        at java.util.concurrent.ForkJoinPool$WorkQueue.topLevelExec(java.base@21.0.1/ForkJoinPool.java:1312)
        at java.util.concurrent.ForkJoinPool.scan(java.base@21.0.1/ForkJoinPool.java:1843)
        at java.util.concurrent.ForkJoinPool.runWorker(java.base@21.0.1/ForkJoinPool.java:1808)
        at java.util.concurrent.ForkJoinWorkerThread.run(java.base@21.0.1/ForkJoinWorkerThread.java:188)
```

正如您所看到的，线程卡在`Object.wait()`中，这是与`synchronized`一起使用的方法。这导致载体线程被固定住，意味着它没有被释放以执行其他虚拟线程。与此同时，持有会话的线程在等待 I/O 操作时已经释放了它们的载体线程：

```
java.base/java.lang.VirtualThread.park(VirtualThread.java:582)
      java.base/java.lang.System$2.parkVirtualThread(System.java:2639)
      java.base/jdk.internal.misc.VirtualThreads.park(VirtualThreads.java:54)
      java.base/java.util.concurrent.locks.LockSupport.park(LockSupport.java:369)
      java.base/sun.nio.ch.Poller.pollIndirect(Poller.java:139)
      java.base/sun.nio.ch.Poller.poll(Poller.java:102)
      java.base/sun.nio.ch.Poller.poll(Poller.java:87)
      java.base/sun.nio.ch.NioSocketImpl.park(NioSocketImpl.java:175)
      java.base/sun.nio.ch.NioSocketImpl.park(NioSocketImpl.java:201)
      java.base/sun.nio.ch.NioSocketImpl.implRead(NioSocketImpl.java:309)
      java.base/sun.nio.ch.NioSocketImpl.read(NioSocketImpl.java:346)
      java.base/sun.nio.ch.NioSocketImpl$1.read(NioSocketImpl.java:796)
      java.base/java.net.Socket$SocketInputStream.read(Socket.java:1099)
      java.base/sun.security.ssl.SSLSocketInputRecord.read(SSLSocketInputRecord.java:489)
      java.base/sun.security.ssl.SSLSocketInputRecord.readHeader(SSLSocketInputRecord.java:483)
      java.base/sun.security.ssl.SSLSocketInputRecord.bytesInCompletePacket(SSLSocketInputRecord.java:70)
      java.base/sun.security.ssl.SSLSocketImpl.readApplicationRecord(SSLSocketImpl.java:1461)
      java.base/sun.security.ssl.SSLSocketImpl$AppInputStream.read(SSLSocketImpl.java:1066)
      org.postgresql.core.VisibleBufferedInputStream.readMore(VisibleBufferedInputStream.java:161)
      org.postgresql.core.VisibleBufferedInputStream.ensureBytes(VisibleBufferedInputStream.java:128)
      org.postgresql.core.VisibleBufferedInputStream.ensureBytes(VisibleBufferedInputStream.java:113)
      org.postgresql.core.VisibleBufferedInputStream.read(VisibleBufferedInputStream.java:73)
      org.postgresql.core.PGStream.receiveChar(PGStream.java:465)
      org.postgresql.core.v3.QueryExecutorImpl.processResults(QueryExecutorImpl.java:2155)
      org.postgresql.core.v3.QueryExecutorImpl.execute(QueryExecutorImpl.java:574)
      org.postgresql.jdbc.PgStatement.internalExecuteBatch(PgStatement.java:896)
      org.postgresql.jdbc.PgStatement.executeBatch(PgStatement.java:919)
      org.postgresql.jdbc.PgPreparedStatement.executeBatch(PgPreparedStatement.java:1685)
      com.mchange.v2.c3p0.impl.NewProxyPreparedStatement.executeBatch(NewProxyPreparedStatement.java:2544)
      com.oltpbenchmark.benchmarks.tpcc.procedures.NewOrder.newOrderTransaction(NewOrder.java:214)
      com.oltpbenchmark.benchmarks.tpcc.procedures.NewOrder.run(NewOrder.java:147)
      com.oltpbenchmark.benchmarks.tpcc.TPCCWorker.executeWork(TPCCWorker.java:66)
      com.oltpbenchmark.api.Worker.doWork(Worker.java:442)
      com.oltpbenchmark.api.Worker.run(Worker.java:304)
      java.base/java.lang.VirtualThread.run(VirtualThread.java:309)
```

因此，我们陷入了以下情况：

1.  所有载体线程都被会话等待者固定住了，这意味着没有可用的载体线程。

1.  拥有会话的虚拟线程无法完成其任务以释放会话。

死锁变得如此容易！

[JEP 444](https://openjdk.org/jeps/444)规定：

> *在两种情况下，虚拟线程由于被固定在其载体上而无法在阻塞操作期间卸载：*
> 
> *当执行代码时，它位于同步块或方法内，或当执行本机方法或外部函数时。*

问题在于这段同步代码可能被深深地嵌入到你所使用的库中。在我们的情况下，它位于 c3p0 库内。所以，[修复](https://github.com/ydb-platform/tpcc/commit/175f0c03d9c16652c85a6103331fec473017797e) 很简单：我们只需用 `java.util.concurrent.Semaphore` 将连接包装起来。有了这个改变，虚拟线程会被阻塞在信号量上，并且至关重要的是，释放的是承载线程而不是深入到 c3p0 中。因此，我们永远不会在 c3p0 内部阻塞，因为只有在有可用的空闲会话时我们才会进入 c3p0 代码。

# 总结

这是弗雷德·布鲁克斯（Fred Brooks）所著《人月神话》的封面艺术。这本书封面艺术的版权被认为属于出版商 Addison-Wesley，或者封面艺术家。

看起来尽管软件开发在几十年的进步，还是[没有银弹](https://en.wikipedia.org/wiki/No_Silver_Bullet)。然而，Java 21 虚拟线程是一个非凡的功能，如果小心使用，将带来重大的好处：即使并发性很高，编写高效的异步代码也很容易。
