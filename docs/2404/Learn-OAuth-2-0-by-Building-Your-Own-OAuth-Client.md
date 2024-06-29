<!--yml

category: 未分类

date: 2024-05-27 13:17:34

-->

# 通过构建自己的OAuth客户端学习OAuth 2.0

> 来源：[https://annotate.dev/p/hello-world/learn-oauth-2-0-by-building-your-own-oauth-client-U2HaZNtvQojn4F](https://annotate.dev/p/hello-world/learn-oauth-2-0-by-building-your-own-oauth-client-U2HaZNtvQojn4F)

您已经使用授权码授予类型构建了一个OAuth客户端。但是，是否应该从零开始开发自己的OAuth客户端呢？一般来说，不应该。微妙的错误或新颖的漏洞可能会引入严重的安全风险。相反，选择经过充分测试的开源OAuth客户端是一个更安全的选择。

理解OAuth客户端的内部工作机制非常重要。它使您了解开源库中可用的配置选项，帮助解决与OAuth相关的问题，并使学习其他OAuth流程（如刷新、PKCE和隐式流）变得轻松。您还可以更好地了解OpenID Connect（OIDC）；这是一个建立在OAuth之上的协议。

在接下来的演示中，您将探索OIDC——它解决了哪些问题，以及支持它所需的最小代码调整。敬请关注！
