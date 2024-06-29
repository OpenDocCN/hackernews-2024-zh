<!--yml

category: 未分类

date: 2024-05-27 14:59:26

-->

# 推出Sunlight，这是一个为了可伸缩性、操作简易性和降低成本而构建的CT实现 - Let's Encrypt

> 来源：[https://letsencrypt.org/2024/03/14/introducing-sunlight.html](https://letsencrypt.org/2024/03/14/introducing-sunlight.html)

Let's Encrypt自豪地推出Sunlight，这是一个新的证书透明度日志实现，我们从头开始构建，考虑了现代Web PKI的机会和约束条件。与[Filippo Valsorda](https://filippo.io/)合作，他领导了设计和实现，我们结合了来自更广泛的透明度日志社区的反馈，包括Google的Chrome和TrustFabric团队、Sigsum项目以及其他CT日志和监控运营者。他们的见解对于塑造项目的方向至关重要。

CT在Web PKI中扮演着重要角色，增强了监视和研究证书签发的能力。然而，CT日志的运作面临着随着证书数量增加而增加的挑战。例如，Let's Encrypt每天签发超过400万个证书，每个证书必须在两个独立的CT日志中记录。我们旗下的“Oak”日志目前已经有超过7亿条条目，反映了这些挑战的重大规模。

在本文中，我们将探讨Sunlight背后的动机以及其设计如何旨在提高CT生态系统的稳健性和多样性，同时改善Let's Encrypt日志的可靠性和性能。

## 数据库的瓶颈

Let's Encrypt自2019年以来一直在[运行公共CT日志](https://letsencrypt.org/docs/ct-logs/)，我们已经积累了大量运维经验，但并非一帆风顺。我们在“Oak”日志的架构中面临的最大挑战是数据存储在关系型数据库中。我们通过将每年的数据拆分为具有自己数据库的“分片”，然后稍后将这些分片缩小到覆盖半年而不是整整一年的方式，[扩展了它](https://letsencrypt.org/2022/05/19/nurturing-ct-log-growth)。

将数据库进一步分割并不是我们希望永远继续进行的事情，因为操作负担和成本会增加。当前每个CT日志分片的存储大小在5到10 TB之间。这对于单个数据库来说已经足够令人担忧：我们之前在MySQL中遇到[16TiB的限制](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/MySQL.KnownIssuesAndLimitations.html#MySQL.Concepts.Limits.FileSize)，导致一个测试日志失败。

扩展读取能力需要大型数据库实例，具有快速磁盘和大量RAM，这些都不便宜。我们曾多次遇到CT日志因客户尝试读取日志中所有数据而过载，从而使数据库过载。当施加速率限制以防止过载时，客户被迫缓慢爬取API，这降低了CT作为检测误颁发证书的快速机制的效率。

## 提供瓷砖

最初，Let’s Encrypt 只计划构建一个新的CT日志实现。然而，我们与Filippo的讨论使我们意识到其他透明系统已经改进了原始的证书透明设计，通过改变读取路径API，我们可以使我们的日志更加健壮和可扩展。特别是[Go Checksum Database](https://golang.org/design/25530-sumdb)受证书透明的启发，但使用更高效的格式发布其数据，作为一系列易于存储和缓存的瓷砖。

证书透明日志是一个二叉树，每个节点包含其两个子节点的哈希。“叶”级别包含日志的实际条目：证书，附加在树的右侧。树的顶部被数字签名。这形成了一个称为默克尔树的密码学可验证结构，可用于检查证书是否在树中，以及树是否仅追加。

日光瓷砖是每个包含256个元素的文件，每个元素都是树“高度”处的哈希值或者是叶级别的证书（或预证书）。Russ Cox 在他的博客上很好地解释了[瓷砖如何工作](https://research.swtch.com/tlog#tiling_a_log)，或者你可以阅读[日光规范的相关部分](https://c2sp.org/sunlight#merkle-tree)。甚至我们目前运行的CT实现 Trillian，也[使用类似这些瓷砖的子树系统](https://github.com/google/trillian/blob/master/docs/storage/storage.md)作为其内部存储。

与以前的CT API中的动态端点不同，将树作为瓷砖提供不需要任何动态计算或请求处理，因此我们可以消除API服务器的需求。因为瓷砖是静态的，它们可以高效地被缓存，与像按哈希获取证明这样的CT API不同，后者对每个证书都有不同的响应，因此没有共享缓存。叶瓷砖还可以压缩存储，进一步节省存储空间！

将日志公开为一系列静态瓷砖的想法是由我们希望通过横向和相对廉价地扩展读取路径而驱动的。我们可以直接在云对象存储（如S3）中公开瓷砖，使用缓存CDN，或使用Web服务器和文件系统。

对象或文件存储易于获取，可以轻松扩展，并且成本显著低于云提供商的数据库。这似乎是显而易见的前进道路。事实上，我们已经在现有的CT日志前面有一个S3后备缓存，这意味着我们当前的数据存储了两次。

## 运行更多的日志

瓦片API改进了读取路径，但我们也希望简化写入路径的架构。使用Trillian，我们运行一组节点以及etcd进行领导选举，选择将处理写入的节点。这有些复杂，我们认为CT生态系统允许不同的权衡。

关键的认识是，证书透明性已经是一个分布式系统，客户端将证书提交到多个日志中，并从任何不可用的日志优雅地切换到其他日志。每个单独日志的写入路径不需要高度可用的领导选举系统。一个简单的单节点写入器可以满足CT日志的99%服务水平目标。

单节点Sunlight架构允许我们使用相同的计算能力运行多个独立的日志。即使每个单独的日志具有较低的潜在正常运行时间，这也增加了系统的整体稳健性。不再需要领导选举。我们使用简单的比较和交换机制来存储检查点，并防止意外同时运行两个实例，这可能导致分叉树，但这比领导选举的开销要小得多。

## 不再有合并延迟

CT的目标之一是将提交到日志的延迟限制在很小范围内。添加了一个名为合并延迟的设计特性来支持这一点。当向日志提交证书时，日志可以立即返回一个签名的证书时间戳（SCT），承诺在日志的最大合并延迟内（通常为24小时）包含它。虽然这看起来是一个不减缓发布速度的良好权衡，但已经发生过多次事件和接近事件，其中一个日志停止操作未合并的证书，错过了其最大合并延迟，并破坏了承诺。

Sunlight采用了一种不同的方法，它在批处理和集成证书到日志时保持提交，从而消除了合并延迟。尽管这会导致轻微的延迟增加，但我们认为避免一些常见的CT日志故障案例是值得的。

它还允许我们在我们的SCTs的扩展中嵌入最终的叶子索引，将CT更接近直接客户端验证Merkle树证明。该扩展还使客户能够从新的基于静态瓦片的API中获取日志包含证明，而无需服务器端查找表或数据库。

## 一个阳光明媚的未来

今天的 Sunlight 发布只是个开始。我们已经发布了[软件](https://github.com/FiloSottile/sunlight)和[规范](https://c2sp.org/sunlight)，并运行了 Sunlight CT 日志。访问[sunlight.dev](https://sunlight.dev)获取相关资源。我们鼓励 CA 开始向[Let’s Encrypt 的新 Sunlight CT 日志](https://letsencrypt.org/docs/ct-logs/#Sunlight)提交测试，以便 CT 监控者和审计员支持消费 Sunlight 日志，并请 CT 程序考虑信任在这种新架构上运行的日志，希望 Sunlight 日志在未来能被浏览器运行的 CT 程序用于生成 SCTs，从而使 CA 能够依赖它们来满足浏览器的 CT 日志要求。

到目前为止，我们收到了积极的反馈，例如“Google 的 TrustFabric 团队，Trillian 的维护者，支持这个方向和 Sunlight 规范。我们一直在为其他生态系统提供可缓存的基于瓦片的日志，使用[无服务器工具](https://github.com/transparency-dev/serverless-log)，并将其与 Trillian 和 ctfe 整合，同时支持 Sunlight API。”

如果您对设计有任何反馈意见，请加入[ct-policy 邮件列表](https://groups.google.com/a/chromium.org/g/ct-policy)，或在 transparency-dev Slack 的 [#sunlight](https://transparency-dev.slack.com/archives/C06PCS2P75Y) 频道中参与讨论（[邀请链接](https://join.slack.com/t/transparency-dev/shared_invite/zt-27pkqo21d-okUFhur7YZ0rFoJVIOPznQ)）。

我们要感谢 Chrome 对 Sunlight 开发的支持，以及亚马逊网络服务（AWS）对我们 CT 日志运营的持续支持。如果您的组织监控或重视 CT，请考虑提供财政支持。了解更多信息，请访问[https://www.abetterinternet.org/sponsor/](https://www.abetterinternet.org/sponsor/)或通过电子邮件联系我们：[sponsor@abetterinternet.org](mailto:sponsor@abetterinternet.org)。
