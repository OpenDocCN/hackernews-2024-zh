<!--yml
category: 未分类
date: 2024-05-27 13:30:27
-->

# QEMU version 9.0.0 released - QEMU

> 来源：[https://www.qemu.org/2024/04/23/qemu-9-0-0/](https://www.qemu.org/2024/04/23/qemu-9-0-0/)

# QEMU version 9.0.0 released

23 Apr 2024

We’d like to announce the availability of the QEMU 9.0.0 release. This release contains 2700+ commits from 220 authors.

You can grab the tarball from our [download page](https://www.qemu.org/download/#source). The full list of changes are available [in the changelog](https://wiki.qemu.org/ChangeLog/9.0).

Highlights include:

*   block: virtio-blk now supports multiqueue where different queues of a single disk can be processed by different I/O threads
*   gdbstub: various improvements such as catching syscalls in user-mode, support for fork-follow modes, and support for siginfo:read
*   memory: preallocation of memory backends can now be handled concurrently using multiple threads in some cases
*   migration: support for “mapped-ram” capability allowing for more efficient VM snapshots, improved support for zero-page detection, and checkpoint-restart support for VFIO
*   ARM: architectural feature support for ECV (Enhanced Counter Virtualization), NV (Nested Virtualization), and NV2 (Enhanced Nested Virtualization)
*   ARM: board support for B-L475E-IOT01A IoT node, mp3-an536 (MPS3 dev board + AN536 firmware), and raspi4b (Raspberry Pi 4 Model B)
*   ARM: additional IO/disk/USB/SPI/ethernet controller and timer support for Freescale i.MX6, Allwinner R40, Banana Pi, npcm7xxx, and virt boards
*   HPPA: numerous bug fixes and SeaBIOS-hppa firmware updated to version 16
*   LoongArch: KVM acceleration support, including LSX/LASX vector extensions
*   RISC-V: ISA/extension support for Zacas, amocas, RVA22 profiles, Zaamo, Zalrsc, Ztso, and more
*   RISC-V: SMBIOS support for RISC-V virt machine, ACPI support for SRAT, SLIT, AIA, PLIC and updated RHCT table support, and numerous fixes
*   s390x: Emulation support for CVDG, CVB, CVBY and CVBG instructions, and fixes for LAE (Load Address Extended) emulation
*   and lots more…

Thank you to everybody who contributed to this release, whether that was by writing code, reporting bugs, improving documentation, testing, or providing the project with CI resources. We couldn’t do these without you!