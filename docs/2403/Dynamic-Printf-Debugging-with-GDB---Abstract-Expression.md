<!--yml

类别：未分类

日期：2024-05-27 14:41:10

-->

# 使用 GDB 进行动态 Printf 调试 – 抽象表达

> 来源：[https://abstractexpr.com/2024/03/03/dynamic-printf-debugging-with-gdb/](https://abstractexpr.com/2024/03/03/dynamic-printf-debugging-with-gdb/)

当我们使用 `printf()` 调试程序时，每次添加新的 `printf()` 语句时都需要重新编译它！对吧！?

错了！

使用 gdb 我们可以在不重新编译程序的情况下添加 `printf()` 语句，甚至可以在其运行时添加它们。

## 使用 GDB 添加 Printfs

假设我们有以下计算数组所有元素和的示例程序：

```
#include <stdio.h>

int main(int argc, char **argv)
{
    int numbers[] = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10};
    int n = sizeof(numbers) / sizeof(numbers[0]);
    int sum = 0;

    for (int i = 0; i < n; i++) {
        sum += numbers[i];
    }

    printf("Sum of array: %d\n", sum);

    return 0;
}

```

将此程序保存为 `sumarray.c` 并使用以下命令编译它：

```
$ gcc -ggdb -o sumarray sumarray.c

```

当我们运行它时，输出将如下所示：

```
$ ./sumarray
Sum of array: 55

```

如果我们想在 `for` 循环之前添加一个调试打印，打印出数组中的元素数量，并在循环内部添加一个调试打印，打印当前循环索引和当前数组元素的值，该怎么办？

由于我们已经使用调试信息编译了我们的程序，我们可以直接用 gdb 打开它：

如果您想在 gdb 中查看源代码以进行定位，只需使用 `list` 命令。或者，您可以使用 `enable tui` 命令获取一个半图形界面，您可以在其中始终看到源代码。

我们在第 8 行添加我们的第一个动态 `printf`，以打印出数组中元素的总数：

```
(gdb) dprintf sumarray.c:8, "Num elements: %d\n", n

```

语法的第一部分类似于在 gdb 中添加断点。之后是一个逗号和一个 C 格式字符串，然后是格式字符串的参数。当然，我们可以一次打印多个变量，就像在添加第二个动态打印时看到的那样：

```
(gdb) dprintf sumarray.c:10, "Index: %d; value: %d\n", i, numbers[i]

```

调试器为我们的程序的每个动态 `printf` 添加了断点。但是，这些断点不会中断程序并将我们放入调试器，而是执行带有给定格式字符串和参数的 `printf()`，然后继续执行程序。

现在我们可以在 gdb 中运行我们的程序：

输出现在应该类似于这样：

```
Num elements: 10
Index: 0; value: 1
Index: 1; value: 2
Index: 2; value: 3
Index: 3; value: 4
Index: 4; value: 5
Index: 5; value: 6
Index: 6; value: 7
Index: 7; value: 8
Index: 8; value: 9
Index: 9; value: 10
Sum of array: 55
[Inferior 1 (process 8368) exited normally]
(gdb)

```

如果我们使用 `info break` 命令列出所有断点，我们将看到 gdb 实际上将我们的动态 `printf` 表示为断点：

```
(gdb) info break
Num     Type           Disp Enb Address            What
1       dprintf        keep y   0x00005555555551df in main at sumarray.c:9
	breakpoint already hit 1 time
        printf "Num elements: %d\n", n
2       dprintf        keep y   0x00005555555551e8 in main at sumarray.c:10
	breakpoint already hit 10 times
        printf "Index: %d; value: %d\n", i, numbers[i]
(gdb)

```

要删除动态 `printf`，您可以使用 `delete` 命令，后跟断点号（例如 `delete 2` 删除第二个动态打印）。

## 条件动态 Printf

当代码运行频繁时（例如在循环中），存在这样的情况：您不希望 `printf()` 调用每次都执行，而只有当变量具有特定值或满足某些其他特殊条件时才执行。否则，输出太多，很难在所有无关的调试输出中找到相关行。

当您在源代码中添加 `printf()` 调用时，您可以将 `printf()` 放在 `if` 块中，这样问题就解决了。

使用 `dprintf`，首先添加动态打印。然后，通过使用 gdb 命令 `condition`，将生成的断点设为条件断点。

在我们的最后一个示例中，gdb 为第二个动态 printf 语句（在循环中打印索引和值的语句）创建了断点号 2。如果我们希望仅在偶数循环索引时打印其输出，我们可以使用以下命令实现：

```
(gdb) condition 2 (i % 2 == 0)

```

通过这个修改，我们的程序在 gdb 中的输出现在会像这样：

```
(gdb) run
Num elements: 10
Index: 0; value: 1
Index: 2; value: 3
Index: 4; value: 5
Index: 6; value: 7
Index: 8; value: 9
Sum of array: 55
[Inferior 1 (process 5896) exited normally]
(gdb) 

```

## 保存和加载动态 Printfs

由于动态 printf 是断点，我们可以像保存其他断点一样保存它们，使用 *save breakpoints* 命令：

```
(gdb) save breakpoints [filename]

```

要加载它们，我们可以使用 *source* 命令：

## 打印被丢弃的函数返回值

有时您想打印函数的返回值。如果在调用它时函数的返回值存储在变量中，则很容易实现。但是，如果调用它的代码忽略了返回值，该怎么办呢？

让我们考虑这个程序：

```
#include <stdio.h>

int func()
{
    printf("func called!");
    return 42;
}

int main(int argc, char **argv)
{
    func();

    return 0;
}

```

这里函数 `func()` 被调用并返回一个整数值。但是调用它的代码忽略了这个事实，并且丢弃了返回值。

将其保存到名为 `return.c` 的文件中，并用以下命令编译它：

```
$ gcc -ggdb -o return return.c

```

如果我们在 x86-64 机器上，并且我们知道该架构的调用约定要求整数和指针类型的返回值存储在 `RAX` 寄存器中，那么这个问题很容易解决。我们只需在函数调用后的行后放置一个 `dprintf`，并打印 `RAX` 寄存器的值：

```
(gdb) dprintf return.c:12, "return value: %d\n", $rax

```

在 ARM64（也称为 AArch64）上，我们必须打印 `x0` 寄存器：

```
(gdb) dprintf return.c:12, "return value: %d\n", $x0

```

当我们运行程序时，

输出将类似于以下内容：

```
return value: 42
func called![Inferior 1 (process 36549) exited normally
```

## 自定义 Printf 函数

不使用 `printf()` 来输出我们的调试消息，gdb 也允许我们配置我们自己的自定义函数，以便在我们的代码中添加每个 `dprintf` 时调用它。

这样的自定义函数可能如下所示：

```
int log_msg(const char *format, ...)
{
    int n;
    va_list args;
    // init the variable length argument list
    va_start(args, format);

    n = vfprintf(logfile, format, args);

    // cleanup the variable length argument list
    va_end(args);

    return n;
}

```

此函数返回一个 `int`，并期望一个格式字符串和可变数量的附加参数。因此，它与 `printf()` 具有相同的原型。

它首先初始化变量参数列表，然后调用 `vfprintf()`（此函数是 `fprintf()` 的一种变体，接受以 `va_list` 变量形式的变量参数列表，以及存储在全局变量中的 `FILE` 指针和格式字符串）。

要使用此函数，我们需要在主函数开始时打开日志文件。我们不能在程序的任何点之前放置 `dprintf`，否则由于全局变量 `logfile` 仍为 `NULL`，我们会得到段错误。

我们修改后的示例程序的完整代码如下：

```
#include <stdio.h>
#include <stdarg.h>

FILE *logfile = NULL;

int log_msg(const char *format, ...)
{
    int n;
    va_list args;
    // init the variable length argument list
    va_start(args, format);

    n = vfprintf(logfile, format, args);

    // cleanup the variable length argument list
    va_end(args);

    return n;
}

int main(int argc, char **argv)
{
    int numbers[] = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10};
    int n = sizeof(numbers) / sizeof(numbers[0]);
    int sum = 0;

    const char *filename = "logfile.txt";
    logfile = fopen(filename, "w");
    if (logfile == NULL) {
        fprintf(stderr, "Failed to open logfile: %s\n", filename);
        return 1;
    }

    for (int i = 0; i < n; i++) {
        sum += numbers[i];
    }

    printf("Sum of array: %d\n", sum);
    fclose(logfile);

    return 0;
}

```

让我们将其保存到名为 `custom_printf.c` 的文件中，并用以下命令编译它：

```
$ gcc -ggdb -o custom_printf custom_printf.c

```

在使用 gdb 加载生成的 `custom_printf` 程序之后，我们首先必须将 `dprintf` 的调用风格设置为 `call`（否则，gdb 将使用内置的 printf 而不是在我们的程序中查找它），并将 `dprintf` 使用的函数设置为我们的 `log_msg` 函数：

```
(gdb) set dprintf-style call
(gdb) set dprintf-function log_msg

```

现在，我们可以为我们的 dprintf 设置新的行号：

```
(gdb) dprintf custom_printf.c:33, "Num elements: %d\n", n
(gdb) dprintf custom_printf.c:35, "Index: %d; value: %d\n", i, numbers[i]

```

一旦您执行程序时，

现在，`dprintfs`的输出应该出现在文件`logfile.txt`中，而不是在屏幕上显示。

## 结论

gdb 的动态 printf 功能是在调试过程中非常强大的替代方案，而不是在代码中添加一次性的 `printf()` 语句。你可以在不重新编译程序的情况下添加新的动态 printf，并且甚至可以在程序运行时添加额外的打印语句。

当调试长时间运行的程序（比如服务器）时，最后一个优势尤其有用。想象一下，你添加了一些初始的动态 printf。然后启动服务器并发送请求。随着你对代码的深入了解，你可以在调试器中舒适地添加更多动态 printf，并每次发送新的请求。分析输出后，你会继续添加更多动态 printf 直到找到 bug。

添加这个功能后，你可以将动态的`printf`保存到文件中，以便于后续的调试会话，还可以打印被丢弃的函数返回值，这样你就能理解为什么 gdb 将 printf 调试推向了新的高度。
