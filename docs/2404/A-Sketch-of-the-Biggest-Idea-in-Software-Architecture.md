<!--yml

category: 未分类

date: 2024-05-27 13:26:12

-->

# 软件架构中最大的理念草图

> 来源：[https://www.oilshell.org/blog/2022/03/backlog-arch.html](https://www.oilshell.org/blog/2022/03/backlog-arch.html)

| [博客](/blog/) | [oilshell.org](/)

# 软件架构中最大的理念草图

2022-03-12 (最后更新 2022-05-25)

这篇文章最初被称为*积压：软件架构*，直到我编辑它并看到一个连贯的主题出现。

它详细阐述了[窄腰](/cross-ref.html?tag=narrow-waist#narrow-waist)：一个与网络、操作系统、语言设计、编译器和分布式系统相关的#[软件架构](/blog/tags.html?tag=software-architecture#software-architecture)的理念。

另一个我考虑过的标题是*运行时软件构成概览*。也就是说，你可以对比这两种构建软件部件的风格：

1.  细粒度静态类型、构建时组合、静态链接、[APIs](/cross-ref.html?tag=API#API)和版本号

1.  粗粒度的**"腰"**、运行时组合、[ABIs](/cross-ref.html?tag=ABI#ABI)、[IPC](/cross-ref.html?tag=IPC#IPC)和无版本协议

许多程序员熟悉第一种风格。本文讨论的是你在大规模和长期视角中看到的**第二种风格**。

这篇文章**长而深**，包含了很多链接，所以你可能需要分次阅读。请告诉我你的想法 [在评论中](https://old.reddit.com/r/oilshell/comments/tcy7ko/a_sketch_of_the_biggest_idea_in_software/?)！我特别欢迎类似材料的参考。

## 背景

我很高兴上一篇文章有很多讨论，[互联网是设计有窄腰的](//www.oilshell.org/blog/2022/02/diagrams.html)：

我写了一些抽象的想法，但读者理解并应用了它们。并且一位读者回答了我关于[narrow waist](/cross-ref.html?tag=narrow-waist#narrow-waist)历史的问题，我在下文的*号召行动*中重复了。

另一方面，也有一些回应展示了我想反驳的误解。特别是，对于权衡考虑的缺乏：

+   **本地**便利 vs. **全球**经济，灵活性，一般性和扩展性

+   **代码**视角 vs. **系统**视角

+   精细**类型** vs. 粗粒度**“腰部”**

要说服这一点，我将深入例证：展示代码，分析现有设计，并提出新的设计。我在[wiki](https://github.com/oilshell/oil/wiki/Perlis-Thompson-Principle)和[Zulip](/cross-ref.html?tag=Zulip#Zulip)上收集了大量资料。

但我可能不应该花几个月时间写作和辩论关于#[software-architecture](/blog/tags.html?tag=software-architecture#software-architecture)。最好根据我提倡的原则**构建**一些东西。

因此，我在这篇文章中压缩了许多主题。我陈述了主要观点，并进行了一些理由说明。

## 激励设计问题

具体来说，这些思想将帮助我们解决以下一些问题：

1.  shell 是否应该有**两个层次**？

    +   外部进程和内部“函数”？字节流和结构化数据流的管道？

    +   我认为进程和字节流仍然应该是“主要”的，因为这使得 shell 更具互操作性和实用性。它们都是基本的[narrow waists](/cross-ref.html?tag=narrow-waist#narrow-waist)。去年一月：["Shells Should Shell Out"](//www.oilshell.org/blog/2021/01/philosophy-design.html#shells-should-shell-out)。

1.  [JSON](/cross-ref.html?tag=JSON#JSON) 是否成为 shell 的新“狭腰”？

    +   它是一个**狭窄的**腰部，但并不像文本或字节流那样普遍。例如，HTML和CSV与JSON不同，它们也不应该相同。

1.  我们如何设计一个更好的分布式操作系统？

1.  [Docker](/cross-ref.html?tag=docker#docker)设计良好吗？它如何改进？

    +   我举这个例子是因为我看到有人声称"Unix哲学是显而易见的"并已被纳入标准实践中。

    +   这与事实相去甚远：[Docker](/cross-ref.html?tag=Docker#Docker)是一个最近的设计，其风格深受反Unix的影响。（现在Oil的构建使用[podman](/cross-ref.html?tag=podman#podman)，这是一个不错的兼容性改进。）

    +   但是尽管它的设计，Docker解决了一个真实的问题，并具有[显著的创新](https://lobste.rs/s/ftkfnu/secure_containerized_browser#c_r6xgbh)。

所以我相信以下的想法与行业中最重要的力量和发展相关。我很高兴看到Docker正在从两个方面"重构"成更加Unix化的东西：通过Red Hat和其他公司成为[OCI](/cross-ref.html?tag=OCI#OCI)，并且逐渐脱离Kubernetes。（相关：[Docker的第二次死亡](https://www.tariqislam.com/posts/kubernetes-docker-dep/)）

## 什么是狭窄的腰部？

大多数读者理解了[上一篇文章](//www.oilshell.org/blog/2022/02/diagrams.html)：我从网络中借用了[narrow waist](/cross-ref.html?tag=narrow-waist#narrow-waist)这个术语，并将其扩展到软件中。

但是对我来说，不是所有的狭窄腰部都是相同的。值得区分这些类别，还有更多：

1.  像[Internet Protocol](https://en.wikipedia.org/wiki/Internet_Protocol)、[UTF-8](/cross-ref.html?tag=utf8#utf8)和[JSON](/cross-ref.html?tag=JSON#JSON)这样的小而简单的机制。

    +   这些"狭窄"的腰部在某种意义上非常显著。Jon Postel、Ken Thompson和Doug Crockford成为它们的"编辑"或创作者并非巧合。

1.  像[POSIX shell](/cross-ref.html?tag=posix-shell-spec#posix-shell-spec)、JavaScript和C++这样的语言标准。

    +   这些都很大且难以重新实现。我通过参与Oil的工作亲身体会到这一点！

    +   这些都是窄腰，因为它们解决了{用户程序...} × {语言实现...}的互操作性问题。

1.  "偶然"窄腰，比如[Win32](https://en.wikipedia.org/wiki/Windows_API)和[x86](https://en.wikipedia.org/wiki/X86)。

    +   它们的发展不受标准机构指导。它们也很大且难以重新实现。

1.  类似于[LLVM](https://llvm.org)的API。正如主页所述，LLVM不是一个虚拟机。它实际上是一个软件库，每次发布都会更改，需要消费者更改其代码。这使得它与其他更多关于运行时组合的窄腰有所不同。

1.  ... ?

因此，更加具体地描述是值得的，下面的文章将进一步界定定义并探索相关概念。

窄腰思想的最明显反对意见是，窄腰只是一个**标准**！标准能够实现互操作性。

但是标准必须有一个来源。窄腰可能会成为标准，也可能不会。此外，LLVM不是一个标准，也不打算成为标准。

小时glass隐喻还建议了为什么这个概念是强大的，以及应该追求什么。你希望有一个小东西与许多其他东西接口。

### 准确定义“Unix哲学”

我花了一周时间起草一篇名为*Unix中的三个窄腰图表*的文章。第一句是：

> 你听过关于“Unix哲学”的模糊说法吗？对此你感到困惑或怀疑吗？

这是一篇有价值的文章，因为出人意料地，[窄腰](/cross-ref.html?tag=narrow-waist#narrow-waist)的概念**提出了关于Unix的新内容**，而且更加具体！我在这里使用了参考文献，包括经典的文献[在这个维基百科页面上](https://en.wikipedia.org/wiki/Unix_philosophy)。

我有这3个窄腰的图表：

1.  **进程**

    +   {本地代码，shebang脚本，shell函数，...} × {启动，终止，管道，重定向，以特权运行，...}

1.  **文件描述符**

    +   {文件，管道，终端，套接字，... } × {读，写，ioctl，... }

1.  树形结构的**命名空间**（未结构化数据的文件系统）

    +   {磁盘，SSD，内存中的 tmpfs，通过回环文件的文件，... } × {ls，mount，使用 git 的版本，通过 HTTP 服务，... }

这些图表显示 Unix 使用多个狭窄的腰部来实现动态和可扩展的**多态性**。

文件描述符的案例展示了权衡的双方。你不知道静态地可以在描述符上使用哪些系统调用。你也不知道会得到哪些错误！我通过以下内容重新学习了这个教训：

+   [Oil 0.9.6 中的一个 bug](//www.oilshell.org/blog/2022/01/notes-themes.html#oil-096-on-december-30th)：`write()` 在 `open()` 返回的描述符指向目录时可能失败，返回 `EISDIR`。

+   [Hello World 中的 Bugs](https://blog.sunfishcode.online/bugs-in-hello-world/)：`write()` 在描述符指向磁盘文件时可能失败，返回 `ENOSPC`。Python 2 存在此问题，但 Python 3 已修复。

尽管文件描述符的多态设计使 Unix 能够组合，这也是 shell 强大的原因之一！我在文章中举了例子。

Go 通过像 `Reader` 和 `Writer` 这样的单函数接口来解决这个问题，并且更一般地使用 `-er` 模式。这里有一个有趣的引用：

> 如果 Haskell 能够拥有[开放多态性]，可能以 Go 为模型。
> 
> — [Philip Wadler: 轻量级 Go](https://youtu.be/Dq0WFigax_c?t=912)

更多：

+   我提到与 Plan 9 的关系（修复 Unix 中的组合 bug）和 REST（[统一接口约束](https://en.wikipedia.org/wiki/Representational_state_transfer#Architectural_constraints)）。

+   我链接了两篇重要的学术论文以及 Unix 相关的分析。

+   我还注意到**文本行**与**文本**之间是一个明显的狭窄腰部，这在[最后一篇文章](//www.oilshell.org/blog/2022/02/diagrams.html)中有所描绘。

    +   实际上，Oil的[QSN](/cross-ref.html?tag=QSN#QSN)格式利用了这个狭窄腰，而GNU的NUL分隔格式则没有。（这是 `xargs -0` 接受的格式，提到在[xargs帖子](//www.oilshell.org/blog/2021/08/xargs.html)中。）

    +   特别是，`wc -l`，`head`，`tail` 和 `tail -f` 与[QSN](/cross-ref.html?tag=QSN#QSN)一起免费工作，但你需要更多像 `head -z` 和 `tail -z` 这样的代码来支持NUL分隔的格式。

这篇文章还没有完成，但这是我最想发布的一篇。

### 狭窄腰的特性

在软件中，[狭窄腰](/cross-ref.html?tag=narrow-waist#narrow-waist)最重要的特征是减少了[O(M x N)代码爆炸](/cross-ref.html?tag=m-by-n-explosion#m-by-n-explosion)，从而实现了互操作性和代码复用。

我也意识到 "narrow" 这个词有两个明显的意义：

1.  就架构**连接**（拓扑）而言。

    +   例如，应用程序和物理网络是通过互联网的狭窄腰部解耦的。它们不直接与彼此接口。

1.  就概念的**大小**而言。

    +   IP是一个小概念，Unix是少数几个概念。

    +   但Web是一个庞大的概念集（HTTP，HTML，SVG等）。C++和Shell也是庞大的。

因此这个问题值得更多的思考，也许需要更多的术语。

下面是更多描述狭窄腰部的方式：

1.  它们是**妥协**。它们使系统经济实惠且可能，但不一定是最优的。

    +   如果你有一个小型或专业的网络，你可以设计比互联网更高效的东西。

1.  它们是通过显式的**设计**和隐式的**进化**混合产生的。

    +   互联网和Web都是经过设计并经历了进化。但我会说Web的进化更多一些。

1.  设计可以做得**好也可以做得差**。进化可以是有目的的，也可以是偶然的。

    +   [JSON](/cross-ref.html?tag=JSON#JSON)是一种明确的设计，比CSV好多了。

    +   我们应该在设计上做得更好，因为由此产生的网络效应意味着我们经常“陷入”糟糕的设计中。

关于演化：

1.  窄腰可以持续几十年，通常以**无版本**方式演化。

    +   例如，Unix shell 是广泛使用的最古老的语言之一，而且在[Thompson shell](/cross-ref.html?tag=thompson-shell#thompson-shell)、[Bourne Shell](/cross-ref.html?tag=bourne-shell#bourne-shell)、[Korn shell](/cross-ref.html?tag=ksh#ksh)、[bash](/cross-ref.html?tag=bash#bash)、[OSH](/cross-ref.html?tag=osh-language#osh-language) 和 [Oil](/cross-ref.html?tag=oil-language#oil-language) 之间有持续的兼容性。

    +   窄腰具有与依赖它的功能数量成比例的**惯性**。

1.  但是窄腰也可以**移动**！

    +   TCP/IP → HTTP

    +   POSIX C APIs → Linux x86 ABI

1.  它们受到极端的**经济压力**和网络效应的影响。例如：

1.  惯性的缺点是窄腰可以**抑制创新**。

1.  窄腰通常被**过度扩展**到新的应用中。

    +   Web 可能被过度扩展，从超链接文档网络发展为应用交付平台（JavaScript 中的[单页应用](https://en.wikipedia.org/wiki/Single-page_application)）。

    +   它也通过[Electron](https://www.electronjs.org/)扩展到桌面 UI 框架。

    +   Linux 可能被过度扩展到嵌入式设备，特别是那些具有实时要求的设备。

一些最近的窄腰包括 [Docker](/cross-ref.html?tag=docker#docker) / [OCI](/cross-ref.html?tag=OCI#OCI)，[语言服务器协议](https://en.wikipedia.org/wiki/Language_Server_Protocol)，和 [WebAssembly](https://en.wikipedia.org/wiki/WebAssembly)。我应该更具体地讨论它们在设计和用户采纳方面的不同成功程度。例如，我认为 WebAssembly 是有用的，但[并非最近所宣称的那样通用](https://news.ycombinator.com/item?id=28581634)。这是一个深度的妥协，涉及到赢家和输家。

### 错误观念

这里有一些对这个观点的常见反对意见。

(1) *文本数据难以通过程序操作*。

这**不是**对窄腰原则的反对！该原则的主要论点是关于互操作性和实施经济性。

我想提出一个**简单 vs. 易用**的论点。用 Rich Hickey 的术语，窄腰是*简单*的（不是"复杂的"），但未必易于使用。例如，Unix shell 可能学习起来很难，但其强大性使得操作系统变得小巧、可扩展。

(2) *网络确实非常混乱，因此不可靠*。

我强烈区分**混乱 vs. 稳定**。混乱的系统未必不可靠。恰恰相反 — 对稳定性的需求才是混乱的**原因**！持续的向后兼容性（例如 CSS 的多次迭代）造成了混乱，但保持系统运行。

这与我一直难以描述的另一个概念相关：**无版本演进**，我将在下文中详细描述。

### 相关思想

我们可以通过以下思想更精确地理解 [窄腰](/cross-ref.html?tag=narrow-waist#narrow-waist)：

1.  [梅特卡夫定律](https://en.wikipedia.org/wiki/Metcalfe%27s_law)表明，一个网络的价值与节点数 N 的平方成正比。

    +   这与窄腰的M × N *架构*连接相关但又不同。架构连接不是网络节点连接。

    +   思考窄腰的架构层次可能会澄清这一点。例如，CSV、JSON 和 HTML 在“级别1”上是窄腰。而每一个都**是**文字，它位于“级别0”。通用操作是“继承”的，这使得系统更小。（这个想法确实需要图示。）

1.  互联网协议遵循[端到端原则](https://en.wikipedia.org/wiki/End-to-end_principle)**，**它是[窄腰](/cross-ref.html?tag=narrow-waist#narrow-waist)。

    +   这并不意味着这两个原则是相同的。我认为在软件应用中，窄腰更具描述性和预测性。

## 示例和详细说明

让我们将这些原则应用于现实世界的系统中。再次强调，[窄腰](/cross-ref.html?tag=narrow-waist#narrow-waist)是软件架构中**最重要**的思想，因为它描述了最大和最长寿命的系统。

### Web以无版本方式发展

我想详细说明许多窄腰的“无版本”属性。您可以对比两种版本控制的哲学：

1.  指示破坏性更改的版本号，例如[语义化版本](https://semver.org/)。

    +   Linux发行版和包管理器[如NPM](https://docs.npmjs.com/about-semantic-versioning)通常将语义化版本与临时约束求解器配对，以找到依赖项的兼容版本集。这种模型可能**脆弱**，因为您可能最终运行的一组版本从未经过测试。

1.  持续向后兼容，即**无版本**演化。

    +   这里还有版本号，但它们表示的是功能添加，而不是破坏性更改。

例如，Web没有不兼容的版本，并且[JSON](/cross-ref.html?tag=JSON#JSON)是由道格·克罗克福特明确设计为无版本。

历史证明了这一规则。在[不要破坏X](//www.oilshell.org/blog/2021/12/backlog-project.html#three-analogies-dont-break-x)中，我提到XHTML和ECMAScript 4都试图通过激进的变化**破坏Web**，但由于狭窄腰带的惯性而失败。

相比之下，HTML5和ECMAScript 5以一种兼容的方式推动了Web的进化。我们应该学习和传播Web的历史，以避免重蹈覆辙。

* * *

这是关于无版本演化的一个好方法：

> 放宽要求应该是一种兼容的变化。加强承诺也应该是一种兼容的变化。
> 
> —— Rich Hickey在[Maybe Not](https://www.youtube.com/watch?v=YR5WdGrpoug)（2018年，YouTube）

例子：

+   HTML5定义了`<hr>`和`<hr />`表示同样的意思，而以前的HTML版本则更为严格。（这是关于[自闭合标签的问题](http://xahlee.info/js/html5_non-closing_tag.html)。）因此，HTML5**放宽了对**网页作者的要求，这是一种兼容性的变化。

+   添加新功能**增强了**浏览器对网页作者的承诺。例如，HTML5添加了`<video>`标签，这是一种兼容性的变化。

### 字节和文本是必不可少的狭窄腰带。

我经常与那些对shell中的临时、不正确的#[解析](/blog/tags.html?tag=parsing#parsing)感到沮丧的人们就“文本与细粒度类型”进行反复讨论。

我认为，支持[JSON](/cross-ref.html?tag=JSON#JSON)、[QSN](/cross-ref.html?tag=QSN#QSN)和HTML的shell将解决这个问题。它将减少shell程序中的解析量，并使其更加正确。

我还声称解析是一个O(M + N)问题，而类型可能会创建O(M × N)问题 —— 事实上也确实如此。

为了更加具体，这里是一个重要的评论，我在 [一月](//www.oilshell.org/blog/2021/01/audio-and-graphics.html)，[六月](//www.oilshell.org/blog/2021/06/hotos-shell-panel.html#conclusion)，[七月](//www.oilshell.org/blog/2021/07/blog-backlog-1.html#concepts)，和 [八月](//www.oilshell.org/blog/2021/08/history-trivia.html#the-first-paper-about-unix-shell-thompson) 中提到过。

我提出了 M × N 的论点，并且使用了像 IntelliJ 和 WebAssembly 的文本格式这样的具体例子：

> 你有 M 种格式和 N 种操作，为全球程序员编写 M × N 工具是不可行的。

我还注意到这种权衡：

> 这不是绝对的；事实上，人们确实试图填写 M × N 网格中的每个单元格 [在某些领域]。他们在某种程度上完成了，这确实有一些优势。

我还引述了 Rust 设计师 Graydon Hoare 在文本方面的看法。尽管他的“抱怨”主要是关于文本的信息密度，但也涉及到文本支持的广泛操作范围。

> [文本] 可以通过算法进行比较、差异化、聚类、校正、总结和过滤。它允许多方编辑。它允许分支对话、潜伏、注释、引用、审查、总结、结构化响应、注释解释，甚至是粉丝的剧情虚构。人们使用文本的广度、规模和深度是任何其他东西都无法比拟的。

我注意到由协议缓冲区生成的代码存在问题的 M × N 爆炸（与源代码相对）。

例如，相等性变得**依赖于架构**，而不是通用的。在许多系统中，这是值得的，但这是一个权衡。

### 标语：文本是唯一能够达成一致的事物。

这里有两种标语的变体。它旨在强调文本作为一个狭窄的腰部的观点。

> 一个 [PowerShell](https://en.wikipedia.org/wiki/PowerShell)、[Elvish](https://elv.sh/)、[Rash](https://docs.racket-lang.org/rash/) 和 [nushell](https://www.nushell.sh/) 脚本的最低公分母是 Bourne shell 脚本（最终会变成 Oil 脚本）。

因为每个替代壳都选择一种不同的结构化数据作为其狭窄的腰部（.NET 对象、树形数据、Racket 数据结构和表格）。**文本**是它们都同意的最结构化的格式，并且**shell**是用文本进行粗粒度组合的语言。

我预测这将成为一个真实的事情，而不是理论上的！我毫不怀疑，已经有 bash 脚本调用 PowerShell 脚本，并且随着替代性 shell 变得流行，将会出现更复杂的聚合物。

这并不意味着那些 shell 不值得使用，或者潜在地更方便。但它强调了需要一个更好的 Bourne 风格的 shell。

and

第二种表达方式：

> 一个 [Common Lisp](https://common-lisp.net/)、[Clojure](https://clojure.org/) 和 [Racket](https://racket-lang.org/) 程序的最低公分母是 Bourne shell 脚本（最终会变成 Oil 脚本）。

再次强调，这些语言相似，但数据模型不兼容。（不仅仅是复合数据结构；Clojure 对数字和字符串的概念来自 JVM。）

这两个口号实际上是另一种表达方式，与[上一篇文章](//www.oilshell.org/blog/2022/02/diagrams.html)的一句口号相似：

> Unix 对每个程序员同样不方便，这是件好事。

### CSV、JSON、HTML - 表格、记录、文档

这篇文章的完整标题是：

> CSV、JSON 和 HTML 之所以不同，是因为表格、记录和文档不同。

这再次回应了这样一个观点，即[JSON](/cross-ref.html?tag=JSON#JSON)是 shell 的“新”狭窄腰部。表格和文档是软件中的基本结构，在 JSON 中表达它们很笨拙。

我可以举例说明，例如：

```
{"name": "alice", "age": 42}
{"name": "bob", "age": 43} 
```

和

```
 ["a", {"href": "/home"}, ["anchor text"]] 
```

这个框架源自于论文[统一表格、对象和文档](https://www.microsoft.com/en-us/research/publication/unifying-tables-objects-and-documents/)（Meijer 和 Schulte，2003），但技术细节有所不同。

### 动态类型和静态类型之间的权衡（常见问题解答）

狭窄的文本与细粒度的静态类型相矛盾。我的目标是突出权衡，并分析每种风格在自然和高效的情况下的应用。

许多程序员似乎认为没有权衡——或者至少他们在互联网上是这么*说*的。我相信当他们创建工作系统时，他们经常使用动态、粗粒度的视角！

[这个最近的评论](https://old.reddit.com/r/ProgrammingLanguages/comments/t0lzeq/types_considered_harmful_pdf_2008/hyfhhu2/)链接到典型的回复，现在已经形成了一个常见问题解答：

这是一个相关的、精彩的视频，我想要进行推广：

我不了解 F# 语言，但显然它对数据有一个非常类似 Clojure 的视角，尽管是静态类型的。"类型提供者"机制解决了仅在运行时可用的类型问题，例如在 SQL 模式中或者隐含在 JSON 和 CSV 文件中。

总体而言，谬误在于我们使用动态类型时是因为“懒得写类型”。有许多有用的程序并非完全静态类型化，我认为这种趋势正在增加。（口号：*糟糕的结构化软件正在吞噬世界*。）

真正的问题在于**规模**在空间和时间上的异质性和可扩展性！

## 细化

在关于可扩展性的讨论中，我说过我在探索*运行时组合和无版本演进的理论和指南*。Shell 是关于运行时的软件组合，而不是通过静态链接进行组合。

因此除了[Perlis-Thompson Principle](/cross-ref.html?tag=perlis-thompson#perlis-thompson)，[窄腰](/cross-ref.html?tag=narrow-waist#narrow-waist)，和[O(M × N)代码爆炸](/cross-ref.html?tag=m-by-n-explosion#m-by-n-explosion)之外，这里还有一些值得探索的概念。

### 投影到腰部

我需要一个关于通过改变数据表示来实现**代码重用**的概念名称，以便投影到一个窄腰部。例如：

1.  `/proc`文件系统将内核元数据投影到文件系统的窄腰上。

    +   现在你可以使用现有的工具像`ls`和`open()`来探索进程的状态。

1.  任何使用[FUSE](https://en.wikipedia.org/wiki/Filesystem_in_Userspace)的系统也是如此，例如Michael Greenberg的[File File System](https://mgree.github.io/ffs/)将[JSON](/cross-ref.html?tag=JSON#JSON)投影到虚拟文件系统上。这允许重用`cd`和`ls`等工具。

1.  [gron](https://github.com/tomnomnom/gron)工具将类似树状的[JSON](/cross-ref.html?tag=JSON#JSON)投影到"文本行"的窄腰上。这允许重用像[grep](/cross-ref.html?tag=grep#grep)和[awk](/cross-ref.html?tag=awk#awk)这样的工具。

注意，[JSON](/cross-ref.html?tag=JSON#JSON)是一个窄腰，但它已经投影到了另外两个窄腰上：文件系统和文本行。哪一个（如果有的话）适合取决于什么样的工具帮助您解决特定的问题。

### 腰部仿真

这是最直接的一个。如上所述，Windows和Linux之间仿真有很大的动机，反之亦然。这个平台获得了成千上万个应用程序的"免费"。

另一个例子是 Illumos 借用 FreeBSD 的 Linux 系统调用 ABI 模拟，以便运行用户上传的[Docker](/cross-ref.html?tag=docker#docker) 容器。这是 **动态、运行时** 的组合，使用[ABI](/cross-ref.html?tag=ABI#ABI)，而不是通过将代码编译到内核[API](/cross-ref.html?tag=API#API) 的静态组合。

### 腰部的扩展

我认为“腰部扩展”是以下思想的一个很好的术语：

+   网络是 Unix 的一种谦逊且精彩的**扩展**，添加了简单的网络和超链接（来自[最近的评论](https://news.ycombinator.com/item?id=26865164)；[2013 年的更长评论](https://news.ycombinator.com/item?id=6131335)）。

    +   这个设计并不明显！存在着一个长期的超文本系统历史，并非都是建立在 Unix 上的。研究并解释这段历史将是很有意义的。

    +   [2006 年的 apenwarr](https://apenwarr.ca/log/20061201)：*网络之所以有效，主要是因为它大部分只是在模仿 Unix 的聪明设计*。 （“有效”是很多软件缺乏的一个重要特性。）

    +   [网站自然地由 Shell 脚本制作](//www.oilshell.org/blog/2020/02/good-parts-sketch.html#web-sites-are-naturally-made-with-shell-scripts)（2020）

+   [git](/cross-ref.html?tag=git#git) 是 Unix 文件系统的分布式扩展。

    +   再次强调，早期的系统并不像现在这样，比如 CVS 和 SVN。与网络一样，这个想法在事后看来是显而易见的。当它变得“必需”时，它就显而易见了，但某人必须发明它。

    +   git 的用户界面看起来很凌乱，但是它有一个清晰的[narrow waist](/cross-ref.html?tag=narrow-waist#narrow-waist)。这篇文章似乎是一个很好的解释，但可能有更好的解释：[Git 是一个纯函数数据结构](https://blog.jayway.com/2013/03/03/git-is-a-purely-functional-data-structure/)。

### 腰部之间的组合

Unix 拥有多个狭腰：进程、文件描述符、文件系统、文本行、非结构化文本和字节。它们中的每一个都允许 M × N 个事物组合，但它们也必须**相互组合**。

“几件构成的事物”相当于[Perlis-Thompson 原则](/cross-ref.html?tag=perlis-thompson#perlis-thompson)。当我开始这个系列时，我不确定这个术语和“狭腰”是否必要 —— 也许它们都等同于“简单性”。

但是在通过示例工作之后，我将它们视为不同但相关的内容。因此，清楚地写出狭腰如何相关将是很好的。这似乎是一种独特的长期架构风格。

再次强调，如果你有参考资料，请告诉我。我**不想**写别人已经解释过的想法，或者在已有术语存在时创造新术语。

### 添加腰部

常见的做法是通过现有的较小狭腰创建一个新的更大狭腰。

例如，[语言服务器协议](https://en.wikipedia.org/wiki/Language_Server_Protocol) 使用 JSON-RPC [进行通知和响应](https://microsoft.github.io/language-server-protocol/overviews/lsp/overview/)。

反过来，[JSON-RPC](https://en.wikipedia.org/wiki/JSON-RPC) 是基于 [JSON](/cross-ref.html?tag=JSON#JSON) 和像 TCP/IP 或管道的传输方式构建的。

### 腰部之间的层次结构

Unix 中的数据表示之间存在清晰的层次结构，影响哪些操作是有效的：

+   在[上一篇文章](//www.oilshell.org/blog/2022/02/diagrams.html)中，我提到**文本**（ASCII、UTF-8）是**字节**的一个特例，并继承了[对字节的操作](//www.oilshell.org/blog/2022/02/diagrams.html#bytes-flat-files)。

+   我之前提到**文本行**是**文本**的一个特例。

+   同样，JSON、CSV 和 HTML 都是**文本**，并继承了[对文本的操作](//www.oilshell.org/blog/2022/02/diagrams.html#text-narrow-waist-of-unix-architecture)。

## 行动号召

### 乔恩·波斯特让互联网的“瘦腰”变窄。

在一位非常有帮助的读者的电子邮件后，我添加了[上篇文章的附录](//www.oilshell.org/blog/2022/02/diagrams.html#addendum-february-28th)。我想要转录[范·雅各布森的视频](https://www.youtube.com/watch?v=69p78tfm29o)的开头10分钟。他描述了互联网规范编辑[乔恩·波斯特](https://en.wikipedia.org/wiki/Jon_Postel)的角色 — 具体来说，他对互联网设计中**几十年来坚持不懈的极简主义追求**：

> 这个“瘦腰”不是上帝赐予你的东西。这是你创造出来的。这是艰苦的工程。

和

> 不幸的是，世界上并没有很多乔恩·波斯特。几乎每个项目都能有一个这样的人将是件好事。

这让我想起了肯·汤普森在[Unix Shell: History and Trivia](//www.oilshell.org/blog/2021/08/history-trivia.html)中引用的情感。他们都喜欢解锁巨大功能的极简主义。

我也很享受阅读这些纪念文章：

关键是**人们**必须采取不同的行动来创建有价值的、可互操作的系统！

## 结论

经过[一年多](//www.oilshell.org/blog/2021/12/review-arch.html)围绕这些#[软件架构](/blog/tags.html?tag=software-architecture#software-architecture)主题的讨论，我对它们感到相当满意。它们为Oil的设计提供了启发，并将继续提供支持。准确定义和例证支持确实有所帮助。

希望这个大纲对你也有用。我希望我能写得更简短，但我没有时间 :-)

并且，[请再次发送](https://old.reddit.com/r/oilshell/comments/tcy7ko/a_sketch_of_the_biggest_idea_in_software/?)相关的参考资料。它们将有助于未来关于这些理念的文章。（我对互联网中的“瘦腰”的历史](//www.oilshell.org/blog/2022/02/diagrams.html#whats-the-history-of-this-idea)并不是很清楚或达成一致的历史感到惊讶。）

* * *

现在我想转换到更“战术”的东西：将Oil翻译成C++！自从[2021年3月的最后一个里程碑](https://www.oilshell.org/blog/2021/03/release-0.8.8.html)以来，这个项目已经搁置了一整年。

如果你认为我们需要一个新的、有原则的Shell，请捐款到我的新[Github赞助者](https://github.com/sponsors/oilshell)页面。我会在即将发布的博客文章中再次请求捐款。所有的钱都将用于贡献者和“员工”，**而不是**我自己！

## 附录

### Lambda演算是一个狭窄的腰部

这是一篇“有趣”的文章，帮助我们定义。它基于《[类型与编程语言](https://mitpress.mit.edu/books/types-and-programming-languages)》第5章的这句引用：

> [λ演算的重要性](https://en.wikipedia.org/wiki/Lambda_calculus)在于它既可以被看作是一个描述计算的简单编程语言 **也可以** 被看作是一个关于其能够进行严格陈述的数学对象。

+   换句话说，它减少了 {任意算法...} × {关于它们的归纳情况的证明} 的M×N爆炸。

+   **派生形式** 关于λ演算与编译器中的 **中间表示** 类似。关于语言的证明之所以费力，与实现编译器的原因非常相似！

### Wiki, Zulip

这篇文章很长，但我还是漏掉了一些重要的东西。正如在*激励性设计问题*部分提到的那样，这些想法涉及到像Docker和Kubernetes这样的基础云软件的设计。

但在我继续写这些话题之前，我希望Oil的状态更好一些。暂时先看看我的Wiki页面和Zulip链接。（我真希望我有一个单一的头脑风暴和研究应用。）

我还在去年12月提到了#[containers](https://oilshell.zulipchat.com/#narrow/stream/308821-containers) Zulip stream。这是一个（草草地勾勒出的）概述线程：

+   [Docker 总结](https://oilshell.zulipchat.com/#narrow/stream/308821-containers/topic/Docker.20Summary/near/264915992)。Docker 的价值及其反 Unix 设计。其他线程通过经验证实了这些说法。
