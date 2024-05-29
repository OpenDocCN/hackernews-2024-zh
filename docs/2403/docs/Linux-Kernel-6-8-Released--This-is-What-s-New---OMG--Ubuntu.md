<!--yml
category: 未分类
date: 2024-05-27 14:49:47
-->

# Linux Kernel 6.8 Released, This is What's New - OMG! Ubuntu

> 来源：[https://www.omgubuntu.co.uk/2024/03/linux-kernel-6-8-new-features](https://www.omgubuntu.co.uk/2024/03/linux-kernel-6-8-new-features)

**After several solid months of development the Linux 6.8 kernel has been officially released.**

This kernel is of particular note to Ubuntu users as it’s the version [chosen to ship in Ubuntu 24.04 LTS](https://www.omgubuntu.co.uk/2024/02/ubuntu-24-04-linux-kernel-6-8-likely) – i.e., as the GA kernel and thereby supported for the duration of the release.

Announcing the release of Linux kernel 6.8 on the official Linux Kernel Mailing List (LKML) Linux founder [Linus Torvalds says](https://lkml.org/lkml/2024/3/10/243):

*“This is not the historically big release that 6.7 was – we seem to be back to a fairly average release size for the last few year.”*

Adding: *“You can see it in the overall diffstats too – this looks like an average release in pretty much all respects, and we don’t have (for example) any obvious big new filesystems or architectures. I think the biggest single new thing in 6.8 is probably the new Xe drm driver, but honestly, the big bulk of changes are just various random updates and fixes all over.”*

So what’s new exactly?

## Linux 6.8: New Features

As you’d expect, Linux kernel 6.8 includes plenty of prep, bring-up, and early enablement for hardware and hardware-enabled features most of us aren’t *currently* using.

This includes the experimental Intel Xe DRM driver Linus mentions in his release announcement, plus further support for AMD Zen 5 and other upcoming AMD hardware, initial code for Qualcomm Snapdragon 8 Gen 3 (and related SoCs), and similar.

> Forget enablement for upcoming hardware – for me, the exciting kernel changes are the ones I can benefit from today!

But for me, the truly exciting Linux kernel changes are the ones I can feel, benefit from, or make use of myself right now.

And thankfully Linux 6.8 comes with a bunch of them!

**Linux 6.8 adds adds Raspberry Pi 5 support to the V3D DRM driver,** plus intros GPUTop and FDINFO. Any distro pairing Mesa 23.3 with Linux 6.8 will now provide a solid graphics experience out-of-the-box on the Pi 5, no extra kernel patches required.

~~This change should ensure Ubuntu 24.04 LTS runs *sweet as pie* on the [Raspberry Pi 5](https://www.omgubuntu.co.uk/2023/09/raspberry-pi-5-officially-announced).~~ Turns out Ubuntu already runs sweet as pie as it’s based on the official [Raspberry Pi kernel](https://github.com/raspberrypi/linux) patch set). But this mainline addition is something other Linux distros will benefit from.

As of this kernel reversion the [zswap](https://docs.kernel.org/admin-guide/mm/zswap.html) subsystem is now able to force cold pages out to real swap if memory pressure gets too much (with opt-out for those who don’t wish to use this). There’s also a **new zswap mode to disable writing back to swap entirely**.

Linux kernel 6.8 can prevent direct writes to block devices with mounted filesystems (excepting Btrfs for now). Devs say writing to mounted devices may lead to filesystem corruption and crashes. This feature is disabled by default but it’s reasoned most Linux distros will choose to enable it.

An adjustment to the Intel P-State CPU frequency scaling driver will see devices powered by Intel ‘Meteor Lake’ CPUs (released at the end of last year) hit their advertised ‘boost’ speeds under Linux. In the previous kernel [they were found](https://www.spinics.net/lists/kernel/msg5059502.html) to be running ~100MHz under.

> Intel Core Ultra Mobile processors will finally hit their advertised boost speeds with Linux kernel 6.8

If you use Linux on the *Lenovo ThinkPad X1 Carbon (Gen 12)*, *Acer Swift Go 14*, *ASUS Expertbook B5*, or any other laptop powered by Intel Core Ultra Mobile processorsexpect a pinch **more performance during peak loads with this Linux kernel**.

On the subject of portables, AMD Ryzen 7000 (and upcoming Ryzen 8000) laptops were suffering from radio frequency interference (RFI) from the Wi-Fi and GPU memory clocks. Linux 6.8 includes **AMD RFI mitigations** (WBRF) to resolve this.

Network-related: Linux 6.8 includes networking buffs that provide better cache efficiency. This is [said to <q>improve</q>](https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=3e7aeb78ab01)*<q>“TCP performances with many concurrent connections up to 40%</q>”* – a sizeable uplift, though to what degree most users will benefit is unclear.

Are you a Linux gamer? If so, take note that Linux 6.8 gains support for:

*   **Nintendo Switch Online controllers**
*   **Powkiddy X5 & RK2023 handheld consoles**
*   **Adafruit Mini I2C gamepads**
*   **Lenovo Legion Go controllers**
*   **Colour management on the Steam Deck**

Driver fixes for the official Steam controller are also included.

In addition to the above there are some other choice highlights in Linux 6.8:

*   **New** `statmount()` **and** `listmount()` **system calls**
*   **New [deadline servers](https://lwn.net/Articles/934415/) mechanism**
*   **Rust kernel support for LoongArch CPUs**
*   **Possible to change the size of tracing sub-buffers**
*   **Guest-first memory feature for KVM**
*   **KSM advisor for auto-tuning kernel samepage merging subsystem**
*   **11% (or so) higher sys call entry performance on IBM Z**
*   **New PHY network driver written in Rust**
*   **Intel Trust Domain Extensions (TDX) host-side support**
*   **Intel IAA compression accelerator**
*   **dmesg info on whether 32-bit support is disabled at boot**
*   `perf` **tool now supports data-type profiling**
*   **Apple M1 Thunderbolt DART support**
*   **Bcachefs gains initial online filesystem check and repair**
*   **AppArmor switches to SHA-256 for policy-hash verification**

Finally, while few (if any) of us run Linux on RISC-V boards there’s no denying that this open-source processor architecture has a very bright future.

Brighter still as Linux 6.8 works with AMD’s MicroBlaze V soft-core RISC-V CPU; adds XIP kernel features and `riscv_hwprobe()` system calls; can **suspend to RAM on RISC-V** when SUSP SBI extension is present; ships a new camera subsystem driver for the StarFive SoC.

For more details on all of the above I (as ever) recommend sifting through the 2-part LWN merge roundups ([part 1](https://lwn.net/Articles/957188/) & [part 2](https://lwn.net/Articles/958178/)) which offer a concise, waffle-free overview *plus* links to related in-depth and further info where needed.

## Getting Linux kernel 6.8

As for getting Linux 6.8, you can download the source code right now to compile the kernel by hand. Bit much; you’re best off waiting for your Linux distribution to package Linux kernel 6.8 officially and push it out to you as a software update.

Next month you can install or upgrade to Ubuntu 24.04 LTS which includes Linux 6.8 by default (and this will be back ported to Ubuntu 22.04 LTS in the next HWE/ Ubuntu 22.04.5 LTS).

Though other Linux blogs suggest it, **using Canonical’s mainline kernel builds is not encouraged** (not least because they’re not signed so may fail to boot in certain situations, won’t get security updates, don’t contain certain patches, etc).

That said, some folks do install Canonical mainline kernel builds in Ubuntu (though some only temporarily for testing). If you can’t wait to get your mitts on Linux kernel 6.8 then those pre-packaged DEBs are *an* option but not an advised one – use ’em at your own risk!