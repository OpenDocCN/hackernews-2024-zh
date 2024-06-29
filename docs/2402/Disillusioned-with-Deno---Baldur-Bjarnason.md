<!--yml

类别：未分类

日期：2024年5月29日13:24:13

-->

# [Denno的幻想破灭](https://www.baldurbjarnason.com/2024/disillusioned-with-deno/) – Baldur Bjarnason

> 来源：[https://www.baldurbjarnason.com/2024/disillusioned-with-deno/](https://www.baldurbjarnason.com/2024/disillusioned-with-deno/)

这是我回顾我在过去几年做的工作系列的一部分。

1.  [*两年回顾：要制定战略首先必须有一个关于事情原委的理论*](/2024/2022-23-strategy-review-part-one/)

1.  [*摆脱软件危机：两年项目回顾*](/2024/out-of-the-software-crisis-two-year-review/)

1.  [*沉没成本谬误：为了一个半成品想法而纠缠太久*](/2024/sunk-cost-fallacy-research-project/)

1.  [*智能幻觉：踏入“人工智能”的一堆*](/2024/the-intelligence-illusion-stepping-into-a-pile-of-ai/)

1.  [*印刷项目回顾：卖印刷书籍的最大问题是软件*](/2024/the-problem-with-print-is-software/)

1.  [*思考印刷品*](/2024/thinking-about-print/)

1.  [*对Deno的幻想破灭*](/2024/disillusioned-with-deno/) (本页)

1.  [*一个不那么混乱的回顾：可教的是一团糟，我需要选一条道*](/2024/uncluttered-retrospective/)

* * *

一旦你决定审查自己的业务和媒体策略，你开始重新思考和审查过去做过的其他决定是不可避免的。

其中一个决定是[Deno](https://deno.com/)，我在过去几年的大部分个人项目和实验中都在使用它。其中大部分项目都比较简单 - 命令行脚本或小型网站。例如，我使用[Lume和Deno](https://lume.land/)制作了[softwarecrisis.dev](https://softwarecrisis.dev/)的通讯简讯页面。我也用Deno制作了[Colophon Cards](https://www.colophon.cards/)之后研究项目的大部分原型。 它非常有用。

有很多赞赏的地方。

Deno具有：

+   整体上对标准浏览器API的支持要比node好得多。

+   不基于标准的API更靠近浏览器的约定，使得前端和后端之间的切换不那么突兀。

+   一个优秀的标准库。

+   安装简单得多。它只有一个二进制文件，使得安装和部署简单得多。

+   大部分你需要的工具：语法检查、测试运行器、基准测试、代码格式化、类型检查器和文档生成。

+   一个明智的运行代码安全模式。

如果您有一个只需要标准库或标准工具的任务，使用Deno就像做梦一样简单。

如果那个任务需要比这更多的东西，你很快就会陷入麻烦中。

Deno背后的公司试图通过两管攻势征服这个问题：

1.  构建大部分网络项目可能需要的常见工具和项目的Deno特定版本。

1.  匆忙地为他们的运行时推出了node和npm的兼容性。

这意味着一个创业公司承担了建立他们自己的[可扩展托管](https://deno.com/deploy)，[持久存储产品](https://deno.com/kv)，[持久执行队列](https://docs.deno.com/kv/manual/queue_overview)，[实时通知](https://docs.deno.com/kv/manual/#watching-for-updates-in-deno-kv)，[定时执行](https://docs.deno.com/kv/manual/cron)，[前端React框架](https://fresh.deno.dev/)，当你阅读本文时，无疑还有更多项目的责任。

他们甚至计划为[Deno 2.0](https://www.reddit.com/r/Deno/comments/15nv8yv/deno_20_previewed_at_seattlejs_conf/)建立一个自己的功能齐全的包注册表。（另外，很让人恼火的是关于Deno未来的大部分信息只能通过视频获取。）

再加上尝试为他们的运行时实现完整的节点和npm兼容性。

基本上：

1.  你的商业模型是托管。

1.  但是，人们通常使用的工具和项目在你的新运行时和托管环境中都不可用，因此你需要自己实现来填补这些空白。

1.  然后你发现，托管的最大需求来自具有传统代码和传统依赖的客户，所以你必须给他们提供一种方法将它们迁移过来。兼容性问题就此产生。

1.  结果发现，“传统”系统中许多令人讨厌的特性存在是有原因的，所以你必须自己实现它们，但显然以更新、现代、更明智（而不那么向后兼容）的方式。

1.  如果不是因为现在你必须维护**两个**大部分运行时的版本，要么作为从“传统”系统到新系统的翻译层，要么作为需要维护的两个完全独立的功能，那这将是不错的，甚至是很好的。

我们之前已经见过这种策略。这基本上是[Cloudflare的](https://www.cloudflare.com/)实现，可能有些细节不同。执行中的大部分差异是因为Cloudflare从特定的一条香肠链的相反端开始：首先是云托管，然后是[开源运行时](https://blog.cloudflare.com/workers-open-source-announcement/)，而不是像Deno那样反过来。

我发现Deno的工具更好，他们的运行时也更易于使用，但两家公司的总体策略是相同的。

[Bun](https://bun.sh/)采用了同样的策略，他们唯一的创新是，由于他们开始较晚，他们从一开始就理解了节点和npm的兼容性需求。

我不认为这种方法对他们任何人都会特别有效。

“传统”兼容性实际上消除了为Deno（或Cloudflare或Bun）创建软件包的任何动力。为什么要使用[`dnt`](https://github.com/denoland/dnt)创建同时兼容Deno和Node的软件包，当你可以只创建一个Node软件包呢？***它仍然可以同时与Deno和Node兼容。***这一点Deno已经确保了。你无论如何都会获得跨运行时的兼容性。

风险在于Deno将有效地成为运行npm生态系统中代码和项目的平台。但它永远不会像Node本身那样“node”。

Node和npm兼容性层始终会存在差距，因为*Node是一个不断变化的移动目标*。它是一个仍在变化的活跃项目。

* * *

如果Deno由一个基金会或开源社区维护，我会少些担忧。它的范围不可避免地会更小，但那也没关系。你不需要世界统治来成为一个成功的社区驱动软件项目。它可以找到自己的利基，随着时间的推移。

但是要成为一家成功的VC资助的初创公司，你确实需要征服世界的一大块。

Deno生态系统中已经存在断裂。许多第三方模块似乎停滞不前，有一段时间没有更新。看起来充满活力的项目往往是那些一般面向浏览器兼容平台的项目，因此它们通过自身的努力几乎不费吹灰之力地获得了Deno、Cloudflare和Bun的支持。

Node也没有停滞不前。它们正在添加支持许多使Deno特殊的功能，例如内置工具、具有覆盖率的[测试运行器](https://nodejs.org/docs/latest/api/test.html)和[HTTP导入](https://nodejs.org/docs/latest/api/esm.html#https-and-http-imports)。甚至[导入映射](https://github.com/nodejs/loaders#milestone-3-usability-improvements)也在他们的路线图上。

Node的命运也不依赖于一个单一初创公司或技术公司的命运。

npm生态系统仍然是一个战略性的负担，但支持导入映射和HTTP导入将在某种程度上缓解这一问题，同时你可以直接从git安装Node软件包，或使用兼容npm的第三方存储库。[任何git远程URL都可以与`npm install`一起使用](https://docs.npmjs.com/cli/v10/commands/npm-install#:~:text=npm%20install%20%3Cgit%20remote%20url%3E)%20%E2%80%93%20无需GitHub。

观察到资金和人才从软件生态系统中流失，公司们集体转向固有保守的LLMs进行开发，以及一般软件开发人员日益增长的挫败感，很难想象在直接与Node竞争中，Deno或Bun能取得胜利的未来。

我并不确定在它们的VC支持的初创公司变得绝望后，我是否还想使用它们中的任何一个。

对于我们这些喜欢使用 JavaScript 工作的人来说，这意味着 Node 在长期内是我们在后端的最佳选择。

现在或许是时候尝试在 Node 中重新创建使 Deno 如此愉快的工作方式了。

但是 Node 的改进并非 Deno 面临的唯一问题。

最重要的问题是，与 Node 的逻辑替代方案——开发者可能会寻求的“无 Node”工作环境——*不会基于 JavaScript*。导入映射意味着浏览器实际上拥有非 JS 项目可以使用的 API 表面，用于构建依赖管理系统。现在，围绕 JavaScript 的许多工具是用 *Rust* 实现的，而不是 JS —— 其中许多工具是由 Deno 本身驱动的 —— 这使得它更容易在 Node 和 Deno 生态系统之外使用。此外，[WASM 组件模型](https://component-model.bytecodealliance.org/) 还承诺使许多依赖项与运行时无关，Rust-based 工具围绕 JS 是逻辑的初始目标。

对于许多人来说，“无 Node”的替代方案可能不是另一个 JavaScript 运行时，而是 *完全不同的东西*。
