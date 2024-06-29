<!--yml

category: 未分类

date: 2024-05-27 13:05:52

-->

# Microsoft员工在安全漏洞中泄露了内部密码 | TechCrunch

> 来源：[https://techcrunch.com/2024/04/09/microsoft-employees-exposed-internal-passwords-security-lapse/](https://techcrunch.com/2024/04/09/microsoft-employees-exposed-internal-passwords-security-lapse/)

Microsoft已解决了一个安全漏洞，使得内部公司文件和凭证暴露在公开互联网上。

安全研究人员Can Yoleri、Murat Özfidan和Egemen Koçhisarlı与SOCRadar合作，这是一家帮助组织发现安全弱点的网络安全公司，他们发现了一个托管在Microsoft Azure云服务上的公开存储服务器，其中存储了与Microsoft的Bing搜索引擎相关的内部信息。

Azure存储服务器存放了包含Microsoft员工用于访问其他内部数据库和系统的密码、密钥和凭证的代码、脚本和配置文件。

但存储服务器本身未受密码保护，任何人都可以访问它。

Yoleri告诉TechCrunch，暴露的数据可能有助于恶意行为者识别或访问Microsoft存储其内部文件的其他位置。识别这些存储位置“可能导致更严重的数据泄露，并可能危及正在使用的服务，”Yoleri说。

研究人员于2月6日通知了Microsoft有关这一安全漏洞，Microsoft在3月5日安全地保护了泄露的文件。

在电子邮件联系时，Microsoft的发言人在发布时未作评论。在周三发布后的声明中，Microsoft的Jeff Jones告诉TechCrunch：“虽然凭证不应该被暴露，但它们是临时的，仅可从内部网络访问，并在测试后被禁用。我们感谢我们的合作伙伴负责报告这个问题。”

Jones并未透露云服务器暴露在互联网上的时间长短，或者除了SOCRadar之外是否有其他人发现了内部数据的暴露情况。

这是Microsoft在一系列云安全事件之后试图重建与客户信任的最新安全疏漏。在去年类似的安全漏洞中，研究人员发现[Microsoft员工在发布到GitHub的代码中暴露了他们自己的企业网络登录信息](https://www.vice.com/en/article/m7gb43/microsoft-employees-exposed-login-credentials-azure-github)。

去年，微软也因为公司承认不知道[中国支持的黑客如何窃取了内部电子邮件签名密钥](https://techcrunch.com/2023/09/08/microsoft-hacker-china-government-storm-0558/)而备受指责，这使得黑客能够广泛访问美国高级政府官员的微软托管收件箱。一组独立的网络安全专家委员会负责调查电子邮件泄露事件的报告上周发布，写道，黑客之所以成功，是因为微软存在一系列“安全失误的连锁反应”。

三月份，微软表示正在应对[持续的网络攻击](https://techcrunch.com/2024/03/08/microsoft-ongoing-cyberattack-russia-apt-29/)，俄罗斯政府支持的黑客已经窃取了公司源代码的部分和微软公司高管的内部电子邮件。

*更新：微软发表了评论。*

> [微软揭示黑客如何窃取其电子邮件签名密钥… 有点像](https://techcrunch.com/2023/09/08/microsoft-hacker-china-government-storm-0558/)
