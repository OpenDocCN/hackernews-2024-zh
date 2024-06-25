<!--yml

分类：未分类

日期：2024-05-27 15:12:06

-->

# Gmail、Yahoo 等的新邮件政策将要求启用 DMARC 的域 | easyDNS

> 来源：[`easydns.com/blog/2024/01/19/new-email-policies-at-gmail-yahoo-et-al-will-require-dmarc-enabled-domains/`](https://easydns.com/blog/2024/01/19/new-email-policies-at-gmail-yahoo-et-al-will-require-dmarc-enabled-domains/)

截至 2 月 1 日，Gmail 和 Yahoo 将开始执行**DMARC**（域基础消息认证、报告和合规性）的要求。 DMARC 是 IETF 协议，如[RFC7489](https://www.rfc-editor.org/rfc/rfc7489)中所述。

最初将影响大发件人，指的是每天在 Gmail 系统内发送超过 5K 封邮件的域（然而事情将会变得越来越困难，在没有 SPF、DKIM 和 DMARC 的情况下，你的邮件将难以被送达）。

它与**SPF**和**DKIM**一起向接收邮件服务器传达它们应该如何处理未通过 SPF 和 DKIM 验证的消息。

你应该已经在所有的发件人域上实施了 SPF —— 而且在 easyMail 上默认启用了 DKIM。如果你在自己的邮件服务器上运行，你的邮件管理员应该已经实施了它。

## SPF 和 DKIM 的快速回顾：

**SPF（[Sender Policy Framework](https://spfwizard.com/)）：**指示哪些邮件服务器可以合法地发件来自你的域名，以便更好地识别钓鱼邮件、伪造邮件和垃圾邮件。*（此外要注意，如果你的本地安装中有启用了**[发件人重写方案（SRS）](https://easydns.com/features/srs-enabled-email-forwarding)**的电子邮件转发器，则你还需要在你的 SRS 域上启用 DMARC。）*

**DKIM（[DomainKeys Identified Mail](https://easydns.com/features/dkim-domainkeys/)）：**对邮件进行加密签名，使中间站无法在你不知情的情况下修改邮件内容。

## DMARC 为其中添加了什么

在 SPF（-all、+all、~all）的最终“all”参数中表示对 SPF 失败的建议操作，但由接收服务器决定如何处理。有些忽略设置，有些发送到垃圾邮件，有些将失败的**-all**弹回，其他人则忽略它。

有了 DMARC，你向接收方传达了两件事：

1.  对于未通过 SPF 或 DKIM 验证的消息应该怎么办

1.  需要向谁报告失败

第二点很重要。在 DMARC 记录中（它们都只是 DNS TXT 记录，就像 SPF 和 DKIM 一样），你可以添加一个地址，指定 DMARC 报告应发送回哪个电子邮件。

## DMARC 报告流向邮件管理员

使用这些报告，您可以监控您的电子邮件来源 - 也许有一个博客服务器、一个 CRM 或一个您忘记包含到您的 SPF 中的列表服务 - 并且一直以来这些消息都在大多被路由到垃圾邮件文件夹，因为它们未能通过 SPF 检查。启用 DMARC 后，您将在报告中找到这些信息 - 并且能够相应地调整您的 SPF 记录。

DMARC 包含一个参数，用于指导您的域的入站电子邮件流量的多少如果它们未通过检查则进行隔离。您甚至可以从“none”（p=none）开始，而推荐的部署方法是从该值开始。

报告会流向您指定的地址 - 通常以 xml 格式和 .zip 格式。一旦启用 DMARC，您会收到大量报告，因此使用您自己的电子邮箱之一来接收它们是一个坏主意。

+   您将不得不创建一个单独的电子邮箱来接收这些报告（我们正在考虑将一个 postmaster@ 电子邮箱与 easyMail 域一起捆绑作为我们实施的一部分）

+   您可能希望使用一个 [DMARC 解析服务](https://partners.easydmarc.com/9sqbo9k9idd8) 像 **easydmarc** 来理解所有这些报告（我不是在开玩笑，他们与我们无关，但我们现在使用他们并与他们合作） 。如果您有技术能力，还有一些开源的 DMARC 分析包，如 **[parsedmarc](https://domainaware.github.io/parsedmarc/)** （如果您是开发人员，请考虑在他的 Github 上提出一些问题，一个人的表演可能需要帮助）。

## DMARC 快速开始：

我们推荐的 DMARC 初始部署是只需像这样向您的区域添加一个 TXT 记录：`_dmarc IN TXT "v=DMARC1;p=none;rua=mailto:dmarc@example.com;ruf=mailto:dmarc@example.com;rf=afrf"`

而满足 Google 服务器的最低可行 DMARC（MVD）推荐记录是：

`_dmarc IN TXT "v=DMARC1;p=none;"`

请注意，此记录在您的区域中的位置与 SPF 稍有不同：

+   您的 SPF TXT 记录位于“@”，即您的区域顶点

+   DMARC 记录是有范围的记录：“_dmarc”

在上述最小记录（以及至少一个 SPF 记录）放置在合适位置后，将使您符合新政策，并保持您的电子邮件投递到 Gmail 和雅虎等地方。如果您到目前为止还没有 SPF 记录，这甚至可能会改善它。

## 在 easyMail 中部署 DMARC

正如我所提到的，我们将在新的独立 easyMail 信箱上添加 postmaster@ 账户（很快会有公告）。

在接下来的几周里，我们将测试所有 easyMail 域名的 DMARC 记录，并在没有记录的情况下通知您 - 到了二月份，您将需要这个记录，否则您发送到 Gmail（以及雅虎等地方）的可投递性将会急剧下降。

## 进一步阅读和下一步：

如果您从未设置过 SPF 记录，无论如何都应该这样做：

请关注此处以获取我们的 DMARC 文档和部署的链接。
