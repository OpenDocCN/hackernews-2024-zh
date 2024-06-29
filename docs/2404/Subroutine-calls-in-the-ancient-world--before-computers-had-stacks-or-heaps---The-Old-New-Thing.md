<!--yml

category: 未分类

date: 2024-05-27 12:52:53

-->

# 古代世界的子程序调用，在计算机没有栈或堆之前 - The Old New Thing

> 来源：[https://devblogs.microsoft.com/oldnewthing/20240401-00/?p=109599](https://devblogs.microsoft.com/oldnewthing/20240401-00/?p=109599)

# 古代世界的子程序调用，在计算机没有栈或堆之前

现在我们对栈和堆理所当然，但在计算机非常古老的日子里，计算机是没有栈或堆的。

告诉最近刚毕业的大学生这件事，你可能也可以告诉他们曾经有一段时间你没有即时访问数百万只猫的视频。

想象一下没有动态内存分配并不难。您只需为所有内容使用固定大小的内存缓冲区。如果必须操作可变大小的数据，您将保留一个足够大以容纳任何您合理预期处理的数据的固定大小缓冲区，如果有人要求更多，则只需用致命错误退出程序。如果您真的好心，您将提供一个编译时配置，以便您的客户可以调整最大容量以适应其数据集。如果您真的高级，您将编写一个自定义分配器，该分配器将在该固定大小缓冲区上运行，以便人们可以从中“分配”和“释放”内存。

但是没有栈如何调用函数？如果没有栈用于返回地址或局部变量的话？

它的运作方式如下。

首先，编译器为每个传入函数参数定义了一个秘密全局变量，以及另一个用于保存返回地址的函数的秘密全局变量。它还为函数的每个局部变量定义了一个秘密全局变量。

生成函数调用时，编译器会将参数值分配给相应的秘密全局变量，并将返回地址分配给函数的秘密“返回地址变量”，然后跳转到函数的起始位置。

函数从其秘密全局变量中读取参数，并使用预定义的与其逻辑局部变量对应的秘密全局变量。当函数完成时，它跳转到函数秘密的“返回地址变量”中保存的地址。

例如，假设您有这样的代码，用类似C语言编写：

```
int add_two_values(int a, int b)
{
    int c = a + b;
    return c;
}

void sample()
{
    int x = add_two_values(31415, 2718);
}

```

编译器会将其转换为类似于这样的内容：

```
int a2v_a;
int a2v_b;
int a2v_c;
void* a2v_retaddr;

int add_two_values()
{
    a2v_c = a2v_a + a2v_b;

    return_value_register = a2v_c;
    goto a2v_retaddr;
}

int sample_x;
void sample()
{
    a2v_a = 31415;
    a2v_b = 2718;
    a2v_retaddr = &resume;
    goto add_two_values;
resume:
    sample_x = return_value_register;
}

```

看看这个：我们做了一个没有栈的函数调用和返回！

现在，您可以通过优化ABI，例如将一些这些值通过寄存器而不是全局变量传递来优化。例如，大多数处理器都有一个特殊的“链接”寄存器和一个特殊的指令“链接分支”，它会自动将链接寄存器设置为“链接分支”指令之后的指令地址，而且也许您可以优化调用约定以将前两个参数传递给寄存器，结果是这样的：

```
int a2v_a;
int a2v_b;
int a2v_c;
void* a2v_retaddr;

int add_two_values()
{
    a2v_a = argument_register_1;
    a2v_b = argument_register_2;
    a2v_retaddr = link_register;

    a2v_c = a2v_a + a2v_b;

    return_value_register = a2v_c;
    goto a2v_retaddr;
}

int sample_x;
void sample()
{
    argument_register_1 = 31415;
    argument_register_2 = 2718;
    branch_with_link add_two_values;
    sample_x = return_value_register;
}

```

有一个小问题：你不能递归。

递归不起作用，因为递归调用会用递归调用的返回地址覆盖返回地址变量，并且当外部调用完成时，它会跳转到错误的位置。

当时的编程语言通过简单地将其声明为非法来解决这个问题：它们不支持递归。¹

**额外的闲聊**：一些编译器甚至更狡猾，使用了自修改的代码：特殊的返回地址变量实际上是函数末尾跳转指令的地址字段！

这偶尔不是一个狡猾的技巧，而是一个实际的必要性：[处理器可能也不支持间接跳转](https://en.wikipedia.org/wiki/MIX)！

在认识到子程序的实际价值后，相当多的处理器增加了一个子程序调用指令，通过将返回地址存储在子程序的第一个字，并从子程序的第二个字开始执行来工作。要从子程序返回，您执行通过子程序开始标签的间接跳转。（据我回忆，一些处理器将返回地址存储在子程序第一个指令之前的字里。）这是使用虚构的汇编语言时的情况：

```
add_two_values:
    nop                     ; return address goes here
    add   r1 = r1, r2       ; actual subroutine begins here
    jmp   @add_two_values   ; indirect jump to return address

sample:
    mov   r1 = 31415        ; first parameter
    mov   r2 = 2718         ; second parameter
    bsr   add_two_values    ; call subroutine
    st    sample_x = r1     ; save return value

```

当 CPU 执行 `bsr` 分支到子程序指令时，它将返回地址存储到 `add_two_values` 的第一个字（覆盖了牺牲的 `nop`），并开始执行下一条指令 `add r1 = r1, r2`。

¹ FORTRAN 最初甚至不支持子程序！那些功能是在 1958 年添加的。而且直到 1991 年，FORTRAN 对递归的支持才成为标准，即使那时，您也必须显式地将子程序声明为 `RECURSIVE`。
