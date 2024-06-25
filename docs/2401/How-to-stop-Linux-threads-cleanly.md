<!--yml

category: 未分类

日期：2024-05-27 14:36:38

-->

# 如何干净地停止 Linux 线程

> 来源：[`mazzo.li/posts/stopping-linux-threads.html`](https://mazzo.li/posts/stopping-linux-threads.html)

# *2024-01-07* 如何干净地停止 Linux 线程

假设你正在编写一个长时间运行的多线程应用程序，在 Linux 上。也许它是一个数据库或某种服务器。让我们想象一下，你不是在一些受控运行时（也许是 JVM、Go 或 BEAM）上运行，而是管理使用 [`clone`](https://man7.org/linux/man-pages/man2/clone.2.html) 系统调用产生的线程。想象一下用 C 创建的线程，使用 [`pthread_create`](https://man7.org/linux/man-pages/man3/pthread_create.3.html)，或者使用 C++ 的 [`std::thread`](https://en.cppreference.com/w/cpp/thread/thread/thread)。

一旦你开始创建线程，你可能也要停止它们。但前者比后者要容易得多。所谓的“停止”，是指在完全终止之前给线程一个运行一些清理操作的机会。换句话说，我们希望在确保释放内存、释放锁、刷新日志等的情况下终止线程。

这个任务遗憾地并不像应该的那样简单，也绝对没有一种大小适合所有的解决方案。本文旨在概述问题空间，并强调一些没有短缺的陷阱，并在最后展示一点小魔术技巧。

## (准)忙等待 #

如果你负担得起的话，可以将每个线程结构化如下：

```
while (true) {
 if (stop) { break; }
 // Perform some work completing in a reasonable time
}
```

`stop` 在这里是一个每个线程的布尔值。当我们想要停止一个线程时，我们将 `stop` 设置为 `true`，然后调用 [`pthread_join`](https://man7.org/linux/man-pages/man3/pthread_join.3.html) 或其等价物以确保线程实际上已经终止。

这里是一个虚构但可行的 C++ 示例：

```
#include <thread>
#include <atomic>
#include <stdio.h>
#include <unistd.h>

static std::atomic<bool> stop = false;

int main() {
 std::thread thr([] {
 // prints every second until stopped
 for (int i = 0; !stop.load(); i++) {
 printf("iterated %d times\n", i);
 sleep(1);
 }
 printf("thread terminating\n");
 });
 // waits 5 seconds, then stops thread
 sleep(5);
 stop.store(true);
 thr.join();
 printf("thread terminated\n");
 return 0;
}
```

输出：

```
iterated 0 times
iterated 1 times
iterated 2 times
iterated 3 times
iterated 4 times
thread terminating
thread terminated
```

如果你可以编写或重构你的代码以在这样的时间片段内工作，那么终止线程就非常容易。

请注意，循环体不需要完全非阻塞——它只需要在我们希望终止迅速的时间内被终止。例如，如果我们的线程正在从套接字读取数据，我们可以将 `SO_TIMEOUT` 设置为 100 毫秒，这样我们就知道循环的每次迭代都会迅速终止。

## 如果我想永久阻塞怎么办？#

准忙等待很好，但有时并不理想。最常见的障碍是我们无法控制的外部代码，它不符合这种模式——比如一个进行阻塞网络调用的第三方库。

正如我们将在后面看到的，没有干净的方式来停止一个运行我们无法控制代码的线程，但还有其他原因不想要以准忙等待模式编写所有代码。

如果我们有很多线程，即使相对较慢的超时也可能因为虚假唤醒而导致显著的调度开销，特别是在已经繁忙的系统上。超时还将使调试和检查系统变得相当麻烦（例如，想象一下`strace`的输出会是什么样子）。

因此，值得考虑如何在线程在系统调用时被阻塞时停止线程。最直接的方法是通过信号。

## 我们需要讨论信号 #

信号是中断线程执行而无需显式协调中断线程的主要方式，因此与本博客文章的主题非常相关。它们也有点混乱。这两个事实引发了不快。

关于信号的良好概述，我推荐出乎意料的信息丰富的[man 页面](https://man7.org/linux/man-pages/man7/signal.7.html)，但我会在这里给出足够的概述。如果您已经了解信号的工作原理，可以跳转到下一节。

信号可以由某些硬件异常引发，也可以由软件发起。最熟悉的软件发起信号的实例是当您按下`ctrl-c`时，您的 shell 向前台进程发送 SIGINT。所有由软件发起的信号都源自于少数几个系统调用 - 例如[`pthread_kill`](https://man7.org/linux/man-pages/man3/pthread_kill.3.html)会向线程发送一个信号。

硬件发起的信号通常会立即处理，而软件发起的信号会在 CPU 在内核完成一些工作后即将重新进入用户模式时处理。无论如何，当一个信号需要在给定的线程中处理时：

1.  如果信号被接收线程阻塞，它将等待处理直到被解除阻塞；

1.  如果信号没有被阻塞，它可能会：

    1.  忽略；

    1.  以“默认”方式处理；

    1.  使用一些自定义的信号处理器进行处理。

哪些信号被阻塞由修改*信号掩码*来控制，使用[`sigprocmask`](https://man7.org/linux/man-pages/man2/sigprocmask.2.html)/[`pthread_sigmask`](https://man7.org/linux/man-pages/man3/pthread_sigmask.3.html)，如果线程没有被阻塞，则采取的操作由[`sigaction`](https://man7.org/linux/man-pages/man2/sigaction.2.html)来控制。

假设信号*没有*被阻塞，路径 2.a 和 2.b 将完全由内核管理，而路径 2.c 将导致内核将控制权传递给用户空间信号处理程序，该处理程序将对信号执行某些操作。

重要的是，如果某个线程在系统调用中（例如在从套接字中读取时被阻塞），并且需要处理信号，则在信号处理程序运行后，系统调用将提前返回错误代码`EINTR`。

信号处理程序代码受[各种约束](https://man7.org/linux/man-pages/man7/signal-safety.7.html)约束，但除此之外，它可以随意操作，包括决定不将控制权返回给执行之前的代码。默认情况下，大多数信号只会导致程序突然停止，可能会出现核心转储。在接下来的几节中，我们将探讨使用信号停止线程的各种方法。

## 线程取消，一个虚假的希望 #

让我们首先检查一种通过信号实现的停止线程的方法，这似乎正是我们想要的：线程取消。

线程取消的 API 非常有前途。[`pthread_cancel(tid)`](https://man7.org/linux/man-pages/man3/pthread_cancel.3.html)会“取消”线程`tid`。`pthread_cancel`的工作方式归结为：

1.  一个特殊的信号被发送到线程`tid`；

1.  你正在使用的 libc（比如[glibc](https://en.wikipedia.org/wiki/Glibc)或[musl](https://en.wikipedia.org/wiki/Musl)）设置了一个处理程序，以便在接收到取消信号时线程会停止。

还有其他细节，但基本上就是这样了。然而，困难在前方。

### 资源管理 + 线程取消 = 😢 #

请记住，信号实质上可以在您的代码的任何地方出现。所以如果我们有这样的代码

```
lock();
// critical work here
unlock();
```

我们可能在关键部分收到一个信号。在线程取消的情况下，我们的线程可能在我们持有锁的情况下被取消，或者有一些要释放的内存，或者总之有一些未完成的资源，并且我们的清理代码永远不会运行。这不好。

虽然有一些缓解情况，但都不足够：

+   线程取消可以被暂时禁用。所以我们可以在任何时候禁用它当我们处于这样一个关键部分。

    然而，一些“关键部分”非常长（考虑一些分配内存的寿命），而且我们还必须确保在正确的时间启用/禁用取消来装饰所有相关代码。

+   Linux 线程包括使用[`pthread_cleanup_push`](https://man7.org/linux/man-pages/man3/pthread_cleanup_push.3.html)和`pthread_cleanup_pop`添加/删除全局清理处理程序的设施。当线程被取消时，这些清理处理程序*会*运行。

    然而，为了确保使用这些函数的安全性，一个人必须再次用不仅仅是一个推/弹出，而且还要暂时禁用取消来避免竞争，因为我们设置了清理。

    再次强调，这样做会非常容易出错，并且会显著减慢我们的代码。

+   默认情况下，线程取消发送的信号只在“取消点”接收到，大致上是可能阻塞的系统调用 - 请参阅[`pthreads(7)`](https://man7.org/linux/man-pages/man7/pthreads.7.html)。

    所以我们只有在关键部分有这样的系统调用时才会遇到麻烦。但是再次强调，我们必须手动确保要么关键部分没有取消点，要么通过其他方式（可能是上述两种措施之一）使其安全。

### 线程取消与现代 C++ 不兼容 #

如果你是 C++/Rust 程序员，你可能会对上面的显式锁定感到不屑一顾 - 你有 RAII 来处理这种情况：

```
{
 const std::lock_guard<std::mutex> lock(mutex);
 // critical work here
 // The destructor for `lock` will do the unlocking
}
```

你可能也在想，在这里由 RAII 管理的关键部分中如果线程取消到达会发生什么。

答案是线程取消将触发类似于抛出异常的堆栈解除的机制（实际上它是用一个特殊的异常实现的），这意味着析构函数 *将* 在取消时运行。这种机制称为 *强制解除*。很棒，对吧？

好吧，由于线程取消是使用异常实现的，并且线程取消可能发生在任意位置，[我们总是可能在 `noexcept` 块中发生取消](https://gcc.gnu.org/legacy-ml/gcc/2017-08/msg00121.html)，这将通过 [`std::terminate`](https://en.cppreference.com/w/cpp/error/terminate) 使你的程序崩溃。

所以自从 C++11，特别是自从 C++14 开始，在 C++ 中线程取消基本上是无用的，因为析构函数默认标记为 `noexcept`。

### 强制解除本身就是不安全的 #

但是请注意，即使在 C++ 中这种机制运行良好，它在许多情况下仍然不安全。考虑以下情况：

```
{
 const std::lock_guard<std::mutex> lock(mutex);
 balance_1 += x;
 balance_2 -= x;
}
```

如果在 `balance_1 += x` 之后强制解开我们的不变性就会失效。这就是为什么 Java 的强制解开形式 [`Thread.stop`](https://docs.oracle.com/javase/8/docs/technotes/guides/concurrency/threadPrimitiveDeprecation.html) 被废弃的原因。

## 你无法干净地停止运行你无法控制的代码的线程 #

顺便说一句，信号的性质（以及由此引申出的线程取消）意味着无法干净地停止你无法控制的代码。你无法保证内存不会泄漏，文件被关闭，全局锁被释放等等。

如果你需要可靠地中断外部代码，最好将其隔离在自己的进程中。它可能仍然会泄漏临时文件和其他持久性资源，但当进程终止时，操作系统将清理掉大部分相关状态。

## 受控线程取消 #

希望现在你已经确信在大多数情况下无限制的线程取消并不是一个好主意。然而，我们可以通过只在特定时间启用线程取消来明确选择情况。因此我们的事件循环变成了：

```
pthread_setcancelstate(PTHREAD_CANCEL_DISABLE);
while (true) {
 pthread_setcancelstate(PTHREAD_CANCEL_ENABLE);
 // syscall that might block indefinitely, e.g. reading
 // from a socket
 pthread_setcancelstate(PTHREAD_CANCEL_DISABLE);
 // Perform some work completing in a reasonable time
}
```

我们默认关闭线程取消，但在进行阻塞系统调用时重新打开它。

重构我们的代码以适应这种模式可能看起来很繁琐。但是，许多具有长时间运行线程的应用程序已经包含了以阻塞系统调用开始的循环（从套接字读取，定时器休眠等），然后是一些不会无限期阻塞的工作。

## 自制线程取消 #

但是一旦我们这样做了，或许值得完全摆脱线程取消。依赖堆栈展开来释放资源不会在可选的 libcs 上是可移植的，而且如果我们想在析构函数外执行一些显式清理操作，我们需要[相当小心](https://udrepper.livejournal.com/21541.html)。

因此，我们可以直接使用信号。我们可以选择 SIGUSR1 作为我们的“停止”信号，安装一个设置我们停止变量的处理程序，并在执行阻塞系统调用之前检查变量。

[这是一个在 C++ 中的示例。](https://gist.github.com/bitonic/d3281b2d0fd95b4fd788aa7e013d1fb9/7f5a705198f5f9c3c250d24ec085bb75796a4752) 代码的有趣部分是设置信号处理程序：

```
// thread_local isn't really necessary here with one thread,
// but it would be necessary if we had many threads we wanted
// to kill separately.
static thread_local std::atomic<bool> stop = false;

static void stop_thread_handler(int signum) {
 stop.store(true);
}

int main() {
 // install signal handler
 {
 struct sigaction act = {{ 0 }};
 act.sa_handler = &stop_thread_handler;
 if (sigaction(SIGUSR1, &act, nullptr) < 0) {
 die_syscall("sigaction");
 }
 }
 ...
```

和在运行系统调用之前检查标志的代码：

```
ssize_t recvlen;
if (stop.load()) {
 break;
} else {
 recvlen = recvfrom(sock, buffer.data(), buffer.size(), 0, nullptr, nullptr);
}
if (recvlen < 0 && errno == EINTR && stop.load()) {
 // we got the signal while running the syscall
 break;
}
```

但是，检查标志并启动系统调用的代码存在竞争条件：

```
if (stop.load()) {
 break;
} else {
 // signal handler runs here, syscall blocks until
 // packet arrives -- no prompt termination!
 recvlen = recvfrom(sock, buffer.data(), buffer.size(), 0, nullptr, nullptr);
}
```

没有简单的方法来检查标志并原子地运行系统调用。

[解决这个问题的另一种方法](https://gist.github.com/bitonic/d3281b2d0fd95b4fd788aa7e013d1fb9/139cc58cbdbba88ef311f7ea1b47a06050f03016)是正常阻止 USR1，并仅在系统调用运行时解除阻止，类似于我们在临时线程取消时所做的。如果系统调用以 `EINTR` 终止，我们知道我们应该退出。

不幸的是，竞争仍然存在，只是在解除阻塞和运行系统调用之间：

```
ptread_sigmask(SIG_SETMASK, &unblock_usr1); // unblock USR1
// signal handler runs here, syscall blocks until
// packet arrives -- no prompt termination!
ssize_t recvlen = recvfrom(sock, buffer.data(), buffer.size(), 0, nullptr, nullptr);
ptread_sigmask(SIG_SETMASK, &block_usr1); // block USR1 again
```

### 原子地更改 sigmask #

然而，通常有一种简单的方法来原子地更改 sigmask 并运行系统调用：

+   [`select`](https://man7.org/linux/man-pages/man2/select.2.html)/[`poll`](https://man7.org/linux/man-pages/man2/poll.2.html)/[`epoll_wait`](https://man7.org/linux/man-pages/man2/epoll_wait.2.html)有带有 `sigmask` 参数的 `pselect`/`ppoll`/`epoll_pwait` 变体；

+   `read`/`write` 和类似的系统调用可以被它们的非阻塞版本替代，并使用阻塞的 `ppoll`；

+   可以使用[`timerfd`](https://man7.org/linux/man-pages/man2/timerfd_create.2.html)或只是使用没有文件描述符但有超时的 `ppoll` 来睡眠；

+   新添加的[`io_uring_enter`](https://man7.org/linux/man-pages/man2/io_uring_enter.2.html)直接支持这种用例。

上述系统调用已经涵盖了非常广泛的范围。

这种风格中，程序的[接收循环变得](https://gist.github.com/bitonic/d3281b2d0fd95b4fd788aa7e013d1fb9/9bd6b18f6329a0b07cba5cb38d3c45bbf27ae968)：

```
struct pollfd pollsock = {
 .fd = sock,
 .events = POLLIN,
};
if (ppoll(&pollsock, 1, nullptr, &usr1_unmasked) < 0) {
 if (errno == EINTR) {
 break;
 }
 die_syscall("ppoll");
}
ssize_t recvlen = recvfrom(sock, buffer.data(), buffer.size(), 0, nullptr, nullptr);
```

### 使其与任何系统调用一起工作 #

遗憾的是，并非所有的系统调用都有变体能让我们在执行过程中原子性地改变 sigmask。[`futex`](https://man7.org/linux/man-pages/man2/futex.2.html)，用于实现用户空间并发原语的主要系统调用，是一个例子，该系统调用*没有*包括这样的设施。

对于`futex`，可以通过`FUTEX_WAKE`中断线程，但事实证明我们可以建立一个机制，可以安全地用*任何*系统调用原子性地检查布尔停止标志。

回顾一下，有问题的代码[看起来像这样](https://gist.github.com/bitonic/d3281b2d0fd95b4fd788aa7e013d1fb9/7f5a705198f5f9c3c250d24ec085bb75796a4752)：

```
if (stop.load()) {
 break;
} else {
 // signal handler runs here, syscall blocks until
 // packet arrives -- no prompt termination!
 recvlen = recvfrom(sock, buffer.data(), buffer.size(), 0, nullptr, nullptr);
}
```

如果我们能确定在标志检查和系统调用之间没有运行信号处理程序，那么我们就会安全。

Linux 4.18 引入了一个系统调用，`rseq`（“可重启序列”），让我们可以实现这一点，尽管需要一些努力。`rseq`机制的工作如下：

+   您编写一些您想要原子运行的代码，用于抢占或信号 - 临界区。

+   在进入临界区之前，我们通过在内核和用户空间之间共享的一小块内存中写入来通知内核即将运行临界区。

+   这块内存包含：

    1.  `start_ip`，标记临界区开始的指令指针；

    1.  `post_commit_offset`，临界区的长度；

    1.  `abort_ip`，如果内核需要抢占临界区，则要跳转到的指令指针。

+   如果内核抢占了一个线程，或者需要向线程交付信号，它会检查线程是否处于`rseq`关键部分，并且如果设置了程序计数器到`abort_ip`。

上述过程强制临界区成为一个连续的块（从`start_ip`到`start_ip+post_commit_offset`），我们必须知道其地址。这些要求迫使我们以内联汇编的方式编写它。

请注意，`rseq`并非完全禁用抢占，它让我们指定一些代码（从`abort_ip`开始的代码）在临界区被中断时执行一些清理工作。因此，临界区的正确工作通常取决于临界区末尾的“提交指令”，使临界区中的更改可见。

在我们的情况下，“提交指令”是`syscall` - 将调用我们感兴趣的系统调用的指令。

这导致了以下用于 6 个参数系统调用存根的 x86-64 小部件，用于原子性地检查停止标志并执行`syscall`：

```
// Returns -1 and sets errno to EINTR if `*stop` was true
// before starting the syscall.
long syscall_or_stop(bool* stop, long n, long a, long b, long c, long d, long e, long f) {
 long ret;
 register long rd __asm__("r10") = d;
 register long re __asm__("r8")  = e;
 register long rf __asm__("r9")  = f;
 __asm__ __volatile__ (
 R"(
 # struct rseq_cs {
 #     __u32   version;
 #     __u32   flags;
 #     __u64   start_ip;
 #     __u64   post_commit_offset;
 #     __u64   abort_ip;
 # } __attribute__((aligned(32)));
 .pushsection __rseq_cs, "aw"
 .balign 32
 1:
 .long 0, 0                # version, flags
 .quad 3f, (4f-3f), 2f     # start_ip, post_commit_offset, abort_ip
 .popsection

 .pushsection __rseq_failure, "ax"
 # sneak in the signature before abort section as
 # `ud1 <sig>(%%rip), %%edi`, so that objdump will print it
 .byte 0x0f, 0xb9, 0x3d
 .long 0x53053053
 2:
 # exit with EINTR
 jmp 5f
 .popsection

 # we set rseq->rseq_cs to our structure above.
 # rseq = thread pointer (that is fs) + __rseq_offset
 # rseq_cs is at offset 8
 leaq 1b(%%rip), %%r12
 movq %%r12, %%fs:8(%[rseq_offset])
 3:
 # critical section start -- check if we should stop
 # and if yes skip the syscall
 testb $255, %[stop]
 jnz 5f
 syscall
 # it's important that syscall is the very last thing we do before
 # exiting the critical section to respect the rseq contract of
 # "no syscalls".
 4:
 jmp 6f

 5:
 movq $-4, %%rax # EINTR

 6:
 )"
 : "=a" (ret) // the output goes in rax
 : [stop] "m" (*stop),
 [rseq_offset] "r" (__rseq_offset),
 "a"(n), "D"(a), "S"(b), "d"(c), "r"(rd), "r"(re), "r"(rf)
 : "cc", "memory", "rcx", "r11", "r12"
 );
 if (ret < 0 && ret > -4096) {
 errno = -ret;
 ret = -1;
 }
 return ret;
}

// A version of recvfrom which atomically checks
// the flag before running.
static long recvfrom_or_stop(bool* stop, int socket, void* buffer, size_t length) {
 return syscall_or_stop(stop, __NR_recvfrom, socket, (long)buffer, length, 0, 0, 0);
}
```

我们正在使用 glibc 最近添加的对 `rseq` 的支持，它提供了一个 `__rseq_offset` 变量，其中包含关键部分信息相对于线程指针的偏移量。在关键部分中，我们只需检查标志，如果设置了，则跳过系统调用，如果未设置，则执行系统调用。如果标志已设置，则假装系统调用失败，并显示 `EINTR`。

您可以在[这里](https://gist.github.com/bitonic/d3281b2d0fd95b4fd788aa7e013d1fb9)找到先前示例的完整代码，使用这种技巧调用`recvfrom`。我并不一定主张使用这种技术，但它绝对是一个有趣的奇特之处。

## 结束 #

在 Linux 线程中，中断和堆栈展开以及保护关键部分免受此类展开的方式尚未达成一致，这令人相当沮丧。虽然不存在技术障碍来实现这样的功能，但是清理拆卸通常是软件的被忽视的部分。

Haskell 是一种语言，其中存在这些功能，以[异步异常](https://hackage.haskell.org/package/base-4.19.0.0/docs/Control-Exception.html#v:throwTo)的形式存在，尽管仍然需要小心地适当地保护关键部分。

## 致谢 #

[Peter Cawley](https://twitter.com/corsix) 在本博客文章中讨论的许多细节上提供了意见并阅读了其草稿。他还提出了 `rseq` 作为可能的解决方案。还要非常感谢[Niklas Hambüchen](https://nh2.me/)、[Alexandru Sçvortov](https://scvalex.net/)、[Alex Sayers](https://www.asayers.com/) 和 Alex Appetiti 读过本博客文章的草稿。
