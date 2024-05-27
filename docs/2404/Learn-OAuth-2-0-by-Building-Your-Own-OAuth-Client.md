<!--yml
category: 未分类
date: 2024-05-27 13:17:34
-->

# Learn OAuth 2.0 by Building Your Own OAuth Client

> 来源：[https://annotate.dev/p/hello-world/learn-oauth-2-0-by-building-your-own-oauth-client-U2HaZNtvQojn4F](https://annotate.dev/p/hello-world/learn-oauth-2-0-by-building-your-own-oauth-client-U2HaZNtvQojn4F)

You've built an OAuth client using the authorization code grant type. However, should you develop your own OAuth client from scratch? Generally, no. Subtle bugs or novel exploits can introduce serious security risks. Instead, opting for well-tested, open-source OAuth clients is a safer choice.

Understanding the inner workings of an OAuth client is important. It gives you knowledge about the configuration options available in open-source libraries, helps in troubleshooting OAuth-related issues, and makes learning other OAuth flows like refresh, PKCE, and implicit a breeze. You're also in a great position to learn about OpenID Connect (OIDC); a protocol built atop of OAuth.

In the next walkthrough, you'll explore OIDC—what problems it addresses and the minimal code adjustments needed to support it. Stay tuned!