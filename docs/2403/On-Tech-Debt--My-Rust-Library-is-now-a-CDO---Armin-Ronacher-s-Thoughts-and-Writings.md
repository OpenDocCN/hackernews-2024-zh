<!--yml

category: 未分类

date: 2024-05-29 12:41:49

-->

# [技术债务：我的Rust库现在是一个CDO | Armin Ronacher的思想和文章](https://lucumr.pocoo.org/2024/3/26/rust-cdo/)

> 来源：[https://lucumr.pocoo.org/2024/3/26/rust-cdo/](https://lucumr.pocoo.org/2024/3/26/rust-cdo/)

# [关于技术债务：我的Rust库现在是一个CDO](https://lucumr.pocoo.org/2024/3/26/rust-cdo/)

written on Tuesday, March 26, 2024

你可能熟悉技术债务。有一个笑话说，如果有技术债务，肯定会有衍生品来处理这些债务吧？我很高兴地说，Rust生态系统已经创造了一个环境，在这个环境中，处理技术债务的一个解决方案看起来是抵押化。

这就是这个奇迹是如何起作用的。假设你有一个库[stuff](https://github.com/mitsuhiko/insta)，它依赖于另一个库[learned-rust-this-way](https://github.com/chyh1990/yaml-rust)。learned-rust-this-way的作者在某个时候对这个项目失去了兴趣，问题继续积累。其中一些问题是功能请求，另一些是合法的错误。然而，作为写下stuff的人，你从未遇到过这些问题。然而很难说learned-rust-this-way不是技术债务。这并不会太困扰你，但它毕竟是债务。

有一天，其他人发现了learned-rust-this-way是债务的一部分。其中一种发生这种情况的方式是因为名称很好。显然，不只有一个人是通过这种方式学习Rust，还有其他人也希望拥有那个名称。但原始作者联系不上了。因此，现在还有一个理由让该软件包[被添加到RUSTSEC数据库](https://github.com/rustsec/advisory-db/issues/1921)，突然间一切都乱套了。几分钟内，CI将开始对直接或间接使用learned-rust-this-way的许多人失败，通知他们发生了某些事情。这是因为RUSTSEC基本上是一个评级机构，他们决定你的债务现在是垃圾。

下一步会发生什么？作为stuff的维护者，你的用户突然开始指责你使用learned-rust-this-way，你受到了影响。压力水平增加。你得摆脱那个东西。为什么？不是因为它对你没用，而是有人指责你的债务。如果我们真的想强调财务术语，这就是你的保证金调用。你的用户要求采取行动来处理你的债务。

那么你能做什么？一种选择是转向替代品（卸载债务）。在这种特殊情况下，由于某种原因，替代learned-rust-this-way的所有选择似乎也不太理想。其中一种是该项目的一个分支，也只有一个维护者，但突然之间又引入了3个依赖项，其中一个已经有了“B-”评级。该生态系统中的另一个选项决定在被指出之前就[默认](https://github.com/dtolnay/serde-yaml/commit/3ba8462f7d3b603d832e0daeb6cfc7168a673d7a)。

记住，你从未积极地接触过learned-rust-this-way。它在过去四年的未维护状态下对你有效。如果你现在fork那个库（并将其命名为learned-rust-this-way-and-its-okay），你现在就受到相同的要求限制。fork那个库就是把现金放在债务堆上。除非你不处理那里的bug报告，否则最终你会像learned-rust-this-way一样被指责。所以虽然这可能会为你赢得时间，但并没有真正解决问题。

然而，以下才是真正有效的做法：你只需将那段代码合并到你自己的库中。现在那些垃圾技术债务突然被评为“AAA”级。只要你再也不碰那段代码，永远不向任何人透露你这样做了，并且继续像以前一样维护你的库，世界就会继续运转。

截至今日：我通过将yaml-rust作为insta的供应商，抵押了它。现在它成为insta代码和yaml-rust的融合体。通过这样做，我成功地将这笔垃圾技术债务升级为完美的AAA评级。

谁赢了？我想没有人真正赢。

至于标题：“CDO”是在2007年金融危机期间变得非常臭名昭著的一种金融工具。你可以在《大空头》中找到一个娱乐性的解释。

此条目被标记为[rust](/tags/rust/)和[thoughts](/tags/thoughts/)
