<!--yml

类别：未分类

日期：2024-05-27 13:31:29

-->

# [企业开源已死 | Jeff Geerling](https://www.jeffgeerling.com/blog/2024/corporate-open-source-dead)

> 来源：[https://www.jeffgeerling.com/blog/2024/corporate-open-source-dead](https://www.jeffgeerling.com/blog/2024/corporate-open-source-dead)

IBM正在[以64亿美元购买HashiCorp](https://www.reuters.com/markets/deals/ibm-buy-hashicorp-64-billion-deal-expand-cloud-software-2024-04-24/)。

这是在[HashiCorp摘取他们的*整个*开发社区](https://www.hashicorp.com/blog/hashicorp-adopts-business-source-license)并抛弃开源转向“商业源代码许可证”四个月后。 

正如[Hacker News上的某人](https://news.ycombinator.com/item?id=40135490)如此雄辩地指出：

> IBM就像是一个榨取水果所有美味的榨汁机

skywhopper回复说：

> HashiCorp已经很好地预先排干了它曾经拥有的任何风味。

[有人怀疑](https://twitter.com/jgunnink/status/1783303926042996928)HashiCorp放弃开源的决定是因为他们想要为更高的价格榨取利润。我是说，*六十亿美元？* 他们甚至不是一个毫无意义的AI公司！

> 这篇博文是我今天发布的视频的转录，[企业开源已死](https://www.youtube.com/watch?v=hNcBk6cwim8)。 您可以在YouTube上观看。

与此同时，[Redis放弃了开放BSD许可证](https://redis.io/blog/redis-adopts-dual-source-available-licensing/)并发明了自己的“源代码可用”许可证。

去年，我报道了[Red Hat找到了一种只是*勉强*遵守](https://www.jeffgeerling.com/blog/2023/im-done-red-hat-enterprise-linux)他们的企业Linux发行版开源GPL许可证的方法。

其他公司如MongoDB、Cockroach Labs、Confluent、Elasticsearch和Sentry也 [也成为了“源代码可获取”](https://thenewstack.io/hashicorp-abandons-open-source-for-business-source-license/)。这从一些较小的参与者开始，但随着甚至是最大的“开源”公司的堕落，开源开发人员正在选择核心选项。

当公司抛售时？ 分叉它们。 字面上！

Terraform，HashiCorp的支柱，被[fork到OpenTofu](https://opentofu.org/blog/the-opentofu-fork-is-now-available/)，并由Linux基金会采纳。 依靠Terraform建立业务的公司迅速转向。 更令人兴奋的是，HashiCorp的另一个大项目Vault的分支OpenBao是[由IBM支持](https://www.techtarget.com/searchitoperations/news/366563095/IBM-engineers-hatch-Linux-Foundation-HashiCorp-Vault-fork)! *那*个分支现在会发生什么？

在Hashi-land至少分支似乎相当简单。 在Redis肆意破坏之后，似乎每周都有一个[新的分支](https://arstechnica.com/information-technology/2024/04/redis-license-change-and-forking-are-a-mess-that-everybody-can-feel-bad-about/)！

一些开发者甚至正在考虑完全放弃 Redis 代码，就像 [redka](https://github.com/nalgeon/redka) 在 SQLite 之上的一个兼容 API 封装！

在 Red Hat 几乎关闭其门店之后，至少他们没有试图在许可证本身上搞花样！Oracle、SUSE 和 CIQ [组建了 OpenELA 联盟](https://www.suse.com/news/OpenELA-for-a-Collaborative-and-Open-Future/) 来维护 Enterprise Linux 的分支。而 CentOS 用户则不得不在 7 月结束 CentOS 7 支持之际，决定是否使用 AlmaLinux 或者现在的 ELA 项目之一。

所有这些举动打破了初创公司和巨头公司用来在过去十年内积累数十亿美元收入的游戏规则。

这一切都是以“开源”为名。

随着免费资金枯竭和利润减少，公司们几乎和社区信任一样迅速[裁员](https://www.cnbc.com/2024/03/12/ibm-tells-employees-of-job-cuts-in-marketing-and-communications.html)。

## 2024 年是企业开源的消亡之年。

2024 年是企业开源——或者至少关于它的任何幻想——最终消亡的一年。

用专有代码库构建产品并收费许可是一回事。你仍然可以围绕这种模式建立社区，这种模式已经运作了几十年。

但当你在开源许可证下构建产品时，培养了一群用户社区，他们又在该软件上建立了自己的业务，然后在收入受到影响时收回许可证，这就完全不同了。

这就是所谓的欺骗性广告。

Bryan Cantrill 多年来一直在敲警钟 —— 是的，*那个 Bryan Cantrill*，发布了这篇珍贵的文章：

[Brian 的演示](https://www.youtube.com/watch?v=-zRN7XLCRhc)，12 年前的，值得一看，底线由 [Drew DeVault](https://drewdevault.com/2023/07/04/Dont-sign-a-CLA-2.html) 总结：

> [贡献者许可协议（CLA）是] 由商业公司采用的一种策略，只有一个目的：在项目下放置地毯，以便在出现不利季度的第一个迹象时撤走。这种策略存在是为了颠覆开源社会契约。

如果你参与签署了 CLA 的项目，在那里你放弃了你的代码，你就是在给公司全权剥夺你使用他们软件的自由。

从公司的角度来看，如果他们想要 CLA 或者想要使用反开源许可证，他们并*不*关心你的自由。他们在保护收入流。他们经常会谈论免费乘客，无论是亚马逊建立竞争性托管解决方案，还是一些找到赚钱方式的初创企业。

但最终，即使你拥有 GPL 代码并向人们收费，如果公司限制你使用、修改和分享代码的方式，它并不是真正的自由。

但这里有一个区别，我知道一些观众已经在责备我。有“自由”软件，也有“开源”软件。

自由软件社区的人们[正确地认识到了危险](https://www.gnu.org/philosophy/open-source-misses-the-point.html)，称自由软件为“开源”。

我认为我们不必对此如此教条，但是自由软件社区与更商业化的“开源”文化之间确实存在*哲学*上的根本差异，比如自由软件基金会和软件自由保护协会等组织支持的差异。

开源文化依赖于信任。信任那些*你我帮助建立的*公司（即使没有在薪水单上）不会拉毛。

但一次又一次，这种信任都被破坏了。

公司开源的这种缓慢衰落是*糟糕的*吗？嗯，这确实令人讨厌，特别是对于像我这样过去与这些社区有关联的开发者。但并非*全*都糟糕。

## 为什么企业开源的衰落并不是坏事

实际上，这可能是一个巨大的机会；像Ansible、HashiCorp、Elasticsearch或Redis这样的新兴企业发展迅速，点燃了它们所在行业。

构建开发者社区，跨越文化和经济障碍，创造改变世界的软件，这是发生了什么？

还有一些项目在做这件事，但是许多项目都屈服于企业金钱，巨额收入将利润置于哲学之上。

但随着资金枯竭，过去五年疯狂的招聘趋势之后越来越多的开发者被裁员，也许小型开发团队可以改变局势。

AI泡沫还没有破灭，所以一些优秀的人被吸入了那个漩涡。

但其他人可能正处于下一个伟大开源项目的边缘。只是...不要添加CLA，好吗？

这不仅仅是开发者的问题；大公司也可以加入其中。像微软和*甚至甲骨文*这样历史上不良的玩家——哎，说出来真是令人痛苦。他们在过去的十年中甚至取得了一些进展！

IBM甚至可以修复一些伤口，比如他们可以重新联合OpenTofu和Terraform。这有先例，就像2015年[IO.js合并回Node.js](https://thenewstack.io/io-js-and-node-js-have-united-and-thats-a-good-thing/)时那样。

人们问红帽公司能做些什么来重新引起我对企业版 Linux 的兴趣。很简单：别再像垃圾一样对待那些对桌面无收益的人。免费加载者是开源的一部分——无论是在家庭实验室还是竞争企业中。

希望与开源开发者交朋友的公司需要表现出他们关心的不仅仅是钱。不幸的是，当前的趋势是拉毛来提升季度收益，因为金钱总是要增加的！

但你知道吗？我更喜欢诚实。如果收入如此依赖销售软件，就...让软件成为专有的。不要那么含糊！

对于不是多亿美元公司的任何人，不要成为下一个拉毛的受害者。警告信号很明显：不要签署CLA。远离需要CLA的项目。

坚持使用尊重你的自由的开源许可证，而不是为了增加收入和为公司做好十亿美元收购准备的许可证。

或许现在是开启一场新的开源叛乱的时候了。也许这一次，金钱*不会*改变公司文化，当新项目从灰堆中涌现时。也许不会，但至少我们可以尝试。
