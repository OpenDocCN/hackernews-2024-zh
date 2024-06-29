<!--yml

category: 未分类

date: 2024-05-27 14:40:00

-->

# 使用共享字典来提升压缩效率  |  博客  |  Chrome开发者

> 来源：[https://developer.chrome.com/blog/shared-dictionary-compression](https://developer.chrome.com/blog/shared-dictionary-compression)

数据压缩是一种经过时间检验的性能优化技术，可减少符合条件的页面资源的大小。有一段时间，主要在Web服务器上使用gzip来压缩常见的基于文本的页面资源，如HTML、CSS和JavaScript文件，并将其发送到客户端进行解压缩。结果是资源加载时间更快，而不会影响页面的预期行为。

尽管gzip本身非常有效，但近年来在网络压缩方面已经实现了进一步的改进。2016年，Brotli算法在Chrome中发布，为符合条件的资源提供了更好的总体压缩比。到2017年底，所有现代浏览器都支持Brotli，并且服务器对其支持开始变得更加普遍。最近，Chrome还[推出了ZStandard压缩](https://chromestatus.com/feature/6186023867908096)。

然而，工作并未就此止步！Chrome团队一直在努力使共享字典在网络上可用，现在已经可以在[两者（Brotli和ZStandard）的起源试验](/origintrials#/view_trial/3693514644397228033)中使用。共享字典可以补充Brotli和ZStandard压缩，大大提高常更新代码的网站的压缩比，有时甚至可以达到**90%甚至更高的压缩比**。本文详细介绍了共享字典的工作原理，以及如何注册起源试验以在您的网站上使用它们来进行Brotli和ZStandard的压缩。

## 共享字典解释

压缩是一种在输入中查找冗余序列并利用这些信息创建更小输出的过程，之后可以反向操作。在网络上，压缩效果显著，因为它大幅减少资源加载时间。Brotli和ZStandard通过使用*压缩字典*进一步提升其效率，该字典是这些算法在压缩过程中可以使用的附加模式集合。事实上，Brotli的高效性在一定程度上是通过使用内部字典实现的。

然而，可以使用自定义的用户策划词典与 Brotli 和 ZStandard，其中包含特定于特定资源的模式。在实践中，自定义词典是一个可以应用于任何输入的外部文件。词典可以高度特定于应用程序的生产代码，或者真正任何内容。给定词典对其输入的适用性有可能对整体压缩效率产生重大影响。与输入内容高度相似的词典将产生比通用或不相似内容的词典更高的压缩比率的输出。

这是自定义压缩词典可以如何有效的一个示例：假设您的网站使用 Angular 框架，当前使用的版本是 1.7.9。这个版本的 Angular 框架未压缩时大约为 172 KiB。使用 Brotli 的默认设置进行压缩后，大小变为约 53 KiB。这产生了接近 70% 的压缩比率。然而，假设您稍后决定升级到 Angular 1.8.3 版本。鉴于该 Angular 版本与 1.7.9 版本的大小几乎相同，您可以预期几乎与先前版本相同的压缩比率。

这就是自定义词典可以派上用场的地方，通过使用称为*增量压缩*的过程，这是一种利用资源先前版本的字典来压缩后续版本的方法。使用前述示例，如果使用 1.7.9 版本作为字典来压缩 Angular 1.8.3 版本，输出将略高于 4 KiB。这表示接近**98%的压缩比率**。显然，压缩词典可以对加载性能产生重大影响，并且它们的有效性已经在[现实应用中得到验证](https://github.com/WICG/compression-dictionary-transport/blob/main/examples.md#static-resource-flow-results)！

然而，在使此流程在网络上工作方面存在一个挑战。问题在于，如果您使用字典压缩资源，则需要相同的字典才能进行*解压缩*。此流程此前曾在网络上尝试过，即[SDCH](https://en.wikipedia.org/wiki/SDCH)，但实现安全性是具有挑战性的。这项针对共享字典压缩的最新提议[解决了这些问题](https://github.com/WICG/compression-dictionary-transport?tab=readme-ov-file#this-time-will-be-different)，同时为静态和动态资源提供了显著的好处。

## Chrome 如何声明对共享字典的支持

所有浏览器通过[`Accept-Encoding` 请求头](https://developer.mozilla.org/docs/Web/HTTP/Headers/Accept-Encoding)来声明它们支持的压缩算法。该头部内容是一个以逗号分隔的支持编码列表：

```
Accept-Encoding: gzip, br, zstd 
```

此特定的 `Accept-Encoding` 头部表明，请求资源的浏览器支持 gzip、Brotli 和 ZStandard 压缩算法。响应请求的 Web 服务器随后可以决定在响应请求时使用哪种算法。

当启用共享字典支持并且资源有可用的字典时，将向 `Accept-Encoding` 头部添加额外的令牌。这些令牌是用于 Brotli 的 `br-d` 和用于 Zstandard 的 `zstd-d`。Chrome 还将包含可用字典的哈希值，接下来将进行讨论。

```
Accept-Encoding: gzip, br, zstd, br-d, zstd-d
Available-Dictionary: :pZGm1Av0IEBKARczz7exkNYsZb8LzaMrV7J32a2fFG4=: 
```

如果 Web 服务器配置为识别此令牌并识别字典，则它可以使用适用的编码响应该请求，以压缩资源。在实践中，如何实现这一点取决于请求是针对静态资源还是动态资源。

## 静态资源的共享字典压缩

静态页面资源是指始终为请求的 URL 生成相同响应的资源。常见的可压缩静态页面资源示例包括 JavaScript 和 CSS 文件。这些资源通常会以某种方式进行版本控制，以便进行缓存（有时在文件名中包含文件内容的哈希，例如 `styles.abcd1234.css`），或使用其他指纹技术标识资源。这些资源类型非常适合使用共享字典提供的增量压缩，因为静态资源通常会被缓存很长时间，并且往往会定期更新。

可通过为静态资源设置 `Use-As-Dictionary` 响应头来指定字典。该头部接受几个键/值对之一，但唯一必需的是 `match`，它接受 [`URLPattern` 语法](https://urlpattern.spec.whatwg.org/) 来指定应使用字典的资源路径：

```
Use-As-Dictionary: match="/dist/styles.*.css" 
```

将 `Use-As-Dictionary` 头部视为适用于匹配其内指定的资源未来版本的机制。例如，假设您的网站将所有样式都打包在一个 CSS 文件中。简单起见，假设该资源的第一个版本位于 `/dist/styles.v1.css`，并带有一个 `Use-As-Dictionary` 响应头，其中包含一个 `match` 值为 `/dist/styles.*.css`。

经过一段时间后，您更新了网站的 CSS，并发布了位于 `/dist/styles.v2.css` 的新版本。由于上一个版本的 `Use-As-Dictionary` 响应头中使用的 `match` 值适用于此请求，浏览器将发送一个 `Available-Dictionary` 头部，其中包含字典的哈希值，编码为 [结构化字段字节序列](https://www.rfc-editor.org/rfc/rfc8941.html#name-byte-sequences)。

```
Accept-Encoding: gzip, br, zstd, br-d, zstd-d
Available-Dictionary: :pZGm1Av0IEBKARczz7exkNYsZb8LzaMrV7J32a2fFG4=: 
```

此时，服务器需在其端配置压缩以确保使用匹配的字典。使用该字典压缩的资源将会被发送，而用户浏览器缓存中的可用字典将用于解压。

如果您经常为您的网站发布新代码，增量压缩可以大有裨益。然而，该过程具有灵活性。如果浏览器未确定用户浏览器缓存中是否存在字典，则不会在`Accept-Encoding`头部中指定额外的`br-d`或`zstd-d`令牌。在这种情况下，将应用标准压缩流程。

## 为动态资源应用共享字典压缩

动态资源也可以从共享字典压缩中受益。动态资源是基于上下文变化的资源，例如新闻网站的主页随着新闻的发布频繁更新。HTML 文档通常是动态资源。在这种情况下，字典可以包含站点常见的 HTML 结构和模板代码，导致仅发送每个页面唯一部分的压缩页面。

由于动态生成资源的特性，必须在客户端加载字典以备后用。提前加载字典意味着对动态资源应用共享字典压缩是有一定假设性的。在这种情况下，希望您的网站能够获得足够的流量，以便字典成本可以分摊在大量的导航中。如果您决定尝试，请首先通过页面 HTML 中的`<link>`元素指定字典的位置：

```
<link rel="dictionary" href="/dictionary.dat"> 
```

当 Chrome 遇到这个`<link>`元素时，页面空闲时它*可能*会获取字典，并以低优先级进行，以避免带宽争用。字典本身的响应必须指定一个`Use-As-Dictionary`头部，并指定适用于哪个动态资源路径：

```
Use-As-Dictionary: match="/product/*" 
```

从这里开始，流程基本与静态资源相同。浏览器将看到字典本身适用于匹配资源，并将在请求中附加一个`Available-Dictionary`头部，其中包含字典内容的哈希值，与前面解释的静态资源流程类似。

## 在构建时压缩静态资源

如果您熟悉捆绑器，可能对它们的各种插件熟悉，这些插件可以在构建时压缩资源，并随后提供这些压缩资源。例如，[Apache 允许您使用指令来提供这些预压缩的资源](https://bash-prompt.net/guides/apache-brotli-static/)以响应请求时使用。

大多数基于 Node.js 的捆绑器支持压缩使用 Node 内置的 Zlib 库。Zlib 支持 Brotli，并且使用它的捆绑器通常提供一个接口直接传递选项到 Zlib，该接口[支持字典辅助压缩](https://nodejs.org/api/zlib.html#class-options)。以下是支持使用字典的几个捆绑器：

请注意，任何给定版本资源的可用字典可能使用资源的任何先前版本之一。这意味着您需要分析用户流量并相应地规划。努力实现平衡，并生成最大数量的返回用户受益的资源。CDN提供商目前正在尝试共享字典压缩。尚无公开使用的实现，但我们预计情况将有所改变！

## 尝试一下！

将共享字典压缩与浏览器现有的压缩能力集成，有潜力显著提高经常更新生产代码并且接收大量重复访问的网站的加载性能。如果您有兴趣尝试共享字典压缩，有两种选择：

1.  如果您只是想自己动手尝试共享字典压缩，了解其工作原理，可以在`chrome://flags`页面启用**压缩字典传输**实验性功能。

1.  如果您有兴趣在生产网站上尝试此功能，看看共享字典压缩如何使真实用户受益，请[注册进行起始试验](/origintrials#/view_trial/3693514644397228033)以获取令牌，并详细了解[起始试验的工作原理](/docs/web-platform/origin-trials)。

## 结论

我们对这项网页压缩技术的重大进展感到非常兴奋，以及它可以使人们每天使用的现有应用程序变得更快。我们鼓励您尝试一下，并且最重要的是，[如果您这么做，我们想听听您的想法](https://github.com/WICG/compression-dictionary-transport/issues)！如果您发现了错误，请[在crbug.com上报告](https://issues.chromium.org/issues)。有关更多资源和工具，请访问[use-as-dictionary.com](https://use-as-dictionary.com/)。最后，如果您对其工作原理有更深入的了解兴趣，请查看[解释文档](https://github.com/WICG/compression-dictionary-transport/blob/main/README.md)！
