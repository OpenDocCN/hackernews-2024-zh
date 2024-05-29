<!--yml
category: 未分类
date: 2024-05-27 14:38:36
-->

# An Easy to Build Real Homemade Computer: Z80-MBC2! : 9 Steps (with Pictures) - Instructables

> 来源：[https://www.instructables.com/An-Easy-to-Build-Real-Homemade-Computer-Z80-MBC2/](https://www.instructables.com/An-Easy-to-Build-Real-Homemade-Computer-Z80-MBC2/)

If you are curious about how a computer works and interacts with "external things", nowadays there are a lot of boards ready to play like Arduino or Raspberry and many others. But this boards have all the same "limit"... they hide the inner part because they use a MCU (Micro Controller Unit) or a SOC (System On Chip) so you can't touch the CPU, I/O, the internal bus and all these stuff that are what makes a computer work.

There is an other option using some older part as 8bit CPUs (the so called "retrocomputing"). They are simple to understand and you can find a lot of documentation and books for free, and allow to build real computers with all the needed function blocks (CPU, I/O, RAM, ROM/EPROM, etc...).

But generally they use hard to find parts, and require outdated instruments like an EPROM programmer and eraser or a GAL programmer, and simpler ones have very limited features.

So I've mixed old and "new" parts to make an unique design that doesn't need any legacy EPROM programmer or fancy ICs, using easy to find components. The Atmega32A MCU acts as an I/O subsystem, "emulating" the EPROM and all the I/O components. More, using an Arduino bootloader, It can be easily programmed with the well known Arduino IDE.

The needed ICs are:

*   Z80 CPU CMOS (Z84C00) 8Mhz or greater
*   Atmega32A
*   TC551001-70 (128KB RAM)
*   74HC00

If you want the 16x GPIO expansion (GPE option) add a MCP23017 too.

The Z80-MBC2 has a multi-boot capability and can run CP/M 2.2, QP/M 2.71 and CP/M 3 (128KB banked memory supported), so you can use a very large amount of SW with it (e.g. you can easily find Basic, C, Assembler, Pascal, Fortran, Cobol compilers, and some of these are already provided in the virtual disks on the SD).

Hard Disks are emulated using a microSD FAT16 or FAT32 formatted (a 1GB microSD is enough), so it is easy exchange the files with your PC (16 HDs for every OS are supported) using **[cpmtoolsGUI](http://star.gmobb.jp/koji/cgi/wiki.cgi?action=ATTACH&page=CpmtoolsGUI&file=CPMTG%5FENG%5F20180903%2Ezip)** .

Of course you need a terminal to interact with the Z80-MBC2, and a common USB-serial adapter together with a terminal emulation SW will be a cheap and simple choice.