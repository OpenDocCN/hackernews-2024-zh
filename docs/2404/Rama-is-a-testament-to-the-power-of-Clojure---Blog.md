<!--yml

category: 未分类

date: 2024-05-27 13:39:48

-->

# 拉玛是Clojure强大力量的证明 - 博客

> 来源：[https://blog.redplanetlabs.com/2024/04/30/rama-is-a-testament-to-the-power-of-clojure/](https://blog.redplanetlabs.com/2024/04/30/rama-is-a-testament-to-the-power-of-clojure/)

从一个想法到一个生产系统，[拉玛](https://redplanetlabs.com/docs/~/index.html)花了超过十年的全职工作时间。想想如果没有Clojure，可能需要多长时间，我不禁颤抖。

拉玛是一个集成和泛化后端开发的编程平台。以前，后端开发需要使用各种数据库、应用服务器、队列、处理系统、部署工具、监控系统等，而现在，拉玛可以独自构建任意规模的端到端后端系统，只需使用一小部分代码。其核心是一种实现新编程范式的新编程语言，与“面向对象”、“命令式”、“逻辑”和“函数式”范式处于同一水平。拉玛的[Clojure API](https://blog.redplanetlabs.com/2023/10/11/introducing-ramas-clojure-api/)直接访问这种新语言，而[Rama的Java API](https://redplanetlabs.com/docs/~/tutorial1.html)则是这种语言的一个薄包装。

Clojure的设计中有很多对开发Rama至关重要的地方。其中有三个特别突出：其定义抽象的灵活性，其强调不可变性，以及其围绕使用纯数据结构编程的取向。除了这些对于维护Rama实现中的简单性至关重要之外，Rama在其分布式编程和索引方法中也采纳了这些原则。

## 在其他语言中需要语言支持才能在库中做到的能力

拉玛的语言是图灵完备的，并且主要通过Clojure宏来定义。因此它仍然是Clojure，但在许多基本方面的语义有所不同。在其核心，拉玛将函数的概念泛化为所谓的[“片段”](https://redplanetlabs.com/docs/~/clj-dataflow-lang.html)。而函数通过接收任意数量的输入参数，然后返回最后一次操作的单个值来工作，片段则可以多次输出（称为“发射”），可以输出到多个“输出流”，并且可以在发射之间或之后进行更多的工作。函数只是片段的一种特殊情况。拉玛片段编译成高效的字节码，而恰好是函数的片段与Java或Clojure中的函数一样高效执行。

尽管 Rama 包含了这种新的编程语言，实现了这种新的编程范式，它仍然是 Clojure。因此它们之间可以完美互操作。Rama 代码可以直接调用 Clojure 代码，反之亦然。它们之间没有任何摩擦。Rama 本身是由常规 Clojure 代码和 Rama 代码混合实现的。

既不是 Rich Hickey，也不是 John McCarthy 曾经设想过会在他们的抽象中构建这种完全不同的编程范式，更不用说一种重新定义几乎每一种编程语言基础（函数）的范式了。他们不需要这样做。Clojure 以及它的 Lisp 前辈是几乎不限制你形成抽象的语言。在其他每种语言中，你至少必须符合它们的语法和基本语义，并且在编译时与运行时控制的能力有限。Lisp 在编译时有很好的控制能力，这让你可以做出令人难以置信的事情。

Lisp 程序员自它发明以来一直在努力解释为什么这是如此强大，为什么这对简化软件开发有重大影响。所以我不会试图总结这种力量的普遍性，而是专注于它对 Rama 的重要性。我会把你引向 Paul Graham 的文章 [“Beating the Averages”](https://paulgraham.com/avg.html)，这篇文章是我大学时期第一次学习 Lisp 时的灵感来源。当我第一次读到这篇文章时，我并没有完全理解它，但我特别被其中的一些句子所吸引：“我们的大部分代码正在做一些在其他语言中非常难以实现的事情。由此产生的软件可以做到我们竞争对手的软件无法做到的事情。也许有某种联系。”

Rama 核心的新语言就是一个例子。其他语言只有在语言级别设计和实现时才能顺畅地支持多种根本不同的范式。否则，你就必须借助字符串操作（如 SQL 中所做的那样），这并不顺畅，会造成复杂性的混乱。任何抽象化都无法完全隐藏这种复杂性，试图这样做通常会带来新的复杂性（例如 [ORMs](https://medium.com/building-the-system/dont-be-a-sucker-and-stop-using-orms-190add65add4)）。

使用 Clojure，你可以在库级别上做到这一点。我们不需要特殊的语言支持，而是在 Clojure 的基本原语之上构建了 Rama。

拉玛语言并非唯一一个混合这种范例的例子。另一个例子是[Specter](https://github.com/redplanetlabs/specter)。Specter 是一个通用的用于查询和操作数据结构的库。它也是拉玛 API 的关键部分（因为拉玛中的视图，称为[PStates](https://redplanetlabs.com/docs/~/pstates.html)，是任意组合的持久化数据结构），也是拉玛实现的关键部分。拉玛源码和测试中约 1% 的代码行是 Specter 的调用点。

你可以在任何语言中定义 Specter 的抽象。在 Clojure 中使其特殊之处在于其性能。使用 Specter 进行查询和操作甚至比手工编写的 Clojure 代码更快。Specter 性能的关键在于其[内联缓存和编译系统](https://github.com/redplanetlabs/specter/wiki/Specter's-inline-caching-implementation)。内联缓存是一种我之前只在语言或虚拟机级别见过的技术。例如，它是 JVM 实现多态性的关键部分。由于 Clojure 的灵活性和在编译时为 Specter 调用点编程的能力，我们能够在库级别利用这一技术。这完全在幕后完成，Specter 的用户获得了一个表现力强、简洁的 API，而且速度非常快。

## 不变性与数据结构取向的力量

Clojure 在众多 Lisp 方言中独树一帜，它[强调不变性](https://clojure.org/about/state)。其核心 API 设计用于处理不可变数据结构。此外，Clojure 鼓励使用普通数据结构表示程序状态，并提供了一个富有表现力的 API 用于操作这些数据结构。引言“比起让 10 个函数操作 10 个数据结构，不如让 100 个函数操作一个数据结构。”是[Clojure 设计理念](https://clojure.org/about/rationale)的一部分。

这些哲学理念对拉玛的发展产生了重大影响，在管理其实现中提供了巨大帮助。在代码段中需要考虑的状态越少，推理就越容易。当项目像拉玛这样大（19 万行源码，22 万行测试），具有许多层抽象和无数子系统时，想要完全“掌握住”整个系统的一部分是不可能的。我经常不得不重新阅读代码部分，以提醒自己有关特定子系统的详细信息。通过减少系统复杂性，尤其是通过不变性的方式，可以获得越来越多的回报，尤其是在代码库越大的情况下。

Clojure 并不是强制在每种情况下都使用不可变性，这也很重要。Rama 追踪许多不同类型的状态，在某些情况下，与其间接地通过像 Haskell 中的 [State Monad](https://en.wikibooks.org/wiki/Haskell/Understanding_monads/State) 这样的东西工作，我们发现直接使用可变性要简单得多。在某些算法中，使用内部的 [volatile](https://clojuredocs.org/clojure.core/volatile!) 实现要简单得多。尽管如此，Rama 中绝大部分的代码都以不可变的风格编写。当我们使用可变性时，几乎总是在单个线程内部进行隔离。与使用像 [atom](https://clojure.org/reference/atoms) 这样的并发可变性不同，我们使用一个 volatile 并将事件发送到拥有它的线程以与其交互。

Rama 接纳并扩展了 Clojure 的不可变性原则，并围绕数据结构编写代码。这些原则是表达端到端后端的 Rama 方法的基础。Rama 编程的核心是实现视图（PStates），这些视图实质上只是使用与内存中数据结构相同的 Specter API 进行交互的数据结构。这与数据库形成鲜明对比，数据库具有固定的数据模型和专用的 API 用于与其交互。任何数据库都可以在 PState 中复制其表达性和 [性能](https://blog.redplanetlabs.com/2024/04/25/better-performance-rama-vs-mongodb-and-cassandra/)，因为数据模型只是数据结构的特定组合（例如，键/值是一个映射，列导向是排序映射的映射，文档是映射的映射等）。

Rama 的语言将 Clojure 的不可变原则延伸到编写分布式、容错和异步代码。与 Clojure 相似之处很多，例如使用词法闭包进行匿名操作、不可变的本地变量以及在重叠时具有相同的语义。在分布式计算方面，Rama 更进一步进行了一些工作，例如范围分析以确定哪些变量需要在网络边界之间传输。Rama 的循环语法类似于 Clojure，并具有额外的功能，即可以在循环迭代期间在集群中跳转进行分布式计算。在 Rama 中，所有这些都通过数据流的力量线性编写，切换线程/节点像任何其他操作一样（称为“分区器”）。

Clojure 的原则只是确实能够极大简化软件开发的良好思想。这些原则在历史上充斥着复杂性的分布式系统/数据库中尤为重要。这也是为什么这些原则对于 Rama 及其实现如此核心的原因。

## 结论

这里存在一个看似矛盾的地方——如果Clojure能带来如此大的生产力增益，为什么它在行业中仍然是一门小众语言？为什么那些使用Clojure的人没有彻底击败竞争对手，以至于每个程序员都在赶紧采用Clojure来平衡竞争力场？

我认为这仅仅是因为Clojure没有涵盖软件开发的所有方面。这并不是对Clojure的批评，而是对其范围的认识。诸如持久数据存储、部署、监控、随时间演变应用程序状态和逻辑、容错性和扩展性等问题，构建端到端软件的巨大成本。通常情况下，当使用数据库时，Clojure的原则会受到损害，因为数据库会强迫你围绕其数据模型和功能来组织代码。

这就是为什么我们对Rama如此兴奋并长时间致力于它的原因，因为Rama解决了构建端到端后端所涉及的一切问题，无论规模大小。Rama提供了灵活的数据存储，以数据结构的形式表达，内置了部署和监控功能，拥有一流的应用程序演进和更新功能，完全具备容错能力，并天生具备可扩展性。它在保持Clojure伟大原则和函数式编程根源的同时实现了所有这些。

*如果你想在[Clojure Slack](https://clojurians.slack.com/)上讨论，我们在#rama频道活跃。*
