<!--yml

类别：未分类

日期：2024-05-27 13:22:52

-->

# 宣布 git bisect-find - Kevin Cox

> 来源：[https://kevincox.ca/2024/05/19/git-bisect-find/](https://kevincox.ca/2024/05/19/git-bisect-find/)

<main class="kevincox-content">

发布于2024-04-19

这是我为了补充[`git bisect`](https://git-scm.com/docs/git-bisect)而编写的一个小工具。`git bisect`是一个很棒的工具。在最基本的用法中，您提供一个“好”的提交（一个尚未包含某些属性的提交）和至少一个“坏”的提交（一个已经包含它的提交）。`git bisect`然后将引导您通过搜索，选择测试的提交，从而将搜索空间减半。这使您能够有效地识别具有特定功能的第一个提交。

然而，`git bisect`的一个前提是你知道一个良好的提交。通常情况下，你会注意到一个 bug，但还不知道一个良好的版本。它是在最近的发布中出现的问题吗？还是新问题？也许过去几个发布版本中曾经出现过问题而未被注意到？当然，您可以开始检查各种修订版本，看看是否存在 bug，但为什么要手动做这些呢？

这就是[`git bitsect-find`](https://gitlab.com/kevincox/git-bisect-find)的用武之地。它只需要一个不良提交，然后开始查找第一个良好的提交。基本用法如下：

首先检查您的不良提交：

然后使用`git bisect-find bad`开始进行二分搜索。

您将被要求启动常规的`git bisect`运行，输入`y`。

然后将检出一个新的候选提交。如果候选提交是不良的，则运行`git bisect-find bad`，然后将检出一个新的候选提交。重复测试和标记过程，直到找到一个良好的提交。

一旦找到一个良好的提交，您就可以使用`git bisect-find`完成。只需运行`git bisect good`，然后继续常规的二分搜索过程。

更完整的文档可以在[README](https://gitlab.com/kevincox/git-bisect-find/-/blob/master/README.md)中找到。还有更多功能可以添加（如自动二分搜索）。但基本功能已经对我非常有用，所以我想分享一下。

它的工作原理并没有太特别的地方。每次跳回两倍远（遵循第一父级）。所有状态均通过常规的`git bisect`进行记录，因此一旦找到一个良好的提交，您的所有`git bisect-find bad`（和`git bisect-find skip`）命令都会像常规的`git bisect bad`和`git bisect skip`一样被记住。

我确实在想，如果跳回两倍会不会更优。显然，直接跳到第一个提交会减少`git bisect-find`步骤。但会导致更多的`git bisect`。不过`git bisect`效率相当高。（更不用说第一个提交可能并不特别有趣。）在某种程度上，我应该弄清楚哪种增长模式可以减少总步数。不过我认为最佳步数也取决于目标提交的年龄分布。但无论如何，对于这个工具来说，两倍的跳跃足够有效，对我非常有帮助。

* * *

</main>
