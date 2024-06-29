<!--yml

category: 未分类

date: 2024-05-27 14:34:37

-->

# 苹果刚刚在iOS 17.4 Beta（欧盟）中破坏了Web应用程序吗？- 开放Web倡导

> 来源：[https://open-web-advocacy.org/blog/did-apple-just-break-web-apps-in-ios17.4-beta-eu/](https://open-web-advocacy.org/blog/did-apple-just-break-web-apps-in-ios17.4-beta-eu/)

我们已经收到了[警报](https://twitter.com/mysk_co/status/1753402489049616427)，称苹果通过iOS 17.4 Beta在欧盟破坏了Web应用（PWA）的支持。安装到主屏幕的站点无法在其自己的顶级活动中启动，而是在Safari中打开。这将Web应用从操作系统的一流公民降级为简单的快捷方式。开发者确认该bug在[欧盟之外](https://twitter.com/mysk_co/status/1753402489049616427)并未发生。

这个bug应该在测试任何Web应用程序的几秒钟内就会被发现。那么这个问题是怎么被忽略的呢？一个可能性是Safari或iOS团队并不定期测试Web应用程序，也没有自动化的回归测试。这可能解释了过去15年中发布到稳定版本的iOS Web应用程序中的一个停滞不前的游行。

另一个解释可能是苹果[混乱的合规提案](/blog/owa-review-apple-dma-compliance-for-web/)所带来的复杂性，它为欧盟创建了iOS的一个单独版本。我们曾写过[苹果提案可能给网页开发者造成的测试问题](/blog/developers-react-apple-eu-dma-compliance/)，或许苹果是这种试图地理围栏浏览器进步的自毁尝试的第一牺牲品。

本文作者甚至因不在欧盟而无法亲自测试这一点。这就是苹果恶意和怨恨的数字市场法合规提案将对全球网页开发者和企业造成的疯狂。

这几乎可以肯定是一个bug，但是由于苹果长期以来在[iOS Safari的特性忽视和否认](/walled-gardens-report/#safari-lags-behind-and-is-missing-key-features)，这引起了开发者社区的怀疑。选择投入不足可能只会带来问题，除非[苹果有效禁止第三方浏览器](/walled-gardens-report/#apple-has-effectively-banned-all-third-party-browsers)。因为第三方浏览器无法提供自己的Web应用功能，如果在iOS Safari上无法工作，它在iOS上也无法工作。

英国竞争与市场局指出了苹果可能的App Store收入保护动机：

> 苹果通过其App Store获取收入，既通过向开发者收费以访问App Store，又通过通过Apple IAP进行支付而收取佣金。因此，苹果从iOS上原生应用的更高使用率中获益。通过要求iOS上的所有浏览器使用WebKit浏览器引擎，**苹果能够控制所有iOS浏览器的最大功能**，因此阻碍了Web应用的发展和使用。这限制了Web应用对原生应用的**竞争约束**，从而保护和增强了苹果的App Store收入。[UK CMA - 移动生态系统临时报告](https://www.gov.uk/government/publications/mobile-ecosystems-market-study-interim-report) （重点添加）

尽管如此，苹果一直声称Web应用是他们应用商店的替代品：

> 对于其他一切，总有开放的互联网。如果App Store的模型和指南不适合您的应用程序或商业理念，那没关系，我们也提供Safari提供出色的Web体验。[苹果应用商店指南](https://developer.apple.com/app-store/review/guidelines/)

最近，大卫·海内迈尔·汉森（“DHH”）与苹果之间发生了争执，原因是DHH认为苹果威胁将他们的HEY电子邮件/日历从他们的App Store中驱逐出去，如果他们不分享收入：

> 苹果刚刚打来电话告诉我们，他们拒绝了 App Store 中的 HEY Calendar 应用（当前版本）。与上次一样的霸道手段：将微妙的拒绝理由推给只留下名字的人，他会轻声告诉你，这是你的钱包还是你的膝盖。[大卫·海内迈尔·汉森 - Ruby on Rails 的创始人兼 Hey 的 CEO](https://twitter.com/dhh/status/1743341929675493806)

当DHH宣布他将推广并构建用于Web应用的工具时，[苹果迅速收回并批准了这个应用：](https://twitter.com/dhh/status/1744745276932604413)

> 我们将使Rails 8成为创建全栈PWA的最佳框架。Web推送、标记、安装提示、服务工作者，应有尽有。我以前就有动力，但现在更加充实。让我们回到制作应用程序的路上，不必为许可或宽恕而求饶！[大卫·海内迈尔·汉森 - Ruby on Rails 的创始人兼 Hey 的 CEO](https://twitter.com/dhh/status/1743664413964374505)

Netflix还因拒绝将其应用移植到苹果新的AR/VR头戴设备Apple Vision Pro而引起轰动：

> 我们的会员将能够在Vision Pro上的Web浏览器上享受Netflix，类似于我们的会员可以在Mac上享受Netflix [Netflix](https://variety.com/2024/digital/news/apples-vision-pro-netflix-youtube-spotify-1235877784/)

发布后，人们发现苹果已经[令人惊讶地完全删除了从其头戴设备安装Web应用的功能](https://twitter.com/SteveMoser/status/1749438049300124008)。这是当前监管时刻的一项大胆的疏漏。

开发者社区对此极度怀疑。

事实很简单，无论这是无能还是恶意，最终结果都是一样的。iOS上Web应用的支持太差劲了。Apple面临的问题是没有有效的浏览器竞争，导致iOS上的致命网络问题频频发生。

想象一下，如果在iOS上允许真正有效的浏览器竞争。在这种情况下，致命的Safari错误将使开发者能够推荐更强大的浏览器。这反过来将迫使Apple投资于Safari，减少iOS用户的糟糕网络体验。这将带来一个充满活力、功能强大的网络，解锁开发者和企业的投资，使它们能够通过网络向iOS用户提供服务。

浏览器竞争对于安全性、功能、稳定性和隐私非常重要。自iOS推出以来，这种竞争一直被Apple所否认，而这种竞争必须在全球范围内恢复，而不仅仅是在欧盟。
