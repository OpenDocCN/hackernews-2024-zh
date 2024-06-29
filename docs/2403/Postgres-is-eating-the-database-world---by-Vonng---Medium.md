<!--yml

category: 未分类

date: 2024-05-27 14:59:54

-->

# Postgres is eating the database world | by Vonng | Medium

> 来源：[https://medium.com/@fengruohang/postgres-is-eating-the-database-world-157c204dcfc4](https://medium.com/@fengruohang/postgres-is-eating-the-database-world-157c204dcfc4)

# DB 领域的游戏规则改变者

**PostgreSQL 的出现改变了数据库领域的范式**：现在，试图打造“新数据库内核”的团队面临一个艰巨的试炼 — 如何在面对开源、功能丰富的 Postgres 时脱颖而出。他们有什么独特的价值主张呢？

在没有发生革命性硬件突破之前，实用、新型通用数据库内核的出现似乎不太可能。没有哪个单一的数据库能够匹敌 PG 的整体实力，尽管有其所有扩展支持 — 即使是 Oracle 也无法，鉴于 PG 是开源且免费的大王牌 ;-)

如果一个小众数据库产品能在特定方面（通常是性能）的某些方面超过 PostgreSQL 十倍，它可能会开辟自己的空间。然而，通常不久之后，PostgreSQL 生态系统就会衍生出开源扩展的替代方案。选择开发 PG 扩展而不是全新数据库，可以让团队在追赶时具备压倒性的速度优势！

根据这一逻辑，PostgreSQL 生态系统正处于雪球效应的状态，积累了优势，并不可避免地朝着垄断的方向发展，这与 Linux 内核在服务器操作系统中的地位几年内非常相似。开发者调查和数据库趋势报告证实了这一轨迹。

[***StackOverflow 2023 Survey: PostgreSQL, the Decathlete***](https://survey.stackoverflow.co/2023/#section-most-popular-technologies-databases)

[***StackOverflow 过去7年的数据库趋势***](https://demo.pigsty.cc/d/sf-survey)

PostgreSQL 长期以来一直是 HackerNews 和 StackOverflow 中最受欢迎的数据库。许多新的开源项目将 PostgreSQL 设为其主要，甚至唯一的数据库选择。许多新一代公司全面采用 PostgreSQL。

正如《“**彻底简化：只用 Postgres**”](https://www.amazingcto.com/postgres-for-everything/) 所说，通过只使用 Postgres，可以简化技术栈，减少组件，加速开发，降低风险，并增加更多功能。Postgres 可以轻松替代许多后端技术，包括 MySQL、Kafka、RabbitMQ、ElasticSearch、Mongo 和 Redis。**只用 Postgres** 不再仅限于少数精英团队，而是正在成为主流最佳实践。

# 还能做什么？

数据库领域的终局似乎是可以预见的。但我们能做什么，应该做什么？

对于绝大多数场景来说，PostgreSQL 已经是接近完美的数据库内核，因此称其为内核“瓶颈”的想法是荒谬的。那些吹嘘内核修改为卖点的 PostgreSQL 和 MySQL 的分支基本上毫无前途。

这类似于当前Linux操作系统内核的情况；尽管有众多Linux发行版，但每个人都选择同一个内核。分叉Linux内核被视为制造不必要的困难，行业对此表示反感。

因此，主要冲突不再是数据库核心本身，而是两个方向 — — 数据库**扩展**和**服务**！前者涉及内部可扩展性，而后者涉及外部组合性。就像操作系统生态系统一样，竞争格局将集中在**数据库分发**上。在数据库领域，只有围绕扩展和服务的分发才有可能最终成功。

核心仍保持温和，与MySQL母公司的分支MariaDB接近摘牌，而AWS凭借在免费核心上提供服务和扩展而蓬勃发展。投资流向了众多PG生态系统的扩展和服务分发：Citus、TimescaleDB、Hydra、PostgresML、ParadeDB、FerretDB、StackGres、Aiven、Neon、Supabase、Tembo、PostgresAI，以及我们自己的PG分发 — — [Pigsty](https://pigsty.io/)。
