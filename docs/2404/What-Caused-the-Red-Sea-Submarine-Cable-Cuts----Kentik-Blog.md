<!--yml

category: 未分类

date: 2024-05-27 12:48:19

-->

# 红海海底光缆被切断的原因是什么？ | Kentik 博客

> 来源：[https://www.kentik.com/blog/what-caused-the-red-sea-submarine-cable-cuts/](https://www.kentik.com/blog/what-caused-the-red-sea-submarine-cable-cuts/)

在最新一次地缘政治与物理互联网的碰撞中，上个月红海发生了三条主要海底光缆被切断的事件，很可能是也门胡塞武装分子袭击过往商船的结果。在这篇文章中，我们将回顾这一情况，并深入探讨海底光缆切断的一些可观察影响。

* * *

2月24日，红海发生了三条海底光缆被切断的事件（Seacom/TGN-EA，EIG和AAE-1），导致从东非到东南亚的服务提供商的互联网流量受到干扰。由于多种因素的影响，红海多年来一直是海底光缆的危险区域，但随着敌对方向对附近航行船只发射导弹，最近情况变得更加严峻。

在本文中，我们将回顾红海独特情况的背景，并详细探讨海底光缆切断的互联网影响和时间安排。本次工作是与《WIRED》杂志合作完成的，他们同时发布了关于光缆切断调查的[报道](https://www.wired.com/story/houthi-internet-cables-ship-anchor-path)。

作为对加沙战争的回应，由胡塞武装控制的也门政府开始向通过附近[巴巴尔曼德卜海峡](https://en.wikipedia.org/wiki/Bab-el-Mandeb)的船只发射导弹和武装无人机，认为这些船只与以色列、英国或美国有某种关联。

去年十一月，从也门发射的[导弹](https://www.timesofisrael.com/missile-from-yemen-intercepted-over-red-sea-as-houthi-chief-vows-to-keep-up-attacks/)袭击了以色列在红海的重要航运港口埃拉特。随后不久，也门电信遭遇了[多小时的停运](https://twitter.com/DougMadory/status/1723016723459125292)，引发了对因导弹袭击而导致互联网中断的猜测。我在[一篇博客文章的结尾部分](/blog/cloud-observer-subsea-cable-maintenance-impacts-cloud-connectivity/)中描述了这一事件，呼吁在海底光缆行业内增加透明度，因为根据光缆所有者GCX的说法，也门电信的中断是由计划维护造成的。

去年十二月底，一篇发表在胡塞武装盟友的Telegram频道上的帖子暗示海底光缆也可能成为他们对加沙进行报复性袭击的目标之一。然而，在媒体广泛报道此威胁后，由胡塞武装控制的也门电信部门发布了一份声明，否认有针对海底光缆的任何攻击行动：[发布声明](https://twitter.com/mtityemen/status/1739659835908432341)。

“（电信部）对社交媒体和其他媒体发表的所谓针对红海巴勒曼德布穿越的海底电缆的威胁声明不属实。”

BGP路由的终极指南

有效的BGP配置对于控制您组织在互联网上的命运至关重要。学习BGP的基础知识和演变。

2022年6月，我[发表了一篇博客文章](/blog/outage-in-egypt-impacted-aws-gcp-and-azure-interregional-connectivity/)，讨论埃及在全球互联网通信中的瓶颈角色。然而，这个瓶颈延伸到毗邻的红海，其本身也存在一系列风险。

在红海等待通过苏伊士运河的货船将在海湾相对较浅的深度处下锚。锚链可能缠绕一个或多个海底电缆的可能性非常真实，并且过去已经发生过。

2012年2月，由于一艘船拖动锚链，[三条海底电缆在红海被切断](https://www.dailymail.co.uk/news/article-2108868/Ships-anchor-accidentally-slices-internet-cable-cutting-access-African-countries.html)。当时，我在[这里](https://web.archive.org/web/20140521135653/http://www.renesys.com/2012/02/east-african-cable-breaks/)写道，关注东非互联网连通性受到的影响，重点是此前这种情况可能导致整个地区通信中断，而现在却幸存下来。

2月24日星期六早晨，三条海底电缆发生切断事件：[EIG](https://www.submarinecablemap.com/submarine-cable/europe-india-gateway-eig)、[Seacom/TGN-EA](https://www.submarinecablemap.com/submarine-cable/seacomtata-tgn-eurasia)和[AAE-1](https://www.submarinecablemap.com/submarine-cable/asia-africa-europe-1-aae-1)。据行业消息来源称，EIG在此前的12月初已经因一次切断而处于停运状态，因此第二次切断对互联网通信的运营影响较小。

Seacom非常值得称赞，通过立即[确认损坏](https://www.itweb.co.za/article/seacom-confirms-cable-outage-in-red-sea/KPNG8v8NyDNM4mwD)了他们的电缆损坏继续保持了作为世界上最开放和沟通最多的海底电缆之一的实践。EIG和AAE-1没有发布类似的内容。

最初对切断原因的推测集中在去年12月电报频道发布的针对海底电缆的威胁。也未解释也门如何实施这样的海底攻击：水下爆炸物、带切割装备的潜水员、潜水器？

不久之后，一个更为真实的理论出现在了潜水电缆行业中。在三根海底电缆发生故障几天前，一艘注册在伯利兹、由英国所有的货船[遭到了来自也门的导弹袭击](https://www.centcom.mil/MEDIA/PRESS-RELEASES/Press-Release-View/Article/3680410/feb-18-summary-of-red-sea-activities/)。船员抛锚并且放弃了失事的船只MV Rubymar。之后，Rubymar开始漂流，拖曳其锚——根据[国际电缆保护委员会](https://www.iscpc.org/)的说法，这是潜水电缆切断的主要原因之一。在3月2日，这艘废弃的船最终[沉没](https://apnews.com/article/yemen-houthi-rebels-rubymar-sinks-red-sea-fb64a490ce935756337ee3606e15d093)，带走了超过41,000吨肥料。

尽管尚未确认，Rubymar的拖曳锚链仍然是二月24日海底电缆切断的主要理论。对于也门电信部门来说，他们[发布了一份声明](https://twitter.com/mtityemen/status/1762453092442706385)否认了参与电缆切断的指控。

正如之前提到的，EIG已经停运，因此我们不期待在互联网测量数据中看到其丢失的影响，但Seacom和AAE-1的损失是可观的。事实上，由于这些电缆的不同地理位置，我相信我已经能够推断出每条电缆切断的时间。有两个明显的中断集群分别发生在09:46 UTC和约09:51 UTC之后——大约相隔五分钟。

Tata是Seacom电缆的部分所有者，在其使用寿命内，该电缆一直是Tata为其东非客户提供国际转接的首选方式。在09:46 UTC发生的东非地区的干扰往往涉及Tata转接的丢失。因此，我相信这就是Seacom电缆遭遇故障的时间点。

相反，第二波关于09:51 UTC的中断主要发生在从红海到亚洲的AAE-1路径上，尽管我们也在东非看到了影响。因此，我认为AAE-1在Seacom切断后几分钟内出现了故障。

让我们来看看在各种类型的测量数据中可见的一些影响，首先是一些Kentik BGP可视化。

下面是关于Arusha Art（AS37143）上游的BGP可视化，显示了41.222.60.0/23的每个BGP来源的百分比随时间的变化。该路由未撤回，但AS37143在Seacom电缆切断的预计时间失去了Tata（AS6453）的服务。

亚博·阿卜杜勒·阿齐兹科技城（AS8895）曾经由Tata（AS6453）独家提供转接，但当Seacom出现故障时，于09:46 UTC撤回。

在下图中，我们可以看到Djibouti Telecom随着AAE-1海底光缆损坏而经历的过渡变化。从AS30990（Djibouti Telecom）的上游视角来看，主要的变化是失去了Cogent（AS174）的服务，被AS6762（意大利电信）替代，直到几个小时后通过另一条光缆恢复Cogent的服务。

上述BGP可视化展示的过渡转变也反映在Kentik的汇总NetFlow中，如下图所示。

上图展示了阿联酋Etisalat发起的一条路由的过渡转变。当查看AS8966（Etisalat）此路由的上游时，我们发现在09:51 UTC时，由于AAE-1海底光缆断裂，从AS1299（Arelion）和AS2914（NTT）的过渡服务丢失了。根据我们的BGP数据，这一丢失被AS6762（意大利电信）、AS7473（新电信）以及AS3356（Lumen）扩展服务所替代。

AAE-1海底光缆断裂的影响可以在东南亚的越南主要运营商VNPT（AS45899）处观察到，例如以下所示的被撤销路由。

最后，这是一个明显受到两条断裂光缆影响的路由的例子。102.213.16.0/23（坦桑尼亚Equity Bank）由AS329242发起，并且由Simbanet（AS37084）独家过境。下面是AS37084的上游视图，显示了当Seacom光缆断裂时Tata（AS6453）服务的丢失，随后AAE-1光缆断裂时Cogent（AS174）服务的丢失。

此外，乔治亚理工学院的IODA工具报告称，东非国家（包括[Tanzania](https://ioda.inetintel.cc.gatech.edu/country/TZ?from=1708612033&until=1708957633)、[Kenya](https://ioda.inetintel.cc.gatech.edu/country/KE?from=1708612033&until=1708957633)、[Uganda](https://ioda.inetintel.cc.gatech.edu/country/UG?from=1708612033&until=1708957633)和[Mozambique](https://ioda.inetintel.cc.gatech.edu/country/MZ?from=1708612033&until=1708957633)）的活跃测量在09:50 UTC左右出现下降，很可能是由于Seacom光缆断裂所致。

红海多年来一直是海底光缆的问题区域，但目前的情况非常特殊。此前从未有敌对行为者在这个充满关键海底光缆的繁忙水域中向船只频繁开火。

红海中的商船仍然[遭到袭击](https://twitter.com/CENTCOM/status/1771726107168960751)，我们还没有摆脱困境。很可能再次有一艘船只被导弹击中，意外地切断另一条海底光缆。再次失去连接欧洲与亚洲的主要海底光缆将是灾难性的——总数本就不多。

最后，请为冒险驶入这些危险水域修理必要设备以维持国际通讯畅通的电缆船员们多想一想。最近也有也门胡塞派电信部长[发布声明](https://twitter.com/AlnomeirMosfer/status/1764666830801506476)，强调了在也门领海进行修理所需的许可要求，其政府继续以导弹和武装无人机袭击船只。

他补充道，这些许可是为了“关心（船只的）安全”。
