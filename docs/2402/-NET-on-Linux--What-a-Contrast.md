<!--yml

类别：未分类

日期：2024-05-27 14:33:24

-->

# .NET 在 Linux 上：形成鲜明对比

> 来源：[https://two-wrongs.com/dotnet-on-linux-update.html](https://two-wrongs.com/dotnet-on-linux-update.html)

大约五年前，我写了[关于 .net 在非 Windows 平台上的简要历史](dotnet-on-non-windows-platforms-brief-historic-summary.html)。我现在偶然发现了那篇文章，它让我重新回忆起了在 Linux 和 MacOS 上开发 .net 时的创伤性经历。

即使在 2019 年，*在* Linux 上（生产环境中）运行 .net 代码基本上没问题，但是开发的文档和工具根本不成体系。¹ 我开始接触 .net 开发时，当时的新雇主正在从 .net Framework 迁移到 .net Core，所以我的非 Windows 工作电脑成为了各种意想不到的兼容性问题的实验田。我仍然不完全明白为什么我认为把我甚至连 *编译* 都无法完成的应用程序迁移到新的框架是个好主意，但最后一切都很顺利。（不仅是这些应用程序使用了 fcl，它们还与 Windows 功能集成。）

那个经历和文章是在 .net Core 2.1 是最新的生产版本时写的。在某种意义上，那似乎就是昨天的事，但在另一个意义上，它却是另一个世界。甚至只是两三年后，工具和文档的改进已经到了我没预料到的水平。今天，我仍然写 .net 代码（为另一家雇主），几乎分不清我是在 Windows 还是 Linux 上。

这反映了一个拥有庞大资金和愿意投资的组织在短时间内可以推动事情发生多大的变化 - 但同时，我认为，这也反映了其他公司（我指的是你，JetBrains）如何为跨平台工具树立标准，以迫使原始公司不至于失去太多市场份额。
