<!--yml

类别：未分类

日期：2024-05-27 14:36:47

-->

# 未能通过《Advent of Code》学会 Zig - ForrestTheWoods

> 来源：[`www.forrestthewoods.com/blog/failing-to-learn-zig-via-advent-of-code/`](https://www.forrestthewoods.com/blog/failing-to-learn-zig-via-advent-of-code/)

这不是我打算写的博客文章。

2018 年，我写了一篇关于 [通过《Advent of Code》学习 Rust](https://www.forrestthewoods.com/blog/learning-rust-via-advent-of-code/) 的文章。这应该是一个令人兴奋的续集，我在这里学习 [Zig](https://ziglang.org/)。不幸的是，我失败了。Zig 给我带来的更多是沮丧而不是快乐，我在 6 天后就失败了。

我最大的失误是决定用 Rust 和 Zig 解决所有《Advent of Code》的难题。不幸的是，我失去了重新在 Zig 中实现它们的动力。一些轻微的办公室竞争也没有帮助。事后看来，我应该咬紧牙关，专注于 Zig。教训已经吸取。

我对我的初始 Zig 经验做了相当不错的笔记。我不想让它白白浪费，所以我想在这里分享一下。

GitHub：[链接](https://github.com/forrestthewoods/aoc2021/blob/master/zig/src/main.zig)

为了帮助理解，我稍微介绍一下我自己。

我是一名拥有 15 年工作经验的游戏/VR 开发者。我大部分时间都在 C++ 和 C#（Unity）中度过。我既做过游戏玩法又做过系统代码。我喜欢 Rust，[讨厌 Python](https://www.forrestthewoods.com/blog/things-i-like-about-python/)。我对 webdev 或 JavaScript 一无所知，并打算保持这种状态。我可能比较喜欢 C++ 而不是 C，但我努力让我的 C++ 简单并且有些像 C。

几个月前，我对 Zig 知之甚少。我知道它的存在，但其他不多。

我参加了令人惊叹的 [Handmade Seattle](https://handmade-seattle.com/) 大会，Zig 在那里有着非常强大的存在感。[Andrew Kelley](https://twitter.com/andy_kelley)，Zig 的生活独裁者，给出了一场关于 [数据导向设计实践指南](https://media.handmade-seattle.com/practical-data-oriented-design/) 的出色演讲。许多演示文稿都是使用 Zig 构建的项目。

我的思维模式是：

+   Rust 试图成为更好的 C++

+   Zig 试图成为更好的 C

我不知道这是否完全准确。我认为这是公平的。

我决定利用 [Advent of Code](https://adventofcode.com/2021/about) 来帮助学习 Zig。我在 2018 年用 Rust 做这件事时过得很愉快。在 AoC 之前，我尽力阅读 Zig 文档。下面是我发现的一些更有用的页面：

下载 Zig 并编译 Hello World 非常简单，只需按照 [入门指南](https://ziglang.org/learn/getting-started/) 页面的说明即可。

当解决 AoC 难题时，我做了很多笔记。我尽力写下我遇到的每个问题或困难。这是一个很长的列表。

接下来的几节以半原始形式包含了这些笔记。我会在之后分享我的心得体会。

对`anyerror!void`感到立即困惑。我认为`!`是`Option` / `std::optional`。所以我认为`!`是一个前缀，但它也是一个后缀？非常困惑。（注：我错了。`?T`是`Option<T>`，而`E!T`是`Result<T,E>`）

搞不清楚如何打印一个整数。在 ZigLearn 中找不到文档或示例。需要搜索`std.debug.print`而不是`std.log.info`来找到示例。很烦人。

建立速度很慢。当我在与基本语法错误作斗争时，需要大约~3 秒钟的最小时间，这真是令人沮丧地慢。我希望有一个快速的`zig check`。

编译错误信息一般般。充满了我不关心的无用信息和编译器调用栈。

Zig 参考文档非常需要示例。搞不清楚如何使用`std.fmt.parseInt`。

默认情况下在运行时捕获内存泄漏非常酷！编译器应该能够在编译时轻松检测到未调用`nums.deinit()`。

搞不清楚如何获取`ArrayList`中的项目数量。[文档](https://ziglang.org/documentation/0.8.1/std/#std;ArrayList)没有提供这个基本信息。`items`的类型是`var`。这没有帮助。

搞不清楚如何访问`ArrayList`中的数组。我得到一个编译错误：`error: array access of non-array type 'std.array_list.ArrayListAligned(u32,null)'`。这确实看起来像是一个数组类型。

需要访问`myArray.items[idx]`而不是`myArray[idx]`。我明白了。但非常不直观，需要了解实现细节。

失败的`expect`有一个巨大的调用栈，非常烦人。

搞不清楚如何比较字符串。在 ZigLearn 或参考文档中找不到任何信息。谷歌搜索“zig string compare”也没用。这真是一个惊人的障碍。

啊哈。字符串根本不存在，只有`[]const u8`。需要使用`std.mem.eql`。

搞不清楚如何创建一个元组的`ArrayList`。从未解决这个问题。不得不创建一个辅助结构体。

很奇怪我可以调用`myArraylist.push`，但需要循环遍历`for (myArrayList.items)`。感觉不一致。我明白了，但超级不直观。我使用过许多编程语言，没有一种语言表现得像这样。

我实际上搞不清楚如何在发布模式下运行。`zig build run`会构建和运行调试版本。最终发现`zig build -Drelease-safe`会构建发布版。网络告诉我需要手动运行。直到写这篇博客文章才发现可以通过`zig build -Drelease-safe run`构建和运行。

`std.mem.tokenize`非常棒。与`foreach`类似的`for`循环结合时，比起原始的 C 语言具有很棒的功能。

`TypeInfo`的[文档](https://ziglang.org/documentation/master/std/#std;builtin.TypeInfo)简直一文不值。

我应该将分配器传递给每个函数吗？感觉不太好。也许我应该创建一个全局变量？全局变量是邪恶的并且感觉很不好。

如何以二进制形式打印数字？在 Zig 中搜索如何做事情的方式有 50% 的机会会把你带到一个正在讨论该功能的 [GitHub 问题](https://github.com/ziglang/zig/issues/1358)。这对我来说发生了很多次。

如何获取 tokenized 行的长度？[tokenize](https://ziglang.org/documentation/master/std/#std;mem.tokenize) 返回什么？它返回 `anytype`。很酷。非常有帮助。不得不编译，失败，并在错误中看到类型是 `[]const u8`。

缺乏 `zig-analyzer` 使得学习变得困难。

我如何创建 lambda / 闭包 / 局部函数？有一个非常复杂的模式，存在很多问题。当然，有一个 [GitHub 问题](https://github.com/ziglang/zig/issues/229) 讨论这个问题。

```
const do_thing = (struct {
    fn call(self: @This(), last_number: u8, tiles: []Tile) ?usize {
        _ = self;
        // do stuff
        return num;
        }
    }
{}).call;

```

位移是一个巨大的痛苦。`const mask : usize = @as(usize, 1) << @truncate(u6, i);`。我最终得到了：`@truncate(u16, @as(

以下行无法编译：`const mask = 1 << (num_bits - 1);`。糟糕。

转换似乎也太复杂了：`@as(type, value)`。糟糕。

不能对布尔值执行 `xor`!? 糟糕。

无法重定向 `zig build run` 的输出。以下方法无效：`zig build run > c:/temp/out.txt`。:(

解析 `i32` 令人愉快（一旦我弄清楚了）。

`tokenize` 令人愉快。

`try` 语法简洁明了。

zig 编译器错误的调用堆栈太长，也不够有帮助。必须向上滚动很长一段时间才能找到实际的错误。

`zig fmt src/main.zig` 很好用。希望它能自动运行在所有文件上。

希望有一种简单的方法来漂亮打印嵌套的 `ArrayList`。

生成数组的数组需要很多痛苦的反复尝试。

努力对 `ArrayList` 进行可变迭代。诀窍是：`for (board.tiles.items) |*tile| { ... }`。我搞不懂。我发现在 Zig 中理解值和引用语义非常困难和令人困惑。

索引检查很酷... 但错误消息不够有帮助。`thread 9888 panic: index out of bounds`。是哪个索引？切片长度是多少？这是重要信息。

VSCode 对异常的断点工作得很好。

一流的调用堆栈支持是出色的。这是 C 和 C++ 中非常缺失的东西。

我的解析器不起作用，但不确定原因。天啊，`tokenize` 不像预期的那样工作。需要使用 `split` 而不是 `tokenize`。令人毛骨悚然。

想要使用正则表达式。没有内置的。有一个 GitHub 项目。不确定如何使用它。

Zig 中没有标准的包管理器。Gyro、zigmod 和子模块都存在。没有标准的。

没有 Zig 教程显示如何正确导入外部 Zig 代码。这感觉像一个巨大的疏忽。

在 VSCode 中调试 Zig 很容易设置。

调试 `ArrayList` 不实用。只显示第一个元素。:(

Zig 的模板/泛型很有趣。`pub fn max(x: anytype, y: anytype) @TypeOf(x, y) { return if (x > y) x else y; }`

编译时间仍然令人沮丧。我希望有一个快速的 `zig check`，类似于 `cargo check`。

将 `i32*i32` 转换为 `usize` 很困难。 （不确定为什么我会做这个笔记。）

我实际上不知道 `@` 前缀代表什么意思。我认为是“内建”的。对于内建的某种未指定的定义。

Zig 无法推断类型真的很烦人。如果我创建了 `var count` 并返回它，函数的返回类型是 `usize`，那么 `var count` 显然是一个 `usize`。

没有笔记。简单的问题，不需要学习任何新东西。也许我开始掌握了基础知识？

没有在 Zig 中完成。

这就是我掉进去的地方。我并不打算停下来。我说我会在几天后赶上。我没有。出于没有特定原因，我的动力消失了。

拾起 Zig 的想法感觉像是一项琐事。我知道我必须在 Zig 中学习一些东西，比如 `HashMap` 和 `HashSet`，而与之抗争的想法并没有带来愉悦的感觉。

在一个愉快的假期之后，我试图在一月份重新学习 Zig + AoC。然后在某个时候，[Zig 0.9](https://ziglang.org/download/0.9.0/release-notes.html) 发布了，所以我决定这将是一个很好的升级练习。

升级 Zig 是一个手动过程。下载一个新的 `zig.exe` 并替换路径中的旧文件。有点乏味。

我立即遇到了关于未使用函数参数的新错误。我过于复杂的本地函数解决方法停止工作了，有时候。我不知道为什么，所以我只是把它变成了一个普通的函数。

接下来，我不得不处理 [allocgate](https://pithlessly.github.io/allocgate.html)。我发现这个发布说明真的很令人沮丧。博客文章详细解释了为什么进行了更改。但它没有清楚地解释我需要对我的代码进行哪些更改才能使其编译。在我看来，这是不必要的令人沮丧的。

当我的代码工作起来时，我意识到了一件令人震惊的事情。Zig 似乎不会报告被优化掉的函数的编译器错误。什么？我需要明确确保所有函数都被调用以检测到分配错误。呃。

到这一点，我简单地对其他项目更感兴趣。Zig 又被搁置了。

那么这对我意味着什么呢？我想我有三个关键印象。

1.  Zig 目前并没有带来愉悦的感觉。

1.  Zig 可能有几个深刻的想法。

1.  Zig 令人兴奋，我将在 2022 年复查 Advent of Code。

Zig 还没有准备好面向大众。这并不具有争议性。Zig 团队明确表示它还没有准备好用于生产。我同意。

我认为 Zig 对大多数程序员来说还没有准备好。它的进展不如我想象的那么顺利。

关于 Zig 的信息非常少。使用谷歌回答问题非常困难。r/zig 不活跃。r/adventofcode 解决线程没有 Zig 解决方案。

Zig Discord 非常活跃，#advent-of-code 频道充满了非常友好、非常乐于助人的人。他们回答了我很多问题。如果没有他们的帮助，由于沮丧，我很可能早就放弃了。

Zig 的文档质量较差。[标准库参考](https://ziglang.org/documentation/0.9.0/std/)是实验性的，没有什么用。[语言参考](https://ziglang.org/documentation/0.9.0/)相当不错，但它是一份参考资料，而不是指南或教程。

我找到的最好的学习资源是[ZigLearn](https://ziglearn.org/)。这是一个不错的开始。它不是最好的引入概念的工具。例如，它从一堆不熟悉的`test "Do Thing"`函数声明开始，而没有解释该语法的含义或如何运行测试。第二个例子介绍了`try`关键字，但没有定义它。这个至关重要的定义在之后的第七部分中出现，位置有些偏僻，很容易被忽略。

编写文档是**困难**的。我尽量不把 Zig 和 Rust 进行比较。然而，我必须得把 Zig 缺乏文档与绝佳的[Rust 书籍](https://doc.rust-lang.org/stable/book/)和[Rust 标准库文档](https://doc.rust-lang.org/stable/std/)进行比较。

我认为 Zig 现在需要投入大量资源到“Zig 书”中。这需要多年的完善和等待，等到 Zig 1.0 太晚了，在我看来。现在就开始，到 Zig 1.2 时应该会相当不错。

我要简要抱怨一下 Zig 页面上标题为[为什么要选择 Zig，而不是已经有 C++、D 和 Rust？](https://ziglang.org/learn/why_zig_rust_d_cpp/)。

Zig 是一种很酷的语言。我认为它有很多有趣的想法。我认为未来有很多原因让人选择使用 Zig。以下是“为什么选择 Zig”的要点：

+   没有隐藏的控制流

+   没有隐藏的分配

+   对于无标准库的一流支持

+   一个可移植的库语言

+   一个包管理器和用于现有项目的构建系统

+   简单

+   工具

这个列表在我看来并不吸引人。Zig 是一种带有许多非常酷的想法的酷语言。这个列表并不酷，也没有包含大部分让我兴奋的特性。“就像 C 语言，但在各个方面更好”。这简直就是一个了不起的销售宣传。

"没有隐藏的控制流"似乎是 Zig 社区的一个呼声。这绝对不是我会把它作为新编程语言最吸引人的第一个要点的原因。我不知道为什么它是每个页面上列出的第一个特性。

我也认为它部分错误。世界上从来没有人因为`a + b`调用一个函数而感到困惑或不满。操作重载可能是非常邪恶的。不允许`operator.`或`operator->`的重载。重载基本的数学运算符是可以的，不会令人困惑，也是一个好主意。我做了很多三维向量的数学计算，所以我愿意在这一点上坚持。

```
fn lerp(a: Vec3f, b: Vec3f, t: f32) Vec3f {
    // Obviously good and easy to read
    return a*(1.0-t) + b*t;

    // Obviously bad and hard to read
    return add(mul(a, 1.0 - t), mul(b, t));
}
```

我认为 Zig 可能有一些深刻的想法。

没有语言分配器函数是深刻的。将分配器传递到每个结构函数中有点繁琐，但也很深刻。在 C++中，你有工作马 `std::vector<Foo>`。指定分配器需要指定一个模板类型，这是一个巨大的痛点。在 Zig 中，你可以轻松地更改分配器类型。`std.heap` 包含 `ArenaAllocator`、`FixedBufferAllocator`、`GeneralPurposeAllocator` 和 `StackFallbackAllocator`。令人愉快。

Zig 似乎有强大的能力整合到现有项目中。它的跨平台交叉编译能力令人羡慕。我认为“在项目的隔离部分使用 Zig”可能是深刻的。我不确定。我强烈怀疑将 Zig 整合到真实项目中比 Zig 团队建议的要困难。

Zig 的 comptime 通用能力令人着迷和深刻。我还没有完全理解它们。我不完全确定它们与 C++模板或 Rust 泛型的比较如何。Zig 的 comptime 能做到 C++模板能做的吗？有什么限制吗？我不确定。

Zig 的[错误处理](https://ziglang.org/documentation/0.9.0/#Errors)是深刻而强大的。但也非常令人困惑。我未能理解可以合并、推断、强制、延迟和追踪的错误集。错误追踪非常酷，我非常希望其他语言也有。我不喜欢诸如 `anyerror!?u32` 这样的语法。

Zig 的[色盲](https://kristoff.it/blog/zig-colorblind-async-await/)异步/协程系统可能尤为深刻。我还没有深入研究到足以形成自己的观点。这可能是相对于 C 甚至 C++的一个巨大特性优势。

Zig 没有引起我的兴奋。我发现学习它很困难和令人沮丧。缺乏文档，缺乏教程，糟糕的编译器错误，不直观的类型，混乱的语法，混乱的引用与值，无法通过谷歌搜索，不可组合的迭代器等等。

然而，Zig 有很多我喜欢并期望在现代语言中看到的东西。运行时安全性、切片、延迟、simd、带标签的联合体（需要语法糖）、`foreach` 风格的 `for` 循环、comptime、标准构建系统、包管理器的承诺、基本迭代器等等。

我对 Zig 的未来感到兴奋。我喜欢它正在探索新的想法，并推动下一代编程语言的发展。我认为在 Zig 1.0 发布时它可能会非常稳定。看起来那将是几年后的事情。

我曾经试探过 Zig 的水，感觉有些冷。我还没准备好全力投入。可能我会在 2022 年的编程之前更多地尝试 Zig。下一次我会更清楚我在签约什么。

感谢阅读。
