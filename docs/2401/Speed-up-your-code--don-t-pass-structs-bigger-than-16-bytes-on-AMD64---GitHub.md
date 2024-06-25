<!--yml

类别：未分类

日期：2024-05-27 14:32:21

-->

# 加速你的代码：不要在AMD64上传递大于16字节的结构体 · GitHub

> 来源：[https://gist.github.com/FeepingCreature/5dff669aad380a123b15659e195fb96c](https://gist.github.com/FeepingCreature/5dff669aad380a123b15659e195fb96c)

# 加速你的代码：不要在AMD64上传递大于16字节的结构体

[](#speed-up-your-code-dont-pass-structs-bigger-than-16-bytes-on-amd64)

或者称为“如何将[Neat](https://neat-lang.github.io)的速度提升了2倍”。

* * *

如果您检查[related_post_gen](https://github.com/jinyus/related_post_gen/)基准测试，您会发现我的语言Neat已经上升了几个位置。我是如何做到这一点的？我是否实现了新的高级优化器通行证，利用语言细节来暴露隐藏的优化潜力？

我将数组更改为作为三个指针参数传递，而不是一个由三个指针结构组成的参数。就是这样。

这个问题困扰着我很长时间。Neat似乎比应该的慢得多，特别是与D相比，如果我查看分析器，我会看到很多奇怪的堆栈移动。编译器似乎把大部分时间都花在重新排列大量堆栈以进行函数调用上。

为什么Neat数组是三个指针？与D语言不同，Neat使用引用计数器。这意味着数组除了起始和结束外，还需要指向数组对象基址的指针，该基址存储了引用计数。事实证明，D数组之所以快而Neat数组之所以如此慢，仅仅是因为24字节而不是16字节将它们置于不同的参数传递模式。

如果我们检查[SystemV AMD64 ABI规范](https://refspecs.linuxbase.org/elf/x86_64-abi-0.99.pdf)（PDF），它告诉我们，任何大于16字节的结构体都是通过指针传递的。（“如果聚合物的大小超过两个八字节，并且第一个八字节不是SSE或其他任何八字节不是SSEUP，则整个参数将通过内存传递。”）要通过内存传递结构体，我们在堆栈上分配一个结构体大小的位置，用我们传递的值填充它，然后将指针传递给函数。

现在，LLVM是一个非常好的优化器，但这并不给它留下太多余地。值*必须*放在堆栈上，这意味着必须在堆栈上有空间，必须将其从可能存放在的寄存器中复制出来，并且必须记住堆栈的哪些部分正在使用，哪些部分可以被另一个调用重用，而这在这方面表现得相当差。

我们可以通过这个基准测试来证明这个问题：

```
==========
harness.h:
==========

#define TYPE double

struct Vector { TYPE x, y, z; };

struct Vector vector_add_struct(struct Vector left, struct Vector right);

struct Vector vector_add_fields(
    TYPE left_x, TYPE left_y, TYPE left_z,
    TYPE right_x, TYPE right_y, TYPE right_z);

==========
harness.c:
==========

#include <stdio.h>
#include <stdlib.h>
#include "harness.h"

int main(int argc, const char *argv[])
{
    int mode = atoi(argv[1]);
    int length = atoi(argv[2]);
    struct Vector result = {0};
    if (mode == 0)
    {
        for (int i = 0; i < length; i++)
            result = vector_add_struct(result, (struct Vector) {i, i, i});
    }
    else
    {
        for (int i = 0; i < length; i++)
            result = vector_add_fields(result.x, result.y, result.z, i, i, i);
    }
    printf("result <%f, %f, %f>\n", result.x, result.y, result.z);
}

=======
impl.c:
=======

#include "harness.h"

struct Vector vector_add_struct(struct Vector left, struct Vector right)
{
    return (struct Vector) {
        left.x + right.x,
        left.y + right.y,
        left.z + right.z,
    };
}

struct Vector vector_add_fields(
    TYPE left_x, TYPE left_y, TYPE left_z,
    TYPE right_x, TYPE right_y, TYPE right_z)
{
    return (struct Vector) {
        left_x + right_x,
        left_y + right_y,
        left_z + right_z,
    };
} 
```

如您所见，根据参数的不同，这要么将一些值作为单独的参数传递，要么作为一个单独的大型结构体传递。模式和运行长度被传递到命令行上，以防止优化器将所有内容都折叠成常数。

我们必须单独编译impl.c以避免内联：

```
clang -O3 impl.c -c -o impl.o
clang -O3 harness.c impl.o -o benchmark
time ./benchmark 0 1000000000
time ./benchmark 1 1000000000 
```

这并不是微妙的变化：仅通过传递三个单独的字段而不是一个向量结构，我就从12.3秒变成了5.3秒！

如果我们检查汇编代码，我们确实可以看到有很多指令用于堆栈重排。事实上，字段版本的一个主要好处是参数已经以 SSE 寄存器的形式进入函数，而不必每次都从堆栈加载。这就是 SystemV ABI 的整个重点，以及它专注于尽可能在寄存器中传递值的原因，所以看到它在这里失败有点令人沮丧。我相信，考虑到 AMD64 上可用的寄存器数量，基准测试会显示对于超过 16 字节的类型，按值传递是有价值的。

事实上，如果你考虑代码所做的事情，通过在堆栈上写入字段，然后（显式而不是隐式地）传递一个指针，（新的、高效的）AMD64 系统 V ABI 事实上已经退化为旧的 x86 cdecl ABI，其中所有东西都是通过堆栈传递的！Cdecl 以其慢而著称，以至于它衍生出了多种旨在加速的调用约定。

当然，在任何真实的情况下，这段代码都会被全部内联。事实上，使用 gcc 打开 LTO（尽管有趣的是 clang 不会！）会消除两个版本之间的任何性能差异。但是，还是不是每个函数都能或应该被内联。

如果你调用的是 C API，你必须使用 C ABI。但是，许多非 C 语言内部的高级类型，尽管它们可能被呈现给编译器的后端作为一个结构体（例如 Neat 的三指针数组），严格来说并不需要*严格*被表示为一个。你是语言的编写者，你可以决定如何传递数组、元组、和 sumtypes 等。这就是为什么我选择将这些所有的东西（如果超过 16 个字节）都作为单独的字段传递，而且基准测试显示了好处。

因此，如果你在 AMD64 上，并且你要么在开发语言，要么在微优化 API，我建议你利用这个免费的加速。你至少应该进行基准测试，看看是否可以从手动拆分超过 16 个字节的结构体中受益。特别是在内部循环中，好处可能会出乎意料地大。

附录：

+   Q: 当然，或许对指针结构体来说这是正确的。但是，根据规范，`double`是在 SSE 类中的。为什么结构体不会被传递到 SSE 寄存器中呢？

+   A: 哥们，我不知道。我能告诉你的就是它没有。
