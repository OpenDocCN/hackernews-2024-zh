<!--yml

category: 未分类

date: 2024-05-27 13:12:53

-->

# 我推荐一些程序员应该尝试的具有挑战性的软件项目列表 | andreinc

> 来源：[https://www.andreinc.net/2024/03/28/programming-projects-ideas](https://www.andreinc.net/2024/03/28/programming-projects-ideas)

在我们开始之前，我想澄清一下，我即将建议的项目想法主要是为那些有兴趣探索新知识领域的人准备的。然而，重要的是要注意，这些想法大多数可能与今天的职场不相关。如果你想为招聘目的添加一些令人印象深刻的内容，也许考虑其他选择会更值得。例如，不要再建立另一个React中的TODO应用（GitHub上有75k个仓库），你可能想专注于通过克隆一个知名网站来进行后端或前端开发。这些项目可能在知识获取和职业前景方面更有益。

话虽如此，一些开发人员对探索传统职场之外的创意领域更感兴趣。对于这些个人来说，编程更像是一种爱好而不是谋生手段。如果你属于这一类别，你可能会对以下项目建议更感兴趣。

你也可以检查关于这个主题的其他文章：

* * *

# 我的列表

* * *

# 较少人知的数据结构

例如，[Austin Z. Henley](https://austinhenley.com/)建议你自己编写[拓扑排序](https://en.wikipedia.org/wiki/Topological_sorting)、[递归下降树解析](https://en.wikipedia.org/wiki/Recursive_descent_parser)、[Bloom Filter](https://en.wikipedia.org/wiki/Bloom_filter)、[Piece Table](https://en.wikipedia.org/wiki/Piece_table)和[Splay Tree](https://en.wikipedia.org/wiki/Splay_tree)的实现。

事实上，许多计算机科学课程已经变得淡化。事实上，一些学校只教授基础知识，比如动态数组、链表、队列、栈和哈希表等。然而，除了这些基础概念之外，还有许多其他值得探索的数据结构和算法。

就我个人而言，我也会选择：

+   [B-Tree](https://en.wikipedia.org/wiki/B-tree) - 我肯定会实现这个。这是一个有趣的数据结构，用于数据库系统或文件系统。它还可以让你以新的方式思考如何改进应用程序中的内存和数据布局；

+   [循环缓冲区](https://en.wikipedia.org/wiki/Circular_buffer) - 这是一个优雅地解决生产者-消费者问题的数据结构，特别是当消费者（暂时）跟不上生产者时。

+   [Cuckoo Filter](https://en.wikipedia.org/wiki/Cuckoo_filter)和[Cuckoo Hash Table](https://en.wikipedia.org/wiki/Cuckoo_hashing) - *Cuckoo*方法的实用性尚待证明，但这绝对值得你花时间去做一些有趣的事情。简而言之，这些数据结构的背后的理念是有趣和创造性的。

+   [开放寻址哈希表](https://en.wikipedia.org/wiki/Open_addressing) - 大多数学校和课程都专注于分离链接技术，漠视了*开放寻址*的优雅方式。那么为什么不写几个开放寻址的实现并与现有的库实现进行比较呢？试着让你的实现更快，炫耀一下，同时学到东西。

# 编写一个分布式哈希表

如果你已经知道如何编写自己的*哈希表*，那么构建[*分布式哈希表*](https://en.wikipedia.org/wiki/Distributed_hash_table)并不是一项不可能的任务。虽然这似乎是一个复杂的项目，但并不一定要*生产就绪*或成为下一个 Redis。

项目结束时，你可能会更加熟悉网络编程和管理并发问题。

奖励：彻底测试你的 DHT 将是一段旅程。

^(如果你被卡住或者结果很糟糕，不要灰心！)

* * *

# 编写一个*科学*计算器

这是一个相对容易的项目，但值得一试*栈*数据结构的力量。你可以通过学习如何评估[RPN 表达式](https://en.wikipedia.org/wiki/Reverse_Polish_notation)并实现[Shunting Yard 算法](https://en.wikipedia.org/wiki/Shunting_yard_algorithm)来实现这一点。在此项目中，挑战自己学习一个以前未接触过的新 GUI 库。

一旦你有了一个可用的计算器，就开始探索一些疯狂的想法：

1.  实现你自己的[sin 函数](https://androidcalculator.com/how-do-calculators-compute-sine/)只是为了刺激一下。

1.  编写一个自己的大数库，以便处理大数运算。

1.  你的计算器能处理将`3`的`2.27`次方吗？

1.  编写一个模块，使用户能够操作矩阵，包括加法、乘法、计算逆矩阵、计算行列式和解线性方程组等。

1.  你的计算器能检查一个数是不是质数吗？

1.  你能写一个模块，给用户关于数字的随机见解吗？

它还可以在[浏览器](http://numcalc.com/)中工作。

* * *

# 用 C + POSIX 编写你自己的 HTTP 服务器

^(*首先，如果你还没有学习 C 的话，开始学习 C。与流行观点相反，早期学习 C 将使你在长远来看成为更好的程序员。我现在比以往任何时候都更加确信这一点。然而，现在不是支持我观点的时候或文章。*)

在实现 HTTP 协议时，请记住你不必涵盖所有内容。这个练习的目的不是为了写出最快的 HTTP 服务器。相反，你做这个是为了：

+   积累频繁处理`char*`所带来的挫折感。当然，你可以创建一个`char*`的抽象来解决这个问题，虽然可能会有 bug，但它将是你独特的作品！

+   学习`fork()`、`pthreads`以及你通常不必处理的其他低级知识，除非在操作系统课程中。

+   了解TCP和网络编程的工作原理。

如果提到C让你不悦，你可以用Rust重写这个项目。那将是*新的*。

* * *

# 编写一种奇怪的编程语言

^(*奇怪编程语言（有时缩写为esolang）是一种旨在测试计算机编程语言设计边界的编程语言，作为概念的证明、软件艺术、作为另一种语言（特别是函数式编程或过程式编程语言）的黑客界面，或作为一个笑话。 ([来源](https://en.wikipedia.org/wiki/Esoteric_programming_language))*)

这是一个让你释放创造力的副业项目。以下是一些帮助你入门的建议：

1.  你的语言可以有一个简单的语法，所以你不需要使用[解析生成器](https://en.wikipedia.org/wiki/Comparison_of_parser_generators)。自己写解析器。一个类似APL的语言，具有有限的*特殊图形符号*集，不是不可能实现或解析。

1.  你听说过[UIUA](https://www.uiua.org/)吗？

1.  尽管这看起来可能很难实现，但看看[Tsevhu](https://www.reddit.com/r/tsevhu/comments/iserji/tsevhu_key_activity/)和[Koilang](https://gammagames.itch.io/koilang)。Tsevhu不是一种编程语言，而是一种语言。你能用代码创造类似的东西吗？

1.  提到Koilang，你有没有看过[piet](https://www.dangermouse.net/esoteric/piet.html)？

1.  它也可以比[阿尔巴尼亚洗衣机](https://esolangs.org/wiki/Albanian_Laundry_Machine)更好笑。

欲获取更多灵感，请查看奇怪语言的[wiki](https://esolangs.org/wiki/Language_list)。

* * *

# 写你自己的虚拟机

我个人曾经应对过这个挑战（查看这篇[文章](/2021/12/01/writing-a-simple-vm-in-less-than-125-lines-of-c)）。然而，我后悔没有设计一套原创指令集，而是实现了[LC-3](https://en.wikipedia.org/wiki/Little_Computer_3)指令集。

你的项目可以是基于寄存器、基于堆栈，或者是混合的。如果你感觉勇敢，它甚至可以有一个JIT编译器。

不管你选择做什么，关键是要有创意。

例如，请看[uxn](https://100r.co/site/uxn.html)，它可以在多个操作系统或设备上运行，并且有一个小的[dans社区为其编写软件](https://github.com/hundredrabbits/awesome-uxn#applications)。甚至[tsoding](https://www.twitch.tv/tsoding)，我最喜欢的技术YouTuber之一，最近也[将康威生命游戏实现为uxn程序](https://www.youtube.com/watch?v=rTb6NFKUmQU)。

* * *

# 为UXN编写游戏

首先，了解[uxn](https://100r.co/site/uxn.html)的工作原理，可以通过阅读官方文档或者按照[这个优秀的教程](https://compudanzas.net/uxn_tutorial.html)来学习。

查看现有示例：

想出一个原创游戏点子。

似乎每个人都更喜欢贪吃蛇、俄罗斯方块、乒乓球和太空侵略者。但还有其他（现在被遗忘的）游戏值得你关注。

为什么不试着实现一些不同的东西：

* * *

# 为TIC-80编写一个游戏

[TIC-80](https://tic80.com/)是一个用于制作、玩耍和分享微型游戏的幻想计算机。

如果你不知道写什么游戏，可以从[这里](https://tic80.com/play)获得灵感。

这更多是一个艺术项目而不是编程项目，但仍然如此。

* * *

# 编写你自己的原创markdown语言

1.  编写一个不完全是markdown但有点异域风情的markdown语言。

1.  扩展现有的markdown实现。你可以从[LiaScript](https://liascript.github.io/)或[R Markdown](https://rmarkdown.rstudio.com/)中获得灵感。

* * *

# 编写一个静态站点生成器

是的，这很无聊，但有些东西需要使用新发明的markdown语言。

* * *

# 曼德勃罗集导航器

你不必是[亚瑟·C·克拉克](https://www.youtube.com/watch?v=5qXSeNKXNPQ)或者数学家才能欣赏[曼德勃罗集](https://en.wikipedia.org/wiki/Mandelbrot_set)的无用之美。

你有考虑过用HTML Canvas构建你自己的曼德勃罗集探索器吗？互联网上有大量的例子：

添加一些创意的元素！例如：

+   要使你的曼德勃罗集导航器独特，你可以整合一个智能坐标系统，帮助你在无限中航行。

+   此外，你可以添加一个书签功能，这样你就可以保存并重新访问你在探索过程中发现的有趣模式。例如，如果你偶然发现了[象谷](https://math.stackexchange.com/questions/2979906/are-there-any-unique-places-in-the-mandelbrot-set-that-have-not-yet-been-seen-gr)，你可以为将来参考而添加书签，并轻松与他人分享。

* * *

# 模拟物理中的各种现象

从光学开始；这可能更容易，而且互联网上有大量的例子：

如果你明白你在做什么，跳到其他区域：

从小事做起，逐渐成为[Bartosz Ciechanowski](https://ciechanow.ski/)。

* * *

# 尝试康威生命游戏

我想你已经熟悉[康威生命游戏](https://en.wikipedia.org/wiki/Conway's_Game_of_Life)了。

创造一些有创意的东西：

+   改变规则，添加更多状态，加入颜色、表情符号和动画；

+   将单元格放入迷宫中，添加传送门，让单元格传送；

+   编写一个由这个细胞自动机驱动的老虎机；

+   实时干预单元格，看它们如何反应，扔点肉给它们；

你可以做很多事情。

* * *

# 用多项式近似现实

如果你学过计算机图形学，你可能已经遇到了[贝塞尔曲线](https://en.wikipedia.org/wiki/B%C3%A9zier_curve)的概念。为什么不开始用它们来逼近*现实*？

一些[Pierre Bezier](https://en.wikipedia.org/wiki/Pierre_B%C3%A9zier)和[Jamiroquai](https://en.wikipedia.org/wiki/Jamiroquai)的粉丝已经玩过这个游戏：

也许现在是你写一个不同渲染器的时候了。

为什么不选择正弦和余弦而不是多项式？希望你明白[我在说什么](https://www.youtube.com/watch?v=-qgreAUpPwM)。

* * *

# 一个符号微分计算器

^(要理解我在说什么，请查看[StackOverflow上的这个问题](https://stackoverflow.com/questions/43455320/difference-between-symbolic-differentiation-and-automatic-differentiation)。)

这将是一个具有挑战性的项目，但并不像你想象的那么难。基本上，你需要想出一些WolframAlpha已经[能够做到的事情](https://www.wolframalpha.com/input?i=derivative++x%5E2*cos%28x-7%29%2F%28sin%28x%29%29)。

你将需要能够解析数学表达式。然后，你将不得不（递归地）应用特定的微分规则（例如[链式法则](https://en.wikipedia.org/wiki/Chain_rule)和[乘积法则](https://en.wikipedia.org/wiki/Product_rule)）。最后，你将需要*简化*所得到的表达式。

你不必实现所有的东西。

你将需要记住微积分的基础知识。

这将是有趣的。

* * *

# 一个老虎机

… 以科学之名。

我不会花太多精力在图形上。这并不是说你想要贡献于人们的不幸和成瘾。但是[老虎机背后的数学](https://www.youtube.com/watch?v=BFlRH99TQOw)可能会很有趣，而且你可以*发挥创意*：

+   为什么不写一个使用π的小数点来给奖品的π老虎机呢。

+   为什么不使用生命游戏槽机，你停在特定的细胞配置上并且给予奖品；

+   为什么不制作一个正弦槽机（你可以从我之前的一个项目[正弦俄罗斯方块](/2024/02/06/the-sinusoidal-tetris)中获取灵感）。

# 为文本游戏编写一个游戏引擎

^(这个想法是由@Snej在[lobste.rs](https://lobste.rs/s/5fsjpu/my_list_challenging_software_projects)上建议的)

我们这一代人中很少有人玩过*基于文本的*游戏，这很正常 - 我们需要更好地利用我们的硬件，而不是在终端中渲染字体。

但曾经有一段时间，像[Zork](https://playclassic.games/games/adventure-dos-games-online/the-hitchhikers-guide-to-the-galaxy/play/)或者[Colossal Cave](https://rickadams.org/adventure/advent/)这样的游戏非常流行。

那么，为什么不为基于文本的冒险游戏构建一个游戏引擎呢？

使引擎*跨平台* - 允许游戏在终端、浏览器或者[SDL](https://en.wikipedia.org/wiki/Simple_DirectMedia_Layer)窗口中运行。

或者仅仅保留终端。现在有许多优秀的TUI库，因此你不必因为被困在[curses](https://en.wikipedia.org/wiki/Ncurses)上而感到沮丧。

# 编写一个平铺窗口管理器

现在[Wayland](https://en.wikipedia.org/wiki/Wayland_(protocol))即将到来，我确信有很多*新*的创意空间。看看[sway](https://swaywm.org/)。

好的，编写一个[平铺窗口管理器](https://en.wikipedia.org/wiki/Tiling_window_manager)可能不是你能想到的最容易的项目。但同时，你可以保持简单。例如，XMonad在启动时大约有1000行代码。
