<!--yml
category: 未分类
date: 2024-05-27 14:36:55
-->

# the perils of pause(2)

> 来源：[https://www.cipht.net/2023/11/30/perils-of-pause.html](https://www.cipht.net/2023/11/30/perils-of-pause.html)

In the case of `pause(2)`, we can use `sigsuspend(2)`, or a few other similar functions. If we need the signal handler, our code might look like this:

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

but in this case, we are just waiting for this signal, and don't need to take other action when it arrives, so `sigwait(2)` suffices:

```
sigset_t set, prev;
sigemptyset(&set);
sigaddset(&set, sig);
sigprocmask(SIG_BLOCK, &set, &prev);
write(1, "ok\n", 3);
int got;
do { sigwait(&set, &got); } while (sig != got);

```

We might use `sigwaitinfo` or `sigtimedwait` if we need more details than just the signal number. If we were certain no other signal could arrive, we could avoid the loop entirely, but it's nice to protect against cases you might not otherwise consider, like testing the program interactively and hitting ^Z (sending SIGTSTP).

(Note that OpenBSD also lacks `sigwaitinfo` and `sigtimedwait` presently.)

For `poll(2)` and `select(2)`, unmasking variants `ppoll(2)` and `pselect(2)` exist for this reason. (Linux also has `signalfd(2)`, which more naturally integrates with polling loops, but note it only reads pending signals, so you still need to mask with `sigprocmask`, and now you have to deal with reading `siginfo` out of a buffer. Oh, and what you actually get out of the fd depends on which process is reading…) There's also the classic [self-pipe trick](https://cr.yp.to/docs/selfpipe.html).