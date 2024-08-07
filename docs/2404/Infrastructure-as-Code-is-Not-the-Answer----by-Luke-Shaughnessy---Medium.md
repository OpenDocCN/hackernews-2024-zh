<!--yml

类别: 未分类

日期：2024-05-27 12:57:07

-->

# 基础设施即代码不是答案！ | by Luke Shaughnessy | Medium

> 来源：[https://lukeshaughnessy.medium.com/infrastructure-as-code-is-not-the-answer-cfaf4882dcba](https://lukeshaughnessy.medium.com/infrastructure-as-code-is-not-the-answer-cfaf4882dcba)

# 基础设施即代码不是答案！

你当然知道销售宣传。如果你在DevOps、平台或SRE领域工作，你可能已经自己进行过。基础设施即代码！好处多多，多样化且不言自明：

+   **它是可复制的！**

+   **它是自我文档化的！**

+   **它是可见的！**

+   **它防止错误！**

+   **它降低成本！**

+   **它防止漂移！**

+   **它防止枯燥工作并增加乐趣！**

+   **自助服务！**

所有这些事情无疑都是真实的。在理想的世界里，所有这些好处都将是显而易见的，并且对于任何愿意进行前期投资将部署基础架构转换为代码和配置管理的组织来说都是巨大的好处。而且我们大多数人都是这样的。但在非理想的情况下呢？采用基础架构即代码(IAC)的实际成本、风险和困难在哪里？在现实的耀眼强光下，炫目的销售宣传从何处开始融化？让我们看看上述的一些好处，并检验一些假设。

**它是可复制的！**

嗯，有点像。我会说基础设施代码在一定时间内是非常可复制的 — 但后来……事情变得不太靠谱。软件包过时了。功能被弃用并停止工作。代码语法发生变化。镜像不再可用。Dockerhub要求你付费。某人的用户帐户被建立起来，而他们已经不在这里工作了。许多使部署工作的相互依赖关系随时间而改变。关键是部署代码有一个保质期，如果你不不断地更新和更新它，当你突然需要它时它将不起作用。

**它是自我文档化的！**

也是这样。但这个论点有几个缺陷。第一个问题与上述问题有关。你的代码在一段时间后开始变得老旧。这意味着第一次查看你的代码的工程师可能会开始沿着兔子洞追逐，试图让一些旧模块在新的云环境中工作，在那里关于如何做事情的一堆假设不再适用。我有同事花了几天时间追踪失败的部署并逐步调试事物，并且因为他们在代码中看到的东西与现实不符而感到完全困惑。事实上，如果他们只是重新开始编写新代码，可能会更快更好。

第二个问题是，依我看来，代码真的不适合用作文档。我自己写过代码，六个月后回头看，完全搞不懂我当时在干嘛。写代码时可能会使用各种折中方法和明显错误的东西，因为很可能在做的时候还在学习这些。在此期间，你可能学到了很多东西，当看到自己最初的几次尝试时可能会想哭。基础设施代码也不例外，可能会固定各种不稳定的假设和方法，只会在未来让后来的工程师感到困惑。

**它是可见的！**

是的，它是可见的 —— 就像我在我们首席技术官办公室的墙上喷涂西里尔文涂鸦一样。这看起来很酷和激进，但没人知道它写的是什么。问题在于代码，它是 *代码*。非工程师不容易阅读，甚至不熟悉 Terraform 或其他工具的工程师也不容易理解。要理解其中的信息需要实践和培训，有时即使是有经验的工程师可能也需要一些时间来完全弄清楚所有信息的位置，特别是如果有大量嵌套文件和依赖项以及调用脚本的脚本。重点是，是的，你可以看到它，但这并不意味着你能迅速或轻松地理解它。

**它防止错误！**

或者，它可能像氢弹核心周围的重氘外壳一样放大错误。我曾听说过一个工程师在他的笔记本上运行了一个“terraform destroy”命令 —— 但是在错误的标签页中。他在生产目录中 —— 开始删除生产环境。意识到错误后，他按下了 Control-C，但为时已晚。许多东西已经消失了，而且更糟糕的是，杀死进程使状态文件处于混乱状态。他们通过手动修复状态文件并重新运行 terraform 来恢复了问题，几个小时后问题得到解决。但这真的是一个任何人都可能犯的非常容易的错误。

**它降低成本！**

如果一切都完美无缺，永远不会改变，而且你一直做同样的事情，那当然。但根据我的经验，保持代码的最新状态需要消耗大量工程时间。工程师每天花费数小时来调试、故障排除、学习和编写代码。如果只需登录控制台点击几个框，可能几分钟就能自动化一个任务，但可能需要几天时间才能自动化，你真的需要花费几天时间来自动化吗？

**它能防止漂移！**

只要你只创建代码来改变环境，这就是真的。一旦有人可以访问控制台，就可能会出现漂移。那么是谁在控制台登录并进行异步更改呢？嗯，有时候是你自己。因为出现了故障，你需要在负载均衡器中添加新的证书，或者因为你正在遭受 DDOS 攻击，需要添加 CloudFront Distro 来吸收传入的请求。或者因为你需要扩展某些东西以处理意外的负载增加。显然，你可以事后回过头来调整代码以反映你所做的更改。这似乎很愚蠢，但据我经验，这种事情在现实世界中经常发生。

**它能减少重复劳动，增加乐趣！**

你想要辛勤工作吗？如何将 Terraform 版本从 0.10 更新到 Terraform 1.3？语法上有相当大的变化，一些自动化工具可以处理。但旧版本需要大量奇怪的变通方法，在新版本中得到改进。这意味着仅靠自动化工具重新做语法是不够的；你需要从头到尾重构整个代码库。我有同事在进行 Terraform 迁移工作已经超过一年。这是一项巨大的工作，需要大量工程师的工时来更新。有时很难说服老板需要这样做，特别是如果你已经在开发阶段消耗了大量的精力。

**自助服务！**

平台团队的圣杯 —— 为整个公司提供自助 PAAS。你的所有基础设施和服务都被彻底自动化，非 DevOps 工程师可以拉取 git 仓库，编辑一些模板文件，然后提交，你的 CI/CD 系统会自动部署。所有日志记录、报警、指标和安全策略都已经内置。我真心喜欢这个想法。我目前正在努力让我的现任公司达到这种状态。但现实又再次对此提出了挑战。例如，我们曾经有一个非常复杂的自动化系统来管理 Github 用户账户。我们会编辑文件，推送到 CI，然后用户和仓库就会被创建。然而，我们意识到这是不必要的复杂化，并且显著减慢了新员工和承包商的入职速度，因为他们需要等待技术人员编辑自动化文件。只需确保团队经理经过审查和安全策略培训，并直接授予他们访问 Github 权限，要快得多且更简单。我们相信他们会以正确的方式进行操作以添加新账户并分配权限。

**结论**

所以这是否意味着我们需要放弃整个IAC概念，回到在Console中点击的操作？不一定总是这样？我觉得这是最好的答案。和过去的好想法一样——比如敏捷、Scrum、DevOps、无服务器架构、微服务，所有这些东西都能带来很大价值。但你不能过于专注，以为IAC是唯一的答案，就像它是你人生的全部意义。IAC有其适用场景，但这仅仅是场景之一。并非所有的情况都相同，总会有棘手的细节需要处理。一个好的工程师或工程经理知道何时放弃理想化的自动化完美，并在需要时允许人们具备一定的灵活性。

**补记 2023年9月22日**

哇哦！我惊讶于人们还在阅读并将此文章添加到他们的阅读列表中。而且，确实有大量在评论区展开激烈反馈！关于这些评论；让我反驳一下；)

