<!--yml

类别：未分类

日期：2024-05-27 15:02:50

-->

# 妈妈测试

> 来源：[`graphite.dev/blog/the-mom-test`](https://graphite.dev/blog/the-mom-test)

*Graphite 目前是一家蓬勃发展的公司，开发了一个由成千上万顶级开发人员使用的代码审查平台。但是，像许多开发工具创业公司一样，它并非一开始就是这样。*

*在这篇文章中，我想聚焦于一本特定书籍中的智慧，这本书一直是我学习的基石：*[*妈妈测试*](https://www.google.com/search?q=the+mom+test+goodreads&rlz=1C5CHFA_enUS877US877&oq=the+mom+test+goodrea&gs_lcrp=EgZjaHJvbWUqBwgAEAAYgAQyBwgAEAAYgAQyBggBEEUYOdIBCDMyNTdqMGo5qAIAsAIA&sourceid=chrome&ie=UTF-8)*。这本书对于任何涉足 DevTools 创业、开源贡献或创建面向开发者产品的人来说都是一本简短而重要的读物。*

**注意**

Greg 每天花费全天时间撰写关于工程实践和开发工具的每周深度探讨。这得益于这些文章帮助宣传了 Graphite。如果你喜欢这篇文章，请今天就试试[Graphite](https://graphite.dev?utm_source=blog-note)，并开始提高 30% 的交付速度吧！

## **从 Airbnb 到创业者**[](/blog/the-mom-test?utm_source=weeklyfoo&utm_medium=web&utm_campaign=weeklyfoo-16&ref=weeklyfoo#from-airbnb-to-startup-founder)

在 Graphite 工作之前，我曾是 Airbnb 的基础设施工程师。在那里，我学到了无数的经验教训，这些教训帮助我成为了一名优秀的软件工程师。然而，我后来会发现，当从零开始构建东西时，**你构建得有多好几乎不重要**。更重要的是**选择正确的构建对象**。这个认识在我踏上创业之路，共同创立 Graphite 时尤为明显。

## **两次转变**[](/blog/the-mom-test?utm_source=weeklyfoo&utm_medium=web&utm_campaign=weeklyfoo-16&ref=weeklyfoo#two-pivots)

我们的团队一直对软件质量和发布流程充满热情。我和我的联合创始人建立的第一个原型是一个“漏洞捕获”工具。尽管我们将原型交到了朋友、LinkedIn 联系人甚至是在网上随机遇到的工程师手中，但反响立刻令人失望。我们转而开发了下一个产品，围绕着 iOS 回滚，并面临着相同的挑战。

老实说，除了少数几个用户之外，没有人关心我们在构建什么。我们开始询问为什么 — 这就是“妈妈测试”出现的地方。

## **在产品开发中接受“妈妈测试”**[](/blog/the-mom-test?utm_source=weeklyfoo&utm_medium=web&utm_campaign=weeklyfoo-16&ref=weeklyfoo#embracing-the-mom-test-in-product-development)

如果我们事先问了正确的问题，我们本可以节省几年的辛劳。这听起来可能很傻，但提出好问题确实非常非常困难。

《妈妈的测试》这本书解释了解决方案——它是关于用一种方式提出问题，从而获取真实、公正的反馈，即使是那些天生支持你的人，比如你的母亲。当我分享我们的 iOS 回滚概念时，通常的反应非常积极，但事后想想，这可能只是人们对我好。

当然，我的朋友会告诉我，“iOS 回滚听起来是个好主意！”但我应该问的是：

+   “上一次搜索回滚 iOS 应用是什么时候？”

+   “你尝试过在线上的答案吗？”

+   “你是一位出色的工程师，告诉我你在这里如何想出一个解决方案。”

+   “鉴于你已经想出了一个解决方案，你还会时不时地 Google 搜索是否有更好的方法吗？”

关键是要提出更深入的问题，询问实际行为，比如是否有人曾经积极寻求过类似你的解决方案。此外，软件工程师作为一个用户群体，是世界上解决自己问题能力最强的人之一。如果他们没有尝试过自己编写解决方案，那么这个问题一开始就不是那么大吗？

## **现有与不存在的解决方案**[](/blog/the-mom-test?utm_source=weeklyfoo&utm_medium=web&utm_campaign=weeklyfoo-16&ref=weeklyfoo#existing-vs-nonexistent-solutions)

在为开发者开发产品时，确保检查一个关键细节：解决方案是否已经存在？

如果一个解决方案不存在，必定有两种情况之一。要么这个想法没有解决足够大的痛点（比如我们要解决的回滚和错误捕获的痛点），要么存在一个根本的原因，这个想法迄今为止还*不能*存在。内存数据库一直是一个好主意，但直到 RAM 的廉价使这个想法解锁。聊天机器人很棒，但受限于大型语言模型的进步。

如果你想要追求打造一个之前从未存在的开发工具，清楚地知道为什么现在才能够构建它，并准备好与许多刚刚被解锁的竞争对手进行竞争。

一个被技术进步解锁的很好的例子是*MemSQL*。

> *2013 年 4 月 23 日，SingleStore 将其首个通用版本的数据库作为 MemSQL 公开发行*。[*[9]*](https://en.wikipedia.org/wiki/SingleStore#cite_note-appdeveloper-9)* 早期版本只支持 *[*行导向*](https://en.wikipedia.org/wiki/Column-oriented_DBMS#Row-oriented_systems)* 表，并且高度优化，适用于所有数据能够存入 *[*主内存*](https://en.wikipedia.org/wiki/Computer_memory)* 的情况。这一设计基于一种观念，即 RAM 的成本将会随着时间呈指数级下降，这与 *[*摩尔定律*](https://en.wikipedia.org/wiki/Moore%27s_law)* 类似的发展趋势。这最终将允许数据库系统的大多数用例将其数据存储在内存中。
> 
> https://en.wikipedia.org/wiki/SingleStore

另一种（我认为更好的）情况是，工具已经存在，但存在一些明显的缺陷。你经常可以找到一个开源项目、一个博客或一个内部的大公司工具与你的想法相匹配。这可能是你最好的验证之一，表明人们对问题的关注程度足以在以前尝试构建它。你需要弄清楚的唯一一件事是：用户*是否仍然*在这个工具周围感到痛苦？你怎样才能做得更好呢？

*PagerDuty*是这种情况的一个很好的例子：

> *我们花了‘09 年的第一个月来构思想法和做研究。我们构思想法的一种方式是考虑到一些更大的公司内部建立的内部工具（比如亚马逊，我们三个人以前都在那里工作过），其他大小的公司都会需要这些工具……亚马逊建立了一个内部工具来处理*[*呼叫调度*](https://support.pagerduty.com/docs/first-schedule)*和通过传呼机进行警报。这个工具是建立在他们内部的故障票务和监控系统之上的，所以当检测到关键问题时，就会通知到正确的人员……做了一些研究后，我们意识到不仅仅是亚马逊建立了一个内部工具来值班—谷歌和 Facebook 都建立了他们自己的版本。这似乎有一个明确的需求。”*
> 
> https://www.pagerduty.com/blog/decade-of-duty/

就 Graphite，我们的第三次也是最后一次的转变而言，这个工具也已经存在。[堆叠的差异和替代代码审查平台早已被发明](https://jg.gg/2018/09/29/stacked-diffs-versus-pull-requests/) 并且在像谷歌和 Facebook 这样的大公司中备受喜爱。[Phabricator](https://www.phacility.com/phabricator/) 和 [Gerrit](https://www.gerritcodereview.com/) 是开源的。用户积极寻找解决方案，[编写脚本](https://github.com/ejoffe/spr)，尝试自己托管等等。每个月都会有[有人在 Twitter 上发推文要求 GitHub 原生支持堆叠的差异](https://twitter.com/acdlite/status/1255640994239766528?lang=en)。与此同时，用户渴望更多。这是一个新的开发工具的绝佳机会。

\为什么我们的前两次尝试都没有获得成功，而我们的第三次成功了呢？显然不是因为我们的工程质量一夜之间翻了一番；它一直保持稳定。答案是，与我们以前的两个想法不同，人们确实在我们解决的问题上感到痛苦。

## **开发工具的试金石测试**[](/blog/the-mom-test?utm_source=weeklyfoo&utm_medium=web&utm_campaign=weeklyfoo-16&ref=weeklyfoo#the-litmus-test-for-dev-tools)

从根本上说，Graphite 以一种我们先前的想法没有的方式通过了母亲测试。如果我们清晰地、公正地问了以下问题，我们就可以从一开始就选择正确的产品来构建：

+   “你上次搜索这样的工具是什么时候？”

+   “你试过安装你找到的东西吗？”

+   “告诉我你在这里拼凑出来的一个脚本。”

+   “自从匆匆搭建一个脚本以来，你上一次还 google 了更好的解决方案是什么时候？”

历史上通过了这个测试的想法有哪些？PagerDuty。合并队列。度量仪表板。Slack 机器人。CI 测试运行器。清单还在继续。如果它在大公司存在，但在外部不存在，那是一个很好的起点。如果没有任何公司内部构建过它，你可能需要检查一下你的玫瑰色眼镜。

## **结束语**[](/blog/the-mom-test?utm_source=weeklyfoo&utm_medium=web&utm_campaign=weeklyfoo-16&ref=weeklyfoo#closing-thoughts)

多年来工作于 Graphite，The Mom Test 的见解对我来说仍然是无价的。整个团队在寻求真正的、未经过滤的反馈时每周都会参考它，确保 Graphite 专注于开发能够满足真正需求的功能。我特别推荐这本书给任何从事产品开发领域的人，尤其是在 DevTools 领域。它不仅仅是问出正确问题的指南，更是理解和满足用户真实需求的路线图。
