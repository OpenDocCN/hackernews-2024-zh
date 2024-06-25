<!--yml

category: 未分类

date: 2024-05-27 14:40:20

-->

# Gmail和雅虎收件箱要求及对发送者的影响 | Mailgun

> 来源：[https://www.mailgun.com/blog/deliverability/gmail-and-yahoo-inbox-updates-2024/](https://www.mailgun.com/blog/deliverability/gmail-and-yahoo-inbox-updates-2024/)

为了保护收件箱，Gmail和雅虎表示将对批量邮件发送者执行新的保护标准。

谷歌，已于2023年5月宣布对不活跃帐户进行清理的行动，[在一份声明中解释](https://blog.google/products/gmail/gmail-security-authentication-spam-protection/)，**强制执行将于2024年2月开始**，他们将仔细监控批量发送者（即每天发送5000条以上的发送者）。 在一份非常相似的声明中，[雅虎解释说](https://blog.postmaster.yahooinc.com/post/730172167494483968/more-secure-less-spam) ，他们也将瞄准2024年第一季度。

尼尔·库马兰，Gmail安全与信任产品组经理

这些即将到来的要求无疑是我们近年来看到的邮箱提供商强制执行的最实质性的要求，但它们并不是新的。 事实上，认证、一键退订和垃圾邮件监控已经是[电子邮件交付的最佳实践](/blog/email/understanding-email-deliverability/)清单上的重要内容已经有一段时间了。

尽管这并不奇怪，但这引起了一些发送者的担忧，在Sinch Mailgun，我们怀疑这些要求很快会在整个行业传播开来。

对于这个问题最直接的答案是，如果您还没有做到，那就需要认真对待某些电子邮件交付的最佳实践。

Gmail和雅虎都强调了2024年来的三项关键变化，如果发送者想被视为合法发送者，他们就需要优先考虑这些变化：

1.  **验证他们的电子邮件：** 发送者将需要使用标准协议如SPF、DKIM和DMARC验证他们的发件人身份。

1.  **启用一键退订：** 发送者需要在邮件中实施单击退订链接，以允许收件人轻松退出，如果他们还没有的话。

1.  **只发送用户想要的邮件：** Gmail和雅虎对垃圾邮件监控变得严肃，发送者需要确保它们保持在设定的垃圾邮件率阈值以下。

这些命令只会影响批量发送者，由谷歌定义为一天发送至Gmail地址5000条或更多消息的发送者。 公告并未指明发送者必须每天发送5000条*每*天，或在一定时间范围内发送多少条消息。 因此，**在检查这些规则是否适用于您时，考虑到您的节假日发送习惯和大型活动是很重要的。** *您*可能不认为自己是批量发送者，但邮箱提供商可能会持不同意见。

正如我们之前提到的，这些要求并不是我们发送电子邮件的方式中意外或革命性的改变，但是仍然有许多发件人没有遵循。例如，多年来一直强烈建议进行电子邮件认证。然而，我们的[电子邮件投递状况报告](/resources/research/state-of-deliverability-2023/)发现，大约有 40% 的发件人不确定或未实施 SPF 和 DKIM，而在使用 DMARC 的发件人中，40% 不确定他们的策略是什么。

Neil Kumaran，Gmail 安全与信任团队产品经理

好消息是，这两家提供商在其声明中都强调了类似的更新，主要集中在保持更高的认证标准、简化取消订阅推广电子邮件和将发件人的垃圾邮件率阈值降低。以下是您可以期待的内容的简要摘要。

因此，现在轮到电子邮件发送者在 2024 年之前做好准备了。您需要做出哪些更改以确保您的电子邮件仍然能够被投递到收件箱中？您如何实施这些更改？

以下是详细清单：

[电子邮件认证](/blog/deliverability/email-authentication-your-id-card-sending/https:/www.mailgun.com/blog/deliverability/email-authentication-your-id-card-sending/)是通过证书和加密确保和确认您的发件人身份的过程。其目的是保护您的身份免受欺骗，并保护您的收件人免受网络钓鱼攻击。这就是为什么**Gmail 和 Yahoo 更新着重于验证您的发件人身份**。在[2022 年](https://support.google.com/accounts/answer/6010255)，Gmail 开始要求发件人采用*某种形式*的认证，这导致 Gmail 用户接收到的未经身份验证的消息下降了[75%](https://blog.google/products/gmail/gmail-security-authentication-spam-protection/)。但是像垃圾邮件发送者、网络钓鱼者和恶意软件这样的复杂问题需要同样复杂的解决方案。

Gmail 对于批量发送者的第一个任务是，他们要按照这些[最佳实践](https://support.google.com/a/answer/174124)对其电子邮件进行身份验证。来自 Gmail 和 Yahoo 的要求是为您的域设置强大的认证，包括“SPF、DKIM 和 DMARC”。此前不是必需的，但是向实施基于域的消息认证、报告和合规性（DMARC）迈出的这一步，正如 Sinch Mailgun 的 Jonathan Torres 在我们关于电子邮件安全与合规性指南中所预测的那样。

“在某个时候，邮箱提供商可能会决定优先处理那些设置了拒绝或隔离的 DMARC 策略的发件人的消息，因为它们是可以验证和信任的发件人。我们还没有看到任何人采取这一步，但是已经为要求发件人设置 DMARC 策略至少不是 p=none 做好了准备。这可能是推广采用的关键。”

Jonathan Torres，Mailgun 的 TAM 团队经理

我们的建议是，如果您是批量发送者，请**设置所有三种身份验证**以保护您的发件人身份和可交付性。以下是具体步骤。

| **Gmail：** Gmail要求同时使用SPF和DKIM。不带有这些协议的消息将被拒收或标记为垃圾邮件。还需要DMARC来防止Gmail中的冒名顶替。 | 如果您是Mailgun用户，我们已经为您提供了SPF和DKIM的支持。但如果您不是，我们已经在以下文章中概述了获取这些身份验证的过程：[SPF基础知识](https://www.mailgun.com/blog/deliverability/spf-records-basics/)和[了解DKIM](https://www.mailgun.com/blog/deliverability/understanding-dkim-how-it-works/)。[对于DMARC，您至少需要设置p=none策略。](https://www.mailgun.com/blog/deliverability/email-authentication-your-id-card-sending/#chapter-3) |
| --- | --- |
| **雅虎：** 将要求使用强身份验证，并要求用户“利用行业标准，如SPF、DKIM和DMARC”。 | 实施DMARC需要更多时间，因为DMARC允许您根据您的电子邮件程序对政策进行选择。立即开始，查看我们的[实施DMARC](https://www.mailgun.com/blog/deliverability/implement-dmarc/)文章。 |

为联系人提供明显可见的退订原因已经有了充分的理由，而且在电子邮件消息的页脚文本中提供一个退订链接已经是普遍实践，但这并不是该要求的重点所在。

向不想收到消息的用户发送消息会严重影响您的参与度指标和垃圾邮件比率，并最终对您的整体声誉造成不利影响。在我们的播客《[电子邮件未死](/resources/podcasts/marcel-becker-yahoo/)》中，我们与雅虎的高级产品经理马塞尔·贝克尔（Marcel Becker）进行了对话，并就雅虎的新要求向他提出了许多问题。

您可以在我们的[关键收获文章](/blog/deliverability/yahoo-requirement-insights-with-marcel-becker/)中了解更多，但这里有一个预览：如果在邮箱UI内可见退订选项，用户更有可能取消订阅消息。许多用户发现将消息移到垃圾邮件文件夹比滚动到电子邮件底部并完成多步骤流程更快更容易。

未来，一键退订链接将变得更加重要。从2024年开始，Gmail和Yahoo都要求发件人提供单击退订的过程，而不是确认您的电子邮件或更新订阅偏好并提供反馈。发件人有两天的时间来执行退订请求。

| **对于 Gmail 和 Yahoo 相同：** 一个单击路径供用户从邮箱提供商的界面内轻松取消订阅你的邮件，并在 2 天内从相关邮件列表中移除地址的内部支持来尊重取消订阅请求。 | 发件人需要按照[RFC 8058](https://www.rfc-editor.org/info/rfc8058)中指定的方式将列表取消订阅的后部头部放入其电子邮件的头部。 |
| --- | --- |

如何从用户的收件箱中消除垃圾邮件？设置一个低的垃圾邮件率阈值并告诉发件人他们不能超过它。

对于雅虎和Gmail，策略是相同的，垃圾投诉率阈值都是 0.3%。当你考虑到许多电子邮件服务提供商（ESPs）和独立公司已经有保持垃圾邮件率低于 0.1% 或每发送 1000 封邮件中有一封被标记为垃圾邮件的内部实践时，这可能听起来像是一个非常低的百分比。

Marcel Becker，雅虎产品管理高级总监。

你的垃圾邮件率，或者[垃圾投诉率](/blog/deliverability/spam-complaint-rate/#chapter-1)，是将你的邮件报告为垃圾邮件的收件人数量与发送的总邮件数量相比较的比率。**保持这个数字低的最佳方法是监控**，在失去兴趣的订阅者受到诱惑按下垃圾邮件按钮之前将其退出，并且迅速响应垃圾投诉率的任何激增，方法是[清理你的列表](/blog/deliverability/importance-of-email-list-cleaning/)并审查你的发送实践。

请记住，Gmail 不像 Yahoo 提供传统的反馈回路，所以你需要确保你已经注册了[Google Postmasters Tools](/blog/deliverability/google-postmaster-tools-understanding-sender-reputation/)来监控你的垃圾邮件率。

| **对于 Gmail 和 Yahoo 相同：** 垃圾邮件投诉阈值为 0.3%。 | 使用诸如[Goo­gle Post­masters Tool­s](https://www.mailgun.com/blog/deliverability/google-postmaster-tools-understanding-sender-reputation/)等资源密切监控你的垃圾邮件率，以及其他参与度指标。采用[列表管理](https://www.mailgun.com/blog/deliverability/how-to-manage-your-email-list/)和[失效策略](https://www.mailgun.com/blog/deliverability/sunset-policies-unengaged-recipients/)等交付最佳实践来优化你的邮件列表，确保你只向参与的收件人发送邮件。使用[交付工具](https://www.mailgun.com/products/inbox/)如电子邮件验证和收件箱放置测试来掌握你的整体交付能力并提高你的收件箱放置。 |
| --- | --- |

在 Sinch Mailgun，电子邮件交付的卓越性始终是我们产品提供的核心。我们不断努力为用户提供交付成功所需的帮助，确保您获得所需的帮助。其中一部分努力是把合适的人放在一起，这样我们就可以提供最准确的信息。在这种精神下，Sinch Mailgun 的交付副总裁 Kate Nowrouzi 与 Yahoo 的高级产品经理 Marcel Becker 和 Google 的反滥用和安全产品总监 Anu Yamunan 坐下来，进行了一次炉边谈话，回答了一些关于这些发件人要求的最常见问题，并找出了背后的原因。

点播网络研讨会

查看我们与 Yahoo 高级产品总监 Marcel Becker、Google 反滥用和安全产品总监 Anu Yamunan 以及 Sinch Mailgun 交付副总裁 Kate Nowrouzi 的炉边谈话，探讨大宗电子邮件发送者的新要求。

[](https://sinch.registration.goldcast.io/events/8658af11-5b57-47c2-a1a5-a6a5bffd8a07)

Mailgun 用户可以放心，他们的电子邮件身份验证协议已经符合 Gmail 和 Yahoo 的要求，因为我们的平台默认会自动强制执行 SPF 和 DKIM。有关更多资源，我们已经准备了一个 [包含您所需的所有内容的库和检查表。](https://try.mailgun.com/googleyahooupdates/)

我们还提供一整套创新的交付工具和服务，旨在使这些保护易于实现。我们的 [**Mailgun 优化交付工具包**](/blog/deliverability/email-deliverability-tools/) 包括用于测试、监控和分析各种重要电子邮件交付要素的优秀工具。

+   [电子邮件验证](/products/inbox/email-verification/)有助于在发送之前从您的列表中删除高风险和无效地址，以帮助减少退信率并保护您的声誉。

+   [收件箱放置测试](/blog/product/inbox-placement-beta-announcement/)显示您的电子邮件最有可能在哪个文件夹或选项卡中落入主要提供商（如 Gmail 和 Yahoo）中，并有助于主动测试您的身份验证状态。

+   [Google 邮局大师集成](/blog/deliverability/google-postmaster-tools-understanding-sender-reputation/) 显示用户报告的垃圾邮件率和其他重要统计数据，如向 Gmail 用户发送电子邮件的 DMARC 的身份验证状态监控。

+   [退信分类](/products/inbox/deliverability/blocklist-monitoring-service/) 帮助您识别由于您的发件人声誉而可能发生的重要退信。

这些工具最终使得您可以轻松掌握您的电子邮件性能。

想要在这些变化中为您的业务提供额外支持吗？查看我们的[可达性服务！](/products/inbox/deliverability-services/)我们拥有一支专业团队，共计超过320年的电子邮件经验，随时为您的公司提供帮助，帮助您应对不断变化的行业标准，并实施最适合您电子邮件需求的定制策略。

了解我们的可达性服务

想要发送大量电子邮件吗？我们的电子邮件专家可以提升您的电子邮件性能。看看我们如何帮助像Lyft、Shopify、Github这样的公司将其电子邮件投递率提高到平均97%。

[](/products/optimize/deliverability-services/)
