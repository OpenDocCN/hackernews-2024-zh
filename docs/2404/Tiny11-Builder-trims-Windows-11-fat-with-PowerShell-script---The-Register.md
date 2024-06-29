<!--yml

category: 未分类

date: 2024-05-27 13:26:47

-->

# Tiny11 Builder 使用 PowerShell 脚本瘦身 Windows 11 • The Register

> 来源：[https://www.theregister.com/2024/04/22/tiny11_builder_update/](https://www.theregister.com/2024/04/22/tiny11_builder_update/)

如果你担心 Windows 11 的臃肿，并希望对其 ISO 文件有更多控制？在周末，Tiny11 Builder 的新版本以 PowerShell 的形式现身了。

Tiny11 是微软旗舰操作系统的简化版本。[我们去年看过它](https://www.theregister.com/2023/11/27/tiny11_2311/)，虽然技术上是一次成功的尝试，但是对设置媒体中包含或不包含的内容有更多控制将是受欢迎的。

最新版本的 Tiny11 Builder 就是一个 PowerShell 脚本，用于处理现有的 Windows 11 ISO 镜像，并生成一个精简版，去掉了像纸牌、闹钟、天气和 Edge 等功能。

软件的作者在周末上传了一个脚本的 [视频](https://youtu.be/SFaIq8-vRTU?si=kvxbFrFbyDcluGTf) 并在 [GitHub](https://github.com/ntdevlabs/tiny11builder) 仓库中分享了这个项目，称其为“更全面解决方案的一个跳板”。

一个用户需要相当熟悉 PowerShell 才能运行脚本，尽管如果你是喜欢在 Windows 上捣鼓的那种人，那么在命令行中输入命令可能不会造成太大问题。

不过，我们也必须对此提出一个相当大的健康警告。是的，这是一个有趣的工具，剔除当今与 Windows 11 一同发货的那些臃肿软件确实让人愉悦，但是生成的 ISO 文件并未得到微软官方认可，也不会得到支持。而且，不能保证生成的内容不会带来其自身的安全漏洞。

Tiny 11 Builder 的作者表示：“我的主要目标是只使用像 DISM 这样的微软工具，而不使用外部工具。” 还有一个无人值守答案文件，可用于在首次启动体验期间绕过微软账户提示。

即使考虑到健康警告，这个脚本本身似乎相对无害，只是简单地去除了微软过度的部分。据作者称：“唯一包含的可执行文件是 oscdimg.exe，它是 Windows ADK 中提供的用于创建可启动 ISO 镜像的工具。”

这是一段令人印象深刻的脚本，尽管目标对象除了爱好者之外并不明确。精简的镜像确实需要更少的磁盘空间，但是 Windows 11 的许多其他问题，如其硬件兼容性要求，仍然存在。由于这只是一个定制的 ISO 安装，你还需要一个激活密钥。®
