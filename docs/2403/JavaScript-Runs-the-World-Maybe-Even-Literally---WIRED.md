<!--yml

category: 未分类

date: 2024-05-27 14:36:05

-->

# JavaScript Runs the World—Maybe Even Literally | WIRED

> 来源：[https://www.wired.com/story/javascript-runs-the-world-maybe-literally/](https://www.wired.com/story/javascript-runs-the-world-maybe-literally/)

莱克斯·弗里德曼在他的流行播客上做了许多长时间的采访。尽管如此，与传奇程序员[约翰·卡马克](https://www.youtube.com/watch?v=I845O57ZSy4)的一集给人一种不羁的导演剪辑感。在五个小时内，卡马克涉及从向量操作到*Doom*的一切内容。但真正让这个延长播放时间合理化的是弗里德曼无意中说的一句话：“我认为如果我们生活在[模拟中](https://www.wired.com/story/living-in-a-simulation/)，那么它是用 JavaScript 写的。”

要点回顾：JavaScript 是使静态网页“动态化”的关键。没有它，互联网可能会变得像一个下班后的街机厅一样，毫无生气和黑暗。如今，这种语言被广泛用于前端和后端开发，支持诸多移动平台和应用，包括 Slack 和 Discord。在弗里德曼的宅文化禅题背景下，理解它的主要要点是：对于任何自尊心强的程序员来说，承认自己*喜欢* JavaScript 简直是一种社交失误，就像一位艺术电影导演承认自己喜欢漫威电影一样。

我想这可能与以下事实有关：JavaScript 的诞生时间比自家酿一罐冲饮茶还短，只用了 10 天。1995 年，网景雇佣了一位名叫布兰登·艾克的程序员，创建了一种可以嵌入其浏览器 Netscape Navigator 的语言。最初被称为 LiveScript，这种语言后来改名为 JavaScript，借助当年引入的与之无关的语言 Java 的热度。至今，很少有人认为 JavaScript 是一种特别设计良好的语言，艾克本人尤其不以为然。“我在 1995 年创造了 JavaScript，从那时起就一直在弥补过失。”他曾经说道。

他到底犯了什么罪？你可以轻松找到大量关于批评 JavaScript 的博客文章、表情包和 Reddit 帖子，但我最喜欢的是软件工程师 Gary Bernhardt 的一个 [四分钟演讲](https://www.destroyallsoftware.com/talks/wat) ，名为 "Wat"。想象一下，首先向一群非英语为母语的人展示 *boil* (*boil*/*boiled*) 和 *chew* (*chew*/*chewed*) 这样动词的现在和过去形式。然后，当你询问他们 *eat* 的词形变化时，谁能怪他们回答 *eat*/*eated* 呢？类似地，"Wat" 演讲是 JavaScript 怪癖和不可预测行为的一个彩蛋合集。假设你想对一组数字进行排序：[50, 100, 1, 10, 9, 5]。在任何理智的语言中调用内置的排序函数会以数字递增的顺序返回列表：[1, 5, 9, 10, 50, 100]。但在 JavaScript 中这样做会返回 [1, 10, 100, 5, 50, 9]，其中 10 和 100 被认为比 5 大。为什么呢？因为 JavaScript 将每个数字解释为字符串类型，进行词法排序而不是数字排序。简直疯了。

当弗里德曼说 JavaScript 主宰世界时，换句话说，他的意思是我们的世界就像底层源代码一样，混乱不堪且令人难以理解。这相当于叹息地宣称，考虑到地球的糟糕状态，人权宣言恐怕是用 Comic Sans 写成的。

在这一点上，我应该承认，虽然 JavaScript 不是我最喜欢的语言，但我喜欢它。事实上，我非常喜欢它。所以每当某些程序员兄弟会对它进行激烈的辩论时，我不由得感到一丝不满。他们经常集中在多年前已经解决的缺陷上。纠结于 JavaScript 最初的缺点，就是忽视了任何软件片段——每种编程语言本质上都是一套软件——都可以接受修订和改进的事实。
