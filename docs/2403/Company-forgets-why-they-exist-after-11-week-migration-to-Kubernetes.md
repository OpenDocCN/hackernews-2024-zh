<!--yml

category: 未分类

date: 2024-05-27 14:30:20

-->

# 公司在将其基础设施迁移到 Kubernetes 后，11 周后忘记了他们存在的原因。

> 来源：[https://www.theolognion.com/p/company-forgets-why-they-exist-after-11-week-migration-to-kubernetes](https://www.theolognion.com/p/company-forgets-why-they-exist-after-11-week-migration-to-kubernetes)

一家新兴的硅谷初创企业 Xenobroom Inc. 于 2020 年 5 月开始了一项长期的服务器基础设施升级过程。根据 CEO 每日日记和 CTO 的工程记录残片显示，在全球大流行期间，公司的日常使用量急剧上升。不久之后，决定将现有基础设施迁移到 Kubernetes。

这一过程比预期的要长：简单的 bash 脚本和单纯的 VPS 机器需要进行彻底改造、重新考虑和重新设计。许多公司内部认为这是一个良好的机会，可以升级软件依赖和库。在单机上运行的大部分 PostgreSQL 数据库可以转变为分布式 KV 存储，充分利用 AWS 的极大灵活性。仅在 'develop' 分支上每日部署的普通演练服务器可以被放弃，采用只有生产环境的 CI 启用工作流，并带有动态路由，允许无缝进行 A/B 测试和地理依赖。

当看似完成这一过程时，公司内无人记得他们产品的目的。

不幸的是，无论是用户还是众多投资者，都不能在这种情况下提供帮助。这两组人都公开承认他们一开始就无法完全理解这个产品，而今，在数周的停机后，几乎不可能恢复其意义。

现在，CEO 寻求 Phutar Afrayughum 的帮助，他是一位据称帮助 Google 在消息应用市场上增加市场份额的心灵感应专家和超感知专家，并且还参与了 Material Design 框架的开发。
