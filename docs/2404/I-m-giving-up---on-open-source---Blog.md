<!--yml

category: 未分类

date: 2024-05-27 13:30:37

-->

# 我放弃了 —— 关于开源的一些想法 - 博客

> 来源：[https://nutjs.dev/blog/i-give-up](https://nutjs.dev/blog/i-give-up)

一年多前，我写了一篇关于[开源以及为什么我对我的某些插件收费](/blog/money)的博客文章。可悲的是，一年后，我已经到了不再愿意继续之前方式的地步。

所以这是我的辞职信。

## 为什么？

自从我开始使用 Linux，我就被开源的理念所吸引。我几乎所有自己构建的东西都是开源的，如果我发现可以改进的地方，我仍然会对我使用的上游项目进行贡献。

十多年前，我还在大学时第一次赞助了一个开源项目，因为我始终坚信，如果一个项目对我有价值，那么支持它是值得的。即使我没有时间亲自为其做贡献，我至少应该支持那些在做贡献的人。

当然，也有人明确表示不想要任何形式的赞助，但如果他们需要，我乐意帮助。在一个开源项目上工作仍然是工作，如果你做得不错，应该得到回报。我也一直认为，如果你开始的项目对公司有价值，他们会回报你，至少这是我自己的公司每个月赞助[Verdaccio](https://verdaccio.org/)和我赞助我依赖的库的维护者的原因。

因此，由于这些天真的信念，我开始在 Apache-2.0 许可下工作 nut.js，因为我认为如果公司和个人都能宽容地使用我的软件，他们也会愿意回报我。现在，在你开始批评我只为了钱时，你不认为全职从事开源项目，仍然能支付账单听起来很棒吗？

结果如何？没有。我得到的只有抱怨。

起初，人们抱怨说图像搜索插件被集成到 nut.js 核心中，他们被迫使用特定版本的 node 或 Electron。然后他们开始抱怨图像搜索插件不兼容 Apple Silicon。我已经明确表明，如果没有机器可以测试，我无法修复这个问题。所以如果没有人愿意借给我一台机器或赞助我买一台，这个问题就无法解决。

你以为有人会采取行动吗？没有。

曾经我决定自己投资，但为新插件收费后，我突然成了那个贪婪的混蛋，不再免费提供一切的人。

公司也是一样。只要一切顺利运行，没有人关心你，但一旦出了问题，猜猜谁来敲我的门？

~~在 nut.js 仓库上的这个公开问题~~，公开指控我某事，完全不属实，这是最后一根稻草。

-- EDIT --

前面链接的问题引发了相当多的反应，有好的也有不好的。

甚至 GitHub 也联系了我，询问是否可以删除该问题，因为情况变得有些棘手。

供参考，这是已删除所有姓名和账户的精简版本：

```
> Not only did you sell out, you also removed all the old versions that were released under an open source license so that others couldn't continue to use out-of-support versions.

`@nut-tree/plugin-ocr` has never been publicly available, nor was any other of the paid addons that were released after introducing non-free addons **almost 2.5 years ago**.

> You should do a better job updating your documentation so that people do not waste their time like I did. This change to closed source was announced where, exactly? All of your READMEs and documentation sites do not mention this

This is mentioned on the nutjs.dev landing page and e.g. [pricing page](https://nutjs.dev/pricing/pricing). The core readme refers to the website for additional information, so I’d say it’s fair enough to expect people to actually read it.

> tl;dr get off GitHub and npm entirely if you want to do the closed-source thing, kthx.

It was a pleasure to meet you, have a great day! 
```

-- 编辑结束 --

这种情况已经发生过几次。在 Discord、Reddit 和现在 GitHub 上，我因为使用 nut.js 的事情被侮辱，但这次，我不只是忍气吞声。

对你来说，开源可能很棒，因为可以免费使用。但事实上，它绝非免费。总有人在为此付出代价，如果不是用户，就是维护者。

每个人的时间都很宝贵，你可能希望明智地使用它。如果花时间在某件事情上很有趣，那很好。但如果它变成负担，那就不再有趣了。

如果人们因为你在空闲时间所做的事情而开始侮辱你，那就是停止的时候了。

开源很棒，但并非可持续。我们在几十年间自我破坏，现在已到难以回头的地步。为了更大的利益公开源代码是一项高尚的事业，但老实说，我认为多年来，使用“开源”已成为逃避软件付费的借口。

如果出了什么问题，责任归谁？当然是维护者。

我玩了将近六年的 nut.js 这个游戏，但现在该结束了。

## 接下来呢？

所有关于 nut.js 的包都将停止在 npm 上公开存在。可用的即用包只能通过私有的 nut.js 包注册表获取，这需要订阅才能使用。

GitHub 仓库将保持公开，所以如果你想继续独自使用 nut.js，你将需要自行负责构建、测试和托管包。

如果你想节省时间和工作量，今天就应该获取许可证，因为随着额外插件的发布，价格也会增加。现有订阅者不会受到此增加的影响。

## 我会完全停止开发 nut.js 吗？

绝对不会。

我会继续开发 nut.js，但更新仓库会有所延迟。新功能、补丁、错误修复和安全更新将首先提供给订阅者。

正如我所说的，如果你想继续使用 nut.js，你将需要自行负责构建、测试和托管包。

一切顺利

Simon
