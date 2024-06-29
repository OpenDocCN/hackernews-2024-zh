<!--yml

category: 未分类

date: 2024-05-27 14:41:50

-->

# Microsoft 正在将 Linux 的 sudo 命令引入 Windows 11 - The Verge

> 来源：[https://www.theverge.com/2024/2/8/24066264/mirosoft-sudo-command-windows-11-feature](https://www.theverge.com/2024/2/8/24066264/mirosoft-sudo-command-windows-11-feature)

Windows 11 将很快引入一个专为开发者设计的内置 sudo 命令。Sudo，即“超级用户执行”，在类似 Linux 和 macOS 这样的基于 Unix 的操作系统上广泛使用，用于以更高的安全权限或作为其他用户运行程序。例如，对于希望测试脚本的开发者来说非常有用。

Microsoft 在 Windows 中使用 sudo 允许开发者直接从非提升的控制台会话中运行提升的工具。“对于希望在不必首先打开新的提升控制台的情况下提升命令的用户来说，这是一种人性化和熟悉的解决方案，” [Jordi Adoumie 解释道](https://devblogs.microsoft.com/commandline/introducing-sudo-for-windows/)，他是 Microsoft 的产品经理。

*Sudo 可在未来版本的 Windows 11 开发者设置中进行控制。*

Image: Microsoft

Sudo 正在作为 Windows 11 的最新 [Canary 构建](https://blogs.windows.com/windows-insider/2024/02/08/announcing-windows-11-insider-preview-build-26052-canary-and-dev-channels/) 的一部分进行测试，因此在今年晚些时候才会在常规版本的 Windows 11 中可用。Microsoft 将允许配置 sudo 命令为三种模式：新窗口、禁用输入和内联。与 Linux 的 sudo 最相似的模式是内联，而其他模式则更加安全。

“在接下来的几个月里，我们将继续扩展 Sudo for Windows 的文档，并分享更多关于在‘内联’配置下运行 sudo 的安全性影响的详细信息，” Adoumie 表示。

Microsoft 也在 [GitHub 上开源了这个 sudo 项目](https://github.com/microsoft/sudo)，并计划在未来几个月内更多地分享 sudo 的计划。

将 sudo 添加到 Windows 是在 Microsoft 全面支持 Linux 并在 Windows 10 中发布 [完整的 Linux 内核](/2019/5/6/18534687/microsoft-windows-10-linux-kernel-feature) 之后。软件制造商还将 [Bash shell 添加到 Windows](/2016/3/30/11331014/microsoft-windows-linux-ubuntu-bash)，在 Windows 10 中支持 [原生 OpenSSH](/2017/12/14/16775764/microsoft-windows-10-openssh-client-support)，甚至将 Ubuntu、SUSE Linux 和 Fedora 带入了 Windows Store。
