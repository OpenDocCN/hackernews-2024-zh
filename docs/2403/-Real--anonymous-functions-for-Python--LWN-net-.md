<!--yml

category: 未分类

date: 2024-05-29 12:49:42

-->

# "真正"的匿名函数用于 Python [LWN.net]

> 来源：[https://lwn.net/Articles/964839/](https://lwn.net/Articles/964839/)

| **你知道吗...?**LWN.net 是一份由订阅者支持的出版物；我们依赖订阅者来维持整个运营。请通过[购买订阅](/subscribe/)支持我们，让 LWN.net 继续在线上运营。 |
| --- |

由**Jake Edge**撰写

2024年3月19日

在 Python 社区中经常出现一些不同的语言增强想法；多年来，许多这些想法已经被讨论和否决多次。即使一个新的想法不太可能进一步发展，它有时仍然很难被压制。最近关于“真正”匿名函数的讨论遵循了一种可以预见的路径，但仍然有理由参与审核这些“新”想法，尽管这种练习的重复性和单调性——确实存在已被采纳的重复功能想法的例子[定义在这里](/Articles/808836/)。

一月底，Dan D'Avella [问](https://discuss.python.org/t/why-not-real-anonymous-functions/44513) 为什么 Python 没有类似 JavaScript 或 Rust 的“*真正*匿名函数”。虽然 Python 函数是常规对象，可以分配给变量或传递给另一个函数，但在他看来，这并未反映在语言的语法中。当然，有 [`lambda`](https://docs.python.org/3/tutorial/controlflow.html#lambda-expressions)，“*但它的用处相当有限，其语法实际上有些笨重*”。他想知道是否曾经提出过更灵活、完全匿名函数的建议，并表示他没有找到任何类似的 PEP。

Python 有两种定义函数的方式，[`def`](https://docs.python.org/3/reference/compound_stmts.html#function-definitions) 用于命名函数，`lambda` 用于匿名函数。虽然命名函数可以包含多个语句，`lambda` 是一个不能包含任何语句的表达式；它返回一个函数对象，该对象使用给定的参数计算表达式：

```
    >>> (lambda x: x * 7)(6)
    42

```

它创建了一个匿名函数，并使用参数六调用它，但这不是 `lambda` 的特别习惯用法。如[LWN 文章](/Articles/847960/)中所述，关于 `lambda` 的替代语法提案，该功能的常见用途是为某些库函数提供提取它们应使用的实际参数的方法，如：

```
    >>> tuples = [ ('a', 37), ('b', 23), ('c', 73) ]
    >>> sorted(tuples, key=lambda x: x[1])
    [('b', 23), ('a', 37), ('c', 73)]

```

它使用 lambda 表达式来提取每个元组的第二个元素作为排序键。

在对D'Avella的回应中，David Lord [想知道](https://discuss.python.org/t/why-not-real-anonymous-functions/44513/3)是否他在询问“多行lambda”，这个想法多年来已经出现过多次。虽然lambda表达式实际上可以跨越多个物理行，使用`\`作为行继续字符，但限制为单个表达式极大地限制了lambda的功能。对于更复杂的计算，最好使用命名函数，在本地作用域内可以避免任何名称冲突问题，正如Guido van Rossum在[2006年的博客文章](https://www.artima.com/weblogs/viewpost.jsp?thread=147358)中所述。在他看来，没有一种Pythonic的方法可以在表达式中嵌入多语句结构。

Lord指出，“<q>关于多语句lambda的讨论已经有了数年甚至*数十年*的前期迭代</q>”，他链接到Van Rossum的帖子展示了部分历史及为何认为提供这一功能存在无法解决的问题。简言之，问题在于Python块使用空白来界定，而不是大括号或关键字像`begin`和`end`，因此任何用于创建具有多语句的函数的表达式都需要一种方法来包含空白，否则它看起来将一点也不像Python。即使可以找到可接受的语法，Van Rossum也表示反对通过增加大量复杂性来避免“<q>只是一个小缺陷</q>”。

D'Avella [说过](https://discuss.python.org/t/why-not-real-anonymous-functions/44513/4)，Van Rossum的观点在18年前可能是有道理的，“<q>特别是考虑到其他语言的发展</q>”。此外，Python本身在这段时间内也发生了相当大的变化；“<q>试想一下在当时写这篇文章时在Python中使用`match`</q>”。虽然确实有许多变化，但D'Avella并未具体说明这些变化如何提供理由修改长期保持的对`lambda`限制的决定，正如讨论串中几位成员所指出的那样。

Terry Jan Reedy [说过](https://discuss.python.org/t/why-not-real-anonymous-functions/44513/8)，Python 的显著特点之一是其可读性，而添加更复杂的`lambda`表达式则与此背道而驰。无论是调试还是测试，命名函数都更胜一筹，除非是最简单的表达式：“<q>当`lambda`表达式足够明显以不需要测试，并且不太可能直接导致异常时，它们的效果最佳。</q>”在实践中，目前的lambda通常表达力已经足够，Reedy表示。

D'Avella [否定](https://discuss.python.org/t/why-not-real-anonymous-functions/44513/11) 这个看法为“<q>完全有效</q>”，但他不同意这个观点；基于他对其他语言的经验，他认为“<q>匿名函数文字是一种优雅且有用的构造</q>”。然而，正如Paul Moore [指出的](https://discuss.python.org/t/why-not-real-anonymous-functions/44513/13)，问题在于有一个需要遵循的过程，以便核心开发人员重新考虑这类长期性质的决定；有人需要解释为什么之前提出的反对意见现在不再有效：

> 或许如果你选择其中一两个，并且详细解释为什么它们不再适用，这可能比仅仅“一切都在变化，所以让我们再次提出曾经被拒绝过的东西”更有说服力。
> 
> 也许现在Python应该支持匿名函数有完全正当的理由。但是如果没有人真正提出一个能够*解决*为什么这个想法之前失败了那么多次的提案，我们永远不会知道。

问题的一部分在于很难追踪以前关于这个特性的讨论，D'Avella [说](https://discuss.python.org/t/why-not-real-anonymous-functions/44513/24)。他原本希望找到一个被拒绝的PEP，但没有找到；他对其他讨论的搜索[并不特别成功](https://discuss.python.org/t/why-not-real-anonymous-functions/44513/21)。Reedy [指出](https://discuss.python.org/t/why-not-real-anonymous-functions/44513/36)了一些不同讨论的来源，这些讨论分散在几个邮件列表和最近的Python论坛的[Ideas分类](https://discuss.python.org/c/ideas/6)中。这种脱节的部分可能是因为D'Avella在考虑“匿名函数”，而Python圈子的讨论通常围绕着`lambda`，因为这是语言中现有的机制。人们怀疑他使用术语`lambda`进行搜索会更成功，因为多语句的`lambda`是一个真正的匿名函数，正如他[同意的](https://discuss.python.org/t/why-not-real-anonymous-functions/44513/27)。

更多关于先前讨论的链接可以在Mike Miller的[帖子](https://discuss.python.org/t/why-not-real-anonymous-functions/44513/38)中找到。他指出的一个[线程](https://mail.python.org/archives/list/python-ideas@python.org/thread/KTRI2NOWA2YGFHOHQ2DFVD2O4UBU5ADY/#XS6P33QMBPRZ2C563OPTWG7NJRZ2K7VZ)导向了Alyssa Coghlan的[2009年的帖子](https://mail.python.org/archives/list/python-ideas@python.org/message/GJ7Z73UVWTVUJDILWWAIMH3VMMTH424O/)，Eric V. Smith认为这篇文章“<q>总结了这个提案的主要问题</q>”。Coghlan说：

> 但是对于Python来说，具有匿名块的想法与使用缩进作为块分隔符的使用相冲突。你必须要么为匿名块内部的语句想出一个非缩进的语法，实质上是发明一种全新的语言，要么在表达式内嵌入大量空白，而不仅仅在语句之间有空白。
> 
> 没有人提出比仅仅命名块并使用普通的def语句来创建它更好的解决方案。

对于史密斯来说，在解决这个问题之前，继续寻找没有意义：

> 我知道人们会有不同意见，但对我而言，如果语法问题得到解决，那么我将欢迎“生成由多个语句组成的函数的表达式”，通常称为多行lambda函数。
> 
> 但是如果没有解决语法问题的解决方案，整个论点都毫无意义。

D'Avella [试图](https://discuss.python.org/t/why-not-real-anonymous-functions/44513/45)提出一些该功能的可能语法，自然地，依赖于匿名块周围的定界符。他使用了`do`和`end`，但其他人尝试过括号或方括号，所有这些都因为不“Pythonic”而搁浅，至少对某些人来说是这样。与此同时，Clint Hepner [认为](https://discuss.python.org/t/why-not-real-anonymous-functions/44513/52)一些人似乎只是因为它们在JavaScript中经常使用——并且至少受到一些人的喜欢——所以他们希望在Python中也能有匿名函数。D'Avella则持魔鬼式辩护者的态度，[想知道](https://discuss.python.org/t/why-not-real-anonymous-functions/44513/53)这种想法有什么问题；毕竟，多种不同的Python功能毕竟也是来自别处。但Brendan Barnwell [说](https://discuss.python.org/t/why-not-real-anonymous-functions/44513/60)那些功能之所以进入Python，并不仅仅因为它们在其他地方存在，而是因为它们在该语言中是有用的：

> 它们被添加是因为另一种语言为*Python*提供了一些功能上的灵感，这可能在编写*Python*代码时很有帮助。不同语言之间的思想交流非常棒，但每种语言都会选择与其现有结构相契合的特性。

为了试图阻止此类未来提案，Neil Girdhar [建议](https://discuss.python.org/t/why-not-real-anonymous-functions/44513/62)在[Python维基](https://wiki.python.org/moin/FrontPage)中添加一个页面，简单描述[提议的更改](https://discuss.python.org/t/why-not-real-anonymous-functions/44513/65)，其利弊，并链接到先前的讨论。尽管有人怀疑这样的页面是否真的会有所帮助，他创建了[一个常见建议功能页面](https://wiki.python.org/moin/CommonIdeas)，并将[一个新的多行lambda页面](https://wiki.python.org/moin/MultiLineLambda)链接到了其中。与此同时，D'Avella的[尝试](https://discuss.python.org/t/why-not-real-anonymous-functions/44513/80)吸引核心开发人员赞助他将要写的PEP ("<q>带着它会被拒绝的预期</q>") 似乎毫无进展。

有一些关于该功能可能用例的示例，包括来自[Chris Angelico](https://discuss.python.org/t/why-not-real-anonymous-functions/44513/59)和[另一个来自"Voidstar"](https://discuss.python.org/t/why-not-real-anonymous-functions/44513/98)，但Cornelius Krupp [指出](https://discuss.python.org/t/why-not-real-anonymous-functions/44513/100)，再次强调，不是缺少用例。用例需要足够引人注目，以克服其他语法上的异议。而D'Avella [同意](https://discuss.python.org/t/why-not-real-anonymous-functions/44513/104)，但他注意到在该线程中关于这个功能的正当性存在疑问。Krupp [说道](https://discuss.python.org/t/why-not-real-anonymous-functions/44513/105)，语法只是第一个障碍：“<q>即使人们找到一个好的语法，说服Python社区和核心开发人员认为这实际上是一个好主意还有相当多的工作要做。</q>”

Python以使用空白符而不是分隔符来结构化其代码而闻名，难以想象这会改变——永远。Python本身将乐意同意：

```
    >>> from __future__ import braces
      File "<stdin>", line 1
    SyntaxError: not a chance

```

与此同时，一个名称的成本，在本地范围内，很可能非常低；生成的代码的可读性可能也会更好。当然，在“注定失败”的提案上工作可能会有一定的价值。尽管可能很难激励人们去工作。该主题无疑会再次被提起，正如Krupp [指出的](https://discuss.python.org/t/why-not-real-anonymous-functions/44513/79)，在过去几年中至少已经提出了三次。

然而，在Python世界中，这是一个持久性的问题；人们提出新想法，或者是一再讨论过的想法，却并不真正想要付出努力将其完成。有时，这项工作在于寻找先前的讨论，并展示为何那时提出的反对意见已不再有效——例如，展示现有代码的示例，也许是来自Python标准库，可以改进。但正如我们以前所看到的那样，不止一次，人们常常对他们的想法如此着迷，以至于惊讶于它并没有因为其明显的优越性而简单地一扫所有的异议。然而，在一个历史悠久的语言或其他项目中，想法在结果之前需要付出大量工作；支持者们最好记住这一点。

* * *

(

[登录](https://lwn.net/Login/?target=/Articles/964839/)

发表评论)
