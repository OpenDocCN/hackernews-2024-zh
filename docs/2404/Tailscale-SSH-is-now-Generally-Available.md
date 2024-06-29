<!--yml

类别：未分类

日期：2024-05-27 13:17:19

-->

# Tailscale SSH 现已普遍提供

> 来源：[https://tailscale.com/blog/tailscale-ssh-ga](https://tailscale.com/blog/tailscale-ssh-ga)

我们很高兴地宣布[Tailscale SSH](https://tailscale.com/ssh)现已普遍提供。Tailscale SSH 允许 Tailscale 管理 SSH 连接的认证和授权。从用户的角度来看，您可以像平常一样使用 SSH — 根据可配置的规则使用 Tailscale 进行认证 — 我们处理单点登录、多因素认证和密钥轮换，并允许您在 ACL 中执行精确的权限控制。结合 SCIM 进行用户和组配置等其他企业功能，您可以使用 Tailscale SSH 打造完全的零信任远程访问解决方案。

感谢其灵活性和强大的默认安全性，Tailscale SSH 已经成为许多用户工作流程的关键组成部分，包括作为企业 ZTNA 策略的基础部分。对于没有额外硬件或防火墙规则需要管理的情况来说，它基本上可以视为普通（有时候略显麻烦的）SSH 进程的升级替代品。

我们对 Tailscale SSH 工作原理有了更多的概念背景，并且知道了发布它的动机，详见[原始发布文章](https://tailscale.com/blog/tailscale-ssh)。您可能还会对我们最近发布的作为 Tailscale 解释视频系列一部分的教程视频感兴趣：

我们从 Tailscale SSH 的 Beta 期间学到了很多，并随之构建了相关功能，因此现在它已经进入一个比 2022 年首次推出时更加强大的工具生态系统。一些亮点包括：

+   我们发布了 Tailscale SSH 控制台，允许指定用户在其 tailnet 上的节点上创建基于浏览器的 SSH 会话

+   我们增加了对远程端口转发和 SELinux 的支持

+   我们推出了[Tailscale SSH 会话录制](https://tailscale.com/kb/1246/tailscale-ssh-session-recording)，使团队能够满足合规性要求，并在不需要信任第三方处理其会话数据（包括我们自己）的情况下监控其 tailnet 上的 SSH 会话。

+   我们开发了我们的[VS Code 扩展](https://marketplace.visualstudio.com/items?itemName=tailscale.vscode-tailscale)，它允许您在整个 tailnet 的节点上编辑远程文件，基于 Tailscale SSH 的基础上。

如果您已经在使用 SSH 进行远程访问，但尚未开始使用 Tailscale SSH，[现在正是时候](https://tailscale.com/kb/1193/tailscale-ssh)。它今天已经在我们的个人、高级和企业计划中提供。
