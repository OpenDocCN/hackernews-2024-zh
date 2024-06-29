<!--yml

分类：未分类

日期：2024-05-27 13:10:12

-->

# 摇树，被误解为园艺技术的算法 — wingolog

> 来源：[https://wingolog.org/archives/2023/11/24/tree-shaking-the-horticulturally-misguided-algorithm](https://wingolog.org/archives/2023/11/24/tree-shaking-the-horticulturally-misguided-algorithm)

让我们来谈谈摇树（tree-shaking）吧！

### 从失望槽上看

但首先，我需要谈谈 WebAssembly 的一个隐秘问题：尽管充满了各种炒作，WebAssembly 在 Web 上的成功有限。

有 [Photoshop](https://medium.com/@addyosmani/photoshop-is-now-on-the-web-38d70954365a)，看起来确实是一个真正的成功案例。5 年前有 [Figma](https://www.figma.com/blog/figma-faster/)，虽然它们现在很少提及 Wasm。还有很多小型的 NPM 库在幕后使用 Wasm，通常是从 C++ 或 Rust 编译而来。我认为 [Blazor](https://dotnet.microsoft.com/en-us/apps/aspnet/web-apps/blazor) 可能会用于一些企业内部应用，尽管这可能是受到它们的市场营销误导。

或许你还记得 5 年前关于 Unreal 引擎的三维第一人称射击游戏的炒作演示，但那是之前的 Unreal 的主要发布版本，一直处于试验阶段；当前的 Unreal 5 不支持 WebAssembly 目标化。

别误会，我认为 WebAssembly 很棒。它在非 Web 环境中取得了很好的成功，我认为它将成为 Web 平台的一个关键且不断增长的部分。不过我怀疑，我们现在才刚刚度过了 [失望槽](https://en.wikipedia.org/wiki/Gartner_hype_cycle)。

值得反思一下 Web Wasm 成功与失败的本质。以 Photoshop 为例，我认为我们可以说 Wasm 在将大型 C++ 程序带入 Web 方面做得非常出色。我知道这需要相当多的工作，但我了解到最终结果本质上是相同的源代码，只是编译成了不同的目标。

同样对于 JavaScript 模块情况，Wasm 在将传统的 C++ 代码搬上 Web 并用 Rust 编写新的 Web 目标代码方面取得了成功。这些通常是 JavaScript 不擅长的任务，或者需要在客户端和服务器部署之间共享实现。

另一方面，对于 DOM-heavy 应用来说，WebAssembly 并不是 Web 上的成功之道。例如，没有人谈论要用 Wasm 重写 [wordpress.com](https://wordpress.com) 的前端。为什么会这样？这可能对你来说听起来像一个愚蠢的问题：Wasm 在这些方面表现不佳。但是为什么呢？如果你深入探讨一下，我认为问题在于编程模型有很大不同：Web 的主要编程模型是 JavaScript，一种具有动态类型和托管内存的语言，而 WebAssembly 1.0 则关注静态类型和线性内存。从 Wasm 到 DOM 的交互非常麻烦，只有最虔诚的 Wasm 爱好者才能克服。

相关地，对于像C或Rust这样的语言，WebAssembly（Wasm）也并不算是一个真正的成功。我猜想wordpress.com并不主要使用C++。对于这类语言的一个关键问题是，例如C#，它想要随带一个垃圾收集器，而这会让人感到很烦。查看今年三月的[这篇文章以获取更多详情](https://wingolog.org/archives/2023/03/20/a-world-to-win-webassembly-for-the-rest-of-us)。

令人高兴的是，这种限制正在消失，因为所有浏览器都将在未来几个月内支持引用类型和垃圾收集；Chrome和Firefox已经支持Wasm GC，而Safari也应该会在不久的将来跟进，这要感谢我的同事Asumu Takikawa的努力。这是一个非常激动人心的进展，我认为它将引发全新的Gartner炒作周期，随着更多语言开始更新其工具链以支持WebAssembly。

### 如果你不喜欢我的桃子

这就引出了今天笔记的核心：在编译器创建紧凑代码的情况下，WebAssembly在网络上将会胜出。如果您的语言编译器工具链可以生成少于几个过程中的千字节有用的Wasm文件，您将会成功。如果您的编译器现在还做不到这一点，您将不得不依赖炒作和已捕获的观众来推广，这最多只会导致不稳定的平衡，直到找到下一个发展方向。

在JavaScript世界中，管理膨胀和交付大小是一个庞大的行业。像[esbuild](https://esbuild.github.io/)这样的打包工具已经成为工具链中无处不在的一部分，将一组JS模块编译成单个文件，该文件应该仅包含程序中使用的函数和数据类型，并且还应用了特定领域的大小压缩策略，例如最小化（使别名更小）。

让我们专注于树摇。这个视觉隐喻是，你编写了一堆代码，但对于任何给定的页面，你只需要其中的一部分。因此，你可以想象一个树，其分支是你使用的模块，而叶子则是模块中的各个定义，然后你猛烈地摇动这棵树，可能会杀死它，同时还会惹恼任何筑巢的鸟。唯一保留下来的是实际上需要的东西。

这并不是树的工作方式：抓住树干并不能告诉你哪些分支对树的任务是必要的。它还[使你心理上准备去寻找错误的固定点](https://pvk.ca/Blog/2012/02/19/fixed-points-and-strike-mandates/)，去除不必要的代码而不是保留必要的代码。

但是，树摇（tree-shaking）这个名称很具有感染力，因此尽管它在园艺和算法上有些不准确，我们还是会坚持使用它。

问题在于，对于具有较厚运行时的语言，最大化摇树并不是一个主要优先事项。以Go为例：[根据golang维基](https://zchee.github.io/golang-wiki/WebAssembly/)，从Go编译到WebAssembly的最简单程序大小为2兆字节，添加导入项可能使其增加到10兆字节或更多。或者看看Python的WebAssembly端口Pyodide：[REPL示例](https://pyodide.org/en/stable/console.html)下载约20兆字节的数据。这些对于技术演示或者在极限情况下非常丰富的应用来说，是可以接受的大小，但它们并不适合Web开发的胜者。

### 摇动一个不同的树

公平地说，无论是Go的内置Wasm支持还是Python的Pyodide端口，都源自上游的工具链，生成小型二进制文件很好但并非必须：在服务器上，谁在乎应用的大小呢？实际上，当针对更小的设备时，我们倾向于看到工具链的替代实现，例如[MicroPython](https://micropython.org/)或[TinyGo](https://github.com/tinygo-org/tinygo)。TinyGo具有一个能够达到不到一千字节的Wasm后端，甚至更小！

这些替代工具链通常带有[某些限制或特殊性](https://tinygo.org/docs/reference/lang-support/)，尽管我们可以将其视为某种程度上的弊端，但可以预期目标平台对语言展示了一些相互设计的反馈。特别是在DOM的海洋中运行，一个面向Wasm的Python程序必然与“本地”Python程序有所不同，这种情况相当奇怪。尽管如此，我认为作为工具链的作者，我们的目标是提供相同的语言，尽管可能会实现标准库的不同。我确信ClojureScript的开发者如果可以的话，会愿意删除他们的[差异文档页面](https://clojurescript.org/about/differences)，也许如果Wasm成为Clojurescript的可行目标，他们就会这么做。

### 对于算法

换句话说，现在Wasm支持GC后，它可能成为Python等语言的Web开发的赢家。你需要一个不同的工具链和一个有效的摇树算法，以防用户体验下降。所以让我们来谈谈摇树！

我正在为[Hoot Scheme编译器](https://gitlab.com/spritely/guile-hoot/)工作，它针对带有GC的Wasm。目前，我们设法将最小的“主”编译单元降低到约70 kB左右，并且正在朝着更低的目标努力；从主模块导入运行时设施（当前的异常处理程序等）的辅助编译单元可以小于一千字节。虽然到达这里并不容易，但我认为对于Python来说可能会更加棘手。

一些背景：[像 Whiffle](https://wingolog.org/archives/2023/11/16/a-whiff-of-whiffle)，Hoot 编译器在用户代码之前添加一个[序言](https://gitlab.com/spritely/guile-hoot/-/blob/main/module/hoot/prelude.scm)。树摇动发生在多个地方：

+   [部分评估](https://git.savannah.gnu.org/cgit/guile.git/tree/module/language/tree-il/peval.scm)将评估未使用的绑定以获取效果，可能会省略它们

+   [修复 letrec](https://git.savannah.gnu.org/cgit/guile.git/tree/module/language/tree-il/fix-letrec.scm)也会执行相同的操作

+   CPS 经常遍历程序，仅跟随引用的函数、值和控制边缘，例如通过[重新编号](https://git.savannah.gnu.org/cgit/guile.git/tree/module/language/cps/renumber.scm)

+   有一个显式的[死代码消除](https://git.savannah.gnu.org/cgit/guile.git/tree/module/language/cps/dce.scm)通过试图省略未使用的无效分配，这种情况可能由于其他优化而产生

+   最后有一个[基本的 WebAssembly 写成的标准库](https://gitlab.com/spritely/guile-hoot/-/blob/main/module/hoot/stdlib.scm?ref_type=heads)，其定义（全局变量、表、导入、函数等）仅在残留的二进制文件中[按需包含](https://gitlab.com/spritely/guile-hoot/-/blob/main/module/wasm/link.scm?ref_type=heads)。

一般来说，过程定义（函数/闭包）是容易的部分：你只需要包含那些代码引用的函数。在 Scheme 这样的语言中，这可以帮助你很长一段时间。

然而，有三个直接的挑战。其中一个是序言中定义的评估模型是`letrec*`：作用域是递归的但是有序的。绑定值可以调用或引用先前定义的值，或者捕获稍后定义的值。如果评估绑定的值需要引用仅稍后定义的值，那么就会出错。再次强调，对于过程来说，这是微不足道的，但一旦有非过程定义，有时编译器将无法证明这种良好的“仅引用早期绑定”属性。在这种情况下，[修复 `letrec`（重新加载）](https://legacy.cs.indiana.edu/~dyb/pubs/letrec-reloaded-abstract.html) 算法将最终残留化被`set!`的绑定，而以上所有树摇动通过都需要精巧的 DCE 过程来删除它们。

更糟糕的是，一些非过程定义是*记录类型*，它们有 vtable 来定义如何打印记录，如何检查一个值是否是这个记录的实例等。即使从未使用，这些 vtable 回调也可能导致保持更多的代码活动。我们稍后会再谈这个问题。

同样地，假设你通过[`display`](https://www.r6rs.org/final/html/r6rs-lib/r6rs-lib-Z-H-9.html#node_idx_832)打印一个字符串。现在不仅引入了整个缓冲I/O设施，而且调用了一个高度多态的函数：`display`可以打印任何东西。对于位向量，存在一种情况，因此你引入了位向量的代码。对于对偶，也是如此。依此类推。

一种解决方法是改用`write-string`，它只写字符串而不写一般数据。尽管如此，你仍然会得到通用的缓冲I/O设施（端口），即使你的程序只使用一种类型的端口。

这让我想到了下一个点，即最优树摇是一个流分析问题。考虑`display`：如果我们知道程序永远不会有位向量，那么在`display`中处理位向量的任何代码都是死代码，我们可以折叠守卫它的分支。但要知道这一点，我们需要知道`display`被调用时带有哪种类型的参数，为此我们需要[更高级别的流分析](https://wingolog.org/archives/2014/07/01/flow-analysis-in-guile)。

对于Python来说，问题在几个方面被加剧。一是因为[面向对象的调度是高阶编程](https://matt.might.net/articles/implementation-of-kcfa-and-0cfa/)。你如何知道`foo.bar`实际上意味着什么？取决于`foo`，这意味着你必须在每个地方和每个调用者的调用者的调用者的表示之间传递`foo`可能是什么。

其次，在Python中查找通常比在Scheme中更动态：你有`__getattr__`方法（是这样吗？；我已经有一段时间没有做Python了），并且用户确实可能使用它们。也许在实践中这并不那么糟糕，流分析可以排除这种动态查找。

最后，也许与此相关的是，在Python中树摇的目标是模块的一团乱麻，而不是具有词法绑定的大术语。这有点像JavaScript，但没有树摇捆绑器的已建立生态系统；Python在未来几年内有很多工作要做。

### 简而言之

带有GC，Wasm使得在除JavaScript之外的其他语言中进行DOM编程成为可能。但是，只有在生成的Wasm模块小的情况下才是可行的，这意味着每种语言的工具链需要显著投入。通常这将采取具有实验性树摇算法的替代工具链的形式，以及其替代标准库促进了树摇。

唉，我去吃午饭了。祝你们组装愉快，同志们！
