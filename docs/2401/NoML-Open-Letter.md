<!--yml

分类：未分类

日期：2024-05-27 15:08:44

-->

# NoML 公开信

> 来源：[`noml.info/`](https://noml.info/)

# NoML 公开信

为那些希望内容在搜索引擎上可搜索但不用于机器学习的人士的规范。

出版商需要改进的方法来指示他们希望内容在搜索和机器学习中如何使用。 使用 robots.txt 并不能涵盖所有用例，因此需要提出一个补充方法，如本文所提。 这是一种可以根据需要应用于单个网页的方法，并且可以在 Web 内容数据集中保留为此目的而使用。

[通过 Github 签署公开信，](https://github.com/Mojeek/noml-open-letter/issues/new?assignees=PrivacyDingus&labels=&projects=&template=sign.md&title=SIGN%3A+NAME) **或者如果您愿意的话**，您可以通过电子邮件签署该信函。%20if%20relevant.%20If%20you%20are%20signing%20the%20letter%20as%20a%20company%20or%20organisation,%20consider%20attaching%20a%20logo)

## NoML 提案

### 四种情况

在 robots.txt 中阻止用户代理（例如搜索蜘蛛 BingBot）用于 ML/AI 产品也可能阻止与之关联的搜索结果的潜在可见性。 因此，如果您不希望您的内容在利用机器学习的产品中使用，您也可能会失去被列在搜索结果中的好处。 这是一个不可接受的情况，其中内容创建者和出版商可能需要在“全部或无”的选择之间做出选择。 因此，我们需要一种方法，该方法解决了以下四种基本用例：

| **1.** 使用内容进行搜索 使用内容进行机器学习 | **2.** 使用内容进行搜索 **不要** 使用内容进行机器学习 |
| --- | --- |
| **3\. 不要** 使用内容进行搜索 使用内容进行机器学习 | **4\. 不要** 使用内容进行搜索 **不要** 使用内容进行机器学习 |

已经提出了如何解决此问题的几个建议。 在这里，我们提出了一些不同的，更简单和我们认为更公平的东西。 它是对元和 X-Robots 标签的使用的简单调整，我们在下面详细解释。

### 说明和示例

由于许多公司正在爬取/抓取网页以收集数据，而且通常不会识别自己，因此需要一种方法来解决不仅仅是搜索引擎爬虫机器人的问题，并使用 robots.txt。 这里提出的建议是向已经存在的[meta 和 X-Robots 标记](https://www.semrush.com/blog/robots-meta/)添加一个新的“noml”值。 元 robots 标签已经用于搜索引擎爬虫，因此像谷歌这样为他们的搜索引擎爬取的公司已经遵循以这种方式提出的请求。 例如，`noindex` 和 `nofollow` 值用于指示搜索引擎如何处理网页。

同样，元数据存储在 Common Crawl 中，其爬取的数据是机器学习和训练 AI 模型中非常常见且经常是最大的数据来源。 `noindex` 属性用于告诉搜索引擎不要在其搜索结果中包含页面，即使它们可以爬取页面。当网页不打算由搜索引擎索引时，使用此标记。 `nofollow` 属性用于告诉搜索引擎不要跟踪网页上的链接。当包含链接的网页不应被视为支持或推荐时使用此属性。类似地，`noml` 可用于指示任何服务（例如搜索引擎或 AI 构建器）不应将该页面的任何数据用于机器学习。

这可以简单地在 HTML 页面中表示为：

`<meta name="robots" content="noml">`

并且对于非 HTML 使用：

`X-Robots-Tag: noml`

就像对于元标签一样，其中名称“robots”指的是所有用户代理标记，您也可以识别单个用户代理。例如，您可以要求 Google 当前不在搜索结果中包含页面：

`<meta name="googlebot" content="noindex">`

同样，您也可以要求 Google **在其搜索**索引中包含页面，但**不要将数据用于机器学习**：

`<meta name="googlebot" content="noml">`

并要求 Microsoft **不在**其 Bing 搜索索引中包含页面，并**不使用**页面进行训练，使用：

`<meta name="bingbot" content="noindex, noml">`

您也可以要求 OpenAI，例如，不要将页面用于机器学习：

`<meta name="gptbot" content="noml">`

但是在这种情况下，由于 OpenAI 没有运营搜索引擎，您可以（也可以）在 [robots.txt](https://en.wikipedia.org/wiki/Robots.txt) 文件中阻止它们爬行：

`User-agent: gptbot`

`Disallow: /`

下面进一步展示了 HTML 和非 HTML 的示例：

|  | 包含页面在搜索中 | 跟踪链接 | 用于机器学习 |
| --- | --- | --- | --- |

| `<meta name="robots">`

* * *

`X-Robots-Tag:` | ✓ | ✓ | ✓ |

| `<meta name="robots" content="noindex">`

* * *

`X-Robots-Tag: noindex` | ✕ | ✓ | ✓ |

| `<meta name="robots" content="nofollow">`

* * *

`X-Robots-Tag: nofollow` | ✓ | ✕ | ✓ |

| `<meta name="robots" content="noml">`

* * *

`X-Robots-Tag: noml` | ✓ | ✓ | ✕ |

| `<meta name="robots" content="noindex, nofollow">`

* * *

`X-Robots-Tag: noindex, nofollow` | ✕ | ✕ | ✓ |

| `<meta name="robots" content="noindex, nofollow, noml">`

* * *

`X-Robots-Tag: noindex, nofollow, noml` | ✕ | ✕ | ✕ |

### 搜索引擎 API 使用

基于爬虫的搜索引擎，如 Mojeek、Bing 和 Google，还通过 API 向其他搜索引擎和服务提供其结果。我们建议这些搜索引擎（就像 Mojeek 一样）在其 API 响应中包含“noml”指令。搜索 API 提供商应该将其 API 服务条款的一部分设定为，由 API 最终用户标记为此类结果不得用于机器学习目的。

### 结论

此提议的解决方案通过简单地向已有方法中添加一个附加值来实现期望的目标，而不一定需要使用更多的用户代理。

使用这个提案，创作者和发布者可以单独指示他们是否希望内容在搜索引擎中可找到和/或用于机器学习。

### 分享这封公开信

## 亲爱的人工智能公司、网络爬虫、搜索引擎、机器学习项目、网络爬取工具等。

我们，签署者，支持采纳此提案，该提案使网络上的创作者、发布者和所有其他内容贡献者能够指示他们的内容是否可用于机器学习训练和应用。

### 组织和公司的签名

### 个人的签名
