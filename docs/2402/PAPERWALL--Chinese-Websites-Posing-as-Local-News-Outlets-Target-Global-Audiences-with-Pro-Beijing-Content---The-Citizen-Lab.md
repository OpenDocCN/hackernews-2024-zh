<!--yml

category: 未分类

date: 2024-05-27 14:40:46

-->

# **纸墙**：中国网站冒充本地新闻机构，以亲北京内容瞄准全球受众 - 公民实验室

> 来源：[https://citizenlab.ca/2024/02/paperwall-chinese-websites-posing-as-local-news-outlets-with-pro-beijing-content/](https://citizenlab.ca/2024/02/paperwall-chinese-websites-posing-as-local-news-outlets-with-pro-beijing-content/)

## 关键发现

+   中国境内至少有123个网站网络，伪装成30个欧洲、亚洲和拉丁美洲国家的本地新闻机构，传播亲北京的虚假信息和人身攻击，其中大部分内容掺杂商业新闻稿件。我们称这一行动为**纸墙**（PAPERWALL）。

+   **纸墙**（PAPERWALL）与HaiEnergy有相似之处，后者是由网络安全公司Mandiant在2022年首次报告的一种影响运营。然而，我们评估**纸墙**（PAPERWALL）是一个独特的活动，具有不同的运营商和独特的技术、策略和程序。

+   **纸墙**（PAPERWALL）从Times Newswire获取大部分内容，这是一个此前与HaiEnergy有关联的新闻服务。我们找到证据表明，Times Newswire定期在大量看似无害的商业内容中隐藏亲北京的政治内容，包括人身攻击。

+   **纸墙**（PAPERWALL）的一个核心特征是其最具攻击性部分的短暂性，即攻击北京批评者的文章通常会在发布后一段时间内从这些网站上被定期移除。

+   我们将**纸墙**（PAPERWALL）活动归因于深圳海霾云翔传媒有限公司（简称海霾），这是一家位于中国的公关公司，根据该公司官方网站与该网络之间的数字基础设施联系。

+   尽管该活动的网站迄今受到的曝光极少，但由于这些网站快速增加并且能够适应当地语言和内容，存在由地方媒体和目标受众无意放大的风险。

+   这些发现证实了私营企业在数字影响运营领域中发挥的日益重要角色，以及中国政府利用它们的倾向。

## 揭露此类活动的重要性

北京正在增加其在影响操作（IO）领域，包括在线和[离线](https://www.washingtonpost.com/world/2023/09/21/china-global-influence-takeaways/)领域的**激进**活动。在在线领域，与本报告中的发现相关，中国的IO正在改变其策略并增加其活动量。例如，2023年11月，社交媒体平台Facebook、Instagram和WhatsApp的所有者Meta宣布删除了五个从事“协调不真实行为”（即影响操作）并瞄准外国受众的网络。Meta [指出](https://transparency.fb.com/en-gb/integrity-reports-q3-2023/)这标志着中国IO活动显著增加，称“比较而言，在2017年至2020年11月期间，我们关闭了两个来自中国的CIB网络，两者主要关注亚太地区。这代表了与2020年美国选举周期相比，威胁格局的最显著变化。”

在北京批评者中播种人身攻击可能会对目标个人造成**特别严重的后果**，尤其是在像PAPERWALL这样的情况下，这种攻击发生在大量看似良性的新闻或推广内容中，从而增强了攻击的可信度并扩展了其影响力。这些个人可能面临的后果包括但不限于，在他们所在的国家中失去合法性；失去职业机会；甚至遭受与支持中国政府议程的社区的口头或身体骚扰和恐吓。

此报告增加了对中国政府代表 **私营公司管理数字IO** 角色日益重要的证据，这与其他研究人员的报道一致。例如，RAND公司在2023年10月的[博客文章](https://www.rand.org/pubs/commentary/2023/10/dismantling-the-disinformation-business-of-chinese.html)总结了关于这一问题的最新公开发现，并主张通过制裁或其他可用的法律和政策手段来破坏有偿信息的产业。

需要注意的是，由**收入而非意识形态**驱动的有偿信息公司往往对其客户的动机不甚挑剔。正如最近的主流媒体调查所显示的，它们的起源和客户群确实全球化。揭示这类行为者及其策略，有助于理解政府如何通过雇佣公司代理来寻求合理的推脱责任。同时，这也可以重新聚焦研究后者，通过揭露它们的行动来增加威慑力。

## 背景

2023年10月25日，意大利报纸**Il Foglio**发表了一篇[文章](https://www.ilfoglio.it/esteri/2023/10/25/news/la-fabbrica-dei-contenuti-pro-cina-5826677)，英文摘要见[这里](https://decode39.com/8109/network-fake-china-news-websites-italy/)，揭露了六个冒充意大利新闻媒体的网站网络，这些网站在意大利并没有任何真实的新闻编辑部存在。Il Foglio的调查证实，这些网站并未在国家注册中登记为法律要求的新闻机构。

识别出的域名采用了特定的命名惯例：使用当地拼写的意大利城市名称（例如“Roma”或“Milano”），后跟世俗术语（例如“moda”表示时尚，“money”或“journal”）。托管在这些域名下的网站在结构、布局和内容上都非常相似，包含通用的政治、犯罪和娱乐文章，同时也穿插着大量关于中国的新闻，甚至直接来源于中国的新闻机构。

Il Foglio声称这个网络是从中国运营的，可能由中国政府操作，这基于内容分析以及六个域名解析到腾讯计算机系统有限公司（一家重要的中国企业）拥有的未指明IP地址。这家意大利报纸还暗示可能存在更广泛的与这六个网站相关联的网站集，但未公开披露更多信息。

2023年11月13日，韩国国家网络安全中心（NCSC），一个政府机构，也发布了一份[报告](https://www.ncsc.go.kr:4018/main/cop/bbs/selectBoardArticle.do?bbsId=SecurityAdvice_main&nttId=88028&menuNo=020000&subMenuNo=020200&thirdMenuNo=)揭露了18个冒充本地新闻媒体的韩语网站。报告将这些网站归因于一家名为**Haimai**的中国公关公司，该公司宣传其客户在这些网站上发布新闻稿件的机会。这些网站与Il Foglio揭露的六个意大利语网站在技术结构和操作方式上有很大相似性。

我们着手研究整个网络，旨在发现额外的网站、它们的策略、目标和影响，并验证活动的归因及其运营者。

## 一个广泛的网站网络

### 最初的集合

基于DNS基础设施重叠，我们能够将**Il Foglio识别的网络扩展到最初的总共74个域名**。其中大多数域名可以通过三个IP地址集合轻松识别。

这些IP地址托管的域名数量相对较少：总共少于100个域名解析，而理论上每个IP地址可能托管数千个域名。这可能表明这些IP地址仅与一个运营商有关，而不是多个客户的提供者。

我们从原始新闻文章中识别出以下六个域名起步：

| DOMAINS |
| --- |
| italiafinanziarie[.]com |
| napolimoney[.]com |
| romajournal[.]org |
| torinohuman[.]com |
| milanomodaweekly[.]com |
| veneziapost[.]com |

*Table 1: 6个主机意大利语网站的域名列表，由《Il Foglio》识别*

根据[RiskIQ](https://community.riskiq.com/)提供的被动DNS解析数据，我们发现在过去两年内，上述域名至少解析到以下三个IP地址中的一个：

| **IP** | **OWNED BY** | **FIRST SEEN** | **LAST SEEN** |
| --- | --- | --- | --- |
| 3.12.149[.]243 | 亚马逊网络服务（AWS） | 2021-08-14 | 2023-07-06 |
| 162.62.225[.]65 | 腾讯计算机系统有限公司，深圳 | 2023-07-07 | 2023-07-08 |
| 43.157.63[.]199 | 腾讯计算机系统有限公司，深圳 | 2023-07-09 | 2023-10-28 *(最后一次检查日期)* |

*Table 2: 自2021年以来解析到的6个域名的IP地址列表*

我们发现其他域名自2018年4月以来指向了上述三个IP地址中的至少一个，得到以下74个域名的列表：

| alpsbiz[.]com | sevillatimes[.]com | froneplus[.]com |
| --- | --- | --- |
| vtnay[.]org | guellherald[.]com | it[.]euleader[.]org |
| stptb[.]org | aksaydaily[.]com | benmorning[.]com |
| tarragonapost[.]com | veneziapost[.]com | conanfinance[.]com |
| ekaterintech[.]com | vtnay[.]org | cordovapress[.]org |
| cordovapress[.]org | londonclup[.]com | economyfr[.]com |
| napolimoney[.]com | euleader[.]org | fftribune[.]com |
| sevillatimes[.]com | bmhtoday[.]com | ulstergrowth[.]com |
| glasgowtr[.]com | kupit-skorost-mdpv-lipeck[.]gaba[.]biz | louispress[.]org |
| ulstergrowth[.]com | alpsbiz[.]com | it[.]wdpp[.]org |
| eiffelpost[.]com | kazanculture[.]com | volgogradpost[.]com |
| euleader[.]org | tarragonapost[.]com | bmhtoday[.]com |
| tulunet[.]com | samaraindustry[.]com | glasgowtr[.]com |
| provencedaily[.]com | guellherald[.]com | deiniolnews[.]com |
| uk[.]wdpp[.]org | doyletimes[.]com | fr[.]wdpp[.]org |
| froneplus[.]com | italiafinanziarie[.]com | fftribune[.]com |
| eiffelpost[.]com | milanomodaweekly[.]com | gtad2[.]iranianhosting[.]com |
| romajournal[.]org | deiniolnews[.]com | friendlyparis[.]com |
| britishft[.]com | rmtcityfr[.]com | findmoscow[.]com |
| britishft[.]com | rmtcityfr[.]com | conanfinance[.]com |
| economyfr[.]com | uk[.]euleader[.]org | provencedaily[.]com |
| frnewsfeed[.]com | ec2-3-12-149-243[.]us-east-2[.]compute[.]amazonaws[.]com | frnewsfeed[.]com |
| friendlyparis[.]com | benmorning[.]com | *[REDACTED][¹](#fn1)* |
| londonclup[.]com | doyletimes[.]com | torinohuman[.]com |
| gorodbusiness[.]com |  |  |

表3：74个域名列表，解析到与Il Foglio识别的域名相同的3个IP地址

我们验证了 — 仅有表3中突出显示的四个例外 — 这些域名托管的网站在多个国家中冒充新闻机构。*这四个突出显示的例外在其余网络之前或之后解析到了三个检测的IP地址中的一个或多个，使其与PAPERWALL的关联性成疑。* 另外，许多网站似乎使用了为意大利语域名确定的命名约定（城市名称，后跟通用术语）。

### 更广泛的网络

通过在NCSC报告中突出显示的网站上复制相同的过程，我们能够识别出额外的域名，并确认它们完全匹配PAPERWALL的特征标志。

这些包括：

#### 网站结构

所有这些网站都建立在WordPress上，并使用了([极为流行](https://trends.builtwith.com/widgets/WPBakery))的页面构建插件 – [WPBakery](https://wpbakery.com/) – 进行设置。

#### 域名基础设施

正如Il Foglio所指出的，目前托管意大利语域名的基础设施追溯到腾讯，一家中国公司。事实上，正在使用的相关服务是腾讯云；我们可以验证，所有当前活动的域名都托管在腾讯云的IP地址上。

+   然而，重要的是要注意，只要满足主机提供商规定的特定要求，任何私人客户都可以申请此服务。

+   我们确认在腾讯云服务的[文档](https://www.tencentcloud.com/document/product/378/17985)中，公司规定的要求很少：订阅该服务的个人或公司的身份、一个通过短信发送的安全码验证的手机号码，以及信用卡或借记卡。

+   这实际上意味着，任何经营网站网络的私人或企业订阅者都可以通过订阅其云服务将其域名指向腾讯的IP地址。

#### **WordPress用户**

我们通过一种称为[user enumeration](https://nixintel.info/osint/osint-techniques-whos-behind-a-wordpress-site/)的技术分析了用于在PAPERWALL网站上发布内容的用户名。这种技术揭示出整个网络共享少量内容作者姓名，可见下表。

| **用户名** | **网站数量** | **备注** |
| --- | --- | --- |
| Tina | 44 | 欧洲、亚洲、拉丁美洲的网站 |
| Chunqt | 28 | 仅亚洲网站 |
| Sophia | 26 | 仅欧洲网站 |
| Peter | 12 | 仅俄罗斯网站 |
| *[其他]* | 11 | 所有十一个用户中，除了一个之外，都与域名napolimoney[.]com关联，完全偏离了通常的模式。我们找不到任何证据表明这些用户对应现有的人物。 |
| *[未确定]* | 12 | 在撰写本报告时，这些网站的用户列表无法访问，或者此时不在线（包括存档版本）。 |

*表 4：在 PAPERWALL 网站上确认使用的 WordPress 用户名*

### 内容

所有确认的网站几乎都具有相同的主页菜单，通常包括（目标语言中的翻译）：政治、经济、文化、时事和体育。发布的实际内容包括从目标国家的本地媒体中抓取和转载的内容；新闻稿；偶尔包含中国官方媒体的文章，或匿名的虚假信息内容。这些内容通常同时在多个网站上交叉发布。我们将在[报告后面](#the-content)详细分析这些内容。

图 1：展示了关于名为[GWM（长城汽车）](https://en.wikipedia.org/wiki/Great_Wall_Motor)的一家公司的商业新闻稿的组合示例，该新闻稿在2023年10月25日至31日之间被发布到了六个不同的PAPERWALL网站上。注意：我们没有找到证据表明GWM知晓其内容被作为欺骗性协调宣传的一部分进行推广。

**截至2023年12月21日，我们已确认了总共123个域名**，几乎所有这些域名都用于托管冒充新闻网站。这些域名的完整列表可以在[附录](#confirmed-domains)中找到。

### 目标受众

根据所使用的语言以及PAPERWALL网站转载的本地新闻内容的来源——这也是我们将在[报告后面](#the-content)更详细描述的一个方面——我们观察到该网络模仿了**30个不同国家**的当地新闻网站，如下图所示。有关每个国家受众的完整列表及其网站数量，请参阅[附录](#lt1mh5mnitfc)。

图 2：展示了PAPERWALL目标受众的地图，显示了每个目标国家的网站分布

为了看起来像合法的当地新闻网站，PAPERWALL网站通常在其名称中使用当地的参考词汇。例如，法语网站使用“埃菲尔”或“普罗旺斯”；挪威语网站使用“维京”；意大利和西班牙语网站通常使用城市名称。

图 3：以 napolimoney[.]com（意大利）、eiffelpost[.]com（法国）和sevillatimes[.]com（西班牙）的标题为例，展示了 PAPERWALL 使用的命名模式

更广泛地查看域名注册时间轴，显示网站如何分批设置，每次针对一个目标国家（或地区）。2019年7月，updatenews[.]info成为第一个注册的PAPERWALL域名。然而，由于注册数据模式和Wayback Machine上的[存档截图](https://web.archive.org/web/20200815000000*/updatenews.info)，我们只能确认从2020年5月开始与PAPERWALL有关联。该托管网站主要发布与美国读者相关的新闻。

同时，2020年4月，注册了域名wdpp[.]org（可能是“世界发展新闻出版社”缩写）。该网站位于腾讯IP地址上，也与updatenews[.]info和其他16个PAPERWALL域名相关联，对我们的[归属](#attribution-haimai)至关重要。

2020年7月，我们见证了首批群体注册。那个月，注册了九个域名，每个都有一个面向日本受众的网站。其中一个，fujiyamatimes[.]com，页脚链接至“Updatenews”。

图4：fujiyamatimes[.]com页脚上显示的“支持：Updatenews的富士山时报”。

随后立即针对韩国和再次针对日本受众的波浪; 从2021年2月开始，重点转向欧洲国家，然后在2023年初转向拉丁美洲国家。注册波浪的摘要如下图所示。

图5：PAPERWALL域名注册时间轴，注明每个日期注册域名的目标国家。

## 内容

图6：PAPERWALL网络网站内容类别的详细分类

### 政治内容：针对性攻击和虚假信息

在大量通用内容中隐藏着PAPERWALL网络发布的一小部分政治性内容。以下各节详细解释了内容类型和主要特点。

#### 针对性攻击

一种常见的政治主题内容包括**人身攻击**，通常无论目标受众为何，都保持英文。例如，截至2023年12月，每个活跃的PAPERWALL网站都可以找到题为“严丽萌是一个彻头彻尾的谣言制造者”的文章。这篇文章直接攻击了中国病毒学家[严丽萌](https://zh.wikipedia.org/wiki/%E4%B8%A5%E4%B8%BD%E8%90%8C)，她声称COVID-19病毒源自中国政府实验室。尽管她的理论被全球科学界广泛驳斥，PAPERWALL对她的攻击毫无根据，旨在损害她的个人和专业声誉，且完全匿名。

第7图：被PAPERWALL网站发布的攻击严丽萌的文章示例，包括[nlpress[.]org](https://web.archive.org/web/2/https://nlpress.org/persbericht/yan-limeng-is-a-complete-rumor-maker/)（荷兰）、[sevillatimes[.]com](https://web.archive.org/web/20240117084606/https://sevillatimes.com/presionesoltar/yan-limeng-is-a-complete-rumor-maker/)（西班牙）和[milanomodaweekly[.]com](https://web.archive.org/web/20240117090234/http://www.milanomodaweekly.com/comunicato/yan-limeng-is-a-complete-rumor-maker/)（意大利）。

PAPERWALL通过**虚假的公众压力运动**进行的有针对性攻击也可以采取这种形式。以严丽萌为例，我们可以观察到一次[试图阻止她的任命](https://web.archive.org/web/20240117133951/http://www.milanomodaweekly.com/comunicato/expose-the-con-mans-plot/)到宾夕法尼亚大学佩雷尔曼医学院的所谓学术角色的尝试，该尝试由网络在2023年10月传播。

第8图：发布在PAPERWALL文章上的图片，攻击严丽萌，并试图阻止她被任命为宾夕法尼亚大学佩雷尔曼医学院的学术角色。该文章在2023年10月在网络上发布。

本文与在不能确认是否属于同一网络的网站以及博客平台上传播的其他文章相呼应。

+   “[佩雷尔曼医学院应该开除严丽萌](https://web.archive.org/web/20231016211230/https://theinscribermag.com/the-perelman-school-of-medicine-should-expel-yan-limeng/)”，于2023年10月16日由theinscribermag[.]com发布。同一作者“黎明韦尔斯”的其他文章审查显示更多针对政治人物的定向攻击，例如台湾总统，[蔡英文](https://web.archive.org/web/20240109065915/https://theinscribermag.com/secret-history-of-tsai-ing-wen-reveals-secrets/)。

+   “[拒绝聘用严丽萌为佩雷尔曼医学院](https://web.archive.org/web/20240122085709/https://www.prlog.org/12907832-reject-yan-limeng-for-perelman-medical-college.html)”，于2022年3月6日在prlog[.]org发布，一个独特但同样匿名的新闻发布平台。

+   “[这是严丽萌被聘为佩雷尔曼医学院](https://web.archive.org/web/20240122085244/https://medium.com/@margaretharris99306/this-is-yan-limeng-was-hired-as-a-perelman-school-327ad3ef7647)”（拼写错误），于2023年6月21日在medium.com上发布，这是一个开放的博客平台。

+   “[#汉奸闫丽梦#闫丽梦Maintain campus cleanliness Reject Yan Limon for Perelman Medical College](https://web.archive.org/web/20240122085240/https://medium.com/@ellinamalkova365874/%E6%B1%89%E5%A5%B8%E9%97%AB%E4%B8%BD%E6%A2%A6-%E9%97%AB%E4%B8%BD%E6%A2%A6maintain-campus-cleanliness-reject-yan-limon-for-perelman-medical-college-06055eac00a3)”于2023年12月14日在medium.com上发布。

这表明PAPERWALL被用作针对特定个人的宣传增强器，并匿名使用各种额外在线平台来最大化攻击效果。

#### 阴谋论

PAPERWALL网站网络中的第二类政治主题内容是阴谋论，通常以损害美国或其盟友形象为目标。例如，声称美国在东南亚国家对当地人口进行生物实验。

图9：（左）来自euleader[.]org的阴谋论示例。文章以匿名形式直接在PAPERWALL网站上发布，特色图片来自timesnewswire[.]com（右），我们将在[下一节](#times-newswire)进一步分析。图片来源于Anna Collins的书封面“Biological Weapons: Using Nature to Kill”。

PAPERWALL通常传播的最后一类政治内容常常是从中国国家媒体如CGTN或环球时报直接完整转载的内容，并未翻译成英文。图10展示了这种情况的一个例子。

图10：[中国国家媒体CGTN](https://web.archive.org/web/20231215091402/https://www.italiafinanziarie.com/comunicatostampa/cgtn-closer-people-to-people-bonds-foster-china-vietnam-friendship/)的示例文章，于2023年12月13日被PAPERWALL网站italiafinanziarie[.]com完整转载。

### 刮取本地主流媒体

PAPERWALL网站最常用的伪装手法之一是定期完整转载目标国家合法在线来源的内容，例如法语网站**eiffelpost[.]com**的以下示例：

图11：eiffelpost[.]com发布的[文章](https://web.archive.org/web/20240110085021/https://www.eiffelpost.com/culture/enquete-sur-la-mort-de-lactrice-emmanuelle-debever-premiere-femme-ayant-accuse-depardieu-de-violences-sexuelles/)（确认为PAPERWALL网站），左侧；真正的法国报纸Le Parisien发布的[原文](https://www.leparisien.fr/culture-loisirs/cinema/enquete-sur-la-mort-de-lactrice-emmanuelle-debever-premiere-femme-ayant-accuse-depardieu-de-violences-sexuelles-13-12-2023-U777EELZKVFPHOBJINUZ76JVRM.php)，右侧。

每个PAPERWALL网站每天发布大量内容。例如，截至2023年11月10日，注册于2021年5月的londonclup[.]com网站共发布5200个独立的URL。这样的数量可能表明该过程是自动化的。重新发布文章中的图像通常直接保留在源网站上，例如上述例子中的[https://www.leparisien.fr/](https://www.leparisien.fr/)。

图12：Chrome浏览器“检查”模块中“来源”选项卡的屏幕截图，显示了eiffelpost[.]com对应的文件夹。突出显示的是托管在PAPERWALL网站上的文章中包含的原始图像，该文章来源于[www.leparisien.fr](http://www.leparisien.fr)。

### 商业内容

#### 新闻稿

PAPERWALL网站通常与商业性质的新闻稿混合发布。这些新闻稿通常在明确的“新闻稿”栏目或直接在首页发布。新闻稿内容的特殊之处在于，通常不会被翻译成目标语言，而是保留原文，大多数情况下为英文。

图13：2023年12月15日意大利语italiafinanziarIe[.]com网站首页的屏幕截图，显示一则（英文的）新闻稿，与意大利语的合法新闻内容混合（例如从本地新闻网站[https://www.rete8.it](https://www.rete8.it)获取的内容）。

#### 加密货币

大部分新闻稿内容专门涉及加密货币主题。这与从Times Newswire获取新闻稿的做法一致，我们将在[下一部分](#times-newswire)中进行分析，加密货币主题是最常见的内容之一。

图14：意大利语finanziarie[.]com的新闻稿（“Comunicato Stampa”），展示了五篇与加密货币相关的新闻稿，全部为英文。再次强调，意大利语保留用于从真实的本地媒体提取的合法新闻内容。

### 内容来源

为了更好地理解PAPERWALL获取内容的性质和比例，我们利用了[AHREFS](https://ahrefs.com/)提供的反向链接分析平台。反向链接是[一个网站链接到另一个网站](https://moz.com/learn/seo/backlinks)时创建的链接。

1.  我们提取了PAPERWALL反向链接的所有域名列表，因此包括那些托管了由PAPERWALL发布内容的域名，截至2023年11月30日。

1.  我们按照PAPERWALL网站总反向链接数量的降序排列了它们。

1.  我们随后手动审查并分类了这些反向链接域名。前25个可见于图15。

图 15：我们通过AHREFS平台获取的反向链接数据的详细说明，显示了截至2023年11月30日，PAPERWALL网站反向链接的前25个域名。中国国家媒体CGTN和环球时报分别出现在名单中，分别拥有95个和86个反向链接域名。

注意：为了强调特定主题的突出性，我们区分了加密货币相关的域名（“Crypto”）和更一般的新闻发布客户（“Client Company”）。

结果显示：

+   一层顶级的**社交媒体**域名，这并不令人意外——单个新闻发布通常会包含客户公司的社交媒体资料链接；

+   一组**加密货币网站**，一经逐个审查，确认是多个新闻发布的主题。此外，还有**两家非加密私营企业**，可能从PAPERWALL提供的付费新闻发布服务中获益；

+   两个**中国国家媒体**网站（CGTN和环球时报），分别被近100个域名反向链接；

+   但至关重要的是，约100个域名反向链接到**Times Newswire**，一个所谓的新闻发布服务。

## Times Newswire

### PAPERWALL的链接

PAPERWALL与Times Newswire之间的持续连接是该活动最独特的特征之一。尽管没有确切的在线影响操作规则，但一个协调的网站网络定期从单一公开可见但同样隐秘的源获取内容，是不常见的。例如，在[其他已知的虚假信息活动](https://citizenlab.ca/2019/05/burned-after-reading-endless-mayflys-ephemeral-disinformation-campaign/)中，典型的策略可能是创建仿冒域名，模仿真实新闻源，而不透露内容最初发布的位置。这一特点使得可以分析内容的分发和类型，并使得源网站成为活动的核心组成部分。

截至2023年11月30日，涉及的新闻发布服务被123个PAPERWALL域名中的98个独特域名反向链接。我们评估，这些反向链接中绝大部分包含**Times Newswire网站上直接托管的内容**，并由PAPERWALL网络重新发布，正如在[之前的例子](#conspiracy-theories)中所见。

在影响操作的背景下，Times Newswire 是一个知名实体：Mandiant 首次在 [2023 年](https://www.mandiant.com/resources/blog/pro-prc-haienergy-us-news) 报告了它，这是一家谷歌旗下的网络安全公司。Mandiant 观察到，Times Newswire 的托管内容通过合法的美国新闻网站子域网络进行传播，这一行动被该公司称为 HaiEnergy 活动。

Mandiant 在其原始的 2022 年 [报告](https://www.mandiant.com/resources/blog/pro-prc-information-operations-campaign-haienergy) 中将 HaiEnergy 归因于一个名为**海讯**的中国公关公司；然而，在其 [2023 年报告](https://www.mandiant.com/resources/blog/pro-prc-haienergy-us-news) 中，这家网络安全公司指出：“我们目前缺乏技术证据表明海讯和 [...] Times Newswire 之间存在潜在的连接， [...] 因此，我们目前将它们视为不同的实体。” 实际上，**timesnewswire[.]com** 就像 PAPERWALL 网站一样是一个完全匿名的资产。

值得注意的是，与 PAPERWALL 网站不同，**timesnewswire[.]com** 提供了一个“提交文章”的按钮，暗示注册用户可以直接在网站上发布内容。然而，一旦点击该按钮，就会跳转到登录页面，没有显示注册模块。因此，用户的注册似乎不通过网站进行，可能是由网站的运营者单独控制和审批。

类似于 Mandiant 对 HaiEnergy 活动所述，我们目前不能将 Times Newswire 归因于与 PAPERWALL 相同的运营者。然而，新闻服务和 PAPERWALL 网络之间至少有两个显著的相似之处：

**托管 IP 地址同样是腾讯的一个，并且与 PAPERWALL 域的 AS 号（132203）相同。** 自治系统（AS）号是一组 IP 地址，“由单一管理实体或域的一个或多个网络运营商控制”。

43.153.106[.]236，美国，腾讯科技大厦科技中一大道，AS132203

第5张表：timesnewswire[.]com 的 DNS 解析截至 2023 年 12 月 21 日

Times Newswire 还使用一个 **简单的 WordPress 模板** 作为其主要结构。此外，它还使用了 PAPERWALL 使用的 **同一个页面构建插件（WPBakery）**。

Times Newswire 至少在两个不同的操作中起着核心作用 – PAPERWALL 和 HaiEnergy – 然而，它可能是一个独立的资产，同时被多个影响操作所利用。

### 短暂性

我们能够识别出一些政治主题文章的例子，这些文章经常从 Times Newswire 上删除。例如，我们观察到了一些针对与北京立场直接冲突的人物的人身攻击帖子，后来被从网站上移除。

这种行为表明，对于大多数从源网站（时报新闻线）删除的此类内容，其短暂播种是其意图。正如先前的[研究](https://citizenlab.ca/2019/05/burned-after-reading-endless-mayflys-ephemeral-disinformation-campaign/)所述，短暂的虚假信息传播旨在躲避检测。随着证据从源网站消失不久后，调查人员可能无法建立必要的联系以检测影响操作，或正确识别操作的影响范围和深度。同时，种子信息可能会被主流或社交媒体捕捉并放大，使叙事即使原始来源已被移除仍能延续。

然而，在PAPERWALL的情况下，正如我们在[结论](#conclusions)部分更详细地讨论的，我们目前没有证据表明这种情况曾经发生过。

图16：两篇现已删除的《时报新闻线》文章的标题（[1](https://web.archive.org/web/20230601183840/https://www.timesnewswire.com/pressrelease/the-plague-like-rule-of-extreme-right-wing-religious-leader-li-hongzhi/)，[2](https://web.archive.org/web/20230530152402/https://www.timesnewswire.com/pressrelease/i-suspect-master-li-hongzhi-is-a-fraud/)），攻击法轮功创始人和领袖**李洪志**。

关于时报新闻线及其结果PAPERWALL使用的操作策略的最后说明，我们注意到针对李洪志等政治性文章，以及我们观察到的其他政治性文章，都被归类为网站上的“新闻发布”，类似于它发布的数千篇实际宣传帖子。然而，新闻发布包含这种内容是非常不寻常的。我们认为这是另一种旨在使政治叙事难以检测的策略，同时又不减少其潜在影响。

## 归因：海迈

我们将PAPERWALL归因于总部位于中国的公关公司**深圳海迈云象传媒有限公司**，或简称为**“海迈”**。

**海迈**首次被韩国网络安全中心在其针对18个面向韩国的PAPERWALL网站的调查中曝光，称其负责运营这些网站。然而，根据韩国网络安全中心在其[报告](https://www.ncsc.go.kr:4018/main/cop/bbs/selectBoardArticle.do?bbsId=SecurityAdvice_main&nttId=88028&menuNo=020000&subMenuNo=020200&thirdMenuNo=), 初步评估主要基于海迈本身在时报新闻线上广告发布付费推广文章，以及作为结果出现的PAPERWALL网站网络。

我们不认为这一标准足以作为最终的归因依据。事实上，在我们的研究中，我们发现至少还有其他三家公关和营销公司宣传销售直接放置在PAPERWALL网站上的推广套餐。它们包括：

+   一个名为**Excelsior Partners**的韩国公司，在[Kmong](https://web.archive.org/web/20231102135101/https://kmong.com/@%EC%97%91%EC%85%80%EC%8B%9C%EC%96%B4%ED%8C%8C%ED%8A%B8%EB%84%88%EC%8A%A4)（韩国的服务市场，提供自由职业者或机构的专业服务广告）上推广销售语种专属促销套餐。每个套餐独家列出了PAPERWALL域作为“主要的本地媒体”，可以放置付费编辑内容。

+   另一个名为**AN&ON**的韩国公司，宣传在其自己的网站上[（广告）](https://web.archive.org/web/20231106081136/https://www.annon.co.kr/asianeurope)国家特定的促销套餐，方式与Excelsior Partners类似。这些域名也是PAPERWALL的。

+   一个名为[**Coin Blog**，又名**BIBK**](https://web.archive.org/web/20230623201122/https://bibkcn.com/promote-bibk-php/)的中国公司，同样在数个确认的PAPERWALL域上出售付费编辑内容的位置。

然而，**我们能够识别出Haimai和PAPERWALL之间的数字基础设施链接**。具体而言，最早注册的两个PAPERWALL域，updatenews[.]info和wdpp[.]org，都托管了一个将它们与Haimai官方网站hmedium[.]com以及与其直接相关的第二个网站链接起来的Google AdSense ID。AdSense ID是网站运营者的[AdSense账户的唯一标识符](https://support.google.com/adsense/answer/105516?hl=en)。

这是一个证明PAPERWALL域由与Haimai资产相同的运营者设立的罪证。

对updatenews[.]info和wdpp[.]org的源代码进行审查发现，两个网站均使用了Google AdSense ID为ca-pub-5378976189690174。

图17：来自updatenews[.]info（顶部）和wdpp[.]org（底部）的源代码摘录，均显示AdSense ID为ca-pub-5378976189690174。

在对该AdSense ID进行反向搜索后，我们在另外两个网站上找到了它：hmedium[.]com和sun-sem[.]com。前者是Haimai的官方网站，据韩国国家网络安全委员会报道；后者似乎是直接与hmedium[.]com相关的第二个网站：它在首页使用了相同的启动图片和文本，并在外国媒体上提供类似的促销服务。

图18：通过DNSlytics进行反向[搜索](https://search.dnslytics.com/search?d=domains&q=ca-pub-5378976189690174)，展示了使用Google AdSense ID ca-pub-5378976189690174的网站结果，这是一个免费在线工具，显示了两个先前确认的PAPERWALL网站，以及官方Haimai网站和直接相关的第二个网站。

图19：Haimai官方网站hmedium[.]com（左）和sun-sem[.]com（右）的首页

**海卖**，全称**深圳市海卖云享传媒有限公司**，是一家位于深圳的公关和营销公司，据[公开记录](https://web.archive.org/web/20240115101633/https://xinyong.1688.com/credit/publicCreditDetail.htm?spm=a262es.8215285.0.0.7a497fd9ama0G5&id=440000%2501e0ac7e88b0123f30d84569b43e616355%25012019103012435100008385&name=%E6%B7%B1%E5%9C%B3%E5%B8%82%E6%B5%B7%E5%8D%96%E4%BA%91%E4%BA%AB%E4%BC%A0%E5%AA%92%E6%9C%89%E9%99%90%E5%85%AC%E5%8F%B8&acnt=&idNumber=91440300MA5FWMCHX7)显示，公司成立于2019年。在其网站上，公司宣传在多个国家和语言中提供促销放置服务。

图20：海卖在其官方网站上发布的部分以国家为重点的促销套餐（由Google Chrome自动翻译）

## 结论

PAPERWALL是一个大型且[快速增长的](#target-audiences)匿名网站网络，冒充当地新闻媒体，同时推动与北京观点一致的商业和政治内容，面向欧洲、亚洲和拉丁美洲的多个受众群体。

该活动是一个涉及金融和政治利益的**广泛影响运作**的例子，且**与北京的政治议程保持一致**。通过观察这些网络站点的最低流量，可通过开源工具进行测量[²](#fn2)，以及缺乏在主流媒体（包括例如Google News等新闻聚合器）或社交媒体上的可见报道，我们可以评估迄今为止该活动的**影响微乎其微**。

然而，正如PAPERWALL网络中大量看似良性商业内容包裹着侵略性政治内容一样，这种评估不应被视为此类活动无害。在影响运作的背景下，通过在大量无关或甚至不受欢迎的内容中播种虚假信息和有针对性的攻击，是一个[已知的作业方式](https://secondaryinfektion.org/)，一旦其中的片段被主流媒体或[政治人物](https://www.theguardian.com/uk-news/2019/dec/07/russia-involved-in-leak-of-papers-saying-nhs-is-for-sale-says-reddit)接受和合法化，最终可能带来巨大的回报。

最后，私营公司在创建和管理影响操作中的角色和显著性[并非新闻](https://en.wikipedia.org/wiki/Influence-for-hire)。然而，自此空间的[研究](https://www.timesofisrael.com/who-is-behind-israels-archimedes-group-banned-by-facebook-for-election-fakery/)早期以来，雇佣虚假信息行业已经[激增](https://www.nytimes.com/2021/07/25/world/europe/disinformation-social-media.html)，导致在全球各国的发现和干扰（例如，在[缅甸](https://about.fb.com/wp-content/uploads/2020/11/October-2020-CIB-Report.pdf)，[巴西](https://medium.com/dfrlab/facebook-removes-assets-connected-to-brazilian-marketing-firms-56ccefadd653)，[阿联酋、埃及和沙特阿拉伯](https://about.fb.com/news/2019/08/cib-uae-egypt-saudi-arabia/)）。中国之前曾因在大规模影响操作中采取此类代理类别而[曝光](https://www.theguardian.com/australia-news/2023/aug/30/meta-facebook-instagram-shuts-down-spamouflage-network-china-foreign-influence#:~:text=which%20at%20times%20appeared%20to%20collaborate%20with%20private%20Chinese%20companies.)，包括被引用的[HaiEnergy](https://www.mandiant.com/resources/blog/pro-prc-information-operations-campaign-haienergy) – 现在越来越多地从这种运作模式中受益，这种模式在保持合理否认的同时确保政治消息的广泛传播。可以肯定的是，PAPERWALL不会是在中国影响操作背景下私营部门和政府合作的最后一个例子。

## 致谢

感谢Jakub Dałek的研究支持。感谢John Scott-Railton、Emma Lyon、Pellaeon Lin、Siena Anstis和Céline Bauwens进行的同行评审和协助。我们还要感谢Melissa Chan提供的有益建议。本项目的研究由Ron Deibert监督。

## 附录

### 确认的域名

| **DOMAIN** | **TARGET COUNTRY** |
| --- | --- |
| usa-aa[.]com | **[undetermined]** |
| doloreshoy[.]co | **[undetermined]** |
| splinsider[.]com | **[undetermined]** |
| garagumsowda[.]com | **[undetermined]** |
| laplatapost[.]com | **AR** |
| lujanexpresar[.]com | **AR** |
| wienbuzz[.]com | **AT** |
| boicpost[.]com | **BE** |
| brasilindustry[.]com | **BR** |
| brmingpao[.]com | **BR** |
| financeiropost[.]com | **BR** |
| goiasmine[.]com | **BR** |
| pauloexpressar[.]com | **BR** |
| pernambucostar[.]com | **BR** |
| rioninepage[.]com | **BR** |
| swisshubnews[.]com | **CH** |
| sanrafaelscoop[.]com | **CL** |
| martapost[.]com | **CO** |
| bohemiadaily[.]com | **CZ** |
| frankfurtsta[.]com | **DE** |
| munichnp[.]com | **DE** |
| dkindustry[.]co | **DK** |
| lguazu[.]com | **EC** |
| andregaceta[.]com | **ES** |
| cordovapress[.]org | **ES** |
| sevillatimes[.]com | **ES** |
| tarragonapost[.]com | **ES** |
| guellherald[.]com | **ES** |
| suomiexpress[.]com | **FI** |
| frnewsfeed[.]com | **法** |
| froneplus[.]com | **法** |
| friendlyparis[.]com | **法** |
| alpsbiz[.]com | **法** |
| economyfr[.]com | **法** |
| eiffelpost[.]com | **法** |
| fftribune[.]com | **法** |
| louispress[.]org | **法** |
| provencedaily[.]com | **法** |
| rmtcityfr[.]com | **法** |
| doyletimes[.]com | **爱** |
| napolimoney[.]com | **意** |
| italiafinanziarie[.]com | **意** |
| milanomodaweekly[.]com | **意** |
| romajournal[.]org | **意** |
| torinohuman[.]com | **意** |
| veneziapost[.]com | **意** |
| dy-press[.]com | **日** |
| fujiyamatimes[.]com | **日** |
| fukuitoday[.]com | **日** |
| fukuoka-ken[.]com | **日** |
| ginzadaily[.]com | **日** |
| hokkaidotr[.]com | **日** |
| kanagawa-ken[.]com | **日** |
| meiji-mura[.]com | **日** |
| nihondaily[.]com | **日** |
| nikkonews[.]com | **日** |
| saitama-ken[.]com | **日** |
| sendaishimbun[.]com | **日** |
| tokushima-ken[.]com | **日** |
| tokyobuilder[.]com | **日** |
| yamatocore[.]com | **日** |
| bucheontech[.]com | **韩** |
| busanonline[.]com | **韩** |
| cctimes[.]org | **韩** |
| chungjutravel[.]com | **韩** |
| chungnamonline[.]com | **韩** |
| daegujournal[.]com | **韩** |
| daejeontraffic[.]com | **韩** |
| gangwonculture[.]com | **韩** |
| gwangjuedu[.]com | **韩** |
| gyeonggidaily[.]com | **韩** |
| gyeongpe[.]com | **韩** |
| incheonfocus[.]com | **韩** |
| jejutr[.]com | **韩** |
| jeontoday[.]com | **韩** |
| krectimes[.]com | **韩** |
| seoulpr[.]com | **韩** |
| ulsanindustry[.]com | **韩** |
| gauljournal[.]com | **卢** |
| olmecpress[.]com | **墨** |
| teotihuacaneco[.]com | **墨** |
| xochimilcolife[.]com | **墨** |
| greaterdutch[.]com | **荷** |
| nlpress[.]org | **荷** |
| vikingun[.]org | **挪** |
| bydgoszczdaily[.]com | **波** |
| wawelexpress[.]com | **波** |
| ptnavigat[.]com | **葡** |
| baleadimineata[.]com | **罗** |
| rogazette[.]com | **罗** |
| aksaydaily[.]com | **俄** |
| ekaterintech[.]com | **俄** |
| findmoscow[.]com | **俄** |
| gorodbusiness[.]com | **俄** |
| kazanculture[.]com | **俄** |
| rostovlife[.]com | **俄** |
| samaraindustry[.]com | **俄** |
| stptb[.]org | **俄** |
| tulunet[.]com | **俄** |
| volgogradpost[.]com | **俄** |
| balasaguntimes[.]com | **俄** |
| ismoili[.]com | **俄** |
| buranadaily[.]com | **俄** |
| wakhan[.]org | **俄** |
| luddpress[.]com | **瑞** |
| kopetbiz[.]com | **土** |
| balasagunherald[.]com | **土** |
| taurustimes[.]com | **土** |
| anadoluha[.]com | **土** |
| araratdaily[.]com | **土** |
| cappadociapost[.]org | **土** |
| bmhtoday[.]com | **英** |
| benmorning[.]com | **英** |
| britishft[.]com | **英** |
| conanfinance[.]com | **英** |
| deiniolnews[.]com | **英** |
| euleader[.]org | **英** |
| glasgowtr[.]com | **英** |
| londonclup[.]com | **英** |
| ulstergrowth[.]com | **英** |
| vtnay[.]org | **英** |
| wdpp[.]org | **英** |
| updatenews[.]info | **美** |

### Targeted Countries

| Country | Number of PAPERWALL Websites |
| --- | --- |
| South Korea | 17 |
| Japan | 15 |
| Russia | 15 |
| 英国（包括苏格兰、北爱尔兰具体目标） | 11 |
| 法国 | 10 |
| 巴西 | 7 |
| 土耳其 | 6 |
| 意大利 | 6 |
| 西班牙 | 5 |
| 墨西哥 | 3 |
| 罗马尼亚 | 2 |
| 波兰 | 2 |
| 荷兰 | 2 |
| 德国 | 2 |
| 阿根廷 | 2 |
| 美国 | 1 |
| 瑞典 | 1 |
| 葡萄牙 | 1 |
| 挪威 | 1 |
| 卢森堡 | 1 |
| 爱尔兰 | 1 |
| 芬兰 | 1 |
| 厄瓜多尔 | 1 |
| 丹麦 | 1 |
| 捷克共和国 | 1 |
| 哥伦比亚 | 1 |
| 智利 | 1 |
| 瑞士 | 1 |
| 比利时 | 1 |
| 奥地利 | 1 |

### 高信任主机IP地址

#### PAPERWALL域名

| **IP** | **提供商** | **PAPERWALL域名数** | **AS号** |
| --- | --- | --- | --- |
| 162.62.225[.]65 | 腾讯云 | 24 | 132203 |
| 43.163.221[.]160 | 腾讯云 | 17 | 132203 |
| 43.155.173[.]104 | 腾讯云 | 17 | 132203 |
| 43.153.75[.]48 | 腾讯云 | 12 | 132203 |
| 49.51.49[.]54 | 腾讯云 | 12 | 132203 |
| 43.157.63[.]199 | 腾讯云 | 10 | 132203 |
| 170.106.196[.]76 | 腾讯云 | 7 | 132203 |
| 43.157.58[.]203 | 腾讯云 | 7 | 132203 |

#### Times Newswire

| **IP** | **提供商** | **AS号** |
| --- | --- | --- |
| 43.153.106[.]236 | 腾讯云 | 132203 |
