<!--yml

category: 未分类

date: 2024-05-27 14:51:41

-->

# Fig 停用，迁移到亚马逊命令行 CodeWhisperer | Fig

> 来源：[https://fig.io/blog/post/fig-is-sunsetting](https://fig.io/blog/post/fig-is-sunsetting)

2024 年新年快乐！去年对我们团队来说是重要的一年：我们在八月加入了亚马逊，到十一月我们已经发布了[亚马逊命令行 CodeWhisperer](https://docs.aws.amazon.com/codewhisperer/latest/userguide/command-line-getting-started-installing.html)。

我们深知生成式 AI 对开发者构建方式的影响。我们正在努力创新，今年稍后将有许多令人激动的新功能推出到命令行 CodeWhisperer。因此，我们决定全力以赴地专注于这些新功能，并因此将于2024年9月1日停用 Fig。

我们将继续支持 Fig 直到2024年9月1日。在此之前，我们鼓励用户迁移到[命令行 CodeWhisperer](https://docs.aws.amazon.com/codewhisperer/latest/userguide/command-line-getting-started-installing.html)。为了尽可能简化过渡，用户可以直接从 Fig 仪表板升级到命令行 CodeWhisperer（更多信息请见下文）。

[命令行 CodeWhisperer](https://docs.aws.amazon.com/codewhisperer/latest/userguide/command-line-getting-started-installing.html) 提供 Fig 核心功能，包括 1) 500 多个 CLI 的 IDE 风格自动完成和 2) 自然语言转换为 bash。个人层级免费，设计更快速和更可靠，比 Fig 更出色。2024 年我们将推出几项令人兴奋的新功能，包括 Linux 支持、AI 聊天、内联 AI 自动完成、SSH 自动完成和私有自动完成。

命令行 CodeWhisperer 不包含类似于 Fig 脚本、Dotfiles、插件或服务器的功能，我们目前不计划支持这些功能。

Fig 仪表板 UI 将为您处理迁移 - 仅需几分钟。迁移过程将把您的数据导出到本地设备，安装命令行 CodeWhisperer，传输您的设置，然后卸载 Fig。安装了 CodeWhisperer 应用后，您将被提示授予 macOS 辅助功能权限，然后登录或注册。

**注意**: 您需要更新至 v2.18 才能看到新的 Fig 仪表板 UI。

Fig 将在2024年9月1日删除所有用户数据。用户可以通过运行 `fig export` 将其数据导出到本地设备。Fig 到命令行 CodeWhisperer 迁移流程将自动为用户执行此操作。

是的！在 Fig 停用之前，您可以同时使用 Fig 和命令行 CodeWhisperer。安装了这两个工具后，Fig 的自动完成功能将关闭，转而使用更可靠的命令行 CodeWhisperer。这意味着您可以继续使用 Fig 进行脚本、Dotfiles、插件和服务器，直到2024年9月1日，同时开始使用命令行 CodeWhisperer 提供的更可靠的 CLI 完成和 AI 功能。

如果您对迁移有任何疑问，请在我们的[公共社区](https://github.com/withfig/fig/issues/new?assignees=&labels=&projects=&template=1_general_question.md)中创建讨论。

对于所有 Fig 的用户、客户和贡献者，我们对多年来您们的反馈、贡献和支持深表感激。我们对我们所做的影响感到非常高兴，并且我们非常期待在未来继续与大家合作！

Brendan、Matt 和亚马逊 CodeWhisperer 的命令行团队。
