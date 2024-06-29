<!--yml

category: 未分类

date: 2024-05-27 14:32:17

-->

# 二进制可执行文件是如何组织的？让我们来探索一下吧！

> 来源：[https://jvns.ca/blog/2014/09/06/how-to-read-an-executable/](https://jvns.ca/blog/2014/09/06/how-to-read-an-executable/)

我曾经以为可执行文件完全是不可渗透的。我编译了一个C程序，然后就是这样了！我有了一个神奇的二进制可执行文件，我再也看不懂它了。

原来并非如此！可执行文件格式是常规的文件格式，你可以理解它们。我将解释一些简单的工具来开始吧！我们将在Linux上操作，使用ELF二进制文件格式。（二进制文件是平台特定的定义，所以这都是平台特定的。）我们将使用C语言，但你也可以轻松地查看任何编译语言的输出。

让我们写一个简单的C程序，`hello.c`：

```
#include <stdio.h>

int main() {
    printf("Penguin!\n");
} 
```

然后我们编译它（`gcc -o hello hello.c`），得到一个名为`hello`的二进制文件。最初这似乎是不可渗透的（我们甚至如何二进制？！），但让我们看看如何进行调查！我们将学习**符号**、**段**和**节**是什么。在高层次上：

+   **符号**就像函数名一样，用来回答“如果我调用`printf`而它在其他地方定义，我该如何找到它？”

+   symbols are organized into **sections** – code lives in one section (`.text`), and data in another (`.data`, `.rodata`)

+   sections are organized into **segments**

Throughout we’ll use a tool called `readelf` to look at these.

因此，让我们深入研究我们的二进制文件吧！

## 步骤1：在文本编辑器中打开它！

这是查看二进制文件的最原始的方法。如果运行`cat hello`，我得到类似这样的内容：

```
ELF>@@H@8
@@@@@@��88@@@@�� ((`(`�
PP`P`��P�td@,,Q�tdR�td((`(`��/lib64/ld-linux-x86-64.so.2GNUGNUϨ�n��8�w�j7*oL�h��
__gmon_start__libc.so.6puts__libc_start_mainGLIBC_2.2.5ui
1```H��k����H���5 H�[]�fff.�H�=p

UH��t�H��]�H`��]Ð�UH����@�����]Ð�����������H�l$�L�d$�H�- L�%

L�l$�L�t$�L�|$�H�\$�H��8L)�A��I��H��I���s���H��t1@L��L��D��A��H��H9�u�H�\H�l$L�d$L�l$

L�t$(L�|$0H��8��Ð�������������UH��SH�H�

H���t�(`DH���H�H���u�H�[]Ð�H��o���H��Penguin!;,����H

```

There’s text here, though! This was not a total failure. In particular it says “Penguin!” and “ELF”. ELF is the name of the binary format. So that’s something! Then there are a bunch of unprintable symbols, which isn’t a huge surprise because this is a binary.

## Step 2: use `readelf` to see the symbol table

Throughout we’re going to use a tool called `readelf` to explore our binary. Let’s start by running `readelf --symbols` on it. (another popular tool to do this is `nm`)

```

$ readelf --symbols hello

Num:    Value          Size Type    Bind   Vis      Ndx Name

    48: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND puts@@GLIBC_2.2.5

    59: 0000000000400410     0 FUNC    GLOBAL DEFAULT   13 _start

    61: 00000000004004f4    16 FUNC    GLOBAL DEFAULT   13 main

```

([full output here](https://gist.github.com/jvns/0f82a7655d32bb6b331e))

Here we see three *symbols*: `main` is the address of my `main()` function. `puts` looks like a reference to the `printf` function I called in it (which I guess the compiler changed to `puts` as an optimization?). `_start` is pretty important.

When the program starts running, you might think it starts at `main`. It doesn’t! It *actually* goes to `_start`. This does a bunch of Very Important Things that I don’t understand very well, including calling `main`. So I won’t explain them.

So, what’s a symbol?

### Symbols

When you write a program, you might write a function called `hello`. When you compile the program, the binary for that function is labelled with a **symbol** called `hello`. If I call a function (like `printf`) from a library, we need a way to look up the code for that function! The process of looking up functions from libraries is called **linking**. It can happen either just after we compile the program (“static linking”) or when we run the program (“dynamic linking”).

So symbols are what allow linking to work! Let’s find the symbol for printf! It’ll be in `libc`, where all the C standard library functions are.

If I run `nm` on my copy of libc, it tells me “no symbols”. But the internet tells me I can use `objdump -tT` instead! This works! `objdump -tT /lib/x86_64-linux-gnu/libc-2.15.so` gives me [this output](https://gist.github.com/jvns/13bae55c3d463cdad809).

If you look at it, you’ll see `sprintf`, `strlen`, `fork`, `exec`, and everything you might expect libc to have. From here we can start to imagine how dynamic linking works – we see that `hello` calls `puts`, and then we can look up the location of `puts` in libc’s symbol table.

## Step 3: use `objdump` to see the binary, and learn about sections!

Opening our binary in a text editor was a bad way to open it. `objdump` is a better way. Here’s an excerpt:

```

$ objdump -s hello

Contents of section .text:

400410 31ed4989 d15e4889 e24883e4 f0505449  1.I..^H..H...PTI

400420 c7c0a005 400048c7 c1100540 0048c7c7  ....@.H....@.H..

400430 f4044000 e8c7ffff fff49090 4883ec08  ..@.........H...

Contents of section .interp:

400238 2f6c6962 36342f6c 642d6c69 6e75782d  /lib64/ld-linux-

400248 7838362d 36342e73 6f2e3200           x86-64.so.2\.

Contents of section .rodata:

4005f8 01000200 50656e67 75696e21 00        ....企鹅！.

```

You can see that it shows us all the bytes in the file as hex on the left, and a translation into ASCII on the right.

The are a whole bunch of **sections** here (see [this gist](https://gist.github.com/jvns/64aa2c85e083e0031609) for the whole thing). This shows you all the bytes in your binary! Some sections we care about:

*   `.text` is the program’s actual code (the assembly). `_start` and `main` are both part of the `.text` section.
*   `.rodata` is where some read-only data is stored (in this case, our string “Penguin!”)
*   `.interp` is the filename of the dynamic linker!

The major difference between *sections* and *segments* is that sections are used at link time (by `ld`) and segments are used at execution time. `objdump` shows us the contents of the sections, which is nice, but doesn’t give us as much metadata about the sections as I’d like. Let’s try `readelf` instead:

```

$ readelf --sections hello

Section Headers:

[Nr] Name              Type             Address           Offset

    Size              EntSize          Flags  Link  Info  Align

[13] .text             PROGBITS         0000000000400410  00000410

    00000000000001d8  0000000000000000  AX       0     0     16

[15] .rodata           PROGBITS         00000000004005f8  000005f8

    000000000000000b  0000000000000000   A       0     0     4

[24] .data             PROGBITS         0000000000601010  00001010

    0000000000000010  0000000000000000  WA       0     0     8

[25] .bss              NOBITS           0000000000601020  00001020

    0000000000000010  0000000000000000  WA       0     0     8

[26] .comment          PROGBITS         0000000000000000  00001020

    000000000000002a  0000000000000001  MS       0     0     1

标志键:

W (write), A (alloc), X (execute), M (merge), S (strings), l (large)

I (信息), L (链接顺序), G (组), T (TLS), E (排除), x (未知)

O (extra OS processing required) o (OS specific), p (processor specific)

```

([full output](https://gist.github.com/jvns/37ce4ad26758b403f6b3))

Neat! We can see `.text` is executable and read-only, `.rodata` (“read only data”) is read-only, and `.data` is read-write.

## Step 4: Look at some assembly!

We mentioned briefly that `.text` contains assembly code. We can actually look at what it is really easily. If we were magicians, we would already be able to read and understand this:

```

.text 段内容:

400410 31ed4989 d15e4889 e24883e4 f0505449  1.I..^H..H...PTI

400420 c7c0a005 400048c7 c1100540 0048c7c7  ....@.H....@.H..

400430 f4044000 e8c7ffff fff49090 4883ec08  ..@.........H...

```

It starts with `31ed4989`. Those are bytes that our CPU interprets as code! And runs! However we are not magicians (I don’t know what `31 ed` means!) and so we will use a disassembler instead.

```

$ objdump -d ./hello

.text 段反汇编:

0000000000400410 <_start>:

400410:       31 ed                   xor    %ebp,%ebp

400412:       49 89 d1                mov    %rdx,%r9

400415:       5e                      pop    %rsi

400416:       48 89 e2                mov    %rsp,%rdx

400419:       48 83 e4 f0             and    $0xfffffffffffffff0,%rsp

```

[full output here](https://gist.github.com/jvns/75298b0a5b6cde5de175)

So we see that `31 ed` is xoring two things. Neat! That’s all the assembly we’ll do for now.

## Step 5: Segments!

Finally, a program is organized into **segments** or **program headers**. Let’s look at the segments for our program using `readelf --segments hello`.

```

程序头:

[... removed ...]

INTERP         0x0000000000000238 0x0000000000400238 0x0000000000400238

                0x000000000000001c 0x000000000000001c  R      1

    [请求程序解释器：/lib64/ld-linux-x86-64.so.2]

LOAD           0x0000000000000000 0x0000000000400000 0x0000000000400000

                0x00000000000006d4 0x00000000000006d4  R E    200000

LOAD           0x0000000000000e28 0x0000000000600e28 0x0000000000600e28

                0x00000000000001f8 0x0000000000000208  RW     200000

[... removed ...]

节到段映射:

分段节...

00

01     .interp

02 .interp .note.ABI-tag .note.gnu.build-id .gnu.hash .dynsym

    .dynstr .gnu.version .gnu.version_r .rela.dyn .rela.plt .init .plt

    .text .fini .rodata .eh_frame_hdr .eh_frame

03     .ctors .dtors .jcr .dynamic .got .got.plt .data .bss

04     .dynamic

05     .note.ABI-tag .note.gnu.build-id

06     .eh_frame_hdr

07

08     .ctors .dtors .jcr .dynamic .got

```

段用于确定如何将程序的不同部分分配到内存中。第一个 `LOAD` 段标记为 R E（读取/执行），第二个为 `RW`（读取/写入）。`.text` 在第一个段中（我们要读取它但从不写入），`.data`、`.bss` 在第二个段中（我们需要写入它们，但不执行它们）。

## 不是魔法！

可执行文件并非魔法。ELF 是一种像其他文件格式一样的文件格式！您可以使用 `readelf`、`nm` 和 `objdump` 来检查您的 Linux 二进制文件。试试看！玩得开心。

其他资源:

非常感谢了不可思议的 [Allison Kaptur](http://akaptur.github.io) 和 [Dan Luu](http://danluu.com) 在阅读草稿时提供的帮助。
