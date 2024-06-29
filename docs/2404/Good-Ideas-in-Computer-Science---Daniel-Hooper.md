<!--yml

category: 未分类

date: 2024-05-27 13:38:45

-->

# 计算机科学中的好理念 ・ Daniel Hooper

> 来源：[https://danielchasehooper.com/posts/good-ideas-in-cs/](https://danielchasehooper.com/posts/good-ideas-in-cs/)

程序员喜欢为他们喜欢的技术争论。C++ vs Rust. Mac vs PC. 这些争论掩盖了计算机科学的胜利——我们都同意的理念。为了发掘这些理念，我最近在Twitter/X上[提出了一个简单的问题](https://twitter.com/DanielcHooper/status/1778795850107424827)：

> 计算机科学中哪些理念是被普遍认为是好的？

“普遍认为是好的”指的是它们没有被争议。这些理念是如此广泛和有效，以至于你可能甚至不认为它们是被发明的。每个理念在所有情况下可能并不适用，但你不会找到一个程序员认为你应该*从不*使用它们。我有意地专注于*理念*，而不是*实现*。例如：Unix 包含许多好的理念，但它不在列表中，因为它是一种实现。

这是我的列表，包括每个理念出现的年份：

故意排除：

+   垃圾收集

    有许多团队的例子表明，为了达到性能目标，他们与垃圾收集器作斗争^(。[CPU/内存性能差距](http://gec.di.uminho.pt/discip/minf/ac0102/1000Gap_Proc-Mem_Speed.pdf)需要对内存进行控制，以编写高性能代码。)

+   数据库

    数据库不只是一个概念，有很多种方式将这些概念组合成“数据库形状”。数据库中的一些好主意：[结构化查询语言](https://zh.wikipedia.org/wiki/SQL)，[B树](https://zh.wikipedia.org/wiki/B树)，[ACID事务](https://zh.wikipedia.org/wiki/ACID)。

+   其他数据结构和算法

    这些理念太多了，很少有像数组和哈希表这样普遍存在于几乎所有程序中的理念。

+   面向对象编程

    有一大群程序员认为面向对象编程不好^(。我推荐使用[data oriented design](https://github.com/dbartolini/data-oriented-design)作为一种替代的世界观。)

到了1974年，也就是50年前，我们已经拥有了现代计算的大部分内容。今天的基础原理与当时相同——一个1974年的C程序员在现代计算机上可能感到宾至如归，除了外星般的速度。我希望在50年后我们会有新的理念，这些理念将被普遍认为是好的。

在[Twitter上讨论](https://twitter.com/DanielcHooper/status/1782446647047311466)

在[Lobste.rs上讨论](https://lobste.rs/s/kruxyr/good_ideas_computer_science)
