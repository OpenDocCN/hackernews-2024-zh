<!--yml

类别：未分类

日期：2024-05-29 13:29:40

-->

# 调度内部

> 来源：[https://tontinton.com/posts/scheduling-internals/](https://tontinton.com/posts/scheduling-internals/)
> 
> 先睹为快！

我记得当我第一次得知你可以在仅一个线程上运行处理数百万客户端的服务器时，我的脑袋简直是被震惊到了 🤯

我使用 Node.js，虽然知道它是单线程的，我在 Python 中使用了`async` / `await`，也使用了线程，但从未问过自己“*这一切怎么可能呢？*”。

这篇文章的目的是传播并发的天才，并希望你也对此感到兴奋。

我的目标是让你想把这篇文章的链接发送给团队中的工程师，并大声问道“*等等，异步工作是怎么回事？*”。

我要回答的问题：

+   为什么不为每个客户端创建一个线程？

+   在等待 I/O 时如何进行休眠？

+   Node.js 如何实现并发？

+   什么是并发？什么是并行？

+   什么是协程？

    +   通过一个实现，我们将逐步构建起来。

+   什么是抢占式和非抢占式调度器？

+   Go 和 Rust 如何在语言中实现并发（有栈 vs 无栈）？

+   Linux、Go 和 Rust 的 Tokio 使用什么调度算法？

我假设您在中级水平上具备阅读代码和操作系统内部的熟练能力，但不要过于强调您不理解的细节，试着把握整体图景！

一切搞定后，让我们开始吧。

# 只需创建一个线程，伙计

让我们尝试在 C 代码中编写一个简单的回显服务器（无论收到什么，我们都返回），我们称之为`echod`：

```
// Assume server_fd is already initialized to start accepting clients. void serve(int server_fd) {
  while (true) {
  int client_fd = accept(server_fd, NULL, NULL);    char buffer[2048];    while (true) {
  ssize_t len = read(client_fd, buffer, sizeof(buffer));
  if (len == -1)
  break;    write(client_fd, buffer, len);
  }    close(client_fd);
  } } 
```

很酷，现在如果我们想要同时处理多个客户端应该怎么办？当一个客户端正在被处理时，另一个试图连接到我们的服务器，但永远不成功，因为我们的服务器只有在处理当前客户端后才会调用`accept`。

我们如何修复这个问题？

大多数人会首先想到“*难道你不能为每个客户端创建一个线程吗？*”，看起来像这样：

```
void serve_client(int client_fd) {
  char buffer[2048];    while (true) {
  ssize_t len = read(client_fd, buffer, sizeof(buffer));
  if (len == -1)
  break;    write(client_fd, buffer, len);
  }    close(client_fd); }   void serve(int server_fd) {
  while (true) {
  int client_fd = accept(server_fd, NULL, NULL);
  run_thread(serve_client, client_fd);
  } } 
```

使用线程的第一个问题是操作系统为新线程分配堆栈的方式。堆栈分配了虚拟内存（Linux 默认为 10MB），物理页面仅在实际写入页面时才会分配。这非常好，因为这意味着你不会在一开始就为每个线程预留10MB的RAM，**但**它确实意味着分配的粒度至少是页面大小（运行`getconf PAGESIZE`，我的机器是4KB）。使用`pthread_attr_setstacksize`不会解决问题，你仍然必须提供一个页面大小的倍数。页面可能比你实际使用的要大得多，这取决于应用程序。

> 我也认为依赖内存超额分配非常恼人。我们被内存耗尽杀手（OOM killer）杀死，而不是在分配失败时有机会清理资源。

创建一堆操作系统线程时需要解决的第二个问题是更改所有相关的限制：

```
$ cat /proc/sys/kernel/threads-max 63704   # Threads in linux are light weight processes (LWP). $ cat /proc/sys/kernel/pid_max 131072   # Maximum number of VMAs a process can own. $ cat /proc/sys/vm/max_map_count 65530   # Each of these can be set by simply writing to the file: $ echo 2000000 > /proc/sys/kernel/threads-max $ echo 2000000 > /proc/sys/kernel/pid_max $ echo 2000000 > /proc/sys/vm/max_map_count 
```

只有我向您展示的文件可能对您的系统不够，例如`systemd`也设置了最大限制。

第三个问题是性能。内核和用户模式之间的上下文切换在 CPU 循环方面是昂贵的。单个上下文切换本身并不昂贵，但如果频繁进行大量切换，成本就会累积起来。

第四个问题是堆栈分配是静态的，我们无法修改堆栈大小（增加）或在未使用时释放已分配的物理页（缩减）。

因为所有这些问题，线程不应该成为同时运行大量任务的首选解决方案（特别是在像`echod`中的 I/O 绑定任务中）。

我们还能以何种方式使`echod`同时为数百万个客户端提供服务？

# 异步 I/O

当调用`read` / `write` / `accept`时，为什么要阻塞整个线程？如果仔细考虑，我们会浪费一种宝贵的资源（CPU），让应用程序等待 I/O。

例如，在调用`read`时，内核会等待从网络接口卡或`NIC`接收到网络数据包。CPU 可以在此期间运行其他任务。

在 Linux 中，您可以通过使用`ioctl(fd, FIONBIO)`或`fcntl & O_NONBLOCK`（POSIX）将套接字标记为非阻塞。对同一套接字的`read`调用将立即返回。如果`NIC`写入了我们尚未读取的数据包，`read`将像往常一样复制缓冲区，否则将返回错误，`errno`等于`EWOULDBLOCK`。

让我们再次将`echod`修补为单线程，但这次使用非阻塞套接字支持多个并发客户端：

```
struct client {
  int fd;  char buffer[2048]; // Not allocated dynamically for brevity.
  ssize_t offset; // -1 when in reading state, otherwise the offset in buffer to write.
  ssize_t length; // The length left to write (only useful when in writing state). };   void set_nonblock(int fd) {
  // Ignore errors for brevity.
  fcntl(fd, F_SETFL, fcntl(fd, F_GETFL) | O_NONBLOCK); }   void serve(int server_fd) {
  set_nonblock(server_fd);    struct client clients[64]; // Maximum 64 concurrent clients.
  int num_clients = 0;    while (true) {
  if (num_clients < ARRAY_SIZE(clients)) {
  int client_fd = accept(server_fd, NULL, NULL);
  if (client_fd != -1) {
  set_nonblock(client_fd);
 clients[num_clients].fd = client_fd; clients[num_clients].offset = -1;
 num_clients++;
  }
  }    for (int i = 0; i < num_clients; i++) {
  struct client* client = &clients[i];
  bool is_reading = client->offset == -1;    ssize_t result;  if (is_reading) {
 result = read(client->fd, client->buffer, sizeof(client->buffer));
  } else {
 result = write(client->fd, client->buffer + client->offset, client->length);
  }    if (result != -1) {
  if (is_reading) {
 client->offset = 0; // Move to writing state.
 client->length = result;  } else {
 client->length -= result;    if (client->length == 0) {
 client->offset = -1; // Move to reading state.
  } else {
 client->offset += result;  }
  }
  } else if (errno != EWOULDBLOCK) {
  close(client->fd);
 num_clients--;
  memcpy(client, &clients[num_clients], sizeof(*client));
 i--; // Don't skip the moved client.
  }
  }
  } } 
```

> 有点冗长，不要试图理解一切，只需知道我们同时处理许多不同的任务。要获取可编译版本，请点击[这里](https://github.com/tontinton/echod-hog/blob/master/main.c)。

这种解决方案的主要问题是我们现在总是在忙于做某事，CPU 的利用率为 100%，即使大多数循环迭代将导致`EWOULDBLOCK`。

> 另一个问题是代码复杂性，现在我们被禁止运行可能会阻塞整个服务器应用程序的代码。

我们真正想要的是在没有有用事情可做时休眠。即当没有客户端等待连接，没有客户端发送任何数据包，并且由于某种原因（也许客户端正在忙于自己的事情），我们尚未能够向客户端发送数据包时。

我们希望这样做的原因有：

+   为了成为在同一台机器上运行的其他应用程序的良好邻居，并且不占用它们可能想要利用的 CPU 循环。

+   CPU 运行得越多，消耗的能量就越多：

    +   电池寿命更短。

    +   更昂贵。

    +   更少环保 🌲🌳🌿

不过好消息是，大多数操作系统提供了适合此类操作的 API。甚至可能提供了太多的 API：

+   **select(2)** - 一个 POSIX API。手册非常棒，让我们复制重要部分：

```
$ man 2 select   ...   select() allows a program to monitor multiple file descriptors, waiting until one or more of the file descriptors become "ready" for some class of I/O operation (e.g., input possible). A file descriptor is considered ready if it is possible to perform a corresponding I/O operation (e.g., read(2), or a sufficiently small write(2)) without blocking.   select() can monitor only file descriptors numbers that are less than FD_SETSIZE; poll(2) and epoll(7) do not have this limitation. See BUGS.   ... 
```

> 主要的要点是select只能监视`FD_SETSIZE`数量的FDs，通常为1024（glibc）。另一个需要注意的是，当FD准备就绪时，它会扫描所有注册的FDs（`O(n)`），因此当你有大量的FDs时，可能会花费大量时间在这上面。

+   **poll(2)** - 一个posix API。不限于`FD_SETSIZE`，但仍然像`select`一样是`O(n)`。

+   **epoll(7)** - 一个Linux API。非可移植的`poll`，但至少它的扩展性更好，因为它是`O(1)`的。

+   **aio(7)** - 一个Linux API。与之前的API不同，它支持套接字和文件，但有一些主要的缺点：

    +   仅支持使用`O_DIRECT`打开的文件，这些文件的处理非常复杂。

    +   如果文件的元数据不可用，会阻塞直到可用。

    +   当存储设备的请求槽位用完时会阻塞（每个存储设备都有固定数量的槽位）。

    +   每次IO提交都会复制72字节，每次完成都会复制32字节。每次IO操作使用2个系统调用复制104字节。

+   **io_uring(7)** - 自Linux 5.1起的一个Linux API。像`aio`一样，它将磁盘和网络操作统一到一个API下，但没有`aio`的缺点。它被设计为快速的，创建了两个队列，这两个队列存在于共享内存中（用户空间和内核空间之间），一个用于提交I/O操作，另一个用于存放I/O操作的结果一旦准备好。更多信息请访问["什么是io_uring？"](https://unixism.net/loti/what_is_io_uring.html)。

> `ScyllaDB`成功地在[seastar](https://seastar.io/)中使用`aio`实现了他们的数据库，您可以在[他们的博客](https://scylladb.com/2017/10/05/io-access-methods-scylla/)上了解更多关于异步磁盘I/O的信息。
> 
> 如果您对除了Linux以外的平台感兴趣，Windows有[I/O Completion Ports](https://learn.microsoft.com/en-us/windows/win32/fileio/i-o-completion-ports)，FreeBSD和MacOS都使用[kqueue](https://man.freebsd.org/cgi/man.cgi?kqueue)。

在`io_uring`之前，异步I/O抽象库使用线程池以非阻塞方式运行磁盘I/O。

类似于[libuv](https://libuv.org/)（Node.js的动力来源）的库可以在仅使用一个线程的情况下运行高并发服务器（最终改为[使用 io_uring](https://github.com/libuv/libuv/pull/3952)）。这类库通常被称为`事件循环`，让我们稍微谈谈它们。

## 事件循环

本质上，事件循环（有时也称为[反应器模式](https://en.wikipedia.org/wiki/Reactor_pattern)）基本上是这样的：

```
while should_continue_running():
 ready_fds = poll_fds() # Implemented using any of the async APIs we talked about.
 events = fds_to_events(ready_fds)
  while event := events.pop():
 event.callback() 
```

是的，就是这样，看看`libuv`的`uv_run`：

```
int uv_run(uv_loop_t* loop, uv_run_mode mode) {
  int r;   // ...    while (r != 0 && loop->stop_flag == 0) {   // ...    uv__run_pending(loop);
  uv__run_idle(loop);
  uv__run_prepare(loop);   // ...    uv__io_poll(loop, timeout);   // ...   r = uv__loop_alive(loop);
  }   // ...    return r; } 
```

这里是`uv__run_pending`，不遗漏任何细节：

```
static void uv__run_pending(uv_loop_t* loop) {
  struct uv__queue* q;  struct uv__queue pq; uv__io_t* w;    uv__queue_move(&loop->pending_queue, &pq);    while (!uv__queue_empty(&pq)) {
 q = uv__queue_head(&pq);
  uv__queue_remove(q);
  uv__queue_init(q);
 w = uv__queue_data(q, uv__io_t, pending_queue);
 w->cb(loop, w, POLLOUT);
  } } 
```

相当令人印象深刻，对吧？这就是所有Node.js应用程序正在运行的内容。

如果需要调用阻塞的第三方库函数怎么办？为此，大多数事件循环库都有一个线程池，您可以在其中运行任意代码，例如在`libuv`中，您可以使用[uv_queue_work()](https://docs.libuv.org/en/v1.x/threadpool.html)。

> 考验时间：像`setTimeout`这样的东西在事件循环中是如何实现的？如果脑海中什么也没有，请尝试克隆`libuv`并阅读`src/timer.c`中`uv__run_timers`的实现。

### 事件驱动开发

使用事件循环时的编程模型本质上是事件驱动的，每个注册事件都有一个准备好时执行的回调函数。

让我们看看使用虚构的事件循环库时`echod`会是什么样子（从底部的`serve`开始）：

```
struct context {
  char buffer[2048];
  size_t offset;  size_t length; };   void on_write(loop_t* loop, struct context* ctx, int client_fd, ssize_t written) {
  if (written == -1) {
  // Error, stop.
  free(ctx);
  return;
  }   ctx->offset += written;    if (ctx->offset < ctx->length) {
  char* buf = ctx->buffer + ctx->offset;
  size_t len = ctx->length - ctx->offset;
  register_write(loop, ctx, client_fd, buf, len, on_write);
  } else {
  register_read(loop, ctx, client_fd, on_read);
  } }   void on_read(loop_t* loop, struct context* ctx, int client_fd, ssize_t read) {
  if (read == -1) {
  // Error, stop.
  free(ctx);
  return;
  }   ctx->offset = 0;
 ctx->length = read;    register_write(loop, ctx, client_fd, ctx->buffer, read, on_write); }   void on_accept(loop_t* loop, void* unused, int server_fd, int client_fd) {
  struct context* ctx = malloc(sizeof(*ctx));
  register_read(loop, ctx, client_fd, ctx->buffer, sizeof(ctx->buffer), on_read);
  register_accept(loop, NULL, server_fd, on_accept); }   void serve(loop_t* loop, int server_fd) {
  register_accept(loop, NULL, server_fd, on_accept);
  run_event_loop(loop); } 
```

如果你之前使用过 JavaScript，这看起来应该很熟悉。

从性能上来说，代码将非常轻量化，并且高度并发，但通常因以下原因而受到批评：

+   **不直观** - 对于大多数习惯于阅读和编写同步代码的程序员而言，看起来很复杂，参见["回调地狱"](http://callbackhell.com)。

+   **难以调试** - 调用堆栈非常短，不会显示你如何到达特定断点的流程。每个回调的调用者始终是`libuv`中的`uv_run`，或者我们示例中的`run_event_loop`。

许多现代编程语言和运行时试图通过让你编写看起来是同步的代码，同时实际上是完全异步的来解决这些问题。在下一章中，我们将学习如何实现。

# 抢占

你有没有想过，只有一个CPU核心的计算机上如何运行多个线程？

在本节中，我们将介绍一种称为**抢占**的秘密技术，它使这种魔术成为可能。

假设我们要同时执行以下两个任务：

```
def task_0():
  while True:
  print('0')   def task_1():
  while True:
  print('1')   run_all([task_0, task_1]) 
```

`run_all`应该做什么以确保这两个任务**并发**运行？

如果我们有两个CPU核心，我们可以简单地在一个核心上运行一个任务，这意味着我们可以**并行**运行两个任务，或者换句话说，在*真实*世界中（至少是我们基于物理学的世界）这两个任务同时运行。

**并发**程序处理同时运行多个事物，但并非同时进行，因此它们可能看起来是**并行**的，即使实际上并非如此。

`run_all`实现**并发**的一种方式是在某段时间内运行一个任务，暂停，然后继续运行下一个任务，并且永远循环，直到所有任务退出为止。为了像**并行**一样神奇地展现出来，你只需配置暂停之前的时间相对于人类经验来说非常短（例如100μs？）即可。

```
from itertools import cycle # cycle([1, 2, 3]) -> (1, 2, 3, 1, 2, 3, ...)   def run_all(tasks):
  for task in cycle(tasks):
  run_task_for_a_little(task) 
```

那么，如何暂停和恢复代码的执行呢？

答案在于程序是确定性状态机，只要向程序的执行者（例如本机代码的CPU）提供相同的输入（例如寄存器、内存等），它无论是今天执行还是几年后执行，输出都将相同。

基本上，暂停任务可以通过复制程序的当前状态来实现，恢复任务可以通过加载保存的状态来实现。

> 程序运行在真实 CPU 上或像 `python` 的字节码或 `JVM` 这样的虚拟 CPU 上都没有关系，它们都是确定性状态机。只要复制所有必要的状态，任务将会继续进行，就像从未暂停过一样。

为了便于理解如何实现抢占（保存和加载程序状态），让我们看看 `setjmp.h`，它在许多不同的 CPU 架构中实现了保存和加载程序状态，并且是 `libc` 的一部分。

参见以下示例（直接从 [wikipedia](https://en.wikipedia.org/wiki/Setjmp.h) 复制）：

```
#include &LTstdio.h> #include &LTsetjmp.h>   static jmp_buf buf;   void second() {
  printf("second\n"); // prints
  longjmp(buf,1); // jumps back to where setjmp was called - making setjmp now return 1 }   void first() {
  second();
  printf("first\n"); // does not print }   int main() {
  if (!setjmp(buf))
  first(); // when executed, setjmp returned 0
  else // when longjmp jumps back, setjmp returns 1
  printf("main\n"); // prints    return 0; } 
```

运行它将输出：

```
$ gcc example.c -o example && ./example second main 
```

`setjmp` 保存程序状态（在 `buf` 中），而 `longjmp` 将 `buf` 中的任何内容加载到 CPU 中。

让我们看看幕后的情况，以下是 `musl`（一个流行的 `libc` 实现）中 `x86_64` 汇编代码的 `setjmp` 和 `longjmp`：

```
setjmp:
  mov %rbx,(%rdi) ; rdi is jmp_buf, move registers onto it  mov %rbp,8(%rdi)
  mov %r12,16(%rdi)
  mov %r13,24(%rdi)
  mov %r14,32(%rdi)
  mov %r15,40(%rdi)
  lea 8(%rsp),%rdx ; this is our rsp WITHOUT current ret addr  mov %rdx,48(%rdi)
  mov (%rsp),%rdx ; save return addr ptr for new rip  mov %rdx,56(%rdi)
  xor %eax,%eax ; always return 0  ret   longjmp:
  xor %eax,%eax
  cmp $1,%esi ; CF = val ? 0 : 1  adc %esi,%eax ; eax = val + !val  mov (%rdi),%rbx ; rdi is the jmp_buf, restore regs from it  mov 8(%rdi),%rbp
  mov 16(%rdi),%r12
  mov 24(%rdi),%r13
  mov 32(%rdi),%r14
  mov 40(%rdi),%r15
  mov 48(%rdi),%rsp
  jmp *56(%rdi) ; goto saved address without altering rsp 
```

> 如果你不理解汇编语言也不要紧。关键是保存和加载程序状态非常简短和简单。

`setjmp` 将所有被调用保存的寄存器保存到 `jmp_buf` 中。被调用保存的寄存器用于保存长期存在的值，在函数调用之间应该保留。`longjmp` 直接将保存在 `jmp_buf` 中的被调用保存的寄存器恢复到 CPU 寄存器中。

> 对于好奇的人，不保存调用者保存的寄存器（例如 `rcx`）的原因是因为对编译器而言 `setjmp` 只是另一个函数调用，这意味着它不会使用调用者保存的寄存器来保存状态。它假定像任何函数调用一样，这些寄存器可能会被修改。

## 非抢占调度程序

现在，我们已经有了一个坚实的基础，可以开始同时运行多个任务了。

不依赖时间来暂停正在运行的任务，我们可以假设程序员手动插入调用 `longjmp`，参见示例（这次是 C 语言的 `setjmp.h`）：

```
#include &LTstdbool.h> #include &LTstdio.h> #include &LTsetjmp.h>   jmp_buf* current_buffer; jmp_buf main_buffer;   #define ARRAY_SIZE(arr) (sizeof(arr) / sizeof(arr[0])) #define YIELD() { if (!setjmp(*current_buffer)) longjmp(main_buffer, 1); }   void task_0() {
  while (true) {
  printf("0\n");
  YIELD();
  } }   void task_1() {
  while (true) {
  printf("1\n");
  YIELD();
  } }   int main() {
  void(*tasks[])(void) = {task_0, task_1};
 jmp_buf buffers[ARRAY_SIZE(tasks)];
  bool started = false;    while (true) {
  for (int i = 0; i < ARRAY_SIZE(tasks); i++) {
  if (setjmp(main_buffer)) {
  continue;
  }   current_buffer = &buffers[i];
  if (!started) {
 tasks[i]();  } else {
  longjmp(buffers[i], 1);
  }
  }   started = true;
  }    return 0; } 
```

让我们运行它：

```
$ gcc example.c -o example && ./example 0 1 0 1 ... ^C 
```

很酷，但是... 实际上有一个隐藏的 bug（你能找到吗？）。

让我们修改 `task_0`，在堆栈上保存一些状态：

```
void task_0() {
  int i = 0;
  while (true) {
  printf("0: %d\n", i++);
  YIELD();
  } } 
```

再次运行它：

```
$ gcc example.c -o example && ./example 0: 0 1 0: 32765 1 0: 32765 1 ... ^C 
```

糟糕... 因为所有任务共享同一个堆栈，每个任务（包括我们的 `main` 函数）可能会覆盖堆栈中的任何内容。请参见下面的示例：

```
---------------- | main's stack | ----------------
 ^ First, a call to setjmp in main saves the stack address here. 
```

```
--------------------------------- | main's stack | task_0's stack | ---------------------------------
 ^ Then, a call to setjmp in task_0 saves the stack address here. 
```

```
--------------------------------- | main's stack | task_0's stack | ---------------------------------
 ^ Once we longjmp back to main, we reset the stack register (rsp in x86_64) back here. 
```

```
Then any pushing to the stack, will overwrite any data saved in task_0's stack. So once we longjmp back to task_0, this is how our stack will look like:   --------------------------------- | main's stack | "random" stuff | ---------------------------------
 ^ No wonder we get a random looking value when we print something inside the stack. 
```

解决方法是为每个任务创建一个堆栈，并在调用任务函数之前切换到该堆栈：

```
@@ -25,6 +26,7 @@ void task_1() {
 int main() { void(*tasks[])(void) = {task_0, task_1}; jmp_buf buffers[ARRAY_SIZE(tasks)]; +    char stacks[ARRAY_SIZE(tasks)][1024];  // Stack size of 1kb.
 bool started = false;   while (true) { @@ -35,7 +37,12 @@ int main() {   current_buffer = &buffers[i]; if (!started) { -                tasks[i](); +                // Stack goes down on push, up on pop. +                char* stack = stacks[i] + sizeof(stacks[i]); +                asm("movq %0, %%rax;" +                    "movq %1, %%rsp;" +                    "call *%%rax" +                    :: "rm" (tasks[i]), "rm" (stack) : "rax");
 } else { longjmp(buffers[i], 1); } 
```

> 保存任务函数在寄存器 `rax` 中的原因是为了避免在堆栈内查找 `tasks[i]`，因为我们刚刚将堆栈更改为其他内存位置。`asm` 语法在 [这里](https://gcc.gnu.org/onlinedocs/gcc/extensions-to-the-c-language-family/how-to-use-inline-assembly-language-in-c-code.html) 全面记录。

最后再运行一次：

```
$ gcc example.c -o example && ./example 0: 0 1 0: 1 1 0: 2 1 ... ^C 
```

我们成功实现了用户模式非抢占调度程序！

在真正的非抢占式（也称为协作式）系统中，运行时应在知道 CPU 在当前任务中没有更多有用工作时让出，例如等待 I/O。他们通过注册 I/O 并将任务移动到不同的队列来做到这一点，该队列保存阻塞任务（调度器跳过运行）。一旦有了 I/O，他们将任务从阻塞队列移回常规队列以执行。例如，可以通过与事件循环集成来完成此操作。

这里有一些流行主流运行时中的非抢占式调度器示例：

+   **Rust 的 [tokio](https://tokio.rs/)** - 要进行让步，你可以调用`tokio::task::yield_now()`，或者运行直到阻塞（例如等待 I/O 或 `tokio::time::sleep()`）。在版本 0.3.1 中，他们引入了[自动让步](https://tokio.rs/blog/2020-04-preemption)。

+   **Go（1.14 版本之前）** - 在发布（版本 1.0）时，要进行让步，你可以调用`runtime.Gosched()`，或者运行直到阻塞。在版本 1.2 中，调度器也偶尔会在函数入口处被调用。

+   **Erlang** - 在[BEAM](https://blog.stenmans.org/theBeamBook/)（erlang 的强大运行时）中，调度器在函数调用时被调用。由于除了递归和列表推导没有其他循环结构，没有办法无限循环而不进行函数调用。不过，你可以通过运行本地的 C 代码来作弊，使用`NIF`（本地实现函数）。

非抢占式调度器是有风险的，因为我们假设开发人员在进行长时间计算时记得放置`yield`调用：

```
// In Go (prior to 1.14), this code would not yield until done. sum := 0 for i := 0; i < 1e8; i++ {
  sum++ } 
```

## 抢占式调度器

抢占式调度器偶尔进行上下文切换（让出），即使没有开发人员插入让出调用。

大多数现代操作系统使用定时器中断。CPU 每隔一段时间收到一个中断。中断停止当前正在运行的任何内容，并且中断处理程序调用调度器，决定是否进行上下文切换。

这很酷，但用户模式应用程序不能注册中断，那么如果我们想在用户模式中实现抢占式调度器，我们该怎么做呢？

一个简单的解决方案是利用内核的抢占式调度器。创建一个线程，定期向运行我们的调度器的线程发送信号。

这正是 Go 在版本 1.14 中使他们的调度器变为抢占式的方式。通过周期性地从他们的监控线程（[runtime.sysmon](https://sobyte.net/post/2021-12/golang-sysmon/)）向运行 goroutines 的调度器线程发送信号。

> 想了解更多关于他们解决方案的信息，我建议你观看["Pardon the Interruption: Loop Preemption in Go 1.14"](https://youtube.com/watch?v=1I1WmeSjRSw)。

## 堆栈式 vs 无堆栈式

直到现在，我一直称它们为任务，以免让你困惑，但它们有许多不同的名称，如纤程（fibers）、greenlets、用户模式线程、绿色线程、虚拟线程、协程和 goroutines。

> 当人们说线程时，通常指的是操作系统线程（由内核调度程序管理）。

协程简单来说是可以暂停和恢复的程序。主要有两种实现方式：为每个协程分配一个堆栈（基于堆栈），或者使每个标记为`async`的函数返回一个可以暂停和恢复该函数所需状态的对象（无堆栈）。

基于堆栈和无堆栈极大地影响API，各有其优缺点。以下是概述：

+   **基于堆栈** - 协程与操作系统线程具有完全相同的API和语义，这是有道理的，因为它们都在运行时分配堆栈。我们使用`setjmp`的示例调度程序是基于堆栈的。Go语言是另一个基于堆栈的实现的示例。就像Go需要定期上下文切换一样，它还需要定期检查是否有足够的空闲堆栈空间继续运行，如果没有，它重新分配堆栈以获得更多内存，复制之前的内容并修复所有指向旧堆栈的指针，使其指向新堆栈。堆栈不仅可以动态增长，还可以在需要时缩小。真正的美妙之处在于，您可以选择在后台同步或异步运行任何函数，而不影响其周围的代码：

```
go fmt.Println("world") fmt.Println("hello") 
```

+   **无堆栈** - 如果您曾经使用过具有`async`和`await`的语言，您就使用过无堆栈的实现。例如，Rust和Python的`asyncio`。Rust的`async`将一段代码块转换为一个状态机，在您`await`它之前不会运行。这种方法的最大优势在于运行时的轻量级，内存分配恰好在需要时进行，这也非常适合Rust的嵌入式用例。这种方法的主要问题是“函数着色”。`async`函数只能在另一个`async`函数内部调用：

```
async fn say() {
 println!("hello world"); }   fn main() {
  // Can't call say(), as main() is not async.    let mut rt = tokio::runtime::Runtime::new().unwrap();
 rt.block_on(async {
  let future = say(); // Calling does not execute.
 future.await; // Starts executing.
  }) } 
```

> Rust在发布之前采用了基于堆栈的方法，但最终切换到了无栈：["为什么选择异步 Rust？"](https://without.boats/blog/why-async-rust/)。

# 调度算法

调度程序还负责决定一旦一个任务完成后应该运行哪个任务。

最简单的方法之一是我们已经在事件循环部分看到的，在任务队列中按照它们被添加的顺序运行任务。

Linux的`SCHED_FIFO`调度程序正是这样做的：

> 每个圆圈都是一个任务。围绕任务的白色进度圆圈显示任务被阻塞的剩余运行时间。
> 
> **紫色框** - 包含准备运行任务的队列。
> 
> **绿色框** - CPU。
> 
> **灰色框** - 被某些东西阻塞的任务（例如I/O）。

使用`SCHED_FIFO`并添加任务运行时限是`SCHED_RR`所做的，允许CPU以更均匀的方式共享：

如果你有一个必须每5毫秒运行一次的任务，即使时间很短也必须运行怎么办？例如，在音频编程中，你需要填充一个信号（如`sin(x)`）到缓冲区中，音频设备以某个间隔从中读取。如果错过了填充此缓冲区，将导致听起来像爆裂噪音的随机信号，可能会破坏整个乐团的录音。

这种类型的程序通常称为软实时程序。硬实时意味着错过截止日期将导致整个系统失败，例如自动驾驶和航天器。

Linux 对软实时系统有一个很好的解决方案，称为`SCHED_DEADLINE`，其中每个线程设置到其截止时间的时间量，调度器始终运行接近达到截止时间的任务：

> **绿色**进度圆圈显示距离截止时间还剩多少时间。
> 
> 关注**粉色**圆圈，它有一个短期限，使其比其他圆圈运行更频繁。
> 
> `SCHED_FIFO`和`SCHED_RR`也可用于软实时系统，因为它们具有确定性质，取决于您需要解决的问题。

为了保证所有任务能够按其配置的截止时间运行，`SCHED_DEADLINE`计算并拒绝具有会夺取过多运行时间的配置的线程。您可以在lwn的["截止日期调度"](https://lwn.net/Articles/743740/)上了解更多信息。

对于像笔记本电脑运行任意进程这样的通用工作负载，通常希望公平性。公平性可以通过持续跟踪哪些进程的 CPU 时间比其他进程少，并始终运行具有最低已跟踪运行时间的任务来实现。Linux 的默认调度器`SCHED_OTHER`，也称为`CFS`（完全公平调度器），正是这样做的。您还可以通过设置`nice`值为进程配置优先级，其中`nice`值较低的进程将被更频繁地调度。

> `CFS`已经服务了26年，但在v6.6中，新的默认调度算法是[EEVDF](https://lwn.net/Articles/925371/)。

## 多核

到目前为止，我基本上忽略了现代计算机拥有多个 CPU 核心的事实。

实现多核调度的最简单方法是继续像以前一样。拥有一个全局任务队列，其中包含准备运行的任务，并在核心准备就绪后运行它们：

您只需确保任务队列在`MPMC`操作（多生产者多消费者）时是线程安全的，可以通过原子操作或锁来实现。

`MPMC`队列比更严格的`SPMC`（单生产者多消费者）队列慢得多，这就是为什么 Go 选择为每个调度器配置一个固定大小的`SPMC`队列（Go 使用`GOMAXPROCS`配置每个核心运行一个调度器），并在`SPMC`队列满时推送到全局`MPSC`队列。

要确保所有核心充分利用，当一个核心处于空闲状态但本地队列中没有任务并且全局队列中也没有任务时，它会**窃取**其他本地队列的任务（这就是它们为什么是多消费者的原因）。

> Go的解决方案非常出色，tokio从中借鉴了很多。我强烈推荐在他们的博客上阅读相关内容：["使Tokio调度器快10倍"](https://tokio.rs/blog/2019-10-scheduler)。

# 结论

祝贺 🥳！你是一个真正的英雄，到达了终点，希望你有所收获。

这个主题还有很多内容需要涵盖，本文中留下的链接是探索并发和并行性无尽兔子洞的绝佳起点。

如果你想自己玩弄这些动画，请点击这里查看[代码](https://github.com/tontinton/sched_animation)。

点击这里返回顶部的动画。
