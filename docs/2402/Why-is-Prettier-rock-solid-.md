<!--yml

category: 未分类

date: 2024-05-27 15:02:36

-->

# 为什么 Prettier 如此稳固？

> 来源：[https://mrmr.io/til/prettier/](https://mrmr.io/til/prettier/)

### 为什么 Prettier 如此稳固？

我一直想知道为什么 Prettier 这么棒。是的，它从`gofmt`学到了要做什么（请注意，`gofmt`不缩进，这是难点）。是的，它是在现实生活中的会议上宣布的，这有助于它获得最初的动力。

这一切都很好，但让我感到难以理解的是它的质量。

+   你可能不喜欢默认设置（我喜欢它们。唯一不同意的是 tabWidth，不应该是 2 而是 4），

+   或许你会发现它太慢了，

但就功能而言，它是**稳固的**。到目前为止，它从未破坏过我的 JS / TS 代码。在 JavaScript 库的荒野中，这样高质量的库是罕见的。实际上，Prettier 本身的后续添加，比如用于 Prettier 的 MDX 格式化器，存在巨大的缺陷，使它们在处理非琐碎内容时变得不合理的风险。那么，Prettier 的核心格式化程序 - HTML / CSS / JS / TS - 为什么都如此稳固呢？

今天，我得到了答案。这是 Prettier 的原始作者写的[关于它的内容](https://archive.jlongster.com/How-I-Became-Better-Programmer)：

> Prettier 就是一个完美的例子。我知道我想要什么，但我不知道如何实现它。经过一番研究，我找到了*这篇论文*，几天后我就清楚地知道自己需要做什么了。一周内我就有了基本的工作。如果我忽视了之前的研究，那就要花费更多的时间了。

你知道它链接到哪篇论文吗？

[一位很赞的印刷家，Philip Wadler](http://homepages.inf.ed.ac.uk/wadler/papers/prettier/prettier.pdf)！

> 乔伊斯·基尔默和大多数计算机科学家都同意：没有什么比一棵树更可爱的诗了。

哈斯克尔人会认出这篇论文和作者。当然，这并不是贬低 Prettier 的作者和多年来的工作成果 - 我只是在谈论即使我不使用它时，Haskell 仍然如何丰富我的生活。
