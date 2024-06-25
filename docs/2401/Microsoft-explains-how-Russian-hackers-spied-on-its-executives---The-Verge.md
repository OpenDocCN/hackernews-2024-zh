<!--yml

分类：未分类

日期：2024-05-27 15:17:41

-->

# 微软解释俄罗斯黑客如何监视其高管 - The Verge

> 来源：[https://www.theverge.com/2024/1/26/24051708/microsoft-hack-russian-security-attack-senior-leadership-emails](https://www.theverge.com/2024/1/26/24051708/microsoft-hack-russian-security-attack-senior-leadership-emails)

微软上周透露，俄罗斯政府赞助的黑客对其公司系统发动了一次国家级攻击，这些黑客曾发动了 [SolarWinds 攻击](/2020/12/13/22173035/hackers-russia-breached-us-government-agencies-email-cozy-bear)。黑客能够访问一些微软高管的电子邮件帐户，潜在地监视他们长达数周甚至数月。

尽管微软在其周五的初始 SEC 披露中没有提供有关攻击者如何获得访问权限的详细信息，但这家软件制造商现在已经 [发布了一份初步分析](https://click.linksynergy.com/deeplink?id=nOD/rLJHOac&mid=24542&murl=https%3A%2F%2Fwww.microsoft.com%2Fen-us%2Fsecurity%2Fblog%2F2024%2F01%2F25%2Fmidnight-blizzard-guidance-for-responders-on-nation-state-attack%2F) ，介绍了黑客是如何越过其安全措施的。它还警告说，同一黑客组织，被称为 Nobelium 或 “午夜暴风雪” [以天气为主题](/2023/4/19/23689456/microsoft-weather-cybersecurity-threat-actors-naming) 的微软所称呼的，一直在针对其他组织进行攻击。

Nobelium 最初通过密码喷射攻击访问了微软的系统。这种类型的攻击是一种暴力攻击，黑客使用潜在密码字典针对账户。关键是，遭到侵犯的非生产测试租户帐户未启用双因素身份验证。微软表示，Nobelium “定制了他们的密码喷射攻击，针对少量账户使用较少的尝试次数以避免被检测到。”

该组织“利用其初始访问权，识别并攻破了一个具有对微软企业环境有提升访问权限的遗留测试 OAuth 应用程序。” OAuth 是一种广泛使用的基于令牌的身份验证的开放标准。它通常用于网站，允许您登录应用程序和服务，而无需向网站提供密码。想想您可能会使用 Gmail 账户登录的网站，那就是 OAuth 的作用。

此提升的访问权限允许该组织创建更多恶意 OAuth 应用程序，并创建帐户以访问微软的企业环境，最终访问提供对电子邮件收件箱的访问权限的 Office 365 Exchange Online 服务。

“午夜暴风雪利用这些恶意 OAuth 应用程序进行身份验证，以访问 Microsoft Exchange Online 并针对微软企业电子邮件帐户，”微软的安全团队解释道。

微软没有透露有多少公司电子邮件账户被攻击和访问，但是公司之前称其为“微软公司电子邮件账户的非常小比例，包括我们的高级领导团队成员和我们的网络安全、法律和其他职能的员工。”

微软也没有透露黑客在多长时间内一直在监视其高级领导团队和其他员工。初次攻击发生在 2023 年 11 月底，但微软直到 1 月 12 日才发现。这意味着攻击者可能在近两个月内一直在监视微软高管。

惠普企业（HPE）[本周早些时候透露](https://www.sec.gov/ix?doc=/Archives/edgar/data/0001645590/000164559024000009/hpe-20240119.htm)同一组黑客此前已经访问过其“基于云的电子邮件环境”。HPE 没有透露提供商的名称，但公司透露，该事件“很可能与从 2023 年 5 月起开始的‘有限数量的[微软]SharePoint 文件泄露’相关。”

微软遭受攻击的时间正好是公司宣布其[计划改革软件安全](/2023/11/2/23943178/microsoft-security-secure-future-initiative-cybersecurity)后的几天，此前发生了重大的 Azure 云攻击。这是继 2021 年有 3 万家组织的[电子邮件服务器被黑客攻击](/2021/3/5/22316189/microsoft-exchange-server-security-exploit-china-attack-30000-organizations)后，再次打击微软的最新网络安全事件，原因是微软 Exchange Server 的漏洞，以及去年中国黑客通过微软云漏洞[侵入美国政府的电子邮件](/2023/7/12/23792371/security-breach-china-us-government-emails-microsoft-cloud-exploit)。微软也是巨型 SolarWinds 攻击的中心，这次攻击发生在近[三年前](/2020/12/13/22173035/hackers-russia-breached-us-government-agencies-email-cozy-bear)，是由同一组织 Nobelium 执行的，该组织也是此次令人尴尬的高管电子邮件攻击的幕后黑手。

微软承认一个显然是关键测试帐户缺乏两步验证的事实可能会引起网络安全社区的关注。虽然这不是微软软件的漏洞，但是一组配置不良的测试环境使得黑客可以悄悄地在微软的企业网络中移动。“非生产测试环境是如何导致微软最高级别官员的妥协的？”CrowdStrike CEO George Kurtz 在本周早些时候接受 [CNBC 的采访](https://www.cnbc.com/video/2024/01/22/crowdstrike-ceo-george-kurtz-on-microsoft-hack-and-what-it-means-for-cybersecurity-landscape.html)时问道。“我认为还会有更多的事情会被曝光。”

Kurtz是正确的，更多内容已经出现，但仍然缺少一些关键细节。微软确实声称，如果今天部署了相同的非生产测试环境，则“强制性的微软政策和工作流程将确保启用MFA和我们的主动保护”，以更好地防御这些攻击。微软仍然有很多解释的工作要做，特别是如果它希望其客户相信它真正正在改进设计、构建、测试和运营软件和服务的方式，以更好地防范安全威胁。
