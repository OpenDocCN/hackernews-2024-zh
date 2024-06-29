<!--yml

category: 未分类

date: 2024-05-27 14:37:50

-->

# “缺失”的图数据类型在70年代被发明了

> 来源：[https://tylerhou.com/posts/datalog-go-brrr/](https://tylerhou.com/posts/datalog-go-brrr/)

*本文是对[Hillel Wayne](https://www.hillelwayne.com/post/graph-types/)的文章《寻找缺失的数据类型》（[HN](https://news.ycombinator.com/item?id=39592444)) 的回应/灵感来源。建议先阅读他的文章。*

为什么编程语言缺乏内置的图和图算法支持？或者说，为什么没有图的“数据类型”？Hillel Wayne在上述链接的帖子中认为有几个原因：

1.  有太多类型的图。

1.  有太多图算法。

1.  即使您专注于特定类型的图表，也有许多可能的不同图表表示方式。

1.  您选择的图表表示方式对性能有很大影响。

我同意所有这些理由。但我不同意Wayne的（暗示的）结论——即图在本质上太复杂，不能被主流编程语言很好地支持。编程语言**可能会**拥有**令人惊奇的**图支持！也许十年后？并且只有在大量研究工作之后？（稍后详述...）

我认为现代编程语言的命令式/结构化编程模型不适合图算法，**这是当前编程语言难以支持图的根本原因。**正如Wayne正确指出的那样，核心问题在于，当您在诸如Python或Rust之类的命令式语言中编写图算法时，您必须为图选择某种显式表示。然后，您的遍历算法依赖于您选择的表示方式。如果后来发现您的表示不再高效，那么为新的表示调整您的算法将是一项艰巨的工作。

如果我们就这样，不选择图的表示方法会怎样呢？如果我们能以声明方式表达我们的图和图算法，并让计算机为我们选择表示方法，会怎么样？这样的程序是否同样高效？这些问题是反问的吗？

答案当然是肯定的。**我们已经有一种声明性编程语言，其中表达图算法非常自然**—[Datalog](https://en.wikipedia.org/wiki/Datalog)，其语义基于关系代数，这是在上世纪70年代开发的。例如，计算图的可达性是一个两行的Datalog程序，并且（在Datalog的扩展中）非常容易计算更复杂的事实，如单源或所有对最短路径。Datalog的实现也非常高效，可以根据数据变化调整查询策略，一些引擎甚至支持高级功能，如实时流/增量视图维护。缺点呢？你得写Datalog。

## Datalog

在 Datalog 中，将图编码为*边关系*（一组边）是自然的。像连通性这样的图查询通过对该边关系的*递归连接*得到很好的表达。（事实上，图中的传递闭包在 Datalog 中表达得如此自然，以至于通常用作[教授 Datalog 的第一个例子！](https://berkeley-cs294-248.github.io/lectures/unit9.pdf#page=37)）这里是这样一个程序的具体例子：

```
edge(1, 2).
edge(2, 3).
edge(3, 4).
 path(a, b) :- edge(a, b).
path(a, b) :- path(a, c), edge(c, b).
```

行1-3将图编码为边关系。特别是这个图是一个（有向）三节点的“链表”图，其中顶点1到顶点2有一条边，顶点2到顶点3有一条边。

用通俗的语言说，第5行和第6行表示：可能有两种方式从<mjx-container class="MathJax" jax="SVG"></mjx-container>到<mjx-container class="MathJax" jax="SVG"></mjx-container>的路径。要么是从<mjx-container class="MathJax" jax="SVG"></mjx-container>直接到<mjx-container class="MathJax" jax="SVG"></mjx-container>的边（第5行），要么是从<mjx-container class="MathJax" jax="SVG"></mjx-container>到另一个顶点<mjx-container class="MathJax" jax="SVG"></mjx-container>的路径，并且从<mjx-container class="MathJax" jax="SVG"></mjx-container>直接到<mjx-container class="MathJax" jax="SVG"></mjx-container>的边（第6行）。这两个规则将重复评估，直到数据库达到*固定点*；即，进一步应用任一规则不再导致更改。

这是如何评估的？首先，数据库从仅包含边开始：

<mjx-container class="MathJax" jax="SVG" display="true"></mjx-container>

尽管 Datalog 规则可以以任意顺序触发，让我们假设它们按程序顺序触发。第5行的规则触发，为每条边添加路径到我们的数据库：

<mjx-container class="MathJax" jax="SVG" display="true"></mjx-container>

要执行第6行，数据库引擎在共同变量<mjx-container class="MathJax" jax="SVG"></mjx-container>上*连接*`edge`和`path`关系。换句话说，等效的 SQL 语句可能是：

```
SELECT path.from, edge.to FROM path INNER JOIN edge ON path.to = edge.from;
```

这是连接的结果。

<mjx-container class="MathJax" jax="SVG" display="true"></mjx-container>

并且 Datalog 引擎将这些新元组添加到数据库中。请注意，在触发第6行规则之前，我们的数据库中有长度为<mjx-container class="MathJax" jax="SVG"></mjx-container>的路径；现在我们有长度为<mjx-container class="MathJax" jax="SVG"></mjx-container>的路径。

<mjx-container class="MathJax" jax="SVG" display="true"></mjx-container>

我们再次触发第5行的规则，但它不会改变我们的数据库。最后，我们触发第6行的规则，得到连接。

<mjx-container class="MathJax" jax="SVG" display="true"></mjx-container>

只有最后一个条目是新的，所以我们将其添加到我们的数据库中：

<mjx-container class="MathJax" jax="SVG" display="true"></mjx-container>

现在程序已经完成了，因为执行任何一个规则都不会改变数据库。请注意，我们不仅计算出了从顶点1到顶点4的路径，正如预期的那样，还计算了每对顶点之间是否存在路径。你可以在Egglog中看到[这个程序的运行情况](https://egraphs-good.github.io/egglog/?example=path)，Egglog是Datalog的一个扩展。

这个程序也很容易增强。例如，如果我们想支持无向传递闭包，那只需添加一个额外的规则即可，该规则表示如果存在从<mjx-container class="MathJax" jax="SVG"></mjx-container>到<mjx-container class="MathJax" jax="SVG"></mjx-container>的路径，则存在从<mjx-container class="MathJax" jax="SVG"></mjx-container>到<mjx-container class="MathJax" jax="SVG"></mjx-container>的路径。

```
path(a, b) :- path(b, a).
```

在Egglog（以及总体上的Datalog）中，你可以表达比图上的传递闭包更强大的程序。例如，Egglog还支持代数数据类型，这意味着路径关系可以用[路径的“证明”](https://egraphs-good.github.io/egglog/?example=pathproof)进行注释。如果每条边的权重已知（`edge(a, b, w)`），那么Egglog还可以计算最短路径；请看[Egglog论文第6页的例子](https://arxiv.org/pdf/2304.04332.pdf#page=6)。

实际上，近年来使用Datalog计算[程序分析](https://en.wikipedia.org/wiki/Program_analysis)的趋势越来越明显。这是因为大多数程序分析算法涉及遍历程序图。这些遍历通常是相互递归的：例如，为了确定一个变量是否指向一个对象，程序需要遍历控制流图——但是为了遍历控制流图，分析需要解析虚拟方法，这就需要进行指向分析！Datalog使得以声明方式表达复杂的相互递归图算法变得更加容易。

## 性能

那么性能如何？当然，用Rust手写的图遍历算法在原始速度上会超过数据库引擎吧？你可能会感到惊讶。当程序分析被重写以使用Datalog时，Datalog实现运行得更快。

[Doop](https://dl.acm.org/doi/10.1145/1639949.1640108)，一个用Datalog编写的指向分析框架，比使用Java编写的前任框架（此外还使用了诸如[二叉决策图](https://en.wikipedia.org/wiki/Binary_decision_diagram)之类的复杂技术）快了一个数量级。最近，[Egglog](https://arxiv.org/abs/2304.04332)在与其前身Egg相比的等式饱和性能上快了一个数量级，Egg是用Rust手写的。

数据库引擎如何如此之快呢？首先，上述Datalog程序的执行模型已经为了清晰起见进行了简化。实际上，有技术使得Datalog的运行比上面展示的“天真”执行快得多。其中主要的技术是[半天真评估](https://pages.cs.wisc.edu/~paris/cs838-s16/lecture-notes/lecture8.pdf#page=2)和[需求转换](https://dl.acm.org/doi/abs/10.1145/1836089.1836094)/[魔术集](https://dl.acm.org/doi/10.1145/28659.28689)。

其次，（正如Wayne所观察到的）图的最有效表示和最有效的遍历算法取决于图的结构。这种依赖于数据的优化正是数据库最擅长的。在数据库系统中，关于如何最佳表示数据集（[非]聚集索引，高效的内存数据结构）以及在这些数据集上查询（连接排序，连接类型）已经进行了数十年的研究。

这些相当不错地对应了图的高效表示方法：例如，邻接表本质上就是边关系的数据库索引！如果你的边关系足够小，那么使用（哈希）连接和内存索引在数据库引擎上进行连接是合理的。这本质上与命令式语言中的正常内存图遍历相同——只不过数据库连接可能会优化得更好。

如果一个图非常大但稀疏，数据库可以选择合并连接关系，这是因为其可预测的内存和磁盘访问模式而具有高吞吐量。如果它非常大但密集，可能会将连接编码为[GPU上的矩阵乘法……？](https://arxiv.org/abs/2311.02206) 还有各种先进的连接技术（最坏情况最优连接：[VAAT](https://berkeley-cs294-248.github.io/lectures/hung-ngo-wcoj.pdf#page=22)，[IAAT](https://berkeley-cs294-248.github.io/lectures/hung-ngo-wcoj.pdf#page=45)），在某些图上性能甚至更好。

复杂的数据库引擎可以根据数据随时间变化调整其存储和查询策略，这一点不用多说。如果你的图在开始时很稀疏，但随着时间变得更加密集，数据库系统可以持续收集你的图的统计数据，并在合适时刻切换到更好的表示形式。当一切正常运作时，这不需要程序员额外的工作。（而且通常数据库会提供调优选项，如果选择的表示或连接顺序不合适的话。）

尽管如此，Datalog仍然面临一些性能挑战。首先，许多支持如此复杂查询优化和花哨连接算法的数据库都不是开源的，这对广泛采用构成了一道障碍。其次，Datalog评估的一个主要假设是数据库必须单调增长——也就是说，它永远不会“删除”事实。这对内存受限系统（例如[遍历巨大文件系统或搜索数万亿顶点](https://www.hillelwayne.com/post/graph-types/#:~:text=Everyone%20I%20talked,of%20a%20thousand.)）构成了挑战，但原则上这些问题可能通过更多的研究来解决。

## 太棒了！除了“编写Datalog”这一部分。

如果Datalog如此出色，为什么它没有得到更多的采纳？

简而言之，Datalog在学术界和一些行业应用之外相对狭隘，因此从“软件工程”角度来看，并不是一种很好的语言。对于习惯于命令式代码的程序员来说，编写Datalog程序很困难，而编写和理解大型Datalog程序也很困难。

幸运的是，近年来在如何使Datalog在日常语言中更易于使用的研究领域有了进展。[Flix](https://flix.dev/)语言最初是作为[扩展Datalog以支持程序分析中的格](https://dl.acm.org/doi/10.1145/2980983.2908096)被创建，并且现在类似于一种命令式语言，还[natively支持Datalog约束和固定点计算](https://doc.flix.dev/fixpoints.html)。将Datalog与传统编程语言“融合”的优势在于，你可以用Datalog表达所有图遍历/固定点算法，希望与传统编程无缝交互。

另一篇研究论文，[《Functional Programming with Datalog (2022)》](https://drops.dagstuhl.de/entities/document/10.4230/LIPIcs.ECOOP.2022.7)，探索了一种将Datalog整合到传统语言中的不同方法：作者们并非将Datalog嵌入传统语言，而是将（简单的）函数式编程语言*转换为*Datalog。这种技术有点激进；在高层次上，他们（滥）用需求转换来“控制”Datalog评估的流程（传统上Datalog规则可以以任何顺序进行评估，但函数式语言通常具有由控制流定义的评估顺序）。

而且还有其他论文我没有提到（[Slog](https://arxiv.org/abs/2211.11573)），但如果我继续列举先前的工作，我只会重复整个[本学期在伯克利提供的声明性程序分析与优化课程的阅读清单](https://inst.eecs.berkeley.edu/~cs294-260/sp24/)，所以我停止。

所有这些的要点是：我认为未来（主流？）某种编程语言很可能会严肃支持**Datalog**。而且这不仅仅是因为Datalog使递归图遍历易于表达——近年来在数据库系统中还有[大量关于增量视图维护的激动人心的研究](https://arxiv.org/abs/2203.16684)。想象一下，编写一个复杂的图遍历（或其他某种可以递增的算法），然后获得高效的增量更新*免费*。

总结一下，图的数据模型——关系和关系代数——确实是在70年代发明的。但最近这个领域有了令人兴奋的进展：Datalog被程序分析社区采用，联接算法的最新进展（[WCOJ](https://dl.acm.org/doi/10.1145/3180143)），增量视图维护的简单框架（[DBSP](https://arxiv.org/abs/2203.16684)），以及总体上对Datalog的重新关注。数据库霸主们向我展示了光明，而光明表明Datalog非常牛。
