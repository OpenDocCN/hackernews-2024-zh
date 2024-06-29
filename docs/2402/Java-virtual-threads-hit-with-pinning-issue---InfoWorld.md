<!--yml

category: 未分类

date: 2024-05-29 13:22:16

-->

# Java虚拟线程遭遇“固定”问题 | InfoWorld

> 来源：[https://www.infoworld.com/article/3713220/java-virtual-threads-hit-with-pinning-issue.html](https://www.infoworld.com/article/3713220/java-virtual-threads-hit-with-pinning-issue.html)

Java的[virtual threads](https://www.infoworld.com/article/3678148/intro-to-virtual-threads-a-new-approach-to-java-concurrency.html)，于2023年9月在[JDK 21](https://www.infoworld.com/article/3689880/jdk-21-the-new-features-in-java-21.html)中引入，旨在更轻松地编写和维护并发应用程序，但在同步方法或同步语句中出现了“固定”问题。

Oracle本周在[Inside Java网站](https://inside.java/)详细说明了虚拟线程固定问题。两种最常见的情况涉及虚拟线程在同步方法中停车，以及虚拟线程在进入同步方法时阻塞，因为对象的关联监视器被另一个线程持有。在这两种情况下，承载或本地线程未被释放以执行其他工作。虚拟线程固定可能会影响性能和可伸缩性，并潜在导致饥饿和死锁，根据该博客文章。

Java的Project Loom的新早期访问版本引入了对对象监视器实现的更改，这两种常见情况下不再固定。Loom团队请求用户帮助测试使用虚拟线程和大量同步库的代码来评估这些更新的对象监视器的可靠性和性能。要报告问题，开发人员应使用[Loom邮件列表](https://mail.openjdk.org/pipermail/loom-dev/)。

Project Loom是OpenJDK项目，旨在开发支持轻量级并发的JVM特性和API。根据Oracle的说法，在[JDK 19](https://www.infoworld.com/article/3653331/jdk-19-the-new-features-in-java-19.html)和[JDK 20](https://www.infoworld.com/article/3676699/jdk-20-the-new-features-in-java-20.html)中预览，虚拟线程是轻量级线程，大大降低了编写、维护和观察高吞吐量并发应用程序的工作量。尽管存在固定问题，Oracle表示虚拟线程在Java社区和生态系统中受到了极大的欢迎。

版权所有 © 2024 IDG Communications, Inc.
