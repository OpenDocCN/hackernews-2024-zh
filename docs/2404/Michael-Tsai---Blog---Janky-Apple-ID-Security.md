<!--yml

类别：未分类

日期：2024-05-27 13:35:39

-->

# 迈克尔·赛 - 博客 - 粗糙的苹果 ID 安全性

> 来源：[https://mjtsai.com/blog/2024/04/26/janky-apple-id-security/](https://mjtsai.com/blog/2024/04/26/janky-apple-id-security/)

我还有[另一个](https://mjtsai.com/blog/2023/10/23/secondary-apple-id-mess-and-inadvertent-password-reset/)苹果 ID 突然被锁定的例子。起初，我的 iPhone 要求我再次输入密码，我以为这是几个月来几乎自从我购买以来它一直在做的“正常”事情。但在这样做之后，它说我的账户被锁定了。

解锁账户需要一个 1 小时的安全延迟，因为我启用了被盗设备保护，并且我不在我熟悉的地点之一。*我在家里。* 但我去 **设置 ‣ 隐私与安全 ‣ 定位服务 ‣ 系统服务 ‣ 重要位置** 检查，出于某种原因，列表中唯一的位置是我每两周去一次的杂货店。它没有识别出手机几乎所有时间都待在的家/办公室位置，这些位置在苹果地图、联系人和查找我的中都标识为家。

所以我去了我的 Mac，那里没有延迟来解锁账户。然而，解锁没有成功。它让我输入密码，向我的手机发送了一个验证码，然后又要求我再次输入密码，但表格却出了问题。我输入了密码并点击了 **Sign In**，按钮保持灰色，显示了一个旋转器，然后停止了，既没有接受密码也没有显示错误。它只是卡在 **Sign In** 按钮无效的状态。这个新的系统设置真的很棒吗？

（其他几个与 Apple ID 相关的表格具有奇怪的布局和非标准行为。如果我还不熟悉这种不幸的现状，我可能会担心它们是假的用户界面试图钓鱼攻击我。）

（iPhone 版的系统设置也陷入了奇怪的状态，在那里 **Apple ID 建议** 屏幕显示了一个旋转器和一个无效的 **Continue** 按钮。整个应用程序周围有一个黑色边框。我不得不强制退出它。然后它再次以同样的方式卡住了。）

唯一需要做的事情就是点击 **Cancel** 来退出该表格。我的两台设备不断弹出关于登录我的 Apple ID 的警报，我仍然不想等一个小时，所以我退出了系统设置并重新启动它。我按照之前完全相同的步骤解锁我的账户，但这一次它让我使用我的 Mac 密码而不是发送验证码到 iPhone 来解锁。这一次，最后一个要求输入我的 Apple ID 密码的表格起作用了。

好消息是手机自动解锁了，并重新使得 Apple ID 服务可用。我不必在那里输入新密码。

不好的消息是，我不得不为这个帐户选择*另一个*新密码。整个过程让我感到不太安全。如果被盗设备保护不能正常工作，会不会在某个时候给我带来真正的麻烦？也许我应该关掉它。有没有办法让我的设备在*不依赖*我的Apple ID的情况下运行？唉，我想没有。

（我还有另一个用于测试Mac的Apple ID，但出于某种原因，每次使用它登录新安装时都需要解锁。尽管如此，我从未被要求重置它的密码。）

之前：

更新（2024-04-26）：[Dave Wood](https://mastodon.social/@davewoodx/112340263148377801):

> 该死的#Apple。我在忙自己的事情，手表和手机却收到提示：“使用您的Apple ID登录”。好吧，为什么？无论如何，我还是输入了密码。然后：被锁定了。什么鬼？更糟糕的是，我因为不在熟悉的地点而无法在一个小时内解锁我的帐户。我在家里，我很少离开。如果我的家不算熟悉，那么到底哪里算？

[Vini Barauna](https://mastodon.social/@vinibarauna/112340769414966434):

> 我妻子的帐户今天早些时候也发生了完全相同的事情。

[Adam Chandler](https://mastodon.social/@AdamChandler@masto.adamchandler.me/112340816977571006):

> 我的两个Apple ID 刚刚在一个小时前被锁定。密码已超过2年，所以这可能是最好的，但我在从亚特兰大起飞时更改了第一个密码，然后我降落在夏洛特时，我的另一个密码也要求更改。由于我的iPhone上的锁定处于活动状态，我在iPad上完成了此操作。我家里有两台Mac需要在我回家后更新到新的密码。

[nickf](https://hachyderm.io/@nickf/112340724862583216):

> 在阅读了你的文章不到20分钟后，同样的事情也发生在我身上，包括需要设置新密码。奇怪！
> 
> 尽管我当时在家里，被盗设备保护还是识别到了。

[Simon Harris](https://social.harukizaemon.com/@haruki_zaemon/112340697514989285):

> 这发生在我不到10分钟之前

[nutbunnies](https://mastodon.social/@nutbunnies/112340677857034645):

> 我今晚也遇到了这个问题。可能是因为入侵或其他原因导致的无声强制密码重置。

[Jonathan Wight](https://mastodon.social/@schwa/112340642293655124):

> Xcodes正在导致我的AppleID严重问题（苹果因“安全原因”而频繁锁定它）。

[Mike Cohen](https://sfba.social/@mikec415/112340656547478733):

> 我也遇到了同样的问题，当时我并未使用Xcode。今天下午有几个人收到了密码重置请求。

[Marc](https://mjtsai.com/blog/2024/04/26/janky-apple-id-security/#comment-4078216):

> 我也遇到了同样的问题，它还清除了我应用程序专用密码，导致多个应用程序出现问题。

[Nic Lake](https://mastodon.social/@niclake/112340929785009766):

> 情况相同。先是手表，然后是iPhone，Mac和Apple TV都这样做。我与在线聊天代理谈过这件事，但他们不告诉我发生了什么，只是说“有时会为您的账户增加随机的安全改进”。

[leo](https://twitter.com/leozera/status/1784050458493059392):

> 我今天下午也发生了这种情况。

[Thomas Vander Wal](https://mastodon.social/@vanderwal/112341025760658192):

> 我在大约晚上8点在用来在厨房听播客的旧iPad上遇到了这个问题，然后所有设备都被锁定了。只有在多次尝试之后，我才能连接我的MBP并重置iCloud密码。然后我才能使用新密码开始解锁所有其他设备。
> 
> 它感觉更像是苹果不经意间的一个黑客行为。

[Tom Bridge](https://theinternet.social/@tbridge/112340463598872085):

> 今晚有其他人的苹果ID被随机锁定吗？我在更改密码并重置所有应用专用密码后，不得不在所有设备上重新登录...

[Chance Miller](https://9to5mac.com/2024/04/26/signed-out-of-apple-id-account-problem-password/):

> 苹果的系统状态网页并未显示今晚任何服务存在问题。但根据社交媒体报告，很明显苹果的背后发生了一些不正常的事情。

更新（2024-04-27）：另请参见：

我不得不生成一个新的应用专用密码，并将其添加到Fantastical中才能进行同步。

虽然我的iPhone没有要求输入新的Apple ID密码，但iMessage静默失败了。它从未要求我重新登录；它只是停止接收新消息。我将它切换关闭然后再打开，然后它开始接收新消息，但在此期间发送的消息从iCloud没有同步下来。

我的第二台Mac确实要求我输入新的Apple ID密码。它也悄悄地停止接收新的iMessages，直到我启动了Messages应用，然后它才提示我登录。它也从未在注销状态下同步收到的消息。

[Giuseppe Carlino](https://mastodon.social/@beps/112341887668096123):

> 与显著位置混乱的情况相同。

[Carlo Zottmann](https://mastodon.social/@czottmann@norden.social/112342375139000057):

> 我的iPhone的“显著位置”并非如此。显然，我住在离我实际家2公里的树林里，并且我无法获取其他100多条位置记录的更多详细信息，这并没有增强信心。

[Brent](https://mastodon.online/@brentharrell/112344134088048230):

> 昨晚我也遇到了这个问题。不得不创建新密码并在每台设备上输入新密码。手表是最糟糕的，因为iPhone键盘不允许密码管理器填充，必须在另一台设备上查看并在iPhone上键入。尝试3次后仍然无效，所以我取消了。回到手表的设置中，我已经登录了。总体而言，所有设备完成至少需要1小时。初始解锁/重置至少需要3次尝试。并非一个令人温暖、舒适的体验。

[John Gruber](https://daringfireball.net/linked/2024/04/27/apple-id-lockout):

> 我刚刚在自己的 iPhone 上检查了一下，只有两个“重要位置”列在 设置 → 隐私与安全 → 位置服务 → 系统服务 → 重要位置，分别是“工作”和我最喜欢的（真的经常去的）杂货店。但“工作”位置距离我家整整三个城市街区（约0.2英里），这使得我的家庭位于不被算作该位置的半径之外。幸运的是，我没有遭遇这种账户锁定的问题，但这也让我更加确认我没有启用被盗设备保护是正确的。

更新（2024-04-28）：[Nick Heer](https://pxlnv.com/linklog/apple-id-lockout/):

> 对我来说目前还不清楚这是否只影响与开发者 Apple ID 有某种关联的账户。我的两个 Apple ID 都与开发者工具有关，但都没有受到这个问题的影响。
> 
> 这个问题已经存在大约十八个小时了。如果 Apple 能说出[任何有用的信息](https://mastodon.social/@niclake/112340915562253907)来承认这个问题将会很有帮助。

我不使用我的常规 Apple ID 来进行开发者工具操作，我的开发者 Apple ID 也没有被锁定。

[Pierre Igot](https://toot.community/@betalogue/112343197838679358):

> 当你的 iCloud/Apple ID 开始以奇怪的方式出现问题，将你扔进一个像卡夫卡式的循环中，有一个“锁定”的账户和密码重置过程以一个无用的“稍后再试”错误消息结束，而系统状态在所有 Apple 服务中始终保持绿色，那就别打电话给 Apple 了。即使他们也不知道发生了什么。等到第二天早上再试一次，可能这次密码重置会奇迹般地成功。

[Francisco Tolmasky](https://mastodon.social/@tolmasky/112343254055660691):

> 我检查了我的“重要位置”，里面只有一个我们上周末第一次去的水上公园。而我整天都呆的、在苹果地图上标记为“我的家”的家并没有列出来。

[Joe Cieplinski](https://mastodon.social/@jcieplinski/112343351216165768):

> 好了。被迫在大约一千台设备上无缘无故地更改密码已经够糟糕的了。现在甚至在尝试生成我需要的几十个应用专用密码时都不接受我的新密码。

[Ryan Jones](https://twitter.com/rjonesy/status/1784372613541757407):

> 我昨晚被 Apple ID 的问题困扰了。糟糕的复制和布局让我甚至认为我的整台机器被黑了。一团糟。

[Ryan Jones](https://twitter.com/rjonesy/status/1784648861220254034):

> 哦天哪，Apple ID 重置把我的 Apple Wallet 搞砸了。
> 
> 我需要再次验证我的卡，但没有按钮或方法。而且如何验证 Apple Cash 卡呢？
> 
> […]
> 
> 哦太好了，家庭共享被关闭了并且报错了。
> 
> 名称和照片共享也是。全部消失了。（即使重新启动后仍然如此。
> 
> 哎呀，iMessage 在设备间不同步。

更新（2024-04-29）：我继续看到有人报告遇到这个问题，并且有报告说苹果支持告诉客户没有普遍问题。令人失望的是，至少两天后仍有新用户遇到问题，而苹果还没有在其[系统状态页面](https://www.apple.com/support/systemstatus/)上发布任何信息。

我决定在家中的 iPhone 上禁用设备防盗保护，iOS 说会有一个小时的安全延迟，因为我不在熟悉的位置。 🤦‍♂️ 它说延迟结束时我会收到通知。几个小时后，通知从未到来，设备防盗保护仍然启用。 🤦‍♂️ 我现在更决心[关闭它](https://mjtsai.com/blog/2024/04/26/janky-apple-id-security/#comment-4078231)，因为我不相信延迟是否正常工作。我回到杂货店，但现在它不再列为重要位置。它现在只显示一个健身房，我很少去那里，比杂货店去得更远。 🤦‍♂️ 不过，当我回家后它确实让我关闭了设备防盗保护，所以也许延迟工作正常，只是通知有问题。

[Dave Wood](https://mastodon.social/@davewoodx/112346518495293507)：

> 我查看了我的 iPhone 认为的重要位置。它已被禁用！所以我没有重要位置。系统如何让我启用了设备防盗保护却没有打开重要位置呢？

[Adam Chandler](https://masto.adamchandler.me/@AdamChandler/112349249791850647)：

> 而且我的 AppleID 又被锁定了。iCloud 锁定的恐怖故事太多了，所以这是我有史以来重置密码最小心的时候。

[David Owens II](https://twitter.com/owensd/status/1784673004963905815)：

> Apple ID 的密码不工作，好吧。
> 
> 尝试重置，但由于这并非与我的设备同步的“iCloud”帐户，而是商店帐户，我的“已登录设备”没有收到通知。
> 
> 所以现在我必须等三天，直到我收到一个重置密码的短信到我的号码……

[Kirk McElhearn](https://www.intego.com/mac-security-blog/apple-id-password-reset-what-we-know/)：

> Significant Locations 在我的 iPhone 上显示了 55 条记录，但只显示了一个最近的位置。无法告诉 iPhone 你希望考虑的重要位置，比如家庭或工作地点，因此如果启用了设备防盗保护，就得听凭 Apple 的定位服务。

我不确定这里发生了什么，因为我看到其他人的截图显示了[多个位置](https://mastodon.social/@davemark/112348959797503247)。而我的 iPhone 只显示了一个。

> 这一事件揭示了依赖于Apple ID的风险之一。随着越来越多的人依赖iCloud，如果你的Apple ID被锁定，可能会造成灾难性后果。没有这个账户，你将无法使用iCloud邮箱、iMessage或FaceTime。如果你将个人甚至工作文件存储在iCloud上，也无法访问。而且，你也无法使用依赖iCloud的第三方应用，比如日历或联系人应用。

由于电子邮件地址可能是访问账户的必要条件（用于验证或者重置密码），我认为将iCloud地址用作任何重要账户的登录名是个糟糕的主意。这也让我三思是否将苹果密码用作我的认证器（实际密码存储在PasswordWallet中）。希望，即使我的账户被锁定，我仍然能够使用认证器，因为信息是本地缓存的。但我们都知道，iCloud往往会无缘无故地丢弃缓存数据。

> 鉴于这个问题的影响范围，苹果应该解释发生了什么。许多用户担心有人访问了他们的账户，并急于重置密码，以为他们的数据可能被盗。目前尚不清楚有多少用户受到影响，但许多国家的用户都进行了密码重置，有些人甚至报告称这个问题一直持续到周日。截至本文写作时，即2024年4月29日星期一，苹果还未作任何表态。

[皮埃尔·伊戈特](https://toot.community/@betalogue/112354548833053336)：

> 通常情况下，苹果出了问题，而他们并没有承认，只是自欺欺人地假装这件事从未发生过。
> 
> 换句话说，苹果依旧如往常的傲慢，这是以牺牲用户为代价的。

更新（2024-05-01）：[皮埃尔·伊戈特](https://toot.community/@betalogue/112361888534658218)：

> 顺便说一句，在iOS的设置中搜索“significant”返回…… ∅。实际上，“重要位置”位于隐私与安全 › 位置服务 › 系统服务下。
> 
> […]
> 
> 无论他们写了什么，iOS的系统设置中搜索它（“significant”或“familiar”）仍然一无所获。

参见：[亚当·恩格斯特](https://tidbits.com/2024/04/27/widespread-reports-of-apple-id-accounts-being-inexplicably-locked/)。

更新（2024-05-03）：[沃纳·克罗克](https://warnercrocker.com/2024/05/03/time-for-apple-to-come-clean-about-icloud-part-2/)：

> 苹果（甚至所有公司，因为每家公司都在线并可能受到黑客攻击）至少应向用户提供开放的沟通。同样重要的是，苹果应向其技术支持人员提供更开放、更好的沟通，特别是在面对这些问题时。
> 
> […]
> 
> 我不会详细描述我的**iCloud头痛**问题。你可以在博客文章[这里](https://warnercrocker.com/2023/10/05/time-for-apple-to-come-clean-about-icloud/)、[这里](https://warnercrocker.com/2023/05/09/apple-icloud-migraines/)、[这里](https://warnercrocker.com/2023/07/25/a-possible-answer-to-those-apple-migraines/)和[这里](https://warnercrocker.com/2023/05/22/apple-icloud-migraines-continue/)找到详细内容。尽管如此，在此事件后需要重新登录Messages让我继续相信Apple在iCloud上有根深蒂固的问题。我已经与这些问题（和Apple）斗争了一年多。

更新（2024-05-07）：[Pierre Igot](https://toot.community/@betalogue/112388728937106140)：

> 从2024年4月大苹果ID密码重置事件后的最新章节：昨天，我试图在MailMate中使用我的mac.com邮箱地址（也就是我的Apple ID）通过Apple的服务器发送邮件。因为苹果几乎不支持（非常勉强）第三方邮件客户端，你需要为MailMate定义两个应用特定密码，一个用于接收邮件，一个用于发送邮件。
> 
> […]
> 
> 站点要求我重新登录。（我刚刚登录过！）好吧。然后，它要求我确认我的Apple ID密码。我输入了我的新密码（上周重置的那个），结果告诉我密码错误！我再试了几次……还是一样。
> 
> 所以我在Apple ID网页上完全退出登录，从头开始，这次用我的Apple ID和（同样的）新密码登录（而不是使用密码）。这次成功了（等等，你不是刚说密码错误了吗？），但是……现在Apple又说我的账户再次被锁定了！

更新（2024-05-09）：[Andrew](https://twitter.com/andrewe/status/1788244436733899130) [Escobar](https://twitter.com/andrewe/status/1788230987387658486)：

> Apple ID 要么出了问题，要么在WWDC之前更新。
> 
> 当我的账户在4月24日被锁定时，所有我的应用特定密码都被清除……我仍然无法设置新密码。
> 
> 我担心苹果甚至没有承认4月26日星期五的Apple ID事件。

[Apple ID](https://mjtsai.com/blog/tag/apple-id/) [Apple Password Manager](https://mjtsai.com/blog/tag/apple-password-manager/) [iMessage](https://mjtsai.com/blog/tag/imessage/) [iOS](https://mjtsai.com/blog/tag/ios/) [iOS 17](https://mjtsai.com/blog/tag/ios-17/) [Mac](https://mjtsai.com/blog/tag/mac/) [macOS 14 Sonoma](https://mjtsai.com/blog/tag/macos-14-sonoma/) [Messages in iCloud](https://mjtsai.com/blog/tag/messages-in-icloud/) [Passwords](https://mjtsai.com/blog/tag/passwords/) [Security](https://mjtsai.com/blog/tag/security/) [System Preferences](https://mjtsai.com/blog/tag/system-preferences/) [Top Posts](https://mjtsai.com/blog/tag/top-posts/)
