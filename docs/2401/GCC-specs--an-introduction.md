<!--yml

类别：未分类

日期：2024-05-27 14:28:59

-->

# GCC规格：简介

> 来源：[https://wozniak.ca/blog/2024/01/02/1/index.html](https://wozniak.ca/blog/2024/01/02/1/index.html)

您是否曾经将`-v`传递给驱动程序并研究其输出？ 其中有很多内容是具有启示性的。 这也是了解规格及其存在原因的好方法。

因为`-v`的输出非常广泛，我将使用一个示例，分成几个部分。 并非所有内容都与理解规格有关，但对于某些人可能是一般性的兴趣。

要获得输出，我在运行Debian 12的amd64机器上运行了以下命令，该机器正在使用GCC 12.2。

```
gcc -v t.c

```

源文件的内容并不重要，只要程序在单个文件中即可。（如果你没有准备好的话，请使用 `int main (void) {}`。）我已根据需要重新格式化输出。 在某些情况下，我省略了输出，并用`(...)`表示。

```
Using built-in specs.
COLLECT_GCC=gcc
COLLECT_LTO_WRAPPER=/usr/lib/gcc/x86_64-linux-gnu/12/lto-wrapper
OFFLOAD_TARGET_NAMES=nvptx-none:amdgcn-amdhsa
OFFLOAD_TARGET_DEFAULT=1
Target: x86_64-linux-gnu
Configured with: ../src/configure -v (...)
Thread model: posix
Supported LTO compression algorithms: zlib zstd
gcc version 12.2.0 (Debian 12.2.0-14) 
COLLECT_GCC_OPTIONS='-v' '-mtune=generic' '-march=x86-64' '-dumpdir' 'a-'

```

第一行说明了驱动程序从哪里获取其规格。 虽然可以覆盖内置规格，但这是以后讨论的问题。

接下来是一些由`gcc`设置的环境变量，用于与子程序和各种其他信息通信。 还有一行“配置为”告诉您GCC是如何配置的，如果您想自己构建它，则非常有用。

在我们的示例中，驱动程序必须使用C编译器编译源代码，将其组装成对象文件，然后将对象链接到可执行文件中。 第一个子程序运行是编译器。 `cc1`是C编译器，`cc1plus`是C++编译器。 请注意，输出中的子程序调用是以以单个空格开头的行开始的。

```
 /usr/lib/gcc/x86_64-linux-gnu/12/cc1 \
   -quiet \
   -v \
   -imultiarch x86_64-linux-gnu \
   t.c \
   -quiet \
   -dumpdir a- \
   -dumpbase t.c \
   -dumpbase-ext .c \
   -mtune=generic \
   -march=x86-64 \
   -version \
   -fasynchronous-unwind-tables \
   -o /tmp/ccJOdUuR.s
(...)

```

我们只向驱动程序提供了参数`-v`和`t.c`，但编译器有更多选项。 有些甚至提供了两次。 如果您查看[文档](https://gcc.gnu.org/onlinedocs/gcc-13.2.0/gcc/Invoking-GCC.html)中的选项，例如，您不会找到列出“dump”选项，因此编译器不使用与驱动程序相同的选项。 也就是说，其中一些匹配。 `-fasynchronous-unwind-tables`是一个代码生成选项，如果目标机器支持，则在调试时非常有用。 `-march`和`-mtune`控制为处理器生成什么样的代码。 这些是特定于目标或主机的选项。 每次都指定它们将是很烦人的。

我已经删除了所有编译器输出，因为它主要是版本信息，尽管它确实会打印出它正在使用的C标准，一些编译器的启发式数据以及编译器可执行文件的校验和。 它还会打印标题的搜索顺序，这在调试标题问题或仅知道系统标题存放在何处时非常方便。

作为一个侧面说明，值得指出的是，如果你愿意，你可以直接运行编译器。你甚至可以传递`--help`来查看它接受的广泛的选项集。直接运行编译器并不是你经常需要做的事情，即使在调试时也是如此，因为`gcc`有一个[-wrapper](https://gcc.gnu.org/onlinedocs/gcc-13.2.0/gcc/Overall-Options.html#index-wrapper)选项可以为您执行此操作。

编译器输出汇编源代码，所以下一步是驱动程序运行汇编器。

```
 as -v --64 -o /tmp/ccj7Fe2p.o /tmp/ccJOdUuR.s
(...)

```

注意，输入是一个临时文件，输出也是如此。驱动程序必须管理所有这些临时文件，并在子程序之间协调它们。如果传递了`-save-temps`，则会改变这样做的方式，并使用更直观的名称。

最后，我们有链接器的调用，包含最混乱和最复杂的一系列参数。

```
/usr/lib/gcc/x86_64-linux-gnu/12/collect2 \
   -plugin /usr/lib/gcc/x86_64-linux-gnu/12/liblto_plugin.so \
   -plugin-opt=/usr/lib/gcc/x86_64-linux-gnu/12/lto-wrapper \
   -plugin-opt=-fresolution=/tmp/cc7cMZRs.res \
   -plugin-opt=-pass-through=-lgcc \
   -plugin-opt=-pass-through=-lgcc_s \
   -plugin-opt=-pass-through=-lc \
   -plugin-opt=-pass-through=-lgcc \
   -plugin-opt=-pass-through=-lgcc_s \
   --build-id --eh-frame-hdr -m elf_x86_64 \
   --hash-style=gnu --as-needed \
   -dynamic-linker /lib64/ld-linux-x86-64.so.2 -pie \
   /usr/lib/gcc/x86_64-linux-gnu/12/../../../x86_64-linux-gnu/Scrt1.o \
   /usr/lib/gcc/x86_64-linux-gnu/12/../../../x86_64-linux-gnu/crti.o \
   /usr/lib/gcc/x86_64-linux-gnu/12/crtbeginS.o \
   -L/usr/lib/gcc/x86_64-linux-gnu/12 \
   -L/usr/lib/gcc/x86_64-linux-gnu/12/../../../x86_64-linux-gnu \
   -L/usr/lib/gcc/x86_64-linux-gnu/12/../../../../lib \
   -L/lib/x86_64-linux-gnu \
   -L/lib/../lib \
   -L/usr/lib/x86_64-linux-gnu \
   -L/usr/lib/../lib \
   -L/usr/lib/gcc/x86_64-linux-gnu/12/../../.. \
   /tmp/ccj7Fe2p.o \
   -lgcc \
   --push-state --as-needed -lgcc_s --pop-state \
   -lc -lgcc \
   --push-state --as-needed -lgcc_s --pop-state \
   /usr/lib/gcc/x86_64-linux-gnu/12/crtendS.o \
   /usr/lib/gcc/x86_64-linux-gnu/12/../../../x86_64-linux-gnu/crtn.o

```

选项的含义超出了本文的范围，但值得注意的是需要哪些参数。有目标文件（`.o`后缀）和库（`-l`选项）以及库搜索路径（`-L`选项）。这些选项的顺序也很重要，驱动程序必须正确处理。此外，要使用哪些系统目标文件（在本例中位于`/usr/lib`中）可能并不明显，因此我们应该感谢驱动程序在幕后所做的所有工作。手动指定链接是繁琐的。

为了完成对驱动程序必须管理的文件的讨论，编译/汇编/链接序列期间创建的所有临时文件在驱动程序退出之前都会被删除。

所有传递给子程序的参数和文件管理都是通过规范完成的。
