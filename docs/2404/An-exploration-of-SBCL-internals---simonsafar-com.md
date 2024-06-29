<!--yml

类别：未分类

日期：2024-05-27 13:25:39

-->

# SBCL 内部的探索 - simonsafar.com

> 来源：[https://simonsafar.com/2020/sbcl/](https://simonsafar.com/2020/sbcl/)

# SBCL 内部的探索

2020/07/10

[SBCL](http://www.sbcl.org) 是一种Common Lisp实现，具有高效的编译器，支持广泛的平台。它有出色的[文档](http://www.sbcl.org/manual/)，甚至有一个[内部指南](http://sbcl.org/sbcl-internals/)（...这是写给那些已经对其工作原理有很多了解的人的）。关于SBCL相当独特的构建过程的[这篇论文](https://research.gold.ac.uk/2336/1/sbcl.pdf)也值得一读。

本文的主要目标（...它甚至可能变成一系列文章）是介绍SBCL的内部结构（...作为一个展示Lisp系统内部工作原理的例子），同时不假设读者对其实现细节有很多了解。这也是一个“公共学习”的实验；在开始写作时，我实际上并不太了解SBCL的工作原理。我们将一起来理解这一点。

我们将试图回答以下问题：

+   运行时如何表示Lisp对象？我们在哪里存储字符串的长度？函数对象是什么样的？数字或Lisp符号呢？

+   编译后的Lisp代码在哪里？我们生成什么样的机器码？如果一个函数调用另一个函数，我们如何解析我们要调用的内容？

+   内存中特殊变量的绑定在哪里？或者线程本地变量呢？

+   REPL（...实际循环）的实现位置在哪里？

+   当我们评估**defun**时，我们如何调用编译器？我们能直接调用编译器，将Lisp形式转换为机器码吗？我们可以通过手动创建对象将机器码块转换为适当的Lisp函数，然后调用它吗？

+   内存分配器在哪里？普通函数如何调用它？它如何调用操作系统？

+   垃圾回收背后的基本理论是什么？它如何跟踪指针？它是否重新定位内存？

+   Lisp系统通常通过加载整个Lisp“核心”来启动。这些文件是什么样子的？我们是否将它们1:1映射到内存？我们能枚举它们所有的对象吗？

+   "Genesis"或冷启动在SBCL源代码中经常遇到是什么？

要跟随我们的进展，我们假设您对Lisp有一定了解，最好不是对x86汇编过于陌生，并且已经安装了SBCL（...我的设置还包括Emacs和SLIME，但请随意选择最适合您的工具）。

# 反汇编

让我们从一个相当基本的工具开始：反汇编器。让我们编写一个真正基本的函数：

```
 (defun testfunc (a b)
  (+ a b 33)) 
```

然后，我们可以通过以下方式调用反汇编器：

```
(disassemble 'testfunc)
```

我们得到的内容大致如下

```
 ; disassembly for TESTFUNC
; Size: 43 bytes. Origin: #x1002440235
; 35:       498B4D60         MOV RCX, [R13+96]                ; no-arg-parsing entry point
                                                              ; thread.binding-stack-pointer
; 39:       48894DF8         MOV [RBP-8], RCX
; 3D:       488B55F0         MOV RDX, [RBP-16]
; 41:       488B7DE8         MOV RDI, [RBP-24]
; 45:       FF1425A800B021   CALL QWORD PTR [#x21B000A8]      ; GENERIC-+
; 4C:       BF42000000       MOV EDI, 66
; 51:       FF1425A800B021   CALL QWORD PTR [#x21B000A8]      ; GENERIC-+
; 58:       488BE5           MOV RSP, RBP
; 5B:       F8               CLC
; 5C:       5D               POP RBP
; 5D:       C3               RET
; 5E:       CC0F             BREAK 15                         ; Invalid argument count trap
NIL 
```

我们要寻找的有趣部分是"MOV EDI, 66"。显然，RDX和RDI是"GENERIC-+"的两个参数，所以正在发生的是我们取两个参数，将它们相加，然后我们用第三个参数填充EDI，并再加上那个参数。由于我们目前还不知道的原因，我们将该整数向左移动一位，所以我们有66而不是33... 但该值显然与我们输入的常量有关。

所以... 让我们尝试稍微改动一下，来检查这个假设。如果你看一下左边的实际机器代码："BF420000..."... 你可能会注意到0x42在十进制中是66。（好吧，这个值必须要从*某处*获取？）如何调整代码中的实际值，使其读取比如0x50？（...这是80作为十进制数... 所以如果我们调用*(testfunc 0 0)*，我们期望它返回40，而不是当前的33。

所以... 我们需要一种实际写入内存位置的方法。我们甚至知道*在哪里*我们想要写入：在代码列表的顶部，我们可以看到

```
; Size: 43 bytes. Origin: #x1002440235
```

表明起源，并为每条指令提供有用的起始点。因此，我们的"0x42"字节位于**0x100244024d**（...因为...4C是第一个字节，...4D是第二个）。SBCL提供了一种从整数创建指针值的方式：[sb-sys:int-sap](http://www.sbcl.org/manual/#Untyped-memory)正是这样做的，我们可以通过调用来读取

```
 CL-USER> (sb-sys:sap-ref-8 (sb-sys:int-sap #x100244024d) 0)
          66 
```

（零值是相对于内存地址的偏移量）。这正是我们预期的。然后，我们可以*setf*相同的字节为我们想要的东西：

```
 CL-USER> (setf (sb-sys:sap-ref-8 (sb-sys:int-sap #x100244024d) 0) #x50)
80 
```

如果我们现在反汇编我们的函数，我们可以看到确实进行了修改：

```
 ; disassembly for TESTFUNC
; Size: 43 bytes. Origin: #x1002440235
; 35:       498B4D60         MOV RCX, [R13+96]                ; no-arg-parsing entry point
                                                              ; thread.binding-stack-pointer
; 39:       48894DF8         MOV [RBP-8], RCX
; 3D:       488B55F0         MOV RDX, [RBP-16]
; 41:       488B7DE8         MOV RDI, [RBP-24]
; 45:       FF1425A800B021   CALL QWORD PTR [#x21B000A8]      ; GENERIC-+
; 4C:       BF50000000       MOV EDI, 80
; 51:       FF1425A800B021   CALL QWORD PTR [#x21B000A8]      ; GENERIC-+
; 58:       488BE5           MOV RSP, RBP
; 5B:       F8               CLC
; 5C:       5D               POP RBP
; 5D:       C3               RET
; 5E:       CC0F             BREAK 15                         ; Invalid argument count trap
NIL 
```

... 而且，再次尝试我们的功能：

```
 CL-USER> (testfunc 0 0)
40 
```

好吧，Lisp毕竟不是魔法！

# 内存布局

现在让我们看看Lisp对象在内存中的布局！例如，一个cons单元是什么样子的？符号呢？或者一个字符串？

我们可以使用我们信赖的反汇编工具来获取事物的地址：

```
 (defun testfunc2 ()
  (format t "hello world")) 
```

... 比这更简单的方法我们找不到。我们得到的是...

```
 ; disassembly for TESTFUNC2
; Size: 70 bytes. Origin: #x10060509AC
; AC:       498B4D60         MOV RCX, [R13+96]                ; no-arg-parsing entry point
                                                              ; thread.binding-stack-pointer
; B0:       48894DF8         MOV [RBP-8], RCX
; B4:       498BBD28020000   MOV RDI, [R13+552]               ; tls: *STANDARD-OUTPUT*
; BB:       83FF61           CMP EDI, 97
; BE:       480F443C25D8B54A20 CMOVEQ RDI, [#x204AB5D8]       ; *STANDARD-OUTPUT*
; C7:       4883EC10         SUB RSP, 16
; CB:       488B1586FFFFFF   MOV RDX, [RIP-122]               ; "hello world"
; D2:       B904000000       MOV ECX, 4
; D7:       48892C24         MOV [RSP], RBP
; DB:       488BEC           MOV RBP, RSP
; DE:       B8F8943120       MOV EAX, #x203194F8              ; # <fdefn write-string="">; E3:       FFD0             CALL RAX
; E5:       BA17001020       MOV EDX, #x20100017              ; NIL
; EA:       488BE5           MOV RSP, RBP
; ED:       F8               CLC
; EE:       5D               POP RBP
; EF:       C3               RET
; F0:       CC0F             BREAK 15                         ; Invalid argument count trap</fdefn> 
```

所以... 呃... 那个字符串"hello world"应该与... [RIP-122]相关联？好吧，这很令人困惑。让我们尝试其他方法；我们将在本文末尾再回到这里。

此时，我们还可以介绍另一个非常有用的SBCL工具：**sb-kernel:get-lisp-obj-address**。我尚未找到关于它的实际文档（...所以我猜它随时可能会改变）；我实际上是从[Stack Overflow上*sds*的答案](https://stackoverflow.com/a/50613440/391376)中了解到它的。他的建议是*不要*在生产环境中使用它。但用来玩耍是完全可以的。

另一个工具是实际上倾倒内存内容。可能会有这样的东西，但写一个能够倾倒指定地址内存内容的东西也不需要花费很多时间。

```
 (defun hexdump-address (address)
  (let ((ptr (sb-sys:int-sap address)))
    (dotimes (row 5)
      (dotimes (col 16)
        (format t "~2,'0X " (sb-sys:sap-ref-8 ptr (+ (* row 16) col))))
      (format t "~%")))) 
```

我们只需将（数值）地址转换为系统区域指针，然后在打印十六进制数时遍历偏移量，并在每16个数后插入换行符。我相信您可以想出更加优雅/功能丰富的版本；尽管这样编写非常快速。

因此：让我们构造一个字符串：

```
 (defparameter *simple-test-string* "aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa") 
```

并尝试找到它的地址：

```
 CL-USER> (sb-kernel:get-lisp-obj-address *simple-test-string*)
68821502975
CL-USER> (hexdump-address 68821502975)
00 61 00 00 00 61 00 00 00 61 00 00 00 61 00 00
00 61 00 00 00 61 00 00 00 61 00 00 00 61 00 00
00 61 00 00 00 61 00 00 00 61 00 00 00 61 00 00
00 61 00 00 00 61 00 00 00 61 00 00 00 61 00 00
00 61 00 00 00 61 00 00 00 61 00 00 00 61 00 00
NIL
CL-USER> 
```

嗯，显然我们已经找到了点什么：0x61是字母"a"的ASCII码，这显然是一些重复的东西，就像我们测试字符串中许多相同的字母一样。此外，我们似乎为每个字符使用了4个字节（... [UTF-32](https://en.wikipedia.org/wiki/UTF-32) 我猜？）然而... 只是看着这个十六进制指针...

```
 CL-USER> (format t "~X" 68821502975)
100614CBFF 
```

... 谁会理智地分配一个以如此奇怪的地址开头的字符串？当然，x86（与例如ARM不同）对非对齐访问更加宽容，但这... 简直是纯粹的丑陋！然后在偏移量为1、5、9等处出现非零字节是怎么回事？这明显不像4字节整数应该看起来的样子。

所有这些的答案是0x100614CBFF实际上并不是一个指针。它不仅仅是一个指针。它是一个*标记化*指针。

在C语言中，编译器提前知道函数参数的类型。如果参数1按照相关调用约定以64位整数的形式传入寄存器，我们可以将其视为这样；如果它是一个指向结构体的指针，则会生成不同的代码。然而，在Lisp函数中，您可以传递*任何类型的对象*；我们如何生成可以确定并处理这两种类型的代码呢？

如果速度不是问题，我们可以采取“一切都是对象”的方法。也就是说，我们假设所有通过寄存器传入的参数都是指向通用对象类型的指针，该类型具有明确定义的布局，指出实际类型对象的位置。我们生成的代码读取此值，并根据实际类型执行不同的操作。

但是... 数字怎么办？这就是*装箱*的地方：为了将数字42视为对象处理，我们需要将其放入一个对象中，该对象说明“这是一个类型为'number'的对象，其值为42”。然后将其存储在堆上的某个位置。

当然，这并不是特别快：我们不能像通过值传递数字那样简单，我们需要一个指针解引用来确定实际值是什么... 然后将返回值打包到类似的盒子中（如果是一个数字），在需要时分配内存，并返回一个指针指向它。即使有了实际的编译代码，这仍然比C语言中简单返回一个数字慢得多。

标记指针来解救我们。与其使用整个64位寄存器来存储指针（... 通常指向对齐良好的内存区域，因此最终会以零位结尾），我们可以使用最后几位来指定这个*值*是什么类型。对于我的SBCL来说，标记是4位长，在值的末尾：只是一个单一的十六进制数字。其余的指针，嗯，就是一个真正的指针。

因此，如果我们只将指针的最后一个十六进制数位清零：

```
 CL-USER> (hexdump-address #x100614CBF0)
E5 00 00 00 00 00 00 00 46 00 00 00 00 00 00 00
61 00 00 00 61 00 00 00 61 00 00 00 61 00 00 00
61 00 00 00 61 00 00 00 61 00 00 00 61 00 00 00
61 00 00 00 61 00 00 00 61 00 00 00 61 00 00 00
61 00 00 00 61 00 00 00 61 00 00 00 61 00 00 00 
```

... 现在的结果看起来更合理了。这看起来像一堆对齐良好、相对较小的整数（记住我们在x86上，这是一个小端架构：**小**位于你**开始**的**末尾**，所以如果我们按照增加顺序列出内存地址，数字会反过来）。第一个字节是... 嗯... 某事，然后还有一件看起来可疑地像是长度加倍的东西（还记得第1部分中关于数字的一位移位吗？很快就会解释！）...

```
 CL-USER> (format t "0x~X" (length *simple-test-string*))
0x23 
```

... 然后一堆0x61来代表“a”。

我们甚至可以在内存区域中查找，将偏移为16的第一个“a”替换为“c”：

```
 CL-USER> (setf (sb-sys:sap-ref-8 (sb-sys:int-sap #x100614CBF0) 16) #x63)
99
CL-USER> *simple-test-string*
"caaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa"
CL-USER> 
```

... 或通过编辑偏移量8处的长度来缩短它：

```
 CL-USER> (setf (sb-sys:sap-ref-8 (sb-sys:int-sap #x100614CBF0) 8) #x5)
5
CL-USER> *simple-test-string*
"caa" 
```

（...这可能会破坏内存分配器，如果它尝试释放这部分内存的话，但在这一点上我们并不特别关心。）

这是编译SBCL对自己变得相当有用的时候。（如果你已经安装了SBCL，实际上只是git克隆，然后运行make.sh；在一台明显不是最新的笔记本上，这个过程花了我4分钟。）具体来说，在编译后，让我们看看SBCL源代码中的**src/runtime/genesis/constants.h**。这是一个生成的代码片段，基本上解释了即将构建的SBCL运行时（用C编写）中Lisp对象的外观。如果你手头没有，这里是一些相关部分：

```
 #define SBCL_GENESIS_CONSTANTS
#define FIXNUM_TAG_MASK 1 /* 0x1 */
#define N_FIXNUM_TAG_BITS 1 /* 0x1 */
#define N_LOWTAG_BITS 4 /* 0x4 */
#define N_WIDETAG_BITS 8 /* 0x8 */
#define N_WORD_BYTES 8 /* 0x8 */
#define LOWTAG_MASK 15 /* 0xF */
#define N_WORD_BITS 64 /* 0x40 */
#define WIDETAG_MASK 255 /* 0xFF */
#define SHORT_HEADER_MAX_WORDS 32767 /* 0x7FFF */

#define EVEN_FIXNUM_LOWTAG 0 /* 0x0 */
#define OTHER_IMMEDIATE_0_LOWTAG 1 /* 0x1 */
#define PAD0_LOWTAG 2 /* 0x2 */
#define INSTANCE_POINTER_LOWTAG 3 /* 0x3 */
#define PAD1_LOWTAG 4 /* 0x4 */
#define OTHER_IMMEDIATE_1_LOWTAG 5 /* 0x5 */
#define PAD2_LOWTAG 6 /* 0x6 */
#define LIST_POINTER_LOWTAG 7 /* 0x7 */
#define ODD_FIXNUM_LOWTAG 8 /* 0x8 */
#define OTHER_IMMEDIATE_2_LOWTAG 9 /* 0x9 */
#define PAD3_LOWTAG 10 /* 0xA */
#define FUN_POINTER_LOWTAG 11 /* 0xB */
#define PAD4_LOWTAG 12 /* 0xC */
#define OTHER_IMMEDIATE_3_LOWTAG 13 /* 0xD */
#define PAD5_LOWTAG 14 /* 0xE */
#define OTHER_POINTER_LOWTAG 15 /* 0xF */ 
```

这相当好地描述了我们到目前为止看到的内容。"lowtag"是Lisp对象末尾的那4个字节；另外，我们在这里显然是64位架构，N_WORD_BITS为64，其中8位保留给lowtag。它还详细介绍了lowtag的各种值，其中重要的一个是...

```
 #define OTHER_POINTER_LOWTAG 15 /* 0xF */ 
```

... 这正是我们在字符串指针末尾看到的`0xF`。显然，“其他指针”有很多。

函数指针的一个例外是（0xB）：

```
 CL-USER> (format t "0x~X" (sb-kernel:get-lisp-obj-address #'testfunc3))
0x10060508AB 
```

再次，这与头部的值非常匹配。列表（0x7）也是如此：

```
 CL-USER> (format t "0x~X" (sb-kernel:get-lisp-obj-address (list 'a 'b)))
0x10043CB067 
```

还有关于数字的解释。SBCL将fixnums存储在与指针完全相同的寄存器中；为了保持尽可能精确的精度，我们只需将以0位结尾的任何内容解释为数字（... 将实际数字位向左移动一次）。同时，如果最后一位是1，我们可以查看最后4位的其余部分，以确定我们正在处理的指针的类型。

**2024/04/22 更新：** 在[Hacker News上有一场讨论](https://news.ycombinator.com/item?id=40115083)!（此外，如果你在找相关文章，你可能也会喜欢[用于shell脚本的Common Lisp](/2021/lisp_scripting/)。）

（另外，也许这应该在某个时候有一个“第2部分”。）
