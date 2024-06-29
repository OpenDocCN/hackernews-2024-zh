<!--yml

category: 未分类

date: 2024-05-29 12:36:56

-->

# Underjord | 对象存储基础

> 来源：[https://underjord.io/fundamentals-of-object-storage.html](https://underjord.io/fundamentals-of-object-storage.html)

# 对象存储基础

2024-03-19

Underjord是一个[小而友好的团队](/team.html)，专注于Elixir咨询和合同工作。如果你喜欢这篇文章，你真的应该试试这段代码。查看我们的[服务](/services.html)获取更多信息。

我做了一个关于对象存储的直播，讨论了如何以及为什么。那些糟糕的旧日子。以及超越基础知识的一些有趣的东西。我想在文本中也覆盖这些内容。

你可以[在这里观看直播](https://youtube.com/live/Sg0Afr_4BMY)。

*透明通知：直播和本博文由[Tigris](https://www.tigrisdata.com/)支持。我们在这些主题上进行合作，他们提供资源并支付部分时间供我从事这些事情，而不是为客户编写软件。我非常高兴和感激有人资助我出版和创意想法。预计近期会有更多输出，请给他们一个机会！*

## 背景

当我开始架构系统时，我们没有对象除了面向对象编程语言外，也没有多少存储。如果我们有存储，要么在同一台物理服务器上，要么是某种基于NFS的可怕东西。

想象一下你建立了你的n层架构。应用程序在数据库前水平扩展。你可以使用一些当时很酷的Redis或Memcache来缓存重负载。对于文件存储，我们必须设置某种共享存储。它们真是个棘手的家伙。

挂载点可能会松动，文件会写入本地文件夹。性能会随机下降。我记得为我的服务器安装了一个文件系统缓存工具，而使用NFS时，它突然会在内核级别硬锁系统。因为NFS生活或至少在那时是在内核中的。

当它工作良好时，让网络文件系统假装是真正的文件系统可以非常实用。但这只是你对自己和正在构建的系统说的一个小小的谎言。如果你倾向于NFS生活方式中类似文件的性质......嗯，那是有风险的。你可能会留下伤疤。

我一开始发现AWS很烦人。不可靠的VPS，你不能把文件放在上面。真是麻烦。但是在我深入了解对象存储（如今所有的对象存储基本上都兼容S3）之前，过了一段时间。EC2实例的愚蠢强化了良好的可伸缩性实践，而S3的简单性使其效果极佳。

人们从网络文件存储中希望得到的通常是可靠性和空间。数据保留在您放置的位置，不会丢失。并且您的数据必须适合。随着数据的增长，它仍然必须适合。因此，本质上是无限的未知无界存储增长。理想情况下，您不希望支付超过当前所需磁盘利用率的费用。

简单、节制和约束对于启动大多数事物都是有益的。但特别是对于实现雄心勃勃的事物，如无限*文件存储。

* 实际上并非无限，但对于大多数情况来说足够接近

对象存储显然更清晰地是一项服务。它不是一个文件系统。当您深入了解时，名称会有所澄清。它也不一定是关于“文件”的。它可能是有史以来最成功的 NoSQL 数据库。键、值，其他东西并不那么重要。

基本操作如下：

+   放置对象

+   获取对象

+   对象列表

+   删除对象

完整列表可以参见 [更长更详细的文档](https://docs.aws.amazon.com/AmazonS3/latest/API/API_Operations_Amazon_Simple_Storage_Service.html)。

## 使用 Elixir

为了展示如何在 Elixir 中使用对象存储的基础知识，我设置了一个混合项目：`mix new bla`。然后我在 `mix.exs` 中添加了以下依赖项：

elixir mix.exs

```
 defp deps do
    [
      {:ex_aws, "~> 2.5"},
      {:ex_aws_s3, "~> 2.5"},
      {:hackney, "~> 1.9"},
      {:sweet_xml, "~> 0.7"},
      {:jason, "~> 1.4"}
    ]
  end
```

然后，我创建了一个 `config/config.exs` 文件，内容如下：

elixir config/config.exs

```
import Config

config :ex_aws, :s3,
  scheme: "https://",
  host: "fly.storage.tigris.dev",
  port: 443
```

要获取您的依赖项：`mix deps.get`

要创建一个存储桶，如果您已经准备好使用 `fly` 命令行工具，那么非常简单，只需执行 `fly tigris create`。直到您**真正**使用它时是免费的，5Gb 和 10k+ 请求/月。当然，您可以选择任何存储桶。但它将提供凭据。

如果您将它们设置为环境变量，`:ex_aws_s3` 将自动读取您的凭据，但不包括存储桶名称。如果您需要多个存储桶，您可以显式地处理它们。这些示例假设使用一个凭据集。

我像这样开始我称之为 `Tigris` 的模块：

elixir lib/tigris.ex

```
defmodule Tigris do
  alias ExAws.S3

  defp bucket!, do: System.fetch_env!("BUCKET_NAME")
end
```

然后我着手列出对象：

elixir lib/tigris.ex

```
#..
  def list!(prefix \\ "") do
    bucket!()
    |> S3.list_objects(prefix: prefix)
    |> ExAws.request!()
    |> then(fn %{body: %{contents: contents}} ->
      contents
    end)
  end
#..
```

您可以在`iex -S mix`中尝试各种操作，以便在提示符中编译和准备好此项目中的所有内容：

elixir iex

```
iex(1)> Tigris.list!()
[]
```

接下来是一些放置和获取操作。我不建议您像这样处理错误。这是为了简单和流而不是最佳实践，绝对不是金融或法律建议。

elixir lib/tigris.ex

```
# ..
  def put!(key, data) do
    bucket!()
    |> S3.put_object(key, data)
    |> ExAws.request!()

    :ok
  end

  def get(key) do
    result =
      bucket!()
      |> S3.get_object(key)
      |> ExAws.request()

    case result do
      {:ok, %{body: body}} -> body
      {:error, {:http_error, 404, _}} -> nil
      {:error, error} -> {:error, error}
    end
  end
# ..
```

我也在`iex`中尝试了这些操作，以展示基础知识。然后，我想提示可能的规模。`Task.async_stream/2` 根据您的机器确保运行足够多的并行任务。

elixir lib/tigris.ex

```
# ..
  def put_tons!(kv) do
    kv
    |> Task.async_stream(fn {key, value} ->
      IO.puts(key)
      put!(key, value)
    end)
    |> Stream.run()
  end
# ..
```

然后：

elixir iex

```
iex(1)> kv = for d <- 1..100, f <- 1..100, into: %{}, do: {"dir-#{d}/file-#{f}.txt", "#{d}  #{f}"}
# snip
iex(2)> Tigris.put_tons!(kv)
```

然后进行整理。不要在您喜欢或需要的数据存储桶上运行此操作：

elixir lib/tigris.ex

```
# ..
  def exterminate! do
    stream =
      bucket!()
      |> S3.list_objects()
      |> ExAws.stream!()
      |> Stream.map(& &1.key)

    S3.delete_all_objects(bucket!(), stream) |> ExAws.request()

    :exterminated
  end
# ..
```

## 预签名 URL

不，我们有点花哨了。因此，对象存储的一个巨大好处是，它可以具有功能，可以使您的应用服务器免于一堆可怕的工作。您是否曾让您的应用程序接收文件上传，然后将其保存到其他地方？荒谬！我们有一个专门的服务！有协议！

elixir lib/tigris.ex

```
# ..
  def presign_get(key) do
    :s3
    |> ExAws.Config.new([])
    |> S3.presigned_url(:get, bucket!(), key, [])
  end
# ..
```

利用它。获取一个预签名 URL（您可以配置过期时间等），用于下载或上传，然后将其交给客户端，让他们自行处理上传，而无需您介入。只需他们和您的对象存储之间。顺便说一下，使用 Tigris 这样做的有趣之处。Tigris 将数据放置在全球靠近上传者的位置。如果需要，它会作为缓存复制到其他地区，但这意味着您在澳大利亚的客户会获得本地延迟。

对上传和下载一样好用。您的应用服务器最差的下载方式是：

+   客户端请求您的应用服务器获取文件。

+   您的应用程序检查请求是否合理，并应该签名。

+   预签名 URL。

+   客户端接收到一个带有重定向到预签名 URL 的响应。

+   透明下载，但您的应用程序不提供字节。

## 多部分上传（流式上传！）

根据 AWS，单个上传可以达到 5Gb。但通常会变得难以管理。我们可以将上传分块为 5Mb 的块。

elixir lib/tigris.ex

```
# ..
  def put_file!(key, from_filepath) do
    from_filepath
    |> S3.Upload.stream_file()
    |> Stream.map(fn chunk ->
      IO.puts("uploading...")
      chunk
    end)
    |> S3.upload(bucket!(), key)
    |> ExAws.request!()
  end
# ..
```

这是一种流式上传方式。这意味着如果您需要进行处理，然后希望立即卸载结果，您可以通过流式上传而不是保留整个巨大的东西来将内存使用量减少到约 5Mb（这取决于情况）。这具体构建了一个从文件上传的流，但使用 Elixir Stream 和 IO 工具可以进行许多变体。

## 范围请求（流式下载！）

有时我们只想要文件的几个部分。有时我们想要构建一个只读 SQLite VFS 的地狱兽，覆盖 S3 API。有时我们想要进行优雅的流式下载。为此，我们有直接的 HTTP 范围请求。

elixir lib/tigris.ex

```
# ..
  def get(key, range \\ nil) do
    opts =
      if range do
        [range: "bytes=#{range}"]
      else
        []
      end

    result =
      bucket!()
      |> S3.get_object(key, opts)
      |> ExAws.request()

    case result do
      {:ok, %{body: body}} -> body
      {:error, {:http_error, 404, _}} -> nil
      {:error, error} -> {:error, error}
    end
  end
# ..
```

这非常有用且强大。当然。您可能大部分时间都在处理文件。直到您不需要了。或者直到您意识到有多少文件格式只需使用正确的字节即可公开有用信息。

使用 [Packmatic](https://github.com/evadne/packmatic) 可以流式传输 Zip 归档文件，因为它允许您在归档末尾保持文件索引。那是一个已知的位置。我们可以搜索文件的最后一部分，并仅获取索引，这意味着文件清单。这意味着我们可以挑选出文件。可以对 MP3 的 ID3 标签执行类似操作，并且仅获取元数据而不必获取整个文件。

如果您使用 Elixir 库 [Explorer](https://hex.pm/packages/explorer)，底层的 Rust 库支持 S3 API，将让您从远程存储的 .parquet 文件等中惰性读取数据。同样的事情。

我希望这解释清楚了为什么范围请求对于文件存储服务非常有用。

## S3 API为何成为标准

到目前为止，我使用过的所有对象存储服务都暴露了一个兼容S3的API。这是有道理的。它帮助您支持预期的功能集，并且您可以免费获得兼容您产品的SDK和数以万计的客户端。

S3 API真的那么好吗？

我不认为是这样的。我在这里整体上谈论的是，它只是足够好，而可靠的“无限”存储价值主张和通用协议使其值得。显然是这样。在与Tigris的CEO Ovais Tariq讨论这篇文章时，他实际上分享了一个更有趣的观点。与我持平淡态度相反。对于Tigris来说，兼容S3 API使切换和开始变得简单是有意义的。然而，S3 API实际上是一个重要的约束。Tigris拥有与S3不同的元数据系统和基础设施。有改进、创新和一些真正优秀的功能，这些功能在相对滞后的S3 API中难以实现得体。当然，您可以在标头和其他聪明的地方弄出一些功能，但这也有其不足之处，因为您在客户端实现时可能会错过任何新的进展。根据我与Ovais及其团队的交流，我们仍将看到他们在对象存储方面的创新。我猜我们将继续关注S3 API是否能应对这一挑战。

* * *

如果您想分享一些您看到的更有趣的对象存储用法，或者只是想告诉我您是否觉得这篇文章有帮助，请随时联系我。我在Fediverse上的用户名是[@lawik](https://twitter.com/lawik)，您也可以通过电子邮件联系我，我的邮箱是[lars@underjord.io](mailto:lars@underjord.io)。感谢阅读。

Underjord 是一个[4人团队](/team.html)，专门从事Elixir咨询和合同工作。如果您喜欢这篇文章，确实应该尝试一下代码。查看我们的[服务](/services.html)以获取更多信息。

注意：或者尝试一下[YouTube频道上的视频](https://youtube.com/c/underjord)。
