<!--yml

category: 未分类

date: 2024-05-27 13:23:05

-->

# “去中心化”身份系统的危险性 — 加密阿哥利主义

> 来源：[https://paper.wf/crypto-agorism/the-dangers-of-decentralized-id-systems](https://paper.wf/crypto-agorism/the-dangers-of-decentralized-id-systems)

许多去中心化身份协议正在开发中，它们声称能增加用户的隐私保护，实现互操作性和便捷的单一登录，保护免受身份盗窃，允许数据的自主所有权。

然而，许多这些协议依赖政府身份证作为基础层（作为姓名、年龄或地址的证明，称为“可验证凭据”）。在这个系统中，用户需要上传带有护照或国民身份证的视频。之后，他们的姓名、年龄或地址将被标记为已验证。然后，平台可以查询此 API，并询问 is_over_18、full_name 或 country_of_residence，但不能访问用户的身份证扫描或任何其他信息（例如 is_over_18 只返回 true 或 false，并不透露用户的姓名、家庭地址或照片）。

对政府身份证的依赖意味着去中心化身份无法保护用户免受国家监视的侵害。就像现有系统一样，它继续排斥数百万无法获得政府身份证的人群：[https://www.statelessness.eu/blog/each-person-left-living-streets-we-are-losing-society](https://www.statelessness.eu/blog/each-person-left-living-streets-we-are-losing-society), [https://unhcr.org/ibelong/about-statelessness](https://unhcr.org/ibelong/about-statelessness), [https://www.penalreform.org/blog/proving-who-i-am-the-plight-of-people](https://www.penalreform.org/blog/proving-who-i-am-the-plight-of-people)

## 问题 1：依赖政府身份证作为基础层

如果去中心化身份仅仅是现有政府身份系统的延伸，它既不能提供隐私保护，也无法实现金融包容。

通过政府身份证 KYC，国家已经将普通人排除在工作、银行、公寓租赁、医疗保健、接收邮件、SIM 卡、合同等领域之外。

如果国家拒绝为某人打印身份证（这今天影响了数百万人），没有上诉途径、替代方案或能提供帮助的非政府组织。例如红十字会、联合国及其他非政府组织并不发行替代身份文件。旗帜理论（如圣基茨护照、巴拿马居留、爱沙尼亚电子居留或 RNS.id）要求存在护照或出生证明。即使是未经证明的人（如加州 AB 60 驾照持有者）也需要外国护照、国民身份证或出生证明，并且无法帮助那些没有任何国家发行身份文件的人。

这种现有的身份证系统是有害的、难以访问的，并且是单一故障点 — 如果去中心化协议依赖这种破碎的基础层，它们将继续伤害和排斥人群。

## 问题 2：国家不会放弃对身份的垄断

幸运的是，一些去中心化身份协议的目标是包容的，而不是要求政府身份证来验证用户的姓名、年龄或位置，它们使用社交媒体、信任网络或生物特征。这消除了国家审查的能力，而是允许你的朋友为你作证，或者允许你通过指纹或虹膜扫描访问服务。

有了信任网络，朋友或家人可以为你的姓名、年龄或位置作证；房东可以为你的地址作证；雇主可以为你的技能作证；顾客可以为企业作证；等等。因为它不依赖政府数据库，而是依赖你认识的人，所以它是真正分散化和可访问的。

生物特征也不依赖于国家的许可。如果你有手指或眼睛，你可以通过指纹或虹膜扫描注册 —— 不需要护照或国民身份证。由于它不依赖于国家发行的文件，生物特征将对无国籍人士、无文件人士以及未在出生时注册的人士（通常被不公正地排除在主流经济之外）具有可访问性。然而，生物特征因多种原因而危险，包括安全性（有人可能迫使你不愿意地提供你的指纹或从照片解码你的虹膜图案）、个人安全（例如逃离家庭暴力或抗议专制政府）、以及隐私（如工作和家庭生活的自然隔离以及在线身份）。

不幸的是，国家通常强制企业、雇主、房东和医疗服务提供者遵守政府身份证规定，不太可能接受信任网络作证或生物特征作为“身份证明”的证据。因此，使用“Worldcoin”虹膜扫描申请工作，或仅基于信任网络中的正面评价租房，将是不可能的。

国家特别使用其政府身份证系统在出生时列入白名单（如果你没有在出生时注册，成年后就没有“赚取存在权”的方法），而移民则依赖于其他国家的白名单（没有护照和出生证明就不可能获得签证）。国家不允许人们通过提供指纹或要求朋友为他们作证来绕过此白名单。

如果国家选择将生物特征或信任网络纳入其身份系统，它将按照自己的条款进行，作为一种补充而不是替代：信任网络平台将要求现有的政府身份证才能注册，并且“Worldcoin”钱包将需要政府身份证才能接收或花费资金。

即使联合国（[https://www.unhcr.org/ibelong/about-statelessness](https://www.unhcr.org/ibelong/about-statelessness)）和世界经济论坛（[https://www.weforum.org/agenda/2020/11/legal-identity-id-app-aid-tech](https://www.weforum.org/agenda/2020/11/legal-identity-id-app-aid-tech)）意识到国家对身份的垄断造成的危害，但无法说服国家为无国籍或未注册的人打印身份证，也无法发行其自认可的非政府身份证。考虑到这一点，信任网络或基于社交媒体的身份协议不太可能成为主流工作、银行业务或公寓租赁的可用选择。

然而，非政府发行的分散式身份（DIDs）可能仍然在非正式经济中找到用途，后者已经提供了对工作、住房、医疗保健等的访问，无需身份证。尽管存在对现金的打击和日益严格的KYC（了解您的客户）法规，但全球范围内仍存在现金交易的非正式经济。此外，加密货币使得可以向全球任何人发送资金，无需银行或身份证，为无法审查的数字经济铺平了道路：[https://anarkio.codeberg.page/agorism](https://anarkio.codeberg.page/agorism), [https://bitcoinmagazine.com/business/kyc-free-bitcoin-circular-economies](https://bitcoinmagazine.com/business/kyc-free-bitcoin-circular-economies) 在这些无需许可的自由市场中，信任网络可能有助于业务评价和声誉，验证求职时的教育和技能，或建立邀请制市场的信任。

## 问题3：分散式身份（Decentralized ID）可能会被审查

一些分散式身份协议使用加密货币地址作为标识符，例如以太坊或比特币闪电网络。然而，已经出现过基于交易历史对用户进行审查的情况（例如使用无KYC交易所、加密货币混合器、赌博或购买灰色市场产品）。

将您的身份和社交生活与财务联系起来已经引起了隐私问题（因为您与之互动的任何人都可以轻松了解您的财富并监视您的收入和购买行为）。更糟糕的是，通过链分析或KYC审查实施审查意味着用户可能被排除在交易所、市场、社交媒体网站等之外。想象一下，因为您最近向赌博网站发送了资金、购买了CBD产品或不愿透露敏感信息（如政府身份证），或是全球10亿无法获得政府身份证的人之一，您可能会被永久封禁在Facebook或Twitter之外。

从技术角度看，加密标识符可能比密码提供更好的安全性。相比于（更强的）比特币私钥，破解不安全的密码要容易得多。密码学还使您能够签署消息，证明内容（例如社交媒体帖子、订单或合同）确实来自您本人，而不是冒名顶替者。

PGP已经提供了加密标识符，您可以选择性地添加您的姓名（或化名）并参与信任网络。您可以使用这个PGP密钥不仅仅用于登录网站（通过解密网站发送给您的代码），还可以通过PGP签名验证内容并安全加密消息、电子邮件和文件。由于PGP密钥不通过透明的区块链与您的财务信息相连，您可以轻松地创建假名和临时PGP密钥，这为您提供了一个私密且可访问的身份框架。

## 问题4：监视和将所有活动链接到一个身份的危险。

但是为什么需要验证姓名呢？为什么不相信某人的话，让他们选择使用什么名字？为什么所有的行为都需要与一个单一的持久物理身份相关联？

在国家政府身份证制度下，国家从“出生证明”到“死亡证明”跟踪个人 — 编制个人的工作、储蓄、购买、家庭地址、汽车、假期、医疗历史、电话通话、互联网历史等详细信息。这种监视水平是不成比例的和不道德的。

个人的生活应该是私密的。信息应该只在需要知道的情况下自愿分享。例如，只有您的雇主、同事和客户需要知道您的工作；只有您的医生、药房和保险公司（除非您自费）需要知道您的医疗历史；许多人只与亲密的朋友或家人分享他们的家庭地址。

在现有的“用户名和密码”模型中，用户可以自由创建自选的身份、化名和临时账户。例如，使用不同的工作和家庭配置文件、在在线聊天组中不分享真实姓名或位置、在活动主义、艺术、音乐或写作中使用化名，或者创建匿名账户加入支持组织（如健康问题、成瘾或家庭暴力支持组织）。将所有内容与单一身份关联可能会导致自我审查、不适（在敏感或健康相关话题中）甚至严重的安全问题（如活动主义、歧视或逃避虐待）。

对于商业交易，如购物、工作或公寓租赁，有许多建立信任的方式，而无需持久或由国家分配的身份，例如：

+   匿名交易：使用现金或加密货币购买面包或公交车票不需要提供姓名。只需支付，产品即为您所有。

+   钥匙和智能卡：房屋钥匙、邮箱钥匙、进入健身房或乘坐公共交通的智能卡。进入依赖于持有钥匙或卡片，因此无需个人身份证明。

+   PIN码和密码：例如，使用通过短信或电子邮件发送的PIN码取件。密码和PIN码可以组合使用，例如登录密码和通过短信或电子邮件发送的二次认证PIN码以确认操作。

+   加密密钥对：比特币使用假名的加密密钥对发送、接收和存储资金。PGP也使用假名的密钥对，以加密消息、签名和验证数据，并参与信任网络。

+   评论与声誉：例如企业的客户评论、求职者的作品集，或者在沙发冲浪或公寓租赁网站上的用户资料。

+   现金存款与担保：租房时现金存款可保护免受盗窃或损坏，而担保则可在网购或远程工作时防止诈骗。

+   非政府身份证：像Digitalcourage、Bitnation和World Passport等组织发行的非政府身份证，比国家颁发的护照更易获得，但不幸的是目前在主流企业中还不被接受。

对于许多商业交易而言，不需要持久或个人身份。在需要姓名的情况下，简单地说出您的姓名应该就足够了（可选通过PIN、PGP签名、信任网或社交媒体个人资料进行验证）。无论如何，经济或社交网络的参与不应要求单一的持久身份或由国家分配的身份。

## 结论

当前由政府门槛化身份证系统引起的监控和排斥明显显示了身份数据库的危险。如果您正在开发去中心化身份系统，请允许用户在不连接政府身份证的情况下参与，允许使用假名和一次性身份，并为偏好传统“用户名和密码”登录的人提供定期服务。不要创建现有破损系统的克隆，而是利用这个机会创建一个可供所有人参与的替代性、包容性和隐私友好的生态系统。

## 进一步阅读

身份危机 – Privacy International [https://privacyinternational.org/campaigns/identity-crisis](https://privacyinternational.org/campaigns/identity-crisis)

打破大型身份识别的神话 – Access Now [https://www.accessnow.org/busting-big-ids-myths](https://www.accessnow.org/busting-big-ids-myths)

在网络空间中不需要真名：关于身份和假名的讨论 – DerGigi [https://dergigi.medium.com/true-names-not-required-fc6647dfe24a](https://dergigi.medium.com/true-names-not-required-fc6647dfe24a)

名字意味着什么？透过匿名性促进包容性的案例 – Common Thread [https://blog.twitter.com/common-thread/en/topics/stories/2021/whats-in-a-name-the-case-for-inclusivity-through-anonymity](https://blog.twitter.com/common-thread/en/topics/stories/2021/whats-in-a-name-the-case-for-inclusivity-through-anonymity)

你不需要看我的ID – Jeffrey Paul [https://sneak.berlin/20200118/you-dont-need-to-see-my-id](https://sneak.berlin/20200118/you-dont-need-to-see-my-id)

证明我的身份：关押中没有法律身份证明的人的困境 – [维琪·普莱斯](https://www.penalreform.org/blog/proving-who-i-am-the-plight-of-people/)

KYC（了解您的客户）的少有讨论的危险及其应对之策 – AnarkioCrypto [https://vonupodcast.com/know-your-customer-kyc-the-rarely-discussed-danger-guest-article-audio](https://vonupodcast.com/know-your-customer-kyc-the-rarely-discussed-danger-guest-article-audio)

护照只是“暂时”的战时措施 – [斯佩兰塔·杜米特鲁](https://fee.org/articles/passports-were-a-temporary-war-measure)

第二次世界大战期间，我们确实有些事情需要隐藏 – [汉斯·德·兹瓦特](https://medium.com/@hansdezwart/during-world-war-ii-we-did-have-something-to-hide-40689565c550)

每一个流落街头的人都是我们社会的损失 – [彼得·巴罗赫](https://www.statelessness.eu/blog/each-person-left-living-streets-we-are-losing-society)
