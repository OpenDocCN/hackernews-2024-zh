<!--yml
category: 未分类
date: 2024-05-27 15:22:06
-->

# On the Costs of Syscalls | Georg's Log

> 来源：[https://gms.tf/on-the-costs-of-syscalls.html](https://gms.tf/on-the-costs-of-syscalls.html)

It's well known that [syscalls](https://en.wikipedia.org/wiki/System_call) are expensive. And that software mitigations against [CPU bugs](https://en.wikipedia.org/wiki/Transient_execution_CPU_vulnerability) (such as Meltdown) even have made them more expensive. But how expensive are they really? To begin to answer this question I wrote a small [micro-benchmark](https://github.com/gsauthof/osjitter/blob/master/bench_syscalls.cc) in order to measure the minimal costs of a syscall. Meaning the cost of syscalls one always has to pay whether a context-switch happens or not, even when the work in the kernel is minuscule, i.e. the costs of switching from [user-mode to kernel-mode](https://en.wikipedia.org/wiki/Context_switch#User_and_kernel_mode_switching) and back.

## Methods

The user-kernel mode-switch [micro-benchmark](https://en.wikipedia.org/wiki/Benchmark_(computing)#Types_of_benchmark) uses Google's [benchmark library](https://github.com/google/benchmark) for the measurements and is [available in a git repository](https://github.com/gsauthof/osjitter/blob/master/bench_syscalls.cc). The repository also contains some helper scripts, e.g. a [playbook](https://github.com/gsauthof/osjitter/blob/master/helper/bench_playbook.py) for distributing it to and executing it on a bunch of hosts. The benchmark library repeats each case until the result is considered stable and the playbook allows for repeated executions of the test cases. In the following sections the median value of 100 repetitions is reported (real time in nanoseconds).

For the benchmark a bunch of syscalls is called that are expected to be very cheap, such as getting the user id (UID), the program id (PID), closing an invalid file descriptor, calling an non-existent syscall etc. Thus, a measurement should really just include two mode switches. As controls, a few cases don't call a syscall but do other cheap stuff.

I ran the benchmark on a heterogeneous set of hosts, i.e. different kernels, operating systems and configurations. For more details see also the Hosts Section.

## Results and Discussion

The following table shows the real time (ns) for the different cases:

| host | 5i4250u | 7i6600u | ac3758 | x2643 | x2667h | x2667s | x2687w | x2689 | x2690 | xg6144 | xg6148 | xg6246 | xg6256 | xg6256b | xs4110 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| name |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| assign | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 |
| clock_gettime | 23 | 21 | 31 | 19 | 16 | 16 | 17 | 14 | 21 | 14 | 24 | 14 | 13 | 13 | 24 |
| clock_gettime_mono | 23 | 22 | 32 | 16 | 16 | 16 | 17 | 14 | 21 | 15 | 24 | 14 | 13 | 13 | 24 |
| clock_gettime_mono_raw | 23 | 22 | 33 | 542 | 544 | 350 | 582 | 332 | 762 | 660 | 427 | 274 | 122 | 290 | 218 |
| clock_gettime_tai | 23 | 21 | 32 | 542 | 546 | 352 | 587 | 333 | 762 | 660 | 427 | 274 | 122 | 292 | 218 |
| close | 568 | 262 | 275 | 484 | 495 | 283 | 514 | 277 | 668 | 610 | 356 | 243 | 93 | 257 | 145 |
| getpid | 558 | 257 | 255 | 2 | 1 | 1 | 2 | 1 | 2 | 1 | 2 | 1 | 1 | 1 | 2 |
| getuid | 560 | 231 | 259 | 464 | 473 | 276 | 505 | 259 | 649 | 592 | 347 | 224 | 78 | 239 | 137 |
| nothing | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 |
| pthread_cond_signal | 3 | 2 | 6 | 14 | 13 | 12 | 14 | 11 | 15 | 10 | 17 | 10 | 10 | 10 | 17 |
| sched_yield | 706 | 374 | 430 | 569 | 560 | 414 | 634 | 346 | 773 | 694 | 454 | 280 | 126 | 300 | 232 |
| sqrt | 6 | 2 | 15 | 4 | 4 | 4 | 4 | 1 | 7 | 1 | 3 | 1 | 1 | 1 | 3 |
| sqrtrec | 4 | 4 | 15 | 2 | 3 | 2 | 3 | 2 | 3 | 3 | 5 | 3 | 3 | 3 | 5 |
| syscall | 560 | 252 | 265 | 440 | 460 | 269 | 497 | 243 | 620 | 579 | 345 | 221 | 76 | 233 | 136 |

### Controls

The cases used as controls are 'nothing' which literally does nothing, 'assign' which just assigns to a variable, `sqrt` which computes the square root of a small constant and `sqrtrec` which stacks a bunch of sqrt calls. The results for these are plausible, i.e. doing nothing is really measured as `10**-7` ns or so, the assignment costs 0.5 ns or so and computing the square root takes only a few ns. Perhaps the most remarkable result is, that computing the square root on a Atom CPU (ac3758) is pretty constant over the 2 cases, whereas on the other hosts its runtime depends on its argument.

### Clock Gettime

Looking at the syscalls, one relation that holds true on all hosts is that the `clock_gettime(CLOCK_REALTIME)` syscall is much faster than `getuid()` or `close()`. This can be explained by the fact that on Linux, `clock_gettime(CLOCK_REALTIME)` and a [few other syscalls](https://manpath.be/f34/7/vdso#L334) are implemented via the efficient [vDSO](https://en.wikipedia.org/wiki/VDSO) mechanism. Meaning when they are called no mode switch happens!

`clock_gettime()` supports different clocks and not all of them are vDSO optimized on all kernels. The table shows that on RHEL 7 querying `CLOCK_MONOTIC_RAW` and `CLOCK_TAI` invokes a real syscall while on Fedora 33 kernels (5.12/5.13) these clock readings are also implemented as vDSO.

### Dummy Signaling

Similarly, the dummy `pthread_cond_signal()` case, which signals without anybody listening, is much cheaper than a real syscall - since the C library doesn't have to call a real syscall but can just invoke some relatively cheap atomic operation.

### Getpid

The `getpid()` syscall is surprisingly fast on RHEL 7\. It turns out that RHEL 7 ships an [older glibc version which caches the ID of a process](https://manpath.be/f34/2/getpid#L43)! Which arguably is a curious optimization, since, what's the point? I mean how often to you have to call `getpid()` in a program, really? At some point (around Fedora 26) this [feature was removed](https://bugzilla.redhat.com/show_bug.cgi?id=1469670) since it apparently caused more [trouble](https://yarchive.net/comp/linux/getpid_caching.html) than it's worth. Perhaps unsurprisingly, that [removal](https://bugzilla.redhat.com/show_bug.cgi?id=1469670) even [broke somebodies workflow](https://xkcd.com/1172/).

### Real Syscalls

So looking at the real syscalls, the user-kernel mode switches cost in the order of a few hundred nanoseconds, on all hosts. The higher costs on some hosts can be explained by CPU bug mitigations being enabled (they are enabled by default) and/or kind of older/lower-end hardware. See also the Hosts Section for some details.

The fastest host is `xg6256` which manages to switch modes in less than 100 ns. It has a fast CPU with good single core performance (Xeon Gold 6256), has frequency scaling disabled and runs at a constant 4.1 GHz frequency above it's base frequency (i.e. a frequency between the base and turbo frequency).

### Sched Yield

The `sched_yield()` syscall could be considered as a minimal work syscall, e.g. when there is nothing to yield to. Also, the benchmark process is running under the standard scheduling policy and on Linux [`sched_yield()` is described as](https://manpath.be/f34/2/sched_yield#L43):

> `sched_yield()` is intended for use with real-time scheduling policies (i.e., `SCHED_FIFO` or `SCHED_RR`). Use of `sched_yield()` with nondeterministic scheduling policies such as `SCHED_OTHER` is unspecified and very likely means your application design is broken.

So unspecified could mean that the syscall just bails out after the process' scheduling policy compares equal to `SCHED_OTHER`.

On most hosts sched_yield 150 ns or so more expensive than a really minimal syscall such as `getuid()` - which indicates some more overhead, but not necessarily a context switch.

### Nanosleep

A bit out of the competition is the `nanosleep()` syscall:

| host | 5i4250u | 7i6600u | ac3758 | x2643 | x2667h | x2667s | x2687w | x2689 | x2690 | xg6144 | xg6148 | xg6246 | xg6256 | xg6256b | xs4110 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| name |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| nanosleep0 | 52632 | 50474 | 52620 | 50588 | 50003 | 50011 | 50000 | 50014 | 50312 | 50018 | 50000 | 50014 | 50000 | 50000 | 54866 |
| nanosleep0_slack1 | 4355 | 2836 | 7076 | 3247 | 2736 | 2483 | 2835 | 3908 | 3401 | 2762 | 2870 | 2446 | 1974 | 2248 | 3837 |
| nanosleep1_slack1 | 4348 | 2840 | 7102 | 3252 | 2736 | 2486 | 2834 | 3908 | 3410 | 2767 | 2871 | 2446 | 1975 | 2246 | 3836 |

Calling `nanosleep()` to sleep for 0 ns or 1 ns seemingly also is a very cheap syscall or even a null operation.

However, in the first case it takes 50 µs an all hosts. Incidentally, 50 µs is also the [default timer slack value](https://manpath.be/f34/2/prctl#L1052) for a normally scheduled process on Linux. The [timer slack mechanism](https://lwn.net/Articles/588086/) extends timer expirations up to the slack value in order to group multiple timers since this reduces wake ups and thus saves energy. And since `nanosleep()` creates a timer, it's also affected by this mechanism.

Thus, the other nanosleep cases [set a minimal timer slack](https://github.com/gsauthof/osjitter/blob/f1a4ca9cbf7516efc61c3bab2fe06ffe83cfb43c/bench_syscalls.cc#L117) of 1 ns which reduces the runtime, as expected. However, it's still much more expensive than the other syscalls. Of course, a timer expiration has a limited accuracy. However, with 0 ns or 1 ns no timer has to be expired, really. It turns out that calling `nanosleep()` unconditionally yields a (voluntary) [context switch](https://en.wikipedia.org/wiki/Context_switch). Even on isolated cores, where the scheduler happily switches to the swapper kernel thread. Thus, the last two nanonsleep cases really measure the context switch costs which are more expensive than a simple mode switch.

The costs of a context switch match what [others are measuring](https://eli.thegreenplace.net/2018/measuring-context-switching-and-memory-overheads-for-linux-threads/#how-expensive-are-context-switches) (modulo division by two).

## Hosts

The following table shows the hosts under benchmark:

|  | host | CPU | mitigations | poll | os | kernel |
| --- | --- | --- | --- | --- | --- | --- |
| 0 | 5i4250u | Core i5-4250U | yes | yes | Fedora 33 | 5.12 |
| 1 | 7i6600u | Core i7-6600U | yes | yes | Fedora 33 | 5.13 |
| 2 | ac3758 | Atom C3758 | no | yes | Fedora 33 | 5.13 |
| 3 | x2643 | Xeon E5-2643 v2 | yes | no | RHEL 7 | 3.10 |
| 4 | x2667h | Xeon E5-2667 v3 | yes | yes | RHEL 7 | 3.10 |
| 5 | x2667s | Xeon E5-2667 v3 | yes | yes | RHEL 7 | 3.10 |
| 6 | x2687w | Xeon E5-2687W v3 | yes | yes | RHEL 7 | 3.10 |
| 7 | x2689 | Xeon E5-2689 v4 | yes | yes | RHEL 7 | 3.10 |
| 8 | x2690 | Xeon E5-2690 0 | yes | yes | RHEL 7 | 3.10 |
| 9 | xg6144 | Xeon Gold 6144 | yes | yes | RHEL 7 | 3.10 |
| 10 | xg6148 | Xeon Gold 6148 | no | yes | RHEL 7 | 3.10 |
| 11 | xg6246 | Xeon Gold 6246 | yes | yes | RHEL 7 | 3.10 |
| 12 | xg6256 | Xeon Gold 6256 | no | yes | RHEL 7 | 3.10 |
| 13 | xg6256b | Xeon Gold 6256 | yes | yes | RHEL 7 | 3.10 |
| 14 | xs4110 | Xeon Silver 4110 | no | yes | RHEL 7 | 3.10 |

Notes:

*   the kernels are the ones packaged by the distributions
*   most RHEL hosts are on RHEL 7.9
*   CPU mitigations are disabled via the `mitigations=off` kernel parameter or similar parameters
*   polling means that CPU frequency scaling and power saving is disabled via kernel parameters and tuned PM QoS settings
*   so the host's CPU runs on a fixed frequency; where possible this frequency is set slightly above the base frequency, e.g. on the Xeon Gold 6256 CPU it's set to 4.1 GHz
*   the Atom CPU doesn't support Hyperthreading and Hyperthreading is disabled on all Xeon hosts
*   all hosts have SELinux and/or Auditing enabled (on Fedora/RHEL these features are enabled by default) which adds some syscall overhead to some degree

## Terminology

There are basically two important separate terms to distinguish in the above discussion:

1.  [Mode Switch](https://en.wikipedia.org/wiki/Context_switch#User_and_kernel_mode_switching) (or Mode Transition)
2.  [Context Switch](https://en.wikipedia.org/wiki/Context_switch)

The definitions of these terms might vary in different literature and different operating systems. Also, in other contexts (no pun intended!) one might describe different modes as different contexts. However, the definitions given in the linked Wikipedia articles are widely used and apply to Linux.

Basically, a mode transition denotes the switch between user mode and kernel mode (or between user space and kernel space) whereas a context switch denotes a switch between different tasks, which is facilitated by the kernel. A context switch requires more work than a mode switch and thus is more expensive.