<!--yml

类别：未分类

日期：2024-05-27 15:13:34

-->

# 我对 Hare 的印象

> 来源：[https://vfoley.xyz/hare/](https://vfoley.xyz/hare/)

[当 Hare 宣布](https://harelang.org/blog/2022-04-25-announcing-hare/) 在 2022 年发布时，我在 Hacker News 上看到了这篇文章，转到了网站，查看了文档，我的反应是冷淡的“无所谓”。Hare 是我所谓的“现代 C 语言”的新参与者：这些语言以 C 语言为基础灵感，试图通过添加新特性、移除一些陷阱，并优化一些尖锐的角来改进 C 语言。这些语言还试图通过省略某些特性（例如异常、RAII）来避免 C++、Ada 或 Rust 的复杂性和[认知负荷](https://github.com/zakirullin/cognitive-load/blob/main/README.md#feature-rich-languages)。现代 C 语言包括：

+   **Zig：** 可能是最著名的现代 C 语言，它的主要亮点是 `comptime`，即在编译时运行 Zig 代码的能力。这个特性在语言中广泛使用，并且它包含了其他语言的许多特性：例如，一个泛型数据结构可以通过调用一个接受类型并返回已填入输入类型的结构体的 comptime 函数来实现。

+   **Odin：** 另一个着名的现代 C 语言，Odin 是比 Zig 更简单的语言，它的灵感来源于 C 语言，还有 Pascal 和 Go。Odin 的一个酷炫特性是它的隐式上下文参数，允许自定义的记录器或内存分配器在不显式出现在函数参数列表中的情况下传递。

+   **C3：** 一种试图与其他语言保持更接近 C 语言的语言，它仍然成功地通过更好的错误处理、模块、泛型和更清晰的语义来改进 C 语言。

+   **Jai：** Jonathan Blow 的语言，目前处于私有测试阶段。与 Zig 一样，Jai 的主要思想是允许程序的任何部分通过在语句前加上 `#run` 来在编译时运行。这种语言目前用于创建 AAA 游戏，但没有人知道它何时会向普通大众开放。

在我对 Hare 文档的第一次快速浏览中，我看到了一个明显改进了 C 语言的语言：它具有更好的错误处理、更强的类型系统、边界检查的切片、`defer` 用于资源清理、UTF-8 支持，并且比 C 语言更少的语法陷阱（例如，switch 语句中没有穿透，更直观的声明语法）。Hare 没有泛型、宏，也没有任何进行编译时编程的手段，这让我对这种语言失去了兴趣。此外，与此列表上的其他语言不同，Hare 使用 QBE 作为后端，而不是 LLVM，这可能会引发对生成的程序速度的质疑。当时，我觉得 Hare 很可爱，但并不是我感兴趣的语言，也不是我认为我 *可能* 感兴趣的语言。

几年后，我在播客[开发者之声](https://www.youtube.com/watch?v=42y2Q9io3Xs)中听到了 Hare 的创建者 Drew DeVault 谈论 Hare。我喜欢 Drew 的讲话，他对编程语言的许多感觉与我自己的相符，而且我在圣诞节和新年有一周的假期，所以我想试试这种语言。当我尝试了 Hare 时，发生了一些意外的事情：我比预期更喜欢它！这种语言比其他现代 C 语言更基础，拥有的特性更少，但我发现它是那种最快最舒适地“适应我的手”的语言。我在几个小时内学会了这种语言的基础知识，内置的 `haredoc` 命令让我可以从我的终端舒适地探索标准库。没过多久，我就能够在 Hare 中编写小型程序，而无需在每一行代码处查阅文档。我越是使用 Hare，就越意识到以前我只是把它当作一堆特性来评价；现在我实际使用它后，整体体验要比简单地阅读特性要好得多。

我的一些读者可能知道，我过去是非常喜欢非常高级的编程语言、函数式编程、高级类型系统特性等的。然而，在过去的几年里，年龄使我对自己编程语言的偏好变得更加保守——我现在更重视简单性，而不是以前那样，并且我发现自己对一些高级的现代编程特性皱起了眉头，抱怨它们的存在是为了解决不常见的问题，或者更糟的是，为了给本来无聊的程序员提供一个令人兴奋的玩具。如果我年轻 5–10 年，我肯定会对 Hare 不屑一顾，但现在我在使用和构建软件时对不同的东西更加重视，Hare 对我来说更有吸引力。

我将有兴趣关注 Hare 的一个方面，即它在安全性方面的发展故事。Hare 已经比 C 语言安全得多：编译器为数组和切片访问插入了边界检查，编译器拒绝编译不处理错误条件的代码，字符串不像 C 语言那样含糊不清，你不知道 `length` 是否包括终止的 NUL 字节等。许多常见的 C 语言陷阱都被消除了。但是，该语言不跟踪所有权和生命周期——程序员必须具备这种纪律——这使得 Hare 程序容易出现使用后释放和双重释放的错误。有关使用线性类型来防止这些错误的早期讨论；时间会告诉我们 Hare 开发者能否在不牺牲语言的简洁性和“感觉”的情况下将其整合到语言中。

最后我要重申一下我对 Hare 的惊喜之情。如果你重视简洁、直接和性能，我建议你试试它几个小时；我想你也会感到惊喜的。