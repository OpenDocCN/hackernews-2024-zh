<!--yml

category: 未分类

date: 2024-05-29 12:42:03

-->

# Telegram 的点对点短信登录服务是一个隐私噩梦 | TechCrunch

> 来源：[https://techcrunch.com/2024/03/25/telegrams-peer-to-peer-sms-login-service-is-a-privacy-nightmare/](https://techcrunch.com/2024/03/25/telegrams-peer-to-peer-sms-login-service-is-a-privacy-nightmare/)

Telegram 推出了一个引起争议的新功能，为用户提供免费的高级会员资格，以允许即时通讯应用程序利用他们的电话号码作为发送一次性 SMS 密码到其他尝试登录平台的用户的中继，引发了对潜在隐私风险和个人信息泄露的担忧。

由 [TGInfoEn](https://t.me/tginfoen) Telegram 频道（通过逆向工程师 [AssembleDebug](https://twitter.com/AssembleDebug)）首先发现的功能，正在为 Android 用户的 Telegram 在一些国家推出。如果您同意让 Telegram 使用您的号码作为 OTP 中继，公司将向您发送一个可转移的代码以获取 Telegram Premium。

[点对点登录计划的服务条款](https://telegram.org/tos/p2pl) 指出，公司每月最多发送 150 条 OTP 消息。参与的用户可能还需为本地和国际短信使用支付费用，必须达到一定的配额才能获得免费订阅。从货币角度来看，通过您的电话账单可能会支付比 Telegram 高级会员更多的费用。

然后是隐私的一个巨大问题，允许陌生人查找您的号码并将其用于垃圾邮件和欺诈。Telegram 允许用户隐藏他们的电话号码不被陌生人看到，但是使用您的号码作为中继却可能允许他们查找到您的 Telegram 账户。

条款表明，参与该计划的人员不会因任何损害而追究 Telegram 的责任，并给予公司绝对的免责权，不受与点对点登录有关的所有索赔的约束。

“您承认并同意，Telegram 对于您因参与 P2PL 计划当前或过去而产生的任何成本、费用、损害或任何其他不良或其他未预见的后果概不承担责任。” 他们读到。

Telegram 告诉用户不要与接收到来自他们号码的 OTP 代码的人互动，但无法强制执行。

该公司在两年前推出了一个订阅服务，具有 [转录、独家贴纸、反应和其他自定义功能](https://techcrunch.com/2022/06/19/telegram-tops-700-million-users-launches-premium-tier/) 的特色。Telegram 为付费用户引入了 [Stories](https://techcrunch.com/2023/07/21/telegram-rolls-out-stories-feature-premium-users/) 等功能。然而，选择加入点对点登录系统的用户必须考虑，将他们的电话号码提供给陌生人以节省几个钱是否值得麻烦。
