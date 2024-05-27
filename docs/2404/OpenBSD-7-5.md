<!--yml
category: 未分类
date: 2024-05-27 12:57:21
-->

# OpenBSD 7.5

> 来源：[https://www.openbsd.org/75.html](https://www.openbsd.org/75.html)

Quick installer information for people familiar with OpenBSD, and the use of the "[disklabel](https://man.openbsd.org/disklabel.8) -E" command. If you are at all confused when installing OpenBSD, read the relevant INSTALL.* file as listed above!

### OpenBSD/alpha:

If your machine can boot from CD, you can write *install75.iso* or *cd75.iso* to a CD and boot from it. Refer to INSTALL.alpha for more details.

### OpenBSD/amd64:

If your machine can boot from CD, you can write *install75.iso* or *cd75.iso* to a CD and boot from it. You may need to adjust your BIOS options first.

If your machine can boot from USB, you can write *install75.img* or *miniroot75.img* to a USB stick and boot from it.

If you can't boot from a CD, floppy disk, or USB, you can install across the network using PXE as described in the included INSTALL.amd64 document.

If you are planning to dual boot OpenBSD with another OS, you will need to read INSTALL.amd64.

### OpenBSD/arm64:

If your machine can boot from CD, you can write *install75.iso* or *cd75.iso* to a CD and boot from it.

To boot from disk, write *install75.img* or *miniroot75.img* to a disk and boot from it after connecting to the serial console. Refer to INSTALL.arm64 for more details.

### OpenBSD/armv7:

Write a system specific miniroot to an SD card and boot from it after connecting to the serial console. Refer to INSTALL.armv7 for more details.

### OpenBSD/hppa:

Boot over the network by following the instructions in INSTALL.hppa or the [hppa platform page](hppa.html#install).

### OpenBSD/i386:

If your machine can boot from CD, you can write *install75.iso* or *cd75.iso* to a CD and boot from it. You may need to adjust your BIOS options first.

If your machine can boot from USB, you can write *install75.img* or *miniroot75.img* to a USB stick and boot from it.

If you can't boot from a CD, floppy disk, or USB, you can install across the network using PXE as described in the included INSTALL.i386 document.

If you are planning on dual booting OpenBSD with another OS, you will need to read INSTALL.i386.

### OpenBSD/landisk:

Write *miniroot75.img* to the start of the CF or disk, and boot normally.

### OpenBSD/loongson:

Write *miniroot75.img* to a USB stick and boot bsd.rd from it or boot bsd.rd via tftp. Refer to the instructions in INSTALL.loongson for more details.

### OpenBSD/luna88k:

Copy 'boot' and 'bsd.rd' to a Mach or UniOS partition, and boot the bootloader from the PROM, and then bsd.rd from the bootloader. Refer to the instructions in INSTALL.luna88k for more details.

### OpenBSD/macppc:

Burn the image from a mirror site to a CDROM, and power on your machine while holding down the *C* key until the display turns on and shows *OpenBSD/macppc boot*.

Alternatively, at the Open Firmware prompt, enter *boot cd:,ofwboot /7.5/macppc/bsd.rd*

### OpenBSD/octeon:

After connecting a serial port, boot bsd.rd over the network via DHCP/tftp. Refer to the instructions in INSTALL.octeon for more details.

### OpenBSD/powerpc64:

To install, write *install75.img* or *miniroot75.img* to a USB stick, plug it into the machine and choose the *OpenBSD install* menu item in Petitboot. Refer to the instructions in INSTALL.powerpc64 for more details.

### OpenBSD/riscv64:

To install, write *install75.img* or *miniroot75.img* to a USB stick, and boot with that drive plugged in. Make sure you also have the microSD card plugged in that shipped with the HiFive Unmatched board. Refer to the instructions in INSTALL.riscv64 for more details.

### OpenBSD/sparc64:

Burn the image from a mirror site to a CDROM, boot from it, and type *boot cdrom*.

If this doesn't work, or if you don't have a CDROM drive, you can write *floppy75.img* or *floppyB75.img* (depending on your machine) to a floppy and boot it with *boot floppy*. Refer to INSTALL.sparc64 for details.

Make sure you use a properly formatted floppy with NO BAD BLOCKS or your install will most likely fail.

You can also write *miniroot75.img* to the swap partition on the disk and boot with *boot disk:b*.

If nothing works, you can boot over the network as described in INSTALL.sparc64.

### Ports Tree

A ports tree archive is also provided. To extract:

> ```
> # cd /usr
> # tar xvfz /tmp/ports.tar.gz
> 
> ```

Go read the [ports](faq/ports/index.html) page if you know nothing about ports at this point. This text is not a manual of how to use ports. Rather, it is a set of notes meant to kickstart the user on the OpenBSD ports system.

The *ports/* directory represents a CVS checkout of our ports. As with our complete source tree, our ports tree is available via [AnonCVS](anoncvs.html). So, in order to keep up to date with the -stable branch, you must make the *ports/* tree available on a read-write medium and update the tree with a command like:

> ```
> # cd /usr/ports
> # cvs -d anoncvs@server.openbsd.org:/cvs update -Pd -rOPENBSD_7_5
> 
> ```

[Of course, you must replace the server name here with a nearby anoncvs server.]

Note that most ports are available as packages on our mirrors. Updated ports for the 7.5 release will be made available if problems arise.

If you're interested in seeing a port added, would like to help out, or just would like to know more, the mailing list [ports@openbsd.org](mail.html) is a good place to know.