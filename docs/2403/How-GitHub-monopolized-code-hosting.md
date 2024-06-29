<!--yml

类别：未分类

日期：2024-05-29 12:48:26

-->

# GitHub 如何垄断代码托管

> 来源：[https://graphite.dev/blog/github-monopoly-on-code-hosting](https://graphite.dev/blog/github-monopoly-on-code-hosting)

我从高中开始写代码。我模糊记得和朋友一起用 [Tortoise SVM](https://tortoisesvn.net/) 创建 Android 游戏分享代码。在大学期间，我学会了克隆 GitHub 仓库来获取计算机科学作业。后来，在实习期间，我加入了使用 GitHub 来审查和合并 PR 的行列。在过去十年间，大多数进入职业阶段的开发者可能和我有着类似的经历 - GitHub 与源代码和代码变更同义，无论是在开源项目还是私人公司团队中。

很容易认为 GitHub 的普及是理所当然的 - 但事情是如何演变到这一步的呢？

我问我的队友戴维是如何发现 GitHub 的。他比我编码时间长十年，经历了职业转变。在2000年代，他回忆说在编程工作中使用 SVN。他会从 SourceForge 下载软件，但认为界面很实用但“糟糕”。最终，他发现自己越来越经常地转向 GitHub 查找文档并下载开源工具，如 [Rails](https://rubyonrails.org/)。这使他开始了解 Git 并最终在工作中使用 git-to-svn 的转换器。但直到2010年，许多公司仍然在 SVN 上托管代码，直到多年后大多数私人组织才完全迁移到 Git。

戴维的轶事进一步激起了我的兴趣。GitHub 是如何进入市场的？之前存在什么，他们填补了什么空白？

**注意**

Greg 每周花费整个工作日深入探讨工程实践和开发工具。这得益于这些文章有助于宣传 Graphite。如果你喜欢这篇文章，请今天就尝试 [Graphite](https://graphite.dev?utm_source=blog-note)，并开始以30%更快的速度交付！

## GitHub 之前的世界[](/blog/github-monopoly-on-code-hosting?utm_source=urbanisierung-dev&ref=urbanisierung-dev#the-world-before-github)

在 GitHub 成立四年之前的2004年，Linus 创建了 Git。虽然许多人仍在使用 SVN，但 Git 作为分布式版本控制系统正迅速增长。它具有强大的价值。与 CVS 和 SVN 等先前工具不同，Git 用户可以在他们的计算机上托管整个源代码副本，无需与集中服务器通信。人们可以更新代码（甚至离线！）并相互分享副本，无需从中央托管的源获取批准（或托管成本）。而在 SVN 中分支代码需要复制整个仓库，而在 Git 中创建分支则既快速又便宜。

可以想象，这些改进导致了开源社区创意开发的爆炸式增长。Git专为分布式民主化开发而构建，并迅速传播开来。尽管它有诸多优势，但在企业代码库中普及程度较慢，私人集中管理的SVN服务器与传统工作流程相对来说服务良好。

跳过四年，到了2008年。开源项目如Rails开始采用Git，继Linux之后。私人组织继续使用SVN和Perforce服务器来集中管理源代码。开源软件主要分发在SourceForge，稍后是Google Code，或是很少见的个人托管服务器上。

尽管SourceForge占据主导地位，但还有很多不足之处。首先，该站点直到[2009年](https://arstechnica.com/information-technology/2009/03/sourceforge-adds-support-for-new-version-control-systems/)才开始支持Git，这是在GitHub创建并吸引了超过10万用户一年后。但差异不仅在技术上存在。今天，当我们想到在线代码托管时，我们会想到一个可以访问以浏览代码、问题和贡献者的网站。但SourceForge的体验并非如此。它及其类似的Google Code等曾侧重于*向终端用户分发软件*，而非*代码协作*。它们通过聚合开源项目的分发解决了一半的难题，但在评论、代码浏览、更改审查和其他现代基本功能方面则几乎无所作为。

情况变得更糟。在SourceForge上创建和管理仓库非常痛苦。首先，该站点要求申请和人工批准才能创建新的仓库。此外，[从未支持私有仓库](https://sourceforge.net/p/forge/site-support/7229/#8326)，这意味着该站点对于闭源托管毫无用处。在项目上留下评论和问题是不直观的，而分叉则是不常见的做法。如果你想为一个项目做贡献，最常见的方式是生成一个补丁，然后通过[项目的邮件服务器发送](https://news.ycombinator.com/item?id=14381076)，而不是分叉并向项目发起拉取请求。20年后了解SourceForge，我被它听起来与当前的Apple App Store多么相似而感到震惊。在这种情况下，GitHub可以填补的市场空白更加明显。

## [现任者：SourceForge 和 Google Code](/blog/github-monopoly-on-code-hosting?utm_source=urbanisierung-dev&ref=urbanisierung-dev#the-incumbents-sourceforge-and-google-code)

查看2008年SourceForge的[登陆页面](https://web.archive.org/web/20080103101749/https://sourceforge.net/)：你注意到了什么？对于成为网上最大的开源专注网站感到自豪的声明。下载数量，以及突出的项目。新软件版本发布，甚至右栏的广告。但没有提到Git，没有关注用户个人资料，也没有私有仓库。重点是分发，而不是开发者协作。

SourceForge上的一个仓库示例。请注意，主要的行动呼叫是一个下载按钮，而不是克隆按钮。

[code.google.com](http://code.google.com)相较于SourceForge逐步进步。该站点专注于免费托管代码和文档的简便性。它利用了谷歌出色的搜索技术来进行发现，并为希望学习的开发人员提供了周到的文档。但关键是，它缺乏后来对开源开发人员如此强大的社交和协作功能。最后，虽然它推出了SVN支持，但Google Code直到2011年才增加了对Git的支持 - 还配有一个名为[Wired的时髦文章](https://www.wired.com/2011/07/google-code-finally-adds-git-support/)。

## 最后，GitHub的发明[](/blog/github-monopoly-on-code-hosting?utm_source=urbanisierung-dev&ref=urbanisierung-dev#finally-the-invention-of-github)

2008年的一个晚上，汤姆·普雷斯顿-维纳（Tom Preston-Werner）和克里斯·万斯特拉斯（Chris Wanstrath）在旧金山一个Ruby on Rails聚会后在一家运动酒吧喝酒。Rails社区开始越来越多地使用Git，但没有像SourceForge那样的中心网站用于托管Git仓库。此外，社交网络明显蓬勃发展，但没有为像他们这样在开源项目上工作的开发人员提供社交网络站点。Git使协作软件开发变得比以往任何时候都更容易，而像SourceForge这样的网站帮助分发版本发布 - 但没有网站作为协作的家园。如果任何人都可以托管源代码，讨论问题，并请求维护者拉入分叉更改 - 一切都支持像Facebook那样的个人资料页面和评论订阅？

即将成为GitHub联合创始人的两人开始在周末项目上构建GitHub的第一个版本。他们在周末使用Rails开发了GitHub的最基本功能，直到它足够完整，可以在克里斯的白天工作中使用。通过日常使用来解决问题，并修复任何阻碍功能的差距。

> “GitHub公司的确是从这个副业中冒出来的，所以我们从来没有什么大愿景或梦想。我们只是想做一些很酷的事情。”
> 
> -[克里斯·万斯特拉斯](https://signalvnoise.com/posts/2486-bootstrapped-profitable-proud-github)

Rails的创作者也是GitHub的早期用户之一，帮助推动了该网站的早期流行。

## GitHub在2008年的外观[](/blog/github-monopoly-on-code-hosting?utm_source=urbanisierung-dev&ref=urbanisierung-dev#how-github-looked-in-2008)

GitHub 的原始标志包含“社交代码托管”，他们的 H1 品牌承诺写着：“Git 仓库托管，不再是痛苦的事情。”这不会让人混淆 - GitHub 推出了两个核心增值服务：社交网络功能和在线托管 Git 仓库的能力。没有其他网站能在这些方面与之竞争。

**项目和开发者动态：** 关注你最喜爱的项目及其工作人员。

**源代码浏览器：** 轻松查看您的代码的任何版本和任何分支或标签。

**公共开发者个人资料：** 查看其他开发者正在进行的工作及其提交次数。

> “杀手级应用程序能够决定平台的成败。对于 GitHub 来说，我认为 Git hub 刚刚获得了一个。” —David Heinemeier Hansson
> 
> “GitHub 令人惊讶的地方在于它如何真正引入了社交因素。Chris 和 Tom 正在向我们展示如何视觉化地进行 git 开发工作。我知道，一旦我开始从外部 git 仓库中拉取提交，我个人确实有一些灵光乍现的时刻。” —Rick Olson
> 
> “你可能在过去的一周听过至少十二次了，但 GitHub 真是太牛了。我以前从未有过理由将我的代码放到类似的托管服务上，但现在我有了。” —Josh Susser

## GitHub 的快速增长[](/blog/github-monopoly-on-code-hosting?utm_source=urbanisierung-dev&ref=urbanisierung-dev#githubs-rapid-growth)

在试用了 MVP 后，GitHub 的联合创始人向朋友们推出了托管公共仓库的免费测试版。

GitHub 在开源社区的增长是爆炸性的，显示了出色的产品市场契合度。它很快被 Rails 项目采纳和强制执行，这意味着任何想使用 Ruby on Rails 的人都必须与 GitHub 互动。（这就是像我的队友 David 第一次了解 GitHub 和 Git 的方式）。

+   在第一年，GitHub 迅速发展，托管了46,000个公共仓库。

+   第二年，GitHub 的仓库数量增长到了90,000个，用户数量增加到了100,000。

+   在第三年，这一数字增长到了100万个仓库，到2011年，GitHub 的规模超过了 SourceForge 和 Google Code。

尽管最初增长迅速，但三位创始人仍然节俭并自给自足。他们努力迅速产生收入。与 SourceForge 专注于广告不同，或者 Perforce 专注于企业销售不同，GitHub 的创始人迅速开始销售用于托管私有仓库的个人订阅服务。这种模式易于理解，自助服务，并且与其他托管产品和社交网络站点相比，在该领域具有一定的独特性。Google Code 和 SourceForge 不仅不托管 Git 仓库，而且也没有私有代码托管选项。除了 GitHub 外，托管私有仓库的唯一选择是运行个人管理的服务器。

除了来自订阅的快速收入，GitHub的联合创始人们还找到了赚钱和节省成本的其他方式。他们尝试了一次性广告位、商品商店、Git培训服务和职位板等替代收入流。为了尽可能降低业务成本，GitHub与Engine Yard合作，后来又与Rackspace合作，提供页脚广告以换取免费托管。

2009年，[GitHub推出了自托管版本](https://github.blog/2009-06-01-announcing-github-fi/https://github.blog/2009-06-01-announcing-github-fi/)，使大型企业能够运行GitHub。工程师不再使用SVN服务器或Perforce等产品，而是可以利用他们新发现的Git工具进行开源和闭源开发。GitHub于2010年推出了面向组织和团队的计划，进一步推动了私有市场的代码托管和变更协作，2011年，他们将Github Fi重新品牌化为正式的企业服务器产品。

> “我们的[GitHub企业版](https://enterprise.github.com)产品旨在帮助我们将GitHub扩展到更多的人群。所以，无论你是被防火墙困住还是完全访问网络，我们希望GitHub能为你工作。”
> 
> -[Chris Wanstrath](https://www.forbes.com/sites/venkateshrao/2012/03/27/github-and-the-democratization-of-programming/?sh=245cb24540a5)

通过创新传统的代码托管业务模型，GitHub赚足了钱，以扩展其团队并进一步推动产品体验的迭代，超过了以往的任何产品。但是，深入的收入关注和自给自足的增长不仅仅是一个选择——对于联合创始人们来说，风险投资并不是一个选择。直到2010年，“制造开发者工具的公司未能吸引到任何重要的投资。”

> "即使像2013年的[Facebook收购Parse](https://techcrunch.com/2013/04/25/facebook-parse/)这样的工具公司退出，报导的8000万美元也被认为偏低。对于某些人来说，开发者工具永远是风险投资（VC）的死区。"
> 
> -[Forbes](https://www.forbes.com/sites/forbestechcouncil/2019/10/16/developer-tools-quietly-become-a-growth-story/?sh=5b0b511c6dea)

这需要Heroku、Atlassian、Stripe、Twilio和Sendgrid的投资支持才能打开市场。直到六年后，GitHub才从大胆的安德森·霍洛维茨那里筹集了1亿美元，这被宣传为[有史以来最大的A轮融资](https://www.forecastr.co/blog/the-largest-series-a-funding-rounds-in-history-5-awesome-stories#:~:text=GitHub%20was%20founded%20in%20San,ever%20written%20at%20that%20time)。

## 谁没有采用GitHub?[](/blog/github-monopoly-on-code-hosting?utm_source=urbanisierung-dev&ref=urbanisierung-dev#who-didnt-adopt-github)

Google内部也从未采用GitHub。在2000年代，他们使用Perforce，后来开发了名为Piper的定制版本控制系统。Google不仅拥有独特的先进版本控制工具，而且据我所知，他们还发明了基于Web的代码审查。[他们2004年的早期代码审查仪表板](https://www.youtube.com/watch?v=sMql3Di4Kgc)从Gmail获得灵感，为企业软件开发工作流设定了黄金标准。他们对Git和GitHub没有直接需求。

Facebook内部也从未采用GitHub进行开发。约2010年，Facebook的Evan Pricely开发了Phabricator - 早于GitHub正式推出企业自托管服务一年。即使GitHub的服务当时已经可用，Facebook可能仍然选择开发他们自己的工具，以更好地与定制的内部解决方案集成，并帮助处理即使是原始Git也难以应对的单仓库规模问题。此外，[Facebook转向了Mercurial](https://graphite.dev/blog/why-facebook-doesnt-use-git)，而GitHub从未正式支持过Mercurial。

一些独角兽公司，比如Airbnb，早早采用了GitHub。其他一些著名公司，如Uber和Pinterest，则自行托管并分叉了Phabricator。虽然我不能确定，但我怀疑选择Phabricator的公司，可能因为：1）它是最好的自托管开源源代码管理服务；2）这些公司充满了前Facebook员工，想念他们以前的工具链。

GitLab于2011年推出，通过专注于成为完整的DevOps平台，与GitHub形成对比，而不是"社交编码"。他们紧跟DevOps蓬勃发展的趋势，大力依赖CI/CD，在一些主要的科技公司如Nvidia中获得市场份额。

在2024年的Graphite项目中，我与数百甚至数千个高科技工程团队交流。极少听说不使用GitHub的公司。StackOverflow的2022年调查称GitHub的专业市场份额仅为GitLab的两倍。但实际上，我可以通过个人经验说，95%的现代科技公司使用GitHub，其中只有少数公司自行托管GitHub企业版。其余的几家公司使用GitLab、Phabricator、Gerrit或者定制的[内部代码托管平台](https://survey.stackoverflow.co/2022#section-version-control-version-control-platforms)。

## 代码托管和GitHub的未来[](/blog/github-monopoly-on-code-hosting?utm_source=urbanisierung-dev&ref=urbanisierung-dev#the-future-of-github-and-code-hosting)

> Linus Torvalds，Git 软件的原始开发者，曾高度赞扬 GitHub，称“GitHub 的托管非常出色。他们在这方面做得很好。我认为 GitHub 在使开源项目托管变得如此简单方面应该受到极大赞扬。” 然而，他也尖锐批评了 GitHub 的合并界面实现，称“Git 自带一个不错的拉取请求生成模块，但 GitHub 却决定用他们自己的完全低劣的版本替换它。因此，我认为 GitHub 在这些方面是无用的。它在托管方面很好，但是拉取请求和在线提交编辑纯粹是垃圾。”
> 
> -[《连线》（Wired）](https://www.wired.com/2012/05/torvalds-github/)

代码托管的未来是什么？在克雷·克里斯滕森（Clay Christensen）的著名书籍《创新者的窘境》中，克里斯滕森认为早期创新产品通常作为集成解决方案开始。我认为 GitHub 和 GitLab 是集成解决方案的例子，为希望管理其源代码的团队提供了一站式服务。

Christensen 认为，随着市场成熟，解决方案变得专业化和模块化。我们已经看到在“社交编码”的几个领域中开始出现这种情况。Jira 和 Linear 提供模块化的问题追踪，而 Jenkins 和 Buildkite 提供模块化的持续集成解决方案。GitHub 最初是 Git 存储库托管解决方案，但随着时间的推移，BitBucket 和 AWS Code Commit 也提供了相同的解决方案。GitHub 现在提供了集成的合并队列，但现在已经有了更专业化的选择，例如 Mergify、Aviator、Trunk 和 Graphite。

GitHub 通过强大的网络效应和产品特性（如分支、论坛式评论和调节）在开源代码领域保持垄断地位。闭源存储库最初由于其在 Git 托管方面的专长而采用 GitHub，但现在这已成为一种商品能力。GitHub 的社交功能在私人公司几乎无用，因为讨论通常在 Slack、Notion、Linear 和 Zoom 上进行。

我认为未来将在开源开发工具和闭源开发工具之间分化。开源协作需要讨论、个人资料、调节、分支和发现。闭源协作需要代码变更在几小时内审核完毕，而不是几天。它需要基于主干的工作流程、合并队列、紧急程序和持续集成/持续部署协调。在两种情况下都存在重叠，但我相信随着时间的推移，世界将进一步专注于不同用例的不同解决方案。

我们已经看到 Facebook 和 Google 提供的专业化解决方案。通过独立于 GitHub 的开源约束演变其源代码控制系统，它们创造了强大的模式，例如 Google 的 PR 收件箱和 Facebook 的堆叠差异。

我希望模块化能够发展到每个工程师都可以选择如何托管他们的源代码的程度，完全独立于他们用来更改源代码的工具。我们已经在开发的其他方面看到了这一点，比如开发人员可以自由选择他们喜欢的集成开发环境或云托管提供商。**GitHub 凭借更好地专注于代码托管和社交编码而赢得了他们的垄断地位，但这并非终点。**未来的某一天，我希望开发人员有五个可行的选项可以选择托管，而不仅仅是一个选项。

> 💡 免责声明：我年轻到在 GitHub 出现之前并没有专业编码的经历。本文中的大部分信息都是从在线来源、采访和我在这个领域工作中的学习中拼凑而成的。这是我对事物发展及其未来走向的最佳近似。

## 完整时间线[](/blog/github-monopoly-on-code-hosting?utm_source=urbanisierung-dev&ref=urbanisierung-dev#the-full-timeline)

1.  **1999:** [SourceForge](https://sourceforge.net/about) 上线，成为第一个免费开源托管提供商。

1.  **Pre-2004:** 使用 [CVS](https://cvs.nongnu.org/) 和 [SVN](https://subversion.apache.org/) 进行版本控制；SourceForge 在托管开源项目方面处于领先地位。

1.  **2004:** Linus Torvalds 创建 [Git](https://en.wikipedia.org/wiki/Git)，通过分布式系统革新了版本控制。

1.  **2006:** [Google Code](https://code.google.com/archive/about) 推出，最初支持 SVN。

1.  **2008:** [GitHub](https://github.com/about) 成立，提供基于 Git 的代码库托管，并专注于社交编码。

1.  **2009:** SourceForge 添加了 Git 支持；GitHub 推出了自托管版本，为 GitHub Enterprise 打下了基础。

1.  **2010:** Facebook 开发 [Phabricator](https://www.phacility.com/phabricator/)，一个基于 Web 的代码审查和软件开发工具套件。

1.  **2011:** [GitLab](https://about.gitlab.com/company/) 成立，强调完整的 DevOps 平台；Google Code 添加了 Git 支持。

1.  **2012:** GitHub 推出 GitHub Enterprise，满足大型组织对私有托管的需求。

1.  **2016:** Google Code 关闭，突显出 GitHub 在代码托管领域的日益增长的优势。

1.  **2018:** [GitHub 推出 GitHub Actions](https://resources.github.com/devops/tools/automation/actions/)，自动化软件开发流程中的工作流。

1.  **2021:** Phabricator 不再积极维护，推动更多用户转向 GitHub。
