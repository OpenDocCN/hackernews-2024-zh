<!--yml

category: 未分类

date: 2024-05-27 13:17:23

-->

# 缓存HTTP前辈的秘密，第1部分

> 来源：[https://csvbase.com/blog/8](https://csvbase.com/blog/8)

在csvbase中，表格大小没有硬性限制。在合理范围内，可以任意大。如果你自己运行实例，可以做任何你想做的事情。大数据并不总是更好的数据，但有这个选择是很好的。

话虽如此，如果那些表格可以被缓存，那不是挺好的吗？这样，如果你第二次阅读，就不必再完全下载它了。

## Cache-control头部 - 带有max-age

或许是这个HTTP服务器头？：

```
Cache-Control: max-age=86400 
```

上述标题说“在重新下载之前继续使用这个文件一天[86400秒]”。太棒了！

等一下。如果有人更改了那个表格，你可能会被困在旧版本上最多24小时。[最终一致性](https://doi.org/10.1145%2F1435417.1435432)曾经非常时髦。但现在不再流行 - 即使是Amazon S3 [也不再返回过时数据](https://aws.amazon.com/blogs/aws/amazon-s3-update-strong-read-after-write-consistency/)。因此，对于csvbase来说，提供过时数据是不行的。

## HTTP/1.1有许多缓存功能

前辈们深思熟虑地关于缓存的想法可以参考[这篇文章](https://doi.org/10.1109/ICDE.2000.839382)。DNS [是一个缓存](https://www.internethalloffame.org/2012/07/23/why-does-net-still-work-christmas-paul-mockapetris/)。[虚拟内存](https://www.linuxatemyram.com/)也是一个缓存。如果你曾经想知道为什么HTTP/1.0发布不到一年后就发布了补丁版本(HTTP/1.1)：更多的缓存。

HTTP/1.0在缓存方面几乎什么也没有。在HTTP/1.1中，标准中添加了一个全新的顶级部分来讨论这个主题。这是必不可少的，因为在过去，重复重新下载未更改的文件是一个真正的头疼。

对于流行网站来说，这种不必要的重新下载尤其糟糕，它们的页面会被许多人反复地重新下载，即使它们没有变化。足以开始为整个互联网带来问题。

HTTP/1.1中的*酷新特性*之一（在Netscape Navigator 4.0中实现，在所有信誉良好的软件经销商处可获得）是`ETag`头。它的工作方式如下：

HTTP服务器为每个文件版本生成唯一的ID。在这里，“版本”不仅指修订版，还包括格式，因此即使从同一URL提供，CSV文件与Parquet文件也会有不同的ID（csvbase [做到了这一点](https://csvbase.com/blog/2)）。HTTP服务器在`ETag`头中返回此ID。

当HTTP客户端再次请求文件时，它在`If-None-Match`头中提供相同的ID。

如果该ETag是最新的，服务器只需将状态设置为`304 Not Modified`并且不返回任何内容。客户端随后知道它应该使用已有的文件。

这是一种既保持缓存又避免过时数据的方式。

## 生成ETag

ETag 应该如何生成？这取决于服务器 - 取决于csvbase。 csvbase可以只是对文件进行md5sum，并将其放入标头。那将是有效的，它将起作用。

好吧，除了。 除此之外，csvbase实际上并不是CSV文件的数据库。 名称有点把戏。 csvbase并不在某些文件系统（或[对象存储](https://calpaterson.com/s3.html)中）保留CSV文件。 相反，上传的CSV文件被完全解析，然后放入SQL数据库中。 当csvbase返回CSV文件时，它是*即时生成*的。 这就是csvbase返回Parquet、JSONlines、JSON等其他格式的方式。

对于csvbase来说，不必每次都重新生成CSV文件（或Parquet文件）会更快，只需检查该文件的md5sum是否与 `If-None-Match` 标头匹配。

因此，我们将不是通过散列文件内容来生成ETag，而是通过散列密钥来创建ETag：

`(表ID，上次更改时间，文件格式，页面编号[如果有的话])`

该键表示文件的版本以及正在查看的页面（JSON和HTML格式是分页的）。 如果您的表格更改了，上次更改时间是不同的，ETag 也会改变。 真棒。

## 缓存多长时间？

此时，我们有了一个缓存系统，它：

1.  免去了客户端下载文件的时间

    +   虽然他们必须提出一个微小的请求

1.  还避免了服务器重新生成文件

1.  防止任何过时结果（即：避免["TTL hell"](https://calpaterson.com/ttl-hell.html)）

现在唯一的问题是告诉客户端缓存多长时间。 多久？

答案是永远。 csvbase 不设置 `max-age`。 客户端可以根据需要缓存数据，**只要在使用数据时每次重新验证**。

这是csvbase的响应的样子

```
HTTP/1.1 content-type: text/csv etag: <some gobbledegook string> cache-control: no-cache, must-revalidate [ then loads of CSV data ] 
```

这里的关键是 `no-cache` pragma。 尽管名字有误导性，但这告诉客户端他们实际上可以缓存此文件，*但是*他们必须每次使用时重新验证。 这是[MDN](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Cache-Control#response_directives)如何描述它的（重点是我的）：

> *"no-cache 响应指令表示可以将响应存储在缓存中，但必须在每次重用之前与源服务器验证响应，即使缓存与源服务器断开连接也是如此。"*
> 
> *"如果您希望缓存始终在重复使用存储的内容时检查内容更新，no-cache 是要使用的指令。 通过要求缓存每个请求与源服务器重新验证。"*

`must-revalidate` 可能是不必要的，但原始RFC建议在特殊情况下，缓存可能会选择仍然提供过时数据。 `must-revalidate` 禁止这样做。 `must-revalidate` 的名称实际上是有意义的，所以我可能不需要引用章节和诗歌，但在这里，再次来自MDN：

> *HTTP允许缓存在与原始服务器断开连接时重用陈旧的响应。must-revalidate是一种防止这种情况发生的方法——要么重新验证存储的响应，要么生成504（网关超时）响应。*

不需要TTL。而且浏览器真的支持这些东西。这一切都有效。

[`csvbase-client`](https://pypi.org/project/csvbase-client/)也支持这一点。您可以用这种方式测试它：

```
import pandas as pd

# read the table
df1 = pd.read_csv("csvbase://calpaterson/opcodes-6502")

# read it again (this came from your local cache)
df2 = pd.read_csv("csvbase://calpaterson/opcodes-6502") 
```

如果您更喜欢只使用`curl`，这里是如何测试它的方法：

```
curl \
--etag-compare opcodes-6502-etag.txt \
--etag-save opcodes-6502-etag.txt \
https://csvbase.com/calpaterson/opcodes-6502.parquet
\-O 
```

## ETags还可以实现哪些其他可能性？

还有更多内容。下次我将使用ETags作为一种有效的并发控制形式。在那之前，请保持关注：

## 另请参阅

我几年前写了[另一篇关于缓存的文章](https://calpaterson.com/ttl-hell.html)。主题有所不同，但目的是相同的：缓存但避免正确性问题。

我喜欢埃里克·布鲁尔（Eric Brewer）2004年关于Inktomi的演讲。[Inktomi](https://en.wikipedia.org/wiki/Inktomi)实际上是一个原始的Google，试图采用白标方式而非自己运行搜索引擎。后来他们转变为内容分发网络，并在[大约57分钟处](https://youtu.be/E91oEn1bnXM?si=zt5vj1TPSBOMkKzG&t=3434)布鲁尔讲述了发布Starr报告的故事，这份报告证明了自HTTP/1.1发布以来对Web缓存投入的很多努力。他在整个演讲中情绪非常激动，因为Inktomi在几年前已经彻底垮掉。

我没有讨论的另一个问题是，许多缓存反向代理并没有非常好的缓存重新验证支持。有关Varnish的通用建议是使用一些[相当复杂的配置](https://developers.cloudflare.com/cache/concepts/default-cache-behavior/)。Cloudflare在处理`no-cache`时[做错了事情](https://developers.cloudflare.com/cache/concepts/cache-control/#origin-cache-control-behavior)，但可以通过解决方案来解决。对于csvbase而言，无法解决的是他们对`Vary`的[不支持](https://simonwillison.net/2023/Nov/20/cloudflare-does-not-consider-vary-values-in-caching-decisions/)。这些都是1997年的HTTP/1.1内容，而浏览器几乎全部支持。但是出于某种原因，反向代理并没有支持这些。
