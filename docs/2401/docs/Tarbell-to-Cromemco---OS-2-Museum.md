<!--yml
category: 未分类
date: 2024-05-27 15:05:16
-->

# Tarbell to Cromemco | OS/2 Museum

> 来源：[https://www.os2museum.com/wp/tarbell-to-cromemco/](https://www.os2museum.com/wp/tarbell-to-cromemco/)

While playing around with [old versions of 86-DOS](https://www.os2museum.com/wp/86-dos-revisited/), I came across a disk image of [86-DOS 1.14](https://archive.org/details/86-dos-1.14). I ran the older 86-DOS versions in the SIMH simulator which can emulate the Cromemco disk controller supported by 86-DOS.

Unfortunately the 86-DOS 1.14 disk was meant to be used with the different and incompatible Tarbell disk controllers. SIMH doesn’t emulate the Tarbell controller and the disk can’t be booted.

Then it occurred to me that 86-DOS came with all the required source code and it should be possible to adapt the Tarbell disk to use the Cromemco disk controller instead. The two 86-DOS variants (Cromemco and Tarbell) used the exact same 8″ floppy format (77 tracks, 26 sectors of 128 bytes each), just the disk controller was different, and therefore only the boot loader and the I/O system (both within the reserved tracks) were different; the actual files on the FAT file system were identical.

Everything necessary is on the 86-DOS 1.14 disk, all that’s missing is a way to edit and reassemble the source files and put the resulting object code into place on the disk. But that’s actually not too difficult either — the only additional requirement is an 86-DOS disk that boots in SIMH. And fortunately there is a suitable 86-DOS 1.10 disk. With that in hand, it is possible to attach both the (bootable) Cromemco 86-DOS 1.10 disk and the (non-bootable) Tarbell 86-DOS 1.14 disk to SIMH and boot.

 The boot disk will be A: and B: will be the not-yet-bootable Tarbell disk. The tools on the 1.14 disk have no trouble running under 86-DOS 1.10.

#### Prepare SIMH environment

Before any work can start, the compressed IMD image has to be converted to an uncompressed one using the IMDU utility, so that SIMH can write to it. This has already been done and an archive with the required disk images and SIMH can be obtained [here](/files/tarbell-to-cromemco.zip). The SIMH emulator set up to run 86-DOS originally came from [here](https://web.archive.org/web/20080819181906/http://www.86dos.org/).

To use the two-disk setup, run ‘altairz80 tartocro’ on a Windows machine. Keep in mind that to exit SIMH, you can use Ctrl-Break.

Once 86-DOS boots, the first step is editing BOOT.ASM and DOSIO.ASM to support Cromemco rather than Tarbell controllers.

To do that, there is EDLIN, which is… not a very nice editor, but it *does* work. Basic EDLIN documentation can be [found on Bitsavers](http://bitsavers.informatik.uni-stuttgart.de/pdf/seattleComputer/86-DOS_0.3_Users_Manual_1980.pdf#page=13&zoom=auto,-138,657). EDLIN has its own command line; to get at a specific line, enter the line number.

#### Fix BOOT.ASM

In BOOT.ASM, CROMEMCOLARGE must be enabled and TARBELLDOUBLE disabled. To do that, change line 18 from ‘CROMEMCOLARGE: EQU 0’ to ‘CROMEMCOLARGE: EQU 1’ and line 20 from ‘TARBELLDOUBLE: EQU 1’ to ‘TARBELLDOUBLE: EQU 0’.

Note that to copy the template (previous line contents) in EDLIN, the F3 key does not work but it is possible to press Esc and then U (this must be capital U not lowercase u; in other words, first press Esc and then Shift+U).

Write out the modified file and exit EDLIN by entering ‘E’ on the EDLIN command line (in this version, the E can be lowercase). The SIMH output will look about like this:

```
B:edlin boot.asm

EDLIN  version 1.01
End of input file
*18
    18:*CROMEMCOLARGE:  EQU     0
    18:*CROMEMCOLARGE:  EQU     1
*20
    20:*TARBELLDOUBLE:  EQU     1
    20:*TARBELLDOUBLE:  EQU     0
*e

B:
```

#### Fix DOSIO.ASM

Now DOSIO.ASM needs to be similarly modified. TARBELLDD must be disabled, CROMEMCO4FDC enabled, and additionally FASTSEEK must be enabled (otherwise the source file won’t assemble). To do that, change line 26 from ‘TARBELLDD: EQU 1’ to ‘TARBELLDD: EQU 0’, line 27 from ‘CROMEMCO4FDC: EQU 0’ to ‘CROMEMCO4FDC: EQU 1’, and finally change line 47 from ‘FASTSEEK: EQU 0’ to ‘FASTSEEK: EQU 1’.

#### Assemble modified source

Now assemble the modified source files. To avoid creating large listing files which take up too much disk space, it is necessary to use the rather odd syntax of the SCP assembler as follows:

```
B:asm boot.bbz

Seattle Computer Products 8086 Assembler Version 2.40
Copyright 1979,80,81 by Seattle Computer Products, Inc.

Error Count =    0

B:asm dosio.bbz

Seattle Computer Products 8086 Assembler Version 2.40
Copyright 1979,80,81 by Seattle Computer Products, Inc.

Error Count =    0

B:
```

The assembler always assumes .ASM extension for source files. The fake ‘bbz’ extension tells ASM to look for the source .ASM file on drive B:, also use drive B: to output the resulting .HEX file, and to produce no listing file (.PRN).

Now the HEX files need to be converted to binary (.COM) files. That is accomplished by the HEX2BIN utility. Simply run ‘hex2bin boot’ and ‘hex2bin dosio’. There should now be a fresh BOOT.COM and DOSIO.COM.

#### Install modified code

The last step requires writing the boot sector and I/O system to the right place on the disk. The boot sector is the first sector, and the I/O system immediately follows it on the first track. Conveniently, the DEBUG utility can do all the hard work.

To put the boot sector in place, run ‘debug boot.com’, then enter ‘w 100 1 0 1’ and finally exit DEBUG with ‘q’. Like this:

```
B:debug boot.com

DEBUG-86  version 1.03
>w 100 1 0 1
>q

B:
```

The W command has four arguments. The first is the starting address, in this case 100h because .COM files are loaded at address 100h in the program segment. The second argument is the drive, with drive B: being addressed as drive 1 in DEBUG. The third argument is the starting record/sector, and zero is the boot sector. The fourth and last argument is the number of records to copy (one).

Now the process must be repeated for the I/O system. Run ‘debug dosio.com’ and then ‘w 100 1 1 8’, exit DEBUG with ‘q’.

```
B:debug dosio.com

DEBUG-86  version 1.03
>w 100 1 1 8
>q

B:
```

The W command is almost the same, except it writes 8 sectors and starts at logical sector 1 (the sector immediately following the boot sector).

#### Boot it up

If everything went well, the modified 86-DOS 1.14 disk is now bootable in SIMH:

```
X:\tarcro>altairz80.exe 86dos114

Altair 8800 (Z80) simulator V3.8-1
2047 bytes [8 pages] loaded at 0.
2047 bytes [8 pages] loaded at ff800.

Press <return> to get monitor prompt, then type 'B' to boot

SCP 8086 Monitor 1.5
>B
☺
86-DOS version 1.14
Copyright 1980,81 Seattle Computer Products, Inc.

COMMAND v. 1.10
Current date is 00-00-19 80
Enter new date:
```

And there it is! 86-DOS 1.14 running in SIMH on the emulated Cromemco controller. All the necessary tools were on the 1.14 disk itself, the only thing missing was a way to run them. Borrowing an existing SIMH compatible 86-DOS disk did the trick.