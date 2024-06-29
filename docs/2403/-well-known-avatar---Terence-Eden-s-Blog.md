<!--yml

category: 未分类

date: 2024-05-27 15:05:07

-->

# .well-known/avatar – Terence Eden的博客

> 来源：[https://shkspr.mobi/blog/2024/03/well-known-avatar/](https://shkspr.mobi/blog/2024/03/well-known-avatar/)

紧随我4年前写的[一篇文章](https://shkspr.mobi/blog/2020/03/one-avatar-to-rule-them-all/)的热潮，有没有一个[user avatar images](https://en.wikipedia.org/wiki/Well-known_URI)的[well-known URI](https://en.wikipedia.org/wiki/Well-known_URI)将会很有用？

当我注册网站服务时，我不希望繁琐地上传一个图像作为我的头像。我希望该服务查看我的电子邮件地址或社交登录，并自动选择我的首选图像。

这是我设想的运行方式。

1.  用户通过电子邮件地址`username@example.com`登录服务。

1.  类似于[WebFinger](https://www.rfc-editor.org/rfc/rfc7033)，服务向以下地址发出请求：

    +   `example.com/.well-known/avatar?resource=acct:username@example.com`

1.  如果请求的`Accept`头部具有`image/*`的MIME类型，则服务器会立即返回一个图像。

1.  如果请求的`Accept`头部具有`application/json`的MIME类型，则服务器可以返回带有[`"rel":"http://webfinger.net/rel/avatar"`](https://webfinger.net/rel/#avatar)的WebFinger风格文档，可能还包括不同图像、格式和大小的列表。

这使得人们在*任何地方*使用相同的头像变得非常简单。

这还意味着，如果你正在设计一个公开显示用户名的服务，你可以在不需要昂贵的API调用的情况下提供头像。例如，Twitter可以在以下位置提供用户的头像：

`twitter.com/.well-known/avatars?resource=acct:edent`

这只是一个想法的草图。在我进一步推进之前，我想知道人们是否认为它有用。

我认为这不会侵犯隐私——用户的图像在所有服务上都是公开的。

用户仍应有权更改其头像选项。

服务不应公开用户的电子邮件地址——它们应该代理图像。

还有其他我应该考虑的事情吗？

为了避免一些常见的观点。

+   这不像[Gravatar](https://shkspr.mobi/blog/2020/03/one-avatar-to-rule-them-all/)那样。它通过成为第三方服务并使用您的电子邮件地址的MD5来运作。

+   这不像[Libravatar](https://www.libravatar.org/)那样。见上文。

+   这不像WebFinger那样。那只返回JSON。

+   这不像h-card那样。那需要服务器解析HTML才能找到图像。

+   这不像[BIMI](https://shkspr.mobi/blog/2022/08/dns-esoterica-bimi-svg-in-dns-txt-wtf/)那样。那是昂贵的，而且只支持SVG。

* * *
