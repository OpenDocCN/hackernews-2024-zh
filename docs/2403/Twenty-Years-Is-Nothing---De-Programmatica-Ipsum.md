<!--yml

category: 未分类

date: 2024-05-27 14:36:29

-->

# 二十年不算什么 - De Programmatica Ipsum

> 来源：[https://deprogrammaticaipsum.com/twenty-years-is-nothing/](https://deprogrammaticaipsum.com/twenty-years-is-nothing/)

在[本杂志的以前版本](/the-winner-takes-it-all/)中，我们曾经提出英语在我们行业中如此普遍，以至于没有人再质疑它的使用。同样的情况也适用于[Git](https://git-scm.com/)。很难想象仅仅二十年前，源代码控制工具的格局更加多样化，选择这样的工具比如今也更加复杂。实际上，Git 甚至还没有出现在地图上。在讨论 Git 的霸权是好是坏之前，让我们先回到过去一小会儿。

在[最著名的探戈之一](https://en.wikipedia.org/wiki/Volver_(song))中，[Carlos Gardel](https://en.wikipedia.org/wiki/Carlos_Gardel) 唱道

> 感到...生活就像一口新鲜的空气，二十年不算什么，那种狂热的眼神在阴影中游荡，寻找你的名字。

## 二十年前

2004年出版的 [Steve McConnell](/steve-mcconnell/)的《代码大全》第二版。在这本厚达900页的巨著的668页，我们发现了书中唯一有关源代码控制的参考：大约不到一页的篇幅。除此之外没有其他内容。 ChatGPT 可以轻松地用一句话总结："版本控制软件很好，带来了一些巨大的好处。” 至此离GitOps 还有很远的距离。

同年，距今本文发表时间几乎完全20年以前，[Subversion 1.0](https://en.wikipedia.org/wiki/Apache_Subversion) 出现了。Subversion 是什么？可能是计算机历史上寿命最短的有良好意图的想法。Subversion（或`svn`）当时应该比CVS更好（不，不是这个[药店](https://www.cvs.com/)，而是这[个东西](https://en.wikipedia.org/wiki/Concurrent_Versions_System)）。所谓“比CVS更好”意味着当时，是事务性的，有点像[数据库](/issue-61-databases/)，并且对支持分支有略微更好的支持。那时候我们的抱负并没有那么高， 孩子们。

然而，Linus Torvalds 追求更高目标。2004年，Linux 内核开发者在如何使用[BitKeeper](https://en.wikipedia.org/wiki/BitKeeper)（专有的分布式版本控制系统，用于管理内核源代码）问题上出现了越来越激烈的分歧。那么，开发者该怎么做呢？好吧，Linus 有一个通过写每个人都需要但却没有人想开始的软件的传统。他还有一个[以自己名字命名事物](https://www.wordnik.com/words/git)的传统。传说中 Git 的第一个版本是[在几周内编写完成的](https://www.linuxjournal.com/content/git-origin-story)。

## CVS

从未听说过 CVS？这是 Joel Spolsky 在2000年9月描述的一种源代码控制系统，他称之为 *不错的* （他的强调），这是他提出的更好软件的 [Joel 测试](https://www.joelonsoftware.com/2000/08/09/the-joel-test-12-steps-to-better-code/) 的第一条：

> 我曾使用商业源代码控制包，也使用过免费的 CVS，让我告诉你，CVS 是 *不错的*。

是的，改进软件的第一步（震惊！）是使用源代码控制软件。（顺便说一下，我于1997年开始我的职业生涯时是一名软件开发人员，不，我们当时没有使用源代码控制，甚至没有使用 CVS。是的，你猜对了：我们只是将 VBScript 文件保存在本地，然后通过 FTP 上传。如果我们互相覆盖了对方的更改，那就只能认了。直到2002年我才第一次使用源代码控制系统，好奇的人可以了解一下，那是 [Rational ClearCase](https://en.wikipedia.org/wiki/Rational_ClearCase)。）

从未听说过 Joel Spolsky？那么，他是 Stack Overflow 的联合创始人，我猜您在职业生涯中某个时候一定使用过。24年前，Joel 是软件工程新兴领域中的首批影响者之一。可以把他想象成 Kelsey Hightower，但他的观点更具争议性。或者是 Steve Yegge，但他的观点不那么具有争议性。

谈到 Stack Overflow，这是2008年源代码控制的最新进展的一个例子。该站点上最早的问题之一，日期为2008年9月8日，准确地询问了 [应该使用哪个版本控制系统](https://stackoverflow.com/questions/49601/is-there-a-barebones-windows-version-control-system-thats-suitable-for-only-one) 适合单人开发流程。（有趣的是，正是在那时候，现实世界中 [2008年金融危机](https://en.wikipedia.org/wiki/2007%E2%80%932008_financial_crisis#2008_(September)) 正在肆虐。毫无疑问，我们的行业确实生活在一个泡沫中。但我又岔开话题了。）

> 我正在尝试找一个我个人使用的尽可能简单的源代码控制工具。我最需要的主要功能是能够阅读/拉取我代码的早期版本。我是唯一的开发者。

对问题的回复包括一个长长的目录，几乎包含当时人类所知的所有版本控制系统。

## Windows 的版本控制奥德赛

但是让我们回到 2000 年：在 Joel Spolsky 发布他的 Joel 测试的几天前，[Mark Lucovsky](https://www.linkedin.com/in/mark-lucovsky-5280034/) 在 [第四届 USENIX Windows 系统研讨会](https://www.usenix.org/legacy/events/usenix-win2000/) 上发表了题为 [“Windows: A Software-Engineering Odyssey”](https://www.usenix.org/legacy/events/usenix-win2000/invitedtalks.html) 的演讲。Lucovsky 先生从 1988 年到 2000 年代中期都是最初的 Windows NT 团队成员。这次演讲的 PowerPoint 幻灯片 [目前还在网上](https://www.usenix.org/legacy/events/usenix-win2000/invitedtalks/lucovsky_html/Lucovsky.ppt)，我强烈推荐你看一下。

因为“Odyssey”的一部分是，你猜对了，源代码控制。在 Lucovsky 先生的 PowerPoint 幻灯片第 14 页，你可以了解到 Windows NT 3.51 使用了一个“内部开发”的系统……到了 Windows 2000 时期已经“奄奄一息”：

> 维护一台机器同步是一项巨大的工作（设置需要 1 周，每天同步需要 2 小时）

糟糕。这不是一个很好的新团队成员入职方式。现在你知道为什么 [敏捷宣言](https://agilemanifesto.org/)，发布于 2001 年，如此革命性了。多亏了 [Raymond Chen](https://devblogs.microsoft.com/oldnewthing/)，可能是 Windows 历史上最重要的讲师之一，我们得知了这个内部开发的系统的名字：[在此](https://devblogs.microsoft.com/oldnewthing/20180122-00/?p=97855)。

> 在早期，微软使用了一个自制的源代码控制系统，正式称为 Source Library Manager，但通常简称为 SLM，并发音为 *slime*。这是一个简单的系统，不支持分支。

在 Mark Lucovsky 先生的 PowerPoint 幻灯片第 24 页，我们了解到微软决定将 Windows 2000 的源代码迁移到一个名为 “Source Depot” 的东西。Raymond Chen [同意](https://devblogs.microsoft.com/oldnewthing/20180122-00/?p=97855)：

> Windows 2000 发布不久后，Windows 源代码过渡到了一个名为 Source Depot 的源代码控制系统，这是 Perforce 的授权分支。

为什么选择 Perforce？这个选择与微软 Windows 源代码库的巨大规模有关：

> 理由或许不如以前那么重要了，但是对于大型仓库，Perforce 比 Subversion 的性能更好。这也是为什么微软获取了 Perforce 的源代码许可来构建 Source Depot 的原因之一；NT 的代码库非常庞大，不多的产品，无论是商业产品还是其他产品，都能处理它。

Mark Lucovsky 在他的演讲第 24 页中概述了 Source Depot 的两个优点：

> +   新机器设置 3 小时 vs. 1 周
> +   
> +   正常同步 5 分钟 vs. 2 小时

微软 Windows 团队今天是否仍在使用 Source Depot？显然不是。在 2017 年，我们了解到微软将所有 300GB 的 Windows 源代码迁移到 Git，[详见 Ars Technica 文章](https://arstechnica.com/information-technology/2017/02/microsoft-hosts-the-windows-source-in-a-monstrous-300gb-git-repository/)，其中还描述了在源代码控制系统中的“微软奥德赛”。

> 很久以前，公司有一个叫做 SourceSafe 的东西，它在名誉上相当于把所有珍贵的源代码扔进垃圾桶，然后点火，这要归因于系统倾向于损坏其数据库。

（我可以确认。遗憾的是，我得这么说。）

然而，微软采用 Git 并不是一帆风顺，并导致了 Git 虚拟文件系统（GVFS）项目的创建：[来源](https://arstechnica.com/information-technology/2017/05/90-of-windows-devs-now-using-git-creating-1760-windows-builds-per-day/)

> 但 Git 并不是设计用来处理由 3.5 百万文件组成的 300GB 仓库的。微软必须启动一个项目来定制 Git，以使其能够处理公司的规模。

## Git 时代

微软对 Git 的热情在 2018 年达到了顶峰，当时[它吞并了 GitHub](https://news.microsoft.com/2018/06/04/microsoft-to-acquire-github-for-7-5-billion/)，这个平台可以说是让 Git 成为主流。三年前，感受到了*时代的气息*，他们发布了[Visual Studio Code](https://en.wikipedia.org/wiki/Visual_Studio_Code)，内置了 Git 支持。

GitHub 早在 2008 年 2 月就向世界介绍了 Pull Requests 的概念，后来被[GitLab](https://docs.gitlab.com/ee/user/project/merge_requests/)、[Gitea](https://docs.gitea.com/next/usage/pull-request)（及其最新的分支[Forgejo](https://forgejo.org/docs/latest/user/pull-requests-and-git-flow/)）以及[BitBucket](https://support.atlassian.com/bitbucket-cloud/docs/tutorial-learn-about-bitbucket-pull-requests/)采纳和调整，这在过去的 15 年里成为了代码审查的重要工具。但事实是，GitHub 在 Git 的世界中也造成了一个悖论：一个分布式源代码控制系统…… 突然变成了集中式的。一些人对此感到[惊讶](https://blog.edwardloveall.com/lets-make-sure-github-doesnt-become-the-only-option)。

我们现在是2024年，Git 无处不在。导致Git在2010年代及显然在2020年代也称霸的漫长演进可以总结为一系列开源程序的演变：70年代的[SCCS](https://en.wikipedia.org/wiki/Source_Code_Control_System)，80年代的[RCS](https://en.wikipedia.org/wiki/Revision_Control_System)，90年代的CVS，和2000年代的Subversion。为了确保平稳的迁移路径，Subversion 可以导入CVS仓库，而Git可以导入Subversion仓库。但更重要的是，版本控制系统从像SCCS和RCS这样的仅本地系统，向客户端-服务器架构（如CVS和Subversion），再到分布式系统（如Git和其他系统，尤其是[Mercurial](https://www.mercurial-scm.org/)）迁移。

（说到Mercurial，你知道Firefox开发者最近[决定放弃它](https://glandium.org/blog/?p=4346)并改用Git了吗？）

如今，我们习惯于*克隆*整个项目到我们的计算机上，之后我们可以安全地断开网络，继续以完全脱机的方式编写软件。这种简单的范式在20年前是完全不可想象的。而且你的本地仓库还包含了项目的每一个修改的完整历史记录。这是一个特性，客户端-服务器系统自然无法提供（剧透：服务器有完整的历史记录，而客户端只有`HEAD`，可以这么说）。

Git（及其便宜的分支设施）对开发者工作流产生了持久的影响。Vincent Driessen 在 2010 年 1 月发布了一篇开创性的文章，标题为[“一个成功的Git分支模型”](https://nvie.com/posts/a-successful-git-branching-model/)，向世界介绍了[有争议的](https://web.archive.org/web/20111007075151/http://scottchacon.com/2011/08/31/github-flow.html)git-flow的概念。为什么有争议？嗯，因为软件行业的大多数意见都是这样的。现在有一个[GitHub flow](https://docs.github.com/en/get-started/using-github/github-flow)，还有一个[Atlassian Gitflow workflow](https://www.atlassian.com/git/tutorials/comparing-workflows/gitflow-workflow)，以及许多其他的分支工作流可供选择。

Git 仓库变得丰富多彩，每次推送、合并或标签操作都会触发某个工作流。整个行业已经涌现出来，包括诸如 [Argo CD](https://argoproj.github.io/cd/), [GitHub Actions](https://docs.github.com/en/actions), [GitLab CI/CD pipelines](https://docs.gitlab.com/ee/ci/pipelines/), 和 [Gitea Runner](https://about.gitea.com/products/runner/) 等名称，提供了新的自动化和便利水平。Git 在这个领域的影响力非常强大，以至于术语 [GitOps](https://www.gitops.tech/) 现在指的是我们行业的一个子集。但是，你不应该将分支用于部署，[已经警告过你了](https://codefresh.io/blog/stop-using-branches-deploying-different-gitops-environments/)。

现在问题很简单：Git 之后又会出现什么？在这一点上，挑战 Git 的巨大流行可能是不可能的。我说“*可能*”是因为在我们的行业中，预测未来是不可能的。有两个有趣的竞争者值得一提：[Pijul](https://pijul.org/ "https://pijul.org/")，用 Rust 编写（尽管该项目 [十年前用 OCaml 开始](https://github.com/8l/pijul "https://github.com/8l/pijul/")）或者 [Fossil](https://fossil-scm.org/ "https://fossil-scm.org/")，由 SQLite 创建者编写。在后一种情况下，SQLite 团队提供了 [不使用 Git 的理由列表](https://sqlite.org/whynotgit.html)：

> 老实说，很少有人会争论 Git 提供了一种次优的用户体验。许多底层实现都反映在用户界面中。界面糟糕到了连一个恶搞网站都出现了，用来生成 [fake git man pages](https://git-man-page-generator.lokaltog.net/)。

等待更好的替代方案时，我们可以阅读 `man 7 giteveryday` 页面，并依赖于 [git-extras](https://github.com/tj/git-extras), [SourceTree](https://www.sourcetreeapp.com/), [TortoiseGit](https://tortoisegit.org/), [Fugitive](https://github.com/tpope/vim-fugitive), [Codeberg](https://codeberg.org/), 和 [Magit](https://magit.vc/)。无论我们喜欢与否，看起来在接下来的二十年里我们都可能将源代码存储在 Git 仓库中。

封面照片由 [Praveen Thirumurugan](https://unsplash.com/@praveentcom?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash) 拍摄，来源于 [Unsplash](https://unsplash.com/photos/a-book-and-a-small-figurine-on-a-desk-KPAQpJYzH0Y?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash)。

继续阅读 "

[Linus Torvalds](https://deprogrammaticaipsum.com/linus-torvalds/)

"或者返回

[Issue 066: 版本控制](/issue/issue-066-version-control)

。你喜欢这篇文章吗？考虑

[订阅](/newsletter/)

到我们的通讯或者

[贡献](/contribute/)

对这本杂志的可持续性做出了贡献。谢谢！
