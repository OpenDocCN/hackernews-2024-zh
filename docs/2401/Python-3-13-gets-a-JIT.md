<!--yml

分类：未分类

日期：2024-05-27 14:38:27

-->

# Python 3.13 支持 JIT

> 来源：[`tonybaloney.github.io/posts/python-gets-a-jit.html`](https://tonybaloney.github.io/posts/python-gets-a-jit.html)

大家新年快乐！在 2023 年 12 月底（确切地说是圣诞节），CPython 核心开发者[Brandt Bucher](https://github.com/brandtbucher)向 Python 3.13 分支提交了一个[小小的拉取请求](https://github.com/python/cpython/pull/113465)，添加了一个 JIT 编译器。

这个变化一旦被接受，将是自 Python 3.11（当时由 Brandt 与 Mark Shannon 提交）加入的[特化自适应解释器](https://peps.python.org/pep-0659/)以来对 CPython 解释器的最大改变之一。

在这篇博文中，我们将看一下这个 JIT，它是什么，它是如何工作的，以及它的好处是什么。

## 什么是 JIT？

JIT，即“即时”编译，是一种编译设计，它暗示着编译发生在代码首次运行时。这是一个非常广泛的术语，可能意味着许多不同的事情。我猜，从技术上讲，Python 编译器已经是 JIT，因为它将 Python 代码编译成字节码。

当人们说及 JIT 编译器时，他们*倾向于*指的是一个生成**机器码**的编译器。这与一个 Ahead of Time（AOT，即预先编译）编译器相对，比如 GNU C 编译器 GCC 或 Rust 编译器 rustc，后者只生成一次机器码，然后以二进制可执行文件的形式分发。

当你运行 Python 代码时，首先会将其编译成字节码。有很多关于这个过程的演讲和视频，所以我不想再重复了，但是关于 Python 字节码需要注意的重要一点是：

1.  它们对 CPU 没有意义，并且需要一个特殊的字节码解释器循环来执行。

1.  它们是高级别的，相当于成千上万条的机器指令。

1.  它们是类型不可知的。

1.  它们是跨平台的。

对于一个非常简单的 Python 函数 `f()`，它定义了一个变量 `a` 并赋值为 `1`：

```
def func():
    a = 1
    return a 
```

它编译为 5 条字节码指令，你可以通过运行 `dis.dis` 来查看：

```
>>> import dis
>>> dis.dis(func)
 34           0 RESUME                   0

 35           2 LOAD_CONST               1 (1)
              4 STORE_FAST               0 (a)

 36           6 LOAD_FAST                0 (a)
              8 RETURN_VALUE 
```

如果你想尝试更复杂的东西，我还有一个更交互式的反汇编器叫做[dissy](https://github.com/tonybaloney/dissy)。

对于这个函数，Python 3.11 编译成了指令 `LOAD_CONST`、`STORE_FAST`、`LOAD_CONST` 和 `RETURN_VALUE`。这些指令在函数被一个大循环写成的 C 中解释时被解释。

如果你用 Python 编写一个非常简陋的 Python 评估循环，等价于 C 中的循环，它会看起来像这样：

```
import dis

def interpret(func):
    stack = []
    variables = {}
    for instruction in dis.get_instructions(func):
        if instruction.opname == "LOAD_CONST":
            stack.append(instruction.argval)
        elif instruction.opname == "LOAD_FAST":
            stack.append(variables[instruction.argval])
        elif instruction.opname == "STORE_FAST":
            variables[instruction.argval] = stack.pop()
        elif instruction.opname == "RETURN_VALUE":
            return stack.pop()

def func():
    a = 1
    return a 
```

如果你把这个解释器赋予我们的测试函数，它会执行它们并打印结果：

```
print(interpret(func)) 
```

这个循环带有一个大的 switch/if-else 语句，是 CPython 解释器循环的等效版本，虽然简化了。CPython 是用 C 编写的，并且由 C 编译器编译。为了简单起见，我会在 Python 中构建这个示例。

对于我们的解释器，每次想要运行函数`func`时，它都必须循环遍历每个指令，并将字节码名称（称为操作码）与每个 if 语句进行比较。这种比较和循环本身都会增加执行的开销。如果你运行函数 10,000 次且字节码从不改变（因为它们是不可变的），那么这种开销就显得多余了。相比每次调用函数都要评估这个循环，生成一个序列代码会更有效率。

这就是 JIT 做的事情。有许多类型的 JIT 编译器。Numba 是 JIT。PyPy 有 JIT。Java 有很多 JIT。Pyston 和 Pyjion 是 JIT。

提议用于 Python 3.13 的 JIT 是复制和补丁 JIT。

## 什么是复制和补丁 JIT？

从来没有听说过复制和补丁 JIT？别担心，我也没有，大多数人也没有。它是[2021 年才最近提出的一个想法](https://dl.acm.org/doi/10.1145/3485513)，旨在作为动态语言运行时的快速算法。

我会尝试通过扩展我们的解释器循环并将其重写为 JIT 来解释什么是复制和补丁 JIT。以前，解释器循环执行了两件事，首先它解释了（查看字节码），然后它执行了（运行指令）。我们可以做的是将这些任务分开，并让解释器输出指令而不执行它们。

复制和补丁 JIT 是这样一个理念，你**复制**每个命令的指令并为该字节码参数填充空白（或**补丁**）。下面是一个重写的例子，我保持循环非常相似，但每次我附加一个代码字符串与要执行的 Python 代码：

```
def copy_and_patch_interpret(func):
    code = 'def f():\n'
    code += '  stack = []\n'
    code += '  variables = {}\n'
    for instruction in dis.get_instructions(func):
        if instruction.opname == "LOAD_CONST":
            code += f'  stack.append({instruction.argval})\n'
        elif instruction.opname == "LOAD_FAST":
            code += f'  stack.append(variables["{instruction.argval}"])\n'
        elif instruction.opname == "STORE_FAST":
            code += f'  variables["{instruction.argval}"] = stack.pop()\n'
        elif instruction.opname == "RETURN_VALUE":
            code += '  return stack.pop()\n'
    code += 'f()'
    return code 
```

原始函数的这个结果是：

```
def f():
  stack = []
  variables = {}
  stack.append(1)
  variables["a"] = stack.pop()
  stack.append(variables["a"])
  return stack.pop()
f() 
```

这次，代码是**顺序**的，不需要循环执行。我们可以存储结果字符串，然后随意运行它：

```
compiled_function = compile(copy_and_patch_interpret(func), filename="<string>", mode="exec")

print(exec(compiled_function))
print(exec(compiled_function))
print(exec(compiled_function)) 
```

那有什么意义呢？嗯，生成的代码做同样的事情，但应该运行得更快。我把这两个实现给了[rich bench](https://pypi.org/project/richbench/)，复制和补丁方法运行得更快*(不过请记住，Python 中的循环与 C 相比非常慢。)*

## 为什么要使用复制和补丁 JIT？

将每个字节码的指令写出来并修补值的技术与“完整”JIT 编译器相比有优缺点。完整的 JIT 编译器通常会将高级字节码如 `LOAD_FAST` 编译成 IL（中间语言）中的较低级别指令。因为每个 CPU 架构都有不同的指令和特性，编写一个直接将高级代码转换为机器代码并支持 32 位和 64 位 CPU 的编译器将是非常复杂的，还有苹果的 ARM 架构以及所有其他的 ARM 变种。相反，大多数 JIT 首先编译为类似于通用机器码的 IL。这些指令是诸如“PUSH 64 位整数”，“POP 64 位浮点数”，“将堆栈上的值相乘”的东西。然后 JIT 可以在运行时将 IL 编译成机器码，通过发出特定于 CPU 的指令并将它们存储在内存中以供以后执行（类似于我们在示例中所做的）。

一旦你拥有 IL，你就可以对代码进行各种有趣的优化，比如[常量传播](https://zh.wikipedia.org/wiki/%E5%B8%B8%E9%87%8F%E5%B9%B3%E6%8E%A8)和循环提升。你可以在[Pyjion 的在线编译器 UI](https://live.trypyjion.com)中看到一个例子。

“完整”JIT 的一个大缺点是编译一次成 IL，然后再次成机器码的过程**缓慢**。它不仅慢，而且占用内存。为了说明这一点，最近研究数据[“Python meets JIT compilers: A simple implementation and a comparative evaluation”](https://doi.org/10.1002/spe.3267)表明，像 GraalPy 和 Jython 这样基于 Java 的 Python JIT，启动时间可能比普通 CPython 长 100 倍，并且消耗多达额外的 1GB 的 RAM 进行编译。已经有了 Python 的完整 JIT 实现。

选择复制和打补丁是因为从字节码编译成机器码是一组“模板”，然后在运行时与正确的值一起拼接和打补丁。这意味着你平均的 Python 用户并不是在他们的 Python 运行时中运行这个复杂的 JIT 编译器架构。Python 自己编写 IL 和 JIT 也是不合理的，因为有许多可供使用的现成的，如 LLVM 和 ryuJIT。但是一个全 JIT 需要将这些捆绑到 Python 中并增加所有的额外开销。一个复制和打补丁的 JIT 只需要将 LLVM JIT 工具安装在编译 CPython 源码的机器上，对于大多数人来说，这意味着构建和打包 CPython 给 python.org 的 CI 的机器上。

## 那么这个 JIT 是如何工作的呢？

用于 Python 的复制-修补编译器通过扩展一些新的（老实说并不广为人知的）Python 3.13 API 来工作。这些更改使得可以在 CPython 中在运行时发现可插拔的优化器，并控制代码的执行方式。这个新的 JIT 是这种新架构的可选优化器。我认为一旦主要的错误被解决，它将成为未来版本的默认选项。

当你从源代码编译 CPython 时，你可以给 configure 脚本提供一个标志 `--enable-experimental-jit`。这将为 Python 字节码生成机器码模板。首先复制每个字节码的 C 代码，例如对于 LOAD_CONST，最简单的：

```
frame->instr_ptr = next_instr;
next_instr += 1;
INSTRUCTION_STATS(LOAD_CONST); // Not used unless compiled with instrumentation
PyObject *value;
value = GETITEM(FRAME_CO_CONSTS, oparg);
Py_INCREF(value);
stack_pointer[0] = value;
stack_pointer += 1;
DISPATCH(); 
```

这个字节码的指令首先由 C 编译器编译成一个小的共享库，然后存储为机器码。因为有一些在运行时通常确定的变量，比如 `oparg`，所以 C 代码被编译时这些参数被留作 `0`。然后有一个需要填充的 0 值列表，称为空位。对于 `LOAD_CONST`，有 2 个需要填充的空位，oparg 和下一条指令：

```
static const Hole _LOAD_CONST_code_holes[3] = {
    {0xd, HoleKind_X86_64_RELOC_UNSIGNED, HoleValue_OPARG, NULL, 0x0},
    {0x46, HoleKind_X86_64_RELOC_UNSIGNED, HoleValue_CONTINUE, NULL, 0x0},
}; 
```

所有的机器码都以字节序列的形式存储在文件 `jit_stencil.h` 中，该文件由新的构建阶段自动生成。反汇编的代码存储为每个字节码模板上方的注释，其中 `JIT_OPARG` 和 `JIT_CONTINUE` 是待填充的空位：

```
0000000000000000 <__JIT_ENTRY>:
pushq   %rbp
movq    %rsp, %rbp
movq    (%rdi), %rax
movq    0x28(%rax), %rax
movabsq $0x0, %rcx
000000000000000d:  X86_64_RELOC_UNSIGNED        __JIT_OPARG
movzwl  %cx, %ecx
movq    0x28(%rax,%rcx,8), %rax
movl    0xc(%rax), %ecx
incl    %ecx
je      0x3d <__JIT_ENTRY+0x3d>
movq    %gs:0x0, %r8
cmpq    (%rax), %r8
jne     0x37 <__JIT_ENTRY+0x37>
movl    %ecx, 0xc(%rax)
jmp     0x3d <__JIT_ENTRY+0x3d>
lock
addq    $0x4, 0x10(%rax)
movq    %rax, (%rsi)
addq    $0x8, %rsi
movabsq $0x0, %rax
0000000000000046:  X86_64_RELOC_UNSIGNED        __JIT_CONTINUE
popq    %rbp
jmpq    *%rax 
```

当新的 JIT 编译器启用时，将每个字节码的机器码指令复制到一个序列中，并用该字节码在代码对象中的参数替换每个模板的值。生成的机器码存储在内存中，然后每次运行 Python 函数时，该机器码直接执行。

如果你编译了[我的分支](https://github.com/brandtbucher/cpython/pull/32)，并尝试在这个[测试脚本](https://gist.github.com/tonybaloney/7e12e416ad69968e297547498f7bcde1)上运行，然后交给 Ada Pro 或 Hopper 进行反汇编，你就能看到 JITted 代码。目前，只有在函数包含`JUMP_BACKWARD`操作码（在`while`语句中使用）时才会使用 JIT，但这在未来会改变。

## 它更快吗？

初始基准测试显示了[2-9%的性能提升](https://github.com/python/cpython/pull/113465#issuecomment-1876225775)。你可能对这个数字感到失望，特别是因为这篇博文一直在谈论汇编和机器码，而没有比它更快的东西，对吧？嗯，记住，CPython 已经是用 C 编写的，而且已经被 C 编译器编译成了机器码。在大多数情况下，这个 JIT 将执行几乎与以前相同的机器码指令。

**然而**，把这个 JIT 当作一系列更大优化的基石来考虑。没有它，这些优化都是不可能的。对于这种变化要在一个开源项目中被接受、理解和维护，它需要从简单的开始。

## 未来是光明的，未来是 JIT 编译的

现有解释器提前编译的挑战在于很少有机会进行严重优化。Python 3.11 的自适应解释器是朝着正确方向迈出的一步，但为了让 Python 见到性能上的飞跃，还需要做出更大的努力。

我认为，虽然这个 JIT 的第一个版本还不会严重影响任何基准测试（至少目前还没有），但它为一些巨大的优化打开了大门，不仅仅是对标准基准套件中的玩具基准程序有利的优化。
