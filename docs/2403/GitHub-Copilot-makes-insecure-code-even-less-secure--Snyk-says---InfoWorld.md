<!--yml

类别：未分类

日期：2024年05月27日 14:49:32

-->

# GitHub Copilot makes insecure code even less secure, Snyk says | InfoWorld

> 来源：[https://www.infoworld.com/article/3713141/github-copilot-makes-insecure-code-even-less-secure-snyk-says.html](https://www.infoworld.com/article/3713141/github-copilot-makes-insecure-code-even-less-secure-snyk-says.html)

GitHub的AI驱动编码助手，[GitHub Copilot](https://github.blog/2022-06-21-github-copilot-is-generally-available-to-all-developers/)，可能在用户现有代码库中存在安全问题时建议不安全的代码，根据开发者安全公司Snyk的说法。

GitHub Copilot可以在代码中复制现有的安全问题，Snyk在2024年2月22日发布的博客文章中说道。“这意味着项目中现有的安全债务会使使用Copilot的开发者更加不安全，”公司称。然而，GitHub Copilot在没有安全问题的项目中，不太可能建议不安全的代码，因为它没有不安全代码的上下文可供参考。

[生成式AI](https://www.infoworld.com/article/3689973/what-is-generative-ai-artificial-intelligence-that-creates.html)编码助手，例如[GitHub Copilot](https://www.infoworld.com/article/3699140/review-codewhisperer-bard-and-copilot.html)，[Amazon CodeWhisperer](https://www.infoworld.com/article/3699140/review-codewhisperer-bard-and-copilot.html)，以及[ChatGPT](https://www.infoworld.com/article/3689172/chatgpt-and-software-development.html)，显著提升了生产力和代码效率，Snyk表示。但这些工具并不理解代码语义，因此不能评判其安全性。

GitHub Copilot根据从大量现有代码库中学习到的模式和结构生成代码片段。虽然这种方法有其优势，但在安全方面也有明显的缺陷，Snyk指出。Copilot的代码建议可能会无意中复制存在于邻近文件中的现有安全漏洞和不良实践。

为了减少AI助手生成的代码中现有安全问题的重复，Snyk建议采取以下步骤：

+   开发者应对代码进行手动审核。

+   安全团队应设置SAST（安全应用程序安全测试）的防护措施，包括政策。

+   开发者应遵循安全编码准则。

+   安全团队应向开发团队提供培训和意识，并根据团队优先级和分级处理问题积压。

+   执行团队应强制执行安全防护措施。

Snyk的数据显示，平均商业软件项目的第一方代码中平均有40个漏洞，其中近三分之一是高严重性问题。“这是AI生成工具可以通过使用这些漏洞作为上下文来复制代码的环境，”Snyk表示。Snyk在商业项目中看到的最常见问题包括跨站脚本、路径遍历、SQL注入以及硬编码的秘密和凭证。

GitHub无法在周三下午晚些时候做出回应，以回应Snyk关于GitHub Copilot的评论。

版权所有 © 2024 IDG通讯公司。
