<!--yml

类别：未分类

日期：2024-05-27 14:42:10

-->

# epoll：推动现代互联网的 API · Graham King

> 来源：[`darkcoding.net/software/epoll-the-api-that-powers-the-modern-internet/`](https://darkcoding.net/software/epoll-the-api-that-powers-the-modern-internet/)

你使用[epoll](https://man7.org/linux/man-pages/man7/epoll.7.html)来获取这篇博客文章。几乎你在互联网上做的任何事情，服务器都会运行 Linux，并使用`epoll`及时且经济地接收和回应你的请求。

+   epoll 是使 Go 成为编写服务器软件的绝佳语言的原因。以下是 Go 中的 epoll 在[netpoll 中的使用](https://github.com/golang/go/blob/f229e7031a6efb2f23241b5da000c3b3203081d6/src/runtime/netpoll_epoll.go#L101-L126)。

+   epoll 是使 nginx 成为[世界上最受欢迎的 Web 服务器](https://news.netcraft.com/archives/2021/12/22/december-2021-web-server-survey.html)的原因（这个博客正在运行 nginx）。以下是[nginx 对 epoll 的使用](https://github.com/nginx/nginx/blob/a64190933e06758d50eea926e6a55974645096fd/src/event/modules/ngx_epoll_module.c#L784-L800)。

+   在大多数编程语言中，当我们说“异步”时，通常指的是这种情况。例如，Rust 的两个主要异步框架中，async-std 使用[轮询](https://github.com/smol-rs/polling/blob/master/src/epoll.rs#L156-L157)，而 tokio 使用[mio](https://github.com/tokio-rs/mio/blob/dca2134ef355b3c0d00e8e338e44e7d9ed63edac/src/sys/unix/selector/epoll.rs#L68)。

除了`epoll`之外，以上所有工作在许多操作系统上都能运行，并支持其他 API。因为互联网[主要由](https://en.wikipedia.org/wiki/Usage_share_of_operating_systems#Public_servers_on_the_Internet) [Linux](https://w3techs.com/technologies/details/os-unix)组成，所以 epoll 是最重要的 API。

#### 问题

运行网络服务的核心问题，也是 epoll 解决的问题，是你的网络非常快，而客户端的网络非常慢。处理请求的服务器通常是这样的：

```
read the user's request (e.g. a browser HTTP GET)
do what they asked (e.g. load some information from the database)
write a response (e.g. HTML that the browser will display) 
```

在上面的“读”和“写”部分期间，服务器处于空闲状态，等待数据或该数据的确认在网络上移动。

#### 在 epoll 之前

在 epoll 之前，克服这个问题的标准方法是运行一个进程池，每个进程处理一个不同的用户请求，通常使用 Apache 的[mod_prefork](https://httpd.apache.org/docs/2.4/mod/prefork.html)。当一个进程等待用户确认数据包时，另一个进程可以使用 CPU。一个新兴的替代方法是使用线程池，它比进程池轻，可以处理低数百个并发用户。多线程是有风险的，因为许多库不是线程安全的。Steven 在 2004 年的参考书[UNIX 网络编程](https://amzn.to/3FUlV4n)中有一章讨论了 preforked vs prethreaded 设计，因为那时候这是你的选择。

然后所有人都来了，即使是数百个并发用户也不够。一篇有影响力的文章，《The C10K problem》（[英文链接](http://www.kegel.com/c10k.html)）于 1999 年开始讨论这个问题。网络请求超时并不少见。人们会[镜像](https://en.wikipedia.org/wiki/Mirror_site)流行的站点以试图分散流量。

#### 解决方案

2000 年，Jonathan Lemon 为 FreeBSD 4.3 解决了这个问题，设计并构建了[kqueue/kevent](https://people.freebsd.org/~jlemon/papers/kqueue.pdf)，使得 BSD 成为高性能网络的早期选择。

独立地，2001 年 7 月，Davide Libenzi 为 Linux 解决了这个问题，使用了[epoll 的第一个草案](http://www.xmailserver.org/linux-patches/nio-improve.html)，它[发展了](https://lwn.net/Articles/16026/)，并[合并](https://lwn.net/Articles/13481/)到了 Linux 内核 2.5.44（一个开发版本）中，于 2002 年 10 月，并于 2003 年 12 月通过发布稳定的内核 2.6 而普及开来。

Jim Blandy 在这里有一个[线程与基于 epoll 的异步的精彩比较](https://github.com/jimblandy/context-switch)。

#### 工作原理

epoll 允许单个线程或进程注册对一长串网络套接字的兴趣（它支持除网络套接字之外的其他东西，例如管道和终端，但你很少有成千上万个这样的东西）。然后，epoll_wait 调用将阻塞，直到其中一个准备好读取或写入。使用 epoll 的单个线程可以处理数以万计的并发（并且大多数是空闲的）请求。

epoll 的缺点是它改变了应用程序的架构。现在，你不再是用直线代码处理每个连接，例如`{读请求，处理，写响应}`，而是有一个更类似游戏引擎的主循环。代码变成了：

```
loop
 epoll_wait on all the connections
 for each of the ready connections:
   continue from where you left off 
```

你可能正在读取一个已准备好的套接字上的请求的一部分，并且正在写另一个套接字上的响应的一部分。你必须记住你的状态，只做套接字不会阻塞的 I/O，并再次调用 epoll_wait。Go 语言和像 C＃、JavaScript 和 Rust 这样的语言中的'async/await'模型之所以如此受欢迎的一大部分原因是它们隐藏了事件循环，使您能够编写直线代码，就好像您仍在执行每个连接一个线程的操作。

#### 结论

如果没有 epoll，今天的互联网经济将会大不相同（每台机器的请求较少，因此需要更多的机器，成本更高），或者我们会在 BSD 上运行我们的服务器。而没有 BSD 的 kqueue（比 epoll 早两年），我们真的会陷入困境，因为唯一的替代方案是专有的（Solaris 8 中的/dev/poll 和 Windows NT 3.5 中的 I/O 完成端口）。

epoll 自其初始发布以来已经得到了改进，特别是 EPOLLONESHOT 和 EPOLLEXCLUSIVE 标志，但核心 API 保持不变。epoll 解决了 Linux 上的 C10K 问题，它驱动了互联网，使我们能够构建快速而廉价的互联网服务。

感谢 Davide！

[在 Hacker News 上讨论](https://news.ycombinator.com/item?id=38948091)

* * *

### 附录：前身

+   Linux 在`epoll`之前就有`poll`和`select`。它们设计用于处理少量文件描述符，并且在这方面的规模为 O(n)。`epoll`的规模为 O(1)。[Kerrisk](https://amzn.to/3r3IVIf)提供的性能数据显示，`poll`和`select`在数百个文件描述符以上就变得无法使用，而`epoll`在数万个文件描述符上仍然保持快速。

+   Linux 在`epoll`之前也有信号驱动 I/O。引用 [UNIX 网络编程](https://amzn.to/3FUlV4n)：

> 不幸的是，对于 TCP 套接字，信号驱动 I/O 几乎毫无用处。

and

> 作者们能够找到的唯一实际使用信号驱动 I/O 与套接字的例子是 NTP 服务器，它使用 UDP。

在发布前，Davide Libenzi 友善地阅读了这篇文章。
