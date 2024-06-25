<!--yml

类别：未分类

日期：2024-05-27 14:48:45

-->

# Chris's Wiki :: blog/programming/GitBranchesSocialConstructs

> 来源：[`utcc.utoronto.ca/~cks/space/blog/programming/GitBranchesSocialConstructs`](https://utcc.utoronto.ca/~cks/space/blog/programming/GitBranchesSocialConstructs)

在 Fediverse 上，[我提出了一个半成品的论点](https://mastodon.social/@cks/111449110724408233)：

> 一个半成品的论点：Git 中的分支是一种社会构建，部分是由技术特性所赋能的。我们谈论某些事情是在分支上完成的，或者存在于分支上，或者在它们交织的树上是什么分支，即使这并不是你能在 Git 存储库中找到的东西。
> 
> （这是因为提交并没有永久地与一个分支关联起来；它们只是当前可以从一个或多个分支到达的。多头可达的提交在哪个分支上取决于我们。）

这其中的背景或多或少可以追溯到[Julia Evans](https://social.jvns.ca/@b0rk/111445767832607539)的[git 分支：直觉与现实](https://jvns.ca/blog/2023/11/23/branches-intuition-reality/)，更确切地说是 Julia Evans 和 [Mark Dominus](https://mathstodon.xyz/@mjd/111446192179078717)（以及 Mark Dominus 的[I wish people would stop insisting that Git branches are nothing but refs](https://blog.plover.com/prog/git/branches.html)）之间的 Fediverse 讨论。

这与我长期以来的观点有关，即现代版本控制系统是其底层数学的用户界面。Git 对 '分支' 的内部数学视图，但很少有人实际使用这种数学视图；相反，我们使用各种 '用户界面' 视图来思考分支以及如何看待它们。Git 通过各种 '美观的' 特性支持这些用户视图。

一些使用 Git 的项目积极努力地创建具有更具体和持久存在的分支。例如，[Go 发布分支上的提交在提交标题中带有分支名称](https://go.googlesource.com/go/+log/refs/heads/release-branch.go1.21)，这是 Go 项目为在 Go 开发和发布中使用的许多分支所做的事情。对于开发分支，这可持久地标记了提交作为在分支上完成的，即使分支已经合并到 'main' 开发分支中。

当然，我通常对 Git 分支的思考方式与它们的技术存在不同，并且分支之间也有所不同。例如，在一个典型的存储库中，我认为 'main' 分支一直延伸到存储库创建的地方，但其他分支只延伸到它们从 'main' 分支分离的地方，尽管这在技术上并不正确。

（Git 分支有点社会构建的另一个迹象是您可以重命名它们（按照评论）。）

PS：在其他版本控制系统中，分支在版本控制历史中具有更持久的存在。这些版本控制系统既不对也不错；我的观点是，它们对“分支”在数学版本控制中的界面和数学概念有了不同的看法。
