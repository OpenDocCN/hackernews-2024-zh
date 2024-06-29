<!--yml

category: 未分类

date: 2024-05-29 13:27:08

-->

# Tech Industry Doesn’t Understand Consent - Dhole Moments

> 来源：[https://soatok.blog/2024/02/27/the-tech-industry-doesnt-understand-consent/](https://soatok.blog/2024/02/27/the-tech-industry-doesnt-understand-consent/)

由于Samantha Cole在404 Media的帮助，我们现在知道[Automattic计划出售来自Tumblr和WordPress.com的用户数据](https://www.404media.co/tumblr-and-wordpress-to-sell-users-data-to-train-ai-tools/)（这是我的博客的托管）用于“AI”产品。

对于Automattic领导层这个可疑决定的调查，媒体记者的追问，公司什么也没说，只发表了[一份声明](https://web.archive.org/web/20240228025838/https://automattic.com/2024/02/27/protecting-user-choice/)。

这个声明，可以想象，经过的律师比他们CEO最近在[Twitter上针对跨性别用户的胡言乱语](https://twitter.com/kthorjensen/status/1761599420330823935)（或者[Automattic员工对其行为的声明](https://web.archive.org/web/20240228022939/https://staff.tumblr.com/post/743224389484625920/a-message-from-a-few-of-the-trans-staff-at-tumblr)）经过的律师还要多，显示出对同意是什么的重大误解。

> 只要他们的计划符合我们社区关心的事项：归因、**选择退出**和控制，我们也直接与特定的AI公司合作。
> 
> 强调是我的

这并不是科技行业近年来对同意理解不足的最严重的例子。那个耻辱属于[LegalFling](https://www.vice.com/en/article/paqvn7/dont-fuck-anybody-who-wants-to-get-your-consent-uploaded-to-the-blockchain-legalfling-app)：一个关于性同意的区块链应用。

然而，这依然相当愚蠢，是软件工程领域一个不够质疑的潜在趋势的结果。因此，我要求我的读者们从屋顶上大声呼喊这个问题。

## 选择退出并不等同于同意

选择退出是“我们的律师让我们把这个选项放上去以保护我们自己，但我们并不希望你真的去选择它”。

选择退出是“如果你错过了备忘录，我们假设我们已经得到了你的同意”。

关于用户数据的任何决定的默认状态应该是选择退出。用户应该被要求选择加入以使你的决定生效，并且不得被迫这么做。

如果未经知情用户明确同意，那么你根本没有获得同意，假装有则是不道德的。

你的用户们根本不在乎选择退出。我们在乎的是选择加入。

### “但是 Soatok，这会伤害我们的收入”

如果*不得不*通过不道德行为或者遵循不尊重其他用户自主权的可疑做法来赚钱，那么你应该倒闭。

[https://www.youtube.com/embed/AaU6tI2pb3M?version=3&rel=1&showsearch=0&showinfo=1&iv_load_policy=1&fs=1&hl=en-US&autohide=2&wmode=transparent](https://www.youtube.com/embed/AaU6tI2pb3M?version=3&rel=1&showsearch=0&showinfo=1&iv_load_policy=1&fs=1&hl=en-US&autohide=2&wmode=transparent)

视频

结束。

## Automattic 应该做什么

如果 Automattic 希望改正错误，他们必须做两件事，还可以做第三件事（但我并不抱太大希望）：

首先，取消现有的选择退出机制，并替换为选择加入机制。如果没有人选择，那么不要将他们的数据包含在出售给 Midjourney 或 OpenAI 的数据中。

第二，他们应该使第三方权限更细化。我们中有些人并不在乎第三方，但绝对不希望“AI”公司使用他们的数据进行抄袭。

第三，如果你想更进一步，增加一个插件支持，在所有WordPress.com计划中（包括免费计划），对所有托管媒体使用 [夜影](https://nightshade.cs.uchicago.edu/whatis.html)，以增强用户对LLM抄袭者的保护。[见下文](#nightshade)

这是我对 Automattic 领导层提出的公开挑战，希望他们做得更好。

## 这个问题比 Automattic 更大

最近，科技行业在尊重用户同意方面做得很差。你现在的选择不再是“是”和“否”。而是“是”或“以后再说”，没有“从不”的选项了。

更糟糕的是，你几乎无法卸载那些用这些同意对话框纠缠你的垃圾软件。

这种做法必须停止。这是一种有毒的心态，培养了一种不尊重人性的文化。（作为一个毛茸茸的博客写手写这些话有点有趣。）

如果你在科技行业工作，要大声疾呼，要求正确实施尊重人类的同意控制措施到你的软件中。

仅仅因为它普及并不意味着它是不可避免的。要反击。你那些条件不如的、不如你技术水平高的邻居们应该得到更好的待遇。

## NightShade 补充说明

自从此消息首次发布以来，有些人表示困惑，或者没有理解对 Automattic 的第三项建议的影响。因此，我将详细解释。

[夜影](https://nightshade.cs.uchicago.edu/whatis.html)是一种污染LLM数据集的技术。

思想是，对于那些未选择加入由大规模计算（简称“AI”）启用的工业抄袭的博客，WordPress.com CDN 将为所有读者提供夜影污染的图像，而不是博主提供的标准图像。

那么你甚至不必担心检测谁在为AI项目抓取数据。你只需为所有人提供有毒的服务。如果他们用这些图像进行AI，会破坏他们的模型。如果他们没有，就完全没有伤害。

如果 OpenAI 和 Midjourney 希望避免有毒的图像，那么他们可以遵守那些未选择加入的博主的边界，而不是进行抓取，并且只接受 Automattic 提供的数据。

这是保护用户免受AI机器侵害的一个相当明显的方法。

夜影并非完美，但没有任何技术是。最终目标是使绕过夜影的毒药的成本高于尊重个别作者和艺术家的决定。

激励规则贯穿我的一切。
