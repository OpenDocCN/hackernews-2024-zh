<!--yml

分类：未分类

日期：2024-05-27 12:58:57

-->

# 为什么 AWS 支持 Valkey | AWS 开源博客

> 来源：[https://aws.amazon.com/blogs/opensource/why-aws-supports-valkey/](https://aws.amazon.com/blogs/opensource/why-aws-supports-valkey/)

在 Redis 公司宣布[移除开源许可](https://redis.com/blog/redis-adopts-dual-source-available-licensing/)并退出 Redis 项目不到一周后，Redis 的贡献者们联合起来将社区转移到了 Linux Foundation，成为[Valkey 项目](https://github.com/valkey-io/valkey)。他们几乎在许可变更公告发布后立即采取行动，响应 GitHub 和社交媒体上社区的呼声，要求分叉项目或加入现有的分支。

这种反应并不奇怪 — 当其项目重新许可时，我们曾看到其他社区做出类似的反应。开源开发者和其他人在建立项目以及依赖其上的产品和服务时投入了大量时间和资源。他们不能只是离开。

低估那些帮助 Redis 和其他开源技术成为今天的样子的忠实用户和贡献者的贡献是一个错误。尽管 Redis 在最近几年的大多数提交可能来自 Redis 的员工，但许多其他贡献者为项目增加了重要价值，这一点不应被忽视 — 尤其是考虑到 Redis 员工也决定了接受哪些贡献。

Redis 与帮助其成长的社区决裂，并让他们陷入困境。现在这个社区摆脱了束缚，将继续像往常一样使用和贡献该项目，而且更加自由。他们希望该项目得到良好支持，并能在他们喜爱的 Linux 发行版或云实例中使用，并且可以按照他们的意愿自由使用。将项目保持在[开源倡议组织（OSI）批准的](https://opensource.org/licenses)BSD 3条款许可下，有助于社区确保 Valkey 的分发和无限制使用。

通过移动到 Linux Foundation，社区向个人和公司保证他们可以继续参与开发，不用担心开源许可证被撤销。此举还确保没有单一组织会对项目产生过大影响。基金会的使命是提供支持开源项目长期健康和可持续性的环境。在开放治理模型下，该项目将吸引多样化的贡献者，更快地创新，并在长期内保持活力和安全。

开源工具 Terraform 现在作为 OpenTofu 继续存在，于一月份发布了其 [首个稳定版本](https://opentofu.org/blog/opentofu-is-going-ga/)，作为一个中立供应商的、由社区驱动的项目。该项目正在快速开发新功能，吸引新的贡献者，并且其采用率正在增长。我们希望 Valkey 项目也能有类似的发展轨迹。

Valkey 背后有一个强大的社区，将继续维护和发展该项目，根据项目社区已经制定的路线图。在 AWS、Google、Oracle、Ericsson、Snap 等企业支持下，该项目正在朝着下一个版本的目标迈进。

## AWS 对 Valkey 的贡献与建设

AWS 承诺长期支持开源项目 Valkey。我们正在向我们的 ElastiCache 和 MemoryDB 托管数据库服务添加 Valkey 的支持，这些服务构建在开源 Redis 上，并与开源 Redis 版本 7.0 及更早的版本兼容。我们的工程师也将继续为开源 Valkey 项目做出贡献，以帮助保障其安全性、增加新功能并进行创新。与此同时，许可证的变更不会影响现有或新的 ElastiCache 和 MemoryDB 应用程序，AWS 客户也无需作出任何更改。

我们对项目的成功有着极大的商业兴趣。我们还有一些经验，即在其开源项目许可证发生变化后，将社区聚集起来形成一个新项目。在 Elastic 在 2021 年更改 Elasticsearch 许可证后，我们采取了必要的步骤，以确保项目保持开放，并可供社区贡献和使用。其结果是 [OpenSearch](https://opensearch.org/)，这是一个采用 Apache 2.0 许可的项目，已经建立了一个多元化的贡献者基地，并且现在也在 [迈向开放治理的步骤](https://aws.amazon.com/blogs/opensource/opensearch-expands-leadership-beyond-aws/)。

我们从中学到了很多经验，包括参与和向上游开源项目的贡献对于我们建立的和我们的客户依赖的项目的重要性。四年前，当 Redis 社区向开放治理模式转变时，我们参与其中并增加了两名新的项目维护者，包括 AWS 的首席工程师和长期贡献者 Madelyn Olson。

在过去的四年中，AWS 向开源 Redis 做出了数百项贡献，包括对 [细粒度访问控制](https://github.com/redis/redis/pull/9974)、[协调故障切换](https://github.com/redis/redis/pull/8315)（以消除数据丢失并减少计划故障切换的停机时间）、以及为 TLS 提供基础工作的初始工作，TLS 是广泛采用的安全协议，旨在促进互联网上的数据隐私和安全性。

Redis 只是 AWS 大量贡献的众多项目中的一个示例。其他一些知名的示例包括 Apache Airflow、Apache Cassandra、Apache Flink、Apache Hudi、Apache Kafka、Apache Lucene、Containerd、Kubernetes、OpenJDK、OpenTelemetry、PostgreSQL、Project Jupyter 和 Rust。请查看[在 AWS 对云原生开源项目的贡献背后](https://aws.amazon.com/blogs/opensource/behind-the-scenes-on-aws-contributions-to-cloud-native-open-source-projects/)以及[在 AWS 对开源数据库的贡献背后](https://aws.amazon.com/blogs/opensource/behind-the-scenes-on-aws-contributions-to-open-source-databases/)，详细了解多个开源项目的关键贡献。

因为 AWS 参与并深度承诺，我们了解社区的历史，并且知道由社区主导的努力符合所有 Redis 用户和贡献者的最佳利益，包括我们的数据库客户。

我们的 ElastiCache 和 MemoryDB 客户将继续享受我们为所有开源托管服务提供的世界级运营支持，并且他们将从多个利益相关者共同协作推动创新的过程中受益。拥有更多不同贡献者的开源项目也能够更快地创新，并在长期内保持更健康和更安全。

我们很高兴作为 Linux Foundation Valkey 项目的众多利益相关者之一做出贡献。请加入我们在 GitHub 上继续[Valkey](https://github.com/valkey-io/valkey)的开源开发。
