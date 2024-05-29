<!--yml
category: 未分类
date: 2024-05-27 14:49:30
-->

# Linux 6.9 will be the first to top ten million Git objects • The Register

> 来源：[https://www.theregister.com/2024/03/11/linux_6_8_arrives/](https://www.theregister.com/2024/03/11/linux_6_8_arrives/)

Linus Torvalds has released version 6.8 of the Linux Kernel.

“So it took a bit longer for the commit counts to come down this release than I tend to prefer,” Torvalds [wrote](https://lkml.iu.edu/hypermail/linux/kernel/2403.1/01820.html) on the Linx kernel mailing list on Sunday, “but a lot of that seemed to be about various selftest updates (networking in particular) rather than any actual real sign of problems.”

“And the last two weeks have been pretty quiet, so I feel there's no real reason to delay 6.8.”

So he delivered it, ending his own [speculation](https://www.theregister.com/2024/03/04/linux_6_8_rc_7/) that this cut of the kernel might need an eighth release candidate.

Torvalds found time to note what he described as “a bit of random git numerology” as when work ended on this version of the kernel the git repository used to track it contained 9.996 million objects.”

“This is the last mainline kernel to have less than ten million git objects,” Torvalds wrote. “Of course, there is absolutely nothing special about it apart from a nice round number. Git doesn't care,” he added.

Fair enough – especially as noted that other trees, such as linux-next, have well and truly passed ten million objects.

Torvalds rated the addition of a new Xe drm driver as the most significant addition to this version of the kernel. The driver powers Intel GPUs, whether they’re integrated into a CPU or standalone devices, but is considered experimental.

Another addition to the kernel adds support for Amazon Web Services’ “Nitro”, the cloud giant’s isolation tech that shifts security and networking chores to a SmartNIC. A new driver exposes the Nitro hardware to the kernel, allowing Linux guests to access the services it offers.

The bcachefs filesystem, one of the big additions to Linux 6.7, received some fixes that helped to improve performance. Another enhancement brought the Rust-featuring kernel to CPUs using the made-in -China LoongArch architecture.

The Raspberry Pi 5’s graphics hardware received more support, and Nintendo’s Switch Online controllers are now supported by Linux.

Torvalds’ post ended with a call for kernel devs to test version 6.8 kernel, plus thanks to those who have already submitted pulls for version 6.9 of the kernel before its merge window opened. ®