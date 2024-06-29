<!--yml

category: 未分类

date: 2024-05-29 12:48:56

-->

# 大牌支持 Redis 的分支

> 来源：[https://www.runtime.news/big-money-lines-up-behind-a-redis-fork/](https://www.runtime.news/big-money-lines-up-behind-a-redis-fork/)

[Newsletter](/tag/newsletter/)

# 大牌支持 Redis 的分支

今天的主题是：为什么企业技术界的一些大牌会支持 Redis 的开源分支，Databricks 加入开放 LLM 游戏，以及企业技术的最新动向。

*欢迎来到 Runtime！今天的主题是：为什么企业技术界的一些大牌会支持 Redis 的开源分支，Databricks 加入开放 LLM 游戏，以及企业技术的最新动向。*

*(本邮件是否转发给您？* [*点击此处注册*](https://www.runtime.news/#/portal/signup) *以获取 Runtime 的每周内容。)*

* * *

## Valkey 获得支持

上周，在 Redis 成为最新一个改变其流行开源项目许可条款的企业技术公司后，一些开发者迅速宣布计划创建一个可继续使用的允许许可项目的分支。本周，该团队得到了一些财力雄厚的支持者的大力支持。

Linux Foundation 宣布将主办 Valkey，"一个 Redis 内存型 NoSQL 数据存储的开源替代方案"，[在新闻稿中如此声明](https://www.linuxfoundation.org/press/linux-foundation-launches-open-source-valkey-community?ref=runtime.news)。由 AWS 的 Madelyn Olson 领导，Valkey（["就像 Walkyrie 一样"](https://github.com/valkey-io/valkey?ref=runtime.news)）将保持 BSD 3-clause 许可证，这与 Redis（公司）计划通过发布新版本的 Redis（项目）而放弃的源可用许可证不同。

+   过去十年中，开源 Redis 项目广泛被采用，作为一种快速数据缓存，可以用于现有数据库的前端或作为数据库本身。

+   Redis（公司）已经筹集了[3.56 亿美元](https://www.crunchbase.com/organization/redis?ref=runtime.news)用于围绕该开源项目开发商业产品，并广泛被认为正准备在不久的将来上市。

+   然而，引用“一系列独特的挑战”，[Redis CEO Rowan Trollope 上周表示](https://redis.com/blog/redis-adopts-dual-source-available-licensing/?ref=runtime.news)，公司将不再允许云服务提供商[无需支付许可费用](https://www.runtime.news/open-source-was-a-zirp-marketing-funnel/)就能基于 Redis 项目开发自己的商业服务。

+   Microsoft 是第一个这样做的，但可能也是最后一个。

**这是因为 AWS、Google Cloud、Oracle、Ericsson 和 Snap** 宣布他们将投入时间和资金支持 Valkey，这是 Redis 项目的开源竞争对手。

+   根据 Linux 基金会的说法，“社区将继续按照其现有路线图开发新功能，包括更可靠的插槽迁移，聚类系统的显著可扩展性和稳定性改进，多线程性能改进，触发器，新命令，向量搜索支持等等。”

+   “我们相信，为那些更倾向于开源的人提供一个保持良好维护和创新的选择将有助于社区的发展，”谷歌云数据库总经理兼副总裁 Andi Gutmans 在声明中表示。

+   Valkey 将允许使用托管版 Redis（例如 [Amazon ElastiCache for Redis](https://aws.amazon.com/elasticache/redis/?ref=runtime.news)）的云客户像往常一样继续使用，同时为云提供商保留了一个有利可图的服务。

**Linux 基金会还支持 OpenTofu**，这是 HashiCorp 的 Terraform 项目的一个分支，是对其去年决定限制其他人如何使用 Terraform 的响应。但这种感觉有所不同。

+   如 [Redmonk 的 Stephen O'Grady 所说](https://twitter.com/sogrady/status/1773405238826836410?s=58&t=vcy_ZBBRCda17yxCrwL9Ew&ref=runtime.news)，“通常情况下，对主要分支保持保守态度，因为它们难以实现，因此通常不太可能成功。然而，这次的名单设置使这一努力与大多数分支不同。”

+   由基金会主导的开源项目存在其缺点 —— 主要是委员会决策问题 —— 但它们承诺稳定性和可预测性，而这正是企业技术用户最终想要的。

+   这些基金会拥有大量资源；作为 Linux 基金会的铂金会员和 Redis 许可证持有者，微软将实际上支持 Redis 和 Valkey 的开发。

**Valkey 要产生影响可能需要几个月甚至几年的时间**，但许多初创公司、投资者和大型云公司将密切关注其进展。

+   由于盈利性已经比增长更为重要，围绕开源项目构建的风险投资公司面临着越来越大的压力，必须最大化收入，而一些东西必须让步。

+   但是 Redis 放弃其开源许可策略，希望迫使云提供商为其技术付费的决定是否会适得其反？

* * *

## 谈到开源

Databricks 长期以来一直是生成式 AI 战斗机的幕后引擎之一，但本周它直接参与了这场争斗。周三 [发布了 DBRX](https://www.databricks.com/blog/announcing-dbrx-new-standard-efficient-open-source-customizable-llms?ref=runtime.news)，这是一个开放的大型语言模型，与 Meta 的 Llama 和 OpenAI 的 GPT-3.5 等其他开放模型以及更早的闭源模型相比具有优势。

DBRX是由Databricks去年以13亿美元收购的MosaicML团队构建的，并且[该团队非常谨慎地将DBRX描述为“开放模型”](https://www.databricks.com/blog/introducing-dbrx-new-state-art-open-llm?ref=runtime.news)，因为[Databricks将要求月活跃用户超过7亿的公司](https://github.com/databricks/dbrx/blob/main/LICENSE?ref=runtime.news)购买使用DBRX的许可证，并禁止任何人将其用于改进与DBRX竞争的LLM。然而，[Databricks的企业博客](https://www.databricks.com/blog/announcing-dbrx-new-standard-efficient-open-source-customizable-llms?ref=runtime.news)，[Wired的](https://www.wired.com/story/dbrx-inside-the-creation-of-the-worlds-most-powerful-open-source-ai-model/?ref=runtime.news)兴奋和独家的幕后报道，以及[无数其他报道](https://www.google.com/search?client=firefox-b-1-d&sca_esv=607135a9cfeed7f2&q=dbrx+open+source&tbm=nws&source=lnms&prmd=ivnsmbtz&sa=X&ved=2ahUKEwixl7i8gZiFAxXBOjQIHUtYBNgQ0pQJegQIDRAB&biw=1920&bih=870&dpr=2)都将DBRX描述为“开源”，而这得益于那些限制。

毫无疑问，像DBRX和Llama这样的模型比GPT家族更开放，但AI社区却在匆忙中践踏[“开源”的既定定义](https://opensource.org/licenses?ref=runtime.news)，试图与技术社区中仍然带有的良好意愿联系起来。你本以为开发预测文本引擎的公司会对语言的精确性更加尊重一些，但现在我们却发现不然。

* * *

## 企业动态

**马特·雷纳**是[Google Cloud全球领域组织的新总裁](https://www.silverliningsinfo.com/apps-services/google-clouds-new-field-org-chief-familiar-face?ref=runtime.news)，在加入公司不到一年后接任了这一高级职位，并在Adaire Fox-Martin离职去Equinix后接替了她。

**史蒂文·钟**是[Starburst的新总裁](https://www.prnewswire.com/news-releases/technology-veterans-join-starburst-to-transform-the-data-warehouse-industry-302099507.html?ref=runtime.news)，在担任Delphix总裁之后，加入了这家数据湖公司。

**塔玛尔·叶霍舒亚**是[Glean产品和技术新总裁](https://www.businesswire.com/news/home/20240326392278/en/?ref=runtime.news)，在担任Slack首席产品官四年后，加入了这家公司。

**瑞克·斯克菲尔德**是[VAST Data的新首席营收官](https://vastdata.com/press-releases/vast-data-appoints-rick-scurfield-to-executive-leadership-team-as-chief?ref=runtime.news)，此前在NetApp的各种销售领导职务上工作了20年。

**丹尼斯·戴曼**成为[Code42的新任首席信息安全官](https://www.code42.com/news/code42-appoints-dennis-dayman-as-chief-information-security-officer/?ref=runtime.news)，此前他曾担任Proofpoint的CISO。

**古德普·迪隆**成为[Contentstack的新任首席营销官](https://www.prnewswire.com/news-releases/contentstack-appoints-former-adobe-executive-gurdeep-dhillon-as-chief-marketing-officer-302099173.html?ref=runtime.news)，此前他在Zuora担任类似角色多年。

* * *

## 运行时间总结

**亚马逊支付剩余的27.5亿美元**，[**这是预计将投资于Anthropic的金额**](https://www.cnbc.com/2024/03/27/amazon-spends-2point7b-on-startup-anthropic-in-largest-venture-investment.html?ref=runtime.news)，去年宣布计划最多投资40亿美元在这家基金会模型公司。

**NIST出现了一些奇怪的情况**，[其关键软件漏洞数据库更新落后](https://www.axios.com/2024/03/26/nist-cyber-vulnerabilities-database?ref=runtime.news)，安全专家依赖该数据库来修补系统。

**微软在Azure AI Studio引入了新的安全措施**，以[防止提示注入](https://www.bloomberg.com/news/articles/2024-03-28/microsoft-creates-tools-to-stop-people-from-tricking-chatbots?ref=runtime.news)，这可能劫持LLM以生成奇怪的输出或访问未经授权的系统。

**沃尔玛的技术基础设施团队正在忙于**处理一系列导致停机的“重大事故”，这些事件影响了收入，[据彭博社报道](https://www.businessinsider.com/walmart-tech-outages-leaked-emails-2024-3?ref=runtime.news)。

**华盛顿大学也在继续努力应对多年来的Workday实施问题**，这导致[数百万美元研究资助的延迟发放](https://www.seattletimes.com/business/uws-340-million-finance-upgrade-is-still-struggling-despite-progress/?ref=runtime.news)。

* * *

*感谢您的阅读 — 运行时间将在假期周末休息，周二见！*
