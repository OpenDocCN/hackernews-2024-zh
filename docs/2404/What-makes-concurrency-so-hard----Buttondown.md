<!--yml

category: 未分类

date: 2024-05-27 13:17:26

-->

# 为什么并发如此困难？ • Buttondown

> 来源：[https://buttondown.email/hillelwayne/archive/what-makes-concurrency-so-hard/](https://buttondown.email/hillelwayne/archive/what-makes-concurrency-so-hard/)

<date>2024年4月16日</date>

# 为什么并发如此困难？

## 是人类大脑的某种特性，还是问题域的某些因素导致了这种情况？

我的许多正式规范项目涉及并发或分布式系统。这正是“难以正确完成”和“错误的代价高昂”的甜蜜点，这导致人们在编写规范时花费时间和金钱。考虑到它对我的工作的相关性，我花费了大量时间思考并发的本质。

正如一个老笑话所说，"并发是计算机科学中最难的两件事之一。"存在许多“偶然”的原因：测试困难，[不可组合性](https://stefan-marr.de/2014/07/why-is-concurrent-programming-hard/)，bug可能长时间潜伏等。但有什么是 *本质上* 使它如此困难的？有什么使并发软件本质上比同步软件更难编写的因素？

我经常听到的原因是，人类思维是线性的，而不是并发的，因此不适合推理竞争条件。但我并不同意：根据我的经验，人类在并发推理方面非常擅长。每次开车时，我们都在进行并发推理！

更普遍地说，有些研究发现，如果你将[并发系统以人类术语](https://citeseerx.ist.psu.edu/document?repid=rep1&type=pdf&doi=77c3ef40423deeed3c73647df93f6f0ecdab2d69)（"[meatspace modeling](https://buttondown.email/hillelwayne/archive/i-am-a-sql-injection-attack-5019/)"）的方式进行框架化，人们很擅长发现竞争条件。因此，虽然并发可能难以推理，但我认为这并不是我们大脑的缺陷所致。

在我看来，一个更好的基础是**状态空间爆炸**。并发很难因为并发系统可以处于许多不同的可能状态，而且状态的数量增长速度远远超出了任何人的准备能力。

### 行为、交错执行和非确定性

考虑到代理人 ^({1, 2, … n}，每个代理人执行一个线性步骤序列。不同的代理人可能有不同的算法。例如，从队列中写入或读取，递增计数器等。第一个进程需要p1个原子步骤才能完成，第二个进程需要p2步，依此类推。代理人严格按顺序完成其程序，但另一个进程可以在每个原子步骤之后交错执行。如果我们有算法A1A2和B1B2，它们可以执行为A1B1A2B2或A1**B2B1**A2，但不是A1B1B2A2。)

在这些条件下，这里有一个描述可能发生的执行顺序（**行为**）数量的方程式：

```
`(p1+p2+...)!
------------
p1!*p2!*...` 
```

如果我们有三个代理同时执行相同的2步骤算法，那就有`6!/8 = 90`种不同的行为。序列中的每个步骤都有可能导致一个不同的状态，所以有6*90=540个最大的不同状态（MDS）。其中任何一个都有可能是“有错误”的状态。

在大多数情况下，实际的状态空间会显著小于MDS，因为不同的行为可能会跨越相同的状态。这个公式增长得非常*快*。三个3步骤的代理会给我们1700种可能的行为（15K MDS），而四个2步骤的代理则有2500种行为（20K MDS）。这一切都没有任何非确定性！如果代理中的一个步骤可以非确定性地执行三种操作之一（发送消息M₁，发送消息M₂，崩溃），那么这会使行为数和MDS都增加三倍。

复杂并发系统通常具有数百万或数千万个状态是非常常见的。即使是我在[模型优化文章](https://learntla.com/topics/optimization.html)中使用的玩具模型，也有大约 800 万个不同的状态（尽管经过一些工作，你可以将其大大降低）。我主要关注状态空间的性能，因为大的状态空间需要更长时间进行模型检查。但这也是为什么并发本质上难以推理的原因。如果我的理论MDS有200万个状态，那么我的实际状态空间只有其大小的1%，我的大脑可以推理出剩下状态的99.9%……这仍然会留下20个我忽略的边缘情况。

### 缩小状态空间

这是我经常使用的一个启发式算法：

> **所有使并发‘更容易’的方法首要关注的是管理状态空间。**

这并非百分之百正确（没有百分之百的启发式算法），但大约百分之六十正确，这已经足够了。状态空间增长迅速，更大的状态空间会引起问题。如果我们想要创建可维护的并发系统，我们需要从缩小空间开始。

比如看看线程。线程共享内存，并且线程调度器有很大的自由度来暂停线程。因此，你会有许多步骤（潜在地每行代码都可能是一个步骤），任何交错都会导致一个不同的状态。我可以使用像互斥锁和屏障这样的编程结构来"修剪"状态空间，以获取我想要的行为，但是考虑到状态空间有多大，我必须进行大量修剪以获得正确的行为。我在实现中可能会犯错误，"扭曲"空间（比如添加死锁），或者没有注意到需要移除的有错误的状态。线程非常容易出错。

相反，我可以切换到内存隔离进程。调度器仍然可以安排任何交错，但是进程不能混淆彼此的内部内存。从内部来看，进程仍在执行 A1A2A3A4，但如果唯一涉及外部资源（或进程间通信）的步骤是 A2 和 A4，那么我们只需考虑 A2A4 如何影响状态空间。三个四步骤代理有 35k 种交错方式，而三个两步骤代理仅有 90 种。这是一个很大的改进！

我们还能做什么？低级原子指令在一个步骤中执行更多操作，因此没有交错的余地。数据库事务需要很多物理时间，但只代表一个 [逻辑时间步骤](https://buttondown.email/hillelwayne/archive/physical-vs-logical-time/)。数据突变通过不可变数据结构的定义避免了新的步骤。

语言有构造来更好地修剪生成的状态空间：go 的通道，promises/futures/async-await，育儿园等等。我认为你也可以将 promises 视为强制执行 "非交错" 的一种方式：等到将来准备好完全执行（或到下一个 yield 点）并在执行之前。*请*不要引用我说过的话。

我 *认为* [CRDTs](https://en.wikipedia.org/wiki/Conflict-free_replicated_data_type) 通过允许交错来减少状态空间，但安排事物以便外部更改是可交换的：A1B1 和 B1A1 导致相同的最终结果，因此没有不同的状态。

再次强调，这只是一个 *非常* 粗略的启发式方法。

### 限制

初级近似法则，更小的状态空间 == 好。但这并没有考虑空间的拓扑结构：有些空间比其他空间更复杂。一个有很多不同循环的空间比一个无环的空间更难处理。^(不确定性的不同形式也很重要："代理继续或重新启动"比 "作者发送消息 M₁ 或 M₂" 导致更复杂的行为。这两种情况都是 "人类在推理并发性方面表现不佳" 再次出现的地方。我们通常通过将状态组减少到具有相同属性的 "状态等价类" 来处理并发性复杂的状态空间。复杂的状态空间有更多的等价类需要处理。）

（"人类在推理并发性方面表现不佳"的另一种方式可能是真实的：我们可能不会 *注意到* 有些事情是不原子化的。）

一些高级范式可以导致特定的状态空间拓扑，这些拓扑有较少的交错或者具有更多等效状态。我听说人们声称 [fork-join](https://en.wikipedia.org/wiki/Fork%E2%80%93join_model) 和 pipe-filter 特别容易让人理解，我理解为 "不会导致复杂的状态空间"。也许还有事件循环？演员模型在这一切中如何适应？

另一个限制：存在着有缺陷的*状态*和有缺陷的*行为*之间的区别。有些行为可以完全通过安全状态，但仍可能导致诸如“永不达到一致性”之类的 bug。这被称为“活性 bug”，我在[这里](https://www.hillelwayne.com/post/safety-and-liveness/)详细讨论了这个问题。活性 bug 要难推理得多。

好了，关于并发性的哲学胡言乱语就到此为止吧。并发很难，如果你感到困扰，不要难过，这不是你的错，而是组合数学的问题。

* * *

### 视频登场

我曾在 David Giard 的 *Technology and Friends* 上谈论 TLA+。在[这里](https://www.youtube.com/watch?v=DERER_0RKGws)查看！

如果你在网上阅读这篇文章，你可以在[这里](/hillelwayne)订阅。更新频率为每周一次。我的主要网站在[这里](https://www.hillelwayne.com)。
