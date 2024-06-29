<!--yml

category: 未分类

date: 2024-05-29 12:44:11

-->

# 争取取代Redis的竞争[LWN.net]

> 来源：[https://lwn.net/SubscriberLink/966631/6bf2063136effa1e/](https://lwn.net/SubscriberLink/966631/6bf2063136effa1e/)

| **本文由LWN订阅者提供支持**LWN.net的订阅者使本文及其周边所有内容成为可能。如果您欣赏我们的内容，请[购买订阅](/subscribe/)，支持我们创作更多文章。 |
| --- |

作者**Joe Brockmeier**

2024年3月28日

2024年3月21日，[Redis Ltd.](https://redis.com/company/)宣布[Redis](https://redis.io/)的“内存数据存储”项目将以非自由、源可用许可证发布，从Redis 7.4版本开始。这个消息虽然不受欢迎，但并非完全出乎意料。这种情况的不寻常之处在于可以选择的Redis替代方案数量众多；至少有四个选项可供选择，包括一个名为[KeyDB](https://docs.keydb.dev/)的预先存在的分支和Linux基金会新宣布的[Valkey](https://www.linuxfoundation.org/press/linux-foundation-launches-open-source-valkey-community)项目。现在的问题是Linux发行版、用户和提供者将选择哪一个（或哪些）来替代它。

#### Redis的简史

Redis有着[复杂的背景故事](https://www.gomomento.com/blog/rip-redis-how-garantia-data-pulled-off-the-biggest-heist-in-open-source-history)。Salvatore Sanfilippo（也被称为“antirez”）开始这个项目，用于一个名为[LLOOGG](https://github.com/antirez/lloogg)的实时日志分析应用，因为MySQL无法满足他的需求，所以他设计了这个简单的字典数据库项目，它将键值对存储在内存中——其名称是“远程字典服务器”的缩写。当然，多年来它已经成熟并积累了[许多更多的功能](https://redis.io/docs/about/)。Redis很快成为[NoSQL运动](http://radar.oreilly.com/2012/02/nosql-non-relational-database.html)的一部分，并于2010年被VMware聘请参与Redis的开发。他在2013年转移到VMware的分拆公司Pivotal，并继续致力于这个项目。

此时，Redis正日益流行，被[Twitter](https://highscalability.com/how-twitter-uses-redis-to-scale-105tb-ram-39mm-qps-10000-ins/)、[Pinterest](https://tanzu.vmware.com/content/blog/using-redis-at-pinterest-for-billions-of-relationships)等高知名度公司广泛采用，并开始出现在Linux发行版中。它于Ubuntu 12.04（2012年4月）版本、Fedora 18（2013年1月）等中进行了打包。Redis支持于2013年9月[加入](https://aws.amazon.com/blogs/aws/amazon-elasticache-now-with-a-dash-of-redis/)Amazon Web Service的ElastiCache服务，利用并推动了Redis的流行。

2013年初，名为Garantia Data的初创公司开始提供Redis服务，并将其定位为"<q>开源Redis</q>"的更好替代品，详情请见[此处](https://web.archive.org/web/20130529183307/http://garantiadata.com/blog/taking-in-memory-nosql-to-the-next-level)。Garantia于2013年11月获得首轮融资，并计划更名为RedisDB。但在与Sanfilippo的一些抵制后，公司最终在2014年初改名为Redis Labs，详情请见[此处](https://techcrunch.com/2014/01/29/database-provider-garantia-data-makes-another-name-change-this-time-to-redis-labs/)。Sanfilippo于2015年加入Redis Labs，担任开源开发主管，并一直服务到2020年[辞职](http://antirez.com/news/133)。

2018年，Redis Labs为其核心数据库之上提供功能的附加模块[采用了](https://redis.com/blog/redis-license-bsd-will-remain-bsd/)新的许可证。公司选择了修改版的[Apache License, Version 2.0](https://www.apache.org/licenses/LICENSE-2.0)，并增加了称为[Commons Clause](http://commonsclause.com/)的条款。该条款[限制了](https://lwn.net/Articles/763179/)销售软件或收费服务的行为。当时的理由是云服务提供商"<q>利用开源社区</q>"，销售他们未开发的基于开源代码的服务。公司承诺Redis"<q>是BSD许可证，将永远保持BSD</q>"。

并非唯一一家开始尝试使用限制性许可证的公司。特别是风险投资支持的数据库公司，开始采用新的许可证以确保能够独家销售使用该软件的服务。MariaDB 于2016年为名为MaxScale的产品创建了[商业源许可证](https://mariadb.com/bsl-faq-adopting/#whatis)（BSL），而MongoDB 则于2018年末发布了[服务端公共许可证](https://www.mongodb.com/licensing/server-side-public-license)（SSPL）。Redis Labs 最终采用了[双重许可方案](https://redis.com/legal/licenses/)，包括SSPL及其自己的[Redis源可用许可证](https://redis.com/legal/rsalv2-agreement/)（RSAL）用于其模块。

公司于2021年中旬[将“Labs”从其名称中删除](https://redis.com/press/redis-labs-becomes-simply-redis/)。Redis再次承诺坚持开源，并表示公司更名“不会影响开源Redis的许可证，它始终是并将继续采用BSD许可证”。公司还实施了一种治理模型，该模型将Redis本身的“架构、设计或理念”的重大决策交由社区“核心团队”负责。可以在Redis的网站上找到治理页面，但已[存档](https://web.archive.org/web/20220326025023/https://redis.io/docs/about/governance/)在互联网档案馆的Wayback Machine上。该页面列出了由五名成员组成的核心团队，其中三名来自Redis（Yossi Gotlieb、Oran Agra和Itamar Haber），以及来自阿里巴巴的Zhao Zhao和来自AWS的Madelyn Olson。

Redis于2021年中旬[宣布](https://redis.com/blog/redis-adopts-dual-source-available-licensing/)，“未来Redis所有版本将以源可用许可证发布”，具体包括SSPL和RSAL。Redis CEO Rowan Trollope 写道，保持BSD许可证现在“与我们成功将Redis推向未来的能力背道而驰”。在这种情况下，“未来版本”指的是Redis 7.4及以后的版本。该公告的常见问题解答页面指出，根据公司的[安全政策](https://github.com/redis/redis/security/policy)，安全补丁将被回溯到早期版本，并保留原始的三条款BSD许可证。

#### 云与开源

SSPL和Redis的RSAL等使用限制性许可证的支持者曾试图将这视为巨头云服务提供商（如AWS）与开源之间的唯一对抗，使用限制性许可证是唯一的合理选择，云服务提供商则是唯一的输家。2019年，Redis Labs的前CEO奥弗·本加尔（Ofer Bengal）在[引述](https://techcrunch.com/2019/02/19/redis-labs-raises-a-60m-series-e-round/)中称，Redis采用其Redis模块的源代码可用许可证后，“有很多不同的看法”，但与云服务提供商竞争是必要的：

> 一些人谴责了[许可证变更]。但在最初的噪音平息之后，特别是在其他公司提出类似概念之后，社区现在理解了，原始的开源概念必须进行修正，因为在云公司利用其垄断力量无偿采用任何成功的开源项目的现代时代，它已不再适用。

在3月20日的公告中，特罗洛普写道：“<q>云服务提供商只有在同意Redis，Redis代码的维护者的许可条款后，才能交付Redis 7.4</q>”，但是，“<q>对于Redis开发社区来说，继续享受双许可下的宽松许可，什么也没有改变</q>”。

"<q>宽松许可</q>"这个词汇的选择是具有误导性的。Redis 7.4版本及之后版本能够采用非自由许可证体系，因为外部开发者已经根据宽松的BSD许可证授权了他们的贡献。SSPL和RSAL的条款与开源社区中“宽松”一词的普遍用法不兼容。

云服务提供商声称他们不参与Redis [仓库](https://github.com/redis/redis)的贡献，这与实际的提交记录难以调和。通过对自7.0.0发布以来的提交使用[gitdm](https://lwn.net/Articles/290957/)进行快速检查，该时间段内有967次提交：

> | 雇主贡献的最重要的变更集 |
> | --- |
> | (未知) | 331 | 34.2% |
> | 腾讯 | 240 | 24.8% |
> | Redis | 189 | 19.5% |
> | 阿里巴巴 | 65 | 6.7% |
> | 华为 | 50 | 5.2% |
> | 亚马逊公司 | 50 | 5.2% |
> | 字节跳动 | 19 | 2.0% |
> | 网易 | 13 | 1.3% |

腾讯的朱彬彬（Binbin Zhu），在项目中贡献了近25%的提交。一些贡献者没有明确的雇主身份，但显然Redis公司并非孤军作战。（请注意，一些个位数的贡献者被省略了。）

#### 变更分发模型

因此，显然代码贡献并非重点。Redis是一家风险投资支持的公司，自2011年以来通过多轮融资筹集了超过3.5亿美元。公司及其投资者似乎已经计算出，他们可以安全地摆脱开源以试图获得更多收入。

如果MongoDB的结果是任何指导，他们有理由相信这是对的。公司于2017年上市，一年多后转向了SSPL。不久之后，主要Linux发行版停止打包数据库，因为它不再符合其许可标准。但是，此时，公司已经把目光投向了[平台模型](https://redmonk.com/sogrady/2017/07/14/mongodb-platform-company/)，鼓励开发人员（及其雇主）使用和支付MongoDB及其附属产品的服务模型。分发MongoDB的源代码可视为[引导损耗](https://en.wikipedia.org/wiki/Loss_leader)策略，以吸引开发人员，而公司则打赌[不关心](https://www.infoworld.com/article/3572324/do-developers-really-care-about-open-source.html)开源的开发人员。

如Redmonk创始人斯蒂芬·奥格拉迪在2017年所写：

> 当开发人员开始在技术选择和方向上越来越多地表现出控制力时，即使在专有替代方案在技术上更为优越的情况下，开源软件的极大可访问性也使其在市场上具有巨大的优势。在可以立即下载的足够选择A与需要销售人员推广的理论上更优选择B之间的选择实际上并不是一个选择。

但是，正如奥格拉迪指出的那样，“<q>与基于服务的替代方案相比，开源通常不那么方便</q>”，如果便利性是最重要的因素，那么开源就有问题。特别是当像MongoDB这样的供应商从专有供应商那里学到“<q>锁定客户有利于业务</q>”时。

对业务有益吗？MongoDB不断增长，增加客户，上一财年带来了16.8亿美元的收入。这比去年增加了30%以上，其Atlas数据库服务的收入也增长了30%以上，表明很多公司更愿意支付使用服务，而不是自行托管。尽管如此，公司仍然亏损——同一时期的亏损超过3.45亿美元。

但投资者可能更关心股价而非实际利润。公司上市时股价约为33美元一股，但现在已超过350美元一股。如果Redis能够产生类似的结果，其投资者可能会感到满意。

#### 分支和替代方案

作为 O'Grady 在去年 [所写](https://redmonk.com/sogrady/2023/08/03/why-opensource-matters/) 的，受风险资本支持的供应商似乎达成了一致意见，认为他们可以远离开源。特别是如果他们没有被 "<q>其他商业利益、基金会和其他感兴趣的行业参与者积极反对</q>" 的话。在这里，Redis 可能错误地评估了行业的情绪。

当 Hashicorp 在去年为其项目采用 BSL 时，其 Terraform 项目的一个分支在几天内出现，并在 Linux Foundation 的支持下被接受，名为 [OpenTofu](https://opentofu.org/)。基金会在 3 月 28 日宣布，它正在支持 Valkey，这是 Redis 7.2.4 的一个直接分支，由 Amazon Web Services（AWS）、Google Cloud、Oracle、Ericsson 和 Snap 支持。

Valkey 分支在 Redis 许可证变更后的几天内诞生。Olson [在推特上写道](https://twitter.com/reconditerose/status/1771237000592642274)，她和 "<q>各种前 Redis 贡献者</q>" 开始在 [fork](https://github.com/placeholderkv/placeholderkv) 上进行工作，使用原始的三条款 BSD 许可证，以 "placeholderkv" 作为临时名称。Olson、Zhao、Viktor Söderqvist 和 Ping Xie 被列为维护者。根据 Olson 的说法，这不是 Redis 的 AWS 分支，而是 "<q>我只是试图与社区保持连续性</q>"。KeyDB 被考虑过，但它已经分化到 "<q>缺少社区习惯的很多东西</q>" 的程度。

KeyDB 分支是在 2019 年创建的，原因是技术上的考量，而不是许可问题。该项目自称为 "<q>Redis 的更快替代方案</q>"，由 John Sully、Eric Blenkarn 和 Ben Schermel 创建，他们希望 [有一个多线程版本](https://docs.keydb.dev/blog/2019/10/07/blog-post/)，但未能说服 Redis 的维护者朝这个方向发展。Sully 和 Schermel 成立了一家名为 KeyDB 的公司，提供了一个专有的企业版本。当 KeyDB 在 2022 年被 Snap 收购时，整个代码库完全开源，采用了三条款的 BSD 许可证。

KeyDB作为Redis的直接替代品的问题在于自从分叉以来它并没有跟上Redis的步伐。它仍然缺少很多Redis 7中的特性，Sully在一个问题上表明，他没有太多时间处理那些 "<q>不直接影响Snap的问题</q>"，尽管项目 "<q>当然欢迎外部帮助，如果社区有兴趣帮忙，我们当然可以指定额外的维护者</q>"。3月22日，Sully更新了另一个问题，并表示正在讨论 "<q>潜在地</q>" 添加维护者以使KeyDB更接近Redis 7。目前还不清楚Valkey是否会取代KeyDB，但Snap的参与使这一长期看来可能性较大。

[SourceHut](https://sourcehut.org/)的创始人兼CEO Drew DeVault也创建了一个基于Redis 7.2.4的名为Redict的分支，但选择使用LGPLv3许可证。在他的[公告文章](https://redict.io/posts/2024-03-22-redict-is-an-independent-fork/)中，他表示许可证的选择是 "<q>一种权衡各种关切的有意的选择</q>"。DeVault希望选择一个既是copyleft又 "<q>尽可能容易让用户遵守</q>" 的许可证，并且易于与Redis兼容的模块或[Lua插件](https://redis.io/docs/interact/programmability/eval-intro/)进行集成，以执行Redis内的操作。他还指出，Redict将没有贡献者许可协议（CLA），但贡献者需要使用[开发者原产地证书](https://developercertificate.org/)验证贡献。尽管与SourceHut有关联，DeVault选择在[Codeberg上托管Redict](https://codeberg.org/redict/redict)以 "<q>为所有熟悉基于GitHub社区的Redis的用户提供舒适和熟悉的用户体验</q>"。

另一个潜在竞争者是微软的 [Garnet](https://github.com/microsoft/garnet)，[于3月18日宣布](https://www.microsoft.com/en-us/research/blog/introducing-garnet-an-open-source-next-generation-faster-cache-store-for-accelerating-applications-and-services/)。根据宣布，它由微软研究部门自2021年以来开发。它是一个远程缓存存储，可以缓存和管理与Redis相同类型的数据，并设计为兼容[Redis序列化协议](https://redis.io/docs/reference/protocol-spec/)。Garnet采用MIT许可证，用.NET C#编写，并不打算作为直接的插拔替代品。然而，它的 [API兼容性](https://microsoft.github.io/garnet/docs/welcome/compatibility) 页面声称它可以被“<q>视为一个足够接近的起点</q>”，可以“<q>与许多Redis客户端无需修改地工作</q>”。但不是所有的。例如，有用户尝试将NodeJS应用切换到Garnet，但 [发现](https://github.com/microsoft/garnet/issues/112) 目前不支持Redis的 `[FLUSHALL](https://redis.io/commands/flushall/)` 命令。支持缺失API在项目的 [路线图](https://microsoft.github.io/garnet/docs/welcome/roadmap) 中有规划。

#### 寻找替代方案

Linux 发行版再次面临混乱局面。尼尔·戈姆帕（Neal Gompa） [在Fedora开发列表上开了个讨论](https://lwn.net/ml/fedora-devel/CAEg-Je_GoiJN6kOj1_K5WqTvA6n0j8r4fi9=C7-WbXLHovM3ow@mail.gmail.com/)，指出了许可证变更和需要移除Fedora中Redis的问题。乔纳森·赖特（Jonathan Wright） [回复说](https://lwn.net/ml/fedora-devel/CAKe4=-JijFraJbn3yXi-ieWA5mXdGzDrZrEVLpnDUY0EGpdr5g@mail.gmail.com/) KeyDB可能是一个替代选择；他在许可证变更之前“<q>松散地在进行打包工作</q>”。他后来 [表示](https://lwn.net/ml/fedora-devel/CAKe4=-J4ed9dRfFr37jb6Mhq1yM0=0RrZb3N8ftxpJag4xF8HQ@mail.gmail.com/) KeyDB将是“<q>向后退一步并带来麻烦</q>”对于那些想要替换Redis较新版本的人。尽管如此，他 [在3月23日写道](https://lwn.net/ml/fedora-devel/CAKe4=-+PxSZLPVT78-CCYrGdvssTOEmJ9v4xoSo0inUfv1cV_Q@mail.gmail.com/) 他已经推送了适用于Fedora和EPEL 8及9的测试版本。

瓦尔基（Valkey）发布后不久，赖特（Wright） [写道](https://lwn.net/ml/fedora-devel/CAKe4%3D-Lzd1mNOhAsja%2BVRXz%3D78Y8ESUb370q90Fm8i5npWGaaw%40mail.gmail.com/) 他将在发布标记版本后尽快打包它。赖特还说他“<q>期待瓦尔基在大多数地方取代Redis成为事实上的替代品。</q>”

Gompa还在openSUSE的Factory讨论列表中[提出了](https://lwn.net/ml/opensuse-factory/CAEg-Je9o5os4ZoBujO_6K=Kv45qebOP5M+BKOfFSd_aw3_vCxw@mail.gmail.com/)这个问题。Dominique Leuenberger以[18个](https://lwn.net/ml/opensuse-factory/3873a1ecbc97440ff0db7fb28b0c6f079b7a6649.camel@opensuse.org/)依赖于Tumbleweed中`redis`包的包列表回复。最初的讨论提到了Redict和KeyDB作为可能的替代品，但当时Valkey尚未公布。

社区发行版必须寻找替代品来替代Redis的问题并非唯一困扰。Jacob Michalskie在[这里](https://lwn.net/ml/opensuse-factory/AB7886BD-E33E-4ACA-8F94-AA4A1B1FECDB@opensuse.org/)指出了openSUSE项目中几个需要替代Redis的服务，包括[Fedora](https://pagure.io/pagure)创建和使用的代码托管软件Pagure，用于[code.opensuse.org](https://code.opensuse.org/)，以及[Discourse](https://www.discourse.org/)论坛软件。

Debian贡献者Guillem Jover在[这里](https://lwn.net/ml/debian-devel/ZfwDpCHE7Zq2vKN-%40thunder.hadrons.org/)提交了KeyDB作为Redis潜在替代品的[包请求](https://wiki.debian.org/RFP)（RFP）。Jover表示他不确定自己是否适合独自维护，但愿意提供帮助。在与Jover的电子邮件交流中，他告诉我他的公司已经从Redis 6迁移到了KeyDB，并且过渡“非常顺利”。根据Jover的说法，“与Redis 7相比，KeyDB可能缺少一些功能，但我们既未注意到任何缺失，也未感觉到任何遗漏。”

Jover表示现在还为时过早以判断这些新分支是否会继续维护，并且Redict的LGPLv3许可可能也会对生态系统造成问题。在Valkey公告后的一封跟进电子邮件中，他表示：“我认为我们可能会进一步将KeyDB打包到Debian中，如果它停止开发，总是可以将其删除或者过渡出去。”

#### 前进的路径

当然，现在预测新分支是否会获得显著关注还为时过早，但Valkey似乎是一个可信的替代方案。快速分支的可能性，获得广泛的社区和行业支持，应该使期望在放弃开源后平稳过渡的供应商三思而后行。

* * *

(

[登录](https://lwn.net/Login/?target=/Articles/966631/)

以发布评论)
