<!--yml
category: 未分类
date: 2024-05-27 14:32:13
-->

# HardenedBSD February 2024 Status Report | HardenedBSD

> 来源：[https://hardenedbsd.org/article/shawn-webb/2024-03-01/hardenedbsd-february-2024-status-report](https://hardenedbsd.org/article/shawn-webb/2024-03-01/hardenedbsd-february-2024-status-report)

I spent most of February getting 15-CURRENT working again. FreeBSD introduced a new library, libsys, which is where the userland side of performing syscalls is implemented. There's an intricate dance between libsys, libc, libthr, and the CSU. I spent some time learning about that dance, and I still feel like there's more to learn.

HardenedBSD 15-CURRENT is mostly fixed. Prior to the libsys change, we built libc with Link-Time Optimizations (LTO). Building libc with LTO was part of the problem, though not the only issue. Once all the issues are resolved, I will re-enable LTO for libc.

FreeBSD also introduced a new pam_xdg(8) PAM module. This module had a few vulnerabilities, which are fixed in HardenedBSD. The two NULL deref bugs are fixed in FreeBSD now, too. The filesystem race condition and recursion limit issues are somewhat mitigated in HardenedBSD, but not completely.

HardenedBSD now has two VisionFive StarFive2 64-bit RISCV SBCs. I spent a little bit of time toying around with them. The kernel boots to the mountroot prompt. I've been wanting to learn hardware hacking, including writing drivers, so these little SBCs might be great for that.

**In ports:**

1.  the u-boot ports are fixed
2.  dns/unbound was updated to 1.19.1
3.  the net/vnstat port was fixed
4.  graphics/mupdf is now built as a PIE with SafeStack enabled
5.  The secadm ports were updated

Looking forward into March: I'm hoping to close two gaps of knowledge: the dance mentioned above and I'd like to return to jemalloc hardening. I plan to also do some infrastructure maintenance--routine updates.