<!--yml

类别：未分类

日期：2024-05-27 13:41:54

-->

# Solene的OpenBSD脚本，用于转换wg-quick VPN文件

> 来源：[https://dataswamp.org/~solene/2024-04-27-openbsd-wg-quick-converter.html](https://dataswamp.org/~solene/2024-04-27-openbsd-wg-quick-converter.html)

# 1\. 引言 [§](#_Introduction)

如果您使用商业VPN，您可能已经注意到它们都提供wg-quick格式的WireGuard配置，这对于在OpenBSD中方便使用并不合适。

由于我目前在VPN提供商工作很多，我经常需要处理配置，我真的需要一个脚本来简化我的工作。

我做了一个Shell脚本，将wg-quick配置转换为hostname.if兼容文件，以便完全集成到OpenBSD中。如果您希望始终连接到给定的VPN服务器，而不是临时连接，这非常实用。

[OpenBSD手册页面：hostname.if](https://man.openbsd.org/hostname.if)

[Sourcehut 项目：wg-quick-to-hostname-if](https://git.sr.ht/~solene/wg-quick-to-hostname-if)

# 2\. 用法 [§](#_Usage)

这非常容易使用，下载脚本并标记为可执行，然后使用您的wg-quick配置作为参数运行它，它将在标准输出中输出hostname.if文件。

```
wg-quick-to-hostname-if fr-wg-001.conf | doas tee /etc/hostname.wg0 
```

在生成的文件中，它使用了一个技巧来动态确定当前默认路由，这对保持与VPN网关的非VPN路由是必需的。

# 3\. 短暂的VPN会话 [§](#_Short_VPN_sessions)

当我在mastodon上分享我的脚本时，Carlos Johnson 分享了他们自己的脚本，非常酷，与我的脚本互补。

如果您更喜欢为有限的会话建立VPN，您可能需要查看他的脚本。

[Carlos Johnson GitHub：file-wg-sh gist](https://gist.github.com/callemo/aea83a8d0e1e09bb0d94ab85dc809675#file-wg-sh)

# 4\. 防止泄漏 [§](#_Prevent_leaks)

如果您希望您的WireGuard VPN是防泄漏的（即如果不是流向VPN网关的网络流量不应离开网络接口），您绝对应该执行以下操作：

+   您的WireGuard VPN应该在rdomain 0上

+   WireGuard VPN应该在另一个rdomain上建立

+   使用PF阻止流向VPN网关之外的其他rdomain上的流量

+   使用VPN提供商的DNS或无记录公共DNS提供商

[更早的博客文章：WireGuard和rdomain](https://dataswamp.org/~solene/2021-10-09-openbsd-wireguard-exit.html)

# 5\. 结论 [§](#_Conclusion)

OpenBSD 使用 ifconfig 配置 WireGuard VPN 的能力一直是一个令人难以置信的功能，但从 wg-quick 文件转换并不总是有趣。但现在，使用商业VPN要容易得多，多亏了一些Shell脚本。
