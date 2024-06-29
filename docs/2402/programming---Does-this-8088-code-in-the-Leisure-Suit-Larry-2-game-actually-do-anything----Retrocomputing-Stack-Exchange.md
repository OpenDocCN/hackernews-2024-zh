<!--yml

类别：未分类

日期：2024-05-29 13:24:42

-->

# 编程 - Leisure Suit Larry 2 游戏中的 8088 代码是否真的有用？ - Retrocomputing Stack Exchange

> 来源：[https://retrocomputing.stackexchange.com/questions/29565/does-this-8088-code-in-the-leisure-suit-larry-2-game-actually-do-anything](https://retrocomputing.stackexchange.com/questions/29565/does-this-8088-code-in-the-leisure-suit-larry-2-game-actually-do-anything)

这绝对不是一个程序员“编写”的程序，而是使用 [调试器](https://en.wikipedia.org/wiki/Debugger) 的反汇编命令列出的一些随机程序段。

你试过在程序中找到那段代码吗？

## 让我们看看我们得到了什么

仅凭这个（不完整的）部分很难说出什么明确的。可见的几个细节是：

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

*1 - 自第二个 INT 21h（L:04）在返回错误时，通过 INT 21h 要求 DOS 返回程序的返回码，可以推断出前一个 DOS 调用（L:01）很可能是一个 [EXEC 调用](https://fd.lod.bz/rbil/interrup/dos_kernel/214b.html)，用于作为子进程运行不同的程序。使用此函数还表明该程序是为 MS-DOS 2.0 或更高版本编写/编译的。

*2 - 提示所使用的调试器很可能是与 MS-DOS 一起提供的那种。

*3 - 第 08 行可能是前一个分支的目标，类似于 L:02 上的分支。这种结构通常是编译器的结构，用于从分支继续到距离超过 128 字节的目的地。

*4 - 一个入口点，类似于 *2，但在这里操作继续

*5 - 在这里使用 IRET 强烈表明该程序可能是一个 .COM 程序，并且这个指令用于终止它。

DOS 准备 COM 程序的内存方式与 CP/M 类似，包括在地址 0000h 处的跳转将其终止的方式。这是通过将其原始的 [终止中断](https://fd.lod.bz/rbil/interrup/dos_kernel/20.html) INT 20h 放置在 PSP 的起始处来完成的 - 对于 COM 程序而言，这是在 CS:0 处 - 并将一个字 0000h 推入栈中完成的。

现在可以使用简单的 RET NEAR 来终止。当然，这仅在 CS 在中间没有改变的情况下才有效。对于大多数简单程序来说，这是一个真实的条件，尤其是从 8080 代码转换而来的程序。为了在程序运行期间可能更改 CS 的情况下仍然使用这种终止方式，COM 程序开始时将 CS 和另一个字 0000h 推入栈中。现在可以使用 RETF，无论 CS 当时内容如何（只要所有栈级别都已清除）。

EXE 程序有一个小问题，因为它们的初始 CS 没有指向 PSP。但是 DS 在这两种类型的程序中都加载了 PSP 的地址，所以将 DS 推入栈而不是 CS 可使代码对两种类型的程序都可用。

由于这种“技巧”会在栈上留下一个字（由 DOS 提供），因此使用 IRET 代替 RET FAR 变得很常见，后者也会将该字一并从栈上移除。尽管没有任何不同，但感觉更好：))

*6 - 在不知道代码所在地址的情况下，几乎不可能进行任何猜测。但是，有两个仅相隔一个字节的跳转目标意味着要么使用了某种（对于x86）不寻常的程序技巧，要么014Fh包含单字节指令。考虑到IRET/INT组合相当不寻常，并且IRET是单字节指令，这很可能是其地址为014Fh的原因。但是再次，先前的JZ不会太有意义，因为无论哪种方式都将导致执行IRET。

*7 - 在DOS功能后检查AH是罕见的，因为标准返回码包含在AL（AX）中。获取返回码（参见L:03/04）反过来确实使用AH和AL不同。AL包含子返回码，而AH包含终止原因。这里的00h/01h是常规终止原因，而02h是关键错误中止。如果前面的INT 21h也是EXEC调用，检查上述01h以上将是有意义的。

*7 - 注意这与第02行使用的目标相同，这似乎是默认的良好退出。

*8 - 注意这与第14行使用的目标相同

* * *

## 从代码中得出的结论

该代码片段很可能是一个启动子程序并检查执行情况后继续的exec函数的一部分。因此，它可能是为不同任务（游戏部分？？）启动叠加模块的某些菜单系统/框架。它似乎是用某种高级语言编写的。

此外，它很可能不是来自DOS或任何操作系统级软件，而是用户空间，因为它使用了INT 21h功能。

使用IRET终止提示指向某些真正老旧的DOS 1.x原始代码，或者使用该技巧以节省代码大小的编译器，如果存在多个终止点，则必须是较旧的一个，因为MS不鼓励使用通过PSP:0和总体上DOS 3的INT 20h返回。

考虑到“良好”执行的目标（L:02/L:17）指向代码中的一个非常低的地址，这可能是从加载和执行子进程开始的无休止循环。再加上它在从子进程返回后检查其自身代码段中的标志来决定如何继续，我可以想象这是一个更大程序的加载器，其唯一工作是在正常终止时重新加载该程序，除非子进程在终止前设置了某些标志。

这样的程序可能是在不添加大量错误检查代码的情况下捕获某些错误的简单解决方案。任何错误检测将只退出子程序，然后从上述代码重新启动。

注意：这些都是猜测，不一定正确。除非在某个程序中找到了确切的代码，否则无法确定其来源。

## 现在，它是什么？

没有。我会说他们刚刚启动了他们的调试器，加载了一些程序，转储了21行的随机部分，并将它们复制到游戏文本中。可能来自游戏或任何其他程序。
