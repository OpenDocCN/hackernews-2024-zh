<!--yml

category: 未分类

date: 2024-05-27 14:46:46

-->

# SQLite 在 macOS 上：与捆绑版本不符合 ACID 标准

> 来源：[https://bonsaidb.io/blog/acid-on-apple/](https://bonsaidb.io/blog/acid-on-apple/)

# SQLite 在 macOS 上：与捆绑版本不符合 ACID 标准

作者为 [Jonathan Johnson](https://github.com/ecton)。发布于 2022-06-14。

我正在构建 [一个数据库](/about)，我认为 SQLite 是“黄金标准”，可以与我的数据库进行比较。最近在对新代码进行基准测试时，我注意到 Apple 捆绑版本的 SQLite 并不符合 ACID 标准。

我不认为自己在这些问题上是专家。如果我的分析中有任何错误，请联系我，我将立即更正。我通过实践学习，并且正如[我在这个话题上的上一篇文章](/blog/durable-writes)所证明的，我也犯过自己的错误。

2022年2月17日，[Scott Perry 在 Twitter 上的对话中写道：](https://twitter.com/numist/status/1494392674014531593)

> 有第三个同步操作可以让您既提升性能又保证写入顺序：F_BARRIERFSYNC。SQLite 在 Darwin 上已经使用了它，它也是 I/O 降低的最佳实践指南的一部分。[https://developer.apple.com/documentation/xcode/reducing-disk-writes](https://developer.apple.com/documentation/xcode/reducing-disk-writes)

一些人（包括我自己）曾解读过“SQLite 在 Darwin 上已经使用了它”这一说法，认为这是默认行为。我的文章将表明这并非事实。在 macOS 12.4（21F79）中，默认情况下，捆绑版本的 SQLite 使用 `fsync()` 进行同步。

通过我的调查，苹果的 SQLite 版本相反地用 `F_BARRIERFSYNC` 替换了 `PRAGMA fullfsync = on` 的实现。

那些期望 `PRAGMA fullfsync` 提供持久性保证以防止电源故障或内核崩溃的 SQLite 用户可以通过自定义 VFS 来覆盖 xSync，或者从源代码构建 SQLite。

要理解其中的原因，让我们来看看如何确保苹果操作系统上的持久写入。

我们需要涵盖两个 API：`fsync()` 和 `fcntl()`。在 Linux 上，`fsync()` 是系统调用，用于告知内核将其文件状态与磁盘完全同步。关于 `fsync()` 的原始 POSIX 规范是否打算提供这种耐久性保证是值得商榷的，但在 Linux 上，它尽最大努力保证所有更改的位都已同步到磁盘，包括刷新任何受影响的易失性写入缓存。

然而，在 macOS 上，`fsync()` 的 man 页面如下所述：

> 需要注意的是，虽然 `fsync()` 将所有数据从主机刷新到驱动器（即“永久存储设备”），但驱动器本身可能要相当长时间才能将数据写入到盘片上，并且可能以非顺序方式写入。
> 
> 具体来说，如果驱动器失去电源或操作系统崩溃，应用程序可能会发现只有部分或没有数据被写入。磁盘驱动器还可能重新排序数据，使得后续写入可能存在，而先前的写入则不存在。
> 
> 这不是一个理论上的边缘情况。这种情况很容易通过真实的工作负载和驱动器断电来重现。
> 
> 对于需要更严格保证其数据完整性的应用程序，Mac OS X 提供了 F_FULLFSYNC fcntl。F_FULLFSYNC fcntl 要求驱动器将所有缓冲数据刷新到永久存储。需要严格写入顺序的应用程序（例如数据库）应使用 F_FULLFSYNC 来确保其数据按其期望的顺序写入。请参阅 fcntl(2) 获取更多详细信息。

Apple 的文档明确指出，为了防止因电源故障或内核恐慌而造成的数据丢失，必须使用 `fcntl()` API 与 `F_FULLFSYNC` 命令。

回顾今年二月，这个主题相当广泛地传播开来，[Michael Tsai 的这篇文章](https://mjtsai.com/blog/2022/02/17/apple-ssd-benchmarks-and-f_fullsync/) 总结了研究结果。简而言之，指出 `F_FULLFSYNC` 在当前实现中非常慢。文章还指出，苹果在其 ["减少磁盘写入"](https://developer.apple.com/documentation/xcode/reducing-disk-writes) 文章中向用户推荐另一个 `fcntl()` 命令：

> 一些应用程序需要写入障碍以确保数据持久性，然后才能继续进行后续操作。大多数应用程序可以使用 fcntl(*:*:) 的 F_BARRIERFSYNC 来实现这一点。
> 
> 只有在您的应用程序需要对数据持久性有强烈期望时才使用 F_FULLFSYNC。请注意，F_FULLFSYNC 代表 iOS 将数据写入磁盘的尽力保证，但在突然断电情况下仍然可能丢失数据。

`F_BARRIERFSYNC` 发出一个 IO 障碍，所有后续的 IO 操作必须等待所有当前写入成功。`fcntl()` 调用在发出障碍后返回，但在数据完全同步之前。这就是为什么使用 `F_BARRIERFSYNC` 不能满足 ACID 的耐久性要求：更改在数据完全同步之前被确认。

我应该注意到，虽然 `fcntl()` 是 Linux 上可用的 API，但 `F_FULLFSYNC` 和 `F_BARRIERFSYNC` 是特定于 Apple 操作系统的。Linux 不需要这些选项，因为 `fsync()` 提供了所需的保证。

当我启动我的新低级存储层（[Sediment](https://github.com/khonsulabs/sediment)）时，我添加了支持选择在 macOS 上使用`F_BARRIERFSYNC`而不是`F_FULLFSYNC`的选项。默认情况下，我仍然会使用`F_FULLSYNC`，因为我希望用户在需要在苹果硬件上获得额外性能时显式选择退出 ACID。这基于 SQLite 在 macOS 上实现非常快速速度的相同方法。

昨天，我创建了一个简单的基准测试，以查看Sediment的性能当前情况。我还不准备分享具体数字，这也不是本文的重点。然而，总结来说，Sediment在Linux上比SQLite更快，在我的M1 MacBook Air上比SQLite慢。

这让我感到困惑，因为如果SQLite和Sediment都使用相同的同步原语，那么在切换操作系统时，性能差异如何会被颠倒？我决定调查SQLite如何利用`F_BARRIERFSYNC`。

我的第一站是文档。SQLite有一个用于[启用`F_FULLFSYNC`的pragma](http://www3.sqlite.org/pragma.html#pragma_fullfsync)，但我找不到任何关于`F_BARRIERFSYNC`的文档。关于`PRAGMA fullfsync`的文档说明了其默认值为关闭状态。

接下来我查看了SQLite源代码：`full_fsync()`在[os_unix.c](https://sqlite.org/src/file?name=src/os_unix.c&ci=b1be2259)中定义。它的职责是基于可用和配置的选项执行完整的fsync。这一部分与Apple操作系统相关：

```
#elif HAVE_FULLFSYNC  if( fullSync ){
 rc = osFcntl(fd, F_FULLFSYNC, 0);
 }else{
 rc = 1;
 }  /* If the FULLFSYNC failed, fall back to attempting an fsync().
 ** It shouldn't be possible for fullfsync to fail on the local ** file system (on OSX), so failure indicates that FULLFSYNC ** isn't supported for this file system. So, attempt an fsync ** and (for now) ignore the overhead of a superfluous fcntl call. ** It'd be better to detect fullfsync support once and avoid ** the fcntl call every time sync is called. */  if( rc ) rc = fsync(fd);   #elif defined(__APPLE__)
  /* fdatasync() on HFS+ doesn't yet flush the file size if it changed correctly
 ** so currently we default to the macro that redefines fdatasync to fsync */ rc = fsync(fd); 
```

SQLite源代码显示了调用带有`F_FULLFSYNC`的`fcntl()`的实现，但没有提到`F_BARRIERFSYNC`。

有最后一件事需要检查：也许苹果会提供一个定制版本的SQLite，其中使用了`F_BARRIERFSYNC`。验证的最佳方法是使用dtrace记录进程所做的系统调用。

我禁用了系统完整性保护，以便我可以跟踪macOS 12.4（21F79）附带的`sqlite3`可执行文件。

```
~ % sudo dtruss -t fcntl sqlite3 test.sqlite SYSCALL(args)    = return SQLite version 3.37.0 2021-12-09 01:34:53 Enter ".help" for usage hints. fcntl(0x3, 0x5F, 0x1)   = 0 0 fcntl(0x3, 0x3F, 0x6BDBD9C0)   = 3 0 fcntl(0x4, 0x32, 0x16BDBDDA8)   = 0 0   sqlite> insert into test (a) values (1); fcntl(0x3, 0x5A, 0x16BDBD0B8)   = 0 0 fcntl(0x3, 0x5A, 0x16BDBD0B8)   = 0 0 fcntl(0x3, 0x5A, 0x16BDBD0B8)   = 0 0 fcntl(0x3, 0x5A, 0x16BDBCBA8)   = 0 0 fcntl(0x3, 0x5A, 0x16BDBDE98)   = 0 0 fcntl(0x3, 0x5A, 0x16BDBDE98)   = 0 0 fcntl(0x3, 0x5A, 0x16BDBDE98)   = 0 0 fcntl(0x3, 0x5A, 0x16BDBDF08)   = 0 0 fcntl(0x4, 0x5F, 0x1)   = 0 0 fcntl(0x4, 0x3F, 0x1)   = 3 0 fcntl(0x5, 0x5F, 0x1)   = 0 0 fcntl(0x3, 0x5A, 0x16BDBDEE8)   = 0 0 fcntl(0x3, 0x5A, 0x16BDBDEE8)   = 0 0 fcntl(0x5, 0x5F, 0x1)   = 0 0 fcntl(0x3, 0x5A, 0x16BDBDE88)   = 0 0 fcntl(0x3, 0x5A, 0x16BDBDE88)   = 0 0 fcntl(0x3, 0x5A, 0x16BDBDEB8)   = 0 0 
```

此日志显示了SQLite执行`insert into test (a) values (1);`语句时发出的所有`fcntl()`调用。第二个参数是命令。我们可以看到SQLite正在使用命令0x3F、0x5A和0x5F。以十进制表示，这些分别是63、90和95。查看`fcntl.h`，我们看到了这些值：

```
#define F_FULLFSYNC 51 /* fsync + ask the drive to flush to the media */ #define F_GETPROTECTIONCLASS 63 /* Get the protection class of a file from the EA, returns int */ #define F_BARRIERFSYNC 85 /* fsync + issue barrier to drive */ 
```

命令90和95是[私有的](https://github.com/apple/darwin-xnu/blob/2ff845c2e033bd0ff64b5b6aa6063a1f8f65aa32/bsd/sys/fcntl.h#L360-L370)。

```
#define F_OFD_SETLK 90 /* Acquire or release open file description lock */ #define F_SETCONFINED 95 /* "confine" OFD to process */ 
```

我们没有看到任何使用命令参数为85（0x55）的`fcntl()`调用。让我们尝试启用`PRAGMA fullfsync`：

```
sqlite> pragma fullfsync=on; sqlite> insert into test (a) values (1); fcntl(0x3, 0x5A, 0x16DD9DEF8)   = 0 0 fcntl(0x3, 0x5A, 0x16DD9DEF8)   = 0 0 fcntl(0x3, 0x5A, 0x16DD9DEF8)   = 0 0 fcntl(0x3, 0x5A, 0x16DD9DF68)   = 0 0 fcntl(0x4, 0x5F, 0x1)   = 0 0 fcntl(0x4, 0x3F, 0x1)   = 3 0 fcntl(0x3, 0x5A, 0x16DD9DF48)   = 0 0 fcntl(0x3, 0x5A, 0x16DD9DF48)   = 0 0 fcntl(0x4, 0x55, 0x0)   = 0 0 fcntl(0x5, 0x5F, 0x1)   = 0 0 fcntl(0x4, 0x55, 0x0)   = 0 0 fcntl(0x3, 0x55, 0x0)   = 0 0 fcntl(0x3, 0x5A, 0x16DD9DEE8)   = 0 0 fcntl(0x3, 0x5A, 0x16DD9DEE8)   = 0 0 fcntl(0x3, 0x5A, 0x16DD9DF18)   = 0 0 
```

如预期，我们有一个新的`fcntl()`命令：0x55。然而，出乎意料的是，与其通过阅读公开可用的SQLite代码期望启用`F_FULLFSYNC`（0x33），我们看到了0x55，即`F_BARRIERFSYNC`。

总结一下，Apple的SQLite默认情况下不使用`F_BARRIERFSYNC`或`F_FULLFSYNC`，并且在启用`PRAGMA fullfsync`时，将`fcntl(.., F_FULLFSYNC, ..)`替换为`fcntl(.., F_BARRIERFSYNC, ..)`。

当我编辑这篇文章时，[Scott Perry确认了这种行为](https://twitter.com/numist/status/1536830264214638593)。

对于大多数消费者应用程序，`F_BARRIERFSYNC`足以提供合理的持久性，并且具有更快的执行速度。但是，在一些需要真正ACID兼容性的情况下，这并不足够。许多（但并非所有）这些情况涉及服务器软件。

由于苹果不再出货服务器硬件以及`F_FULLFSYNC`在[苹果驱动器上](https://news.ycombinator.com/item?id=30371857)的性能，苹果选择在其版本的SQLite中使用`F_BARRIERFSYNC`是难以指责的决定。我希望他们选择了另一种方式，比如新的编译指示或更改默认的`fsync()`行为，而不是替换`PRAGMA fullfsync`。

当一个特性被[文档化为仅适用于macOS](http://www3.sqlite.org/pragma.html#pragma_fullfsync)时，在macOS上的行为与文档描述不一致时，这非常令人困惑。目前来看，如果开发人员希望获取文档化的行为，最简单的方式可能是从源代码构建SQLite。

我没有在iOS上测试这些发现 -- 多年来，我没有尝试在设备上进行任何追踪。我怀疑苹果不会为iOS和macOS维护单独的SQLite版本，但由于他们的SQLite版本是闭源的，我们无法轻易验证。

不论苹果将来如何改变SQLite的同步方式，我鼓励苹果在发布SQLite更新时与其其他开源存储库一起发布。我无法想象苹果对SQLite所做的更改会被视为专有，了解在苹果硬件上SQLite源代码与发布版本之间的差异对于理解SQLite在苹果操作系统上提供的保证是很重要的。
