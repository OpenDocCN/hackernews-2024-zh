<!--yml
category: 未分类
date: 2024-05-28 17:56:39
-->

# Rust-Written Linux Scheduler Showing Promising Results For Gaming Performance - Phoronix

> 来源：[https://www.phoronix.com/news/Rust-Linux-Scheduler-Experiment](https://www.phoronix.com/news/Rust-Linux-Scheduler-Experiment)

A Canonical engineer has been experimenting with implementing a Linux scheduler within the Rust programming language. His early results are interesting and hopeful around the potential of a Rust-based scheduler that works via sched_ext for implementing a scheduler using eBPF that can be loaded during run-time.

Andrea Righi who is a Linux kernel engineer at Ubuntu maker Canonical

[tweeted](https://twitter.com/arighi/status/1746938387968254371)

that he's been experimenting with a Rust scheduler:

> "I ended up writing a Linux scheduler in Rust using sched-ext during Christmas break, just for fun. I'm pretty shocked to see that it doesn't just work, but it can even outperform the default Linux scheduler (EEVDF) with certain workloads (i.e., gaming)."

He shared a YouTube video showing a game with the "scx_rustland" scheduler outperforming the default Linux kernel scheduler while running a parallel kernel build in the background:

[https://www.youtube.com/embed/oCfVbz9jvVQ?si=dZCm3asl3SlytaQR](https://www.youtube.com/embed/oCfVbz9jvVQ?si=dZCm3asl3SlytaQR)

VIDEO

For those interested the code is hosted on

[GitHub](https://github.com/sched-ext/scx/)

. It's an interesting Christmas break adventure and will be interesting to see where it leads.