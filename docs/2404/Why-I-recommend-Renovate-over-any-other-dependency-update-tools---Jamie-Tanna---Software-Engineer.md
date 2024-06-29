<!--yml

category: 未分类

date: 2024-05-27 13:08:49

-->

# 为什么我推荐使用Renovate而不是其他任何依赖更新工具 · Jamie Tanna | 软件工程师

> 来源：[https://www.jvt.me/posts/2024/04/12/use-renovate/](https://www.jvt.me/posts/2024/04/12/use-renovate/)

如果你之前读过我的博客，或者在工作或开源世界中与我互动过，你可能知道我是[Renovate](https://docs.renovatebot.com/)的铁杆粉丝。

对于那些不了解的人，Renovate是依赖更新工具领域的重要角色之一，通常与Dependabot或Snyk进行比较。

在过去的大约5年里，我一直在使用Renovate，同时也混合使用Dependabot和Snyk来保持稳定。我*非常喜欢*Renovate，并且确保可以在可能的情况下运行Renovate。

我还有在自托管模式下操作Renovate的经验，以及使用[Mend提供的托管Renovate应用](https://developer.mend.io)和Mend Enterprise SAAS，并且[我写了关于自托管Renovate的经验教训](https://www.jvt.me/posts/2024/05/03/renovate-self-hosting-lessons/)。

Renovate拥有一些真正关键的功能，使其在竞争中脱颖而出，我相信在我写这篇文章的时间里，他们已经推出了一些新功能 😻

（旁注：我现在主要撰写这篇博客，因为我最近一直在宣扬Renovate的好处，而不是像在Deliveroo时写另一份内部文档那样，我想[将其作为博客式文档](https://www.jvt.me/posts/2017/06/25/blogumentation/)来写）。

## 先前的艺术

几年前，我写了一篇关于[使用Renovate的一些技巧](https://www.jvt.me/posts/2021/09/26/whitesource-renovate-tips/)，以便更轻松地保持软件更新，我现在依然坚持这些观点。

自那时以来，我已经在多个不同的生态系统、仓库规模和合并依赖更新的舒适度上工作过，并学到了更多关于有效使用Renovate的经验 - 但通过这一切，我仍然非常确信Renovate是该生态系统中最好的工具。

## 可配置性

Renovate具有*极高的*可配置性，有[数十个配置选项](https://docs.renovatebot.com/configuration-options/)可以调整您的体验。但是Renovate并不会变得“过于”可配置，以至于您花更多时间调整配置而不是进行更改，但它足够灵活，很可能您可以按需进行操作。

## 中心化的、可共享的预设

当我们在Deliveroo推出Dependabot时，这真的让我印象深刻，我们的团队管理着大约30个仓库，所以对于每个仓库，我们都需要手工制作一个`dependabot.yml`，因为Dependabot需要告诉它哪些目录包含哪些依赖项，以及多久更新一次它们。

使用大约一周后，我们开始需要调整其中一些仓库的配置以减少噪音，然后需要在所有仓库中提出PR并进行更新。

因为Dependabot需要每个存储库的特定配置，这并不容易自动化，即使有[用于自动化一些大量更新的很好的工具](https://www.jvt.me/posts/2023/01/21/bulk-git-repo-changes/)，这使得这个过程相当繁琐和令人沮丧。

与Renovate相比，这里有对[可共享的配置预设](https://docs.renovatebot.com/config-presets/)的一流支持，您可以“扩展”多个配置。

这使得希望保持一致性的团队（例如，他们希望多久收到一次AWS和Google Cloud SDK更新，或者希望在PR上有哪些标签）可以为他们的团队创建一个共享预设，定义这些。然后，他们的每个存储库可以“扩展”此配置，并在存储库特定级别上定义自己的配置。

这也使得为您的组织提供良好的默认设置成为可能，例如`base.json`：

```
{  "$schema": "https://docs.renovatebot.com/renovate-schema.json", "postUpdateOptions": [ "gomodTidy", "gomodUpdateImportPaths" ], "regexManagers": [ { "fileMatch": [ "^Makefile$" ], "matchStrings": [ "curl .*https://raw.githubusercontent.com/golangci/golangci-lint/master/install.sh | sh -s -- .* (?<currentValue>.*?)\\n" ], "depNameTemplate": "github.com/golangci/golangci-lint", "datasourceTemplate": "go" } ] } 
```

这样可以为团队提供一个相对有见地的良好起点，有一个`default.json`：

```
{  "$schema": "https://docs.renovatebot.com/renovate-schema.json", "extends": [ "local>your-org-here/renovate-config:base", "config:best-practices" ] } 
```

然后，团队只需在其存储库的`renovate.json`中设置以下内容，他们就能获得入门的好处，减少了许多初始工作：

```
{  "$schema": "https://docs.renovatebot.com/renovate-schema.json", "extends": [ "local>your-org-here/renovate-config" ] } 
```

就是这样 👏

## 良好的默认设置

关于手工制作`dependabot.yml`的评论，Renovate的一个很大的优点是有一些很好的默认设置，它可以自动检测您的存储库使用的生态系统，并适当地提出PR。

这种轻松入门真的非常棒，甚至可以为您的存储库提出[入门PR](https://docs.renovatebot.com/getting-started/installing-onboarding/#repository-onboarding)来使其更加简单。

例如，在Deliveroo，我们做到了[所有存储库的默认配置](https://www.jvt.me/posts/2023/01/30/renovate-global-defaults/)，为我们提供了一个简单的更新Docker镜像的手段，但是如果团队想要管理所有内容，他们可以添加一个`renovate.json`，这样功能会更加全面。

另外，Renovate还带有一些内置配置，例如[“最佳实践”指南](https://docs.renovatebot.com/upgrade-best-practices/)和相关预设，这使得更容易掌握社区的最佳实践，而不需要为了您认为最好的事物而纠缠不清。

如果您觉得`config:best-practices`有点多，作为一个起点，有`config:recommended`，您总是可以降级或排除您不想遵循的规则。或者如果您*真的*想要控制事物，`config:base`是您应该引入的最小值。

## 分组

如上所建议，可以调整如何更新不同的包，您可以[将多个更新分组到单个PR](https://docs.renovatebot.com/noise-reduction/#package-grouping)。

例如，假设您的应用程序中使用了 9 种不同的 AWS 服务，每次 SDK 更新时，您不想收到 9 个 PR，而是希望只收到一个。在这种情况下，您可以编写以下的 Renovate 配置：

```
{  "$schema": "https://docs.renovatebot.com/renovate-schema.json", "packageRules": [ { "groupName": "aws-sdk-go", "matchPackagePatterns": [  "^github.com/aws/aws-sdk-go-v2" ] } ] } 
```

这很棒的一点是，您还可以将所有补丁更新捆绑到一个单独的 PR 中：

```
{  "packageRules": [ { "matchPackagePatterns": [ "*" ], "matchUpdateTypes": [ "patch" ], "groupName": "all patch dependencies", "groupSlug": "all-patch" } ] } 
```

这是 Dependabot 最近添加的功能，这很棒，因为 Renovate 多年来就有了 😝

## 用于一次性升级

因为 Renovate 是一个可以在命令行上运行的开源工具，这意味着您也可以使用 [Renovate 一次性执行](https://www.jvt.me/posts/2022/12/12/renovate-one-off/) 的功能，例如将组织中的每个人都升级到给定依赖项的最低版本，或执行不经常执行的更新集。

我经常提到的一件事是 Renovate 支持大量的包管理器、包生态系统和版本工具。

Renovate 的 [机器人比较指南文档](https://docs.renovatebot.com/bot-comparison/) 中有一个很好的例子，链接到以下差异：

由于 Dependabot 更专注于在 GitHub 平台上使用（需要引证？），似乎不支持 CircleCI、GitLab CI 或其他竞争对手的工具。

## 添加对额外生态系统的支持

从您选择的依赖更新工具中您习惯的一件事是标准的包管理器支持，它会在您的存储库中更新 `go.mod` 或 `pom.xml`。

但是要理解的一点是，您的项目中有比包管理器中安装的依赖项更多的依赖关系。我认为 Renovate 很棒的一点是，除了管理 `Dockerfile`、`build.gradle`、`.gitlab-ci.yml` 等文件外，它还会管理您的 `.ruby-version` 或 ASDF 版本管理器的配置。

但是，有关一些非标准或组织特定的版本跟踪方式呢？

例如，安装 `golangci-lint` 代码检查工具推荐使用 `curl | sh`：

```
$(GOBIN)/golangci-lint:  curl -sSfL https://raw.githubusercontent.com/golangci/golangci-lint/master/install.sh | sh -s -- -b $(GOBIN) v1.57.2 
```

并且 `Dockerfile` 中通常会定义依赖项的版本：

```
ARG MONGODB_VERSION=6.0.4 
```

或者，如果您有像以下这样的自定义部署配置：

```
# this is an (imaginary) company specific deployment configuration file application:  lb: image:  uk-tooling/load-balancer@v3.0.0 
```

很不可能其他工具开箱即用支持这些功能，而 Renovate 是一样的。

但是 Renovate *确实* 让您能够自行管理这些版本。

使用 Renovate 的 [自定义管理器](https://docs.renovatebot.com/modules/manager/regex/) 我们可以编写这样的配置文件：

```
{  "$schema": "https://docs.renovatebot.com/renovate-schema.json", "regexManagers": [ { "fileMatch": [ "^Makefile$" ], "matchStrings": [ "curl .*https://raw.githubusercontent.com/golangci/golangci-lint/master/install.sh | sh -s -- .* (?<currentValue>.*?)\\n" ], "depNameTemplate": "github.com/golangci/golangci-lint", "datasourceTemplate": "go" } ] } 
```

这将允许我们管理 `golangci-lint` 的安装。

理想情况下，这种配置可能会被上游接受，使 Renovate 可以开箱即用地执行，但正如我们从上面的例子中看到的那样，可能会有组织或仓库特定的事物，因此将其上游化并不合理。

## 因为您可以做更多事情

而且您可以做更多的事情 - 我提到您可以在命令行上进行一次性更新，但例如我还编写了一个工具 [renovate-graph](https://gitlab.com/tanna.dev/renovate-graph)，它会获取 Renovate 检测到的依赖数据，并提供给您可以消费的 JSON 数据块。

这支撑了 [dependency-management-data](https://dmd.tanna.dev) 背后的大部分强大功能，并可以用于其他用途，比如将 Renovate 数据导出转换为软件材料清单（SBOM）[转换](https://www.jvt.me/posts/2023/11/03/renovate-to-sbom/)。

## 依赖项仪表板

我喜欢 Renovate 的一件事是您可以启用 [Dependency Dashboard](https://docs.renovatebot.com/key-concepts/dashboard/)，这为您提供了检测到的依赖项、开放的 PR、以及可能正在等待或失败更新的概览。

这是一个非常有用的见解，一目了然地显示我们在更新方面落后了多少，可以看到您是否想要花更多时间专注于更新，或者看看如何消除噪音。

当您将 PR 合并到默认分支时，Renovate 将重新基于开放的 PR，以便更轻松地审查，并确保与最新更改兼容。然而，如果您没有像更改那样频繁地进行更新，您可能会有大量的 PR 不断构建，这是能源和 CI 分钟的浪费。

相反，您可以使用依赖项仪表板，因为它具有例如[要求主要升级在手动批准后才能进行](https://docs.renovatebot.com/key-concepts/dashboard/#require-approval-for-major-updates)的能力，因为您可能需要一些人工干预来处理该 PR，并且只能在您真正准备处理它时提出。

## 开源

对我来说非常重要的一个因素是 Renovate 是开源的，并且开放给社区进行贡献。

虽然最好将其拆分为另一篇文章，但我喜欢 AGPL3，并且这是一种确保任何将 Renovate 作为平台托管的人都确保其用户获得访问源代码的好方法。

另外，Snyk 是专有的，而~~Dependabot 是开源的（或者说是源码可用的，不是“开源”）并带有[许可证](https://github.com/dependabot/dependabot-core?tab=readme-ov-file#license)，该许可证甚至不在SPDX许可证列表上~~。 **更新 2024-05-19**：Dependabot 现在采用 MIT 许可证 👏

Renovate 也被出色地设置为一个社区项目，他们每月都在发布数百个 PR，同时很好地管理社区。在过去几年中，我已经更积极地进行了一些贡献，看到了一些变化，这是一个管理良好的项目，也是我希望在某个时候能够复制的指示！

同时，背后支持Renovate的[Mend](https://mend.io)公司，投入了大量时间和金钱进行项目的开发，以及他们在其商业产品上的持续投入，这使得该项目具有可持续性并保持自由和开放。

## 出色的文档

借助卓越的社区和维护者的贡献，还有一些在技术写作和文档方面真正出色的工作，这使得许多任务变得简单易解决。

如果有比文档更复杂或定制的需求，通常可以在GitHub讨论中由社区回答，并且有可能转化为文档的改进，如果需要的话。

甚至还有一个很棒的[比较表，比较Renovate和其他依赖更新机器人](https://docs.renovatebot.com/bot-comparison/)，以及[关于如何最好地更新你的项目的建议](https://docs.renovatebot.com/upgrade-best-practices/)。

最近，通过阅读文档，我发现了一些之前不知道的页面和功能。

## 总体来说

总之，Renovate在许多方面都比任何其他替代品更好，我相信我还可以谈论更多让它出色的地方。

无论是自主托管以实现最大控制（例如能够访问内部工件注册表），还是运行托管应用以便操作更简单，或者偶尔在命令行上运行它，作为工程师，它都可能极大地提升你的体验。

希望你能去看看！
