<!--yml

category: 未分类

date: 2024-05-27 14:36:55

-->

# pause(2)的危险

> 来源：[https://www.cipht.net/2023/11/30/perils-of-pause.html](https://www.cipht.net/2023/11/30/perils-of-pause.html)

在`pause(2)`的情况下，我们可以使用`sigsuspend(2)`或其他几个类似的函数。如果我们需要信号处理程序，我们的代码可能如下所示：

```
if (sigaction(sig, &(struct sigaction){.sa_handler=h}, NULL))
    abort();
sigset_t set, prev;
sigemptyset(&set);
sigaddset(&set, sig);
sigprocmask(SIG_BLOCK, &set, &prev);
write(1, "ok\n", 3);
do { sigsuspend(&prev); } while (sig != got);

```

但在这种情况下，我们只是等待这个信号，并且不需要在其到达时采取其他操作，因此`sigwait(2)`就足够了：

```
sigset_t set, prev;
sigemptyset(&set);
sigaddset(&set, sig);
sigprocmask(SIG_BLOCK, &set, &prev);
write(1, "ok\n", 3);
int got;
do { sigwait(&set, &got); } while (sig != got);

```

如果我们需要比仅信号号码更多的细节，我们可能会使用`sigwaitinfo`或`sigtimedwait`。如果我们确信不会收到其他信号，我们可以完全避免循环，但保护对于那些可能考虑不到的情况（例如在交互式测试程序时按下^Z发送SIGTSTP）是个好习惯。

（请注意，OpenBSD目前也缺少`sigwaitinfo`和`sigtimedwait`。）

对于`poll(2)`和`select(2)`，存在解除掩码变体`ppoll(2)`和`pselect(2)`，因此存在这个原因。（Linux还具有`signalfd(2)`，它与轮询循环更自然地集成，但请注意它仅读取挂起的信号，因此仍需使用`sigprocmask`进行掩码，并且现在必须处理从缓冲区读取`siginfo`的情况。哦，实际从fd读取的内容取决于哪个进程正在读取…）还有经典的[self-pipe技巧](https://cr.yp.to/docs/selfpipe.html)。
