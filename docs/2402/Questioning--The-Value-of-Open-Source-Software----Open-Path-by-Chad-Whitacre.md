<!--yml

category: 未分类

date: 2024-05-27 14:48:24

-->

# 质疑“开源软件的价值” ❧ Open Path by Chad Whitacre

> 来源：[https://openpath.chadwhitacre.com/2024/questioning-the-value-of-open-source-software/](https://openpath.chadwhitacre.com/2024/questioning-the-value-of-open-source-software/)

## 质疑“开源软件的价值”

简而言之，我认为新的 HBS 工作论文在某些方面基本上是错误的，尽管其中有一些有用的部分。

披露 / 广告：我为

[Sentry](https://sentry.io/welcome/)

.

哈佛商学院（HBS）最近发表了一篇名为《[开源软件的价值](https://papers.ssrn.com/sol3/papers.cfm?abstract_id=4693148)》的工作论文，由博士后 [Manuel Hoffmann](https://www.linkedin.com/in/manuel-hoffmann-b4798773/)、助理教授 [Frank Nagle](https://www.linkedin.com/in/frank-nagle/) 和研究生 [Yanuo Zhou](https://www.linkedin.com/in/yanuo-zhou/) 合著。Linux Foundation 赞助了这项研究，该研究基于 [Census II](https://www.linuxfoundation.org/research/census-ii-of-free-and-open-source-software-application-libraries) 项目。

这是 [Frank 的简介](https://www.linkedin.com/feed/update/urn:li:activity:7153860222812372993/)：

> 如果没有开源软件存在，公司将不得不比他们实际花费的软件费用多出 3.5 倍。更确切地说，我们的结果显示，如果社会必须替换这些软件包一次（例如，开源软件仍然存在，但所有最广泛使用的软件包被删除并需要重写），将花费 $4.15 billion；但如果每家使用这些软件包的公司都必须重写它们使用的每个软件包（例如，如果根本不存在开源软件），将花费 $8.8 trillion。

好吧，那么这就是三个主要的结果：

1.  如果没有开源软件存在，公司将比当前多花费 3.5 倍的软件费用。

1.  重建所有于 2020 年底存在的开源软件将耗费 $4.15 billion（Census II 背后的快照时间，如果我没记错的话）。

1.  如果每家公司分别重建其在 2020 年底使用的所有 OSS，将耗费 $8.8 trillion。

第一个结果来源于第三个。该工作论文估计，2020 年实际的软件支出为 $3.4 trillion，并且 3.4 + 8.8 = 12.2 trillion，这是实际支出的 3.5 倍（第 18 页）。第三个结果来源于第二个，通过估算每个 OSS 软件重建成本乘以使用该软件的公司数的估算（第 13 页）。因此，论证的流程是，在 2020 年底的快照给定的情况下：

1.  重建所有开源软件将耗费 $4.15 billion。

1.  每家公司重建其使用的所有 OSS 软件的总成本为 $8.8 trillion。

1.  公司为他们使用的所有软件花费了 $3.4 trillion，除了他们从开源软件中免费获得的内容。

1.  因此，如果没有开源软件存在，公司将不得不比他们实际上花费的软件费用多出 3.5 倍。

现在，我在这篇工作论文中发现了一些有趣的东西。论证的第一步（即第二个头条结果）在逻辑上看起来还行，因为它基于[COCOMO II](https://en.wikipedia.org/wiki/COCOMO)，这似乎是相当成熟的（虽然我确信在内部很混乱）。我学习了一些基本的经济术语（[存量和流量](https://en.wikipedia.org/wiki/Stock_and_flow)变量）。我了解了[GHTorrent](https://gousios.org/bibliography/G13.html)和[BuiltWith](https://builtwith.com/)。关于开源价值的商品市场分析附录似乎很有帮助。关于负责大部分开源软件生产的程序员数量的数据（第20页）非常有趣。我将不得不重新审视我在[我的上一篇文章](https://openpath.chadwhitacre.com/2024/the-open-source-sustainability-crisis/)中的假设，“全面可持续的开源社区将与大型技术公司相当”，即10,000至100,000名工程师。看来可能差一个数量级。我打算在[未来的一篇文章](https://github.com/chadwhitacre/openpath/issues/20)中做到这一点。

尽管如此，**我不接受头条结果1（“公司将花费3.5倍”）**，因为我认为结果3（“成本将达到8.8万亿美元”）是无关紧要的。

## 一个关键缺陷？

对我来说，论证的第二步似乎存在严重缺陷。如果是这样，结果3就是误导性的，结果1是错误的。获取软件的选项有买入、借用或自建。借用意味着使用开源软件。如果我们排除这个选项，公司仍然有两个选择：买入或自建。自建选项估计需要花费8.8万亿美元。这份工作论文将其称为“劳动市场估值方法”。

> 这个思想实验假设我们生活在一个不存在开源软件的世界，每家使用特定开源软件的公司都必须*重新创造*这个软件。（第12页，重点强调）

这是工作论文中的主要方法论，也是“3.5倍”头条结果的来源。

那么买入选项呢？如果开源软件停止存在，公司肯定会填补这个空白。作者确实探讨了这种选择。他们称之为“商品市场估值方法”。

> 通过商品市场方法，这个思想实验依然是我们生活在一个不存在开源软件的世界，但必须通过一家公司*重新创建*这个软件，然后对目前免费的商品收费。（第38页，重点强调）

对我来说，这比劳动市场方法更有价值，因为它更接近在没有开源软件的替代时间线中我们实际期望发生的情况。

与劳动市场方法的结果8.8万亿美元相比，商品市场方法的结果**低了四个数量级，仅为177百万美元**（第37页）。如果我理解正确，使用这个结果作为第2步（结果3）将使论点变得极其脆弱，而第1结果基本上消失了：3.4 + 0.000177 = 3.400177万亿美元。如果是这样，**公司将比它们当前的软件支出多花1.00005倍**，如果开源软件不存在的话。

工作论文在附录中包含了商品市场方法，但在摘要中并未报告结果。我建议要么更好地为使用劳动市场方法进行辩护，要么修改工作论文，使商品市场方法成为主要方法（要么放弃劳动市场方法，要么将其移到附录）。目前看来，将劳动市场方法作为这项研究的主要方法论似乎使得第1和第3结果毫无价值。

## 还有，Golang怎么了？

另一件在工作论文中引人注目的事情是决定纳入Go语言：

> 最后，对于某些分析，我们深入挖掘并展示了GitHub分类的前5种编程语言（2022年2020年的数据；我们数据所在年份）。前5种编程语言包括C（包括C#和C++）、Java、JavaScript、Python和TypeScript。我们还添加了Go语言到这些顶级语言列表中，因为它在开源软件中的使用越来越广泛。（第11-12页）

查看[引用的GitHub数据](https://octoverse.github.com/2022/top-programming-languages)，Go语言甚至未进入前10名。

为什么将Go语言纳入，而其他语言未纳入？给出的理由是“它是越来越广泛使用的开源语言”，这个解释并不令人满意。还有其他问题需要考虑，比如为什么将C#和C++合并到C中，而TypeScript没有合并到JavaScript中。尤其之所以特别引人注目的是包含Go语言，因为它在需求方分析中的影响显著偏高。

我在工作论文中没有找到标题结果与选择用于深入分析的语言之间存在直接联系的地方，就像图1中那样。我也没有找到工作论文背后的开放数据链接，因此无法自行进行调查。

## 结论

多年前，我同样尝试估算了 [开源软件的价值](https://gratipay.news/open-source-captures-almost-none-of-the-value-it-creates-9015eb7e293e)，这些估算后来用来 [指导](https://gratipay.news/your-company-should-probably-pay-2000-per-person-for-open-source-9205443e209d) 我对 [尝试](https://blog.sentry.io/we-just-gave-500-000-dollars-to-open-source-maintainers/) 解决 [开源可持续性危机](https://fossfunders.com/) 的努力。这类估算总是复杂的事务，涉及大量假设和泛泛而谈。然而，它们对于理解危机的性质和范围至关重要。我们需要在这个领域做出良好的工作。

我不接受 “[开源软件的价值](https://papers.ssrn.com/sol3/papers.cfm?abstract_id=4693148)” 工作论文的主要结论，即如果没有开源软件，公司将花费软件支出的3.5倍。根据我的阅读，如果没有开源软件，公司几乎不会多花一分钱。这也挑战了我自己的估算。我期待在 [未来的一篇文章](https://github.com/chadwhitacre/openpath/issues/20) 中深入探讨其含义。
