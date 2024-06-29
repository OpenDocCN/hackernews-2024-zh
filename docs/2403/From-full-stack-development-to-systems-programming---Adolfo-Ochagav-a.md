<!--yml

category: 未分类

date: 2024-05-29 12:39:28

-->

# 从全栈开发到系统编程 | 阿道夫·奥查加维亚

> 来源：[https://ochagavia.nl/blog/from-full-stack-development-to-systems-programming/](https://ochagavia.nl/blog/from-full-stack-development-to-systems-programming/)

<content>在攻读计算机科学学位期间，有一个让我忙得不可开交的大问题：毕业后我该干什么？多亏了大学里扎实的基础和我在现实世界中对Rust的贡献经验，我觉得自己可以做任何事情。那么...该如何选择呢？

## 深陷全栈开发

在阅读[《永远别一个人吃饭》](https://www.goodreads.com/book/show/84699.Never_Eat_Alone)后，我决定开始结交人际，以解答我的问题。我联系了朋友的朋友们，他们都是程序员，与互联网上的陌生人交谈，经常参加本地的编程聚会，甚至自己组织聚会。这些经历改变了我的关注点：我不再试图找到特定的技术领域来*工作*，而是开始寻找我愿意*一起工作*的人群。

答案在我硕士最后一年变得清晰：我决定加入[Utrecht的Infi](https://infi.nl/)，一家为新兴企业提供全栈开发服务的精品咨询公司。最初吸引我的是公司的氛围，我开玩笑地形容它像一个永无止境的聚会。人们对计算机充满了真正的好奇心^(，我有约20%的工作时间用于个人发展（从副业项目到开源贡献，再到公司埋单的哲学课程）.)

因为全栈开发对我来说有些新鲜（一切都是新的），我非常享受学习这门技艺的挑战。这包括人的方面，如与利益相关者对接，以我们团队适合的敏捷方式工作（而不是令人恐惧的自上而下的充满流行语的闹剧），进行双人编程会议，帮助新同事迅速上手等。这感觉就像是我应该工作的地方^(，即使我的大学和开源背景使我在工作中使用的技术根基比我在工作中使用的要深入。)

## 自雇

我在Infi的时光结束了，当我决定花更多时间做志愿工作时。您可以在[这里](https://ochagavia.nl/blog/becoming-a-contractor/)找到完整的描述，但最终我成为了自由职业者，并开始从事任何我能找到的项目。

起初我继续做全栈开发，但在2022年发生了意想不到的事情：旧金山的一家公司聘请我[实现MySQL的协议](https://ochagavia.nl/blog/implementing-the-mysql-server-protocol-for-fun-and-profit/)！这是我人生中第一次因为技术技能而得到报酬。我研究了一个文档不足的网络协议的复杂性，提出了一个稳健的实现，并将其开发成难以误用的库。

我感到惊讶。这种工作总是让我觉得自己达不到，像是为少数幸运的人预留的。事实上，这个项目之后，事情回归到了“正常”的开发工作，尽管这次更加注重后端。

## 系统编程突破

2023年是特别的一年。我的几篇与Rust相关的博客文章引起了[Prefix.dev](https://prefix.dev)的注意，他们为Conda生态系统开发程序员工具。他们聘请我[探索容器的深渊](https://ochagavia.nl/blog/crafting-container-images-without-dockerfiles/)，[加速他们开源库的开发](https://ochagavia.nl/blog/the-birth-of-a-package-manager/)，甚至[实现一个新的依赖解决器](https://ochagavia.nl/blog/the-magic-of-dependency-resolution/)。

同年，我还被[Stormshield](https://www.stormshield.com/)聘请，以增强Rust主要的[QUIC](https://en.wikipedia.org/wiki/QUIC)实现，^(后来被[ISRG](https://www.abetterinternet.org/)（以[Let’s Encrypt](https://letsencrypt.org/)闻名）聘请，创建[Rust主要TLS库的持续基准测试设置](https://ochagavia.nl/blog/continuous-benchmarking-for-rustls/)。)

这些日子我又回到了低级容器工作，现在是为[Outerbounds](https://outerbounds.com/)，旨在显著降低[Metaflow](https://github.com/Netflix/metaflow)“flows”的启动时间。剧透：结果*非常*有希望，我希望很快能发布一篇关于这次冒险的文章^。

可以想象，在所有这些项目之后，我越来越将自己视为[系统程序员](https://en.wikipedia.org/wiki/Systems_programming)。偶尔我还会做全栈开发，主要是作为一种爱好。我喜欢用简单的东西让最终用户开心的感觉。这也让我能解决自己的问题，并将结果程序包装成一个愉快的界面。更重要的是，这让我与更广泛的行业保持联系。

## 那人类的一面呢？

我现在做的工作非常注重研究。而且是远程工作。这意味着大量基于文本的沟通，没有机会与同事偶遇。这与我作为一个敏捷团队的全栈开发者的起点非常不同！不过，我非常喜欢这样的工作，并且在短期内（不论是全栈还是其他）都不愿意回到办公室工作。

在这种情况下，培养工作的人际交往方面现在主要取决于我的主动性。我努力写作，让同事感到我关心工作和他们；通过一对一视频通话，我努力了解与我一起工作的人；我经常与这个博客的读者和其他网络上的人保持联系。

基于个人兴趣交往比我在办公室里“被迫”采用的风格更符合我的性格。这也让我能与这个领域里*非常*有才华的人保持联系。我期待更多！

#### 奖金：您是专家吗？

最近，我的文章[潜行的通才](https://ochagavia.nl/blog/the-undercover-generalist/)在网络上走红。我的观点是，即使你是个通才，有时候出于营销原因把自己呈现为专家也是有道理的。在同一篇文章中，我把自己称为潜行的通才，但写完这篇博客文章后，我不太确定了！现在我真的想专注于系统编程……这让我觉得自己像个专家。有趣的是，[有人在回应《潜行的通才》时写道](https://gusvanhorn.blogspot.com/2024/02/marketing-isnt-only-reason-to-specialize.html)：*“专业化不仅是一种营销方式。它是成为通才的一部分，它必须从一个或多个特定领域的专业知识开始。”* 这是否意味着我既是通才*又*是专家？我想时间会告诉我们。

*在[HN](https://news.ycombinator.com/item?id=39817026)上评论。*
