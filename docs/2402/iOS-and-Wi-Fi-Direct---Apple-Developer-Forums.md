<!--yml

category: 未分类

日期：2024-05-29 13:20:08

-->

# iOS 和 Wi-Fi Direct | Apple 开发者论坛

> 来源：[https://forums.developer.apple.com/forums/thread/12885](https://forums.developer.apple.com/forums/thread/12885)
> 
> 当两台设备不需要处于同一网络时，它是如何做到的？

通过一个 Apple 特定（非 Wi-Fi Direct）的点对点 Wi-Fi 协议。

> 看起来 WiTap 示例代码要求所有实例必须已经在同一个网络上，是这样吗？

不。虽然不要相信我的话，自己试试看：

1.  抓住两个 iOS 设备

    **注意** 唯一的要求是它们必须足够新，有闪电连接器。点对点 Wi-Fi 支持与闪电连接器无关，这只是一个有帮助的巧合，支持点对点 Wi-Fi 的硬件也恰好有闪电连接器，因此这是一个易于识别该支持的简便方法。

1.  在每台设备上，忘记基础设施 Wi-Fi 网络

1.  在每台设备上，禁用蓝牙（它实现了另一种 Apple 特定的 Bonjour + TCP/IP 点对点网络）

1.  在每台设备上运行 WiTap 并开始一个‘游戏’

> 这是我们试图解决的问题：没有基础设施 Wi-Fi 网络。我们的外围设备可以托管一个网络，但我们希望避免用户离开我们的应用去设置中连接到我们的外围设备。

啊，这是在本主题中首次提及非 Apple 硬件。对于非 Apple 硬件，情况更具挑战性，因为点对点网络协议（无论是 Wi-Fi 还是蓝牙）未经第三方使用文档化。

你的选择：

在典型的家庭配件设置中，你希望配件加入与家中所有其他设备相同的基础设施 Wi-Fi。苹果通过无线配件配置（WAC）机制明确支持这一点。这允许 iOS 设备将其 Wi-Fi 配置传递给配件，以便配件可以加入与当前 iOS 设备相同的 Wi-Fi 网络。

你可以在 WWDC 2014 会话 701 "为 iOS 和 OS X 设计配件" 中了解更多信息。

[https://developer.apple.com/videos/](https://developer.apple.com/videos/)

WAC 是[MFi](https://developer.apple.com/programs/mfi/)的一部分。

+   HomeKit 配件，也是在 MFi 的监督下创建的，拥有它们自己的一套能力。如果你对这个方面感兴趣，请告诉我。

+   大多数其他的即时网络和点对点 Wi-Fi 场景没有良好的解决方案。通常你可以使事情运转，但倾向于是一个手动的过程。

分享并享受

—

Quinn "The Eskimo!"

苹果开发者关系，开发者技术支持，核心 OS/硬件

```
let myEmail = "eskimo" + "1" + "@apple.com"
```
