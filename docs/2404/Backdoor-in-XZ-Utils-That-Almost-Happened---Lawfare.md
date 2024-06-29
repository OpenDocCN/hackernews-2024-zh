<!--yml

分类：未分类

日期：2024-05-27 13:09:32

-->

# XZ Utils几乎发生的后门事件 | Lawfare

> 来源：[https://www.lawfaremedia.org/article/backdoor-in-xz-utils-that-almost-happened](https://www.lawfaremedia.org/article/backdoor-in-xz-utils-that-almost-happened)

上周，互联网逃过了一次可能对全球网络安全造成灾难性后果的大国攻击。这是一场没有发生的灾难，所以不会引起太多关注——但确实应该引起。这起攻击事件的[故事](https://arstechnica.com/security/2024/04/what-we-know-about-the-xz-utils-backdoor-that-almost-infected-the-world/)以及其[发现](https://www.nytimes.com/2024/04/03/technology/prevent-cyberattack-linux.html)都蕴含着重要的道德意义：全球互联网的安全取决于无数个由更加鲜为人知的志愿者编写和维护的不起眼软件。这是一个不可持续的局面，恶意行为者正在利用这一点。然而，对此几乎没有采取任何实质性的改善措施。

程序员不喜欢做额外的工作。如果他们能找到已经写好的能够实现他们想要功能的代码，他们会使用它，而不是重新创造这个功能。这些代码仓库称为库，托管在GitHub等网站上。这里有关于一切的库：显示3D对象，拼写检查，进行复杂数学运算，管理电子商务购物车，传输文件——无所不包。库对现代编程至关重要；它们是复杂软件的构建模块。它们提供的模块化使得软件项目变得可管理。你使用的一切都包含几十个这样的库：有些商业化，有些是开源的和免费提供的。它们对完成软件的功能性至关重要，也对其安全性至关重要。

你可能从未听说过一个名为XZ Utils的开源库，但它安装在数亿台计算机上。很可能你的计算机上也有。它肯定存在于你使用的任何企业或组织网络中。这是一个可自由获取的数据压缩库。与数百个类似的其他鲜为人知的库一样，它同样重要。

许多开源库，如XZ Utils，都是由志愿者维护的。就XZ Utils而言，这个人是Lasse Collin。自从他2009年编写了它以来，他一直负责这个项目。至少在2022年，他有一些 “[长期的心理健康问题](https://www.mail-archive.com/xz-devel@tukaani.org/msg00567.html)”（需要明确的是，在这个故事中，他不应受到责备。这是一个系统性的问题。）

至少从2021年开始，[柯林被个人针对](https://boehs.org/node/everything-i-know-about-the-xz-backdoor)。我们不知道是谁，但我们有账户名：贾坦、吉加尔·库马尔、丹尼斯·恩斯。这些都不是真名。他们逼迫柯林转移了对XZ Utils的控制权。在2023年初，他们成功了。贾坦花了一整年的时间慢慢地把一个后门整合到XZ Utils中：禁用可能发现他行动的系统，打下基础，最终在今年早些时候添加了完整的后门。3月25日，又有一个名叫汉斯·詹森（另一个假名）试图促使各种Unix系统升级到XZ Utils的新版本。

而所有人都准备好了。这只是例行更新。在短短几周内，它将成为Debian和Red Hat Linux的一部分，而这两者在互联网上的服务器中占据了[绝大多数](https://www.wired.com/2016/08/linux-took-web-now-taking-world/)。但是在3月29日，另一位未受薪资支持的志愿者安德烈斯·弗洛伊德（一个真实的人，他在微软工作，但业余时间做这件事）注意到了新版本XZ Utils处理的异常情况。这种情况很容易被忽视，更容易被忽略。但出于某种原因，弗洛伊德注意到了异常，并[发现了](https://www.theverge.com/2024/4/2/24119342/xz-utils-linux-backdoor-attempt)这个后门。

这是一件[精巧的作品](https://infosec.exchange/@fr0gger/112189232773640259)。它影响了SSH远程登录协议，基本上是通过添加一个隐藏的功能来要求特定密钥才能启用。持有该密钥的人可以使用这个后门的SSH来上传和执行目标机器上的任意代码。SSH以root权限运行，因此该代码可以做任何事情。让你的想象力飞翔吧。

这不是一个黑客能随意制作出来的东西。这个后门是多年工程努力的结果。代码在源代码中如何逃避检测，如何静悄悄地躺着直到激活，以及它的巨大威力和灵活性，都表明了一个普遍接受的假设：一个主要的国家实力派系背后的行动。

如果这个漏洞没有被发现，它很可能最终会出现在互联网上每台计算机和服务器上。虽然不清楚这个后门是否会影响Windows和Mac，但它会影响Linux。还记得2020年，俄罗斯曾经在[SolarWinds中植入后门](https://www.lawfaremedia.org/article/solarwinds-breach-why-your-work-computers-are-down-today)，影响了1.4万个网络？那时候看起来很严重，但这个后门的危害将是数量级上更大。而这场灾难之所以被避免，完全是因为一个志愿者偶然发现了它。而这一切的发生，仅仅是因为第一个未受薪资支持的志愿者，一个事实上成为国家安全的单一故障点，被外国行动者个人针对并利用了才成为可能。

这绝不是运行关键国家基础设施的方法。然而，我们在这里。这是对我们[软件供应链](https://www.atlanticcouncil.org/in-depth-research-reports/report/breaking-trust-shades-of-crisis-across-an-insecure-software-supply-chain/)的一次攻击。这次攻击颠覆了软件依赖关系。SolarWinds攻击针对更新过程。其他攻击则针对系统设计、开发和部署。这种攻击变得越来越普遍和有效，也越来越成为国家的选择武器。

我们无法计算我们计算机系统中有多少这种单点故障。也没有办法知道多少未获报酬和未受到赏识的关键软件库维护者会受到压力。 （再次强调，不要责怪他们。要责怪那些乐于利用他们未付劳动的行业。）或者还有多少人意外创建了可利用的漏洞。还有多少其他的胁迫尝试正在进行？ 十个？ 一百个？ 看起来 XZ Utils 操作不可能是唯一的例子。

解决方案很难。禁止开源不起作用；正是因为XZ Utils是开源的，工程师才及时发现了问题。禁止软件库也不行；现代软件无法没有它们运行。多年来，安全工程师一直在推动所谓的“[软件材料清单](https://www.cisa.gov/sbom)”：一种成分清单，以便在这些包之一受到威胁时，网络所有者至少知道他们是否容易受攻击。行业非常讨厌这个想法，并且多年来一直在与之抗争，但也许[潮流正在改变](https://www.ntia.gov/blog/ntia-releases-minimum-elements-software-bill-materials)。

核心问题在于科技公司不喜欢花费额外的钱，甚至比程序员更不喜欢额外的工作。如果有免费的软件，他们会使用它，并且不会进行太多内部的安全测试。更容易的软件开发意味着更低的成本，意味着更多的利润。市场经济奖励这种不安全性。

我们需要一些可持续的方法来资助成为事实上关键基础设施的开源项目。公开羞辱在这里可能会有所帮助。 [开源安全基金会](https://openssf.org/)（OSSF）成立于2022年，在另一个开源库——Log4j中发现关键漏洞之后，[解决了这个问题](https://alpha-omega.dev/)。大科技公司在关键的Log4j供应链漏洞之后承诺了3000万美元的资金，但从未兑现。他们仍然乐意利用所有这些免费劳动和资源，正如[最近的微软趣闻](https://aussie.zone/post/8540166)所示。受益于这些自由可用库的公司需要真正挺身而出，政府可以强制他们这样做。

如果公司愿意投入资金，有很多技术可以应用于解决这个问题。责任将起到帮助作用。网络安全和基础设施安全局（CISA）的“设计安全”倡议将会起到帮助作用，CISA最终与[OSSF合作](https://www.cisa.gov/news-events/alerts/2024/02/08/cisa-partners-openssf-securing-software-repositories-working-group-release-principles-package)解决这个问题。当然，这些库的安全性需要成为任何广泛的政府网络安全倡议的一部分。

我们这次非常幸运，但也许我们可以从没有发生的灾难中吸取教训。像电力网络、通讯网络和运输系统一样，软件供应链是[关键基础设施](https://www.atlanticcouncil.org/in-depth-research-reports/report/open-source-software-as-infrastructure/)的一部分，属于国家安全的一部分，并且容易受到外国攻击。美国政府需要意识到这是一个国家安全问题，并开始相应对待它。
