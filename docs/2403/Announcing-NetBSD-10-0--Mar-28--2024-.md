<!--yml
category: 未分类
date: 2024-05-29 12:47:37
-->

# Announcing NetBSD 10.0 (Mar 28, 2024)

> 来源：[https://www.netbsd.org/releases/formal-10/NetBSD-10.0.html](https://www.netbsd.org/releases/formal-10/NetBSD-10.0.html)

# Announcing NetBSD 10.0 (Mar 28, 2024)

The NetBSD Project is pleased to announce NetBSD 10.0, the eighteenth major release of the NetBSD operating system.

It represents cumulative improvements to the operating system since NetBSD 9.x was originally branched in 2019.

An existing installation can be upgraded by booting an installation image and selecting the Upgrade option.

If you are using other update methods, update the kernel and modules *first*, then reboot and update your userspace. You will need to adjust any package repository URL and update all third-party packages. Note also the addition of the new `gpufw` set, which may need to be installed separately with [sysinst(8)](//man.NetBSD.org/NetBSD-10.0/sysinst.8).

Please take particular note of *Changes to system behaviour and compatibility* if you are upgrading from an earlier release.

### Performance and scalability

[Benchmarks of NetBSD 10](https://mail-index.NetBSD.org/current-users/2020/11/07/msg039815.html) show huge performance and scalability gains over NetBSD 9.x, especially on multiprocessor and multicore systems, for compute and filesystem-bound applications. Areas of improvement included:

*   Switched the kernel's file path lookup cache to use faster per-directory red-black trees.
*   Improved scheduler performance, including the ability to more appropriately spread load on a mixture of slow and fast cores (e.g. big.LITTLE Arm CPUs).
*   Various optimizations for the machine-independent virtual memory system:

    *   Switched to a faster radix tree algorithm for memory page lookups.
    *   Improved tracking of clean/dirty pages, speeding up [fsync(2)](//man.NetBSD.org/NetBSD-10.0/fsync.2) on large files by orders of magnitude.
    *   Improved parallelization: rewritten page allocator with awareness of CPU topology, replaced global counters with per-CPU counters, and reduced lock contention.

*   Improved the performance of the [select(2)](//man.NetBSD.org/NetBSD-10.0/select.2) and [poll(2)](//man.NetBSD.org/NetBSD-10.0/poll.2) system calls.
*   Improved the performance of `tmpfs`. Implemented lazy update of atime/mtime.
*   Various optimizations of architecture-dependent x86 and AArch64 code, vastly improved network and I/O throughput on aarch64.
*   Various boot speed improvements.

### Security and quality assurance

*   Compatibility with WireGuard:

    *   A new interface, [wg(4)](//man.NetBSD.org/NetBSD-10.0/wg.4), provides a VPN tunnel compatible with the WireGuard specification. *The driver is experimental and needs more testing.*
    *   A userspace implementation using a rump kernel server is also included, see [wg-userspace(8)](//man.NetBSD.org/NetBSD-10.0/wg-userspace.8)
    *   The NetBSD implementation works with WireGuard implementations used by commercial VPN providers, Android, Linux, and more.

*   Stronger, faster cryptography:

    *   Added an implementation of the Adiantum cipher for efficient disk encryption with [cgd(4)](//man.NetBSD.org/NetBSD-10.0/cgd.4) on systems without AES acceleration.
    *   Added support for shared keys to [cgdconfig(8)](//man.NetBSD.org/NetBSD-10.0/cgdconfig.8), allowing multiple derived subkeys to be used across multiple drives.
    *   Switched the default password hashing algorithm to Argon2id, winner of the Password Hashing Competition. The algorithm's hardness automatically scales with system performance. Added support for Argon2id to [cgdconfig(8)](//man.NetBSD.org/NetBSD-10.0/cgdconfig.8) for use in password-based disk encryption.
    *   The kernel now takes advantage of CPU acceleration and vectorization for common cryptographic algorithms on x86 and Arm, including AES and ChaCha. All in-kernel implementations of AES are now constant-time on all architectures.
    *   Swap encryption is now automatic using the `vm.swap_encrypt=1` [sysctl(8)](//man.NetBSD.org/NetBSD-10.0/sysctl.8) variable.

*   Support for new Armv8-A security features:

    *   *Privileged Access Never* - helps prevent inadvertent userspace memory access by the kernel.
    *   *Pointer Authentication* - helps defend against return-oriented programming attacks on buffer overrun.
    *   *Branch Target Identification* - limits the locations to which branch instructions can jump.

*   More sanitizers, testing capabilities, and quality assurance:

    *   *Kernel Concurrency Sanitizer* - detects race conditions in the kernel at runtime.
    *   *Kernel Memory Sanitizer* - detects uninitialized memory in the kernel at runtime.
    *   A new virtual USB host driver (vHCI) allows fuzzing and detecting bugs in USB drivers from userspace, even if the hardware is unavailable to developers.
    *   More than 2000 new test cases were added.
    *   Completed various kernel-wide audits of internal API usage: [membar_ops(3)](//man.NetBSD.org/NetBSD-10.0/membar_ops.3), [autoconf(9)](//man.NetBSD.org/NetBSD-10.0/autoconf.9), device detach...
    *   Added restrictions on hardlink creation to [secmodel_extensions(9)](//man.NetBSD.org/NetBSD-10.0/secmodel_extensions.9).

*   Improved support for Arm:

    *   Allwinner V3s SoC support, found in e.g. the Lichee Pi Zero.
    *   Amlogic G12 SoC support, found in e.g. the ODROID-N2+.
    *   [Apple M1](https://wiki.NetBSD.org/ports/evbarm/apple/) SoC support, e.g. the M1 Mac Mini.
    *   NXP i.MX 8M SoC support, found in e.g. the HummingBoard Pulse.
    *   NXP i.MX 6SoloX SoC support, found in e.g. the UDOO Neo Full.
    *   Raspberry Pi 4 support. Boot NetBSD from USB with [EDK II UEFI firmware](https://github.com/pftf/RPi4) installed to the SD card, or copy EDK II to the `/boot` partition.
    *   Rockchip RK356X support, found in e.g. the PINE64 Quartz64 (with [EDK II UEFI firmware](https://github.com/jaredmcneill/quartz64_uefi) installed).
    *   Rockchip RK3588 support, found in e.g. the Orange Pi 5 (with [EDK II UEFI firmware](https://github.com/edk2-porting/edk2-rk3588) installed)
    *   Rockchip RK3288 support, found in e.g. the Asus Tinker Board.
    *   Added support for booting the Raspberry Pi 0-3 in big endian mode.
    *   Added support for ACPI Collaborative Processor Performance Control, used for CPU performance adjustment on ServerReady hardware.
    *   Added support for [compat_linux(8)](//man.NetBSD.org/NetBSD-10.0/compat_linux.8) on AArch64, making it possible to run Linux userspace programs when the `compat_linux` module is enabled in `/etc/modules.conf`.
    *   Added support for spiflash on Rockchip RK3328.
    *   Moved support for the Xilinx Zynq-7000 into the `GENERIC` evbarm kernel (based on FDTs).
    *   Enabled the `rkv1crypto` driver on the PINE64 Rock64 and NanoPi R2S.
    *   UEFI bootloader improvements: support for other-endian FFS file systems, booting from [raid(4)](//man.NetBSD.org/NetBSD-10.0/raid.4) volumes, ISO9660 (.iso file system) support, `boot.cfg` support, `gop` command for changing the video mode, loading kernel modules directly from the bootloader.

*   New drivers:

    *   [aht20temp(4)](//man.NetBSD.org/NetBSD-10.0/aht20temp.4) - a driver for Aosong AHT20 temperature and humidity sensors.
    *   [eqos(4)](//man.NetBSD.org/NetBSD-10.0/eqos.4) - a driver for DesignWare Ethernet Quality-of-Service controllers.
    *   genet - a driver for Broadcom GENETv5 Ethernet controllers, found on the Raspberry Pi 4.
    *   [ixl(4)](//man.NetBSD.org/NetBSD-10.0/ixl.4) - a driver for Intel Ethernet 700 series 10/25/40 Gigabit Ethernet adapters.
    *   [iavf(4)](//man.NetBSD.org/NetBSD-10.0/iavf.4) - a driver for Intel Ethernet Adaptive Virtual Functions.
    *   [mcommphy(4)](//man.NetBSD.org/NetBSD-10.0/mcommphy.4) - a driver for Motorcomm YT8511C / YT8511H Gigabit Ethernet transceivers.
    *   [mos(4)](//man.NetBSD.org/NetBSD-10.0/mos.4) - a driver for MosChip MCS7730/7830/7832 USB Ethernet devices.
    *   [nct(4)](//man.NetBSD.org/NetBSD-10.0/nct.4) - a driver for Nuvoton NCT5104D GPIO controllers, found on PC Engines APU systems.
    *   pcf8574 - a GPIO driver used for LEDs and indicators on some SPARC64 hardware.
    *   [qat(4)](//man.NetBSD.org/NetBSD-10.0/qat.4) - a driver for Intel QuickAssist cryptographic accelerators.
    *   [rge(4)](//man.NetBSD.org/NetBSD-10.0/rge.4) - a driver for Realtek 8125 2.5 Gigabit Ethernet adapters.
    *   [scmd(4)](//man.NetBSD.org/NetBSD-10.0/scmd.4) - a driver for Sparkfun Serial Controlled Motors.
    *   [sgp40mox(4)](//man.NetBSD.org/NetBSD-10.0/sgp40mox.4) - a driver for Sensirion SGP40 MOx gas sensors.
    *   [sht3xtemp(4)](//man.NetBSD.org/NetBSD-10.0/sht3xtemp.4) - a driver for Sensirion SHT30/SHT31/SHT35 humidity/temperature sensors.
    *   [sht4xtemp(4)](//man.NetBSD.org/NetBSD-10.0/sht4xtemp.4) - a driver for Sensirion SHT40/SHT41/SHT45 humidity/temperature sensors.
    *   [uintuos(4)](//man.NetBSD.org/NetBSD-10.0/uintuos.4) - a driver for Wacom Intuos drawing tablet pens.
    *   [wwanc(4)](//man.NetBSD.org/NetBSD-10.0/wwanc.4) - a driver for Intel XMM7360 LTE modems.

*   Improved drivers:

    *   Synced the GPU drivers in the kernel with Linux 5.6, bringing lots of new hardware support for accelerated graphics, for Intel (via *i915*), Nvidia (via *nouveau*), and AMD (via *amdgpu* and *radeon*) graphics processors.
    *   [acpi(4)](//man.NetBSD.org/NetBSD-10.0/acpi.4) - added `/dev/acpi`. a character device for accessing ACPI tables. [acpidump(8)](//man.NetBSD.org/NetBSD-10.0/acpidump.8) no longer requires `options INSECURE`.
    *   amdsmn, amdzentemp - added support for AMD Family 17h/Axh, 17h/6xh, 19h/6xh, 19h/7xh.
    *   [aq(4)](//man.NetBSD.org/NetBSD-10.0/aq.4) - added hardware TCP/UDP RX checksum offloading for Aquantia 2.5/5/10 Gigabit Ethernet interfaces, and support for the Marvell AQC113 10G adapter.
    *   [bge(4)](//man.NetBSD.org/NetBSD-10.0/bge.4) - removed requirement of big kernel lock (support for `NET_MPSAFE` kernel option for Broadcom 10/100/Gigabit Ethernet interfaces).
    *   [ciss(4)](//man.NetBSD.org/NetBSD-10.0/ciss.4) - added support for PERFORMANT mode and MSI/MSI-X on supported HP Smart Array RAID controllers.
    *   [ichsmb(4)](//man.NetBSD.org/NetBSD-10.0/ichsmb.4) - added support for Intel 700 series, Alder Lake-N, and Snow Ridge devices.
    *   [itesio(4)](//man.NetBSD.org/NetBSD-10.0/itesio.4) - added support for IT8625E sensors.
    *   [ixv(4)](//man.NetBSD.org/NetBSD-10.0/ixv.4) - added support for mailbox API version 1.5, used on vmware ESXi.
    *   [mcx(4)](//man.NetBSD.org/NetBSD-10.0/mcx.4) - added hardware checksum offloading, hardware VLAN tagging, and support for multiple receive queues for Mellanox ConnectX multi-Gigabit Ethernet interfaces.
    *   [onewire(4)](//man.NetBSD.org/NetBSD-10.0/onewire.4), [owtemp(4)](//man.NetBSD.org/NetBSD-10.0/owtemp.4) - reduced CPU overhead, improved reliability.
    *   [pci(4)](//man.NetBSD.org/NetBSD-10.0/pci.4) - added support for Enhanced Allocations, as seen in Cavium ThunderX based boards.
    *   [pci(4)](//man.NetBSD.org/NetBSD-10.0/pci.4) - added more PCIe 5.x decoding support.
    *   [pms(4)](//man.NetBSD.org/NetBSD-10.0/pms.4) - many improvements to Synaptics trackpad support.
    *   [tty(4)](//man.NetBSD.org/NetBSD-10.0/tty.4) - removed use of the "big kernel lock" from the console tty, improving responsiveness on multi-core systems.
    *   [u3g(4)](//man.NetBSD.org/NetBSD-10.0/u3g.4) - added support for ZTE MF112 and D-Link DWM222 3G USB modems.
    *   [udl(4)](//man.NetBSD.org/NetBSD-10.0/udl.4) - improve performance when refreshing mostly static USB displays.
    *   [urndis(4)](//man.NetBSD.org/NetBSD-10.0/urndis.4) - added device quirks for the OnePlus 5T.
    *   [urtwn(4)](//man.NetBSD.org/NetBSD-10.0/urtwn.4) - added support for the TRENDnet TEW-648UBM Micro Wireless USB Adapter.
    *   [wm(4)](//man.NetBSD.org/NetBSD-10.0/wm.4) - added support for Intel Tiger Lake and newer devices (I219V 15-V9 and LM 16-19).
    *   [xhci(4)](//man.NetBSD.org/NetBSD-10.0/xhci.4) - added initial support for isochronous pipes (works with e.g. USB 3.x webcams)

*   Improved support for MIPS:

    *   A bootable image, `octeon.img.gz`. is now provided for Cavium OCTEON MIPS64 boards, such as the Ubiquiti ERLite-3. The `ERLITE` kernel configuration was renamed to `OCTEON`.
    *   Added support for [ofctl(8)](//man.NetBSD.org/NetBSD-10.0/ofctl.8) and `/dev/openfirm`. enabled on Cavium Octeon cores.
    *   Added flattened device tree, USB 3, CPU core support on Cavium Octeon.
    *   Added support for kernel modules.
    *   Ported [dtrace(1)](//man.NetBSD.org/NetBSD-10.0/dtrace.1) and the [crash(8)](//man.NetBSD.org/NetBSD-10.0/crash.8) kernel debugger.
    *   Increased the maximum text size for binaries from 64MB to 128MB for O32.

*   Improved support for vintage hardware:

    *   alpha: Many performance and MP stability improvements. Enabled multiprocessor support by default in `GENERIC` kernels.
    *   amiga: Support for Kickstart 3.2 (the release from 2020).
    *   amiga: `loadbsd` bootloader now loads the kernel into the highest priority memory segment instead of the largest segment.
    *   atari: Added box drawing character support to the `ite(4)` framebuffer driver.
    *   evbppc: Added support for the DHT Walnut 405GP board.
    *   evbppc: Added support for the Nintendo Wii.
    *   hp300: Implemented bitmap operations support for the EVRX framebuffer on the HP9000/425e.
    *   hp300: Added support for multiple `rd(4)` disks on all `punits` for HPDisk.
    *   hppa: Enabled support for kernel modules in `GENERIC`.
    *   luna68k: Added support for keyboard LED and buzzer controls via [wskbd(4)](//man.NetBSD.org/NetBSD-10.0/wskbd.4).
    *   luna68k: Added `psgpam(4)` driver for the YM2149 PSG/SSG sound chip.
    *   luna68k: Improved framebuffer graphics and text console performance.
    *   macppc: Improved iMac G5 compatibility: added support for the GeForce framebuffer, and CPU temperature and fan sensors.
    *   mac68k: Added support for synchronous transfer to the esp(4) SCSI driver on Quadra/Centris AV models.
    *   next68k: Many fixes to get the port working again.
    *   sparc: [wsdisplay(4)](//man.NetBSD.org/NetBSD-10.0/wsdisplay.4) performance improvements.
    *   sparc64: Added environment monitoring for the Sun Enterprise 250.
    *   x68k: Added Emulate3Buttons support to the monolithic X server.
    *   x68k: Added box drawing character support to the `ite(4)` framebuffer driver.
    *   vax: Ported the `gpx(4)` and `smg(4)` framebuffer drivers for the VAXstation 3100 from old versions of OpenBSD.
    *   vax: Support booting on machines with as little as 8MB and as high as 512MB RAM.

### Virtualization improvements

*   Many improvements to Xen support:

    *   Added support for Xen PVH.
    *   Added support for Xen PV drivers under HVM guests.
    *   Added support for jumbo frames and feature-sg to paravirtualized network interfaces.
    *   Dom0 kernels now have multiprocessor support enabled.
    *   Xen kernels now use the same kernel modules as native kernels.
    *   Paravirtualized network devices ([xennet(4)](//man.NetBSD.org/NetBSD-10.0/xennet.4)), block devices ([xbd(4)](//man.NetBSD.org/NetBSD-10.0/xbd.4)) are now `MPSAFE` and can take advantage of kernel paralellization.

*   Many improvements to HyperV support:

    *   Added support for multichannel in vmbus and [hvn(4)](//man.NetBSD.org/NetBSD-10.0/hvn.4).
    *   Added support for changing MTU and TX aggregation in [hvn(4)](//man.NetBSD.org/NetBSD-10.0/hvn.4).
    *   Improved VLAN and IP checksum offloading support.

*   VirtIO driver enhancements:

    *   Added support for VirtIO 1.0 to the [virtio(4)](//man.NetBSD.org/NetBSD-10.0/virtio.4) drivers, which previously supported version 0.9.
    *   New [vio9p(4)](//man.NetBSD.org/NetBSD-10.0/vio9p.4) driver allows mounting VirtIO 9P filesystems exported by the VM host.
    *   New [viocon(4)](//man.NetBSD.org/NetBSD-10.0/viocon.4) serial driver.

*   NetBSD Virtual Machine Monitor ([nvmm(4)](//man.NetBSD.org/NetBSD-10.0/nvmm.4)) improvements:

    *   Allow the host to suspend and resume while a virtual machine is running.
    *   Added support for REP CMPS x86 instructions.

*   Added support for QEMU's virtual "mipssim" machine to NetBSD/evbmips, including extensions for [virtio(4)](//man.NetBSD.org/NetBSD-10.0/virtio.4).
*   Added support for running NetBSD/alpha in QEMU.
*   Added support for VMware ESXi-Arm and Oracle Cloud to NetBSD/aarch64.

### Features and general improvements

*   Networking stack improvements:

    *   Implemented RFC 7048 in the kernel's network stack, relaxing rules for IPv6 Neighbor Discovery retransmissions. IPv6 Neighbor Detection is now address agnostic and is used by ARP.
    *   [ipsec(4)](//man.NetBSD.org/NetBSD-10.0/ipsec.4) - added net.key.allow_different_idtype [sysctl(7)](//man.NetBSD.org/NetBSD-10.0/sysctl.7) option to improve interconnectivity with some VPN appliances.
    *   [lagg(4)](//man.NetBSD.org/NetBSD-10.0/lagg.4) - new scalable link aggregation and link failover interface, replaces [agr(4)](//man.NetBSD.org/NetBSD-10.0/agr.4).
    *   [vether(4)](//man.NetBSD.org/NetBSD-10.0/vether.4) - new virtual Ethernet interface with configurable address for use as a [bridge(4)](//man.NetBSD.org/NetBSD-10.0/bridge.4) endpoint, replaces [tap(4)](//man.NetBSD.org/NetBSD-10.0/tap.4) in some scenarios.

*   File system and storage improvements:

    *   Added support for POSIX.1e access control lists to FFS via extended attributes (from FreeBSD). A new file system type, FFSv2ea has been introduced for this purpose. Users can use [fsck_ffs(8)](//man.NetBSD.org/NetBSD-10.0/fsck_ffs.8) to upgrade a file system to support extended attributes. For compatibility with previous releases, FFSv2ea is not yet the default.
    *   Various UDF changes to enable bug-compatibility with UDF file systems on Windows 10.
    *   [fstat(1)](//man.NetBSD.org/NetBSD-10.0/fstat.1) - added basic ZFS support.
    *   [refuse(3)](//man.NetBSD.org/NetBSD-10.0/refuse.3) - now supports all FUSE API variants from FUSE 1.1 to FUSE 3.10.
    *   [raid(4)](//man.NetBSD.org/NetBSD-10.0/raid.4) - added support for swapped-endian configurations.
    *   [blkdiscard(8)](//man.NetBSD.org/NetBSD-10.0/blkdiscard.8) - new front end for [fdiscard(2)](//man.NetBSD.org/NetBSD-10.0/fdiscard.2) to manually TRIM a disk.
    *   [raidctl(8)](//man.NetBSD.org/NetBSD-10.0/raidctl.8) - added `-t` option to test the validity of config files.
    *   [newfs_udf(8)](//man.NetBSD.org/NetBSD-10.0/newfs_udf.8) - added support for formatting of UDF 2.50 with a metadata partition.
    *   [scan_ffs(8)](//man.NetBSD.org/NetBSD-10.0/scan_ffs.8) - added `SIGINFO` support, to display the status of the scan when Ctrl+T is pressed.

*   New userspace programs:

    *   [aiomixer(1)](//man.NetBSD.org/NetBSD-10.0/aiomixer.1) - [curses(3)](//man.NetBSD.org/NetBSD-10.0/curses.3)-based console audio mixer.
    *   [realpath(1)](//man.NetBSD.org/NetBSD-10.0/realpath.1) - prints absolute paths from relative paths, including resolving symbolic links.
    *   [tradcpp(1)](//man.NetBSD.org/NetBSD-10.0/tradcpp.1) - K&R style C macro processor, for programs in base that use C preprocessor macros in their configuration files but should still work without a C compiler installed.
    *   [ioctlprint(1)](//man.NetBSD.org/NetBSD-10.0/ioctlprint.1) - prints descriptive `ioctl` values.
    *   [testpat(6)](//man.NetBSD.org/NetBSD-10.0/testpat.6) - display a color test pattern.
    *   [warp(6)](//man.NetBSD.org/NetBSD-10.0/warp.6) - classic BSD space war game (copyright donated to the NetBSD Foundation by Larry Wall).
    *   [fsck_udf(8)](//man.NetBSD.org/NetBSD-10.0/fsck_udf.8) - new command for repairing damage to Universal Disk Format file systems, making UDF a suitable reliable read-write choice for cross-system shared disks.

*   Improvements to userspace programs:

    *   [audioplay(1)](//man.NetBSD.org/NetBSD-10.0/audioplay.1) - added ability to decode 64-bit and 32-bit IEEE floating point RIFF WAVE files.
    *   [date(1)](//man.NetBSD.org/NetBSD-10.0/date.1) - added `-f` option to set the time.
    *   [date(1)](//man.NetBSD.org/NetBSD-10.0/date.1) - added `-R` option for displaying time in RFC 5322 format, similar to GNU date.
    *   [df(1)](//man.NetBSD.org/NetBSD-10.0/df.1) - added `-b` (output unit: blocks; 512), `-H` (`-h` using SI units), `-N` (suppress the header line), and `-f` (show only free space) options.
    *   [env(1)](//man.NetBSD.org/NetBSD-10.0/env.1) - added `-u` flag to remove an environment variable, and `-0` to allow variable input separated by NUL characters.
    *   [ftp(1)](//man.NetBSD.org/NetBSD-10.0/ftp.1) - added SSL/TLS certificate verification.
    *   [ftp(1)](//man.NetBSD.org/NetBSD-10.0/ftp.1) - follow redirects to relative HTTP URLs.
    *   [ftp(1)](//man.NetBSD.org/NetBSD-10.0/ftp.1) - add timeout for SSL connection setup, using `-q QUITTIME`. defaulting to 60 seconds.
    *   [grep(1)](//man.NetBSD.org/NetBSD-10.0/grep.1) - with `-r` and no file argument, search the current directory.
    *   [indent(1)](//man.NetBSD.org/NetBSD-10.0/indent.1) - support for newer standard C syntax.
    *   [ldd(1)](//man.NetBSD.org/NetBSD-10.0/ldd.1) - added a `-v` option to increase verbosity and show all executable processing errors.
    *   [make(1)](//man.NetBSD.org/NetBSD-10.0/make.1) - added a `randomize-targets` option to aid in debugging
    *   [make(1)](//man.NetBSD.org/NetBSD-10.0/make.1) - added a `.break` keyword to terminate `.for` loops early.
    *   [mv(1)](//man.NetBSD.org/NetBSD-10.0/mv.1) - added a '-h' option to atomically replace a symlink to a directory.
    *   [netstat(1)](//man.NetBSD.org/NetBSD-10.0/netstat.1) - added various new packet counters.
    *   [nbperf(1)](//man.NetBSD.org/NetBSD-10.0/nbperf.1) - various optimizations; reduced memory footprint by 30%.
    *   [patch(1)](//man.NetBSD.org/NetBSD-10.0/patch.1) - added support for patching files with excessively long lines.
    *   [pgrep(1)](//man.NetBSD.org/NetBSD-10.0/pgrep.1) - added `-q` flag to silence output, like in [grep(1)](//man.NetBSD.org/NetBSD-10.0/grep.1).
    *   [pmap(1)](//man.NetBSD.org/NetBSD-10.0/pmap.1) - added `-t` flag to print the pmap as the underlying RB tree.
    *   [ps(1)](//man.NetBSD.org/NetBSD-10.0/ps.1) - added `-G` flag to take a single group argument, as required by POSIX.2.
    *   [rcp(1)](//man.NetBSD.org/NetBSD-10.0/rcp.1) - added `SIGINFO` (ctrl+t status) support.
    *   [sh(1)](//man.NetBSD.org/NetBSD-10.0/sh.1), [ksh(1)](//man.NetBSD.org/NetBSD-10.0/ksh.1), [csh(1)](//man.NetBSD.org/NetBSD-10.0/csh.1) - added `jobs -Z` to set the process title, as in zsh.
    *   [sh(1)](//man.NetBSD.org/NetBSD-10.0/sh.1) - added command auto-completion.
    *   [sh(1)](//man.NetBSD.org/NetBSD-10.0/sh.1) - added `-l` option to force the creation of a login shell.
    *   [sh(1)](//man.NetBSD.org/NetBSD-10.0/sh.1) - added `-d ''` option to the `read` command to accompany `find -print0` and `xargs -0` for compatibility with upcoming POSIX releases.
    *   [script(1)](//man.NetBSD.org/NetBSD-10.0/script.1) - added proper playback of [curses(3)](//man.NetBSD.org/NetBSD-10.0/curses.3) sessions.
    *   [vacation(1)](//man.NetBSD.org/NetBSD-10.0/vacation.1) - check 'Auto-Submitted:' (RFC 3834) mail header in addition to 'Precedence:' (RFC 2076), and set 'Precedence:' in addition to 'Auto-Submitted:'.
    *   [vmstat(1)](//man.NetBSD.org/NetBSD-10.0/vmstat.1) - added fast [sysctl(7)](//man.NetBSD.org/NetBSD-10.0/sysctl.7)-based kernel hash statistics generation
    *   [worms(6)](//man.NetBSD.org/NetBSD-10.0/worms.6) - added more types of worms, `-C` to use colour, `-S` option to set the random number generator seed,
    *   [cgdconfig(8)](//man.NetBSD.org/NetBSD-10.0/cgdconfig.8) - added `-T` option to print all generated keys.
    *   [crash(8)](//man.NetBSD.org/NetBSD-10.0/crash.8) - added PowerPC and MIPS support.
    *   [httpd(8)](//man.NetBSD.org/NetBSD-10.0/httpd.8) - added [blocklistd(8)](//man.NetBSD.org/NetBSD-10.0/blocklistd.8) support.
    *   [httpd(8)](//man.NetBSD.org/NetBSD-10.0/httpd.8) - added a `-q` flag to disable log messages.
    *   [inetd(8)](//man.NetBSD.org/NetBSD-10.0/inetd.8) - added a `-f` flag to run in the foreground.
    *   [iostat(8)](//man.NetBSD.org/NetBSD-10.0/iostat.8) - added a `-z` flag to suppress output of inactive devices.
    *   [sysinst(8)](//man.NetBSD.org/NetBSD-10.0/sysinst.8) - added support for configuring Wi-Fi devices.
    *   [sysinst(8)](//man.NetBSD.org/NetBSD-10.0/sysinst.8) - automatically configure lookups of `.local` domains in [nsswitch.conf(5)](//man.NetBSD.org/NetBSD-10.0/nsswitch.conf.5) when Multicast DNS is enabled.
    *   [tprof(8)](//man.NetBSD.org/NetBSD-10.0/tprof.8) - added `top` subcommand to display hardware profiling results in real-time.
    *   [tprof(8)](//man.NetBSD.org/NetBSD-10.0/tprof.8) - added `count` subcommand to output raw performance event counters.
    *   [tprof(8)](//man.NetBSD.org/NetBSD-10.0/tprof.8) - added support for profiling AMD family 19h (Zen 3 and Zen 4), Intel Comet Lake.
    *   [wsfontload(8)](//man.NetBSD.org/NetBSD-10.0/wsfontload.8) - added a `-l` flag to list all loaded and built-in fonts.
    *   [wsfontload(8)](//man.NetBSD.org/NetBSD-10.0/wsfontload.8) - support for a new font format with embedded metadata.
    *   [wsmoused(8)](//man.NetBSD.org/NetBSD-10.0/wsmoused.8) - added support for absolute mouse position events, e.g. touchscreens.

*   New and extended APIs:

    *   [eventfd(2)](//man.NetBSD.org/NetBSD-10.0/eventfd.2), [timerfd(2)](//man.NetBSD.org/NetBSD-10.0/timerfd.2) - new native system calls compatible with Linux, also used in [compat_linux(8)](//man.NetBSD.org/NetBSD-10.0/compat_linux.8)
    *   [fexecve(2)](//man.NetBSD.org/NetBSD-10.0/fexecve.2) - new system call for executing a file from a file descriptor, conforming to The Open Group Extended API Set 2.
    *   [kqueue(2)](//man.NetBSD.org/NetBSD-10.0/kqueue.2) - added `EVFILT_USER` for user-established events.
    *   [ppoll(2)](//man.NetBSD.org/NetBSD-10.0/ppoll.2) - an alias of the native system call `pollts` for compatibility with other operating systems.
    *   [curses(3)](//man.NetBSD.org/NetBSD-10.0/curses.3) - added stub mouse functions and `curses_version()` for compatibility with ncurses.
    *   [curses(3)](//man.NetBSD.org/NetBSD-10.0/curses.3) - improved Unicode support.
    *   [fetch(3)](//man.NetBSD.org/NetBSD-10.0/fetch.3) - enable Server Name Indication for TLS connections.
    *   [getentropy(3)](//man.NetBSD.org/NetBSD-10.0/getentropy.3) - new libc function for getting random data from the kernel similarly to `KERN_ARND`. compatible with various other OSes.
    *   [math(3)](//man.NetBSD.org/NetBSD-10.0/math.3) - added definitions for more long double functions.
    *   [ossaudio(3)](//man.NetBSD.org/NetBSD-10.0/ossaudio.3) - added an implementation of the OSSv4 mixer API.
    *   [hosts_access(3)](//man.NetBSD.org/NetBSD-10.0/hosts_access.3) - added [blocklistd(8)](//man.NetBSD.org/NetBSD-10.0/blocklistd.8) support, enabling all programs using libwrap to block access from denied hosts.
    *   [regex(3)](//man.NetBSD.org/NetBSD-10.0/regex.3) - added native language support, and support for GNU extensions (off by default; from FreeBSD).

*   Miscellaneous improvements:

    *   Added BSD-licensed Spleen bitmap fonts for low and high-DPI displays to the X11 sets and `/usr/share/wscons/fonts`. made them the default for [ctwm(1)](//man.NetBSD.org/NetBSD-10.0/ctwm.1).
    *   Added Terminus console fonts to `/usr/share/wscons/fonts`.
    *   [wskbd(4)](//man.NetBSD.org/NetBSD-10.0/wskbd.4) - added definitions for French BPO and German Neo 2 layouts.
    *   [wsmouse(4)](//man.NetBSD.org/NetBSD-10.0/wsmouse.4) - added "precision scrolling" event types compatible with OpenBSD and use them in touchpad drivers and [Xorg(1)](//man.NetBSD.org/NetBSD-10.0/Xorg.1).
    *   [compat_linux(8)](//man.NetBSD.org/NetBSD-10.0/compat_linux.8) - added `eventfd`. `timerfd`. POSIX timers, `preadv`. and `pwritev`.

### Changes to system behaviour and compatibility

*   Networking setups using [tap(4)](//man.NetBSD.org/NetBSD-10.0/tap.4) as a [bridge(4)](//man.NetBSD.org/NetBSD-10.0/bridge.4) endpoint must be updated to use [vether(4)](//man.NetBSD.org/NetBSD-10.0/vether.4) instead, as [tap(4)](//man.NetBSD.org/NetBSD-10.0/tap.4)'s link state is now based on whether it has been opened by an application.
*   For security reasons, [compat_linux(8)](//man.NetBSD.org/NetBSD-10.0/compat_linux.8) is now disabled by default. To load it at boot time, add `compat_linux` to `/etc/modules.conf`.
*   The default package database for new installations was changed to `/usr/pkg/pkgdb` for consistency with other pkgsrc platforms, replacing `/var/db/pkg`.
*   Many systems that would have been fine previously may now print warnings about **not enough entropy** to the kernel message buffer. This is because now only vetted hardware sources count towards the kernel's entropy estimation. See [entropy(7)](//man.NetBSD.org/NetBSD-10.0/entropy.7).
*   IEEE 802.11 (Wi-Fi) devices now require SSID configuration in order to associate with an open access point.
*   `blacklistd(8)`, a daemon that can block and release ports on demand to avoid DoS abuse, was renamed to [blocklistd(8)](//man.NetBSD.org/NetBSD-10.0/blocklistd.8).
*   Changed the default shell of the `toor` user to `/rescue/sh` to ensure that a user with a statically linked shell exists on the default install, in case of trouble.
*   Xorg now determines the default keyboard layout based on wscons configuration instead of `/etc/xorg.conf`. To override the default, use [setxkbmap(1)](//man.NetBSD.org/NetBSD-10.0/setxkbmap.1).
*   Disabled automatic unloading of kernel modules - kernel modules must now opt-in to automatic unloading.
*   Combined the midi and sequencer kernel modules into a single `midi_seq` module.
*   arm: ROCKPro64 [ld(4)](//man.NetBSD.org/NetBSD-10.0/ld.4) disk device ordering was changed due to the addition of sdio devices.
*   arm: Switched to XZ-compressed sets for AArch64 (please update your scripts accordingly).
*   x86: HDMI and DisplayPort audio were enabled in the `GENERIC` kernel config. Users may have to change the default audio device with [audiocfg(1)](//man.NetBSD.org/NetBSD-10.0/audiocfg.1) if they want to continue using non-HDMI/DP audio.
*   [crunchgen(1)](//man.NetBSD.org/NetBSD-10.0/crunchgen.1) - various special variable handling flags were removed and replaced with `-V`.
*   [pam(8)](//man.NetBSD.org/NetBSD-10.0/pam.8) - [pam_krb5(8)](//man.NetBSD.org/NetBSD-10.0/pam_krb5.8) and [pam_ksu(8)](//man.NetBSD.org/NetBSD-10.0/pam_ksu.8) were disabled by default in `/etc/pam.d`.
*   [kqueue(2)](//man.NetBSD.org/NetBSD-10.0/kqueue.2) - the `udata` type was changed from `intptr_t` to `void *` for compatibility with other BSDs.
*   [curses(3)](//man.NetBSD.org/NetBSD-10.0/curses.3) - changed the default colour pair to 0 in line with other curses implementations.
*   [proplib(3)](//man.NetBSD.org/NetBSD-10.0/proplib.3) - various API changes and additions. older APIs that have been replaced now produce deprecation warnings.
*   [iconv(3)](//man.NetBSD.org/NetBSD-10.0/iconv.3) - the input argument was changed to be non-const to match current POSIX, previously being const for compatibility with other standards (e.g. SUSv2).
*   [resolver(3)](//man.NetBSD.org/NetBSD-10.0/resolver.3) - the default was changed to check-names (see [resolv.conf(5)](//man.NetBSD.org/NetBSD-10.0/resolv.conf.5)), which means that hostnames that contain invalid characters will not resolve.

### Removed obsolete components

Many obsolete components were removed with the aim of making the network stack and kernel more maintainable, and to make future system-wide improvements (e.g. improved SMP) easier. In some cases removed drivers couldn't be tested due to lack of available hardware and interest, or contained serious long-term bugs.

*   Drivers and support for networking technologies largely replaced by Ethernet: HIPPI, FDDI, and TokenRing.
*   Drivers and support for some old evbarm boards (those that required a custom kernel), as part of the move to support every evbarm device with one `GENERIC` kernel. Hardware dropped includes TI OMAP devices other than the OMAP3530 and AM335x, such as the Gumstix and Hawkboard.
*   In-kernel SMBFS - `nsmb(4)` and `mount_smbfs(8)`. This did not support modern versions of the SMB protocol, and userspace implementations are more functional.
*   In-kernel IPv6 Router Advertisment handling - now handled in userspace by [dhcpcd(8)](//man.NetBSD.org/NetBSD-10.0/dhcpcd.8).
*   `azalia(4)` - a driver which was replaced by [hdaudio(4)](//man.NetBSD.org/NetBSD-10.0/hdaudio.4) in past releases.
*   `de(4)` - a driver which was replaced by [tlp(4)](//man.NetBSD.org/NetBSD-10.0/tlp.4) in past releases.
*   `strip(4)` - a driver for Metricom Ricochet packet radios.
*   `urio(4)` - a driver for Diamond Multimedia Rio500 MP3 players.
*   `uscanner(4)` - a driver for very old USB scanners, use [ugen(4)](//man.NetBSD.org/NetBSD-10.0/ugen.4) and SANE instead.
*   `uyap(4)` - a driver for USB YAP phone firmware loaders.
*   `uyurex(4)` - a driver for a novelty device made by the art group Maywa-denki in 2008.
*   `sup(1)` - a client for the CMU Software Upgrade Protocol. It is available in pkgsrc.
*   Support for ISD's non-standard ATA protocol in [umass(4)](//man.NetBSD.org/NetBSD-10.0/umass.4), used for accessing storage in early Archos MP3 players.
*   `CIRCLEQ` from [queue(3)](//man.NetBSD.org/NetBSD-10.0/queue.3), it was deprecated since NetBSD 7 due to pointer aliasing violations.
*   Several libraries from the X11 distribution: `libXTrap`, `libXevie`, and `libglut` (while GLUT is still useful with modern X servers, libglut users are recommended to switch to FreeGLUT, which is available in pkgsrc). If necessary, removed libraries can continue to be used by installing `compat90` from pkgsrc.

Various third-party components included in the NetBSD base system were updated:

The complete list of changes can be found in the [CHANGES](https://cdn.NetBSD.org/pub/NetBSD/NetBSD-10.0/CHANGES) and [CHANGES-10.0](https://cdn.NetBSD.org/pub/NetBSD/NetBSD-10.0/CHANGES-10.0) files in the top level directory of the NetBSD 10.0 release tree.

Complete source and binaries for NetBSD 10.0 are available for download at many sites around the world. You can download NetBSD 10.0 from our [main CDN](https://cdn.NetBSD.org/pub/NetBSD/NetBSD-10.0/), or use a [mirror site](https://www.NetBSD.org/mirrors/) close to you. A list of hashes, signed by the [NetBSD Security Officer's PGP key](//cdn.NetBSD.org/pub/NetBSD/security/PGP/security-officer@netbsd.org.asc), is available for the NetBSD 10.0 distribution in [this file](https://cdn.NetBSD.org/pub/NetBSD/security/hashes/NetBSD-10.0_hashes.asc).

NetBSD is free. All of the code is under non-restrictive licenses, and may be used without paying royalties to anyone. Free support services are available via our mailing lists and website. Commercial support is available from a variety of sources. More extensive information on NetBSD is available from our website:

## System families supported by NetBSD 10.0

The NetBSD 10.0 release provides supported binary distributions for the following systems:

Ports included in the release but not fully supported or functional:

NetBSD 10.0 is dedicated to the memory of Ryo Shimizu, who passed before it could be released.

ryo@'s contributions to NetBSD, to our community, to ARM and networking (and indeed, to this release) were beyond immense. We are all deeply saddened at the loss of an excellent technical contributor and good friend.

The NetBSD Foundation would like to thank all those who have contributed code, hardware, documentation, funds, colocation for our servers, web pages and other documentation, release engineering, and other resources over the years. More information on the people who make NetBSD happen is available at:

We would also like to thank the Tasty Lime and the Network Security Lab at Columbia University's Computer Science Department for current colocation services. Thanks to [Fastly](https://www.fastly.com/) for providing the CDN services.

NetBSD is a free, fast, secure, and highly portable Unix-like Open Source operating system. It is available for a wide range of platforms, from large-scale servers and powerful desktop systems to handheld and embedded devices. Its clean design and advanced features make it excellent for use in both production and research environments, and the source code is freely available under a business-friendly license. NetBSD is developed and supported by a large and vibrant international community. Many applications are readily available through [pkgsrc, the NetBSD Packages Collection](//pkgsrc.org).

## About the NetBSD Foundation

The [NetBSD Foundation](../../foundation/) was chartered in 1995, with the task of overseeing core NetBSD project services, promoting the project within industry and the open source community, and holding intellectual property rights on much of the NetBSD code base. Day-to-day operations of the project are handled by volunteers.

As a non-profit organization with no commercial backing, the NetBSD Foundation depends on donations from its users, and we would like to ask you to consider [making a donation](../../donations/) to the NetBSD Foundation in support of continuing production of our fine operating system. Your generous donation would be particularly welcome to help with ongoing upgrades and maintenance, as well as with operating expenses for the NetBSD Foundation.

Donations can be done via PayPal to `<[paypal@NetBSD.org](mailto:paypal@NetBSD.org)>`, or via Google Checkout and are fully tax-deductible in the US. See [www.NetBSD.org/donations/](//www.NetBSD.org/donations/#how-to-donate) for more information, or contact `<[finance-exec@NetBSD.org](mailto:finance-exec@NetBSD.org)>` directly.

* * *

Back to

*[NetBSD 10.x formal releases](./)*