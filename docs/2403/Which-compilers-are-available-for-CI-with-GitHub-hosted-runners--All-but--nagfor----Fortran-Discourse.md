<!--yml

分类：未分类

日期：2024-05-27 14:51:58

-->

# 在 GitHub 托管的运行器上可以使用哪些编译器进行 CI？除了 `nagfor` 之外的所有编译器 - Fortran 论坛

> 来源：[https://fortran-lang.discourse.group/t/which-compilers-are-available-for-ci-with-github-hosted-runners-all-but-nagfor/7554](https://fortran-lang.discourse.group/t/which-compilers-are-available-for-ci-with-github-hosted-runners-all-but-nagfor/7554)

[更新：此帖子被链接到 Hack 新闻并[引起了一些关注](https://news.ycombinator.com/item?id=39656405)。]

[测试一直在](https://github.com/orgs/libprima/discussions/39) PRIMA 的开发中起着核心作用，这些测试被 [GitHub Actions 自动化](https://github.com/libprima/prima?tab=readme-ov-file#how)。CI（持续集成）和 GitHub Actions 的概念不仅改变了我的生活，而且也让我眼界大开。它们使我能够以无法想象的强度和广度测试我的代码。

GitHub Actions 结合 **GitHub 托管的运行器** 尤为实用。有了它们，你可以在一个全新的环境中测试代码。此外，你无需担心由于测试而弄乱你的计算机。最重要的是，你可以访问云中几乎无限数量的计算机，而不是被办公室里可用的物理计算机所限制。

在 GitHub Actions 上与 **GitHub 托管的运行器** 一起使用的 Fortran 编译器有哪些？根据我的经验，市场上所有主要的编译器都支持，**除了来自 NAG 的 `nagfor`**。这里甚至包括已停用的编译器，如 [g95](https://en.wikipedia.org/wiki/G95) 和 [Oracle sunf95](https://www.oracle.com/tools/developerstudio/downloads/developer-studio-jsp.html)，但 [IBM Open XL Fortran compiler](https://www.ibm.com/products/open-xl-fortran-aix-compiler-power) 和 [Cray Fortran compiler](https://cpe.ext.hpe.com/docs/cce/) 不支持，因为它们仅适用于特定供应商的平台。

`nagfor` 由于其 [特殊的许可管理方式](https://support.nag.com/content/kusari-frequently-asked-questions) 不能与 **GitHub 托管的运行器** 一起使用。每个许可证只能在具有特定“Kusari ID”的计算机上使用。你可以在一台计算机上停用许可证并将其移动到另一台计算机上，但你需要发送电子邮件给 NAG，告知新的 ID 并请求新的许可证密钥 — 这是一个无法自动化的过程。

在 **一切都在向云端迁移** 的今天，成为这样的一个例外并不是特别理想，甚至 [MATLAB 也可以在 GitHub 托管的运行器上使用](https://github.com/matlab-actions/setup-matlab)，更不用说其他编译器，如 Intel `ifx`、NVIDIA `nvfortran` 和 AOCC `flang`。我认为这是一件迫切需要处理的事情 — 即使这并不是我的事情。我曾经向 NAG 支持咨询过将来是否会支持 GitHub 托管的运行器，不幸的是，回复是 —

> “我们的开发人员认为这个项目与我们的产品无关。”

所以，完全没有希望了。**对于如此优秀的编译器，真是可惜！**

* * *

* * *

P.S.:

如何在GitHub Actions上使用GitHub托管的运行器使编译器可用？除了由[@awvwgk](https://github.com/fortran-lang/setup-fortran)和其他贡献者提供的精彩[`fortran-lang/setup-fortran`](https://github.com/fortran-lang/setup-fortran)，我还使用以下脚本，这些脚本维护在[GitHub - equipez/github_actions_scripts](https://github.com/equipez/github_actions_scripts)。请注意，我的脚本是自制的，仅用于个人使用，并且仅安装编译器的最新可用版本，而您可以使用[`fortran-lang/setup-fortran`](https://github.com/fortran-lang/setup-fortran)控制版本。**这些脚本不适用于本地机器，因为它们可能对您的系统进行不必要的更改。**

查看[我的工作流程](https://github.com/libprima/prima/blob/ec42cb0c49e1425d4fe5838dcc20c2d3891308c9/.github/workflows/lint_hosted.yml#L89-L114)作为在GitHub托管的运行器上使用这些脚本的具体示例。
