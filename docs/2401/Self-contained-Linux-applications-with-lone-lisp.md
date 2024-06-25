<!--yml

分类：未分类

日期：2024-05-27 15:03:56

-->

# 使用孤立的 Lisp 创建自包含的 Linux 应用程序

> 来源：[`www.matheusmoreira.com/articles/self-contained-lone-lisp-applications`](https://www.matheusmoreira.com/articles/self-contained-lone-lisp-applications)

# 使用孤立的 Lisp 创建自包含的 Linux 应用程序

我开始 [孤立 Lisp 项目](https://github.com/lone-lang/lone) 来创建一个专门为 Linux 设计的 Lisp 语言和环境。我在其中构建了对任意 Linux 系统调用 的支持，以便能够实现任何程序而无需任何外部依赖，并且以便程序员可以使用每个 Linux 特性。

虽然还远未准备好用于生产，但我在语言开发中已经取得了重要的里程碑：现在可以完全使用 Lisp 编写自包含、独立、脱离环境、可重新分发的 Linux 应用程序。

现在，Lisp 代码可以直接嵌入到 lone 解释器的副本中，然后可以复制到并在任何相同架构的 Linux 系统上运行，而无需修改或任何外部依赖。该应用程序仅受其使用的系统调用限制：较新的系统调用自然需要较新的内核。

在本文中，我将展示这个功能，解释它是如何工作的以及实现它的过程。我见过的每一个脚本捆绑工具都会将代码解压到某个文件系统位置，然后再读取回来。我想出了一种不同的解决方案。

## 在孤立的 Lisp 中的 `env` 的一个简单实现

`env` 实用程序是类 Unix 操作系统中最简单的程序之一。它最基本的功能是打印用户的环境。这个功能可以在孤立的 Lisp 中轻松实现：

```
(import (lone print) (linux environment))

(print environment) 
```

运行这个简单的程序会产生一个环境变量及其值的表格：

```
$ ./lone < env.ln
{ "HOME" "/home/matheus" "EDITOR" "vim" … } 
```

### 将 `env.ln` 嵌入到解释器中

为了将 `env.ln` 转换为一个自包含的 `env` 程序，必须将 lone Lisp 代码嵌入到解释器的副本中。这可以通过专用的 `lone-embed` 工具实现：

```
$ cp lone env
$ lone-embed env env.ln 
```

解释器将无缝地加载并执行代码：

```
$ ./env
{ "HOME" "/home/matheus" "EDITOR" "vim" … } 
```

## 精灵片段

使用 `strace` 跟踪应用程序的系统调用会揭示一个有趣的属性：

```
$ strace env
execve("env", ["env"], 0x7fe9752d40 /* 31 vars */) = 0
write(1, "{ ", 2)                                  = 2
write(1, "\"", 1)                                  = 1
write(1, "HOME", 4)                                = 4
write(1, "\"", 1)                                  = 1
write(1, " ", 1)                                   = 1
… 
```

它开始立即写入其输出。它没有必要从文件系统中读取。代码必须在其他地方。

```
$ xxd env | tail -n7
00032fe0: 0000 0000 0000 0000 0000 0000 0000 0000  ................
00032ff0: 0000 0000 0000 0000 0000 0000 0000 0000  ................
00033000: 7b20 7275 6e20 2830 202e 2036 3329 207d  { run (0 . 63) }
00033010: 0a28 696d 706f 7274 2028 6c6f 6e65 2070  .(import (lone p
00033020: 7269 6e74 2920 286c 696e 7578 2065 6e76  rint) (linux env
00033030: 6972 6f6e 6d65 6e74 2929 0a0a 2870 7269  ironment))..(pri
00033040: 6e74 2065 6e76 6972 6f6e 6d65 6e74 290a  nt environment). 
```

它就在可执行文件的最后，以对齐的偏移量。通过一些精灵魔法，解释器在运行时反射其自身的内容。它从内部加载代码并对其进行评估。而且完全不需要使用任何系统调用。

这种魔法的源头是 ELF 段。

```
$ readelf --segments env | grep 33000
LOAD           0x0000000000033000 0x0000000000312000 0x0000000000312000
LOOS+0xc6f6e65 0x0000000000033000 0x0000000000312000 0x0000000000312000 
```

ELF 段，也称为程序头，描述了程序的内存映像。有许多类型的段，但特别有趣的是`LOAD`段，它们告诉 Linux 文件的哪些部分必须映射到哪些地址，以使程序能够正确运行。

`lone-embed`工具将 Lisp 代码复制到 ELF 中，然后为其创建一个`LOAD`段。Linux 在解释器甚至启动之前，就在加载时自动映射嵌入的代码。

### 寻找段

代码将会在内存中，但是其位置和大小是未知的。它可能位于虚拟地址空间的任何地方。为了解决这个问题，Linux 提供了一个巧妙的解决方案：它告诉我们它在哪里。

除了参数和环境向量之外，进程还接收辅助向量。它实质上是一个以空值结尾的各种类型键值对的数组，并且它就位于环境向量之后的栈上。

```
struct lone_auxiliary_value {
    union {
        char *c_string;
        void *pointer;
        long integer;
        unsigned long unsigned_integer;
    } as;
};

struct lone_auxiliary_vector {
    long type;
    struct lone_auxiliary_value value;
} *auxv;

void **pointer = (void **) envp;
while (*pointer++ != 0);
auxv = (struct auxiliary_vector *) pointer; 
```

通过这种机制，Linux 将各种有用的信息传递给程序。其中包括处理器架构和功能、当前用户和组 ID、一些随机字节、[vDSO](https://www.man7.org/linux/man-pages/man7/vdso.7.html)的位置、系统的页面大小...

以及程序头表的位置。

```
struct lone_auxiliary_value lone_auxiliary_vector_value(struct lone_auxiliary_vector *auxiliary, long type)
{
    for (; auxiliary->type != AT_NULL; ++auxiliary)
        if (auxiliary->type == type)
            return auxiliary->value;

    return (struct lone_auxiliary_value) { .as.integer = 0 };
}

struct lone_elf_segments lone_auxiliary_vector_elf_segments(struct lone_auxiliary_vector *auxv)
{
    return (struct lone_elf_segments) {
        .entry_size  = lone_auxiliary_vector_value(auxv, AT_PHENT).as.unsigned_integer,
        .entry_count = lone_auxiliary_vector_value(auxv, AT_PHNUM).as.unsigned_integer,
        .segments    = lone_auxiliary_vector_value(auxv, AT_PHDR).as.pointer
    };
} 
```

该表只是一系列连续的[ELF 程序头结构](https://en.wikipedia.org/wiki/Executable_and_Linkable_Format#Program_header)。有了这个指针，程序只需要扫描表并找到正确的条目。然而，不是简单地搜索`LOAD`条目。尝试这样做会暴露出一些问题。

第一个问题是有很多这些可加载段，它们彼此之间都无法区分。ELF 节具有用于标识的唯一名称，程序头没有任何内容。

第二个问题是它们的对齐要求。地址和大小通常对齐到页面边界。这混淆了它们所包含数据的真实大小。

#### 单独段

为了解决这个问题，我创建了自己的自定义段类型：`LONE`段。

ELF 为操作系统特定段提供了非常慷慨的数字范围。`LOOS`到`HIOS`之间的所有值都可以由操作系统分配。这些定义表示介于`0x60000000`和`0x6FFFFFFF`之间的整数范围。那是 268,435,455 个魔术数。

所以我就挑了一个。这就是`LOOS+0xc6f6e65`的意思。按 ASCII 拼写了`lone`，它就自己解决了。

```
 #define PT_LONE 0x6c6f6e65 
```

我想，如果 GNU 能做到，那么我也能做到。

`LONE`段不可加载，因此没有任何对齐要求，允许它准确描述嵌入段。它还充当一个魔术数，使得在程序头表中轻松搜索它变得简单。一旦找到，它包含了加载和执行代码所需的全部信息。

```
typedef Elf64_Phdr lone_elf_segment; 
typedef Elf32_Phdr lone_elf_segment; 

lone_elf_segment *lone_auxiliary_vector_embedded_segment(struct lone_auxiliary_vector *auxv)
{
    struct lone_elf_segments table = lone_auxiliary_vector_elf_segments(auxv);

    for (size_t i = 0; i < table.entry_count; ++i) {
        lone_elf_segment *entry = &table.segments[i];

        if (entry->p_type == PT_LONE)
            return entry;
    }

    return 0;
}

lone_elf_segment *segment = lone_auxiliary_vector_embedded_segment(auxv);
struct lone_bytes data;
data.count = segment->p_memsz;
data.pointer = (unsigned char *) segment->p_vaddr; 
```

解释器现在已经嵌入了数据的地址和大小到其自己的可执行文件中。此时一切顺利进行。

唯一剩下的就是理解这些字节。我选择简单地在其中放置一个描述符对象，并让解释器读取它。看起来是最简单的解决方案。

```
{ run (0 . 63) } 
```

只是一个普通的哈希表，采用孤独的 lisp 语法。run 键包含解释器应该运行的代码的偏移量和大小，相对于描述符对象的末尾。它只是读取该片段并对其进行评估。

由于它只是一个普通的哈希表，所以可以使用任意模式进行无限扩展。我计划实现一个模块键，以包含任意数量的单模块，以便程序可以将其库静态链接到单解释器中。我所要做的就是将嵌入式段放入模块搜索路径中，放在其他所有段之前。我想我也可以允许通过描述符对象配置解释器，消除了对命令行开关的需求。

## 特殊的链接器功能

还记得我在文章开头简要提到的`lone-embed`工具吗？它是一个我专门为此目的构建的 ELF 修补工具，在这里承担了*大量的*繁重工作。

当程序被编译和链接时，程序头部被固定。然而，为了完成所有这些，我需要向程序头表追加新的段。这证明比预期的要*困难得多*。

程序头表通常位于 ELF 头部之后，文件内容之前。向其中追加新项目将需要调整表的大小，这将需要将其后的*所有*地址向上移动，使指向旧地址的指针失效。据我所知，如果没有重新发明链接器本身，这是不可能做到的。

我试图将表移到文件的末尾，但也无法让它起作用。我的程序在甚至到达入口点之前就出现段错误，gdb 毫无用处，我无法理解发生了什么，并被迫绝望地将`readelf`的输出倒出来，希望有人能发现问题。好吧，*有人*确实做到了，而且还很快，但显然这不是可持续的软件开发策略。

### `mold`链接器拯救了一天

链接器处于特权位置。它知道关于程序内存布局的一切，并且似乎可以随意地向其中添加新的 ELF 段。如果存在解决方案，我相信它将在链接器中找到。

所以我开始学习[链接脚本](https://sourceware.org/binutils/docs/ld/Scripts.html)。结果发现它有一个`PHDRS`命令，正是我所需要的，但我无法弄清楚如何使用它。无论我尝试什么，我都只能得到“无法分配头部”的错误。我得出结论，简单地*请求*这个功能会更容易些。

我给 GNU binutils 邮件列表[发了封邮件](https://sourceware.org/pipermail/binutils/2023-November/130640.html)…… 然后我创建了一个 [LLVM 问题](https://github.com/llvm/llvm-project/issues/72386)……

然后我提了一个 `mold` 的问题。维护者立刻明白了我的意图，并且[让它成为现实](https://github.com/rui314/mold/commit/eb6c213f2a9aa8a101b2b52a791be369d165e6a9)，只需要改变一行代码。真是*太棒了*。

我迫不及待地等待新版本的发布，但是我太兴奋了，居然在我的智能手机上从源代码编译了这个庞大的 C++ 项目，*只是为了与之集成*。我所需要做的就是在 `LDFLAGS` 中加入 `-Wl,--spare-program-headers,2`。这给了我两个`NULL`程序头，让我的补丁工具可以随意编辑。它工作得*非常完美*。

到目前为止，`mold` 是唯一具有这个功能的链接器，而且对于 `lone-embed` 来说绝对是必需的。如果它在 ELF 中找不到至少两个`NULL`程序头，它将彻底拒绝修补。

如果其他人也能获得这个功能就好了。除非我能找到一种方法，将程序头表移动到文件末尾而不会破坏一切，否则我基本上被锁定使用 `mold` 了。嗯，我并不在乎。它是一个很棒的链接器，*而且*它是自由软件。我对此很满意！
