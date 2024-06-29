<!--yml

类别：未分类

日期：2024年05月27日 15:02:25

-->

# Rust 和 C 文件系统 API [LWN.net]

> 来源：[https://lwn.net/Articles/958072/](https://lwn.net/Articles/958072/)

| **本文由 LWN 订阅者支持**LWN.net 的订阅者使这篇文章 — 以及其周边的一切 — 成为可能。如果您欣赏我们的内容，请[订阅](/subscribe/)，支持接下来的文章。 |
| --- |

由 **Jonathan Corbet** 提供

2024年01月15日

如

[Rust-for-Linux 项目](https://rust-for-linux.com/)

随着内核不断累积抽象层，使得 Rust 代码能够与现有的 C 代码接口。随着围绕一组

[文件系统抽象](/ml/linux-fsdevel/20231018122518.128049-1-wedsonaf@gmail.com/)

由 Wedson Almeida Filho 在十二月发布的文章显示，尽管如此，对于这些抽象设计方法存在一些张力。大多数内核的 C 程序员偏爱的方法似乎将会胜出，但随着 Rust 在内核中的使用增加，这是一个可能会反复讨论的问题。

如果 Rust 开发者希望使用发布的抽象实现文件系统，他们的工作将是组合一个看起来像这个封面信件中的示例的实现：

```
    impl FileSystem for MyFS {
        fn super_params(sb: &NewSuperBlock<Self>) -> Result<SuperParams<Self::Data>>;
        fn init_root(sb: &SuperBlock<Self>) -> Result<ARef<INode<Self>>>;
        fn read_dir(inode: &INode<Self>, emitter: &mut DirEmitter) -> Result;
        fn lookup(parent: &INode<Self>, name: &[u8]) -> Result<ARef<INode<Self>>>;
        fn read_folio(inode: &INode<Self>, folio: LockedFolio<'_>) -> Result;
    }

```

这里定义的函数执行内核可能要求文件系统实现的任务：例如 `read_dir()` 用于读取目录内容，或者 `lookup()` 用于在目录中查找文件名。所有这些操作都定义为一个称为 `FileSystem` 的单一特性。

与 C 代码中 API 定义的方式不同，这里的组织方式将文件和文件系统相关操作分布在多种对象类型中。一个文件系统作为一个整体由 [`struct super_block`](https://elixir.bootlin.com/linux/v6.7/source/include/linux/fs.h#L1192) 定义，它具有一组关联操作在 [`struct super_operations`](https://elixir.bootlin.com/linux/v6.7/source/include/linux/fs.h#L2059) 中。但文件系统实现了许多其他对象类型和相关操作，包括索引节点 ([`inode`](https://elixir.bootlin.com/linux/v6.7/source/include/linux/fs.h#L639), [`inode_operations`](https://elixir.bootlin.com/linux/v6.7/source/include/linux/fs.h#L1968))、目录条目 ([`dentry`](https://elixir.bootlin.com/linux/v6.7/source/include/linux/dcache.h#L82), [`dentry_operations`](https://elixir.bootlin.com/linux/v6.7/source/include/linux/dcache.h#L128))、文件 ([`file`](https://elixir.bootlin.com/linux/v6.7/source/include/linux/fs.h#L992), [`file_operations`](https://elixir.bootlin.com/linux/v6.7/source/include/linux/fs.h#L1916)) 和地址空间 ([`address_space`](https://elixir.bootlin.com/linux/v6.7/source/include/linux/fs.h#L448), [`address_space_operations`](https://elixir.bootlin.com/linux/v6.7/source/include/linux/fs.h#L404))。

作为 C 侧对象模型运作方式的示例，考虑 `lookup()` 是一个 inode 操作，`iterate_shared()`（用于实现 Rust 特性中定义的 `read_dir()` 函数）是一个文件操作，而 `read_folio()` 是一个地址空间操作。

Matthew Wilcox [有几个问题](/ml/linux-fsdevel/ZT7BPUAxsHQ%2FH%2FHm@casper.infradead.org/) 关于所提出的抽象，首先是涉及到一些操作的 `inode` 参数。在内核的 C 代码中，这些函数接受一个 `struct inode` 指针，快速转换为特定于文件系统的结构体指针。这里缺乏类型安全性；函数无法知道实际上是否传递了正确类型的 inode 指针。在 Rust 中，似乎应该可以做得更好。

Almeida [回应道](/ml/linux-fsdevel/CANeycqrm1KCH=hOf2WyCg8BVZkX3DnPpaA3srrajgRfz0x=PiQ@mail.gmail.com/) 这个接口确实做得更好。`inode` 参数的类型是 `&INode<Self>`，将该参数的实际类型与文件系统类型绑定在一起；在不遇到编译错误的情况下，无法向这些函数传递错误类型的 inode。

不过，Wilcox 的另一个问题证明更难回答。在 C 代码中用于读取目录的文件操作是：

```
    int (*iterate_shared) (struct file *, struct dir_context *);

```

Rust 代码中的等效部分（上述的 `read_dir()`）将一个 inode 引用作为参数，而不是 `struct file` 指针。Wilcox 指出，虽然 "<q>玩具文件系统</q>" 只需 inode 中存储的信息，其他文件系统需要 `file` 结构中的信息。因此，在接口中不保留该结构似乎有些奇怪。Almeida 回答道，迄今在 Rust 中实现的文件系统不需要从 `struct file` 中获取任何内容；他补充说：“<q>将 `file` 传递给 `read_dir` 将要求我们引入一个无人使用的不必要抽象，我们已被告知不要这样做</q>”。但他说，接口可以在必要时进行更改。

Wilcox [做出了强烈回应](/ml/linux-fsdevel/ZZWhQGkl0xPiBD5%2F@casper.infradead.org/)：

> 因此，在至少有一个非玩具文件系统实现之前，我们不应合并任何内容，甚至不应再次提交进行审查。要么坚持我们已经定义的对象导向（即分离的 aops、iops、fops 等，具有基本相似的参数），要么提出对我们在 C 中已有的更改。只处理玩具文件系统会导致不良架构。

Almeida 对这条消息感到不满；他问道：“<q>你是不是在说，Rust 不能像 C 一样拥有性能特征相同但 API 不同的库，除非我们也修复 C 的 API？</q>” Wilcox 回答道，内核的对象模型存在是有其原因的，Rust 方面不应在没有充分理由的情况下改变这一模型。Al Viro 补充说，现有的对象集合和操作应当被视为“<q>外部给定</q>”；他表示可以出于充分理由而进行更改，但在这里并没有这样的理由。

相反，Kent Overstreet 认为，Rust 的抽象是设计更清晰接口的一种方式，这个接口不应需要与 C API 匹配。清理后者是“<q>一项巨大的麻烦</q>”，因为需要同时更改所有现有的文件系统，而在 Rust 中创建更好的东西相对容易。

> 因此，对我来说，似乎更容易在 Rust 端实现更干净的版本，一旦我们知道它的样子，也许我们再更新 C 版本以匹配——或者也许我们会点燃所有东西，继续用 Rust 重写一切。

与此同时，Almeida 抱怨说，当没有东西使用它时将 `file` 结构传递给 `read_dir()` 的做法正是 Rust 开发者被建议避免的事情。这些开发者长期以来一直在努力解决将抽象合并以便可以在没有能够合并用户的情况下使用它们的问题。Wilcox 回答说，建议被误解了；Rust 开发者被要求不要合并没有用户的抽象，而不是改变他们正在合并的抽象的接口。Greg Kroah-Hartman 表示同意，称这些抽象应该适用于所有文件系统，而不仅仅是当前已实现的那些。Dave Chinner 说，这个问题正是为什么他一直建议 Rust 开发者重新实现 ext2 的原因，因为虽然这个文件系统相对简单，但使用了大部分核心文件系统 API。

最终，Almeida [屈服了](/ml/linux-fsdevel/CANeycqrubugocT0ZEhcUY4H+kytzhm-E4-PoWtvNobYr32auDA@mail.gmail.com/)，并表示他将用独立文件、inode 和地址空间特性制作新版本的抽象；`read_dir()` 将更新为使用 `File<T>` 引用。Wilcox [同意](/ml/linux-fsdevel/ZZ6T6aBjOf+vA9sB@casper.infradead.org/) 这种方法似乎是正确的前进方向。

因此，这次具体的讨论似乎已经有了结论。但是在 Rust 中实现内核功能肯定会提供无数机会，创建比内核 C 代码数十年来演变的更干净、更安全的新接口。有时这些 API 将展示对 C 代码演变原因的误解；有时它们确实会更好。但无论如何，与 C API 显著不同的 Rust API 将使维护和未来开发更加困难，因此在 Rust 端创建与 C 端不同的 API 想法仍将面临强烈的抵制。

如同在 2023 年维护者峰会上[讨论的](/Articles/952029/)一样，一个答案是使 C 代码演变成与 Rust 开发的更好接口相匹配。这个想法有些合理，但也要求 Rust 开发者在 C 中做大量工作，而他们正试图摆脱这件事。改变核心内核 API、更新所有这些 API 的用户并获得这些变化的接受将不是一件简单的任务。这样的政策无疑会阻碍 Rust 端开发更好接口；结果将是更好的可维护性，但这是有真实成本的。

Overstreet 提到可能在某个时候会发生的事情：“<q>点火烧掉一切，并继续用 Rust 重写所有内容</q>”。如果每个人使用的 API 都在 Rust 代码中，API 分歧就不是问题了。你的编辑器的预测能力非常有限，但有几件事情似乎很可能会发生：有提议在某个时候用 Rust 实现替换一些核心代码，而抵制这样做的力量将是强大的。在这次讨论中，David Howells 明确表示他不希望 Rust 出现在核心内核中。

但是，这是未来的讨论内容；Rust 首先必须在内核边缘证明自己。但一旦骆驼的鼻子（或螃蟹的鼻子）进了帐篷，其余的事情似乎都有可能跟随。敬请关注，这将会很有趣。

* * *

(

[登录](https://lwn.net/Login/?target=/Articles/958072/)

发表评论)
