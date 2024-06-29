<!--yml

类别：未分类

日期：2024年5月29日13:21:37

-->

# Emergetools | Twitter to X深度挖掘

> 来源：[https://www.emergetools.com/deep-dives/twitter-vs-x](https://www.emergetools.com/deep-dives/twitter-vs-x)

*2024年2月22日*

作为iOS应用，Twitter一直如同成熟的典范。自[2010年收购Tweetie以来](https://techcrunch.com/2010/04/09/twitter-acquires-tweetie/)，Twitter一直是总统和企业家们首选的多巴胺解决方案。一切在2022年改变，当埃隆收购了Twitter，并裁减了75%的员工，将其重新品牌为X。

一个产品突然从“成熟”模式转向“快速行动，打破陈规”的模式非常罕见，因此今天我们看一下从Twitter最终版本（v9.54）到X（v10.25）iOS应用实际上发生了什么变化。

## Twitter vs X

在过去约6个月的时间里，自从Twitter变成X后，iOS应用增加了13.3 MB。

X射线衍射可视化展示了这款应用从Twitter变成X的过程。

Twitter和X大多数是相同的应用程序，包括一些大型动态框架，自定义共享和通知扩展插件以及本地化资源。iOS应用程序逃脱了马斯克对整个堆栈进行彻底重写的预测，但仍存在一些显著差异：

## 资产

从应用构建的角度来看，从Twitter到X的最大变化之一是现在应用程序包含了五种不同版本的“X”标志，每个大小约为1-3 MB。顶级资产目录从938 kB增加到11.2 MB。

火星X，星星X，地球X……一定是非常重要的才会占用这么多*空间*。

这些是备用的应用图标 — iOS可以动态切换图标，但这需要1024x1024像素的图像资源。这些资源可能很重，特别是在使用高清地球照片时。

嘿，看，甚至有一个“网络安全”图标！

而上一个版本的Twitter由于图像压缩可能带来的潜在尺寸节省很少，而这些X图像更重，可能会受益于轻微的压缩。

虽然节省10 MB可能看起来很少，但X在全球拥有数百万用户。在这个规模上，即使1 MB也可能对[环境产生重大影响](/blog/posts/CostOfAByte)。

## 动态框架

从Twitter到X的架构变化不大。它主要依赖于一小部分非常大的动态框架，这些框架从Twitter到X保持不变。X的77%是dylibs — 即231 MB中的177 MB。

+   `TwitterSPMMigration.framework`

+   `T1Twitter.framework`

+   `TwitterAppSPMMigration.framework`

+   `PeriscopeSDK.framework`

+   `WebRTC.framework`

除了Twitter原生框架外，WebRTC是一个实时互联网通信框架（也可用于视频通话），而Periscope则是一款已停止服务并并入Twitter的直播应用。这些少量dylib文件显然符合Apple的指导方针，因为[dyld](/glossary/dyld)被优化用于快速链接少量的动态框架。

绝大多数应用代码都存在于这些框架中，这也带来了 Twitter 和 X 中最大的框架。

## SPM 迁移

这些框架的命名揭示了一些信息。

TwitterSPMMigration 的 X 射线

"TwitterSPMMigration" 表明 Twitter 正在从 Cocoapods 或 Carthage 切换到 Swift Package Manager。然而，这个框架至少从 2022 年 5 月（Emerge 首次分析 Twitter 应用时）就存在于应用中。那时，"TwitterSPMMigration" 的大小约为 30 MB，而现在则增加到了约 80 MB。迁移可能仍在进行中，也可能完全暂停。也许 Twitter/X 正在将其用作 Umbrella 框架。你的猜测和我们一样好 🤷。从 Twitter 到 X，我们看到 "TwitterSPMMigration" 的二进制大小增加了 (+3.6 MB)，但似乎没有什么显著的变化。

## 资源重复

X 和 Twitter 中各有 3 个扩展程序：

+   `ShareExtension.appex`

+   `TwitterNotificationContentExtenison.appex`

+   `TwitterNotificationServiceExtension.appex`

所有这 3 个都导入了 `TwitterAppearance.bundle`，它可能包含共享 UI 组件。`TwitterAppeance` 也在主要的 Twitter 目标中。总之，该 Bundle 在应用中被多次使用。将这个 Bundle 作为 dylib 共享，可以减少 Twitter 的大小约 8.4 MB。

这是另一种动态框架是最佳解决方案的上下文。动态框架旨在在两个二进制文件之间共享代码和资源——框架存在于一个地方，动态链接器在启动应用程序或扩展程序时加载它。

"TwitterAppearance" 的重复在收购之前就已存在。在新的 X 应用中，除了我们之前指出的 "X" Logo 外，大多数情况下并没有添加显著的重复。一个例外是我们之前指出的 T1Twitter 框架，在其资产目录中每个 Logo 都被重复两次，尽管这些资产更小，约为 200 KB。

T1Twitter 框架中的 Logo 资源重复

## Grok 是怎么样的？

在 12 月，X 宣布推出 Grok，其 AI 聊天机器人的版本。当前版本的 X 包含与 Grok 功能相关的多个类，您可以在下面看到。

## 结语

Twitter 应用包从 X 版本发布以来并没有经历过任何根本性的变化。无论是因为迁移未完成还是出于快速发布的战略倾向，应用的架构特色在 X 版本中都保持不变。

最后的思考：你*可能*不需要那么多应用程序中的 Logo。

## 分析链接
