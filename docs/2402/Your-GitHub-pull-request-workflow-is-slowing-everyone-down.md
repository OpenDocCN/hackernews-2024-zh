<!--yml

category: 未分类

date: 2024-05-29 13:19:36

-->

# 你的GitHub拉取请求工作流正在减慢所有人的速度

> 来源：[https://graphite.dev/blog/your-github-pr-workflow-is-slow](https://graphite.dev/blog/your-github-pr-workflow-is-slow)

作为开发者，我们力求编写干净、模块化的代码，易于维护。

然而，在迫于快速交付功能的压力和现代应用程序复杂性之间，实践起来需要大量工作。任何有意义的工程任务都很容易导致大型、纠缠不清的拉取请求，这成为开发过程中的瓶颈。

在本指南中，我们将探讨堆叠工作流如何帮助您克服这些挑战，并更快地交付更好的代码。无论您是初次尝试堆叠还是希望优化现有流程，此工作流都可以简化协作，加速审查周期，并促进团队的编码最佳实践。

## **为什么你的GitHub拉取请求工作流很慢**[](/blog/your-github-pr-workflow-is-slow#why-your-github-pull-request-workflow-is-slow)

在Graphite，我们分析了来自成千上万个存储库的数据，以了解GitHub上常见的拉取请求（PR）工作流如何导致开发者受阻、审查缓慢和代码质量降低。

### **PRs are too big**[](/blog/your-github-pr-workflow-is-slow#prs-are-too-big)

单个最重要的瓶颈是PR的大小 - 大型PR会使代码审查变得令人沮丧和无效。GitHub上的平均PR包含[900+行](https://www.keypup.io/product/average-pull-request-size-metric)代码变更。

为了速度和质量，PR应保持在200行以下，最好是50行。

以这个角度来看，一个巨大的500+行PR通常需要9天左右才能合并，而小于100行的PR则可以在数小时内从创建到落地。

### **PRs take too long to review and progress**[](/blog/your-github-pr-workflow-is-slow#prs-take-too-long-to-review-and-progress)

当PR包含数千行变更时，进行适当的审查变得异常困难。开发者往往需要花费数日来修订初始反馈后的代码，而审阅者则需重新扫描所有代码，找出变更，并对每个修订版进行批准或拒绝。

在大型PR上的来回审查周期减缓了团队的开发速度，并不一定彻底。根据Graphite的数据，只有24%的超过1000行的大型PR收到**任何**审查评论。

### **Large PRs introduce more bugs**[](/blog/your-github-pr-workflow-is-slow#large-prs-introduce-more-bugs)

当代码未经彻底审查时，由于审阅者通常无法给予每个PR应有的关注，易于有漏洞被忽略。

问题很容易在变更之间被掩盖，评审者通常会做出假设，而不是测试，以避免感到不知所措。特别有趣的发现之一是，随着PR的大小增加（文件更改的数量），评审者在每个文件上花费的时间显著减少（对于更改了8个或更多文件的PR）。

这些快速的评审很容易导致质量问题，这些问题会产生更多的bug修复PR、技术债务和团队的挫败感。工程师们感觉他们的进展逆转，因为他们花费了越来越多的时间来修补bug，而不是开发功能。

## **标准的GitHub拉取请求工作流程**[](/blog/your-github-pr-workflow-is-slow#standard-github-pull-request-workflow)

数百万开发者每天使用GitHub的拉取请求模型，但实际上团队经常遇到的痛点减缓了进展速度。

让我们来了解标准的GitHub PR工作流程。

### **1\. 创建一个特性分支**[](/blog/your-github-pr-workflow-is-slow#1-create-a-feature-branch)

通常，GitHub项目遵循一种分支模型，开发人员为每个用户故事或功能创建一个新分支。

首先，**检出**主分支（*这是生产就绪代码的存放地*），然后使用以下方式创建特性分支：

```
git checkout -b new-feature
```

**这为您的更改生成了一个独立的流。**

现在，这看起来可能很好 —— 您有一个安全的空间来构建新功能，而不会影响**主分支**。这让**主分支**保持稳定，因此您的CI/CD流水线畅通无阻。

**但是随着您的分支历史与主分支的进一步分离，您会注意到几个重要的问题：**

+   合并[上游更改](https://stackoverflow.com/questions/2739376/definition-of-downstream-and-upstream)由于重新基础冲突而变得乏味。

+   在推送[热修复](https://en.wikipedia.org/wiki/Hotfix)时，可能会打断其他依赖分支，失去您的位置。

+   几周后，多次新的主要版本发布，您的代码现在依赖于过时的分支，可能需要许多更改才能与最新的分支配合使用。

曾经是一个高效的分支，随着您在越来越多的技术债务中挣扎，现在成为一个停滞的孤立。

### **2\. 编写代码并提交更改**[](/blog/your-github-pr-workflow-is-slow#2-write-code-and-commit-changes)

由于工作流程不会一夜之间改变，您继续编写您的大型新功能。您优化于构建原型，而不是维护严格的隔离。

在您专用的分支内迭代相关组件似乎更快。

**不知不觉间，您已经：**

+   修改关键服务的1,500行代码。

+   涵盖多个微服务的数据模型更改。

+   多个新的前端路由和UI流程。

所有这些都被堆砌在一个难以管理的提交历史中。

但至少它仍然安全地隔离在您的特性分支之后，对吗？实际上并非如此 —— 您的分支在所有这些前端之间高度耦合，稍后要进行解耦可能会变得非常具有挑战性，需要大量的重新基础。

准备就绪后，将本地代码更改提交并推送到远程功能分支。现在你的更改已保存在远程存储库中。

### **3\. 打开拉取请求**[](/blog/your-github-pr-workflow-is-slow#3-open-a-pull-request)

一旦你的功能分支包含要合并到主分支的更改，下一步就是打开一个拉取请求（PR）。

**拉取请求让你通知队友们你的代码已经准备好审查和反馈。**

要创建一个 PR，你需要去你的代码管理工具，并为你的分支打开一个拉取请求。

你可以在你的 PR 中写下简短的描述性变更摘要，例如，“添加用户个人资料功能。”

对于 PR 正文，你会写一篇更长的描述，详细解释：

+   你做了哪些具体的变更以及添加了哪些新功能。

+   你是如何实现这些技术细节的。

+   你为什么认为这些变更是必要的。

+   队友们测试你构建的新功能的步骤。

现在，这个 PR 成为了与分支关联的集中讨论场所。你会@团队成员和相关方请求他们的代码审查。

### **4\. 处理审查意见**[](/blog/your-github-pr-workflow-is-slow#4-address-review-comments)

在你的代码经过审查后，队友们会彻底检查其中的错误、可能潜入的不良编码实践、你可能未考虑的边缘情况以及简化的机会。

**无论大小，问题都会不可避免地出现。**

在 PR 页面上，你会在代码级别进行反复的交流。队友们可以评论特定的代码行以指出问题或提出澄清问题。

对于每一轮反馈，你都会深入到你的分支中来处理评论。这导致了大量的重做——修复缺陷、重构架构、添加测试用例等，这些都基于审阅者的见解。

一旦你将新的提交推送到你的分支，它们将自动出现在拉取请求中以获取进一步的反馈。这个审阅周期会继续，直到所有问题都得到解决。

### **5\. 合并你的拉取请求**[](/blog/your-github-pr-workflow-is-slow#5-merge-your-pull-request)

最后，在所有反馈都得到充分解决并且你的代码获得了来自队友的 LGTM（"looks good to me"）后，就是时候将你批准的拉取请求合并到**主分支**中了。

这个合并将你的代码与**主分支**整合在一起，这样可以：

+   将你的分支提交合并到主分支。

+   自动关闭拉取请求。

+   删除你的功能分支（*可选，取决于你的内部工作流程*）。

你的更改现在完全融入到生产代码中。但是对于每个新的（甚至是紧密相关的）功能，你都需要从头开始，重复整个工作流程。

**如果有更好的方法会怎样？** 输入[堆叠](https://stacking.dev)。

## **通过堆叠来改进 PR 工作流**[](/blog/your-github-pr-workflow-is-slow#improving-pr-workflows-with-stacking)

*堆栈是通过将大型功能拆分为小的、依赖的拉取请求，逐步集成到 GitHub 拉取请求工作流程中的开发方法。一旦所有递增的拉取请求都得到批准，它们就会与主分支合并。*

探索一下如何解决传统 PR 工作流中的痛点，并且有意义地提高您的开发速度。

### **什么是堆栈？**[](/blog/your-github-pr-workflow-is-slow#what-is-stacking)

堆栈使用“堆栈”结构进行特性开发，每个小的拉取请求都建立在彼此之上：

通过堆栈 PR，每个 PR 变得更加专注，易于审查和修复。新工作可以继续在获得批准之前建立在现有的 PR 上。

这样工程师可以快速打开多个小的 PR 并并行进行，而不会被阻塞。更改持续地发布，而不是落入大量的单体代码堆积中。

### **堆栈是如何工作的？**[](/blog/your-github-pr-workflow-is-slow#how-does-stacking-work)

堆栈通过让开发人员将他们的工作拆分为多个小而互相关联的拉取请求，来改变构建大型功能的方式。而不是一个巨大的 PR，他们创建了一堆相互依赖的专注变更。

*例如，开发人员可能正在编写一个新的注册系统。与其在一个巨大的 1000 行 PR 中完成所有前端、后端和数据库工作，不如将其分解。一个 PR 实现注册 API 端点。另一个将其连接到数据库。第三个则构建注册页面并利用新的 API。*

每个 PR 默认仅限于 1 个 <200 行的提交，范围严格控制。这迫使开发人员有意识地限制工作于相关的更改 —— 注册端点 PR 不能悄悄加入无关的样式调整。

这些专注的 PR 互相链接，形成一个连贯的堆栈 —— 注册页面 PR 从 API PR 分支出来，后者又依赖于数据库 PR。随着逐步审查，更改逐步集成到上游。

类似 Graphite 这样的开发工具，自动化处理堆栈中所有的依赖关系，更新分支并持续进行变基，以保持分支同步。因此，通过更小的更改，堆栈显著减少了审查和修订所需的时间，帮助您实施新功能和更快地修复错误，胜过传统的工作流程。

### **为什么堆栈可以改进拉取请求工作流程**[](/blog/your-github-pr-workflow-is-slow#why-stacking-improves-the-pull-request-workflow)

将团队从传统工作流转向堆栈工作流，从根本上提升了变更的结构化、审查和集成方式。它优化了生产力、质量和理解度。以下是它帮助您的开发生产力的几种方式。

#### **分解大型更改**[](/blog/your-github-pr-workflow-is-slow#breaks-down-large-changes)

堆叠的核心是将大型功能工作分解为一系列较小的拉取请求。每个PR通常限制为一个专注于独立更改的提交。这种限制引导开发人员有意识地只进行单一更改，沿途进行压缩合并和变基，而不是在PR中混杂“拼写错误修复”等随机不必要的提交。

#### **加快审核周期**[](/blog/your-github-pr-workflow-is-slow#accelerates-review-cycles)

在传统实践中，开发人员将更改批量打包到巨型PR中，这使得审阅者不堪重负。由于审核大块代码需要花费很长时间，反馈也会越推迟。

相比之下，堆叠小型PR加速了反馈循环，因为审阅者需要解析的复杂性大大降低，能够及时和彻底地提供反馈。

#### **简化错误修复**[](/blog/your-github-pr-workflow-is-slow#simplifies-bug-fixes)

原子化确保修复保持干净——如果任何PR引入问题，简单地回滚单独的变更就很容易。PR越小，修复代码的速度就越快。例如，一个50行PR中的bug修复可能只需几分钟，而500行PR中的bug修复则要花费更多时间。

#### **实现稳定集成**[](/blog/your-github-pr-workflow-is-slow#enables-steady-integration)

在典型的巨型PR工作流程中，最终获得巨大更改的批准和合并往往带来一种不舒服的压力。特别是在审查过程漫长的情况下，管理人员开始强烈推动加快合并步伐，解锁其他工作。

相反，堆叠的PR可以通过持续交付微小增量变更快速流入主分支。每个变更都是隔离的，风险较低，如果合并导致bug，回滚也是无痛的。通过在发布数百或数千行之前拆分审查，堆叠消除了围绕巨型PR累积的瓶颈和紧张情绪。

现在你知道堆叠如何极大地利于内部开发人员和运维工作流程，那么如何为你的团队实现这种流程呢？

## **GitHub的拉取请求工作流程与堆叠**[](/blog/your-github-pr-workflow-is-slow#github-pull-request-workflow-with-stacking)

让我们看看堆叠工作流程是如何运作的。目标是通过稳定的集成优化GitHub工作流，而不是进行大而笨重的代码更改。我们希望我们的变更能够顺畅地流过，而不是将它们变成阻碍其他开发人员的大型PR。

为了简化操作，我们将使用[**Graphite**](https://graphite.dev/)——一个强大的工具，自动化许多堆叠工作流程步骤。

### **1\. 检出主分支以开始新的工作**[](/blog/your-github-pr-workflow-is-slow#1-checkout-main-branch-to-start-fresh)

假设你正在开发一个新功能——**应用通知**。

一旦你安装并集成了Graphite到你的Git仓库，你可以检查最新的主分支代码，以确保你从最新的上下文开始：

与 Git 工作流不同，容易忽视保持更新的情况，Graphite 将您的工作流程集中在持续与当前主线状态集成上。

### **2\. 通过堆叠分支逐步构建功能**[](/blog/your-github-pr-workflow-is-slow#2-build-feature-incrementally-through-stacked-branches)

在检出主分支后，通过迭代和专注的工作开始构建您的功能。

作为我们**应用通知**功能的第一步，我们将通过将更改分阶段添加到新分支并提交它们，一气呵成使用 `gt create`：

```
gt create -am "feat: add notification scaffolding”
```

此命令将添加您的更改并一次性创建一个新分支。然后，您可以通过创建和堆叠额外的分支来继续迭代：

```
gt create -am "feat: implement email templates""
```

在进行异步审查时，电子邮件模板基于脚手架基础构建。工程师们避免了依赖工作进展的阻碍。通过稳定的流而不是大型拉取请求，变更能够平稳集成。

### **3\. 将多次提交的分支合并为一个**[](/blog/your-github-pr-workflow-is-slow#3-squash-multi-commit-branches)

如果您的一个分支有多次提交，Graphite 使压缩并合并它们变得简单，确保一次 PR 只有一个提交，保持一致性。

```
gt log◉ 06-28-second_branch (current)│ 10 minutes ago│| 6s7a8d7 - last committed change│ d7d41b6 - committing another change│ 8c6d8de - committing some changes│◯ 06-28-first_branch│ 5 minutes ago││ 232e8cf - initial commit│◯ main│ 10 minutes ago││ 1e0b290 - Merging a pull requestgt squashgt log◉ 06-28-second_branch (current)│ just now|│ 9e13a52 - a single commit│◯ 06-28-first_branch│ 5 minutes ago││ 232e8cf - initial commit│◯ main│ 10 minutes ago││ 1e0b290 - Merging a pull request
```

通过清理您的 PR 提交历史记录，确保清晰简洁的主分支历史记录，从而轻松查看随时间变化的具体内容。

### **4\. 逐步将更改合并到主分支**[](/blog/your-github-pr-workflow-is-slow#4-merge-to-main-incrementally)

在逐步完成所有功能工作后，您的小型 PR 可轻松合并到主分支，而不会成为其他开发者的瓶颈或阻碍。

只需提交一堆分支即可生成一系列关联的拉取请求：

Graphite 自动为分支创建 PR，使变更可以向下流动。它甚至使用模板自动填充标题和描述，因此您可以准备好基本的 PR 模板——填写模板，然后开始吧。

最终，在获得批准后，您的功能堆栈顺利地合并到**主**分支。

### **5\. 大大减少错误的可能性**[](/blog/your-github-pr-workflow-is-slow#5-greatly-reduces-the-possibility-of-bugs)

在由许多小的、依赖性强的拉取请求组成的堆叠工作流中，每个变更都足够小，以确保彻底审查。由于大型功能被分解为原子变更并逐步合并，轻松回滚回归和修复错误，避免大型、单块式变更带来的不必要停机。

## **在 Graphite 中通过堆叠简化您的开发工作流程**[](/blog/your-github-pr-workflow-is-slow#simplify-your-dev-workflows-with-stacking-in-graphite)

大多数开发者仍然在应对许多不同部分的挑战——分散的工具、交织的分支，以及代码审查中的停滞不前。

[**Graphite**](https://graphite.dev/) **提供了更好的前进方式**。

Graphite 并行化审查和开发，自动化 Git 的所有复杂性，让您有更多时间构建。

+   开发人员在代码审查上保持不受阻碍，继续推动更改。

+   评审员在将特性分解为小块时提供快速而彻底的反馈。

+   管理者因项目可靠地发布而放心。

Graphite 将 GitHub 的 PR 组织成堆栈，同时简化和加速代码审查。

**立即开始使用** [**Graphite**](https://graphite.dev/) **构建。**
