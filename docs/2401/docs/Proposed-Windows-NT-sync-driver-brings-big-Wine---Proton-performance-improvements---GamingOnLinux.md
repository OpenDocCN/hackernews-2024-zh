<!--yml
category: 未分类
date: 2024-05-27 15:09:19
-->

# Proposed Windows NT sync driver brings big Wine / Proton performance improvements | GamingOnLinux

> 来源：[https://www.gamingonlinux.com/2024/01/proposed-windows-nt-sync-driver-brings-big-wine-proton-performance-improvements/](https://www.gamingonlinux.com/2024/01/proposed-windows-nt-sync-driver-brings-big-wine-proton-performance-improvements/)

Zeb Figura of CodeWeavers [has proposed](https://lore.kernel.org/lkml/20240124004028.16826-1-zfigura@codeweavers.com/) a Windows NT synchronization primitive driver to help performance of running games and applications designed for Windows on Linux with Wine / Proton.

As per the RFC (request for comments) posted onto the Linux kernel mailing list Figura notes: "This patch series introduces a new char misc driver, /dev/ntsync, which is used to implement Windows NT synchronization primitives."

Then giving some background on it: "The Wine project emulates the Windows API in user space. One particular part of
that API, namely the NT synchronization primitives, have historically been implemented via RPC to a dedicated "kernel" process. However, more recent applications use these APIs more strenuously, and the overhead of RPC has become a bottleneck.

The NT synchronization APIs are too complex to implement on top of existing primitives without sacrificing correctness. Certain operations, such as NtPulseEvent() or the "wait-for-all" mode of NtWaitForMultipleObjects(), require direct control over the underlying wait queue, and implementing a wait queue sufficiently robust for Wine in user space is not possible. This proposed driver, therefore, implements the problematic interfaces directly in the Linux kernel."

Figura did a presentation on it at the Linux Plumbers Conference in 2023 that you can see below:

What kind of performance difference might we see with something like this? The difference depends on your hardware and what you're trying to run but some FPS examples were given from multiple users including:

*What hardware and software each was tested on was not noted.*

Some impressive numbers but again it's hardware dependent, but even so any performance uplift would be welcome, so hopefully the patch series gets some good comments and approval after any tweaks are done that may be needed. Looks like we could end up seeing a nice future performance improvement for Linux and Steam Deck gaming with Wine and Proton.

*Source: [Phoronix](https://www.phoronix.com/news/Windows-NT-Sync-RFC-Linux)*

Article taken from [GamingOnLinux.com.](https://www.gamingonlinux.com/)