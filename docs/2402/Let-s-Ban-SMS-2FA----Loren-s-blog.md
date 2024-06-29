<!--yml

分类：未分类

日期：2024-05-29 13:24:18

-->

# 让我们禁止短信双因素认证 // Loren的博客

> 来源：[https://lorendb.dev/posts/lets-ban-sms-2fa/](https://lorendb.dev/posts/lets-ban-sms-2fa/)

我说过了。

现在我已经说清楚了，让我们谈一谈为什么我们实际上应该禁止短信双因素认证。

> 一个快速说明：我重新撰写了这篇文章，以呈现更加平衡的观点。你可以在互联网档案馆找到旧版本。

## 问题

短信双因素认证本身就有些自相矛盾。双因素认证通常被定位为增加您的安全性的一种方式；然而，[SMS消息](https://en.wikipedia.org/wiki/SMS)本身是不安全的，因为它是未加密的，而且SMS消息是通过无线方式公开广播的。这意味着身边的坏人可以轻松窥视发送给你的2FA代码。

然而，在这里，未加密的消息广播并不是唯一的问题。[SIM交换](https://en.wikipedia.org/wiki/SIM_swap_scam)是另一种方法，可以允许攻击者访问你的短信消息；在这种攻击中，攻击者打电话给你的手机公司，声称他们是你，要求将你的电话号码转移到他们的SIM卡上。尽管FCC正在[采取行动打击SIM交换](https://www.theverge.com/2023/7/11/23791183/fcc-sim-swapping-port-out-phone-hijacking-security-protection)，但这种攻击仍然非常相关；事实上，上个月（2024年1月），美国证券交易委员会的Twitter账号在SIM交换攻击中被[黑客攻击](https://www.techradar.com/pro/sec-reveals-how-its-twitter-account-was-hacked-and-its-rather-embarrassing)。

说实话，SIM交换并不是一种简单的攻击。它通常需要社会工程学来完成，这使得它对于路过的攻击者来说不那么有用。然而，[eSIM交换](https://www.bleepingcomputer.com/news/security/sim-swappers-hijacking-phone-numbers-in-esim-attacks/)开始流行起来。这种攻击完成起来不那么复杂，因为它不需要社会工程学来操作电话公司。你只需要获取受害者的登录凭据，登录他们的手机提供商账户，然后交换eSIM。

## 公司并不在乎

你可能会想知道为什么这是个如此大的问题。当然，许多网站提供短信作为双因素认证的方法，但你并不需要启用它，对吧？然而，事实证明像PayPal和Amazon这样的平台并不关心短信双因素认证的不安全性。如果你想为你的账户设置任何形式的双因素认证，你就被迫设置短信作为后备选项。这使得它们的双因素认证支持几乎成了一个笑话，因为任何攻击你的Amazon或PayPal账户的人理论上都可以通过截取你的短信来覆盖你的双因素认证。他们的理由是？“如果你丢失了你的双因素认证设备，我们会给你发一条短信验证码来恢复访问你的账户。”

(关于账户恢复的更多信息：只要提供有效的电子邮件地址，PayPal会很乐意发送短信来验证密码重置，而Amazon则允许您仅通过电话号码重置密码。这意味着只要您知道某人拥有与其电话号码关联的Amazon账户，拦截其短信就可以完全访问他们的Amazon账户。)

即使对于那些没有强制要求SMS双因素认证的服务，SMS仍然作为许多网站的选项呈现。它的易用性无疑导致许多人说：“哦，我只需使用SMS因为更方便”，这使得这些人更容易受到攻击。

说到这一点，我必须赞扬Twitter。在2023年，他们[停用了](https://blog.twitter.com/en_us/topics/product/2023/an-update-on-two-factor-authentication-using-sms-on-twitter)未订阅Twitter Blue的用户的SMS双因素认证，称其不安全为理由。我更希望Twitter彻底禁止SMS双因素认证，但即使他们将其作为鼓励用户购买Twitter的一个理由，这仍然是一个受欢迎的改变。

## 修复起来并不是那么难

现在，有些人可能会开始抱怨改变他们的SMS双因素认证部署将是一个巨大的技术问题。不，不会的。公司在这里有几个选项：

+   使用[基于时间的一次性密码（TOTP）](https://en.wikipedia.org/wiki/Time-based_one-time_password)；添加TOTP支持将非常简单，因为它只需要算法来基于原始密钥值计算代码（您可能还想加入一个QR码生成器，但这在技术上是可选的）。而且，您不需要自己实现；GitHub上有许多[基于TOTP的库（和应用程序）](https://github.com/topics/totp-generator)。这确实需要用户安装一个TOTP应用程序，但有许多值得信赖的TOTP应用程序可供选择，因此这不应该是一个大问题。

+   使用[通行证](https://www.passkeys.com)，这是最新和最优秀的双因素认证安全技术。通行证（据我所知）类似于软件定义的WebAuthn密钥，其操作方式类似于[硬件安全密钥](https://www.yubico.com/product/security-key-series/security-key-c-nfc-by-yubico-black/)（顺便说一句，显然Yubico停止生产蓝色版本的了）。它们受到现代浏览器和手机操作系统的支持，还有像Bitwarden这样的密码管理器，通常设计得非常简单易用。用户通常不需要安装额外的应用程序来使用通行证。

+   如果你的2FA流程被硬编码为必须使用类似短信的流程，其中你向用户发送包含一次性代码的消息，用户必须将其回传给你，你至少可以用其他服务替换SMS。一些平台支持电子邮件，但你也可以使用端到端加密的[Matrix](https://matrix.org)聊天，或者甚至可以像[GitHub](https://github.blog/2022-01-25-secure-your-github-account-github-mobile-2fa/)那样在你的应用程序中嵌入2FA发送服务。如果你真的无法放弃SMS，至少尝试在可能的情况下将消息升级到[RCS](https://en.wikipedia.org/wiki/Rich_Communication_Services)。（如果你正在使用Twilio Verify，则已自动升级到[RCS](https://www.twilio.com/docs/verify/rcs)）。

## 我们可以采取的行动

如何应对短信两步验证（SMS 2FA）？事实证明，我们可以采取许多意想不到的措施。

1.  对于任何可以关闭的帐户，请禁用SMS 2FA。显然，你必须保留Amazon和PayPal的SMS 2FA，但至少尽可能减少使用SMS 2FA。

1.  给强制使用SMS 2FA的公司写信，要求他们移除它。

1.  告诉他人关于SMS 2FA的风险。了解这一风险的人越多，其中一人成为SIM卡交换的受害者的机会就越小。

1.  如果你在部署SMS 2FA的公司工作，请要求他们从产品中移除它。如果你在PayPal、Amazon或其他可能成为攻击目标的地方工作，尤其如此。这包括银行（是的，我的银行强制使用SMS 2FA，我对此并不满意）。

1.  如果你居住在美国，写信给你的国会议员，敦促他提出立法禁止使用未加密的基于消息的2FA（并提到SEC的Twitter黑客事件，以显示政府也容易受到这种攻击的影响）。如果你不在美国居住，请写信给代表你的政府的人（例如，欧盟居民可以写信给他们的欧洲议会议员）。尽管这可能看起来有些不太可能，但你的信息可能会导致真正的法律出台。欧盟居民尤其有可能成功，鉴于欧盟在立法理性技术法律方面的历史立场（即[GDPR](https://en.wikipedia.org/wiki/General_Data_Protection_Regulation)和[DMA](https://en.wikipedia.org/wiki/Digital_Markets_Act)）。

1.  分享这篇文章，或者如果你有自己的博客，请就此问题撰写自己的文章！这个问题得到的公众关注越多，越好。

## 我稍微退缩的那一部分

尽管我说了这么多，短信两步验证完全邪恶吗？答案实际上很复杂。首先，虽然短信两步验证不安全，但比没有两步验证要好，因为它可以作为对一般黑客的威慑。其次，它很用户友好。与TOTP、密码密钥和物理安全密钥相比，简单的短信验证方案更容易学习使用。要成功地取消短信两步验证，我们需要教育大众如何使用替代方案，或者创建一个真正简单易用的替代方案。我认为密码密钥可能会成为一个好的解决方案，因为设备和浏览器制造商已经付出了很多努力，使它们成为一个无缝的选项。

## 结论

希望你喜欢阅读这篇小小的抱怨。希望你能因此受到启发，以某种方式采取行动来帮助解决短信两步验证的问题。如果你给你的政府代表写了一封信，或者在你的公司开始了一场讨论，试图取消短信两步验证，请在下方留言！如果这篇文章引起了足够的关注，人们开始记录他们的行动，我将创建一个页面来追踪我们所有的短信两步验证活动，以帮助监控这个情况。

# 评论

由 [仙人掌评论](https://cactus.chat) 提供动力
