<!--yml

category: 未分类

date: 2024-05-27 13:04:00

-->

# Linux 下创建真正微小 ELF 可执行文件的快速教程

> 来源：[https://www.muppetlabs.com/%7Ebreadbox/software/tiny/teensy.html](https://www.muppetlabs.com/%7Ebreadbox/software/tiny/teensy.html)

# Linux 下创建真正微小 ELF 可执行文件的快速教程

（或者，“大小确实重要”）

* * *

> 她仔细地研究了大约 15 分钟。最后，她开口了。“这上面写着什么。”她皱着眉说，“但是字真的很小。”

[戴夫·巴里，《专栏作家的大案》](https://www.muppetlabs.com/%7Ebreadbox/software/tiny/teensy.html)

如果你是一个对软件膨胀感到厌倦的程序员，那么也许你会在这里找到完美的解药。

本文探讨了挤压简单程序中多余字节的方法。（当然，本文档的更实际目的是描述 ELF 文件格式和 Linux 操作系统的一些内部工作原理。但希望在此过程中你也能学到如何制作真正微小的 ELF 可执行文件的一些知识。）

请注意，这里提供的信息和示例在很大程度上是针对运行在 Intel x86 架构下的 Linux 平台上的 ELF 可执行文件。我想大部分信息也适用于其他基于 ELF 的 Unix 系统，但是我对这些系统的经验太有限了，无法确切地说。

如果你对汇编代码不太熟悉的话，可能会发现本文档的某些部分有些难以理解。（本文档中出现的汇编代码是使用 Nasm 编写的；参见 [http://www.nasm.us/](http://www.nasm.us/)。）

* * *

为了开始，我们需要一个程序。几乎任何程序都可以，但程序越简单越好，因为我们更关心能否尽可能地减小可执行文件的大小，而不是程序的功能。

让我们来看一个非常简单的程序，什么也不做，只是将一个数字返回给操作系统。为什么不呢？毕竟，Unix 已经带有不止两个这样的程序：true 和 false。由于 0 和 1 已经被使用了，我们将使用数字 42。

那么，这是我们的第一个版本：

```
  /* tiny.c */
  int main(void) { return 42; }

```

我们可以这样编译和测试：

```
  $ gcc -Wall tiny.c
  $ ./a.out ; echo $?
  42

```

那么。它有多大呢？在我的机器上，我得到的结果是：

```
  $ wc -c a.out
     3998 a.out

```

（你的版本可能有所不同。）诚然，按今天的标准来说，这还算是相当小的，但它几乎肯定比实际需要的要大。

显而易见的第一步是剥离可执行文件：

```
  $ gcc -Wall -s tiny.c
  $ ./a.out ; echo $?
  42
  $ wc -c a.out
     2632 a.out

```

这肯定是一个改进。接下来，如何优化呢？

```
  $ gcc -Wall -s -O3 tiny.c
  $ wc -c a.out
     2616 a.out

```

这也有所帮助，但仅此而已。这是有道理的：几乎没有任何东西可以优化。

看起来一个仅有一条语句的 C 程序几乎不可能再进行更多的缩减了。我们将不得不离开 C，改用汇编语言。希望这样能够去掉 C 程序自动引入的所有额外开销。

所以，到了我们的第二个版本。我们只需要在 main() 中返回 42。在汇编语言中，这意味着函数应该将累加器 eax 设置为 42，然后返回：

```
  ; tiny.asm
  BITS 32
  GLOBAL main
  SECTION .text
  main:
                mov     eax, 42
                ret

```

我们可以这样构建和测试：

```
  $ nasm -f elf tiny.asm
  $ gcc -Wall -s tiny.o
  $ ./a.out ; echo $?
  42

```

（嘿，谁说汇编代码很难？）现在它有多大呢？

```
  $ wc -c a.out
     2604 a.out

```

看起来我们仅减少了区区十二个字节。所以 C 自动引入的所有额外开销都如何呢？

嗯，问题在于我们仍然通过使用 main() 接口引入了大量开销。链接器仍然为我们添加了一个与操作系统的接口，并且实际上是这个接口调用了 main()。那么如果我们不需要它，该如何绕过呢？

链接器默认使用的实际入口点是带有名称 <def>_start</def> 的符号。当我们使用 gcc 进行链接时，它会自动包含一个 _start 程序，用于设置 argc 和 argv 等，然后调用 main()。

那么，让我们看看是否可以绕过这一点，并定义我们自己的 _start 程序：

```
  ; tiny.asm
  BITS 32
  GLOBAL _start
  SECTION .text
  _start:
                mov     eax, 42
                ret

```

gcc 会如我们所愿吗？

```
  $ nasm -f elf tiny.asm
  $ gcc -Wall -s tiny.o
  tiny.o(.text+0x0): multiple definition of `_start'
  /usr/lib/crt1.o(.text+0x0): first defined here
  /usr/lib/crt1.o(.text+0x36): undefined reference to `main'

```

不。嗯，实际上会，但首先我们需要学会如何要求我们想要的东西。

正好 gcc 识别出一个名为 <def>-nostartfiles</def> 的选项。从 gcc 的信息页上看：

> -nostartfiles
> 
> 在链接时不使用标准的系统启动文件。通常使用标准库。

啊哈！现在让我们看看我们能做什么：

```
  $ nasm -f elf tiny.asm
  $ gcc -Wall -s -nostartfiles tiny.o
  $ ./a.out ; echo $?
  Segmentation fault
  139

```

嗯，gcc 没有抱怨，但程序不工作。出了什么问题？

我们所犯的错误在于将 _start 当作 C 函数处理，并试图从中返回。实际上，它根本不是一个函数。它只是目标文件中的一个符号，链接器用它来定位程序的入口点。当我们的程序被调用时，它是直接调用的。如果我们看一下栈顶的值，会发现它是数字 1，这显然不像是地址。事实上，栈上存储的是我们程序的 argc 值。在这之后是 argv 数组的元素，包括终止的 NULL 元素，然后是 envp 的元素。就是这些。栈上没有返回地址。

那么，_start 怎么退出呢？嗯，它调用 exit() 函数！毕竟，这就是它的存在意义。

其实，我撒了个谎。它实际上是调用 _exit() 函数。（注意前面的下划线。）exit() 要为进程完成一些任务，但这些任务从未启动过，因为我们绕过了库的启动代码。因此，我们还需要绕过库的关闭代码，直接进入操作系统的关闭处理。

所以，让我们再试一次。我们要调用 _exit()，这是一个带有单个整数参数的函数。所以我们要做的就是将这个数字推送到栈上并调用这个函数。（我们还需要将 _exit() 声明为外部函数。）这是我们的汇编代码：

```
  ; tiny.asm
  BITS 32
  EXTERN _exit
  GLOBAL _start
  SECTION .text
  _start:
                push    dword 42
                call    _exit

```

然后我们像之前一样构建和测试：

```
  $ nasm -f elf tiny.asm
  $ gcc -Wall -s -nostartfiles tiny.o
  $ ./a.out ; echo $?
  42

```

终于成功了！现在它有多大？

```
  $ wc -c a.out
     1340 a.out

```

将近一半的大小！不错。不错。嗯……gcc还有哪些其他有趣而又隐秘的选项呢？

嗯，这个出现在文档中的`-nostartfiles`之后，确实引人注目：

> `-nostdlib`
> 
> 链接时不使用标准系统库和启动文件。只传递您指定的文件给链接器。

那肯定值得调查一下：

```
  $ gcc -Wall -s -nostdlib tiny.o
  tiny.o(.text+0x6): undefined reference to `_exit'

```

噢。没错…… _exit()毕竟是一个库函数。必须从某处填写它。

好吧。但是，毫无疑问，我们不需要libc的帮助来结束一个程序，对吧？

不，我们不需要。如果我们愿意放弃所有可移植性的假设，我们可以使我们的程序退出，而无需与任何其他东西链接。不过，在此之前，我们需要知道如何在Linux下进行<def>系统调用</def>。

* * *

Linux像大多数操作系统一样，通过系统调用为其托管的程序提供基本必需品。这包括诸如打开文件、读写文件句柄以及当然关闭进程等操作。

Linux系统调用接口是一条单一的指令：<def>int 0x80</def>。所有系统调用都通过这个中断完成。要进行系统调用，eax应包含指示调用哪个系统调用的编号，如果有的话，其他寄存器用于保存参数。如果系统调用需要一个参数，它将在ebx中；如果需要两个参数，将使用ebx和ecx。同样地，如果需要第三、第四或第五个参数，则使用edx、esi和edi。从系统调用返回时，eax将包含返回值。如果发生错误，eax将包含负值，其绝对值表示错误。

不同系统调用的编号列在`/usr/include/asm/unistd.h`中。快速浏览可以告诉我们，退出系统调用分配的编号是1。像C函数一样，它接受一个参数，即返回给父进程的值，因此这将放入ebx中。

现在我们知道了所有必要的信息，可以创建程序的下一个版本，而无需从任何外部函数获得帮助：

```
  ; tiny.asm
  BITS 32
  GLOBAL _start
  SECTION .text
  _start:
                mov     eax, 1
                mov     ebx, 42  
                int     0x80

```

我们开始吧：

```
  $ nasm -f elf tiny.asm
  $ gcc -Wall -s -nostdlib tiny.o
  $ ./a.out ; echo $?
  42

```

咔嚓！大小呢？

```
  $ wc -c a.out
      372 a.out

```

现在*这*才算小！几乎是之前版本的四分之一大小！

所以...我们还能做些什么来使它更小吗？

如何使用更短的指令？

如果我们为汇编代码生成一个列表文件，我们将找到以下内容：

```
  00000000 B801000000        mov        eax, 1
  00000005 BB2A000000        mov        ebx, 42
  0000000A CD80              int        0x80

```

哎呀，我们不需要初始化ebx的所有内容，因为操作系统只会使用最低字节。仅设置bl就足够了，而且只需两个字节而不是五个字节。

我们还可以通过将eax异或为零然后使用一个一字节的增量指令将其设置为一；这将节省两个字节。

```
  00000000 31C0              xor        eax, eax
  00000002 40                inc        eax
  00000003 B32A              mov        bl, 42
  00000005 CD80              int        0x80

```

我认为可以非常肯定地说，我们不会将这个程序做得比这更小了。

作为旁注，既然我们没有使用它的任何附加功能，我们也可以停止使用gcc来链接我们的可执行文件，只需自己调用链接器ld：

```
  $ nasm -f elf tiny.asm
  $ ld -s tiny.o
  $ ./a.out ; echo $?
  42
  $ wc -c a.out
      368 a.out

```

小了四个字节。（嘿！我们不是削减了五个字节吗？是的，我们确实是这样做了，但是在ELF文件中的对齐考虑导致它需要额外的一个填充字节。）

所以...我们到底达到了尽头吗？这就是我们能做到的最小尺寸了吗？

唔。我们的程序现在只有七个字节长了。ELF文件真的需要361字节的开销吗？这个文件里到底有什么？

我们可以使用objdump来查看文件的内容：

```
  $ objdump -x a.out | less

```

输出可能看起来像胡言乱语，但现在让我们专注于段列表：

```
  Sections:
  Idx Name          Size      VMA       LMA       File off  Algn
    0 .text         00000007  08048080  08048080  00000080  2**4
                    CONTENTS, ALLOC, LOAD, READONLY, CODE
    1 .comment      0000001c  00000000  00000000  00000087  2**0
                    CONTENTS, READONLY

```

完整的.text部分显示为七个字节长，正如我们所指定的那样。因此，可以安全地得出结论，我们现在完全控制了程序的机器语言内容。

但是还有另一个名为".comment"的部分。谁要求*那个*？它甚至有28个字节长！我们可能不确定这个.comment部分是什么，但很可能不是一个必要的功能……

.comment 部分被列在文件偏移 00000087（十六进制）处。如果我们使用一个 hexdump 程序来查看文件的这一区域，我们会看到：

```
  00000080: 31C0 40B3 2ACD 8000 5468 6520 4E65 7477  1.@.*...The Netw
  00000090: 6964 6520 4173 7365 6D62 6C65 7220 302E  ide Assembler 0.
  000000A0: 3938 0000 2E73 796D 7461 6200 2E73 7472  98...symtab..str

```

唔，唔，唔。谁能想到 Nasm 会像这样破坏我们的探索？也许我们应该转而使用 gas，尽管 AT&T 语法仍然......

唉，如果我们这样做：

```
  ; tiny.s
  .globl _start
  .text
  _start:
                xorl    %eax, %eax
                incl    %eax
                movb    $42, %bl
                int     $0x80

```

... 我们将会发现：

```
  $ gcc -s -nostdlib tiny.s
  $ ./a.out ; echo $?
  42
  $ wc -c a.out
      368 a.out

```

... 没有区别！

嗯，实际上有些区别。再次转向 objdump，我们看到：

```
  Sections:
  Idx Name          Size      VMA       LMA       File off  Algn
    0 .text         00000007  08048074  08048074  00000074  2**2
                    CONTENTS, ALLOC, LOAD, READONLY, CODE
    1 .data         00000000  0804907c  0804907c  0000007c  2**2
                    CONTENTS, ALLOC, LOAD, DATA
    2 .bss          00000000  0804907c  0804907c  0000007c  2**2
                    ALLOC

```

没有注释部分，但现在我们有两个用来存储不存在数据的无用部分。尽管这些部分的长度为零字节，它们仍然产生开销，毫无道理地增加了我们的文件大小。

好了，那么所有这些额外开销究竟是什么，我们该如何摆脱它？

好吧，要回答这些问题，我们必须开始深入研究一些真正的魔法。我们需要理解 ELF 格式。

* * *

关于 Intel-386 架构 ELF 格式的权威文档可以在 [http://refspecs.linuxbase.org/elf/elf.pdf](http://refspecs.linuxbase.org/elf/elf.pdf) 找到。（你也可以在 [http://www.muppetlabs.com/~breadbox/software/ELF.txt](http://www.muppetlabs.com/~breadbox/software/ELF.txt) 找到该标准的版本 1.0 的纯文本版本。）这份规范涵盖了大量内容，所以如果你不想自己读完整个文档，我能理解。基本上，这里是我们需要知道的内容：

每个ELF文件都以一个称为<def>ELF头部</def>的结构开始。该结构长为52字节，包含描述文件内容的几个信息。例如，前16字节包含一个"标识符"，其中包括文件的魔术数字签名（7F 45 4C 46），以及一些指示内容为32位或64位、小端或大端等的单字节标志。ELF头部的其他字段包含信息，例如目标体系结构；ELF文件是可执行文件、目标文件还是共享对象库；程序的起始地址；以及<def>程序头部表</def>和<def>节头部表</def>在文件中的位置。

这两个表可以出现在文件的任何位置，但通常前者紧随在ELF头部之后，后者出现在或接近文件的末尾。这两个表具有类似的目的，即识别文件的组成部分。然而，节头部表更专注于标识程序各部分在文件中的位置，而程序头部表描述了这些部分在内存中如何加载的位置和方式。简而言之，节头部表用于编译器和链接器使用，而程序头部表用于程序加载器使用。对于目标文件，程序头部表是可选的，在实践中几乎不会出现。同样地，对于可执行文件来说，节头部表是可选的，但几乎*总是*存在！

所以，这就是我们第一个问题的答案。我们程序中的一个相当大的开销是一个完全不必要的节头表，也许还有一些同样无用的部分，这些部分不会对我们程序的内存映像产生贡献。

因此，我们转向我们的第二个问题：我们如何才能摆脱所有这些？

唉，我们在这里是自己一个人了。没有标准工具愿意制作没有任何种类节头表的可执行文件。如果我们想要这样的东西，我们必须自己来做。

然而这并不意味着我们必须拿出二进制编辑器，手工编写十六进制值。好在老式的Nasm有一个扁平的二进制输出格式，这对我们非常有用。现在我们所需的只是一个空白的ELF可执行文件的图像，我们可以用我们的程序来填充它。我们的程序，仅此而已。

我们可以查看ELF规范和 /usr/include/linux/elf.h，以及标准工具创建的可执行文件，来了解我们的空白ELF可执行文件应该是什么样子。但是，如果你是急性子，你可以直接使用我这里提供的一个：

```
  BITS 32

                org     0x08048000

  ehdr:                                                 ; Elf32_Ehdr
                db      0x7F, "ELF", 1, 1, 1, 0         ;   e_ident
        times 8 db      0
                dw      2                               ;   e_type
                dw      3                               ;   e_machine
                dd      1                               ;   e_version
                dd      _start                          ;   e_entry
                dd      phdr - $$                       ;   e_phoff
                dd      0                               ;   e_shoff
                dd      0                               ;   e_flags
                dw      ehdrsize                        ;   e_ehsize
                dw      phdrsize                        ;   e_phentsize
                dw      1                               ;   e_phnum
                dw      0                               ;   e_shentsize
                dw      0                               ;   e_shnum
                dw      0                               ;   e_shstrndx

  ehdrsize      equ     $ - ehdr

  phdr:                                                 ; Elf32_Phdr
                dd      1                               ;   p_type
                dd      0                               ;   p_offset
                dd      $$                              ;   p_vaddr
                dd      $$                              ;   p_paddr
                dd      filesize                        ;   p_filesz
                dd      filesize                        ;   p_memsz
                dd      5                               ;   p_flags
                dd      0x1000                          ;   p_align

  phdrsize      equ     $ - phdr

  _start:

  ; your program here

  filesize      equ     $ - $$

```

这个图像包含一个ELF头部，标识文件为Intel 386可执行文件，没有节头表和一个程序头表，其中包含一个条目。该条目指示程序加载器将整个文件加载到内存中（程序包含其ELF头部和程序头表在其内存映像中是正常行为），从内存地址0x08048000开始加载（这是可执行文件加载的默认地址），并且从程序头表后紧接着的_start处开始执行代码。没有 .data 段，没有 .bss 段，没有注释 — 只有必需的内容。

所以，让我们加入我们的小程序：

```
  ; tiny.asm
                org     0x08048000

  ;
  ; (as above)
  ;

  _start:
                mov     bl, 42
                xor     eax, eax
                inc     eax
                int     0x80

  filesize      equ     $ - $$

```

并且尝试它：

```
  $ nasm -f bin -o a.out tiny.asm
  $ chmod +x a.out
  $ ./a.out ; echo $?
  42

```

我们刚刚完全从头开始创建了一个可执行文件。怎么样？现在，看看它的大小：

```
  $ wc -c a.out
       91 a.out

```

*九十一个* *字节*。比我们上一个尝试的大小小了四分之一，比我们第一次尝试的大小小了四十分之一！

更重要的是，这一次我们可以对每一个字节负责。我们完全知道可执行文件中的内容，以及为什么需要这些内容。这终于是极限了。比这还小我们就不能再做了。

或者，我们能吗？

* * *

嗯，如果你真的停下来读一下ELF规范，你可能会注意到一些事实。1）ELF文件的不同部分可以放在任何地方（除了必须在文件顶部的ELF头部），它们甚至可以彼此重叠。2）头部中的一些字段实际上是不被使用的。

特别是，我在考虑那个16字节识别字段末尾的一串零。它们纯粹是填充，为ELF标准的未来扩展腾出空间。所以操作系统根本不应关心里面是什么。而且我们已经把所有东西加载到内存中了，我们的程序只有七个字节长……

我们是否能把我们的代码放在ELF头部本身？

为什么不呢？

```
  ; tiny.asm

  BITS 32

                org     0x08048000

  ehdr:                                                 ; Elf32_Ehdr
                db      0x7F, "ELF"                     ;   e_ident
                db      1, 1, 1, 0, 0
  _start:       mov     bl, 42
                xor     eax, eax
                inc     eax
                int     0x80
                dw      2                               ;   e_type
                dw      3                               ;   e_machine
                dd      1                               ;   e_version
                dd      _start                          ;   e_entry
                dd      phdr - $$                       ;   e_phoff
                dd      0                               ;   e_shoff
                dd      0                               ;   e_flags
                dw      ehdrsize                        ;   e_ehsize
                dw      phdrsize                        ;   e_phentsize
                dw      1                               ;   e_phnum
                dw      0                               ;   e_shentsize
                dw      0                               ;   e_shnum
                dw      0                               ;   e_shstrndx

  ehdrsize      equ     $ - ehdr

  phdr:                                                 ; Elf32_Phdr
                dd      1                               ;   p_type
                dd      0                               ;   p_offset
                dd      $$                              ;   p_vaddr
                dd      $$                              ;   p_paddr
                dd      filesize                        ;   p_filesz
                dd      filesize                        ;   p_memsz
                dd      5                               ;   p_flags
                dd      0x1000                          ;   p_align

  phdrsize      equ     $ - phdr

  filesize      equ     $ - $$

```

毕竟，字节就是字节！

```
  $ nasm -f bin -o a.out tiny.asm
  $ chmod +x a.out
  $ ./a.out ; echo $?
  42
  $ wc -c a.out
       84 a.out

```

不错，对吧？

现在我们已经降到了最低点。我们的文件正好与一个ELF头部和一个程序头表条目一样长，这两者是我们必须的，才能被加载到内存中运行。所以现在没有什么可以再减少的了！

除了……

好吧，如果我们能对程序头表做与我们刚才对程序所做的事情一样的事情呢？让它与ELF头部重叠，这可能吗？

的确是可以的。看看我们的程序。注意ELF头部中的最后八个字节与程序头表中的前八个字节有某种相似之处。一种可能被描述为“相同”的相似之处。

所以……

```
  ; tiny.asm

  BITS 32

                org     0x08048000

  ehdr:
                db      0x7F, "ELF"             ; e_ident
                db      1, 1, 1, 0, 0
  _start:       mov     bl, 42
                xor     eax, eax
                inc     eax
                int     0x80
                dw      2                       ; e_type
                dw      3                       ; e_machine
                dd      1                       ; e_version
                dd      _start                  ; e_entry
                dd      phdr - $$               ; e_phoff
                dd      0                       ; e_shoff
                dd      0                       ; e_flags
                dw      ehdrsize                ; e_ehsize
                dw      phdrsize                ; e_phentsize
  phdr:         dd      1                       ; e_phnum       ; p_type
                                                ; e_shentsize
                dd      0                       ; e_shnum       ; p_offset
                                                ; e_shstrndx
  ehdrsize      equ     $ - ehdr
                dd      $$                                      ; p_vaddr
                dd      $$                                      ; p_paddr
                dd      filesize                                ; p_filesz
                dd      filesize                                ; p_memsz
                dd      5                                       ; p_flags
                dd      0x1000                                  ; p_align
  phdrsize      equ     $ - phdr

  filesize      equ     $ - $$

```

而Linux确实一点也不介意我们的节俭：

```
  $ nasm -f bin -o a.out tiny.asm
  $ chmod +x a.out
  $ ./a.out ; echo $?
  42
  $ wc -c a.out
       76 a.out

```

现在我们*真的*已经降到了最低点。没有办法比这更重叠这两个结构了。这些字节根本无法匹配。这就是终点！

除非，也就是说，我们能改变结构的内容，使它们更进一步地匹配……

Linux实际上看了这些字段中的多少？例如，Linux是否真的检查e_machine字段是否包含3（表示Intel 386目标），还是只是假定它包含？

事实上，在这种情况下确实如此。但令人惊讶的是，许多其他字段正在悄悄地被忽略。

所以：下面是 ELF 头部中的必要和非必要部分。前四个字节必须包含魔术数字，否则 Linux 将不予处理。然而，e_ident 字段中的其他三个字节未经检查，这意味着我们至少有连续的十二个字节可以随意设置。e_type 必须设置为 2，表示可执行文件，而 e_machine 必须设置为 3，正如前文所述。e_version 像 e_ident 内部的版本号一样，完全被忽略。（这在当前只有一种 ELF 标准版本的情况下可以理解。）e_entry 自然必须有效，因为它指向程序的起始位置。显然，e_phoff 需要包含文件中程序头表的正确偏移量，而 e_phnum 需要包含该表中的正确条目数。然而，据文档描述，对于 Intel 而言 e_flags 目前未使用，因此我们可以自由重用它。e_ehsize 应该用于验证 ELF 头部的预期大小，但 Linux 不予理会。e_phentsize 同样用于验证程序头表条目的大小。旧内核未对此进行检查，但现在需要正确设置。ELF 头部中的其他一切都与不适用于可执行文件的节头表有关。

那么现在来看程序头表条目如何处理？好吧，p_type 必须包含 1，以标记它为一个可加载段。p_offset 确实需要正确的文件偏移量来开始加载。同样，p_vaddr 需要包含正确的加载地址。然而，请注意，我们不一定要加载到 0x08048000。只要地址位于 0x00000000 以上、0x80000000 以下，并且页面对齐，几乎可以使用任何地址。p_paddr 字段文档中说明会被忽略，因此可以确保是空闲的。p_filesz 表示从文件中加载到内存的字节数，而 p_memsz 表示内存段需要多大，因此这些数字应该是相对合理的。p_flags 表示要给予内存段的权限。它需要是可读的 (4)，否则根本不能使用，同时还需要是可执行的 (1)，否则我们无法在其中执行代码。其他位也可能设置，但至少需要有这些权限。最后，p_align 给出内存段的对齐要求。当重定位包含位置独立代码的段时（例如共享库），这个字段主要用于这个目的，因此对于可执行文件，Linux 将忽略我们在这里存储的任何垃圾。

总而言之，这是相当多的灵活性。特别是，稍加审视就会发现 ELF 头部中大部分必要字段位于前半部分——而后半部分几乎完全可以用于混淆。考虑到这一点，我们可以将这两个结构交叉排列得比之前更多：

```
  ; tiny.asm

  BITS 32

                org     0x00200000

                db      0x7F, "ELF"             ; e_ident
                db      1, 1, 1, 0, 0
  _start:
                mov     bl, 42
                xor     eax, eax
                inc     eax
                int     0x80
                dw      2                       ; e_type
                dw      3                       ; e_machine
                dd      1                       ; e_version
                dd      _start                  ; e_entry
                dd      phdr - $$               ; e_phoff
  phdr:         dd      1                       ; e_shoff       ; p_type
                dd      0                       ; e_flags       ; p_offset
                dd      $$                      ; e_ehsize      ; p_vaddr
                                                ; e_phentsize
                dw      1                       ; e_phnum       ; p_paddr
                dw      0                       ; e_shentsize
                dd      filesize                ; e_shnum       ; p_filesz
                                                ; e_shstrndx
                dd      filesize                                ; p_memsz
                dd      5                                       ; p_flags
                dd      0x1000                                  ; p_align

  filesize      equ     $ - $$

```

正如你可以（希望）看到的那样，程序头表的前二十个字节现在与 ELF 头部的最后二十个字节重叠。实际上，这两者非常好地嵌合在一起。重叠区域内只有 ELF 头部的两个部分是重要的。第一个是 `e_phnum` 字段，恰好与 `p_paddr` 字段重合，后者是程序头表中少数几个绝对被忽略的字段之一。另一个是 `e_phentsize` 字段，它与 `p_vaddr` 字段的上半部分重合。通过选择一个非标准的程序加载地址，其上半部分等于 `0x0020`，这些字段得以匹配。

现在我们真正抛弃了所有的可移植性假设……

```
  $ nasm -f bin -o a.out tiny.asm
  $ chmod +x a.out
  $ ./a.out ; echo $?
  42
  $ wc -c a.out
       64 a.out

```

……但它确实起作用！而且程序正好少了十二个字节，完全符合预期。

这就是我说我们不能再做得比这更好的地方，但当然，我们已经知道我们可以——如果我们可以使程序头表完全位于 ELF 头部内。这个圣杯能够实现吗？

好吧，我们不能再将其上移十二个字节，否则会遇到无法调和两个结构中的几个字段的绝望障碍。唯一的其他可能性是让它紧接着第四个字节开始。这将使程序头表的第一部分舒适地位于 e_ident 区域内，但仍然存在其余部分的问题。经过一些实验，看起来这似乎不太可能。

然而，事实证明程序头表中还有几个字段可以倒置。

我们注意到，`p_memsz` 指示分配多少内存用于存储器段。显然，它至少应与 `p_filesz` 一样大，但如果更大也没有坏处。毕竟，我们要求内存并不意味着我们必须使用它。

其次，事实证明，与我所有的期望相反，`p_flags` 字段中的可执行位可以被移除。原来可读和可执行位是多余的：任何一个都将意味着另一个。

因此，考虑到这些事实，我们可以将文件重新组织为这个小怪物：

```
  ; tiny.asm

  BITS 32

                org     0x00010000

                db      0x7F, "ELF"             ; e_ident
                dd      1                                       ; p_type
                dd      0                                       ; p_offset
                dd      $$                                      ; p_vaddr 
                dw      2                       ; e_type        ; p_paddr
                dw      3                       ; e_machine
                dd      _start                  ; e_version     ; p_filesz
                dd      _start                  ; e_entry       ; p_memsz
                dd      4                       ; e_phoff       ; p_flags
  _start:
                mov     bl, 42                  ; e_shoff       ; p_align
                xor     eax, eax
                inc     eax                     ; e_flags
                int     0x80
                db      0
                dw      0x34                    ; e_ehsize
                dw      0x20                    ; e_phentsize
                dw      1                       ; e_phnum
                dw      0                       ; e_shentsize
                dw      0                       ; e_shnum
                dw      0                       ; e_shstrndx

  filesize      equ     $ - $$

```

`p_flags` 字段已从 5 更改为 4，正如我们注意到我们可以这样做。这个 4 也是 `e_phoff` 字段的值，它给出了程序头表在文件中的偏移量，而这正是我们定位它的地方。程序（还记得吗？）已被移动到 ELF 头的较低部分，从 `e_shoff` 字段开始，结束在 `e_flags` 字段内部。

请注意，加载地址已更改为一个更低的数字——事实上，尽可能低。这将使 `e_entry` 字段中的值保持在一个相当小的数字，这是很好的，因为它也是 `p_memsz` 的数字。 （实际上，使用虚拟内存几乎无所谓——我们可以将其保留在原始值，它仍然可以正常工作。但是保持礼貌也无妨。）

更改到 `p_filesz` 可能需要解释。因为我们没有在 `p_flags` 字段中设置写入位，Linux 不会允许我们定义大于 `p_filesz` 的 `p_memsz` 值，因为如果这些额外的字节不可写，则无法对其进行零初始化。由于我们不能改变 `p_flags` 字段而不使程序头表失调，你可能会认为唯一的解决方案是将 `p_memsz` 值降低到等于 `p_filesz`（这将使其无法与 `e_entry` 共享）。然而，另一种解决方案存在，即将 `p_filesz` 增加到等于 `p_memsz`。这意味着它们都比实际文件大小大得多——事实上大得多——但它使加载器免于写入只读内存，这正是它关心的。

-   因此...

```
  $ nasm -f bin -o a.out tiny.asm
  $ chmod +x a.out
  $ ./a.out ; echo $?
  42
  $ wc -c a.out
       52 a.out

```

……因此，随着程序头表和程序本身完全嵌入到ELF头部中，我们的可执行文件现在正好与ELF头部一样大！不多，也不少。而且*依然*可以在Linux上运行而毫无怨言！

现在，最终，我们真正且无疑地达到了绝对的最小可能。这点毋庸置疑，对吧？毕竟，我们必须拥有一个完整的ELF头部（即使它被严重破坏），否则Linux就不会浪费时间与我们交流！

对吧？

错了。我们还有最后一个卑鄙的手段。

如果文件的大小不完全符合完整的ELF头部大小，Linux仍然会正常工作，并用零填充缺失的字节。我们的文件末尾有不少于七个零，如果我们从文件镜像中删除它们：

```
  ; tiny.asm

  BITS 32

                org     0x00010000

                db      0x7F, "ELF"             ; e_ident
                dd      1                                       ; p_type
                dd      0                                       ; p_offset
                dd      $$                                      ; p_vaddr 
                dw      2                       ; e_type        ; p_paddr
                dw      3                       ; e_machine
                dd      _start                  ; e_version     ; p_filesz
                dd      _start                  ; e_entry       ; p_memsz
                dd      4                       ; e_phoff       ; p_flags
  _start:
                mov     bl, 42                  ; e_shoff       ; p_align
                xor     eax, eax
                inc     eax                     ; e_flags
                int     0x80
                db      0
                dw      0x34                    ; e_ehsize
                dw      0x20                    ; e_phentsize
                db      1                       ; e_phnum
                                                ; e_shentsize
                                                ; e_shnum
                                                ; e_shstrndx

  filesize      equ     $ - $$

```

……令人难以置信的是，我们仍然可以产生一个可运行的可执行文件：

```
  $ nasm -f bin -o a.out tiny.asm
  $ chmod +x a.out
  $ ./a.out ; echo $?
  42
  $ wc -c a.out
       45 a.out

```

*最后*，我们终于走到了尽头。无法回避的事实是文件的第45个字节，指定程序头表中条目数，必须是非零且必须存在，并且必须位于距离ELF头部开头的第45个位置。我们不得不得出结论，没有更多可以做的了。

* * *

这个45字节的文件比我们使用标准工具创建的最小ELF可执行文件小于八分之一，比我们使用纯C代码创建的最小文件小于五十分之一。我们已经剥离了文件中的所有可能，并且将大部分无法剥离的内容双重利用。

当然，这个文件中一半的数值违反了ELF标准的某些部分，真是奇迹般地Linux甚至愿意打个喷嚏在上面，更不用说给它一个进程ID了。这不是你通常愿意承认自己编写的程序类型。

另一方面，这个可执行文件中的每一个字节都可以被解释和证明。最近你创建了多少个可执行文件，你可以对*这*说这种话？

[(下一页)](teensyps.html)

* * *

[微型](http://www.muppetlabs.com/~breadbox/software/tiny/)

[软件](http://www.muppetlabs.com/~breadbox/software/)

[布莱恩·雷特尔](http://www.muppetlabs.com/~breadbox/)
