<!--yml

category: 未分类

date: 2024-05-27 14:49:23

-->

# 在 C 中实现作用域延迟 - 由 Sir Whinesalot

> 来源：[https://btmc.substack.com/p/implementing-scoped-defer-in-c](https://btmc.substack.com/p/implementing-scoped-defer-in-c)

*注：在阅读本文之前，你应该阅读我的文章[在 C 中实现生成器](https://btmc.substack.com/p/implementing-generators-yield-in)，因为我将在这里依赖类似的技术。*

在 C 中的错误处理和资源管理可能非常麻烦。例如，考虑一个需要将日志文件分割成三个独立文件（“info”、“warning” 和 “error”）的程序：

```
FILE* log_file = fopen("log.txt", "r");
if (!log_file) {
  return -1;
}

FILE* info_log_file = fopen("info_log.txt", "w");
if (!info_log_file) {
  fclose(log_file);
  return -1;
}

FILE* warning_log_file = fopen("warning_log.txt", "w");
if (!warning_log_file) {
  fclose(info_log_file);
  fclose(log_file);
  return -1;
}

FILE* error_log_file = fopen("error_log.txt", "w");
if (!error_log_file) {
  fclose(warning_log_file);
  fclose(info_log_file);
  fclose(log_file);
  return -1;
}

// ...
// code to actually read and write the files goes here.
// if any operation fails we also need to "cleanup" there.
// ...

fclose(error_log_file);
fclose(warning_log_file);
fclose(info_log_file);
fclose(log_file);
return 0;
```

到处复制粘贴 `fclose` 调用不仅很烦人，而且极易出错。上面是最*简单*的例子。想象一下，如果我们只想在原始文件中出现相应日志类型行时创建各种输出文件，或者仅在设置了某个配置选项时创建并写入其中一个文件。可能会在这些操作之间进行内存管理（malloc/free）。

在本文中，我将向你展示一个非常巧妙的宏技巧，使得在标准 C 中处理*资源*管理尽可能愉快。

大多数编程语言提供了内置的资源管理机制。

C++ 和 Rust 都支持[基于作用域的资源管理](https://en.wikipedia.org/wiki/Resource_acquisition_is_initialization)，其中每种类型可能都有一个相关的“析构函数”（在 Rust 中称为 `drop`），由编译器隐式地插入到存储资源的变量所在作用域的末尾。当作用域结束时，无论是自然结束、异常还是返回语句，资源都会被清理。

这种解决方案特别好，因为 API 的使用者无需担心任何问题，它只是工作而已，但需要将对象生命周期与作用域绑定。

像 C#、Java 和 Python 这样的语言有特殊的资源处理块：在 C# 中是 `using` 语句，在 Java 中是 `try-with-resource` 语句，在 Python 中是 `with` 语句，它们在作用域结束时隐式调用清理函数。在 C# 中还可以使 `using` 语句不引入新的作用域，而是将资源绑定到*当前*作用域。

不是调用析构函数，而是调用语言“清理协议”的特定方法：在 C# 中是 `Dispose()`，在 Java 中是 `close()`，在 Python 中是 `__exit__()`。

这种解决方案的缺点是 API 的使用者必须记住使用资源处理语句才能进行清理，但允许以非作用域导向的方式管理常规对象。

像 Go、Zig、Odin 和 C3 这样的语言支持 `defer` 语句，允许指定清理操作，而不是隐式调用某些标准清理协议方法或析构函数：

```
// Go
log_file, err := os.Open("log.txt")
if err != nil {
  return nil, err
}
defer log_file.Close()

info_log_file, err := os.Open("info_log.txt", "w")
if err != nil {
  return err
}
defer info_log_file.Close()

warning_log_file, err := os.Open("warning_log.txt", "w")
if err != nil {
  return err
}
defer warning_log_file.Close()

error_log_file, err := os.Open("error_log.txt", "w")
if err != nil {
  return err
}
defer error_log_file.Close()

// ...

return nil
```

这种解决方案的缺点是 API 的使用者不仅必须记住使用 defer，还必须知道调用哪个特定的函数。

* * *

需要注意的重要一点是，Go 的 defer 与其他提到的语言工作方式不同。其他语言的 defer 非常简单：可以理解为在作用域结束之前（包括如果由 return 语句退出）将清理代码“复制粘贴”进去。

Go 的 defer 更像是一个命令式的指令。它将要执行的代码块工作类似于闭包，捕获引用变量在 defer 语句执行时的值（而不是在清理时！）。它还会在函数结束时而不是作用域结束时运行清理代码，因此需要额外的存储空间。例如，在循环中执行 10 次 defer 语句将向“清理堆栈”添加 10 条清理指令。

Go 的 defer 风格效率较低，其行为可能会令人意外。

* * *

defer 的优点在于它完全明确，并且给用户最大的控制权。

完全明确？最大的控制？听起来像是 C 语言。已经有一些现有的 defer 实现用于 C，但我对它们都不满意：

+   [参考实现](https://gustedt.gitlabpages.inria.fr/defer/) 的 [N2895 defer 提案](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n2895.htm) 太过复杂，甚至需要额外的预处理器。它依赖于 GCC 扩展和 setjmp + longjmp。需要所有这些复杂性来支持没有人要求的功能，依我看。它是“Go 风格”的。

+   ceraii 的 [实现](https://github.com/seleznevae/ceraii/blob/master/src/ceraii.h) 也相当复杂（虽然不至于那么极端），并依赖于 setjmp + longjmp。它是“Go 风格”的。

+   moon-chilled 的 [Defer 宏](https://github.com/moon-chilled/Defer/blob/master/defer.h) 是最简单的，但也需要使用 GCC 扩展或者 setjmp + longjmp，以及每个函数开始时需要一个 32 元素缓冲区来存储 defer。它是“Go 风格”的。

我不想要这些，我想要一个像 Zig、Odin、C3 和 D 中那样简单高效的作用域 defer，但是要在纯标准 C 中实现。我想要 [N3199 提案](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n3199.htm) 的作用域 defer。

所以直到那被接受（即永远不会），现在是动手的时候了。

在 C 中进行资源管理的经典解决方案是（滥用）使用 goto 语句来稍微整理一下事情：

```
`int result = 0;
FILE* log_file = fopen("log.txt", "r");
if (!log_file) {
  result = -1;
  goto DEFER_0;
}

FILE* info_log_file = fopen("info_log.txt", "w");
if (!info_log_file) {
  result = -1;
  goto DEFER_1;
}

FILE* warning_log_file = fopen("warning_log.txt", "w");
if (!warning_log_file) {
  result = -1;
  goto DEFER_2;
}

FILE* error_log_file = fopen("error_log.txt", "w");
if (!error_log_file) {
  result = -1;
  goto DEFER_3;
}

// ...

DEFER_4:
  fclose(error_log_file);
DEFER_3:
  fclose(warning_log_file);
DEFER_2:
  fclose(info_log_file);
DEFER_1:
  fclose(log_file);
DEFER_0:
  return result;`
```

我们避免了复制粘贴问题，但这也有其自身的问题。资源清理必须明确地以后进先出的顺序写在函数末尾。现在不再使用 return，而是需要使用 goto 来跳转到正确的清理点。当某些资源在内部作用域内分配和释放时，事情变得更加复杂。这种技术也部分地导致了 [goto fail](https://www.imperialviolet.org/2014/02/22/applebug.html)。

现在只有一个资源清理调用，并且能够跳转，这打开了一些选择。让我们稍微重新组织一下代码：

```
`int result = 0;
FILE* log_file = fopen("log.txt", "r");
if (!log_file) {
  result = -1;
  goto DEFER_0;
}
if (0) {
  DEFER_1:
    fclose(log_file);
    goto DEFER_0;
}

FILE* info_log_file = fopen("info_log.txt", "w");
if (!info_log_file) {
  result = -1;
  goto DEFER_1;
}
if (0) {
  DEFER_2:
    fclose(info_log_file);
    goto DEFER_1;
}

FILE* warning_log_file = fopen("warning_log.txt", "w");
if (!warning_log_file) {
  result = -1;
  goto DEFER_2;
}
if (0) {
  DEFER_3:
    fclose(warning_log_file);
    goto DEFER_2;
}

FILE* error_log_file = fopen("error_log.txt", "w");
if (!error_log_file) {
  result = -1;
  goto DEFER_3;
}
if (0) {
  DEFER_4:
    fclose(error_log_file);
    goto DEFER_3;
}

goto DEFER_4;
DEFER_0:
  return result;`
```

天哪，这看起来甚至更糟！但这是一个重要的改变，现在清理代码就在初始化代码旁边，就在 `defer` 应该在的地方。

让我们尝试制作一些宏：

```
`#define CAT(A, B) A##B
#define defer_return(I, V) do {goto CAT(DEFER_, I)} while(0)
#define defer(I, F, P) if(0){CAT(DEFER_, I): F; goto CAT(DEFER_, P)}
#define run_deferred(I) goto CAT(DEFER_, I); DEFER_0: return result;

int result = 0;
FILE* log_file = fopen("log.txt", "r");
if (!log_file) {
  defer_return(0, -1);
}
defer(1, fclose(log_file), 0);

FILE* info_log_file = fopen("info_log.txt", "w");
if (!info_log_file) {
  defer_return(1, -1);
}
defer(2, fclose(info_log_file), 1);

FILE* warning_log_file = fopen("warning_log.txt", "w");
if (!warning_log_file) {
  defer_return(2, -1);
}
defer(3, fclose(warning_log_file), 2);

FILE* error_log_file = fopen("error_log.txt", "w");
if (!error_log_file) {
  defer_return(3, -1);
}
defer(4, fclose(error_log_file), 3);

run_deferred();`
```

更好了，但还不够好。像这样手动计数 `defer` 真的很烦人。我们需要更高级的技巧。

在 C 预处理器中没有好方法来“计数”以避免上述问题。我们最接近的东西是 `__LINE__` 宏，它存储当前行，但没有办法引用我们延迟的“上一行”。

我们不再试图在预处理器级别完成所有工作，而是将部分工作移到运行时。让我们重新设计上面的示例，使用类似于 Duff's 设备的 `switch` 语句而不是直接使用 `goto`：

```
`#define defer_return do {goto _d_start;} while(0)
#define defer(I, F) {_d=I; if(0){case I: _d--; F; goto _d_start;}}
#define defer_block int _d = -1; _d_start: switch(_d){case -1:
#define run_deferred() goto _d_start;}

int result = 0;
defer_block {
  FILE* log_file = fopen("log.txt", "r");
  if (!log_file) {
    result = -1; defer_return;
  }
  defer(1, fclose(log_file));

  FILE* info_log_file = fopen("info_log.txt", "w");
  if (!info_log_file) {
    result = -1; defer_return;
  }
  defer(2, fclose(info_log_file));

  FILE* warning_log_file = fopen("warning_log.txt", "w");
  if (!warning_log_file) {
    result = -1; defer_return;
  }
  defer(3, fclose(warning_log_file));

  FILE* error_log_file = fopen("error_log.txt", "w");
  if (!error_log_file) {
    result = -1; defer_return;
  }
  defer(4, fclose(error_log_file));
} run_deferred();
return result;`
```

现在我们只需要在 `defer` 中进行 1 次计数就可以了。基本思路是每次调用 `defer` 都将 `defer` 计数（`_d`）设为 I。它还添加了一个 `if(0)` 块，该块在其主体中具有相应的 `switch case` 标签。在主体内部执行操作，递减 `_d`，并再次跳转到 `switch`。

因此，一旦第一次跳回到 `switch` 发生，`defer` 块将按照相反的顺序执行，直到 `_d` 变为 0，这不是一个有效的情况，`switch` 将停止。

语法：

```
defer_block {...} run_deferred();
```

的模型是：

```
do {...} while(...);
```

我个人认为这样做很容易理解。但这还不够好！如果用户在 `defer` 计数中写入错误的值，一切都会乱套。我们还缺少适当作用域 `defer` 的内部支持。

我们不再通过增加和减少来跟踪当前的 `defer`，而是通过存储它们的 `__LINE__` 来跟踪 `defer` 发生的位置，并通过名为 `_d_p` 的辅助变量跟踪上一个 `defer`：

```
#define defer(F) {int _d_p = _d; \ 
  _d = __LINE__; if(0){case __LINE__: _d = _d_p; F; goto _d_start;}}
```

`_d_p`（上一个 `defer`）变量声明在一个新的作用域内，以遮蔽任何先前使用的变量。它记住 `_d` 在设置为当前行之前的最后一个值。现在我们可以使用行作为 `case` 常量，避免显式的 `defer` 计数。而不是递减，我们通过 `_d_p` 将 `_d` 设置回其先前的值。我们需要确保的最后一件事是，当第一个 `_d_p` 被设置时，它指向“无处”，这样 `switch` 就会结束，我们通过在 `switch` 开始后立即将 `_d` 设置为一个不可能的行来实现这一点：

```
#define defer_block int _d = 0; \ 
  _d_start: switch(_d){case 0: _d = -1;
```

现在我们已经让单层 `defer` 工作了，而不需要手动计数。

我们想要支持的最后一件事是嵌套作用域，您可以在特定作用域中执行一部分 `defer` 而无需运行整个代码。我们还需要使 `defer_return` 能够运行当前作用域到顶级 `defer` 块的每个 `defer`。

首先要做的是引入一种方法来打开新的 `defer` 作用域。我们将使用到目前为止最复杂的技巧来实现这一点：

```
`#define _DEFER_SNAPSHOT(A) {int _d_p = _d; \ 
  _d = __LINE__; if(0){case __LINE__: _d = _d_p; A;}}
#define defer(F) _DEFER_SNAPSHOT(F; goto _d_start)
#define defer_scope
  do { _DEFER_SNAPSHOT(if(_d_r){goto _d_start;} else {continue;})`
```

我们将延迟的逻辑提取到一个`_DEFER_SNAPSHOT`宏中，该宏与`defer_scope`共享。其思想是，`defer_scope`的行为类似于defer，它创建一个“跳转点”，但不是执行动作，而是根据新的`_d_r`（延迟返回）变量做出决定：

+   如果`_d_r`为true，则继续“展开”过程。

+   如果`_d_r`为false，则通过`continue`跳出其`do {} while(0)`循环。

此变量由更新的`defer_return`设置。我们还添加了`defer_break`的替代方案，用于仅结束当前作用域：

```
`#define defer_break { goto _d_start; }
#define defer_return { _d_r = 1; goto _d_start; }`
```

我们对`run_deferred`进行了一点改动，使其适用于顶层延迟块和内部延迟作用域：

```
`#define run_deferred() goto _d_start; } while (0)`
```

对于内部作用域，`while(0)`结束`do`循环。对于顶层块，它只是一个额外的`while(0)`，什么也不做。

最后，我们需要在开始defer_block时初始化`_d_r`变量：

```
`#define defer_block int _d_r = 0; int _d = 0; \ 
  _d_start: switch(_d){case 0: _d = -1;`
```

至此，我们完成了作用域延迟：

```
`int result = 0;
defer_block {
  FILE* log_file = fopen("log.txt", "r");
  if (!log_file) {
    result = -1; defer_return;
  }
  defer(fclose(log_file));

  defer_scope {
    FILE* info_log_file = fopen("info_log.txt", "w");
    if (!info_log_file) {
      result = -1; defer_return;
    }
    defer(fclose(info_log_file));

    FILE* warning_log_file = fopen("warning_log.txt", "w");
    if (!warning_log_file) {
      result = -1; defer_return;
    }
    defer(fclose(warning_log_file));

  } run_deferred();

  FILE* error_log_file = fopen("error_log.txt", "w");
  if (!error_log_file) {
    result = -1; defer_return;
  }
  defer(fclose(error_log_file));
} run_deferred();
return result;`
```

与我用来在C中实现生成器的小宏不同，这个有点…过于了。但它比基于longjmp的方法简单得多，效率也更高，所以如果你想在C中使用延迟，请试试这个方法。

如果你可以接受使用GCC（或clang），那么[__attribute__((cleanup))](https://gcc.gnu.org/onlinedocs/gcc/Common-Variable-Attributes.html#index-cleanup-variable-attribute)就是最佳选择，它非常有效，即使它只允许调用具有特定原型的函数，而不是任意代码块。