1.  是的，这是一个点击标题的文章标题，你点击了它，对吧？这是我写的所有文章中获得最多互动的一篇文章，你们只能接受它。然而，许多评论来自动态浏览标题而未继续阅读的人。你会发现，如果你完整阅读，我从未说过IAC不应被使用，或者它可能不是最佳解决方案。但如果标题写为“如果实施不良，IAC存在风险且不一定适应所有用例”，你可能会打个哈欠并跳过整篇文章。而事实是，确实如此：从绝对意义上说，IAC并不是唯一答案。有很多答案，IAC只是其中之一。我的观点是，作为工程师，我们应该灵活思考而非盲目遵循教条。总有可能会有更好的解决方案，这正是我们的工作要去思考的。

1.  “都去哪儿了，聪明的家伙，所有的建议呢？”这是评论中看似一半的讨论内容。确实，我并未为我指出的每一个问题提供解决方案。我面向的是负责思考问题解决方案的工程师，暴露问题应该像对他们有吸引力的诱饵一样。我相信你们都能想到很多可以解决我列出的问题的方法。就从仅仅添加一个“不”字开始解决起。不要让你的代码过期。不要编写晦涩不清的代码。不要忘记对代码库进行维护和更新。不要让工程师在终端执行Terraform Destroy（使用CI，费解）看看这里的意思？我想大多数阅读者理解了，但有些人因为没有明确表达而感到困惑，所以接下来我将详细阐述。

1.  “这位作者显然从未参与过复杂基础设施的工作！”是的，他有。已有20年了。我是在穿着外套、站在服务器房冷通道里写PXE引导脚本的时候开始自动化的，因为离回到我的办公桌太远了。我见过很多这样的技术进出。Puppet，Chef，Ansible，Terraform，YAML K8s清单。事实上，我刚刚开始担任一家公司的DevOps经理，该公司最近由于RIFs、自然流失和其他私募股权投资的把戏而失去了大部分工程人员。这意味着我们这些新人需要从头开始逆向工程所有现有的基础设施代码；尽管有些代码可以工作，但很多都不行。最终，我们不得不假设没有任何东西能工作，并且基本上重新开始。将部分工作的代码变成完全工作的代码比重新开始更难。公司把未来押在比毫无用处更糟糕的代码上；它是一根拖累，使得维持运行的难度远远超出必要的范围。文章中提到的许多问题都源自这段经历。

1.  控制台生活 — 正如我之前所说，使用情况各不相同。有时候，用户友好的图形界面完全可以，特别是当您不真正需要全部的基础设施即代码的开销时。用户账户管理就是一个很好的例子。真的不应该让运维人员去添加和移除用户，这就是为什么有项目经理、经理和团队领导的存在。教一个销售经理使用Git来添加新员工是荒谬的。关键在于，您需要仔细考虑何时实际上需要将事物纳入您的自动化代码的范围内。

总之，感谢过去几年里抽出时间阅读这篇文章的所有人，希望对您有所帮助！
