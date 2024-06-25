<!--yml

分类：未分类

日期：2024 年 5 月 27 日 14:41:32

-->

# Outlook 是微软的新数据收集服务 | Proton

> 来源：[`proton.me/blog/outlook-is-microsofts-new-data-collection-service`](https://proton.me/blog/outlook-is-microsofts-new-data-collection-service)

最后更新于 2024 年 3 月 1 日，于 2024 年 1 月 5 日发布

随着微软[推出新的 Windows Outlook](https://www.windowscentral.com/software-apps/windows-11/whats-new-with-the-outlook-app-for-windows-11)，该公司似乎已将其电子邮件应用转变为监视工具，用于定向广告。

每个人都在谈论 Google 和 Apple 的隐私洗白活动，因为它们挖掘你的在线数据以获取广告收入。但现在看来，Outlook 不再仅仅是一个电子邮件服务；它是微软的 801 个外部合作伙伴的数据收集机制，也是微软自身的广告投放系统。

这是如何以及为什么。

## **微软与 801 家第三方分享你的数据**

下载新的 Windows Outlook 的一些欧洲用户将会遇到一个模态框，其中披露了关于微软和数百家第三方如何处理他们的数据的令人不安的信息：

该窗口通知用户，微软和那 801 家第三方将其数据用于多种目的，包括：

+   存储和/或访问用户设备上的信息

+   开发和改进产品

+   个性化广告和内容

+   测量广告和内容

+   派生受众洞察

+   获取精确的地理位置数据

+   通过设备扫描识别用户

这一最新版本的 Outlook 证实了大型科技公司更多的利润边际正在变得越来越依赖于收集你的个人数据。Outlook 还提示你选择广告在屏幕上的显示方式，表明广告是交易的关键部分。

登录新 Outlook 的 Mac 用户甚至会遇到看起来像收件箱消息的广告。一些广告是微软应用程序的，而另一些来自销售产品的第三方。

## **微软的“广告合作伙伴”**

多亏了欧盟的通用数据保护条例，欧洲人至少被告知将有数百家第三方能够查看他们的数据。由于美国政府拒绝通过隐私立法，美国人甚至从未被告知这一点。

在 Outlook 设置中，英国用户可以查看一个“广告合作伙伴列表”，其中显示了与微软合作的令人不安的广告公司数量。这些第三方公司 - 被称为合作伙伴 - 的名字包括“ADMAX”和“ADSOCY”。在美国和瑞士的用户设置中无法找到此选项。

在某种程度上，新的 Outlook 让你选择如何使用你的数据，但这并不像点击一个单一的开关那样简单。

“根据他们收集、使用和处理的数据类型以及其他因素，包括隐私设计，某些合作伙伴依赖您的同意，而其他人则需要您选择退出，”英国用户的首选项页面写道。“点击下面列出的每个广告公司以查看其隐私政策并行使您的选择。”

并非每个合作伙伴都有相同的规定。用户可以在决定之前阅读每个个别的隐私政策，但并非强制阅读。

此类政策通常冗长、啰嗦且出了名地难以理解。但对于许多公司来说，这就是目的。这些政策有意写成这样，以赋予公司最大的自由度来处理您的数据。这通常意味着将您的个人详细信息出售给第三方广告商和数据经纪人，同时使您难以选择退出。

使用新 Outlook，Microsoft 强迫用户输入迷宫般的隐私声明，以夺回对其数据的部分控制。当然，微软知道几乎没有人[阅读隐私政策](https://www.pewresearch.org/internet/2019/11/15/americans-attitudes-and-experiences-with-privacy-policies-and-laws/)。如果每个人都理解这些政策，收入就会受到威胁。

## **新 Outlook 窃取你的电子邮件密码**

Microsoft 将 Outlook 与云服务整合后，引起了[隐私警报](https://news.ycombinator.com/item?id=38219568)。

当你在新的 Outlook 中[同步来自诸如 Yahoo 或 Gmail 的第三方电子邮件帐户](https://support.microsoft.com/en-us/office/sync-your-account-in-outlook-to-the-microsoft-cloud-985f9e19-d308-4e85-9d1d-0c6f32f8e981#OfficeVersion=New_Outlook_for_Windows)时，你有可能给了 Microsoft 访问 IMAP 和 SMTP 凭据、电子邮件、联系人和与这些帐户相关的事件的风险，据德国 IT 博客[Heise Online](https://www.heise.de/news/Microsoft-krallt-sich-Zugangsdaten-Achtung-vorm-neuen-Outlook-9357691.html?wt_mc=rss.red.ho.ho.atom.beitrag.beitrag)报道。

“尽管微软解释说随时可以切换回以前的应用程序，但数据已经被公司存储了，”Heise 报道。“这使得微软可以阅读电子邮件。”

没有将所有这些信息与 Microsoft 云同步，你无法使用新 Outlook — 只有取消的选项，[根据开发者论坛 XDA](https://www.xda-developers.com/privacy-implications-new-microsoft-outlook/)。它还配置为将登录详细信息 — 包括用户名和密码 — 直接发送到 Microsoft 服务器。

尽管此传输受到传输层安全性（TLS）的保护，但根据 Heise Online 的说法，您的 IMAP 和 SMTP 用户名和密码以纯文本形式传输到微软。[XDA（新窗口）](https://www.xda-developers.com/privacy-implications-new-microsoft-outlook/) 能够展示他们在 Microsoft 服务器上测试第三方电子邮件服务提供商的测试凭据。

微软使自身能够随时访问您的电子邮件帐户而不需要您的知识，允许它扫描和分析您的电子邮件 —— 并与第三方分享。

对于不了解隐私影响的用户来说，使用新 Outlook 可能看起来是无害的。但它可能意味着欢迎微软进入您的数据保险箱，并给予他们完全自由，以潜在地将其用于他们想要的方式。

德国数据保护和信息自由专员乌尔里希·凯尔伯教授对新 Outlook 的数据功能表示担忧。他在 Mastodon 上 [宣布（新窗口）](https://social.bund.de/@bfdi/111381793883035665) 他打算要求爱尔兰数据保护专员报告，该专员负责确保微软等公司遵守数据保护和隐私标准。

微软尚未对其最新的数据抓取提出批评做出公开回应。但这家软件巨头一直坦诚地表示，他们正推动使用定向广告来实现新的收入高度。2021 年，微软广告业务获得了 100 亿美元。但 [微软希望将总额翻倍（新窗口）](https://www.businessinsider.com/how-microsoft-advertising-plans-to-grow-a-20-billion-business-2022-10?r=US&IR=T)。

## **微软收集哪些数据？**

根据其广告政策，微软不会使用电子邮件、聊天或文档中的个人数据来定向广告。但弹出的广告可能是根据公司了解到的其他数据选择的 —— 例如“[您的兴趣和喜好、您的位置、您的交易、您如何使用我们的产品、您的搜索查询或您查看的内容（新窗口）](https://privacy.microsoft.com/en-us/privacystatement#mainadvertisingmodule)”。

对微软隐私政策的深入了解显示了它 [可能提取的个人数据（新窗口）](https://privacy.microsoft.com/en-US/privacystatement#mainpersonaldatawecollectmodule)：

+   姓名和联系数据

+   密码

+   人口统计数据

+   付款数据

+   订阅和许可数据

+   搜索查询

+   设备和使用数据

+   错误报告和性能数据

+   语音数据

+   文本、墨迹和输入数据

+   图像

+   位置数据

+   内容

+   反馈和评级

+   流量数据

该政策提供了一瞥 [您的数据可能流向何处（新窗口）](https://privacy.microsoft.com/en-US/privacystatement#mainpersonaldatawecollectmodule)：

+   服务提供商

+   用户指导实体

+   付款处理提供商

+   为微软执行在线广告服务的第三方

## **微软朝着数据收入迈进**

谷歌推出[扩展其收集数据权限的隐私政策（新窗口）](https://googleblog.blogspot.com/2012/01/updating-our-privacy-policies-and-terms.html)时，该公司受到监管机构和竞争对手的批评，包括微软，后者刊登了[整版报纸广告（新窗口）](https://cdn.geekwire.com/wp-content/uploads/2012/02/MICUS0004299_NYT_v2-2.pdf)，告诉谷歌用户谷歌不尊重他们的隐私。

然而不久之后，[微软发布了一个隐私政策（新窗口）](https://www.nytimes.com/2012/10/20/technology/microsoft-expands-gathering-and-use-of-data-from-web-products.html)，允许其使用个人信息销售定向广告，积极朝着曾经谴责的方向迈进。

微软自那时起一直在朝着监控收入的重要举措迈进，效仿谷歌、Facebook，以及最近的苹果。像其他大型科技公司一样，微软意识到通过收集和分析用户数据可以产生巨额收入。这种以数据为中心的思维方式已经成为已建立的公司争夺监控现金蛋糕一部分的更大趋势的一部分。

2014 年，萨蒂亚·纳德拉被任命为 CEO 标志着微软的一个转折点，同年该公司因承认[阅读记者 Hotmail 账户的电子邮件（新窗口）](https://www.theguardian.com/technology/2014/mar/21/microsoft-tightens-privacy-policy-journalists-emails)而受到审查，迫使该公司[收紧其隐私政策（新窗口）](https://www.pcworld.com/article/444479/microsoft-tweaks-privacy-policies-after-email-spying-backlash.html)。

纳德拉上任不到三个月，就发布了一份来自市场情报公司的研究，得出的结论是“利用其数据的公司有可能比不利用数据的公司增加 1.6 万亿美元的收入”，作者肖珊娜·祖博夫在她的书 [*《监控资本主义时代》*(新窗口)](https://www.theguardian.com/books/2019/oct/04/shoshana-zuboff-surveillance-capitalism-assault-human-automomy-digital-privacy) 中写道。

随后的关键发展包括必应搜索引擎和数字助手 Cortana，两者都旨在捕捉和分析用户数据。2015 年发布的 Windows 10 进一步强调了微软朝着这个新方向的承诺。来自隐私社区的审查来得很快。

软件工程师戴维·奥尔巴赫在 *Slate*（新窗口） 中写道，Windows 10“目前是一个迫切需要改革的隐私泥潭”，描述了新操作系统如何“赋予自己将大量数据传递给微软服务器的权利，使用你的带宽进行微软自己的目的，并对你的 Windows 使用进行概括。”

微软朝着广告方向的转变[延续了其 2021 年收购 Xandr](https://www.adexchanger.com/online-advertising/xandr-formerly-appnexus-is-now-formerly-att-after-its-acquisition-by-microsoft/)，但后来决定利用其封闭式花园创造的被动用户群体，将焦点转向[在其服务中首先展示第一方广告](https://www.adexchanger.com/platforms/microsoft-is-deprioritizing-third-party-ad-tech-amid-reorgs-and-layoffs/)。

鉴于这个方向，Outlook 的新形式有一定的合理性。

在[2022 年接受 Business Insider 采访时](https://www.businessinsider.com/how-microsoft-advertising-plans-to-grow-a-20-billion-business-2022-10?r=US&IR=T)，微软广告部门负责人 Rob Wilk 谈到了诸如 Xbox 这样的平台带来的机遇，其中包括游戏机业务以及已登录的账户——“我们要涉足的领域之一”，他说道。

“想象一下，不久的将来，所有这些元素都将被整合在一起，为我们的广告客户提供更清晰、更干净的选择，”Wilk 说道。“而且，别忘了，我们还拥有数十亿用户的游戏和微软 Windows 业务的浏览信息和数据——这使我们具有了理解意图的独特优势。”

Wilk 将微软的广告推广称为“[新的信仰](https://www.adexchanger.com/online-advertising/microsoft-ads-chief-rob-wilk-on-why-advertising-is-the-companys-newfound-religion/)”。

## **以利润为名的监控**

微软声称收集您的数据是为了“[为您提供丰富、交互式的体验。](https://privacy.microsoft.com/en-us/privacystatement)”

然而，在大型科技公司的领域里，广告和广告收入已成为自成体系的目标，以利润为名对您的个人数据进行监控。

随着将新 Outlook 推出为数据收集和广告投放服务，微软显露出自己与谷歌和 Meta 并无二致。对于这些公司来说，将隐私设置为默认意味着失去他们已经上瘾的收入。

还有其他公司采用的业务模式，首先关注在线安全和隐私。

Proton 就是其中之一。

## **转向真正的隐私**

Proton 使用端到端加密来保护您的电子邮件、日历、云存储的文件、密码和登录凭据，以及您的[互联网连接](https://protonvpn.com/)。我们的安全架构旨在使您的数据即使对我们也是不可见的，因为我们的商业模式让您拥有更多隐私，而不是更少。

Proton 提供免费的开源技术，以扩大在线隐私、安全和自由的访问。但您始终可以升级到付费计划以访问额外功能，这样您就可以用金钱而不是敏感数据付费。

并且 Proton 让切换变得容易。只需几个简单的步骤，您就可以迁移到一个值得信赖的电子邮件服务。

我们相信建立一个为人民服务而不仅仅是为了利润的互联网。微软和谷歌等公司为了收入而经常进行的隐私洗白只是更好互联网的又一个障碍，而在这个互联网中，隐私是默认设置。
