<!--yml

category: 未分类

date: 2024-05-27 12:48:30

-->

# Redis分支之战？

> 来源：[https://www.thestack.technology/battle-of-the-redis-forks-begins/](https://www.thestack.technology/battle-of-the-redis-forks-begins/)

*查看我们3月28日发布的关于“Valkey”的更新故事* [*此处*](https://www.thestack.technology/redis-fork-valkey-linux-foundation/)*。*

Redis在上周抛弃了开源许可证，给其免费核心产品的贡献者社区带来了混乱。（它正在将BSD许可证换成更为严格的双重许可证，Redis源代码可用许可证和MongoDB的服务器端公共许可证。）

项目不到一天时间就有了多个新的分支。能否像Terraform社区在Hashicorp于2023年8月10日做出“毒丸”决定后围绕OpenTofu汇集部队一样，围绕一个分支集结起来，这是一个未知数。

但是，AWS的Madelyn Olson率先出发。与阿里云的[Zhou Zhou](https://www.alibabacloud.com/blog/redis-core-team-welcomes-zhao-zhao-from-alibaba-cloud-to-run-the-redis-project_596709?ref=thestack.technology)和其他几位现有的核心Redis维护者一起，在上周五[启动了一个分支](https://github.com/placeholderkv/placeholderkv?ref=thestack.technology)，她说：“我们对许可证的更改感到不满，并希望建立一个新的真正开放的社区来填补Redis留下的空白…”

“在Redis改变许可证之前，我们有一个非常激动人心的社区路线图。包括集群改进、性能提升、触发器等等。我们将继续推动这一进程，”Olson在[X](https://twitter.com/reconditerose/status/1771237000592642274?ref=thestack.technology)上发帖。

（Redis已经被全球项目和基础设施深度嵌入，并已被下载超过10亿次，被数百万开发者用作[内存数据存储](https://redis.io/docs/get-started/data-store/?ref=thestack.technology)，作为[缓存](https://redis.io/docs/manual/client-side-caching/?ref=thestack.technology)，以及[文档数据库](https://redis.io/docs/get-started/document-database/?ref=thestack.technology)等。）

那么，这是一个由AWS支持的“官方”Redis分支吗？（Olson在AWS工作，负责Redis已有六年。）“不是。AWS雇佣了我，但这只是我试图与社区保持连续性。AWS知道我在做什么，并正在准备他们自己的回应。”Olson发帖道。

（当Elasticsearch在2021年[改变](https://www.elastic.co/pricing/faq/licensing?ref=thestack.technology)其开源许可证时，AWS在一次举措中推出了Apache 2.0许可的分支Opensearch，取代了原来的开源“采矿者”，在*某些*人眼中，它变成了社区的倡导者…）

软件开发者Drew DeVault（是[SourceHut](https://sourcehut.org/?ref=thestack.technology)、[wlroots](https://gitlab.freedesktop.org/wlroots?ref=thestack.technology)以及其他几个项目的幕后黑客，同时也是一位多产的FOSS博客作者）也迅速分叉了Redis，推出了“Redict”，他将其描述为“Redis OSS 7.2.4的独立、非商业分支，基于Redis OSS的BSD 3-Clause源代码… 从此处开始的所有更改均受Lesser GNU General Public License，LGPL-3.0-only的许可证保护。” DeVault在博客中补充道，“围绕…

+   “利用这个机会移除一些长期弃用的功能，比如“redis-trib”

+   消除销售依赖并转向上游Lua，jemalloc

+   为了更具下游独立性，移除例如示例systemd或upstart服务

+   我们还将分叉[Hiredis](https://github.com/redis/hiredis?ref=thestack.technology)，因为它是Redict的内部依赖。

DeVault说，该项目将“用于建立一个独立于专有基础设施如GitHub和Slack的社区。

“源代码托管在[Codeberg](https://codeberg.org/redict/redict?ref=thestack.technology)，这是一个由德国非营利组织运营的Forgejo实例，应该为熟悉GitHub社区的任何人提供舒适和熟悉的用户体验…”

（多年来，软件自由保护协会一直在支持“[放弃GitHub](https://sfconservancy.org/GiveUpGitHub/?ref=thestack.technology)”的努力，指出专有软件锁定的风险，微软甚至对复制左许可证项目进行代码挖掘以训练其Copilot AI产品等问题。）

AWS的Olson在X上发布说，Redict的[愿景](https://redict.io/posts/2024-03-22-redict-is-an-independent-fork/?ref=thestack.technology)，[](https://t.co/IqHFvdG8y7?ref=thestack.technology)“最终与我的期望不是特别一致。我希望继续大力投资于新功能，就像我们在许可证更改之前所做的那样，他们似乎更倾向于快速构建一个基本上是FOSS项目的构想” – 但稍后在X上补充了第二个笔记说“我完全支持redict并在帮助他们解决问题上保持一致。我与创作者保持联系，希望我们能继续合作…”

目前显然还处于早期阶段，社区变化不小，但看起来可能会出现一个严肃的Redis分支。如果AWS和阿里云支持那些对这个受欢迎项目做出重要贡献的员工（更不用说其他潜在的赞助商了），它可能会迅速腾飞。敬请关注……
