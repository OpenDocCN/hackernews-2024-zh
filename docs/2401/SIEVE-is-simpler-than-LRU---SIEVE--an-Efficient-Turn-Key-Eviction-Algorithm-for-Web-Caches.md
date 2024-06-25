<!--yml

类别：未分类

日期：2024-05-27 14:28:53

-->

# SIEVE 比 LRU 更简单 - SIEVE：Web 缓存的高效即插即用淘汰算法

> 来源：[https://cachemon.github.io/SIEVE-website/blog/2023/12/17/sieve-is-simpler-than-lru/](https://cachemon.github.io/SIEVE-website/blog/2023/12/17/sieve-is-simpler-than-lru/)

# SIEVE 比 LRU 更简单

缓存是一种存储临时数据以便快速访问以保持在线世界运行平稳的方法。但是有限的空间意味着需要做出关键决策：保留什么，丢弃什么。这就是**淘汰算法**发挥作用的地方。我们的团队最近设计了一种名为**SIEVE**的新的缓存淘汰算法：它非常有效且简单，只有一个队列。

***更新***：*原有的写作风格引起了相当大的关注。我们已经更新了博客文章，使其更加清晰和直接。* 我们还在 [https://observablehq.com/@1a1a11a/sieve-miss-ratio-plots](https://observablehq.com/@1a1a11a/sieve-miss-ratio-plots) 中包含了一个易于重现的结果。

## 简单性的重要性

保持缓存淘汰算法简单是很重要的。复杂的算法有时会带来头痛。例如，当未命中率高时，它们可能很难调试和分析。此外，复杂性意味着需要 CPU 周期来做出决策。

另一方面，简单的淘汰方法通常非常出色，提供了良好的可预测性和吞吐量。例如，Apache traffic server 使用 FIFO，[MemC3](https://www.usenix.org/conference/nsdi13/technical-sessions/presentation/fan) 使用 CLOCK，[Segcache](https://www.usenix.org/conference/nsdi21/presentation/yang-juncheng) 使用 FIFO-Merge 淘汰算法。需要注意的是，尽管这些方法在吞吐量方面表现出色，但在缓存效率方面可能会做出一些妥协。

## 认识 SIEVE

SIEVE 是一个缓存淘汰算法，它决定在缓存中保留什么，丢弃什么。它既实现了简单性又实现了效率。

### 设计

**数据结构。** SIEVE 只需要一个队列和一个称为“hand”的指针。 **队列在对象之间维护插入顺序**。队列中的每个对象使用一位来跟踪已访问/未访问的状态。手指指向缓存中的下一个淘汰候选对象，并从队列尾部移动到队列头部。

**SIEVE 操作。** 缓存命中将访问对象的访问位更改为 1。对于已经设置了访问位的热门对象，SIEVE 不需要执行任何操作。在缓存未命中时，SIEVE 检查手指指向的对象。如果它已经被访问过，则将访问位重置，并且手指移动到下一个位置（保留的对象保留在队列的原始位置）。它会一直进行这个过程，直到遇到访问位为 0 的对象并将其淘汰。淘汰后，手指指向下一个位置（队列中的上一个对象）。虽然被淘汰的对象大部分时间都在队列的中间，但新对象总是插入到队列的头部。

SIEVE 的示意图

**SIEVE vs. CLOCK** 乍一看，SIEVE 与 CLOCK/Second Chance/FIFO-Reinsertion 相似 - *请注意，它们是相同淘汰算法的不同实现*。每个算法维护一个队列，其中每个对象都与一个访问位相关联，以跟踪其访问状态。访问过的对象在淘汰时被保留（也称为“幸存”）。值得注意的是，在 SIEVE 和 FIFO-Reinsertion 中，新对象都是插入到队列的头部。然而，SIEVE 中的指针会随着时间从尾部移动到头部，而 FIFO-Reinsertion 中的指针会一直停留在尾部。*关键区别在于保留对象的位置*。SIEVE 将其保留在原来的位置，而 FIFO-Reinsertion 将其插入到头部，与新插入的对象一起。

SIEVE vs. FIFO-Reinsertion

请参阅本博客文章末尾的 [sieve cache implementation code](#sieve-cache-code) 以获取详细示例。

### 性能

我们评估 SIEVE 来回答以下问题：

+   SIEVE 是否比最先进的缓存淘汰算法效率更高？

+   SIEVE 是否比其他算法更简单？

+   SIEVE 能否提高缓存的吞吐量和可伸缩性？

在本博客文章中，我们只展示了部分评估结果。如果您对完整的评估结果感兴趣，请参阅[我们的论文](https://yazhuozhang.com/assets/pdf/nsdi24-sieve.pdf)。

#### 效率

我们的评估涉及来自不同数据集的 1559 个跟踪，包含 2,470,017 亿次请求到 14,852 亿个对象，结果显示，SIEVE 在超过 45% 的跟踪上表现优于所有最先进的淘汰算法。

下图显示了不同算法在跟踪数据中错失率（与 FIFO 相比）的减少。箱线图上的须根据 p10 和 p90 进行定义，使我们能够忽略极端数据并集中在典型情况上。SIEVE 在几乎所有百分位数上都显示出最显著的减少。例如，SIEVE 在 10% 的跟踪数据上（顶部须）将 FIFO 的错失率减少了超过 42%，在最大的 CDN 数据集中的平均值为 21%。相比之下，其他所有算法在此数据集上的减少都较小。与先进算法相比，例如 ARC，SIEVE 的错失率最多可以减少 63.2%，平均值为 1.5%。

#### 简单性

SIEVE 非常简单。我们深入研究了五种不同编程语言中最流行的缓存库和系统：C++、Go、JavaScript、Python 和 Rust。

尽管在这些库中实现 LRU 的方式各不相同 - 有些选择双链表，其他选择数组 - 但集成 SIEVE 很容易。正如表中所示，将 LRU 替换为 SIEVE 所需的代码修改量很小。在所有情况下，代码修改量都不超过 21 行（不包括测试）。

#### 吞吐量

除了效率之外，吞吐量是缓存系统的另一个重要指标。虽然我们已经在五种不同的库中实现了 SIEVE，但我们专注于 Cachelib 的结果。

与基于 LRU 的算法相比，SIEVE 不需要在每次缓存命中时进行 "提升"。因此，它更快速、更可伸缩。在单线程上，SIEVE 比经过优化的 LRU（TwoQ）快 16%（17%），并且在测试的跟踪数据上。在 16 个线程上，SIEVE 的吞吐量比经过优化的 LRU 和 TwoQ 高出 2 倍以上。

### SIEVE 超越了一种淘汰算法。

除了作为缓存淘汰算法之外，SIEVE 还可以作为设计更先进的淘汰策略的缓存原语。作为缓存原语，SIEVE 可以促进更先进的淘汰算法的设计。

我们已经将 SIEVE 插入到 [LeCaR](https://www.usenix.org/conference/hotstorage18/presentation/vietri)，[TwoQ](https://www.vldb.org/conf/1994/P439.PDF)，[ARC](https://www.usenix.org/conference/fast-03/arc-self-tuning-low-overhead-replacement-cache) 和 [S3-FIFO](https://dl.acm.org/doi/10.1145/3600006.3613147) 中，用 SIEVE 一个替换它们的 LRU 或 FIFO 队列。

与 SIEVE 相比，LeCaR 的效率要低得多；然而，当将 LeCaR 中的 LRU 替换为 SIEVE 时，平均减少了 4.5% 的 LeCaR 的错失率。TwoQ 和 ARC 的效率接近 SIEVE；然而，当将 LRU 替换为 SIEVE 时，两个算法的效率都得到了提升。这些结果突显了 SIEVE 作为强大的缓存原语设计高级缓存淘汰算法的潜力。

### SIEVE 不是防扫描的。

除了 Web 缓存工作负载外，我们还对一些旧的块缓存工作负载进行了评估。然而，SIEVE 有时会显示比 LRU 更高的未命中比例。造成这种差异的主要原因是 SIEVE 不具备抗扫描性。在块缓存工作负载中，频繁出现扫描，流行对象通常与扫描对象混合在一起。因此，插入后两个对象都会被迅速逐出。请注意，我们犯了一个错误，并将“抗扫描”用于表示其他含义。对于传统意义上的扫描（大于缓存大小），SIEVE 是抗扫描的。

由于 SIEVE 不使用幽灵缓存——一个跟踪最近被逐出项的影子缓存，以便做出更智能的未来逐出决策——当流行对象再次被请求时，它无法识别。在缓存大小较大时，这个问题不太严重，但是当缓存大小较小时，我们观察到拥有幽灵是抗扫描的关键。

[马克最新的博客文章](https://brooker.co.za/blog/2023/12/15/sieve.html)探讨了通过为每个项目添加一个小计数器来使筛选器抗扫描的方法。它展示了在不同工作负载下的一些收益和损失。

## 我们期待听到您的声音

随着我们完成这篇博客文章，我们想要向开源和共享迹象的人们和组织致以最诚挚的感谢。SIEVE 提供了一个引人入胜的机会来探索和增强网络缓存的效率。**如果您有问题或想法，或者已经尝试过 SIEVE，请务必与我们联系：）**

## 附录

SIEVE Python 实现

```
`class Node:
 def __init__(self, value): self.value = value self.visited = False self.prev = None self.next = None   class SieveCache:
 def __init__(self, capacity): self.capacity = capacity self.cache = {}  # To store cache items as {value: node} self.head = None self.tail = None self.hand = None self.size = 0   def _add_to_head(self, node): node.next = self.head node.prev = None if self.head: self.head.prev = node self.head = node if self.tail is None: self.tail = node   def _remove_node(self, node): if node.prev: node.prev.next = node.next else: self.head = node.next if node.next: node.next.prev = node.prev else: self.tail = node.prev   def _evict(self): obj = self.hand if self.hand else self.tail while obj and obj.visited: obj.visited = False obj = obj.prev if obj.prev else self.tail self.hand = obj.prev if obj.prev else None del self.cache[obj.value] self._remove_node(obj) self.size -= 1   def access(self, x): if x in self.cache:  # Cache Hit node = self.cache[x] node.visited = True else:  # Cache Miss if self.size == self.capacity:  # Cache Full self._evict()  # Eviction new_node = Node(x) self._add_to_head(new_node) self.cache[x] = new_node self.size += 1 new_node.visited = False  # Insertion   def show_cache(self): current = self.head while current: print(f'{current.value} (Visited: {current.visited})', end=' -> ' if current.next else '\n') current = current.next   # Example usage cache = SieveCache(3) cache.access('A') cache.access('B') cache.access('C') cache.access('D') cache.show_cache()` 
```
