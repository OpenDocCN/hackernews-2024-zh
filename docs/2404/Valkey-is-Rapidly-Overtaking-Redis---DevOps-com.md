<!--yml

category: 未分类

date: 2024-05-27 13:19:42

-->

# Valkey正在迅速超越Redis - DevOps.com

> 来源：[https://devops.com/valkey-is-rapidly-overtaking-redis/](https://devops.com/valkey-is-rapidly-overtaking-redis/)

三月份，[Redis宣布](https://redis.io/blog/redis-adopts-dual-source-available-licensing/)放弃其[BSD 3条款许可证](https://opensource.org/license/bsd-3-clause)，而采用“源代码可获取” [Redis源代码可获取许可证](https://redis.com/legal/rsalv2-agreement/)（RSALv2）和[服务器端公共许可证](https://redis.com/legal/server-side-public-license-sspl/)（SSPLv1）。这一举措令[开发者和用户感到不满](https://www.theregister.com/2024/03/22/redis_changes_license/)。因此，社区成员便[通过](https://www.theregister.com/2024/04/12/linux_foundation_opinion/) [Valkey](https://www.theregister.com/2024/04/12/linux_foundation_opinion/)的方式分叉了代码，得到了Linux基金会的支持。这一举措比预期中更为受欢迎。

马德琳·奥尔森，亚马逊网络服务（AWS）软件工程师兼Valkey维护者，在西雅图的[开源峰会](https://events.linuxfoundation.org/open-source-summit-north-america/)上宣布了[第一个发布](https://valkey.io/blog/update/2024/04/valkey-7-2-5-out/)，[Valkey](https://valkey.io/blog/update/2024/04/valkey-7-2-5-out/) [7.2.5 GA](https://valkey.io/blog/update/2024/04/valkey-7-2-5-out/)。此版本基于Redis最后的开源版本7.2.4。这是个好消息，但不是大新闻。这是我们从分叉项目期待的。

**相反，重要的新闻是对Valkey的即时支持。**

AWS、Ericsson、Google Cloud、Oracle和Verizon都表示他们支持这一分支。现在阿里云、Aiven、Heroku和Percona也在支持Valkey。我见过许多分叉项目起飞并超越其父项目——LibreOffice超过OpenOffice就是一个例子——但我从未见过有这么多财富500强企业因为另一个项目而转身放弃了一个项目。

奥尔森在她的主题演讲中表示：“这些公司计划继续或甚至增加他们的贡献。长期健康和项目可行性需要他们的支持。社区感谢他们的支持，这将帮助项目保持开放并使我们的开发人员保持用户需求的步伐。”

Valkey分支的维护者具备满足业务需求所需的技术实力。正如奥尔森所指出的那样，其技术领导委员会拥有26年的经验，并且在开源项目中贡献了超过1,000次提交。事实上，腾讯的[朱斌斌](https://lwn.net/Articles/966631/)，作为委员会成员，单独贡献了几乎四分之一的开源Redis提交。

**这个回应不是Redis原本期望的。**

在分叉时，Redis CEO Rowan Trollope 似乎并不太担心。“所有主要的云服务提供商都从 Redis 开源项目中获得了商业利益，所以他们在基金会内启动分叉并不奇怪。” 他写道。他补充道：“创新一直是 Redis 成功与任何替代方案之间的差异化因素。”

现在有这么多顶级维护者在 Valkey，这种说法可能很难坚持下去。正如 Heroku 的首席技术官 Gail Frederick 在会议上所说：“有了这些经验丰富的贡献者和广泛的行业支持，Valkey 正在继续开源模式，这种模式已经对全球开发者产生了变革性影响。”

这么多顶级 Redis（软件）的客户转向 Valkey 给 Redis（公司）带来了严峻的挑战。毕竟，[Redis 曾是 AWS 上最受欢迎的数据库](https://www.theregister.com/2020/11/23/redis_the_most_popular_db_on_aws/)，根据 DB-Engine 的最新统计，[它是最受欢迎的键值型 DBMS](https://db-engines.com/en/ranking)。

**看起来许多这些客户现在正在前往 Valkey。**

AWS 高级开发者倡导者兼前 Redis 员工 Kyle Davis 表示：“[AWS 致力于长期支持开源](https://aws.amazon.com/blogs/opensource/why-aws-supports-valkey/)，并将在我们基于开源 Redis 构建的 ElastiCache 和 MemoryDB 管理数据库服务中添加 Valkey 支持。”

Oracle 的数据和 AI 产品管理总监 Karan Singh 写道：“Oracle 云基础设施（OCI）缓存将继续支持其当前基于 Redis 7.0 的服务，因为许可证变更对它没有影响。” 不过，Oracle 正在从其服务中删除 Redis 的名称，并且正在 “[工作中整合](https://blogs.oracle.com/cloud-infrastructure/post/oracle-supports-valkey) [Valkey](https://blogs.oracle.com/cloud-infrastructure/post/oracle-supports-valkey) [支持到 OCI 缓存服务](https://blogs.oracle.com/cloud-infrastructure/post/oracle-supports-valkey) 以确保我们在 OCI 上的独特内存数据库产品继续从开源进步中受益，增强客户价值。”

实证数据表明用户也在逐渐远离 Redis。Percona CEO Peter Zaitsev 指出，[DB-Engine 已经显示出对 Redis 兴趣的下降](https://twitter.com/PeterZaitsev/status/1781104932302758170%20target=)。

Redis 的这一举动引起了很多嘲讽。例如，The Duckbill Group 的首席云经济学家 Corey Quinn 在推特上写道：“早上好，除了 [Redis 的小偷们](https://twitter.com/QuinnyPig/status/1778484953661194524)。你们没有创造它；你们从社区偷走了它。如果你们想要它的话，找建造它的那些人，那就是 Valkey。”

这里我们看到的不是开源爱好者在耍酷，而是公司和开发者们一起大举远离 Redis。

*照片来源：[Happy Lee](https://unsplash.com/@happy_lee?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash)，来自[Unsplash](https://unsplash.com/photos/a-sign-post-with-many-different-colored-signs-on-it-bOkC_HLLeGs?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash)*
