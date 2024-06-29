<!--yml

category: 未分类

date: 2024-05-27 12:50:17

-->

# 开发Flying Toasters，一个适用于Apple Vision Pro的屏幕保护程序：从0到1的旅程 · 嵌入的SIGPROC

> 来源：[https://abhipray.com/posts/flying_toasters/](https://abhipray.com/posts/flying_toasters/)

我最近决定在两个工作之间的两周休息期间制作一个[适用于Apple Vision Pro（AVP）的应用](http://apps.apple.com/us/app/flying-toasters/id6479964879)。我想探索其潜力并学习一些新技能。我的日常工作主要涉及低级固件；我已经有一段时间没有构建用户界面的应用了——我上次在2011年高中时制作了iOS应用！这是一个机会，看看应用开发在2024年如何发展，尤其是现代代码生成工具。

## 灵感和“屏幕保护程序”概念 [*链接到标题*](#inspiration-and-the-screensaver-idea)

*在与朋友们进行头脑风暴之后，有人建议我研究经典的“飞行烤面包机”屏幕保护程序，由After Dark（1989年发布）制作。我观看了其视频，并认为这将是一个完美的起始项目！*

*[https://www.youtube.com/embed/Ft5DIBAvXIU](https://www.youtube.com/embed/Ft5DIBAvXIU)

VIDEO*

*我特别兴奋地将“屏幕保护程序”的概念引入这个新平台。在历史上，屏幕保护程序在CRT监视器上有着实际的用途——防止屏幕烧损。来自维基百科：*

> *屏幕烧损，图像烧损，幽灵影像或阴影图像，是电子显示器（如旧计算机监视器或电视机的阴极射线管（CRT））上区域永久变色的现象。这是由屏幕的累积非均匀使用引起的。对抗屏幕烧损的一种方法是使用屏幕保护程序，它会移动图像以确保屏幕上的任何区域不会长时间保持照明状态。*

*如今，屏幕保护程序主要用于美观和娱乐目的，在用户不活动期间激活。我的目标是为AVP屏幕保护程序引入类似的基于不活动的触发器。最初，我的计划是利用凝视跟踪来识别用户可能“走神”的时刻。由于隐私考虑，苹果限制了对这类传感器数据的访问。作为一种变通方法，用户可以通过定期点击应用主控窗口内的按钮来表明他们的活动，有效地重置不活动计时器。该应用还允许用户灵活地定制超时持续时间，将屏幕保护程序转变为温和的提示，让用户从使用AVP中休息。*

*该屏幕保护程序展示了飞行的烤面包机在您的真实环境中翱翔。它们从一个显示太阳的门户出现，并消失在一个显示月亮的门户中。增添一些傻乎乎的小功能也很有趣：*

+   *手势控制：按您的喜好缩放和旋转门户。*

+   *烤面包机互动：轻触烤面包机以触发幽默短语。偶尔还会出现婴儿烤面包机！*

+   *定制化：调整烤面包机数量、烤度、颜色和音乐。或者，如果屏幕上的混乱过于严重，可以激活“幽灵模式”以防止碰撞。*

*查看应用商店上的应用预览和截图以获得视觉效果：[https://apps.apple.com/us/app/flying-toasters/id6479964879](https://apps.apple.com/us/app/flying-toasters/id6479964879)*

*这里是我在开发过程中克服的一些具体挑战：*

+   *学习Swift：虽然我在高中时使用过Objective-C开发iOS应用程序，但我想学习Apple推荐的应用程序编写方式。我花时间阅读[Swift教程](https://carlosicaza.com/swiftbooks/SwiftLanguage.pdf)并在[Swift Playgrounds](https://developer.apple.com/swift-playgrounds/)上练习。ChatGPT和Gemini也是代码编辑和调试的好资源。*

+   *3D动画/艺术：我以前没有3D动画经验。我从一个可3D打印的烤面包机模型开始，并跟随[tutorials](https://www.youtube.com/watch?v=VuMu4tAzFjw)来为翅膀添加动画。创建这些吐司更棘手！我无法为不同烤度找到合适的纹理 – 在大量试验和Photoshop的生成填充尝试后，我终于得到了那些看起来逼真的纹理。*

+   *Portal Gestures: Apple’s [示例代码](https://developer.apple.com/documentation/realitykit/transforming-realitykit-entities-with-gestures?changes=_8) 对于实现门户的直观缩放和旋转手势至关重要。*

+   *Toaster Physics vs. Animation: 我发现用RealityKit的物理引擎和严格的动画路径（使用FromToByAnimation()）并不总是很合适。我的混乱碰撞解决方案是什么？随机发射点，最小距离约束，随机[cubic-bezier](https://cubic-bezier.com)曲线路径，以及在这些烤面包机上的大量线性/角阻尼。如果烤面包机的移动仍然主观上混乱，用户可以选择打开幽灵模式来避免碰撞。*

+   *头部跟踪：我希望烤面包机的幽默短语出现在气泡中，并始终面向用户。RealityKit不直接跟踪头部位置，但我采用了一个聪明的[ARKit workaround](https://stackoverflow.com/questions/77577395/how-to-know-users-position-in-surrounding-space-in-visionos/77616297#77616297)来实现这一点。*

*这个项目太有趣了！如果您拥有AVP并且最终看到飞烤面包机动作，请告诉我您的想法！正如原始的飞烤面包机屏幕保护程序在十年间的演变一样（请参阅[YouTube播放列表](https://www.youtube.com/watch?v=Ft5DIBAvXIU&list=PLxRwNKfqQOI1_pp6Cp4p1MnpCsoqCx2cK&index=2)），如果有足够的兴趣和反馈，我会改进这个体验。所以请分享您的反馈！*
