<!--yml

分类：未分类

日期：2024年5月27日 14:48:54

-->

# 在 Fly.io 上从 S3 切换到 Tigris

> 来源：[https://benhoyt.com/writings/flyio-and-tigris/](https://benhoyt.com/writings/flyio-and-tigris/)

# 在 Fly.io 上从 S3 切换到 Tigris

2024年2月

> **前往：** [存储桶](#creating-a-bucket) | [端点](#endpoint-change) | [缓存](#caching-differences) | [认证](#authentication-differences) | [复制](#copying-files-and-shadow-buckets) | [数据库迁移](#database-migration) | [最终想法](#final-thoughts)

一年前，我将我的 [GiftyWeddings.com](https://giftyweddings.com/) 边缘项目从托管在 Amazon EC2 切换到使用 [Fly.io](https://fly.io/)。这是一次 [有趣的体验](/writings/flyio/)，并且每月省下了几美元。

上周，有人从 [Tigris](https://www.tigrisdata.com/) 联系我，问我是否想尝试他们在 Fly.io 基础设施上构建的与 `fly` CLI 集成的 S3 兼容存储服务的私有 beta 版本。（Tigris 付了我一点钱来试用他们的 beta 版并撰写文章，但他们对内容没有任何发言权。）

我以前没听说过 Tigris，但这引起了我的兴趣 — 我在 Fly.io 上有很好的经验，我认为更多的小公司与 AWS 巨头竞争很棒。

几天前我仍然使用 Amazon S3 来运行用 Go 编写的 Gifty 服务器，但是现在不再如此。

切换很简单：复制文件，更新服务器配置以指向 Tigris 终端而不是 S3，并 `fly deploy`。由于 Tigris 处理认证和缓存的方式不同，我不得不进行了一个小的代码更改，并碰到了一些小问题，但总体上非常顺利。

## 创建存储桶

第一步是为用户上传的图像创建一个测试存储桶。我直接使用 `fly storage create` 命令：

```
$ cd gifty  # change to app directory
$ fly storage create --public
? Choose a name, use the default, or leave blank to generate one:
    test-gifty-registry-images
Your Tigris project (test-gifty-registry-images) is ready. See details and
next steps with: https://fly.io/docs/reference/tigris/

Setting the following secrets on gifty:
BUCKET_NAME
AWS_ENDPOINT_URL_S3
AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY
AWS_REGION

Updating existing machines in 'gifty' with rolling strategy

-------
 ✔ Machine 4d89601c6e9487 [app] update succeeded
-------
Checking DNS configuration for gifty.fly.dev 
```

令我惊讶的是，只需创建一个存储桶就自动设置了一堆环境变量并重新部署了我的应用程序。这是因为我在应用程序目录，即应用程序的 `fly.toml` 配置文件所在的位置，但这种行为似乎出乎意料且过于神奇 — 毕竟，我只是为本地开发创建了一个测试存储桶。

这在 [文档中有说明](https://fly.io/docs/reference/tigris/#create-and-manage-a-tigris-storage-bucket)，所以这部分是我的错 — 虽然他们可能应该补充“并重新部署”：

> 在 Fly.io 应用上下文中运行以下命令 — 在应用程序目录内或指定 `-a yourapp` — 将自动在您的应用程序上设置密钥。

这个功能可能会造成停机，所以我认为应该选择性地启用，例如 `--update-app` 或 `--add-secrets` 命令行选项。

没有简便的方法来访问那些自动设置的密钥的值。这是出于安全和设计考虑，但这意味着我不得不创建一个新的测试存储桶（在应用程序目录之外），以获取本地测试的凭证：

```
$ cd ..  # change out of app directory
$ fly storage create --public --org ben-hoyt --name test-gifty-registry-images2
Your Tigris project (test-gifty-registry-images2) is ready. See details and
next steps with: https://fly.io/docs/reference/tigris/

Set one or more of the following secrets on your target app.
AWS_REGION: auto
BUCKET_NAME: test-gifty-registry-images2
AWS_ENDPOINT_URL_S3: https://fly.storage.tigris.dev
AWS_ACCESS_KEY_ID: ***hidden***
AWS_SECRET_ACCESS_KEY: ***hidden*** 
```

好吧，这样更好了。然后我用那个访问密钥和密码更新了我的本地应用配置，它运行得很……大多数情况下都行。我首先得解决一些小问题，如下所述。

另一个创建存储桶时的怪癖——如果`fly` CLI的开发人员正在阅读这篇文章——是当你运行`fly storage create`时，它会以未定义的顺序打印出环境变量。你可以在上面看到：在第一个例子中，`BUCKET_NAME`首先被打印出来；在第二个例子中，`AWS_REGION`先出现。这不是问题，但有点奇怪，我反复确认确实两次打印出了相同的键。

## 端点变更

很明显，我需要开始使用新的Tigris API端点，而不是默认的AWS端点。在创建S3客户端时，我进行了以下操作：

```
cfg := aws.NewConfig().WithCredentials(creds).WithRegion(region) 
```

为了使用正确的端点，我添加了以下几行：

```
endpoint := os.Getenv("GIFTY_S3_BACKUP_ENDPOINT")
if endpoint != "" {
	cfg = cfg.WithEndpoint(endpoint)
} 
```

请注意，我仍在使用AWS SDK for Go的v1版本。这在[v2版本](https://www.tigrisdata.com/docs/sdks/s3/aws-go-sdk/)中有所不同。

## 缓存差异

当我使用Amazon S3时，用户为他们的婚礼注册表上传了一张新图片时，我发现只需上传到相同的文件名（键）即可。当页面重新加载并重新获取图片时，新图片就会出现。

我不确定即使使用S3，这是否是个好主意，因为最初S3只是[最终一致性](https://en.wikipedia.org/wiki/Eventual_consistency)的。然而，到了2020年，他们转向了[强一致性读后写](https://aws.amazon.com/blogs/aws/amazon-s3-update-strong-read-after-write-consistency/)，因此在PUT后的所有对文件的GET操作都会立即获取到更新的内容。

当我开始使用Tigris并重新上传图片时，我必须按下Ctrl-Shift-R（重新加载，覆盖缓存），新图片才会出现。Tigris似乎采用了更积极的缓存策略：从上传到Tigris的图片的响应头中可以看到，默认情况下它发送了`Cache-Control: max-age=3600`头部，尽管我没有设置这个头部。

我不确定这是一个好的默认设置，但因为他们同时想要成为[CDN](https://en.wikipedia.org/wiki/Content_delivery_network)和存储系统，我可以理解他们为什么选择了这种方式。这迫使人们使用像[基于内容的文件名](https://github.com/benhoyt/cdnupload)这样的缓存技术，这是件好事。

*更新：阅读本文后，Tigris添加了一篇关于[缓存的页面](https://www.tigrisdata.com/docs/caching/)，详细记录了这种行为和他们在这一点上的意图。*

无论如何，这给我提高Gifty处理上传的机会。我改变了代码，向文件名添加了一个简短的基于内容的哈希值：不再使用`{registryID}.jpg`作为键，而是使用`{registryID}-{contentHash}.jpg`，这解决了缓存问题：

```
// Add content-based hash to filename to break caching (12 hex digits).
hash := sha1.New()
_, _ = hash.Write(data)
key := fmt.Sprintf("%d-%x.jpg", registry.ID, hash.Sum(nil)[:6]) 
```

我还改变了完整的图片URL，不再硬编码`amazonaws.com`，并使URL格式可配置化：

```
// Old version
path := fmt.Sprintf("https://%s.s3-%s.amazonaws.com/%s",
    h.config.S3ImageBucket, h.config.S3ImageRegion, key)

// New version; under Tigris, I set S3ImageURLFormat to
// "https://fly.storage.tigris.dev/gifty-registry-images/%[1]s"
path := fmt.Sprintf(h.config.S3ImageURLFormat, key) 
```

## 认证差异

当使用 S3 时，我有两个存储桶——一个用于注册图像，一个用于备份——它们都使用相同的 AWS 密钥和密钥进行身份验证。这可能不是很好（也许我应该为备份使用不同的凭据），但根据我的经验，对于小型站点来说，只有一个密钥也是相当典型的。

当使用 Tigris 时，您会为创建的每个存储桶获得新的密钥和密钥。因此，我不得不进行一些代码调整，以添加两个新的与 S3 相关的环境变量。几行代码后，备份开始正常工作。

## 复制文件和“影子存储桶”

现在，在我的测试环境中一切正常运行后，我想要将生产站点切换过来。

对于从 S3 迁移，Tigris 有一个很酷的功能叫做[影子存储桶](https://www.tigrisdata.com/docs/migration/)，它允许您将 Tigris 存储桶指向原始 S3 存储桶，如果 Tigris 存储桶中缺少文件，则会无缝从“影子存储桶”中获取文件（并将文件复制到 Tigris，以便将来的请求可用）。

我可能应该尝试一下这个功能，但我决定不这样做，因为我的最大存储桶中只有几百个文件，而且我希望在完成后进行干净的切换，避免对 S3 的依赖。

因此，我使用 `aws s3 sync` 命令将文件从 S3 复制到本地驱动器，然后从本地复制到新的 Tigris 存储桶。我的站点规模非常小，所以在切换期间几分钟内几乎不可能遗漏任何文件。

我创建了两个 `aws` CLI 配置文件，分别用于新的 Tigris 存储桶的凭证，称为 `gifty-registry-images` 和 `gifty-backup`。默认配置文件仍然指向 Amazon S3。

这是我从 S3 复制到 Tigris 所使用的命令（我首先使用了方便的 `--dryrun` 选项）：

```
# Copy the user-uploaded images
$ mkdir gifty-registry-images
$ cd gifty-registry-images
$ aws s3 sync s3://gifty-registry-images .
...
$ aws s3 sync . s3://gifty-registry-images --profile gifty-registry-images
...

# Copy the backup files
$ cd ..
$ mkdir gifty-backup
$ cd gifty-backup
$ aws s3 sync s3://gifty-backup .
...
$ aws s3 sync . s3://gifty-backup --profile gifty-backup
... 
```

## 数据库迁移

只要我继续支付 AWS 费用，现有的 S3 URL 将继续正常工作。但是如前所述，我希望摆脱对 S3 的依赖。

我在站点的 SQLite 数据库中存储完整的图像 URL。所以在复制文件后，我使用简单的 SQLite 查询更新了数据库中的图像 URL：

```
-- First check how many S3 image URLs there are
sqlite> SELECT count(*) FROM registry
        WHERE image_url LIKE 'https://gifty-registry-images.s3%';
389

-- Perform the update
sqlite> UPDATE registry SET image_url = replace(image_url,
        	'https://gifty-registry-images.s3-us-west-1.amazonaws.com/',
        	'https://fly.storage.tigris.dev/gifty-registry-images/')
        WHERE image_url LIKE 'https://gifty-registry-images.s3%';

-- Ensure there are no more S3 URLs
sqlite> SELECT count(*) FROM registry
        WHERE image_url LIKE 'https://gifty-registry-images.s3%';
0 
```

之后，该站点完全迁移到了 Tigris 和 Fly.io，完全不再使用 AWS！

## 最后的想法

### 性能

我进行了一个快速的、不科学的测试，比较了从旧的 AWS 存储桶和新的 Tigris 存储桶中获取图像所需的时间。Tigris 存储桶“默认为全球”，因此它们可以使用最近的 Fly.io 地区（悉尼），这比 AWS 存储桶所在的 `us-west-1` 地区更接近新西兰。

正如预期的那样，从 Tigris 下载速度明显更快：对于小图像，AWS 大约需要 1 秒，而 Tigris 只需 300 毫秒。对于中等大小的图像，AWS 大约需要 1.3 秒，而 Tigris 只需 600 毫秒。

### 定价

我为 Gifty 的 AWS 账单——目前仅使用 S3——每月只有几美分。我相信，他们在管理开销上花费的比我实际支付的还要多。所以对我来说，定价并不是一个问题，但是，Tigris 的定价与 S3 类似：

+   [Tigris](https://www.tigrisdata.com/docs/pricing/)：存储费用为每GB每月$0.02，获取请求为每1000个请求$0.0005。

+   [S3](https://aws.amazon.com/s3/pricing/)：存储费用为每GB每月$0.023，获取请求为每1000个请求$0.0004。

无论如何，Tigris在测试版注册过程中给了我$150的免费信用额度。这相当于7500 GB月或3亿次GET请求 - 几乎肯定超出了我小型网站一生中的使用量。

### 免责声明和耐用性

有件事你可能想知道：当我注册他们的[**早期访问测试版**](https://hello.tigrisdata.com/forms/early-access/)时，Tigris给我发了封电子邮件说：“请在测试版期间避免将此服务用于生产工作负载！”

我问过，这只是一个免责声明来规避他们的责任，还是更像是“文件会随机丢失，并且我们将每晚删除所有数据”的声明。他们说这更像是一个免责声明，并且Fly.io和Tigris都把这个作为一个生产平台对待。显然存在风险，但我对此感到满意，而且他们很快就会退出测试版。

我也对Tigris的耐用性与S3相比有所疑惑，后者宣称其对象有99.999999999%的耐用性。Tigris显然对他们的[架构](https://www.tigrisdata.com/docs/concepts/architecture/)进行了很多考虑，但我想知道是否可能测量他们的耐用性并与[S3的惊人数字](https://aws.amazon.com/blogs/aws/new-amazon-s3-reduced-redundancy-storage-rrs/)进行比较？不过，S3的十一个9的数字[实际上有意义吗](https://stackoverflow.com/a/4188501/68707)？

### 结束

尽管如此，我在进行切换时有非常好的经验，并且到目前为止还没有遇到任何问题 - 但我相信时间会告诉我们一切！我希望Tigris和Fly.io继续做得不错，并为大型云公司引入一些急需的竞争。
