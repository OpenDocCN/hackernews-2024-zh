<!--yml

category: 未分类

日期：2024-05-27 15:04:14

-->

# TCP 谜题 | Triton DataCenter

> 来源：[https://www.tritondatacenter.com/blog/tcp-puzzlers](https://www.tritondatacenter.com/blog/tcp-puzzlers)

曾经有人说过，我们真正理解一个系统是在理解它如何失败时。尽管在大学时编写了一个（玩具级的）TCP 实现，并在行业中工作了几年，我仍在深入学习 TCP 如何工作 —— 以及它如何失败。最令人惊讶的是，一些这些故障是多么基础。我将它们呈现为谜题，以 [Car Talk](http://www.cartalk.com/content/puzzlers) 和旧的 [Java 谜题](https://www.youtube.com/watch?v=wbp-3BJWsU8) 的风格。像这些谜题中最好的一样，这些问题非常简单明了，但解决方案往往令人惊讶。与其专注于神秘的细节，它们希望能阐明 TCP 如何运作的一些深刻原则。

## 先决条件

这些谜题假设您对在类 Unix 系统上使用 TCP 有一些基本的了解，但您在深入学习之前无需掌握这些知识。作为复习：

+   TCP 状态、用于建立连接的三次握手，以及连接终止的方式在 [TCP 维基百科页面](https://zh.wikipedia.org/wiki/%E6%94%B6%E9%9B%86%E5%9C%A8%E5%82%B3%E8%BE%93%E6%8E%A7%E5%88%B6%E5%8D%8F%E8%AE%AE%E4%B8%8A%E7%9A%84%E7%8A%B6%E6%80%81) 上有相当简洁的描述。

+   程序通常使用 `read`、`write`、`connect`、`bind`、`listen` 和 `accept` 与套接字交互。还有 `send` 和 `recv`，但对于我们的目的，它们与 `read` 和 `write` 的工作方式相同。

+   我将讨论使用 `poll` 的程序。虽然大多数系统使用像 `kqueue`、事件端口或 `epoll` 这样更高效的东西，但在我们的目的上，它们是等效的。至于使用阻塞操作而不是任何这些机制的应用程序：一旦您了解了 TCP 失败模式如何影响 `poll`，理解它如何影响阻塞操作也是非常容易的。

您可以自行尝试所有这些示例。我使用了两台在 VMware Fusion 下运行的虚拟机。结果与我们生产系统中的经验相符。我在 SmartOS 上使用 `nc(1)` 工具进行测试，并且我认为这里展示的行为不依赖于操作系统。我正在使用 illumos 特有的 [truss(1)](http://illumos.org/man/truss) 工具来跟踪系统调用并获取一些粗略的时间信息。您可以尝试在 OS X 上使用 [dtruss(1m)](https://developer.apple.com/legacy/library/documentation/Darwin/Reference/ManPages/man1/dtruss.1m.html)，或者在 GNU/Linux 上使用 [strace(1)](http://linux.die.net/man/1/strace) 来获取类似的信息。

`nc(1)` 是一个相当简单的工具。我们将以两种模式使用它：

+   作为服务器。在这种模式下，`nc` 将设置一个监听套接字，调用 `accept`，并阻塞直到接收到一个连接。

+   作为客户端。在这种模式下，`nc` 将创建一个套接字并与远程服务器建立连接。

在两种模式下，一旦连接，每一方都使用 `poll` 等待 stdin 或已连接的套接字准备好读取数据。接收的数据打印到终端。您在终端输入的数据发送到套接字。按下 CTRL-C，关闭套接字并退出进程。

在这些示例中，我的客户端称为 `kang`，服务器称为 `kodos`。

## 热身：正常的 TCP 断开连接

这个示例演示了一个非常基本的情况，只是为了开个头。假设我们在 `kodos` 上设置了一个服务器：

服务器

```
[root@kodos ~]# truss -d -t bind,listen,accept,poll,read,write nc -l -p 8080Base time stamp:  1464310423.7650  [ Fri May 27 00:53:43 UTC 2016 ] 0.0027 bind(3, 0x08065790, 32, SOV_SOCKBSD)            = 0 0.0028 listen(3, 1, SOV_DEFAULT)                       = 0accept(3, 0x08047B3C, 0x08047C3C, SOV_DEFAULT, 0) (sleeping...)
```

（请记住，在这些示例中，我使用 `truss` 打印出 `nc` 进行的系统调用。`-d` 标志打印相对时间戳，`-t` 标志选择我们想要看到的系统调用。）

现在在 `kang` 上，我建立一个连接：

客户端

```
[root@kang ~]# truss -d -t connect,pollsys,read,write,close nc 10.88.88.140 8080Base time stamp:  1464310447.6295  [ Fri May 27 00:54:07 UTC 2016 ]... 0.0062 connect(3, 0x08066DD8, 16, SOV_DEFAULT)         = 0pollsys(0x08045670, 2, 0x00000000, 0x00000000) (sleeping...)
```

在 kodos 上，我们看到：

服务器

```
23.8934 accept(3, 0x08047B3C, 0x08047C3C, SOV_DEFAULT, 0) = 4pollsys(0x08045680, 2, 0x00000000, 0x00000000) (sleeping...)
```

此时，TCP 连接处于 ESTABLISHED 状态，并且两个进程都在 `poll`。我们可以使用每个系统上的 `netstat` 工具查看：

服务器

```
[root@kodos ~]# netstat -f inet -P tcp -nTCP: IPv4   Local Address        Remote Address    Swind Send-Q Rwind Recv-Q    State–––––––––––––––––––– –––––––––––––––––––– ––––– –––––- ––––– –––––- ––––––––––-10.88.88.140.8080    10.88.88.139.33226   1049792      0 1049800      0 ESTABLISHED...
```

客户端

```
[root@kang ~]# netstat -f inet -P tcp -nTCP: IPv4   Local Address        Remote Address    Swind Send-Q Rwind Recv-Q    State–––––––––––––––––––– –––––––––––––––––––– ––––– –––––- ––––– –––––- ––––––––––-10.88.88.139.33226   10.88.88.140.8080    32806      0 1049800      0 ESTABLISHED...
```

问题是，**当我们关闭其中一个进程时，另一个进程会发生什么？** 它会发现吗？如何发现？尝试预测特定的系统调用行为，并解释每个系统调用为什么会这样做。

让我们试试。我将 CTRL-C 服务器在 kodos 上：

服务器

```
pollsys(0x08045680, 2, 0x00000000, 0x00000000) (sleeping...)^C127.6307          Received signal #2, SIGINT, in pollsys() [default]
```

这是我们在 `kang` 上看到的内容：

客户端

```
pollsys(0x08045670, 2, 0x00000000, 0x00000000) (sleeping...)126.1771        pollsys(0x08045670, 2, 0x00000000, 0x00000000)  = 1126.1774        read(3, 0x08043670, 1024)                       = 0126.1776        close(3)                                        = 0[root@kang ~]# 
```

发生了什么？让我们具体说明：

1.  我们向服务器发送了 SIGINT 信号，导致进程退出。退出时关闭文件描述符。

1.  当关闭一个 `ESTABLISHED` 套接字的最后一个文件描述符时，kodos 上的 TCP 栈通过连接发送 FIN，并进入 `FIN_WAIT_1` 状态。

1.  kang 上的 TCP 栈接收到 FIN 数据包，将自己的连接转换为 `CLOSE_WAIT`，并发送 ACK 回去。由于 `nc` 客户端在准备读取此套接字时被阻塞，内核使用 `POLLIN` 事件唤醒此线程。

1.  `nc` 客户端在套接字上看到 `POLLIN`，调用 `read`，立即返回 0。这表示流结束。`nc` 假定我们已完成对该套接字的使用并关闭它。

1.  与此同时，kodos 上的 TCP 栈接收到 ACK 并进入 `FIN_WAIT_2` 状态。

1.  由于 kang 上的 `nc` 客户端关闭了它的套接字，kang 上的 TCP 栈发送 FIN 到 kodos。连接在 kang 上进入 `LAST_ACK` 状态。

1.  kodos 上的 TCP 栈接收到 FIN，连接进入 `TIME_WAIT` 状态，并且 kodos 上的栈确认了 FIN。

1.  kang 上的 TCP 栈接收到 FIN 的 ACK，并完全移除该连接。

1.  两分钟后，kodos 上的 TCP 连接关闭，栈完全移除该连接。

（步骤可能会稍微重新排序，并且 kang 可能会通过 `CLOSING` 而不是 `FIN_WAIT_2` 进行转换。）

根据 netstat，这是最终的状态：

服务器

```
[root@kodos ~]# netstat -f inet -P tcp -nTCP: IPv4   Local Address        Remote Address    Swind Send-Q Rwind Recv-Q    State–––––––––––––––––––– –––––––––––––––––––– ––––– –––––- ––––– –––––- ––––––––––-10.88.88.140.8080    10.88.88.139.33226   1049792      0 1049800      0 TIME_WAIT
```

在 kang 上对于此连接根本没有输出。

中间状态转换非常快，但您可以使用[DTrace TCP提供程序](http://dtracebook.com/index.php/Network_Lower_Level_Protocols:tcpstate.d)来查看它们。您可以使用[snoop(1m)](http://illumos.org/man/snoop)或[tcpdump(1)](http://www.tcpdump.org/tcpdump_man.html)来查看数据包流量。

**结论：** 我们已经看到了连接建立和关闭的系统调用的正常路径。请注意，当kodos的连接关闭时，kang立即发现了这一点——它从`poll`中醒来，并且`read`返回0表示流的结束。此时，kang *选择* 关闭它的套接字，从而清理了kodos上的连接状态。稍后我们会重新讨论这个问题，看看如果kang在这里不关闭它的套接字，结果会如何不同。

## 谜题1：电源循环

**如果一个系统被电源循环，已建立但空闲的TCP连接会发生什么？**

由于大多数进程在优雅重启时都会通过正常退出（例如，如果使用"reboot"命令），因此如果您在kodos上键入"reboot"而不是用CTRL-C杀死服务器，您基本上会得到相同的结果。但是，如果我们在前面的例子中对kodos进行了电源循环，会发生什么？毫无疑问，kang最终会发现，对吧？

让我们试试。建立连接：

服务器

```
[root@kodos ~]# truss -d -t bind,listen,accept,poll,read,write nc -l -p 8080Base time stamp:  1464312528.4308  [ Fri May 27 01:28:48 UTC 2016 ] 0.0036 bind(3, 0x08065790, 32, SOV_SOCKBSD)            = 0 0.0036 listen(3, 1, SOV_DEFAULT)                       = 0 0.2518 accept(3, 0x08047B3C, 0x08047C3C, SOV_DEFAULT, 0) = 4pollsys(0x08045680, 2, 0x00000000, 0x00000000) (sleeping...)
```

客户端

```
[root@kang ~]# truss -d -t open,connect,pollsys,read,write,close nc 10.88.88.140 8080Base time stamp:  1464312535.7634  [ Fri May 27 01:28:55 UTC 2016 ]... 0.0055 connect(3, 0x08066DD8, 16, SOV_DEFAULT)         = 0pollsys(0x08045670, 2, 0x00000000, 0x00000000) (sleeping...)
```

现在我将使用VMware的"reboot"功能来对系统进行电源循环。重要的是这必须是一个真正的电源循环（或者操作系统崩溃）——任何导致优雅关闭的操作看起来更像上面的第一种情况。

20分钟后，kang仍然坐在原地：

客户端

```
pollsys(0x08045670, 2, 0x00000000, 0x00000000) (sleeping...)
```

我们倾向于认为TCP的工作是在多个系统之间维护一致的抽象（即TCP连接），所以发现这种抽象被打破的情况让人感到惊讶。而且，如果你认为这是nc(1)的问题，那就错了。在kodos上的"netstat"显示没有连接到kang，但kang显示与kodos之间有一个完全健康的连接：

客户端

```
[root@kang ~]# netstat -f inet -P tcp -nTCP: IPv4   Local Address        Remote Address    Swind Send-Q Rwind Recv-Q    State–––––––––––––––––––– –––––––––––––––––––– ––––– –––––- ––––– –––––- ––––––––––-10.88.88.139.50277   10.88.88.140.8080    32806      0 1049800      0 ESTABLISHED...
```

我们可以永远不碰它，kang *永远* 不会发现kodos已经重启。

**现在，假设此时kang试图向kodos发送数据。会发生什么？**

让我们试试：

客户端

```
pollsys(0x08045670, 2, 0x00000000, 0x00000000) (sleeping...)kodos, are you there?3872.6918       pollsys(0x08045670, 2, 0x00000000, 0x00000000)  = 13872.6920       read(0, " k o d o s ,   a r e   y".., 1024)     = 223872.6924       write(3, " k o d o s ,   a r e   y".., 22)      = 223872.6932       pollsys(0x08045670, 2, 0x00000000, 0x00000000)  = 13872.6932       read(3, 0x08043670, 1024)                       Err#131 ECONNRESET3872.6933       close(3)                                        = 0[root@kang ~]#
```

当我键入消息并按回车键时，kodos被唤醒，从标准输入读取消息，并通过套接字发送它。*`write`调用成功完成!* `nc`回到`poll`等待另一个事件，最终发现可以无阻塞地读取套接字，然后调用`read`。这次，`read`失败，返回`ECONNRESET`。这意味着什么？[POSIX关于read(2)的定义](http://pubs.opengroup.org/onlinepubs/009695399/functions/read.html)说这意味着：

```
[ECONNRESET]A read was attempted on a socket and the connection was forcibly closed by its peer.
```

[illumos read(2) man page](http://illumos.org/man/read.2)提供了更多的上下文：

```
 ECONNRESET              The filedes argument refers to a connection oriented              socket and the connection was forcibly closed by the peer              and is no longer valid.  I/O can no longer be performed to              filedes.
```

这个错误不是指`read`调用本身出错，而是套接字本身已经断开。此时大多数其他套接字操作也会以同样的方式失败。

那么这里发生了什么？在 kang 上的 `nc` 尝试发送数据时，TCP 栈仍然不知道连接已中断。kang 发送了一个数据包到 kodos，后者因为对连接一无所知而回应了 RST。kang 看到了这个 RST 并关闭了它的连接。它不能关闭套接字文件描述符 —— 因为文件描述符不是这样工作的 —— 但是在 `nc` 关闭文件描述符之前，后续操作会因 ECONNRESET 而失败。

**结论：**

1.  强制断电和优雅关闭非常不同。在测试分布式系统时，这种情况需要专门测试。不能简单地终止一个进程并期望它测试相同的事情。

1.  **存在一些情况，其中一方认为 TCP 连接已建立，但另一方并未这样认为，并且这种情况永远不会自动解决。** 可以通过 TCP 或应用级别的保活来解决这个问题。

1.  kang 最终发现远程端消失的唯一原因是它采取了行动：发送数据并接收到响应，指示连接已断开。

这引发了一个问题：如果 kodos 由于某种原因没有响应数据消息会发生什么？

## 谜题 2：关闭电源

**如果 TCP 连接的一个端点长时间关闭电源会发生什么？** 另一个端点是否会发现？如果会，如何发现？何时发现？

再次，我会用 `nc` 建立连接：

服务器

```
[root@kodos ~]# truss -d -t bind,listen,accept,poll,read,write nc -l -p 8080Base time stamp:  1464385399.1661  [ Fri May 27 21:43:19 UTC 2016 ] 0.0030 bind(3, 0x08065790, 32, SOV_SOCKBSD)        = 0 0.0031 listen(3, 1, SOV_DEFAULT)           = 0accept(3, 0x08047B3C, 0x08047C3C, SOV_DEFAULT, 0) (sleeping...) 6.5491 accept(3, 0x08047B3C, 0x08047C3C, SOV_DEFAULT, 0) = 4pollsys(0x08045680, 2, 0x00000000, 0x00000000) (sleeping...)
```

客户端

```
[root@kang ~]# truss -d -t open,connect,pollsys,read,write,close nc 10.88.88.140 8080Base time stamp:  1464330881.0984  [ Fri May 27 06:34:41 UTC 2016 ]... 0.0057 connect(3, 0x08066DD8, 16, SOV_DEFAULT)         = 0pollsys(0x08045670, 2, 0x00000000, 0x00000000) (sleeping...)
```

现在，我会突然切断 kodos 的电源，并尝试从 kang 发送数据：

客户端

```
pollsys(0x08045670, 2, 0x00000000, 0x00000000) (sleeping...)114.4971        pollsys(0x08045670, 2, 0x00000000, 0x00000000)  = 1114.4974        read(0, "\n", 1024)                             = 1114.4975        write(3, "\n", 1)                               = 1pollsys(0x08045670, 2, 0x00000000, 0x00000000) (sleeping...)
```

`write` 操作正常完成，但有一段时间没有看到任何内容。稍后超过 5 分钟，我看到了这个：

客户端

```
pollsys(0x08045670, 2, 0x00000000, 0x00000000) (sleeping...)425.5664        pollsys(0x08045670, 2, 0x00000000, 0x00000000)  = 1425.5665        read(3, 0x08043670, 1024)                       Err#145 ETIMEDOUT425.5666        close(3)                                        = 0
```

这看起来与系统重新启动而非关机的情况相似，除了两点：系统注意到需要花费 5 分钟，而且报告的错误是 ETIMEDOUT。请注意，这并不是说 `read` 函数超时了。我们在其他套接字操作中也会看到同样的错误，并且随后的套接字操作可能会立即失败，并显示相同的 ETIMEDOUT 错误。这是因为*套接字*进入了一个状态，其中底层连接已超时。具体原因是远程端太长时间未确认数据包 —— 在这个系统上配置为 5 分钟。

**结论：**

1.  当远程系统关机而不是重新启动时，首个系统只有在尝试发送数据时才会发现。如果不发送数据包，则永远无法发现已终止的连接。

1.  当系统试图发送数据足够长时间而没有从远程端收到确认时，TCP 连接将被终止，并且在它们上的套接字操作将因 ETIMEDOUT 而失败。

## 谜题 3：没有失败的断开连接

这一次，与其给你一个具体案例并询问发生了什么，我来反过来：这里有一个观察，看看你能否弄清楚它如何发生。我们讨论过几种情况，其中 kang 可能认为自己与 kodos 建立了连接，但 kodos 并不知情。**kang 是否可能与 kodos 建立连接，而 kodos 却毫不知情，可以持续无限期（即，这不会自行解决），即使在 kodos 没有断电、电源循环或操作系统的其他故障以及网络设备故障的情况下？**

这里有个提示：考虑上面一个连接卡在 `ESTABLISHED` 的情况。在这种情况下，可以说应用程序有责任处理问题，因为按定义它仍然保持套接字打开，并且可以通过发送数据发现这一点，此时连接最终将被终止。但是如果应用程序不再拥有套接字呢？

在热身阶段，我们看了 kodos 的 `nc` 关闭了其套接字的情况，并且我们说 kang 的 `nc` 读取了 0（指示流的结束），然后关闭了其套接字。如果它没有关闭套接字，而是保持打开状态呢？显然，它不能从中读取数据。但 TCP 没有任何规定说你不能向已发送 FIN 的另一端发送更多数据。**FIN 只表示数据流在发送 FIN 的方向上结束。**

为了证明这一点，我们不能在 kang 上使用 `nc`，因为当它读取 0 时会自动关闭其套接字。因此，我编写了一个名为 [dnc](https://github.com/davepacheco/experiment-dnc) 的 `nc` 的演示版本，它简单地跳过了这种行为。它还明确打印出它正在进行的系统调用。这将让我们有机会观察 TCP 的状态。

首先，我们将像往常一样建立连接：

服务器

```
[root@kodos ~]# truss -d -t bind,listen,accept,poll,read,write nc -l -p 8080Base time stamp:  1464392924.7841  [ Fri May 27 23:48:44 UTC 2016 ] 0.0028 bind(3, 0x08065790, 32, SOV_SOCKBSD)            = 0 0.0028 listen(3, 1, SOV_DEFAULT)                       = 0accept(3, 0x08047B2C, 0x08047C2C, SOV_DEFAULT, 0) (sleeping...) 1.9356 accept(3, 0x08047B2C, 0x08047C2C, SOV_DEFAULT, 0) = 4pollsys(0x08045670, 2, 0x00000000, 0x00000000) (sleeping...)
```

客户端

```
[root@kang ~]# dnc 10.88.88.140 80802016-05-27T08:40:02Z: establishing connection2016-05-27T08:40:02Z: connected2016-05-27T08:40:02Z: entering poll()
```

现在让我们验证预期的连接状态 `ESTABLISHED` 在双方都正常：

服务器

```
[root@kodos ~]# netstat -f inet -P tcp -nTCP: IPv4   Local Address        Remote Address    Swind Send-Q Rwind Recv-Q    State–––––––––––––––––––– –––––––––––––––––––– ––––– –––––- ––––– –––––- ––––––––––-10.88.88.140.8080    10.88.88.139.37259   1049792      0 1049800      0 ESTABLISHED
```

客户端

```
[root@kang ~]# netstat -f inet -P tcp -nTCP: IPv4   Local Address        Remote Address    Swind Send-Q Rwind Recv-Q    State–––––––––––––––––––– –––––––––––––––––––– ––––– –––––- ––––– –––––- ––––––––––-10.88.88.139.37259   10.88.88.140.8080    32806      0 1049800      0 ESTABLISHED
```

现在，让我们在 kodos 上 CTRL-C 终止 `nc` 进程：

服务器

```
pollsys(0x08045670, 2, 0x00000000, 0x00000000) (sleeping...)^C[root@kodos ~]# 
```

我们立即在 kang 上看到了这个：

客户端

```
2016-05-27T08:40:12Z: poll returned events 0x0/0x12016-05-27T08:40:12Z: reading from socket2016-05-27T08:40:12Z: read end-of-stream from socket2016-05-27T08:40:12Z: read 0 bytes from socket2016-05-27T08:40:12Z: entering poll()
```

现在让我们看看 TCP 连接状态：

服务器

```
[root@kodos ~]# netstat -f inet -P tcp -nTCP: IPv4   Local Address        Remote Address    Swind Send-Q Rwind Recv-Q    State–––––––––––––––––––– –––––––––––––––––––– ––––– –––––- ––––– –––––- ––––––––––-10.88.88.140.8080    10.88.88.139.37259   1049792      0 1049800      0 FIN_WAIT_2
```

客户端

```
[root@kang ~]# netstat -f inet -P tcp -nTCP: IPv4   Local Address        Remote Address    Swind Send-Q Rwind Recv-Q    State–––––––––––––––––––– –––––––––––––––––––– ––––– –––––- ––––– –––––- ––––––––––-10.88.88.139.37259   10.88.88.140.8080    1049792      0 1049800      0 CLOSE_WAIT
```

这是有道理的：kodos 发送了一个 FIN 给 kang。`FIN_WAIT_2` 表示 kodos 收到了 kang 对其发送的 FIN 的 ACK，而 `CLOSE_WAIT` 表示 kang 收到了 FIN *但尚未发送 FIN 回来*。**对于 TCP 连接而言，这是一个完全有效的状态，可以持续无限期。** 想象一下，kodos 向 kang 发送了一个请求，不再计划发送任何其他请求；kang 可以愉快地在数小时内响应数据，这是可行的。只是在我们的情况下，kodos *确实*已经关闭了套接字。

让我们等一分钟，然后再次检查 TCP 连接状态。一分钟后，在 kodos 上连接完全消失，但在 kang 上仍然存在：

客户端

```
[root@kang ~]# netstat -f inet -P tcp -nTCP: IPv4   Local Address        Remote Address    Swind Send-Q Rwind Recv-Q    State–––––––––––––––––––– –––––––––––––––––––– ––––– –––––- ––––– –––––- ––––––––––-10.88.88.139.37259   10.88.88.140.8080    1049792      0 1049800      0 CLOSE_WAIT
```

这里发生了什么？我们遇到了 TCP 栈中一个不太为人知的特殊情况：当应用程序关闭了一个套接字后，TCP 栈发送了一个 FIN，远程 TCP 栈确认了 FIN，本地 TCP 栈等待了一个固定的时间，然后 *关闭连接*。为什么要这样做呢？以防远程端重新启动。这种情况实际上类似于上面一个情况，其中一方有一个 `ESTABLISHED` 连接，而另一方却不知道。不同之处在于，在这种特定情况下，应用程序已经关闭了套接字，因此没有其他组件可以处理这个问题。结果是，TCP 栈等待了一段固定的时间，然后关闭连接（而不向另一端发送任何内容）。

跟进问题：**如果 kang 在这一点向 kodos 发送数据会发生什么？**请记住，kang 仍然认为连接是打开的，但在 kodos 上已经被拆除。

客户端

```
2016-05-27T08:40:12Z: entering poll()kodos, are you there?2016-05-27T08:41:34Z: poll returned events 0x1/0x02016-05-27T08:41:34Z: reading from stdin2016-05-27T08:41:34Z: writing 22 bytes read from stdin to socket2016-05-27T08:41:34Z: entering poll()2016-05-27T08:41:34Z: poll returned events 0x0/0x102016-05-27T08:41:34Z: reading from socketdnc: read: Connection reset by peer
```

这与我们在谜题1中看到的情况相同：`write()` 实际上成功了，因为 TCP 栈还不知道连接已经关闭。但它确实收到了一个 RST，这会唤醒 `poll()` 中的线程，随后的 `read()` 返回 ECONNRESET。

**结论：**

+   甚至在没有任何操作系统、网络或硬件故障的情况下，两个方面对于连接的状态可能会产生分歧。

+   在像上面那种情况下，kang 不可能区分 kodos 是在专心等待接收数据，还是 kodos 已经关闭了套接字而不再监听（至少，没有发送数据包的情况下）。因此，也许设计一个在正常操作期间使用这些半开放状态的套接字的系统并不是一个好主意。

## 结论

TCP 通常被描述为在两个系统之间维护一致的抽象——"TCP 连接"——的协议。我们知道，在面对某些类型的网络和软件问题时，连接会失败，但并不总是明显地看到抽象失败的情况，即两个系统在连接状态上有分歧。具体而言：

+   有可能一个系统认为它与远程系统有一个正常工作的已建立的 TCP 连接，而远程系统对这个连接一无所知。

+   即使在没有网络、硬件或操作系统故障的情况下，这种情况也是可能的。

这些行为并不反映 TCP 的缺陷。恰恰相反，在所有这些情况中，TCP 的实现似乎都表现得尽可能合理。如果我们试图实现自己的通信机制而不使用 TCP，这些情况的存在可能会提醒我们底层问题的复杂性。这些都是分布式系统的固有问题，而 TCP 连接本质上就是一个分布式系统。

尽管如此，所有这些中最重要的一课是**跨多个系统的TCP连接这一概念是一个方便的虚构**。当出现问题时，关键是将其视为两个独立的状态机，它们同时操作以尝试维护连接的一致视图。应用程序有责任处理这些不同之处（通常使用保持活动机制）。

此外，应用程序的文件描述符与底层的TCP连接之间存在断开。即使应用程序关闭了文件描述符，TCP连接（与关闭相关的各种状态）仍然存在，并且由于故障，当底层TCP连接关闭时文件描述符可以打开。

其他需要牢记的教训包括：

+   系统的非优雅重新启动（当操作系统崩溃时会发生）与用户进程退出或关闭其套接字不同。特别是要测试这种情况。在远程系统在连接超时之前重新上线时重启，与关机也是不同的。

+   当TCP套接字关闭时，内核没有主动通知。你只有在对套接字文件描述符调用 `read()`、`write()` 或其他套接字操作时才能发现这一点。如果由于某种原因你的程序没有这样做，你将永远不会发现连接失败。

一些相关的笔记，我发现并不那么常见：

+   `ECONNRESET` 是一个套接字错误，你可以在 `read()`、`write()` 和其他操作中看到，表明远程对等方已发送RST。

+   `ETIMEDOUT` 是一个套接字错误，你可以在 `read()`、`write()` 和其他操作中看到，表明连接相关的某个超时已经过去。我见过的情况大多数是远程端太长时间没有确认某个数据包。这些通常是数据包、FIN包或者KeepAlive探测。

更重要的是，这些错误都不意味着你尝试做的读取或写入操作有任何问题 — 只是套接字本身已关闭。

如果你已经阅读到这里，并且这类问题听起来对你有趣，[我们正在招聘](https://www.joyent.com/about/careers)!

*Dave Pacheco 撰写*
