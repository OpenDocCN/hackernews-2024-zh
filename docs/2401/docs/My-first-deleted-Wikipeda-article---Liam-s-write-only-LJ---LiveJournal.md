<!--yml
category: 未分类
date: 2024-05-27 14:37:54
-->

# My first deleted Wikipeda article - Liam's write-only LJ — LiveJournal

> 来源：[https://lproven.livejournal.com/189791.html](https://lproven.livejournal.com/189791.html)

This is, so far, the only article of mine that's been completely deleted from Wikipedia. I've salvaged a copy of it from

[Search.com](https://www.livejournal.com/away?to=http%3A%2F%2Fwww.search.com%2Freference%2FNon-PC_compatible_x86_computers)

, since it's gone from the site's history now, and am putting it here until I decide what else to do with it.

**Non-PC compatible x86 computers**

For much of the time that Microsoft operating systems have dominated business computing, it has been a given that all computers based on Intel x86 family CPUs were IBM PC Compatibles. This is not the case and never has been.

**MS-DOS systems**

From the earliest days of the PC market, there have been other MS-DOS systems which were not IBM PC-compatible, such as the Sirius Victor computers and some early models of Apricot Computers. These had 8088 or 8086 processors and ran their own versions of MS-DOS, but the memory layout, display controllers and other hardware were not identical to IBM's components and were located in different areas of memory. The companies had anticipated that by producing technically-superior designs which could run the popular MS-DOS operating system, they would be able to take advantage of the large library of MS-DOS applications software.

Aside from its low hardware specification, perhaps the biggest perceived "design flaw" of the IBM PC was the what is nowadays called Upper Memory Area - the location of the I/O area at the top of the memory map (from 640 KiB up to 1 MiB). Locating the I/O area at the bottom of the memory map instead made it possible and indeed simple to expand these machines' memories. More RAM just appeared at the top of the memory map, up to the 1 MiB addressable limit of the 8086 processor. Thus. machines such as the Victor could routinely take 800-900 KiB of RAM, considerably more than the PC's 640 KiB limit. Other improvements over the IBM PC design were high-density 1.2 MiB floppy drives as standard, integrated sound cards and monochrome graphics (comparable to the IBM PC's Hercules Graphics Card).

However, although this was, in theory, enough to guarantee commercial success, that is not in reality a commercial success because in its early versions, MS-DOS was very basic and provided only meager functionality to applications developers. So rather than using DOS' APIs to control the hardware, most software authors "hit the metal" and controlled the PC hardware by writing directly to peripherals' control registers and so on. This meant that most PC applications could not run on other, non-PC-compatible x86 machines.

This would have become irrelevant with the advent of Windows and especially Windows NT, as it provided extensive APIs to Windows applications and discouraged developers from accessing the hardware registers. However, the first version of Windows to be successful was Windows 3.0 in 1990, too late for the proprietary DOS machines, although some of the last non-PC-compatible Apricot systems did support their own special version of Windows 2.0 and therefore probably ran many of the Windows 2.0 applications.

**Multiuser and network machines**

There were even less PC-like 8086 systems, such as the Jarogate Sprite, a multiuser Concurrent CP/M machine which resembled a minicomputer, supporting only dumb terminals - it had no display controller or keyboard interface on the system unit. Another was the 3Com 3Server and later 3Server386, which were dedicated PC LAN servers, with support for neither displays and keyboards of their own nor even terminals - control, setup and administration was done remotely, over the network. These ran 3Com's DOS-based NOS, 3+Share, which contained its own internal multitasking system. Although the 3Server achieved some commercial success, it had a less successful companion product, a diskless dedicated LAN workstation: the 3Station.

The rise of the PC compatible and its huge software library, including a range of alternative operating systems, killed off these early systems, but later ones were to appear.

**UNIX workstations**

Intel's 80386 processor was the first to include memory management hardware and thus be capable of running a full UNIX operating system. Sun Microsystems exploited this with the release of the Sun386i, a 386-based UNIX workstation. Though based on a processor mainly used in PCs and featuring ISA expansion slots, this was nonetheless a workstation, not a PC. Storage was on SCSI devices, it accepted a Sun Type 4 keyboard and optical mouse and booted via Sun firmware into SunOS 4\. The development codename of these machines was Roadrunner.

Even though is was natively running UNIX, Sun developed this machine with the possibility to break into the PC market in mind. The Sun386i generally included a PC emulator (emulating an 8086) that could run MS-DOS and so normal DOS applications. As it was just a UNIX task more than one instance of the emulator could be used, essentially resulting in a multi-tasking DOS system. performance was pretty good, but the 386i didn't achieve a big market success. Old UNIX customers preferred the Sun-3 line, because most commercial UNIX applications were not available on the Sun386i. New customers mainly interested in MS-DOS had no really use for UNIX and preferred a real PC which ran MS-DOS natively faster and cheaper. Sun dropped the 386i line quickly when their SPARC systems reached the market. A successor workstation based on the 80486 processor (codename Apache) made it to prototype stage and would have been named the Sun486i; under 200 were made.

**Windows NT workstations**

Microsoft's Windows NT operating system was originally intended to be a cross-platform offering and up to NT 4 it ran on systems powered by MIPS, DEC Alpha and later PowerPC processors from a variety of manufacturers. This was a spin-off of the unsuccessful Advanced Computing Environment initiative and its Advanced RISC Computing specification (ARCS).

Reportedly, a development version also ran on SPARC but this was never commercially released.

However, the continuing rise in power of x86 led to support for all non-Intel CPUs being dropped with the advent of Windows 2000, although for some time, Microsoft allegedly maintained the Alpha port internally to ensure that the code remained portable.

Another short-lived generation of dedicated NT workstations arrived in the mid-1990s in the form of the SGI Visual Workstations. Again, these run workstation firmware - in this case, ARCS - and have a totally un-PC-like arrangement of bus, display and so on, although it used an x86 processor. This is the first time the HAL in Windows NT helped to increase the success of the workstations, since most x86 applications can run on it, thanks to the HAL.

**Apple Macintosh**

On the 6th June 2005, at Apple's Worldwide Developer Conference, Apple CEO Steve Jobs announced that future Macintosh computers would be based upon Intel processors instead of the company's previous line of PowerPC-based machines. The first to ship was the iMac, on January 10th, 2006\. The transition is expected to be complete by the end of 2006.

The first development systems shipped to developers consist of a relatively standard Pentium 4 motherboard in a PowerMac G5 case -- complete with a normal PC BIOS ROM.

This had led many observers to conclude that this means that future Intel-based Macintosh systems will be generic PC compatibles, but this is not the case -- final shipping products replaced the traditional BIOS with the newer EFI standard. Apple's new computers are merely the latest in a long line of Intel-based computers which are not PCs.

See the Apple Intel transition article for more information.

**Summary**

A number of factors determine if a machine is a PC compatible or not. These include, but are not restricted to, an IBM-PC-compatible BIOS, a compatible set of I/O ports, an Upper Memory Area containing video ROM and RAM and so on; in general terms, the detailed arrangement of the memory map of the system, including its firmware, its interrupts, its I/O ports and other factors, plus an x86 microprocessor.

These factors do not include its expansion bus. Some machines may have no such bus and some may have completely incompatible ones - such as the IBM Personal System/2 machines. On the other hand, the existence of PC standard slots does not mean that a machine is PC-compatible. Many non-PC architectures feature or have used ISA slots, including the Commodore Amiga, the Sinclair QL-compatible Q40 and Q60 machines and the Peters Plus Sprinter; PCI has been used in PowerPC Apple Macintosh machines and Sun workstations and AGP in more recent Macs.

Although many of these machines have been relatively obscure or unsuccessful, the presence of an x86 processor and a PC-like expansion bus, and even the ability to run Microsoft operating systems, does not mean that a computer is a PC.

**External links**

*

[Sun 386i/250 pictures and description](https://www.livejournal.com/away?to=http%3A%2F%2Fsites.inka.de%2Fpcde%2Fcollection%2Fsun386i_250.html)

*

[Sun 386i/150 technical information](https://www.livejournal.com/away?to=http%3A%2F%2Fwww.machine-room.org%2Fcomputers%2F7297%2F)

*

[Sun 386i/260 technical information](https://www.livejournal.com/away?to=http%3A%2F%2Fwww.machine-room.org%2Fcomputers%2F7298%2F)

*

[Some (inaccurate) information on the Jarogate Sprite](https://www.livejournal.com/away?to=http%3A%2F%2Fwww.computer-archiv.de%2Fcomp0775.htm)

- in German