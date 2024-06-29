<!--yml

category: 未分类

date: 2024-05-29 12:44:22

-->

# 介绍 Babylon.js 7.0。我们的使命是构建世界上最强大、最美观、最简单和最开放的网络渲染引擎之一… | 由 Babylon.js 提供 | Medium

> 来源：[https://babylonjs.medium.com/introducing-babylon-js-7-0-a141cd7ede0d](https://babylonjs.medium.com/introducing-babylon-js-7-0-a141cd7ede0d)

# 介绍 Babylon.js 7.0

我们的使命是构建世界上最强大、最美观、最简单和最开放的网络渲染引擎之一，今天我们很高兴地宣布，Babylon.js 7.0 迈出了又一步。

Babylon.js 7.0 是对一年来新增功能、优化和性能提升的[庆祝](https://aka.ms/bayblon7Celebration)，为全球的网页开发者开启了新的能力。

让我们深入了解，看看有什么新的好处等着你。

# Procedural Geometry（节点几何）

我们非常高兴地宣布，Babylon.js 现在支持程序几何。我们称其为“节点几何”，这是一个先进的系统，允许你使用非破坏性的节点树系统创建程序几何。

如果你已经熟悉 Babylon 的[节点材质](https://doc.babylonjs.com/features/featuresDeepDive/materials/node_material)和[节点材质编辑器（NME）](https://nme.babylonjs.com/)，那么你在[节点几何编辑器](https://nge.babylonjs.com/)中会感到如同回家一般。这个易于使用的工具允许你组装一个非破坏性的几何树，让你可以创造从微小的几何变化到整个程序化世界/景观的任何东西。

程序几何提供了在运行/构建时创建复杂几何的能力。这意味着最终用户不必下载大型的3D资产。相反，本地设备可以使用CPU创建这些资产。这意味着与要求您的网页访问者下载数百MB的3D资产相比，他们可以下载几KB的节点几何数据，并允许他们的机器创建几何。这为创作者提供了极大的灵活性，以提供独特的网络体验的完美加载/性能优化！

除了亲自尝试之外，没有更好的方式来体验节点几何将会带来的所有新功能！

亲自尝试：[节点几何编辑器](https://nge.babylonjs.com/)

查看演示：[https://aka.ms/babylon7NgeDemo](https://aka.ms/babylon7NgeDemo)

了解更多：[https://aka.ms/babylon7NgeDoc](https://aka.ms/babylon7NgeDoc)

# 全局光照

为了提升网页创作者创造更美观和沉浸式体验的能力，是 Babylon 使命的基础部分之一。通过 Babylon.js 7.0，我们很高兴地介绍了基本的全局光照支持。这一高度期待和先进的功能允许 Babylon.js 场景以更接近现实的方式渲染更生动的体验，通过让光线和阴影在环境中“反弹”。

全局光照代表网络渲染的重大进步，并且与 Babylon.js 7.0 带来的所有内容一样，完全免费和开源。要真正欣赏其为场景增添的美感，你只需亲自体验。

全局光照演示：[https://aka.ms/babylon7GIDemo](https://aka.ms/babylon7GIDemo)

在这里了解更多信息：[https://aka.ms/babylon7GIDoc](https://aka.ms/babylon7GIDoc)

# 高斯分布渲染

[高斯分布](https://en.wikipedia.org/wiki/Gaussian_splatting) 是一种使用[神经辐射场](https://en.wikipedia.org/wiki/Neural_radiance_field)、点云和广告牌捕捉和显示体积数据的新技术。更简单地说，它是一种先进的方式，允许人们以无与伦比的视觉保真度和性能捕捉和显示现实世界。

Babylon.js 7.0 增加了在网络上跨所有设备运行时以 60 fps 渲染高斯分布的支持！

尝试一下：[https://aka.ms/babylon7GSplatDemo](https://aka.ms/babylon7GSplatDemo)

在这里了解更多信息：[https://aka.ms/babylon7GSplatDoc](https://aka.ms/babylon7GSplatDoc)

# 摆动物理学

Babylon.js 7.0 在已有的 Havok 物理支持基础上，继续增加了对摆动动画的支持。这使得任何有骨骼的角色模型可以在按下按钮时自由倒塌和摆动。你还在等什么？赶紧试试吧！

在这里尝试：[https://aka.ms/babylon7RagdollDemo](https://aka.ms/babylon7RagdollDemo)

在这里了解更多信息：[https://aka.ms/babylon7RagdollDoc](https://aka.ms/babylon7RagdollDoc)

# 最先进的 WebXR 支持

Babylon.js 继续支持完整的 WebXR 规范，使得在网络上创建极具沉浸感的体验尽可能简单。通过此版本，Babylon.js 7.0 增加了对几项新的 WebXR 功能的支持，包括：全屏 GUI、可触摸的 UI 元素、世界尺度、抗锯齿多视图，以及同时使用手部和控制器的能力！如果你对在网络上创建沉浸式体验感兴趣，那么这个版本一定适合你！

查看详情：[https://aka.ms/babylon7WebXRDemo](https://aka.ms/babylon7WebXRDemo)

在这里了解更多信息：[https://aka.ms/babylon7WebXRLayers](https://aka.ms/babylon7WebXRLayers), [https://aka.ms/babylon7WebXRHC](https://aka.ms/babylon7WebXRHC)

# 苹果 Vision Pro 支持

苹果的 Vision Pro 是一个令人兴奋的产品，以迷人的新方式融合现实和虚拟世界。我们很高兴分享 Babylon.js 7.0 完全支持 Vision Pro，让苹果粉丝可以通过网络一键体验沉浸式世界。

如果你有 Apple Vision Pro，点击这里进入 Babylon.js 世界：[https://aka.ms/babylon7VPDemo](https://aka.ms/babylon7VPDemo)

# 先进的动画系统更新

计算机动画领域不断发展，而Babylon.js也与时俱进。Babylon.js 7.0为底层动画引擎带来了一些令人兴奋的新功能，为Web上的实时动画解锁了强大的新能力。这些更新增加了混合动画组和遮罩特定动画部分的能力，使创作者可以像以前从未有过的那样精细调整他们的体验。有兴趣用前进步行循环与侧向平移混合，再加上活跃的形态目标唇同步吗？由于动画系统的激动人心的更新，现在这一切都成为可能。

了解更多信息：[https://aka.ms/babylon7AnimGroupDoc](https://aka.ms/babylon7AnimGroupDoc)，[https://aka.ms/babylon7AnimMaskDoc](https://aka.ms/babylon7AnimMaskDoc)

# 现代glTF支持

每个Babylon.js版本都更新了对完整glTF规范的支持。这意味着您始终可以依赖Babylon平台在Web上呈现最先进和前沿的渲染进展。借助Babylon.js 7.0，这一丰富的传统继续延续，并增加了对分散和各向异性glTF扩展的支持。

KHR_materials_dispersion演示：[https://aka.ms/babylon7Dispersion](https://aka.ms/babylon7Dispersion)

KHR_materials_anisotropy演示：[https://aka.ms/babylon7Anisotropy](https://aka.ms/babylon7Anisotropy)

# 一个令人难以置信的社区

Babylon.js由全球最令人难以置信的开源社区之一提供动力和支持。来自全球各地的500多人共同努力，使其成为最强大、最美丽、最简单和最开放的Web渲染引擎之一。借助Babylon.js 7.0，这个社区继续发光，通过向这个充满激情的共享和培育平台添加一些令人难以置信的新功能和扩展。

# 脂肪线

内置于核心引擎中，新的Greased Line系统为Web创作者开启了一些令人兴奋的新可能性。这种新的特殊线条类型是使用网格系统构建的，用于显示任意指定宽度的线条。额外增添的非商标幻想之尘是，这些线条配备了特殊的着色器，使线条始终面向摄像机，因此无论摄像机移动到何处，都可以保持一致的视角。

观看演示：[https://aka.ms/babylon7GLDemo](https://aka.ms/babylon7GLDemo)

了解更多信息：[https://aka.ms/babylon7GLDoc](https://aka.ms/babylon7GLDoc)

# 高级地面投影

这个实在太酷了，简直难以用言语形容，但我们会尽力。想象一下，将一个360度的天空盒/环境，神奇地转变成一个“虚拟”的地面，仿佛支撑您场景中的3D对象。这种视觉错觉在您的场景中提供了从地面到天空的完美平滑过渡。听起来像魔法吗？那你还等什么？快去看看吧！

观看演示：[https://aka.ms/babylon7GProjDemo](https://aka.ms/babylon7GProjDemo)

了解更多：[https://aka.ms/babylon7GProjDoc](https://aka.ms/babylon7GProjDoc)

# 无缝纹理贴花

自Babylon.js 6.0推出以来，纹理贴花为在场景中的3D对象上投影图像提供了强大的新选项。在Babylon.js 7.0中，通过允许贴花在UV边界上无缝映射，该系统被赋予了超能力！这意味着无论您的UV布局是多么漂亮或不那么漂亮，贴花都能完美地适用于任何3D对象！

查看详情：[https://aka.ms/babylon7SeamTsDemo](https://aka.ms/babylon7SeamTsDemo)

了解更多：[https://aka.ms/babylon7SeamTsDoc](https://aka.ms/babylon7SeamTsDoc)

# MMD支持社区扩展

这一激动人心的扩展版本，Babylon.js 7.0，使创作者能够从流行的3D创作软件[MikuMikuDance](https://mikumikudance.en.softonic.com/)（简称MMD）中导入3D资产和动画。除了加载资产和动画外，它还支持IK求解器、形变系统、同步音频播放、播放器控制等等。

查看详情：[https://aka.ms/babylon7MMDDemo](https://aka.ms/babylon7MMDDemo)

在这里了解更多：[https://aka.ms/babylon7MMDDoc](https://aka.ms/babylon7MMDDoc)

# 谢谢您

随着每一次Babylon.js的进化，网络渲染技术都会迎来一场革命，以及一种无比的感激之情。Babylon平台的出色之处，完全依赖于不可思议的开发者社区、500多位贡献者以及坚定的倡导者，他们贡献了他们的知识、专业技能、帮助和热情给这项令人惊叹的技术。“感谢”你们每一位对Babylon.js的贡献，让它成为全球最强大、最美丽、最简单和开放的网络渲染平台之一。
