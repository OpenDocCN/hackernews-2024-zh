<!--yml
category: 未分类
date: 2024-05-29 13:24:42
-->

# programming - Does this 8088 code in the Leisure Suit Larry 2 game actually do anything? - Retrocomputing Stack Exchange

> 来源：[https://retrocomputing.stackexchange.com/questions/29565/does-this-8088-code-in-the-leisure-suit-larry-2-game-actually-do-anything](https://retrocomputing.stackexchange.com/questions/29565/does-this-8088-code-in-the-leisure-suit-larry-2-game-actually-do-anything)

It's most definitely not a program 'written' by a programmer, but rather a listing of some random program segment using the disassembly command of a [debugger](https://en.wikipedia.org/wiki/Debugger).

Have you tried to find that code within the program?

## Let's See What We Got

Hard to say anything definitive with only this (incomplete) section. A few details visible are:

```
01: INT 21         ; An unknown (*1) call to DOS, function code would have been set prior to this line
02: JB 0103        ; Jump to address 0103h if DOS returns NO error
03: MOV AH,4D      ; Get return code from DOS (*1)
04: INT 21
05: CS:            ; (*2)
06: MOV [OBEA],AX  ; Save return information within the program code at offset 0BEAh
07: JMP 02B2       ; Continue at 02B2h
08: JMP 1451       ; Continue at 1451h (*3)
09: CS:            ; (*4)
10: TEST BYTE PTR [0C59],01 ; Test the lowest bit of the byte at 0C59h within code 
11: JZ 0150        ; Continue at 0150h if not set (*6)
12: CS:
13: TEST BYTE PTR [OC59],02 ; Test the second lowest bit 
14: JZ 014F        ; Continue at 014Fh if not set (*6)
15: IRET           ; Return from Interrupt to terminate the program(*5,*6)
16: INT 21         ; Again a an unknown DOS call (*7)
17: JB 0103        ; Continue at 0103h if no error reported (*8)
18: TEST BYTE PTR [0C59],04 ; Test the third lowest bit
19: JZ 0169        ; Continue at 0169h if not set
20: CMP AH,01      ; check AH for 01h (*7)
21: JA 014F        ; If AH was greater than 01h continue at 014Fh (*9) 
```

*1 - Since the second INT 21h (L:04), made when an error is returned, asks DOS for a [program's return code](https://fd.lod.bz/rbil/interrup/dos_kernel/214d.html), the previous DOS call (L:01) was most likely an [EXEC call](https://fd.lod.bz/rbil/interrup/dos_kernel/214b.html) to run a different program as child process. Using this function also tells that the program was written/compiled for MS-DOS 2.0 or later.

*2 - A hint that the debugger used was most likely the one supplied with MS-DOS.

*3 - Line 08 might be a target from a previous branch, one like the one on L:02\. Such would be a typical compiler structure to continue from a branch to a destination more than 128 bytes away.

*4 - An entry point, like *2, except here operation continues

*5 - Using an IRET here is a strong indicator that the program may be a .COM program and this being used to terminate it.

DOS prepares the memory of a COM program much like CP/M, including a way that a jump at address 0000h will terminate it. This is done by placing it's original [termination interrupt](https://fd.lod.bz/rbil/interrup/dos_kernel/20.html) INT 20h at the start of the PSP - which is at CS:0 for COM programs - and pushing a word of 0000h onto the stack.

Now a simple RET NEAR can be used to terminate. This of course only works if CS doesn't get changed inbetween. A condition true for most simple programs, especially such converted from 8080 code. To still use that way of termination while possibly changing CS during program run it was common for COM programs to start by pushing CS and another word of 0000h onto the stack. Now a RETF could used, no matter what content CS had at the moment (and as long as all stack levels were cleaned up).

EXE programs added a little issue as their initial CS is not pointing to the PSP. But DS is loaded in both types with the PSP address, so pushing DS instead of CS would make the code usable for both types of program.

Since that 'trick' would leave a a single word on stack (the one provided by DOS to start with), it became common to use an IRET instead of a RET FAR, which does take that word as well off the stack. Not that it made a difference, but it felt better :))

*6 - It's extreme hard to do any guess without knowing the address that code is located. But having two jumps targeted only one byte apart means that either some (for x86) unusual program trick is used, or 014Fh containing a single byte instruction. Considering that an IRET/INT combination is rather unusual and that IRET is a single byte instruction, it could quite well be that it's address is 014Fh. But then again the previous JZ wont make much sense, as either way would lead to taking the IRET.

*7 - Checking AH after DOS functions is rare, as the standard return code is contained in AL (AX). Get Return code (see L:03/04) in turn does use AH and AL different. AL contains the child return code, while AH contains the termination reason. Here 00h/01h are regular termination reasons, while 02h is a critical error abort. Checking for above 01h would make sense if the previous INT 21h was as well an EXEC call.

*7 - Note this being he same target as used on line 02, which seems to be the default good exit.

*8 - Note this being he same target as used on line 14

* * *

## Conclusions From Code

The code snipplet is most likely part of an exec function that starts a child program and checks if the execution went well and continues afterwards. It might thus be some menu system/framework starting overlay modules for different tasks (game sections??). It seems to be written in some High level language.

Also, it's most likely not from DOS or any OS level software, but user space, as it used INT 21h functions.

Using the IRET termination hints to some really old code with DOS 1.x origin, or a compiler using that trick to save code size if here are multiple termination points. It must be an older one, as MS discouraged use of returning via PSP:0 and INT 20h in general with DOS 3

Taking into account that the target for 'good' execution (L:02/L:17) points to a very low address within this code, this could be an endless loop starting around the loading and executing a child process. Together with the fact that it checks flags within its own code segment after returning from the child process to decide how to continue, I could imagine this being a loader for a larger program, whose only job is to reload that program whenever it got normally terminated, unless some flags have been set by the child before termination.

A program like this could be a simple solution to catch certain errors without adding lots of error checking code within the application. Any error detection would just exit the child, which then gets restarted from above code.

Note: These are all wild guesses, not necessary true. There is no way to tell where it's originated, unless one finds exactly that code in some program.

## Now, What is It?

Nothing. I'd say they just started up their debugger, loaded some program, dumped a random section of 21 lines and copied them into the game text. Could be from the game or any other program.