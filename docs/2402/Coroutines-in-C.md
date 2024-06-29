<!--yml

分类: 未分类

日期: 2024-05-29 13:21:59

-->

# C语言中的协程

> 来源：[https://www.chiark.greenend.org.uk/~sgtatham/coroutines.html](https://www.chiark.greenend.org.uk/~sgtatham/coroutines.html)

# C语言中的协程

*by [Simon Tatham](http://pobox.com/~anakin/)*

[协程三部曲：**C预处理器** | [C++20原生](https://www.chiark.greenend.org.uk/~sgtatham/quasiblog/coroutines-c++20/) | [一般哲学](https://www.chiark.greenend.org.uk/~sgtatham/quasiblog/coroutines-philosophy/) ]

## 介绍

构建一个大型程序总是一项困难的工作。经常出现的特定问题之一是：如果你有一段生成数据的代码，和另一段消耗数据的代码，哪一个应该是调用者，哪一个应该是被调用者？

这里是一个非常简单的行长解压缩代码片段，以及一个同样简单的解析器代码片段：

|

```
 /* Decompression code */
    while (1) {
        c = getchar();
        if (c == EOF)
            break;
        if (c == 0xFF) {
            len = getchar();
            c = getchar();
            while (len--)
                emit(c);
        } else
            emit(c);
    }
    emit(EOF);
```

|

```
 /* Parser code */
    while (1) {
        c = getchar();
        if (c == EOF)
            break;
        if (isalpha(c)) {
            do {
                add_to_token(c);
                c = getchar();
            } while (isalpha(c));
            got_token(WORD);
        }
        add_to_token(c);
        got_token(PUNCT);
    }
```

|

这些代码片段都非常简单，易于阅读和理解。一个通过调用 `emit()` 逐个生成字符；另一个通过调用 `getchar()` 逐个消耗字符。如果能让 `emit()` 的调用和 `getchar()` 的调用相互传递数据，那么连接这两个片段使得解压缩器的输出直接进入解析器就会很简单了。

在许多现代操作系统中，你可以使用两个进程或两个线程之间的管道来实现这一点。解压缩器中的 `emit()` 写入一个管道，而解析器中的 `getchar()` 从同一管道的另一端读取。简单而稳健，但也笨重且不可移植。通常情况下，你不希望为了这么简单的任务而将程序分成线程。

在本文中，我提出了一种解决这种结构问题的创造性解决方案。

## 重写

传统的答案是重写通信通道的一端，使其成为可以调用的函数。以下是每个示例片段可能意味着什么的示例。

|

```
int decompressor(void) {
    static int repchar;
    static int replen;
    if (replen &gt 0) {
        replen--;
        return repchar;
    }
    c = getchar();
    if (c == EOF)
        return EOF;
    if (c == 0xFF) {
        replen = getchar();
        repchar = getchar();
        replen--;
        return repchar;
    } else
        return c;
}
```

|

```
void parser(int c) {
    static enum {
        START, IN_WORD
    } state;
    switch (state) {
        case IN_WORD:
        if (isalpha(c)) {
            add_to_token(c);
            return;
        }
        got_token(WORD);
        state = START;
        /* fall through */

        case START:
        add_to_token(c);
        if (isalpha(c))
            state = IN_WORD;
        else
            got_token(PUNCT);
        break;
    }
}
```

|

当然你不需要重写它们两个都；只需一个即可。如果你按照所示的形式重写解压缩器，使其每次调用时返回一个字符，那么原始的解析器代码可以用调用 `decompressor()` 代替 `getchar()`，程序就会很顺利。反之，如果按照所示的形式重写解析器，使其每次对每个输入字符调用一次，那么原始的解压缩代码可以毫无问题地调用 `parser()` 而不是 `emit()`。只有当你喜欢自找苦吃时，你才会想要同时重写 *这两个* 函数作为被调用者。

而这确实是重点。这两个重写的函数都与它们的原始版本相比非常丑陋。在这里进行的两个过程在作为调用者而不是被调用者写入时更容易阅读。试图通过阅读代码推断由解析器识别的语法，或者由解压程序理解的压缩数据格式，你会发现原始版本都很清晰，而重写的形式都不太清晰。如果我们不必把任一段代码搞得乱七八糟，那就更好了。

## 克努斯的协程

在《计算机程序设计艺术》中，唐纳德·克努斯提出了解决这类问题的方法。他的答案是完全放弃堆栈的概念。不要再把一个进程看作是调用者，另一个是被调用者，而是把它们视为协作的平等体。

在实际操作中：用一个稍微不同的传统“调用”原语替换传统的“调用”原语。新的“调用”将返回值保存在堆栈之外的某个位置，并且会跳转到另一个保存的返回值指定的位置。因此，每当解压程序发出另一个字符时，它会保存其程序计数器并跳转到解析器内部的最后已知位置 - 每当解析器需要另一个字符时，它会保存自己的程序计数器并跳转到解压程序保存的位置。控制在这两个例程之间来回穿梭，正好在必要时。

理论上很好，但实际上你只能在汇编语言中做到这一点，因为没有常用的高级语言支持协程调用原语。像C这样的语言完全依赖于它们基于堆栈的结构，因此当控制从任何函数传递到任何其他函数时，一个必须是调用者，另一个必须是被调用者。所以，如果你想编写可移植的代码，这种技术至少和Unix管道解决方案一样不切实际。

## 基于堆栈的协程

所以，我们*真正*想要的是在C语言中模仿克努斯的协程调用原语的能力。我们必须接受，在C级别上，一个函数将是调用者，另一个函数将是被调用者。在调用者中，我们没有问题；我们编写原始算法，几乎完全按照原样，并且每当它有（或需要）一个字符时，它调用另一个函数。

被调用者面临的所有问题。对于我们的被调用者，我们希望有一个具有“返回并继续”操作的函数：从函数中返回，并在下次调用时从*return*语句后面的位置恢复控制。例如，我们希望能够编写一个这样的函数

```
int function(void) {
    int i;
    for (i = 0; i &lt 10; i++)
        return i;   /* won't work, but wouldn't it be nice */
}
```

并且有十个连续的调用来返回0到9这些数字。

我们如何实现这个呢？嗯，我们可以使用`goto`语句将控制转移到函数中的任意点。所以，如果我们使用一个状态变量，我们可以这样做：

```
int function(void) {
    static int i, state = 0;
    switch (state) {
        case 0: goto LABEL0;
        case 1: goto LABEL1;
    }
    LABEL0: /* start of function */
    for (i = 0; i &lt 10; i++) {
        state = 1; /* so we will come back to LABEL1 */
        return i;
        LABEL1:; /* resume control straight after the return */
    }
}
```

这种方法有效。我们在可能需要恢复控制的点上有一组标签：一个在开始时，另一个在每个 `return` 语句之后。我们有一个状态变量，在函数调用之间保持，告诉我们下一步需要恢复控制的标签。在任何返回之前，我们更新状态变量以指向正确的标签；在任何调用之后，我们根据变量的值进行 `switch`，以找出要跳转到哪里。

尽管仍然很丑陋。最糟糕的部分是必须手动维护标签集，并且必须在函数体和初始 `switch` 语句之间保持一致。每次添加新的返回语句时，我们必须发明一个新的标签名称并将其添加到 `switch` 中的列表中；每次删除返回语句时，我们必须删除其对应的标签。我们刚刚将我们的维护工作量增加了两倍。

## 达夫设备

C 语言中著名的“达夫设备”利用了 `case` 语句仍然可以在其匹配 `switch` 语句的子块中合法的事实。汤姆·达夫将其用于优化的输出循环：

```
 switch (count % 8) {
        case 0:        do {  *to = *from++;
        case 7:              *to = *from++;
        case 6:              *to = *from++;
        case 5:              *to = *from++;
        case 4:              *to = *from++;
        case 3:              *to = *from++;
        case 2:              *to = *from++;
        case 1:              *to = *from++;
                       } while ((count -= 8) &gt 0);
    }
```

我们可以在协程技巧中稍作修改。我们可以使用 `switch` 语句执行跳转本身，而不是使用 `switch` 语句决定执行哪个 `goto` 语句：

```
int function(void) {
    static int i, state = 0;
    switch (state) {
        case 0: /* start of function */
        for (i = 0; i &lt 10; i++) {
            state = 1; /* so we will come back to "case 1" */
            return i;
            case 1:; /* resume control straight after the return */
        }
    }
}
```

现在看起来很有希望了。现在我们所要做的就是构建几个精心选择的宏，然后我们可以将令人讨厌的细节隐藏在一个看起来合理的东西中：

```
#define crBegin static int state=0; switch(state) { case 0:
#define crReturn(i,x) do { state=i; return x; case i:; } while (0)
#define crFinish }
int function(void) {
    static int i;
    crBegin;
    for (i = 0; i &lt 10; i++)
        crReturn(1, i);
    crFinish;
}
```

（请注意使用 `do ... while(0)` 来确保 `crReturn` 直接放置在 `if` 和 `else` 之间时不需要加大括号）

这几乎就是我们想要的。我们可以使用 `crReturn` 以一种方式从函数中返回，以便在下一次调用时控制恢复到返回之后的位置。当然，我们必须遵守一些基本规则（将函数体用 `crBegin` 和 `crFinish` 包围起来；如果需要跨 `crReturn` 保留所有局部变量，则将它们声明为 `static`；*永远不要*在显式的 `switch` 语句中放置 `crReturn` ）；但这些并不限制我们太多。

`crReturn` 的唯一问题是其第一个参数。正如我们在前一节中为新标签命名时要避免与现有标签名称冲突一样，现在我们必须确保所有传递给 `crReturn` 的状态参数都是不同的。后果将相当温和 - 编译器将捕获并在运行时防止其做出可怕的事情 - 但我们仍然需要避免这样做。

即使这个问题也可以解决。ANSI C 提供了特殊的宏名 `__LINE__`，它展开为当前源代码行号。因此，我们可以将 `crReturn` 重写为

```
#define crReturn(x) do { state=__LINE__; return x; \
                         case __LINE__:; } while (0)
```

然后我们再也不必担心那些状态参数了，只要我们遵守第四个基本规则（永远不要在同一行上放置两个 `crReturn` 语句）。

## 评估

现在我们有了这个怪物，让我们使用它来重新编写我们最初的代码片段。

|

```
int decompressor(void) {
    static int c, len;
    crBegin;
    while (1) {
        c = getchar();
        if (c == EOF)
            break;
        if (c == 0xFF) {
            len = getchar();
            c = getchar();
            while (len--)
	        crReturn(c);
        } else
	    crReturn(c);
    }
    crReturn(EOF);
    crFinish;
}
```

|

```
void parser(int c) {
    crBegin;
    while (1) {
        /* first char already in c */
        if (c == EOF)
            break;
        if (isalpha(c)) {
            do {
                add_to_token(c);
		crReturn( );
            } while (isalpha(c));
            got_token(WORD);
        }
        add_to_token(c);
        got_token(PUNCT);
	crReturn( );
    }
    crFinish;
}
```

|

我们已将解压器和解析器都重写为被调用者，这次不需要像上次那样进行大规模的重构。每个函数的结构完全反映了其原始形式的结构。读者可以更轻松地推断出解析器所识别的语法，或解压器使用的压缩数据格式，而不是阅读晦涩的状态机代码。控制流一旦您理解了新格式，就会变得直观：当解压器有一个字符时，它通过`crReturn`将其返回给调用者，并在需要另一个字符时等待再次调用。当解析器需要另一个字符时，它使用`crReturn`返回，并在参数`c`中等待用新字符再次调用。

代码结构上有一个小的结构性变化：`parser()`现在在循环结束时有它的`getchar()`（实际上是相应的`crReturn`），因为当进入函数时第一个字符已经在`c`中了。我们可以接受这种结构上的小变化，或者如果我们确实感觉很强烈，我们可以指定`parser()`在开始传递字符之前需要一个“初始化”调用。

当然，和以前一样，我们不必使用协程宏重写两个例程。一个足矣；另一个可以作为其调用者。

我们已经实现了我们的目标：在生产者和消费者之间传递数据的便携式 ANSI C 方法，而不需要将其重写为显式状态机。我们通过将 C 预处理器与 `switch` 语句的一个不常用特性结合起来创建了一个*隐式*状态机。

## 编码标准

当然，这种技巧违反了书中的所有编码标准。试图在公司的代码中这样做，您可能会受到严厉的警告，甚至是纪律处分！您在宏中嵌入了不匹配的大括号，使用了`case`在子块中，至于具有极具破坏性内容的`crReturn`宏……真是令人惊讶，您居然没有因为如此不负责任的编码实践而当场被解雇。您应该为自己感到羞愧。

我要声明编码标准在这里有问题。我在本文中展示的例子并不长，也不复杂，即便重写为状态机，仍然可以理解。但是随着函数变得越来越长，需要重写的程度变得更大，清晰度的损失也变得更为严重。

考虑。由小块形式构建的函数

```
 case STATE1:
    /* perform some activity */
    if (condition) state = STATE2; else state = STATE3;
```

对读者来说，与由小块形式构建的函数并没有太大的不同

```
 LABEL1:
    /* perform some activity */
    if (condition) goto LABEL2; else goto LABEL3;
```

一个是调用者，另一个是被调用者，没错，但是函数的视觉结构是相同的，它们对底层算法的洞察力也是一样的微小。那些因为使用我的协程宏而解雇你的人，也同样会因为用 `goto` 语句连接的小块构建函数而解雇你！这一次他们是对的，因为这样布置函数会严重模糊算法的结构。

编程规范旨在提高代码的清晰度。通过将关键的 `switch`、`return` 和 `case` 语句隐藏在“混淆”的宏定义中，编程规范会认为你已经模糊了程序的语法结构，并且违反了清晰度的要求。但实际上，你这样做是为了揭示程序的 *算法* 结构，这才是读者真正想要了解的内容！

任何强调语法清晰而损害算法清晰的编码标准都应该重新编写。如果你的雇主因为你使用这一技巧而解雇你，告诉他们，当保安把你拖出大楼时，你可以重复这样说。

## 修正和代码

在实际应用中，这种玩具般的协程实现可能不会很有用，因为它依赖于 `static` 变量，所以不能实现可重入性或多线程性。理想情况下，在真实应用中，你希望能够在几个不同的上下文中调用同一个函数，并且在每个上下文的每次调用中，控制都能从上次返回的地方继续执行。

这很容易实现。我们增加一个额外的函数参数，即指向上下文结构的指针；我们将所有局部状态和协程状态变量声明为该结构的元素。

这有点丑陋，因为突然间你必须使用 `ctx->i` 作为循环计数器，而以前你只需使用 `i`；几乎所有重要的变量都成为协程上下文结构的一部分。但这消除了可重入性的问题，并且仍然没有影响到例程的 *结构*。

（当然，如果 C 语言有 Pascal 的 `with` 语句，我们可以安排宏使得这层间接性真正透明化。遗憾。不过，至少 C++ 用户可以通过将协程作为类成员来管理所有局部变量，从而使得作用域隐式化。）

此处包含一个实现此协程技巧的 C 头文件，其中定义了两组预定义宏，分别以 `scr` 和 `ccr` 为前缀。`scr` 宏是该技巧的简单形式，适用于可以使用 `static` 变量的情况；`ccr` 宏则提供高级的可重入形式。完整的文档记录在头文件的注释中。

请注意，Visual C++版本6不喜欢这个协程技巧，因为它的默认调试状态（用于编辑和继续的程序数据库）对`__LINE__`宏做了一些奇怪的事情。要在VC++ 6中编译使用协程的程序，您必须关闭编辑和继续功能。（在项目设置中，转到“C/C++”选项卡，类别“常规”，设置“调试信息”。选择除“程序数据库用于编辑和继续”之外的任何选项。）

（头文件采用MIT许可，因此您可以在任何您喜欢的地方使用它，没有限制。如果您发现MIT许可不允许您做某些事情，[给我发邮件](mailto:anakin@pobox.com)，我可能会给您明确的许可。）

点击此链接获取`coroutine.h`。

谢谢阅读。分享和享受！

## 参考文献

* * *

*$Id$*

版权所有 © 2000 Simon Tatham.

本文为[开放内容](https://www.opencontent.org/)。

您可以按照[开放内容许可](https://www.opencontent.org/openpub/)的条款复制和使用这段文字。

请将评论和批评发送至[anakin@pobox.com](mailto:anakin@pobox.com)。
