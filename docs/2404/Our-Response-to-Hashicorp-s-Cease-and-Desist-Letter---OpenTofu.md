<!--yml

category: 未分类

date: 2024-05-27 13:07:04

-->

# 我们对 Hashicorp 停止与终止信函的回应 | OpenTofu

> 来源：[https://opentofu.org/blog/our-response-to-hashicorps-cease-and-desist/](https://opentofu.org/blog/our-response-to-hashicorps-cease-and-desist/)

4月3日，我们收到了 HashiCorp 关于我们在 OpenTofu 中实现“removed”块的版权侵权的停止和终止信函，指控我们核心开发者的版权侵权。同一天还发现有一篇文章发表了相同的指控。我们已调查了这些主张，并发布了 C&D 信函、我们的回应以及源代码来源文档。

**OpenTofu团队坚决反对任何关于其误用、错误引用或以其他方式误用 HashiCorp BSL 代码的暗示。所有这些声明都毫无事实根据。**

HashiCorp 在一封停止和终止信函中声称版权侵权。这些主张**完全没有根据**。

涉及的代码清楚地显示是从早期MPL-2.0许可的旧代码复制而来。当他们实现此功能时，HashiCorp似乎也复制了相同的代码。我们详细的SCO分析以及他们自己的注释都清楚显示了这一点。

*为了防止对个人的进一步骚扰，我们已从这些文件中删除了任何个人信息。*

尽管发生了这些事件，我们已成功开发了 OpenTofu 1.7 的重要功能，包括状态加密，“import” 块的“for_each”实现，以及新支持的**提供者定义函数**，这些都是最近发布的提供者插件协议支持的。

在此，我们将在下周发布一个新的预发布版本，并且我们渴望从社区中获得反馈。

— OpenTofu 团队

* * *

*这篇博客文章中的图片包含 HashiCorp 根据 BUSL-1.1 许可的代码。然而，出于本文的目的，我们在[17 U.S. Code § 107](https://www.govinfo.gov/content/pkg/USCODE-2022-title17/html/USCODE-2022-title17-chap1-sec107.htm) 下进行非商业的转型公平使用。

您可以在[美国版权局的网站](https://www.copyright.gov/fair-use/)了解更多关于公平使用的信息。*
