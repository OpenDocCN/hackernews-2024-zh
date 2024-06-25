<!--yml

category: 未分类

date: 2024-05-27 15:24:10

-->

# OWA对苹果的网络媒体通信法规合规提案的评审 - 开放网络倡导

> 来源：[https://open-web-advocacy.org/blog/owa-review-apple-dma-compliance-for-web/](https://open-web-advocacy.org/blog/owa-review-apple-dma-compliance-for-web/)

## 苹果是否会让浏览器和Web应用程序公平竞争？

我们相信第三方浏览器应该被允许使用它们安全提供给其他平台的相同引擎，在iOS上公平竞争。此外，通过浏览器之间的激烈竞争提供的功能性、稳定性和安全性使得通过真正的互操作性平台——网络绕过并挑战应用商店这个门户监管者成为可能。

考虑到这一点，OWA一直在审查[苹果关于遵守欧盟数字市场法的提案](https://developer.apple.com/support/dma-and-apps-in-the-eu/#browser-alt-eu)，以确定苹果是否打算真正遵守其有关浏览器和Web应用程序的法律义务。正如OWA已经[详细论述过的](/walled-gardens-report/)，在iOS上浏览器的真正选择是对门户监管力量最重要的制衡，因此这个问题的答案非常重要。

**可惜迄今为止，答案似乎是否定的。**

让我们深入探讨我们关注的三个主要点：

1.  浏览器供应商是否能够有效地将自己的引擎带到 iOS 上？

1.  浏览器供应商是否能够在iOS上实现适当的Web应用程序支持？

1.  浏览器供应商是否能够与Safari公平竞争？

## 浏览器供应商是否能够有效地将自己的引擎带到 iOS 上？

正如OWA所预期的，苹果正在提议一个新的[Web浏览器引擎授权](https://developer.apple.com/support/alternative-browser-engines/#web-entitlement)，允许浏览器应用程序具有一些任何现代、安全浏览器所需的扩展权限。苹果还提出了浏览器供应商必须满足的要求。其中一些涉及网络内容渲染的功能方面，确保浏览器引擎至少符合标准。其他一些涉及网络浏览的安全方面，还有其他一些则强加了特定约束条件（根据苹果的观点）与隐私相关。

安全要求是最不受反对的，主要浏览器已经符合[NIST关于关键软件供应链的指南](https://www.nist.gov/itl/executive-order-improving-nations-cybersecurity/security-measures-eo-critical-software-use-2)，因此，据推测，它们也符合苹果对这些流程的较松的重新阐述。OWA认为，只要规则合理、成比例，并且不排斥那些认真对待安全性的较小的浏览器供应商与其自己的引擎在 iOS 上竞争，那么由苹果设定一个最低标准是合理的（并且法案允许）。

DMA 只授予 Apple *“严格必要和成比例的措施，以确保第三方软件应用程序或软件应用程序商店不危及硬件或操作系统的完整性”，*其必要性和成比例性的证明责任在于 Apple。我们同意该法案的意图，但认为诸如性能和功能等方面应该交给竞争。

Apple 要求浏览器通过 *“Web 平台测试达到 90%”，*但这是荒谬的，需要立即澄清。[Web 平台测试项目](https://wpt.fyi/about) 是针对浏览器引擎的全面测试集，但没有普遍认可的测试集，可以从中测量 90%，因为每个引擎项目根据最小实现的特性启用或禁用测试组。一个几乎不实现任何功能的引擎可以轻松地保持 *其提供的功能* 的 90% 通过率，但远未达到广泛标准一致性的目标。测试集也会因浏览器的桌面版和移动版而变化。没有澄清哪组测试将计算 90%，开发者就生活在怀疑之中。另一个问题是 Apple 的措辞可能要求提供无法产生的通过率统计数据，因为在 iOS 上没有任何浏览器、使用任何引擎（尽管测试了 Android 浏览器）运行任何一组 WPT 测试集 — 包括 Safari。

Apple 政策中的隐私要求引发了深刻的关注。与其通过启用市场竞争来改善隐私，Apple 要求特定的技术机制，而其自己的产品并不遵守（例如，“引导” cookie 用于将 Web 应用安装到主屏幕）。它还创建了模糊不清、无法执行的关于“知情用户激活”的条款，读起来像是 Apple 审查员拒绝浏览器的一张空白支票，而不是基于分析。即使进行了非常简短的调查，我们发现 Apple [尚未遵循这些指南](https://webkit.org/blog/8311/intelligent-tracking-prevention-2-0/#:~:text=Temporary%20Compatibility%20Fix%3A%20Automatic%20Storage%20Access%20for%20Popups)。Apple 关于其要求一致性的许多问题的自身记录都很糟糕，OWA 认为这种措辞需要被删减并更具客观性。这里的主要关注点是 Apple 可能会将此规则用作工具，拒绝第三方浏览器实现重要的 Web 应用功能，以保持该功能专属于通过其 App Store 销售的原生应用。

要求要求替代浏览器实现与输入相关的特定 API 更令人担忧。苹果要求使用特定的 iOS API 来实现文本输入、滚动、拖动和上下文菜单等功能。这看似合适，以提供 iOS 用户统一的体验，但一致性可以通过多种方式实现。事实上，大多数浏览器引擎都在极力努力地与其宿主操作系统进行良好集成，提供最佳的体验。在这一领域的严格要求似乎很难得到正当理由，尤其是考虑到 macOS Safari 长期以来为了竞争优势而滥用私有 API 的历史。

该法案的目的之一是允许第三方，如浏览器供应商，向用户提供比门户的选择更多、质量更高的软件。第三方浏览器已经在其自己的浏览器中构建了执行许多此功能（如滚动）的代码。网页开发者长期以来一直抱怨，iOS Safari 在除了最简单的用例之外都存在滚动错误。强制要求第三方浏览器必须用苹果的库替换他们自己的，可以说更优秀的库，会削弱竞争力，并可能需要额外的工作，而且没有法律依据或用户利益。

强制浏览器供应商采用这些 API 可能会反过来损害质量，延迟竞争性浏览器的发布数月，而没有实质性地提高一致性。考虑到这些 API 的最后一刻宣布，苹果试图将其限制在欧盟范围内的尝试，以及库比诺长达数年的延迟行动，这尤为令人恶心。DMA 所试图纠正的错误会随着时间的流逝而增加，而阻止竞争对手及时进入市场的条款与法律的精神相悖。

实际上，其中一些 API 对于竞争引擎代码库有着深远的影响。因此，它很可能增加竞争引擎的复杂性，并使浏览器团队的进一步开发和维护变得更加困难。当苹果在非苹果平台上发布 Safari 时，并没有受到类似的限制，并且不太可能为 iOS Safari 自身接受这些限制。采用这些 API 应该是可选的，让供应商可以根据需要自由地将它们纳入，并以自己的步调进行。

## 浏览器供应商能否在 iOS 上实现适当的 Web 应用支持？

尽管一些 API 已经可用（并且是强制性的）供浏览器引擎实现特定功能，但其他 API 的缺失是显著的。

自 2007 年首款 iPhone 推出以来，Safari 就可以访问用于安装和管理 Web 应用的私有 API，然而这个核心、长期存在的 API 在苹果的遵从提案中却没有。就像 Safari 一样，替代浏览器必须能够将 Web 应用添加到主屏幕、应用程序库，并且在系统设置中显示并修改每个个别的 Web 应用设置。它们还必须能够在安装它们的浏览器相同的引擎中运行 —— 就像 Safari 安装的 Web 应用已经做了 15 年一样。到目前为止，我们获取的文档显示，替代浏览器将无法访问此类功能。

DMA 明确提到了浏览器竞争（包括使用自己的引擎的能力）对于防止门户网站决定 Web 应用功能的重要性。

> 特别是，每个浏览器都建立在一个网页浏览器引擎上，这个引擎负责关键的浏览器功能，如速度、可靠性和网页兼容性。**当门户网站操作并强加网页浏览器引擎时，他们就有能力确定不仅适用于自己的网页浏览器，还适用于竞争对手的网页浏览器以及网页软件应用的功能和标准。** [数字市场法] (https://eur-lex.europa.eu/legal-content/EN/TXT/?toc=OJ%3AL%3A2022%3A265%3ATOC&uri=uriserv%3AOJ.L_.2022.265.01.0001.01.ENG)（重点添加）

英国竞争与市场管理局指出，苹果有动机阻止 Web 应用公平竞争：

> 苹果通过其应用商店产生收入，既通过向开发者收取访问应用商店的费用，又通过从通过苹果IAP进行的支付中获取佣金。因此，苹果受益于iOS上本机应用的更高使用率。通过要求iOS上的所有浏览器使用WebKit浏览器引擎，**苹果能够控制iOS上所有浏览器的最大功能**，因此，限制了Web应用程序的开发和使用。这限制了Web应用程序对本机应用的**竞争约束**，从而保护和惠及苹果的应用商店收入。[英国竞争与市场局 - 移动生态系统中期报告](https://www.gov.uk/government/publications/mobile-ecosystems-market-study-interim-report)（强调添加）

最后，苹果在其应用商店指南的开头段落中指出，Web（即Web应用程序）是其应用商店的替代品。

> 对于其他一切，始终有开放的互联网。如果App Store的模式和指南不适合您的应用程序或业务创意，那没关系，我们也提供Safari，以获得出色的网络体验。[苹果应用商店指南](https://developer.apple.com/app-store/review/guidelines/)

总结所有这些，再加上[苹果拒绝在iOS Safari中实现关键功能，如安装提示](https://open-web-advocacy.org/walled-gardens-report/#ios-web-app-installation---a-well-hidden-safari-exclusive)（将Web应用程序隐藏在公众视野之外）、[其他缺失的功能](https://open-web-advocacy.org/walled-gardens-report/#safari-lags-behind-and-is-missing-key-features)以及[重大且严重的错误](https://open-web-advocacy.org/walled-gardens-report/#safari-has-been-buggy-for-a-long-time)，第三方浏览器是否能够为Web应用程序提供更好的解决方案的问题变得至关重要。按照现有情况，苹果的提议中没有任何迹象表明他们有意让第三方浏览器安装由其自己引擎驱动的Web应用程序。

替代浏览器还应能够处理Web应用程序的推送通知，并在其安装时显示标记。在苹果的文件中没有提及允许实现此类功能的API。

当用户在操作系统的任何地方点击与之相关的 URL 时，Web 应用程序也应该被允许打开。这是苹果称之为“通用链接”的一种模式，根据文档，浏览器不能为 Web 应用程序实现相应的功能：*[“具有 com.apple.developer.web-browser 托管权限的应用程序不得声称对特定域的通用链接作出响应。”](https://developer.apple.com/documentation/xcode/preparing-your-app-to-be-the-default-browser#Adhere-to-browser-restrictions)* [启用此功能的规范](https://wicg.github.io/web-app-launch/)已经被其他浏览器实现，而苹果对该功能的禁止则构成对竞争对手的不合理限制，将削弱 Web 应用程序在公平竞争的环境中的竞争能力。

DMA 规定了门户不能阻止第三方访问操作系统或设备功能，如果这些问题只是疏忽，我们希望委员会和苹果能够迅速解决。时间紧迫。

## 浏览器供应商是否能够与 Safari 公平竞争？

### 苹果提议的浏览器选择屏幕

根据 DMA 的规定，苹果计划在用户首次打开 Safari 时引入默认浏览器选择屏幕。目前我们对选择屏幕的设计几乎没有任何信息，[除了它将以随机顺序显示用户所在欧盟国家最受欢迎的 12 种浏览器](https://9to5mac.com/2024/01/26/apple-shares-more-details-about-the-new-default-web-browser-prompt-in-ios-17-4/)。苹果有许多方法可以操纵这个选择屏幕，使其失效，因此我们将密切关注最终的实施方式。

苹果对第三方浏览器的反竞争行为之严重程度难以言表。**十几年来**，甚至*都不可能*将不同的浏览器设置为默认。这种明显偏袒 Safari 的行为使其在市场上占据了领先地位。对于这种激进、持续、长期的自我偏爱行为，选择屏幕的影响令人怀疑，但 OWA 对此尝试表示欢迎，并敦促委员会严格监督这个过程。如果之前的版本被判不符合规定，还不应排除对已经受过影响的用户重新进行选择投票的可能性。

### 苹果系统提供的应用内浏览器忽略了用户对默认浏览器的选择

苹果还通过[操作系统提供的“应用内浏览器”机制](https://developer.apple.com/documentation/safariservices/sfsafariviewcontroller)自我偏好Safari。OWA预期会提及用于默认竞争浏览器处理应用程序通过此功能进行导航的API，但这一点却没有。苹果在这方面持续自我偏好的累积效应是使网络遗忘。即使用户将竞争对手设为默认浏览器，该浏览器也不会用于处理大部分浏览。这是一种竞争约束，与DMA精神不相容，而苹果的提议忽略了用户对默认浏览器的选择。

### 苹果将限制这些更改在欧盟内生效。

苹果提出只允许竞争对手在欧盟内运输其自己的引擎。这是一种试图添加毒丸的举措，增加竞争对手维护多个iOS浏览器的成本（一个用于欧洲，另一个用于全球其他地区），并强加苹果不承担的成本，而且如果由竞争对手强加，苹果永远不会接受。OWA预期会有一定程度的可怕但合法的恶意顺从，但在欧盟之外继续限制引擎选择是极其严重的。

这种额外的负担对于小型浏览器来说是难以承受的，相比Safari，它们将受到劣势的影响，这加剧了 OWA 在报告和监管文件中多次描述的15年反竞争行为所造成的损害。[Mozilla已经记录了这些条款，称其为_“制造障碍以阻止iOS上真正的浏览器竞争”_，OWA也同意。

苹果将此更改限制在欧盟的另一个含义是，Web开发人员难以跨越这一界限测试他们的Web应用程序和网站。欧盟的Web开发人员将无法测试基于WebKit的备用浏览器的版本。由于苹果的WebKit限制，这些iOS浏览器已经很难测试“frankenbrowsers”，这增加了进一步的成本和复杂性。

非欧盟用户将使用不同版本的“Firefox”、“Chrome”、“Opera”，考虑到在苹果浏览器上实现现代化体验的困难，增加这项工作将增加企业构建具有竞争力的网络体验的痛苦。

同样，欧盟之外的开发人员将无法在只在欧盟可用的浏览器上测试其网站。这将导致欧盟用户和欧盟Web应用程序和网站的用户在其外部遇到重大问题和错误，进一步损害了欧盟企业和用户。然而，这些问题只会影响使用备用浏览器的用户。Safari将具有显著优势，其引擎在全球范围内都是相同的。

显然，委员会在欧盟之外没有管辖权，不能要求其条款在全球范围内遵守，但是苹果将真正的浏览器选择限制在欧盟范围内的提议显示，远非仅仅因为在过去十年里在iOS上有效地禁止了第三方浏览器，苹果就会坚持这种行为，直到在地球上的每个主要司法管辖区都受到法律的约束。

### 苹果将阻止第三方浏览器在iPad上进行竞争

苹果选择将这些浏览器更改限制在iPhone上，并且不与iPad共享。这是目前允许的，原因是与欧盟关于iPadOS是否受DMA约束的[持续争议](https://open-web-advocacy.org/blog/owa-eu-dma-submission-apple-ipados/)。在我们的提交中，我们认为iPadOS是iOS的子集，有轻微的更改，即使不是这样，它在欧盟的市场规模和实力足以成为自己的核心平台服务。

没有合理的理由解释为什么浏览器应该被允许在iPhone上竞争而不是iPad。这只是苹果试图做到尽可能最少的情况，带来的额外好处是在iPad上搞乱浏览器或网络应用程序的竞争。

### 浏览器供应商能否更新其现有的浏览器？

> 与使用系统提供的网络浏览器引擎的任何应用程序的二进制文件分开[苹果文档](https://developer.apple.com/support/alternative-browser-engines/)

苹果在更新现有的浏览器应用程序以使用自己的引擎方面的工作方式模糊不清，目前尚不清楚是否：

+   浏览器供应商必须使用自己的浏览器引擎发布一个单独的应用程序。

+   浏览器供应商可以无缝地将其现有用户切换到现在使用自己的浏览器引擎的更新浏览器。

如果是第一种情况，那就非常棘手，因为这基本上将每个想要使用自己引擎的第三方浏览器的用户重置为零。考虑到选择屏幕将在浏览器供应商有机会发布其真正的浏览器之前进行，这一点至关重要。

浏览器供应商需要能够以稳定、可预测的方式推出他们的引擎。这意味着完全控制用自己的引擎替换webkit的能力，包括能够进行浏览器引擎分阶段发布和能够回滚的能力。

### 苹果针对希望使用自己引擎的浏览器的新合同

我们团队没有律师，但在我们这些非律师的眼里，苹果合同中的一些条款似乎不仅不公平，而且直接违反了DMA。

据我们所知，“允许”使用自己引擎的具体合同在成功的消费者操作系统的历史上是苹果独有的。

> 虽然苹果在此附录或开发者协议下的其他权利，或法律或公平法所提供的任何其他补救措施均不受限制，但若苹果有理由相信您或您的替代网络浏览器引擎应用（EU）未能遵守此附录或开发者协议的要求，苹果保留在通知您后立即撤销您对任何或全部替代网络浏览器引擎 API 的访问权的权利；要求您从您的替代网络浏览器引擎应用程序（EU）中删除您的授权文件配置文件；终止此附录；阻止更新、隐藏或删除您的替代网络浏览器引擎应用（EU）和/或其他应用程序从应用商店中；阻止您的应用程序在苹果平台上分发；以及/或暂停或将您从苹果开发者计划中移除。[苹果新的浏览器引擎合同](https://developer.apple.com/contact/request/download/web_browser_engine.pdf)

该条款基本上是说，如果您未能遵守苹果的所有条款（其中一些相当模糊，可能违反了DMA），他们不仅可以拒绝该更新，直到修复为止阻止您的应用程序，禁止您的应用程序，而且实际上可以永久移除和禁止贵公司在所有苹果平台上的每一个应用程序的分发（据推测，包括 macOS）。

另一条款规定，浏览器还必须遵守苹果应用商店指南（“指南”是一个令人愉快的奥威尔式短语，指的是铁证如山的规则）。 苹果在公平判断违反其应用商店规则方面声誉不佳。

> 也许让你感到惊讶的是，苹果对其文本的解释往往看起来最多是反复无常，而在最坏的情况下似乎是出于自利。[迪特·博恩 - The Verge](https://www.theverge.com/2020/6/17/21293813/apple-app-store-policies-hey-30-percent-developers-the-trial-by-franz-kafka)
> 
> 本文中的任何内容均不意味着苹果将接受您的替代网络浏览器引擎应用（EU）在应用商店上分发，并且您承认并同意苹果可以自行决定以任何理由拒绝或停止分发您的替代网络浏览器引擎应用（EU）。 为明确起见，一旦您的替代网络浏览器引擎应用（EU）已被选择通过应用商店进行分发，它将被视为开发者协议下的“已许可应用程序”。[苹果新的浏览器引擎合同](https://developer.apple.com/contact/request/download/web_browser_engine.pdf)

该条款基本上是说他们可以因任何理由拒绝您的浏览器。

幸运的是，DMA 不允许这样做。 规则需要公平、合理和非歧视性。 此外，浏览器供应商将能够就应用商店裁决向替代争议解决机制提起上诉。

> 门户管理员应对商业用户其软件应用商店、在线搜索引擎和在根据第 3(9) 条指定决定中列出的在线社交网络服务的公平、合理和非歧视性的一般准入条件进行适用。为此，门户管理员应发布一般准入条件，包括一种替代争议解决机制。委员会应评估发布的一般准入条件是否符合本段要求。[数字市场法](https://eur-lex.europa.eu/legal-content/EN/TXT/?toc=OJ%3AL%3A2022%3A265%3ATOC&uri=uriserv%3AOJ.L_.2022.265.01.0001.01.ENG#:~:text=Any%20practice%20that%20would%20in%20any%20way%20inhibit%20or%20hinder%20those%20users%20in%20raising%20their%20concerns%20or%20in%20seeking%20available%20redress%2C%20for%20instance%20by%20means%20of%20confidentiality%20clauses%20in%20agreements%20or%20other%20written%20terms%2C%20should%20therefore%20be%20prohibited.)
> 
> 苹果可以随时，有时，有或无事先通知您，修改、移除或重新发布苹果材料或备选网络浏览器引擎 API，或其任何部分。您理解，此类修改可能要求您自行承担更改或更新您的备选网络浏览器引擎应用程序（欧盟）的费用，并且此类应用程序的功能和功能可能会停止运行。除非适用法律另有规定，苹果没有明示或暗示的义务提供或继续提供苹果材料或备选网络浏览器引擎 API，并且可以随时暂停或终止您对其全部或任何部分的访问。[苹果新的浏览器引擎合同](https://developer.apple.com/contact/request/download/web_browser_engine.pdf)

这个条款表示，苹果可以在没有任何通知的情况下破坏、移除或更改任何 API。这看起来几乎是不公平或不合理的。

> 保密性您同意，关于备选网络浏览器引擎 API 或备选网络浏览器引擎（欧盟）授权配置文件的任何非公开信息应按照开发者协议第 9 条（保密性）的条款视为“苹果机密信息”加以对待。您同意仅出于行使您在本补充协议下的权利和履行您的义务的目的使用此类苹果机密信息，并同意不将此类苹果机密信息用于任何其他目的，也不得为您自己或任何第三方的利益使用，未经苹果事先书面同意。您进一步同意不向除您的员工或承包商以外的任何人披露或传播苹果机密信息，这些员工或承包商必须有权知晓，并受有一份禁止未经授权使用或披露苹果机密信息的书面协议的约束。[苹果新的浏览器引擎合同](https://developer.apple.com/contact/request/download/web_browser_engine.pdf)

这禁止公开讨论苹果的 API 或与除了那些受类似协议约束的员工或承包商之外的任何方分享有关它们的信息。除了显而易见地试图压制对苹果的 API 中存在的缺陷或反竞争行为的公开讨论之外，这似乎也违反了 DMA。鉴于所有主要浏览器引擎的代码库都是开源的（包括 Safari 的），似乎没有任何合法的理由需要这样的保密条款。

> 任何以任何方式阻碍或妨碍用户提出其关注或寻求可用补救措施的做法，例如通过协议或其他书面条款中的保密条款，因此应当被禁止。[数字市场法案](https://eur-lex.europa.eu/legal-content/EN/TXT/?toc=OJ%3AL%3A2022%3A265%3ATOC&uri=uriserv%3AOJ.L_.2022.265.01.0001.01.ENG#:~:text=Any%20practice%20that%20would%20in%20any%20way%20inhibit%20or%20hinder%20those%20users%20in%20raising%20their%20concerns%20or%20in%20seeking%20available%20redress%2C%20for%20instance%20by%20means%20of%20confidentiality%20clauses%20in%20agreements%20or%20other%20written%20terms%2C%20should%20therefore%20be%20prohibited)

### 第三方浏览器将被有效地阻止在 iOS 上的第三方应用商店上发布其浏览器。

苹果的提议包括允许第三方原生应用商店与其在 iOS 上竞争。这直接是由该法案强制执行的。

尽管允许第三方原生应用商店不是我们组织的斗争的一部分，但对于浏览器来说，作为免费产品，希望在 iOS 上的每个主要应用商店上都有列表。苹果的提议似乎旨在阻止任何免费应用甚至考虑在 iOS 上的第三方原生应用商店上列出。

为了在第三方应用商店上列出他们的应用，应用开发者必须签署一份允许他们在 DMA 下被授予的权利的备用合同，但该合同的定价明显不同。对于我们这些非律师来说，有一个可选的半合法合同和一个不合法的标准合同是令人困惑的。

合同之间的定价差异值得关注，特别是对于像浏览器这样的免费应用。如果供应商选择第二份合同，苹果将豁免一些应用商店费用，但会增加一项新的“核心技术费”，这相当于每个用户每年 50 美分的费用。

苹果自己的话是：

> 核心技术费（CTF）是欧盟（EU）新业务条款的一个组成部分，反映了苹果通过持续投资于使开发者能够构建和共享创新应用的工具、技术和服务而提供给开发者的价值。开发者可以选择保持 App Store 的当前业务条款，或者采用欧盟 iOS 应用的新业务条款。[苹果文档](https://developer.apple.com/support/core-technology-fee/)

这听起来非常像对硬件和软件 API 的访问费。但是 DMA 明确禁止这种费用：

> 第 6(7) 条 第三方监管者应当允许服务提供商和硬件提供商**免费、有效地与通过其在指定决定中列出的操作系统或虚拟助手访问或控制的相同硬件和软件功能进行互操作性，并为实现互操作性访问**，这些功能与门户提供的服务或硬件提供商提供的服务或硬件相同。此外，第三方监管者应当允许企业用户和提供核心平台服务的服务的替代提供者**免费、有效地与通过其在指定决定中列出的操作系统、硬件或软件功能进行互操作性，并为实现互操作性访问**，这些功能与提供此类服务时，该门户提供的服务相同或使用的硬件或软件功能相同。[数字市场法案](https://eur-lex.europa.eu/legal-content/EN/TXT/?toc=OJ%3AL%3A2022%3A265%3ATOC&uri=uriserv%3AOJ.L_.2022.265.01.0001.01.ENG)（已加重点）

“免费” 是关键词。

令人惊讶的是，苹果增加了一些措辞，阻止开发者切换回标准合同，同时授予自己在任何时候更改任何合同的权利：

> 随时采纳新商业条款的开发者将无法切换回 Apple 现有的欧盟应用程序的商业条款。苹果将继续提前通知开发者我们条款的变更，以便他们可以对自己的业务做出明智的决定。[苹果文档](https://developer.apple.com/support/dma-and-apps-in-the-eu/#dev-qa:~:text=Developers%20who%20adopt%20the%20new%20business%20terms%20at%20any%20time%20will%20not%20be%20able%20to%20switch%20back%20to%20Apple%E2%80%99s%20existing%20business%20terms%20for%20their%20EU%20apps.%20Apple%20will%20continue%20to%20give%20developers%20advance%20notice%20of%20changes%20to%20our%20terms%2C%20so%20they%20can%20make%20informed%20choices%20about%20their%20businesses%20moving%20forward.)

如果浏览器在第三方应用商店上架，将需要支付多少费用？iOS 有 16.5 亿用户。1% 的浏览器市场份额代表着 1650 万用户。如果一个浏览器供应商敢于在第三方商店上架他们的应用，即使只有一次，即使没有人从第三方商店下载它，苹果也会每年向他们收取 825 万美元的费用，每 1% 的市场份额每年收费，且没有任何追索权利以改回先前的合同。

在我们看来（以及许多其他人的看法），这旨在尽可能地使免费应用难以在第三方应用商店上架，剥夺这些应用商店的这些应用，并削弱它们起步的能力。

DMA 还包含了可能禁止这种行为的额外条款：

> 第6(12)条：“守门人应对其软件应用商店、在线搜索引擎和在第3(9)条指定决定中列明的在线社交网络服务的商业用户适用公平、合理和非歧视的一般访问条件。”

苹果的政策对免费应用程序（如浏览器）和第三方应用商店都是不公平、不合理和歧视性的。

如果苹果的提案听起来荒谬，你并不孤单。我们对苹果的律师如何认为这可能符合DMA的规定感到困惑，以及这将如何使本地应用开发者感到疏远的程度感到困惑。尽管他们可能意识到这些提案是多么不受欢迎，但他们仍然愿意推动这些提案，这本身就是市场力量的一个鲜明例证。

我们希望欧盟拒绝这一提案，并迫使苹果制定一个针对在欧盟通过苹果的App Store分发的所有原生应用开发者的单一非选择性合同，其条款公平、合理且非歧视性。

### 苹果应该做什么，以及他们最终将会失败的原因

苹果在这里本可以有所作为。他们本可以宣布允许iOS上的浏览器竞争的全球规则。这些规则可以包括安全的最低标准。他们本可以与每个主要浏览器供应商的团队合作数月，讨论他们需要哪些API以及如何有效地共享它们。他们本可以升级他们的应用内浏览器（SFSafariViewController）和他们的Web应用程序安装功能，以支持用户选择浏览器并允许这些第三方浏览器和Web应用程序进行公平竞争。他们本可以大幅增加Safari的预算，并努力说服消费者接受他们对Web的愿景，以预期激烈的移动浏览器竞争。

苹果没有做到这一点。即使是慷慨的观察者也会得出结论，他们的提案似乎旨在规避数字市场法案的意图和文字。他们的公告充满了对欧盟监管机构的蔑视。

> 这些提案似乎不是为了遵守DMA而是为了维护苹果的封闭式生态系统，
> 
> 应用生态系统中变化的有限范围很少能解决真正存在的权力失衡问题。[阿姆斯特丹大学竞争法专家玛丽亚·罗德里格斯博士](https://pc-tablet.com/clash-of-titans-apples-dma-compliance-proposals-raise-concerns-about-choice-and-competition/)

这是一个错误，尽管苹果大谈保护消费者，但他们将重点放在收入和允许新竞争的限制上，这暴露了他们的真实意图。公众不是傻瓜，也不会被愚弄。

即使在传统上支持苹果的论坛中，对 [苹果的浏览器竞争提案](https://www.macrumors.com/2024/01/26/mozilla-on-apple-eu-browser-engine-change/) 的反应也极为负面。苹果正在失去其可能享有的公平竞争声誉。随着每一项市场调查或法案的通过，这种行为都被迫暴露出来。

## 接下来的事情

苹果的合规提案只是一个提案，欧委会可以全部或部分接受或拒绝。苹果的提交将由欧盟审查，并可能会发生变化。现在网页开发者的声音比以往任何时候都更加重要，因为 OWA 与监管机构保持联系，并可以放大那些致力于在唯一开放的、可互操作的、无税收平台上构建竞争性体验的人们的担忧。

到目前为止，欧盟只是这样说：

> 数字市场公平开放，DMA 将打开互联网的大门。变革已经发生。从 3 月 7 日起，我们将评估公司的提案，并听取第三方的反馈意见。如果提出的解决方案不够好，我们将毫不犹豫地采取强有力的行动。[欧盟产业主管 Thierry Breton](https://www.reuters.com/technology/apple-faces-strong-action-if-app-store-changes-fall-short-eus-breton-says-2024-01-26/)

我们对苹果的提案的初步阅读表明，其计划既不符合 DMA 的文字要求，也不符合其精神要求。随着我们解析文本，我们将提供更多更新。虽然我们有理由为备选浏览器引擎现在能够发布到 iOS 感到高兴，但我们仍然担心，监管者这种戏法会损害 DMA 的意图。公平竞争的网络浏览器和对 iOS 上 Web 应用程序的适当支持并不过分。实际上，我们认为这现在是欧盟公民的法定权利。

> 尽管法律专家预计欧盟会挑战苹果对 DMA 的不诚实合规性，但开发者应该抓住这个机会重新思考他们的本地应用程序附庸地位。他们应该将 web 应用程序推向极限，然后要求进一步改进平台。互联网不需要佣金支付、基于使用情况的技术费用，也不需要平台租户的许可。互联网可以解放 iPhone，即使苹果不愿意。[Thomas Claburn - The Register](https://www.theregister.com/2024/01/27/apple_europe_ios_analysis/)

我们将继续斗争，并将向欧盟提出监管者的合规计划存在的缺陷和风险。

敬请关注，随着更多信息的公布，我们将就其进展发布帖子。

## 链接

我们认为读者能够自行决定非常重要。

[这是 DMA 的完整文本](https://eur-lex.europa.eu/legal-content/EN/TXT/?toc=OJ%3AL%3A2022%3A265%3ATOC&uri=uriserv%3AOJ.L_.2022.265.01.0001.01.ENG)。最令人感兴趣的是 [第 5 条](https://eur-lex.europa.eu/legal-content/EN/TXT/?toc=OJ%3AL%3A2022%3A265%3ATOC&uri=uriserv%3AOJ.L_.2022.265.01.0001.01.ENG#:~:text=on%2Dgoing%20basis.-,第三章,限制可争夺性或不公平的门户守门人行为的做法-第 5 条) 和 [第 6 条](https://eur-lex.europa.eu/legal-content/EN/TXT/?toc=OJ%3AL%3A2022%3A265%3ATOC&uri=uriserv%3AOJ.L_.2022.265.01.0001.01.ENG#:~:text=Obligations%20for%20gatekeepers%20susceptible%20of%20being%20further%20specified%20under%20Article%C2%A08) ，只有几页，但是从第一页开始的引言为这些条款提供了背景，比如提到了[网络应用](https://eur-lex.europa.eu/legal-content/EN/TXT/?toc=OJ%3AL%3A2022%3A265%3ATOC&uri=uriserv%3AOJ.L_.2022.265.01.0001.01.ENG#:~:text=web%20software%20applications)。

以下是链接到苹果公司关于他们新的浏览器引擎规则和 API 的文档的链接列表。