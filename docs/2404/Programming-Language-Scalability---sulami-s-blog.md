<!--yml

category: 未分类

date: 2024-05-27 13:10:43

-->

# 编程语言的可扩展性 - sulami 的博客

> 来源：[https://blog.sulami.xyz/posts/programming-language-scalability/](https://blog.sulami.xyz/posts/programming-language-scalability/)

我一直在思考编程语言的可扩展性。这里我指的不是性能，而是组织的可扩展性。让我们从一点背景开始。

我最初加入 CircleCI 的原因之一是因为他们在使用 Clojure，而那时我也在尝试 Clojure。我相信他们选择这种语言对早期工程师有了自我选择的效应，可能对公司有利。但无论如何，在我离开公司之前的某个时候，我们得到了一个命令，要把所有东西都迁移到 Go 上。公司正经历着一个快速增长阶段，招聘 Clojure 工程师变得越来越困难。我们已经有了相当多的 Go 代码，所以选择是显而易见的。另一个影响这个决定的因素是，大多数 Clojure 工程师可以轻松地切换到 Go 服务并进行一些改动，反之则不太成立。这并不是因为 Clojure 工程师更优秀，而是因为在一个陌生的 Clojure 代码库中要发挥作用更加困难。

在 Clojure 方面，我们还遇到了一些技术上的增长痛点。由于 Clojure 是一种高度动态的语言，我们有相当数量的运行时错误，只能通过增加测试和可观察性来对抗这些错误。静态类型的缺乏也使得除了直接熟悉代码的人之外，其他人的开发更加困难，因为几乎所有东西都倾向于成为一个值的集合形式。宏系统和强调表达能力的文化导致了各种不同的设计模式，每一种都针对其各自作者的偏好进行了优化，但对其他人来说则难以理解。另外，Clojure 没有标准的服务结构方法，所以每个人都是自行发明。在不同的 Clojure 服务之间自由跳转并不容易，因为必须学习不同的惯用法。

在我目前的工作中，我们正在运行一个 Ruby on Rails 的单体应用。工程部门规模较小，大约有 30 人，但这个单体应用已经存在了大约十年。尽管在日本特别是这里，Rails 工程师比 Clojure 工程师更容易招聘，但我们正经历与 CircleCI 类似的痛点。Ruby 也是动态类型，并且有一种几乎像普通英语的领域特定语言文化。"自动抽象"被视为一种美德。因此，对于几乎每种情况，我们有各种竞争模式，每一种都需要一些学习。在单体架构中，没有定义或强制边界，并且很难推理任何给定变更的影响。个人困扰的是元编程和模块系统，它们共同使得跟踪方法来源变得困难，即使对于 LSP 服务器也是如此。

在这两种情况下，也许可以通过更好的技术领导或者其他不同的实践来缓解问题，但是我相信应该为自己的成功设定条件。随着组织的成长，不能指望每个人都擅长自己的工作，即使是平均水平，如果统计数据是可信的。因此，我想谈谈编程语言的可扩展性，我将其定义为：

> 如果一个不熟悉某个编写在其中的代码库的工程师能够更快地生成正确的代码，那么编程语言的可扩展性就更高。

可扩展性通常与*峰值效能*相矛盾，即对于非常熟悉代码库的工程师的最大效能，因为驱动峰值效能的特性通常是为特定用例定制的抽象，如宏和支持领域特定语言。这些抽象可以使领域专家更有效率，但对其他人则构成额外的进入障碍。在这一谱系的极端端点上是代码高尔夫语言。

有许多因素影响可扩展性。流行程度决定了招聘的容易程度，并且在某种程度上也决定了生态系统的支持，例如库、工具和可用知识。防护栏有助于防止错误，其中最大的错误是内存安全和静态类型检查。后者还能更好地支持编辑器集成，例如 LSP，从而增强了可发现性。标准化的构建过程、格式化、测试等工具进一步减少了学习多种方法的潜在需求。另一方面，支持和大量使用语言扩展功能的文化会降低可扩展性，正如[Lisp 诅咒](http://www.winestockwebdesign.com/Essays/Lisp_Curse.html)中详细描述的那样。与主流差异较大的语言通常更难学习，无论是具有许多括号的古怪语法，还是涉及单子的奇特执行模型。

要明确的是，语言的可扩展性并不仅仅是管理者希望优化的内容，以确保工程师的可互换性。尽管独立团队是一个值得追求的目标，但实际上，跨团队的依赖很常见，能够提交变更到其他团队的代码可以减少产生的开销。当然，有些代码可能多个月或多年未曾改动，甚至原作者自己也无法宣称熟悉了。

举个例子，我收集了一些关于编程语言可扩展性的主观看法，大致按照在Web服务后端系统背景下从最可扩展到最不可扩展的顺序排列。在其他情境中，这个列表当然会有所不同。

<details><summary>一份非详尽的编程语言列表</summary>

+   Rust

    +   足够流行，并由几个公司赞助支持，被视为主流

    +   通过强大的类型系统和LSP集成提供出色的支持

    +   通过cargo提供出色的构建工具支持

    +   简洁而不晦涩

    +   在主流语言中学习曲线最陡峭

+   Go

    +   无处不在，尤其是受到Google的支持

    +   非常容易上手

    +   静态类型，注重显式代码

    +   在我看来，它过度简化程序员的负担，实际上增加了许多复杂性

    +   一些值得质疑的设计决策，比如`nil`和错误处理的人机工程学，但也包括模块系统和[许多小事物](https://fasterthanli.me/articles/i-want-off-mr-golangs-wild-ride)

+   Java/Kotlin

    +   由许多公司赞助支持的无处不在

    +   拥有最佳的库生态系统之一

    +   提供出色的工具支持，特别是IDE和性能分析工具

    +   易于过度复杂的设计模式

    +   构建工具陈旧

+   Python

+   Ruby

    +   足够普及（尽管正在衰退），由Stripe和Shopify等几个公司赞助支持

    +   动态类型，甚至比Python的边界执行还差

    +   元编程文化和领域特定语言的普遍存在

+   JavaScript/TypeScript

    +   无处不在

    +   TypeScript拥有较好的类型系统之一

    +   非常优化，具备不同的运行时选项

    +   设计为网页浏览器的脚本语言，因此会有相关缺陷

    +   随着时间的推移构建了许多抽象层，特别是约定

    +   容易导致几年一次的范式转变

+   Erlang/Elixir

    +   对于一般用途而言相当具有专门化，尽管在某些特定的上下文中使用

    +   动态类型

    +   对于一般用途的生态系统有些受限

    +   Actor模型和服务器需要一些学习

+   Clojure

+   Haskell

    +   静态类型

    +   非常专门化

    +   难以学习，更难精通

    +   随程序规模增长而复杂性不断增加

    +   几乎需要具有自定义运算符的领域特定语言</details>

如果我想从那个列表中选择可行的候选项，我可能会在Rust、Go或Java/Kotlin之间做出选择。C#可能也行，但我已经接近十年没有写过了。我通过一个简单的启发式方法得出这些结论，因为所有这些语言都是静态类型的，被认为是主流的，几乎是乏味的。我排除了C和C++，因为它们缺乏防护设施，排除了Python和Ruby，因为它们缺乏静态类型。作为一门语言，Typescript可能很好，但我个人对工具的稳定性有些怀疑，这可能是没有根据的。它确实很受欢迎，甚至在后端也是如此。

编程语言的可扩展性只对组织来说才有趣，而且只有在一定规模以上才重要。如果你的工程团队在未来五年内可以在一个房间里容纳下来，那可能并不重要。传统智慧认为，开始新项目的最佳技术是你所熟悉的技术。有时技术要求可能会胜过组织要求。无论如何，你都应该谨慎使用你的[创新代币](https://mcfunley.com/choose-boring-technology)。
