<!--yml
category: 未分类
date: 2024-05-27 12:58:56
-->

# Programming With DOS Debugger - Susam Pal

> 来源：[https://susam.net/programming-with-dos-debugger.html](https://susam.net/programming-with-dos-debugger.html)

<main>

# Programming With DOS Debugger

By **Susam Pal** on 11 Feb 2003

## Introduction

MS-DOS as well as Windows 98 come with a debugger program named `DEBUG.EXE` that can be used to work with assembly language instructions and machine code. In MS-DOS version 6.22, this program is named `DEBUG.EXE` and it is typically present at `C:\DOS\DEBUG.EXE`. On Windows 98, this program is usually present at `C:\Windows\Command\Debug.exe`. It is a line-oriented debugger that supports various useful features to work with and debug binary executable programs consisting of machine code.

In this post, we see how we can use this debugger program to assemble a few minimal programs that print some characters to standard output. We first create a 7-byte program that prints a single character. Then we create a 23-byte program that prints the "hello, world" string. All the steps provided in this post work well with Windows 98 too.

## Contents

## Print Character

Let us first see how to create a tiny 7-byte program that prints the character `A` to standard output. The following `DEBUG.EXE` session shows how we do it.

```
C:\>`DEBUG`
-`A`
1165:0100 `MOV AH, 2`
1165:0102 `MOV DL, 41`
1165:0104 `INT 21`
1165:0106 `RET`
1165:0107
-`G`
A
Program terminated normally
-`N A.COM`
-`R CX`
CX 0000
:`7`
-`W`
Writing 00007 bytes
-`Q`

C:\>

```

Now we can execute this program as follows:

```
C:\>`A`
A
C:\>

```

The debugger command `A` creates machine executable code from assembly language instructions. The machine code created is written to the main memory at address CS:0100 by default. The first three instructions generate the software interrupt 0x21 (decimal 33) with AH set to 2 and DL set to 0x41 (decimal 65) which happens to be the ASCII code of the character `A`. Interrupt 0x21 offers a wide variety of DOS services. Setting AH to 2 tells this interrupt to invoke the function that prints a single character to standard output. This function expects DL to be set to the ASCII code of the character we want to print.

The command `G` executes the program in memory from the current location. The current location is defined by the current value of CS:IP which is CS:0100 by default. We use this command to confirm that the program runs as expected.

Next we prepare to write the machine code to a binary executable file. The command `N` is used to specify the name of the file. The command `W` is used to write the machine code to the file. This command expects the registers BX and CX to contain the number of bytes to be written to the file. When the DOS debugger starts, BX is already initialised to 0, so we only set the register CX to 7 with the `R CX` command. Finally, we use the command `Q` to quit the debugger and return to MS-DOS.

## Hello, World

The following `DEBUG.EXE` session shows how to create a program that prints a string.

```
C:\>`DEBUG`
-`A`
1165:0100 `MOV AH, 9`
1165:0102 `MOV DX, 108`
1165:0105 `INT 21`
1165:0107 `RET`
1165:0108 `DB 'hello, world', D, A, '/main>`
1165:0117
-`G`
hello, world

Program terminated normally
-`N HELLO.COM`
-`R CX`
CX 0000
:`17`
-`W`
Writing 00017 bytes
-`Q`

C:\>

```

Now we can execute this 23-byte program like this:

```
C:\>`HELLO`
hello, world

C:\>

```

In the program above we use the pseudo-instruction `DB` to define the bytes of the string we want to print. We add the trailing bytes 0xD and 0xA to print the carriage return (CR) and the line feed (LF) characters so that the string is terminated with a newline. Finally, the string is terminated with the byte for dollar sign (`'/main>`) because the software interrupt we generate next expects the string to be terminated with this symbol's byte value.

We use the software interrupt 0x21 again. However, this time we set AH to 9 to invoke the function that prints a string. This function expects DS:DX to point to the address of a string terminated with the byte value of `'/main>`. The register `DS` has the same value as that of `CS`, so we only set `DX` to the offset at which the string begins.

## Debugger Scripting

We have already seen above how to assemble a "hello, world" program in the previous section. We started the debugger program, typed some commands, and typed assembly language instructions to create our program. It is also possible to prepare a separate input file with all the debugger commands and assembly language instructions in it. We then feed this file to the debugger program. This can be useful while writing more complex programs where we cannot afford to lose our assembly language source code if we inadvertently crash the debugger by executing an illegal instruction.

To create a separate input file that can be fed to the debugger, we may use the DOS command `EDIT HELLO.TXT` to open a new file with MS-DOS Editor, then type in the following debugger commands, and then save and exit the editor.

```
A
MOV AH, 9
MOV DX, 108
INT 21
RET
DB 'hello, world', D, A, '/main>

N HELLO.COM
R CX
17
W
Q

```

This is almost the same as the inputs we typed into the debugger in the previous section. The only difference from the previous section is that we omit the `G` command here because we don't really need to run the program while assembling it, although we could do so if we really wanted to.

Then we can run the DOS command `DEBUG < HELLO.TXT` to assemble the program and create the binary executable file. Here is a DOS session example that shows what the output of this command looks like:

```
C:\>`DEBUG < HELLO.TXT`
-A
1165:0100 MOV AH, 9
1165:0102 MOV DX, 108
1165:0105 INT 21
1165:0107 RET
1165:0108 DB 'hello, world', D, A, '/main>
1165:0117
-N HELLO.COM
-R CX
CX 0000
:17
-W
Writing 00017 bytes
-Q

C:\>

```

The output is in fact very similar to the debugger session in the previous section.

## Disassembly

Now that we have seen how to assemble simple programs into binary executable files using the debugger, we will now briefly see how to disassemble the binary executable files. This could be useful when we want to debug an existing program.

```
C:\>`DEBUG A.COM`
-`U 100 106`
117C:0100 B402          MOV     AH,02
117C:0102 B241          MOV     DL,41
117C:0104 CD21          INT     21
117C:0106 C3            RET

```

The debugger command `U` (unassemble) is used to translate the binary machine code to assembly language mnemonics.

```
C:\>`DEBUG HELLO.COM`
-`U 100 116`
117C:0100 B409          MOV     AH,09
117C:0102 BA0801        MOV     DX,0108
117C:0105 CD21          INT     21
117C:0107 C3            RET
117C:0108 68            DB      68
117C:0109 65            DB      65
117C:010A 6C            DB      6C
117C:010B 6C            DB      6C
117C:010C 6F            DB      6F
117C:010D 2C20          SUB     AL,20
117C:010F 776F          JA      0180
117C:0111 726C          JB      017F
117C:0113 64            DB      64
117C:0114 0D0A24        OR      AX,240A
-`D 100 116`
117C:0100  B4 09 BA 08 01 CD 21 C3-68 65 6C 6C 6F 2C 20 77   ......!.hello, w
117C:0110  6F 72 6C 64 0D 0A 24                              orld..$

```

## INT 20 vs RET

Another way to terminate a .COM program is to simply use the instruction `INT 20`. This consumes two bytes in the machine code: `CD 20`. While producing the smallest possible executables was not really the goal of this post, the code examples above indulge in a little bit of size reduction by using the `RET` instruction to terminate the program. This consumes only one byte: `C3`. This works because when a .COM file starts, the register SP contains FFFE. The stack memory locations at offset FFFE and FFFF contain 00 and 00, respectively. Further, the memory address offset 0000 contains the instruction `INT 20`. Here is a demonstration of these facts using the debugger program:

```
C:\>`DEBUG HELLO.COM`
-`R SP`
SP FFFE
:
-`D FFFE`
117C:FFF0                                            00 00
-`U 0 1`
117C:0000 CD20          INT     20

```

As a result, executing the `RET` instruction pops 0000 off the stack at FFFE and loads it into IP. This results in the instruction `INT 20` at offset 0000 getting executed which leads to program termination.

While both `INT 20` and `RET` lead to successful program termination both in DOS as well as while debugging with `DEBUG.EXE`, there is some difference between them which affects the debugging experience. Terminating the program with `INT 20` allows us to run the program repeatedly within the debugger by repeated applications of the `G` debugger command. But when we terminate the program with `RET`, we cannot run the program repeatedly in this manner. The program runs and terminates successfully the first time we run it in the debugger but the stack does not get reinitialised with zeros to prepare it for another execution of the program within the debugger. Therefore when we try to run the program the second time using the `G` command, the program does not terminate successfully. It hangs instead. It is possible to work around this by reinitialising the stack with the debugger command `E FFFE 0 0` before running `G` again.

## Conclusion

Although the DOS debugger is very limited in features in comparison with sophisticated assemblers like NASM, MASM, etc., this humble program can perform some of the basic operations involved in working with assembly language and machine code. It can read and write binary executable files, examine memory, execute machine instructions in memory, modify registers, edit binary files, etc. The fact that this debugger program is always available with MS-DOS or Windows 98 system means that these systems are ready for some rudimentary assembly language programming without requiring any additional tools.

</main>