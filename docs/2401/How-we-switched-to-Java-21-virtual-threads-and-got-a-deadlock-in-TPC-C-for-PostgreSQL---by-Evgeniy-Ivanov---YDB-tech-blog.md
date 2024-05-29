<!--yml
category: 未分类
date: 2024-05-27 14:51:05
-->

# How we switched to Java 21 virtual threads and got a deadlock in TPC-C for PostgreSQL | by Evgeniy Ivanov | YDB.tech blog

> 来源：[https://blog.ydb.tech/how-we-switched-to-java-21-virtual-threads-and-got-deadlock-in-tpc-c-for-postgresql-cca2fe08d70b](https://blog.ydb.tech/how-we-switched-to-java-21-virtual-threads-and-got-deadlock-in-tpc-c-for-postgresql-cca2fe08d70b)

# **How we switched to Java 21 virtual threads and got a deadlock in TPC-C for PostgreSQL**

Dining Java 21 philosophers have a problem

In our previous [post](/ydb-meets-tpc-c-distributed-transactions-performance-now-revealed-42f1ed44bd73) about TPC-C, we discussed some drawbacks in the original TPC-C implementation from the [Benchbase](https://github.com/cmu-db/benchbase) project (which is great nevertheless). One of the drawbacks was the concurrency limit due to spawning too many physical threads, and we solved it by switching to Java 21 virtual threads. Later we discovered that, as usual, there is no free lunch. In this post, we present a case study on how we encountered a deadlock with virtual threads in TPC-C for PostgreSQL, even without the dining philosophers problem.

This post might be useful for Java developers who are considering switching to virtual threads. We revise some fundamental background and then highlight an important concern behind virtual threads: deadlocks might be unpredictable because they could happen deep inside the libraries you use. Fortunately, debugging is straightforward and we explain how to find these deadlocks when they happen.

# Why are we talking about PostgreSQL in the YDB blog

PostgreSQL is an open-source database management system renowned for its high performance, rich feature set, advanced level of SQL compliance, and vibrant and supportive community. It’s great until you take into consideration horizontal scalability and fault tolerance. Then you end up with PostgreSQL-based third-party solutions like Citus, which implement sharded PostgreSQL. Having a single elephant might be fun. Being a mahout of a herd of elephants is a challenge, especially if you want these elephants to maintain multiple consistent replicas and perform distributed transactions with serializable isolation.

As opposed to this, [YDB](https://ydb.tech/) is a distributed database management system by its original design. YDB’s distributed transactions are first-class citizens and run at a serializable isolation level by default. Now, we are actively moving towards PostgreSQL compatibility because we see strong demand among PostgreSQL users to make their existing applications automatically scalable and fault-tolerant. That’s why we maintain [TPC-C for PostgreSQL](https://github.com/ydb-platform/tpcc) (we hope to get it merged into upstream Benchbase soon).

# A very short background and motivation

Let’s recap some fundamental concepts: concurrency, parallel execution, and asynchronous vs. synchronous requests.

Concurrency means that tasks are performed at the same time, either in parallel or sequentially. For example, you might have two activities: writing your code in an editor and having a Slack chat with your colleagues. You perform these tasks concurrently, but not in parallel. Or you might take a walk with your dog and speak on the phone with a friend. Again, you perform these two tasks concurrently, but this time, in parallel.

Now, consider the case when your application wants to make a request to the database. The request is sent through the network, serviced by the database, and the reply is sent back to your application. Note that the network round trip might be the most expensive part of the request and could take several milliseconds. What can you do on the application side while waiting for the reply?

1\. The request might be synchronous, i.e., it will block the calling thread. This approach is very easy to write code for: on line 1, you have the request; on line 2, you can process the response:

```
String userName = get_username_from_db(userId);
System.out.printf("Hello, %s!", userName);
```

2\. The request might be asynchronous. Your thread is not blocked and continues the execution, while the request is processed in parallel:

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

In either case, there are two concurrent tasks: your thread is waiting for the reply from the database, and the database is handling the request. Synchronous code is extremely simple to write and read. But what if you need to make thousands of requests to the database simultaneously? You will have to spawn a thread per request. Spawning a thread in Linux is cheap, though there are strong concerns behind spawning too many threads:

1.  Each thread requires a stack. You can’t allocate less memory than the page size on your system, which is usually about 4 KiB unless you use hugepages, where the default page size is 2 MiB.
2.  There is the Linux scheduler. You can try to spawn 100,000 threads ready to execute only if you have a reset button.

This is why, until Java 21, there was no way to write synchronous code with a high level of concurrency: you can’t spawn many threads. Concurrently (pun intended), the Go language revolutionized this: goroutines provide very lightweight concurrency so that you can write synchronous code *efficiently*. We recommend this [talk](https://youtu.be/-K11rY57K7k?si=XT8mkXL2ypmVj4ig) about the Go scheduler by Dmitry Vyukov. Java 21 introduced virtual threads which are in many senses similar to goroutines. Keep in mind that goroutines and virtual threads are not inventions, but rather a reincarnation of the good old concept of user-level threads.

Now, you can understand the problem with synchronous database requests in the original Benchbase TPC-C implementation. If your database can handle a high load, you must run many TPC-C warehouses, spawning many threads. With physical threads, we failed to run more than 30,000 terminal-threads, while with virtual threads, we can easily have hundreds of thousands of terminal-vthreads.

# Deadlocks made easy

Imagine that you already have multithreaded Java code. Adding an option to use virtual threads is surprisingly easy and can be incredibly beneficial. By simply replacing your standard thread creation with the new virtual thread builders, your application can handle thousands of concurrent tasks without the overhead associated with physical threads. Here is an [example](https://github.com/ydb-platform/tpcc/blob/c8474bc7cf40456de0fe4c7fb060d867dd985ede/src/main/java/com/oltpbenchmark/ThreadBench.java#L65-L69) from our TPC-C implementation:

```
if (useRealThreads) {
    thread = new Thread(worker);
} else {
    thread = Thread.ofVirtual().unstarted(worker);
}
```

That’s all it takes; now, you’re using virtual threads. Under the hood, the Java Virtual Machine creates a pool of `carrier threads`, which execute your `virtual threads`. This transition appears seamless until, unexpectedly, your application freezes.

Our PostgreSQL TPC-C implementation utilizes [c3p0](https://www.mchange.com/projects/c3p0/) for connection pooling. [The TPC-C standard](https://www.tpc.org/tpcc/) dictates that each terminal must have its own connection. However, in many real-world scenarios, this isn’t practical, so we’ve included an option to limit the number of database connections.

The number of terminals is much greater than the number of available connections. Consequently, some terminals must wait for a session to become available, i.e., released by another terminal.

When we initiated the TPC-C run, the application froze. Fortunately, debugging such cases is straightforward:

1.  Capture the thread stacks using `jstack <PID>`.
2.  Create a more detailed dump of the current state, which includes information about carrier threads and virtual threads, using `jcmd <PID> Thread.dump_to_file -format=text jcmd.dump.1`.

Upon investigation, we discovered that some virtual threads waiting for a session had pinned their carrier thread. Here is the stack for one such virtual thread:

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

and the stack of its carrying thread:

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

As you can see, the thread is hanging in `Object.wait()`, a method used in conjunction with `synchronized`. This causes the carrier thread to become pinned, meaning it is not released to execute some other virtual thread. Meanwhile, the session holders have released their carrier threads while they are waiting for I/O operations:

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

Thus, we ended up in the following situation:

1.  All carrier threads are pinned by session waiters, meaning there are no carrier threads available.
2.  Virtual threads holding the sessions can’t finish their tasks to release the sessions.

Deadlock made easy!

[JEP 444](https://openjdk.org/jeps/444) states that:

> *There are two scenarios in which a virtual thread cannot be unmounted during blocking operations because it is pinned to its carrier:*
> 
> *When it executes code inside a synchronized block or method, or When it executes a native method or a foreign function.*

The problem is that this synchronized code might be deeply embedded within the libraries you use. In our case, it was within the c3p0 library. So, the [fix](https://github.com/ydb-platform/tpcc/commit/175f0c03d9c16652c85a6103331fec473017797e) is straightforward: we simply wrapped the connection with a `java.util.concurrent.Semaphore`. With this change, virtual threads are blocked on the semaphore and, crucially, release the carrier thread instead of delving inside c3p0\. Thus, we never block inside c3p0 because we enter c3p0 code only when there is a free session available.

# Summing up

This is the front cover art for the book The Mythical Man-Month written by Fred Brooks. The book cover art copyright is believed to belong to the publisher, Addison-Wesley, or the cover artist.

It seems that despite decades of progress in software development, there is still [no silver bullet](https://en.wikipedia.org/wiki/No_Silver_Bullet). Yet, Java 21 virtual threads are a remarkable feature, offering significant benefits if used carefully: it’s very easy to write an efficient async code even when concurrency is high.