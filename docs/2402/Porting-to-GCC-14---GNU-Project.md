<!--yml

类别：未分类

日期：2024-05-27 15:00:10

-->

# 移植到GCC 14 - GNU项目

> 来源：[https://gcc.gnu.org/gcc-14/porting_to.html#c](https://gcc.gnu.org/gcc-14/porting_to.html#c)

# 移植到GCC 14

GCC 14发布系列与之前的GCC版本在[多方面](changes.html)有所不同。其中一些是由于bug修复，一些旧行为已经有意更改以支持新标准，或者以符合标准的方式放宽以促进编译或运行时性能。

其中一些变更是用户可见的，可能在移植到GCC 14时引起困扰。本文档旨在识别常见问题并提供解决方案。如果您有改进建议，请告知我们！

## C语言问题

### 现在某些警告已经成为错误

> 函数原型化首先用于帮助强制执行对函数传递的参数类型的类型检查，以及检查函数返回类型的赋值兼容性。标准C：ANSI草案成熟。《PC Magazine》，1988年9月13日，第117页。

初始ISO C标准及其1999年修订版删除了许多被广泛认为是应用程序bug源的C语言特性的支持，因为这些特性可能会由于意外的误用而导致bug。为了向后兼容性，GCC 13及更早版本仅将这些特性的使用诊断为警告。尽管这些警告已经默认启用了多个版本，但经验表明，这些警告很容易被忽视，导致难以诊断的bug。在GCC 14中，这些问题现在作为错误报告，不创建输出文件，为程序员提供更清晰的反馈，表明出现了问题。

GNU项目的大多数组件已经为兼容性修复过，但并非所有这些组件都在修复后发布过版本。从其上游版本更新这些源文件可能需要更新捆绑了[gnulib](https://www.gnu.org/software/gnulib/)或[autoconf-archive](https://www.gnu.org/software/autoconf-archive/)的程序。

几个GNU/Linux发行版从其上游邮件列表和错误跟踪器上的移植工作中共享了补丁，但当然，今天许多展示这些历史C兼容性问题的程序已经停止维护。

#### 隐式`int`类型（`-Werror=implicit-int`）

在大多数情况下，简单地添加缺失的`int`关键字即可解决错误。例如，像标志变量一样

```
  static initialized;

```

变为：

```
  static  initialized;

```

如果函数的返回类型被省略，应添加的正确类型可以是`int`或`void`，这取决于函数是否返回值。

在某些情况下，先前暗示的`int`类型是不正确的。这主要发生在旧式函数定义中，当参数类型未在参数列表外声明时。使用正确的类型可能需要避免整数转换错误（见下文）。在下面的示例中，`s`的类型应为`const char *`，而不是`int`：

```
  write_string (fd, s)
  {
    write (fd, s, strlen (s));
  }

```

修正后的标准 C 源代码可能如下所示（仍忽略错误处理和短写）：

```

  write_string ( fd, s)
  {
    write (fd, s, strlen (s));
  }

```

#### 隐式函数声明（`-Werror=implicit-function-declaration`）

现在不再可能调用未声明的函数。通常的解决方案是包含一个带有适当函数原型的头文件。请注意，GCC 将根据函数原型执行进一步的类型检查，这可以揭示需要额外更改的进一步类型错误。

对于在标准头文件中声明的知名函数，GCC 提供了带有适当 `#include` 指令的修复提示：

```
error: implicit declaration of function ‘strlen’ [-Wimplicit-function-declaration]
    5 |   return strlen (s);
      |          ^~~~~~
note: include ‘<string.h>’ or provide a declaration of ‘strlen’
  +++ |+#include <string.h>
    1 |

```

在 GNU 系统上，标准描述的头文件（如 C 标准或 POSIX）可能要求在编译开始时定义某些宏，然后才能提供所需的函数声明。详见 GNU C 库手册中的 [Feature Test Macros](https://sourceware.org/glibc/manual/latest/html_node/Feature-Test-Macros.html)。

一些程序使用 `-std=c11` 或类似的 `-std` 选项构建，不包含字符串 `gnu`，但仍然在标准 C 头文件（如 `<stdio.h>`）中使用 POSIX 或其他扩展。GNU C 库在标准 C 模式下会自动禁止这些扩展，这可能导致隐式函数声明。为了解决这个问题，可以放弃 `-std=c11` 选项，改用 `-std=gnu11`，或者使用 `-std=c11 -D_DEFAULT_SOURCE` 重新启用常见扩展。另外，使用 Autoconf 的项目可以启用 `AC_USE_SYSTEM_EXTENSIONS`。

如果在同一项目中调用未声明的函数且尚未存在适当的共享头文件，则应向被调用者和包含函数定义的源文件都包含的头文件中添加声明。这样可以确保 GCC 检查原型与函数定义的匹配情况。GCC 在使用 `-flto -Werror=lto-type-mismatch` 等选项进行构建时可以跨翻译单元执行类型检查，这有助于添加适当的原型。

在一些罕见情况下，添加原型可能会以不适当的方式改变 ABI。在这些情况下，需要声明一个没有原型的函数：

```
  void no_prototype ();

```

然而，这是 C23 中即将改变含义的过时 C 特性（声明具有原型且不接受参数的函数，类似于 C++）。通常，可以使用应用默认参数推广的原型。

在 GNU 系统上构建库代码时，[可能会调用未定义的（而不仅仅是未声明的）函数](https://sourceware.org/binutils/docs-2.42/ld/Options.html#index---allow-shlib-undefined)，并且仍然可以运行库中的其他代码，特别是如果使用了 ELF 惰性绑定。仅执行未定义函数调用将导致惰性绑定错误和程序崩溃。

#### 函数原型中的拼写错误（`-Werror=declaration-missing-parameter-type`）

函数声明中拼写错误的类型名称在更多情况下被视为错误，会有新的`-Wdeclaration-missing-parameter-type`警告。下例中的第二行现在被视为错误（先前是一个不受特定`-W…`选项控制的警告）：

```
  typedef int *int_array;
  int first_element (intarray);

```

修正拼写错误的方法是：

```
  typedef int *int_array;
  int first_element ();

```

在此之后，GCC将对函数参数进行类型检查，可能需要进一步更改。（以前，函数声明被视为没有原型。）

#### 不正确使用`return`语句（`-Werror=return-mismatch`）

GCC不再接受在声明为返回`void`的函数中具有表达式的`return`语句，或者返回非`void`类型的函数中没有表达式的`return`语句。

要解决此问题，请删除不正确的表达式（或将其转换为具有副作用的语句表达式，直接在`return`语句之前），或者根据情况添加虚拟返回值。如果没有合适的虚拟返回值，可能需要进一步更改以实现适当的错误处理。

以前，这些不匹配被诊断为`-Wreturn-type`警告。此警告仍然存在，并且默认情况下不被视为错误。现在它涵盖了剩余的潜在正确性问题，例如函数的结束括号`}`达到但不返回`void`。

默认情况下，GCC仍接受从返回`void`类型的函数内返回表达式作为GNU扩展，该扩展与此领域的标准C++规则相匹配。

#### 将指针用作整数及其反向用法（`-Werror=int-conversion`）

GCC不再将整数类型和指针类型视为在赋值中等效（包括隐式函数参数和返回值的赋值），而是以类型错误的方式进行编译失败。

在解决这些`int`转换问题之前，解决缺少的`int`类型、隐式函数声明和不正确的`return`语句用法是有意义的。其中一些将由缺少类型而被视为`int`，以及隐式声明函数的默认`int`返回类型。

要解决剩余的`int`转换问题，请向适当的指针或整数类型添加强制转换。在GNU系统上，标准类型`intptr_t`和`uintptr_t`（定义在`<stdint.h>`中）总是足够大以存储所有指针值。如果需要通用指针类型，请考虑使用`void *`。

在某些情况下，传递整数变量的地址可能比传递变量值的强制转换更合适。

#### 指针类型的类型检查（`-Werror=incompatible-pointer-types`）

GCC不再允许将所有指针类型隐式转换为所有其他指针类型。此行为现在仅限于`void *`类型及其修饰变体。

要解决由此导致的编译错误，可以添加适当的转换，并考虑在更多地方使用`void *`（特别是对于在引入`void *`到C语言之前存在的旧程序）。

如果程序不仔细跟踪指针类型，则可能包含别名违规，因此请考虑使用`-fno-strict-aliasing`进行构建。（无论是手动编写转换还是由GCC自动执行，对严格别名违规都没有影响。）

不兼容的函数指针类型经常涉及回调函数，这些函数具有比分配给它们的函数指针更具体的参数类型（或者不太具体的返回类型）。例如，试图对字符串数组进行排序的旧代码可能如下所示：

```
#include <stddef.h>
#include <stdlib.h>
#include <string.h>

int
compare (char **a, char **b)
{
  return strcmp (*a, *b);
}

void
sort (char **array, size_t length)
{
  qsort (array, length, sizeof (*array), compare);
}

```

为了解决这个问题，回调函数应该用正确的类型定义，并且在使用之前需要适当地进行参数转换（因为通过不正确类型的函数指针调用函数是未定义的）：

```
int
compare (, )
{

  return strcmp (*a, *b);
}

```

对象指针类型可能会出现类似的问题。考虑一个声明为`void **`类型参数的函数，你想传递一个`int *`类型变量的地址：

```
extern int *pointer;
extern void operate (int command, void **);

operate (0, &pointer);

```

在这些情况下，可能需要使用正确的`void *`类型制作指针的副本：

```
extern int *pointer;
extern void operate (int command, void **);

operate (0, &pointer);

```

正如最初提到的，添加转换在这里不会消除`operate`函数实现中的任何严格别名违规。当然，在一般情况下，引入这样的额外副本可能会改变程序的行为。

有些编程风格依赖于相关对象指针类型之间的隐式转换来实现类似于C++风格的`struct`继承。可以通过`-fplan9-extensions`选项和派生`struct`中基类型的未命名初始字段来避免编写手动转换。

有些程序使用像`void (*) (void)`这样的具体函数指针类型作为通用函数指针类型（类似于对象指针类型的`void *`），并依赖于隐式转换到和从这种类型。这是因为C语言没有提供通用函数指针类型，并且标准C不允许函数指针类型和对象指针类型之间的转换。在大多数目标中，GCC支持`void *`和函数指针类型之间的隐式转换。然而，为了实现可移植的解决方案，需要使用具体的函数指针类型，并配合显式转换。

#### 对Autoconf和一般构建环境探测的影响

大多数[Autoconf](https://www.gnu.org/software/autoconf/)探测器检查构建系统功能的方式编写，如果某个功能缺失会触发编译器错误。新的错误可能导致GCC在之前工作正常的探测器上编译失败，与探测的实际目标无关。这些失败的探测器倾向于禁用程序功能及其测试，导致特性悄然丢失。

在这种情况下，可以使用 `diff` 比较生成的 `config.log`、`config.h` 和其他源代码文件，以确保没有意外的差异。

这一现象还影响了 CMake、Meson 和各种用于脚本语言的 C 扩展模块的构建工具中的类似过程。

Autoconf 自至少版本 2.69 起已支持 C99 编译器的通用核心探测。（特定功能探测可能已在更近版本中解决了问题。）2.69 之前的版本可能具有依赖于 C99 中已删除的 C 特性的通用探测（例如标准 C 支持的探测），因此在 GCC 14 下会失败。

#### 将错误转换为警告

无法移植到标准 C 的源代码可以使用 `-fpermissive`、`-std=gnu89` 或 `-std=c89` 进行编译。尽管名称如此，后两个选项也会打开对预标准 C 构造的支持。使用 `-fpermissive` 选项，程序可以使用 C99 内联语义和已从 C99 中删除的特性。或者，可以使用相关的 `-Wno-error=…` 选项将个别错误降级为警告，或使用 `-Wno-…` 完全禁用。例如，`-Wno-error=incompatible-pointer-types` 可关闭大多数指针赋值的类型检查。

一些构建系统不会将 `CFLAGS` 变量传递给所有构建部分，可能需要将 `CC` 设置为类似 `gcc -fpermissive` 的内容。如果构建系统不支持 `CC` 变量中的空格，则可能需要类似以下的包装脚本：

```
#!/bin/sh
exec /usr/bin/gcc -fpermissive "$@"

```

#### 适应 C 代码生成器

无法更新以生成有效标准 C 代码的 C 代码生成器可以发出 [`#pragma GCC diagnostic warning`](/onlinedocs/gcc-14.1.0/gcc/Diagnostic-Pragmas.html) 指令，将这些错误变回警告：

```
#if defined __GNUC__ && __GNUC__ >= 14
#pragma GCC diagnostic warning "-Wimplicit-function-declaration"
#pragma GCC diagnostic warning "-Wincompatible-pointer-types"
#pragma GCC diagnostic warning "-Wint-conversion"
#pragma GCC diagnostic warning "-Wreturn-mismatch"
#endif

```

不包括 `-Wimplicit-int` 和 `-Wdeclaration-missing-parameter-type`，因为它们在代码生成器中应该很容易解决。

#### 未来的方向

本节涉及与 C 标准及其他向后兼容性隐患相关的语言特性的潜在未来更改。这些计划可能会改变，并且仅在此提供指导，以优先考虑未来兼容性的源代码级变更。

目前尚不清楚 GCC 何时可以默认启用 C23 的 `bool` 关键字（使 `bool` 类型可用，而无需包含 `#include <stdbool.h>`）。许多程序定义自己的 `bool` 类型，有时与内置的 `_Bool` 类型大小不同。进一步的复杂性在于，即使大小相同，自定义的 `bool` 通常也没有陷阱表示，而 `_Bool` 和新的 `bool` 类型则有。这意味着在处理不受信任、不一定格式良好的输入数据时，可能会出现微妙的兼容性问题。

GCC 默认不太可能警告未声明为原型的函数声明。这意味着没有严格的理由来转换

```
void do_something ();

```

转变为

```
void do_something (void);

```

除了将多余的忽略参数诊断为错误外，未来版本的GCC可能会警告没有原型的函数调用，这些函数指定了这些多余的参数（例如 `do_something (1)`）。最终，GCC将因为这些在C23中的约束违反而将此类调用诊断为错误。

即使C23作为默认语言方言，GCC可能会继续支持旧式函数定义。

## C++ 语言问题

一些C++标准库头文件已更改为不再包括库内部使用的其他头文件。因此，没有正确包含正确头文件的C++程序将不再编译。

在使用GCC 14编译时，以下头文件在libstdc++中使用较少，可能需要显式包含：

+   `<algorithm>`（用于 `std::copy_n`、`std::find_if`、`std::lower_bound`、`std::remove`、`std::reverse`、`std::sort` 等）

+   `<cstdint>`（用于 `std::int8_t`、`std::int32_t` 等）

### Pragma `GCC target` 现在影响预处理器符号

pragma `GCC target` 的行为及其如何影响ISA宏在GCC 14中已发生变化。在使用集成预处理器进行编译时，`GCC target` pragma在C中定义和未定义相应的ISA宏，但在分开步骤调用预处理器或使用 `-save-temps` 选项时不会这样做。在C++中，ISA宏的定义方式并没有实际效果。

在GCC 14中，C++的行为类似于早期版本中具有集成预处理的C。此外，在两种语言中，ISA宏在预处理时与编译分开时定义和未定义的方式如预期那样。

这可能导致不同的行为，特别是在C++中。例如，下面的C++片段的一部分将在GCC 14中被（悄悄地）编译为错误的指令集。

```
  #if ! __AVX2__
  #pragma GCC push_options
  #pragma GCC target("avx2")
  #endif

  /* Code to be compiled for AVX2\. */

  /* With GCC 14, __AVX2__ here will always be defined and pop_options
  never invoked. */
  #if ! __AVX2__
  #pragma GCC pop_options
  #endif

  /* With GCC 14, all following functions will be compiled for AVX2
  which was not intended. */

```

在这种情况下的修复方法是记住是否需要在新的用户定义的宏中执行 `pop_options`。
