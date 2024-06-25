<!--yml

category: 未分类

日期：2024-05-27 14:30:29

-->

# 一亿行挑战 - Gunnar Morling

> 来源：[`www.morling.dev/blog/one-billion-row-challenge/`](https://www.morling.dev/blog/one-billion-row-challenge/)

*更新 1 月 4 日：哇，这事情真的火了！* *1BRC 在互联网上的几个地方都有讨论，包括[Hacker News](https://news.ycombinator.com/item?id=38851337)、[lobste.rs](https://lobste.rs/s/u2qcnf/one_billion_row_challenge)和[Reddit](https://old.reddit.com/r/programming/comments/18x0x0u/the_one_billion_row_challenge/)。*

*非常感谢所有的投稿，这远远超出了我的预期！* *由于参赛作品数量庞大，我有点拖后腿，我会一点一点地逐个评估。* *我还对挑战的[规则](https://github.com/gunnarmorling/1brc#faq)做了一些澄清，请务必在提交任何作品之前阅读它们。*

让我们以真正的编程者风格开始 2024 年—我很兴奋地宣布[One Billion Row Challenge](https://github.com/gunnarmorling/onebrc)（1BRC），从 1 月 1 日持续到 1 月 31 日。

如果你决定接受这个任务，你的任务看起来很简单：编写一个 Java 程序，从文本文件中检索温度测量值，并计算每个气象站的最小、平均和最大温度。只有一个小问题：这个文件有**10 亿行**！

文本文件具有简单的结构，每行一个测量值：

```
 |  
```

1

2

3

4

5

6

```

 |  
```

汉堡;12.0

布拉瓦约;8.9

巴邻港;38.8

圣约翰;15.2

克拉科夫;12.6

...

```

 | 
```

该程序应该按照字母顺序打印出每个站点的最小、平均和最大值，如下所示：

```
 |  
```

1

```

 |  
```

{阿巴=5.0/18.0/27.4, 阿比让=15.7/26.0/34.1, 阿贝谢=12.1/29.4/35.6, 阿克拉=14.7/26.4/33.1, 亚的斯亚贝巴=2.1/16.0/24.3, 阿德莱德=4.1/17.3/29.7, ...}

```

 | 
```

1BRC 挑战的目标是为这个任务创建最快的实现，同时探索现代 Java 的优势，并找出你可以推动这个平台的程度。所以抓住你的（虚拟）线程，利用 Vector API 和 SIMD，优化你的 GC，利用 AOT 编译，或者使用任何其他你能想到的技巧。

1BRC 的参与规则很简单（更多详情见[这里](https://github.com/gunnarmorling/onebrc#running-the-challenge)）：

+   任何提交必须用 Java 编写

+   可以使用通过[SDKMan](https://sdkman.io/)可用的任何 Java 分发，以及来自[openjdk.net](https://openjdk.net)的早期访问构建，包括 Valhalla 等 OpenJDK 项目的 EA 构建

+   不得使用外部依赖

要参加挑战，请从 GitHub 克隆[1brc 存储库](https://github.com/gunnarmorling/1brc)，并按照 README 文件中的说明操作。有一个非常[基本的实现](https://github.com/gunnarmorling/1brc/blob/main/src/main/java/dev/morling/onebrc/CalculateAverage_baseline.java)任务，您可以将其用作比较的基准，并确保您自己的实现输出正确结果。一旦您对自己的工作满意，请对上游存储库打开拉取请求，以将您的实现提交给挑战。

所有提交将通过在[Hetzner Cloud CCX33](https://www.hetzner.com/cloud)实例上运行程序进行评估（8 个专用 vCPU，32 GB RAM）。使用`time`程序来测量执行时间，即端到端时间。每个参与者将连续运行五次。最慢和最快的运行将被丢弃。剩下三次运行的平均值是该参与者的结果，并将被添加到[排行榜](https://github.com/gunnarmorling/onebrc#results)中。如果您有任何疑问或想讨论任何潜在的 1BRC 优化技术，请加入 GitHub 存储库中的[讨论](https://github.com/gunnarmorling/1brc/discussions)。

至于奖品，通过参加这个挑战，你可能会学到新东西，激励他人，并为看到自己的名字列在上面的排行榜上感到自豪。有传言说获胜者可能会得到一件独一无二的 1️⃣🐝🏎️ T 恤。

所以不要等了，加入这个挑战，看看 Java 有多快——我真的很好奇社区会为这个挑战想出什么。Happy 2024, coder style!
