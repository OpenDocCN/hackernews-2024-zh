<!--yml

category: 未分类

date: 2024-05-29 12:48:53

-->

# Wayland 打破了你的糟糕软件 | Oro

> 来源：[https://orowith2os.gitlab.io/posts/wayland-breaks-your-bad-software/](https://orowith2os.gitlab.io/posts/wayland-breaks-your-bad-software/)

X11 简而言之，根本不适合任何现代系统。完全停止。为了让它在现代系统上运行，只是一些hack。不要试图通过“嗯，它对我有效”或“但是 Wayland 不工作”的方式逃避。除非你的工作流程（和硬件）来自20多年前，否则你几乎没有理由坚持使用 Xorg，特别是随着用户体验依赖于更新和新特性的不断恶化。

几乎所有*两个月前没有*工作的东西现在都可以工作了，并且正在取得大量进展，以便几乎每个人都可以使用 - 是的，甚至是你，NVIDIA 用户。或者，在某些情况下，Wayland 甚至不该决定事物应该如何运行 - 这纯粹取决于你选择的设置。

说了这么多，让我们继续吧。请期待我直言不讳，冗长。我还会有些技术性内容。看到 X 有多可怕后，我可能会开始流泪。

如果有任何需要改进的地方，或者发现有错的地方，请提交问题或拉取请求到（截至撰写本文时）[我的网站存储库](https://gitlab.com/OroWith2Os/orowith2os.gitlab.io)

## Wayland 就是这个[](#wayland-is-it)

*Wayland 是新一代桌面应该呈现的样子，也是一些*实际上*呈现的样子！*

*不再让应用程序在未经许可的情况下监听您的按键，或者干扰您的显示。改进的电池寿命。更简单的 API。*

*[Marcan](https://marcan.st/)，一位曾协助建立[Asahi Linux](https://asahilinux.org)的人，写了一系列文章详细说明了这些事情，虽然更多是关于技术性的内容，解释了为什么 Wayland 被认为是它的标签。*

*让我们看看[这个](https://social.treehouse.systems/@marcan/110371565062371963)，它基本上解释了这种情况的一切：*

> *（简化的）X 历史和我们的发展过程。回顾90年代和2000年代，X 直接在用户空间运行显示驱动程序。那是一个可怕的想法，使得 X 和 TTY 层之间的互操作成为噩梦。这也意味着你需要为所有东西编写 X 驱动程序。这也意味着 X 必须以 root 权限运行。如果 X 崩溃，那么整台机器的可用性可能会非常低。*
> 
> *然后 KMS 出现了，并将模式设置移到了内核中。伴随着一个通用的 API，这使得需要为 GPU 编写特定驱动程序来显示内容的需求过时了。但 X 仍然继续使用特定于 GPU 的驱动程序。为什么？因为 X 依赖于2D 加速，在现代硬件中根本不存在这样的概念，所以它仍然需要 GPU 特定的驱动程序来实现这一点。*
> 
> *X的开发人员当然意识到现代硬件不能再进行2D处理了，于是Glamor应运而生，它在OpenGL之上实现了X三十年来的2D加速API。现在你可以在任何带有3D驱动程序的现代GPU上运行X了。因此，最终我们可以在没有任何特定于GPU的驱动程序的情况下运行X，但由于X仍然希望有“一个驱动程序”，因此出现了xf86-video-modesetting，这本来应该是未来的方向。它打算在任何带有Mesa/KMS驱动程序的现代GPU上运行。*
> 
> *这是在2015年。而问题在于，X在那时已经开始衰退。显示设置非常糟糕。英特尔淘汰了他们的GPU特定DDX驱动程序，并且开始腐烂，但是直到今年（2023年，整整8年后），显示设置才能处理无撕裂的输出。只需问问任何使用Ivy Bridge/Haswell时代英特尔用户，这一切是多么混乱。与此同时，Nvidia和AMD继续维护它们各自的DDX驱动程序，并且大部分时间让用户免受核心X慢慢死去的影响，因此人们认为这是一个平台/供应商的问题，尽管X已经拥有本应该是平台中立的解决方案，只是未能达到预期的水平。*
> 
> *因此，当像ARM系统这样的其他平台出现时，我们卡在了显示设置中。没有人想编写X DDX。甚至没有人知道，除了曾经做过的人之外，那些人早已筋疲力尽。因此，如果你不是AMD或Nvidia，X将永远被困在一个次优的体验中，因为本来应该处理所有这一切的核心通用代码实际上并不尽如人意。*
> 
> *此外，ARM平台必须处理分离的显示和渲染设备，这是显示设置无法自动处理的。所以现在我们需要特定于平台的X配置文件才能使其工作。*
> 
> *然后还有我们。当苹果设计M1时，他们决定在显示控制器中放置一个协处理器CPU。而不是在macOS中运行显示驱动程序，他们将大部分内容移到固件中。这意味着从Linux的角度来看，我们并不是在裸机上运行，而是在macOS合成器的抽象层上运行。这个抽象层没有像vblank IRQ这样的东西，也没有传统的光标平面，并且对像素格式和色彩空间有非常明确的意见。所有这些都与现代Wayland合成器很好地配合，后者使用了与此模型相当匹配的KMS抽象层（这是未来，其他平台都在朝这个方向发展）。*
> 
> *但是X及其显示设置驱动程序仍然停留在过去。它试图做一些荒谬的事情，比如直接在可见帧缓冲区中进行绘制，而不是后备缓冲区，或者期望有“vblank IRQ”，尽管现在已经不需要了。当没有硬件光标平面时，它实现了软件回退，但代码有问题并且会闪烁。等等。这些都是核心X的问题，遗留的废话和错误。它们只是偶尔会更多地影响较小的平台，特别是伤害我们。*
> 
> *甚至没有涉及到核心X协议的基本问题，比如Mac上无法识别Fn键，因为Mac有软件Fn键，而evdev键码表中的该键码过大，或者像今天所有8个修饰键已被使用，我们需要一个额外的Fn键。这些问题在不破坏X11协议和客户端的情况下无法得到适当的修复。*
> 
> *所以不，X在Asahi上永远不会正常工作。因为它存在bug，已经有8年之久，这段时间内没有人修复它，当然现在也没有人去修复它。试图拥有一个供应商中立的驱动程序的尝试太少太迟，而且到那时动力已经转向Wayland。如果8年前继续X的开发足够长时间以使得模式设置达到标准，今天Asahi的情况可能会不同。但事实并非如此，现在我们面临的现实是，无可挽回。*

*这基本上概括了所有内容，所以我会更详细地讲解一些马坎没有解释清楚的东西。*

## *架构和性能[](#architecture-and-performance)*

**一些新硬件，如Apple Silicon，在固件中处理了大部分显示内容。因此，它实际上与DRM/KMS提供的抽象相似，而Wayland非常适合这种情况。Marcan在这里复制的帖子详细解释了所有这些。**

**这实际上与架构非常密切相关：Wayland的性能更为卓越。**

**在Xorg上，你有几个进程：一个用于显示服务器，一个或两个用于合成器和窗口管理器。你可能会问，为什么这样不好？好吧，那是几个设计效率低下的进程。有一些扩展可以把它包扎起来，但X仍然存在固有的效率问题。毕竟，X服务器处理了很多事情。现在你又添加了合成器和窗口管理器，它们提供效果和窗口装饰。Xwayland继承了大部分核心问题，但仍然有一个显著的区别：在底层有一个更好的合成器，更为系统优化。而且你的大多数应用程序可能不会通过Xwayland运行，因此你可以获得大部分的好处。*

**在Wayland上，解决方案很简单：做更少的事情。协议更简单，合成器的工作量更小，还有一些基本的设计改变，允许合成器更灵活地做一些事情，比如[DMA-BUF access](https://wayland.app/protocols/linux-dmabuf-unstable-v1)，并跳过大部分的合成过程，直接将像游戏和视频这样的内容扫描到屏幕上。**

**因此，使用Wayland，在极端情况下你可以获得一点性能提升，*而且*在移动系统上，你甚至可以获得更长的电池续航！这是一件相当重要的事情，特别是当涉及到像Steam Deck（使用Gamescope，一个Wayland微合成器）或笔记本电脑的时候。*你*希望在上课时笔记本电脑没电吗？我想不会。**

**更不用说使其*更*容易维护了。Xorg的代码基础无法维护，最近甚至达到了[有史以来的开发低谷](https://www.phoronix.com/news/XServer-2022-Development-Pace)！这是*成千上万*个错误，*数十*个功能，全部都不会被修复和添加。哦，总线因子……该死，最近Xorg唯一新增的东西之一是对libei的支持，甚至那也只是为了*Xwayland*。自那以来浏览提交列表，基本上所有东西都是为了Xwayland。**

**现在看看Wayland合成器，你会发现在过去的*一周*内新增了新功能和改进！可以确信，由于核心协议维护和工作的困难性，它们不会很快被放弃。**

**或许可以说这是每个合成器需要做所有事情的原因，但事实上并没有特定的原因说明为什么不能创建像Xorg架构那样的东西。毕竟这就是KDE所做的。Shell和合成器是两个独立的进程。你还有wlroots，它提供了相当多的辅助工具，使其更像是Wayland生态系统的Xorg。**

## **安全性[](#security)**

***如果您曾经使用过Xorg，您可能会注意到一些工具，特别是那些记录屏幕或监听按键的工具，从来不需要权限来执行。这是一个*重大*问题。***

***任意应用程序可以录制到您屏幕上的任何内容。视频通话、私人消息、网页。您能看到的一切都可以在没有您许可的情况下被抓取……以及所有您*看不见*的东西。***

***不想忘记您的密码？别担心，键盘记录器可以为您保存它们，而且完全不需要您告诉它们！它们是不是很好呢！***

***如果你想在Wayland上使用这些功能，你不能擅自去做。这不好™。因此，我们有[ScreenCast](https://flatpak.github.io/xdg-desktop-portal/#gdbus-org.freedesktop.portal.ScreenCast)，[GlobalShortcuts](https://flatpak.github.io/xdg-desktop-portal/#gdbus-org.freedesktop.portal.GlobalShortcuts)和[InputCapture](https://flatpak.github.io/xdg-desktop-portal/#gdbus-org.freedesktop.portal.InputCapture)门户。不要指望在没有权限的情况下使用敏感的API，如果你这样做了，简单地说：请友好地离开 :)***

## ***屏幕共享[](#screensharing)***

***技术上说，Wayland 与屏幕共享毫无关系 - 所有这些都是分开处理的，在 [PipeWire](https://pipewire.org) 和 [xdg-desktop-portal](https://github.com/flatpak/xdg-desktop-portal) 中。***

***要让应用程序获取有关正在显示的任何信息，它们必须通过 xdg-desktop-portal - [ScreenCast](https://flatpak.github.io/xdg-desktop-portal/#gdbus-org.freedesktop.portal.ScreenCast)，具体来说。这将提示用户选择应用程序应具有访问权限的窗口或屏幕。从那里，应用程序仅获取到被允许的资源，并且以高效的方式。***

***看看 Xorg，你会得到一个更糟糕的用户体验：应用程序需要访问 *所有* 的内容，才能看到它们想要捕获的应用程序，而且完全不需要用户输入。而且利用该视频流的内容可能会 *非常* 低效。***

***关于“Wayland 不做任何事情”的规则，有一个例外 - wlroots。他们的一般经验法则是使用 Wayland 协议来处理事务。屏幕共享、输入仿真、远程桌面，所有这些都是在 wlroots 上的 Wayland 协议的基础上完成的。看看 GNOME 和 KDE，它们会优先通过适当的门户进行。但这样做，你无法像屏幕共享那样实现互操作性。所以，你有建立在这些协议之上的门户；对于 wlroots 来说，这就是 [xdg-desktop-portal-wlr](https://github.com/emersion/xdg-desktop-portal-wlr)。***

## ***多监视器缩放[](#multi-monitor-scaling)***

***在 Xorg 上不存在。在 Wayland 上从一开始就存在。遗憾的是，这也影响了 Xwayland。***

***Xorg，从一开始就只需处理一个显示器。毕竟，它起源于大约 1980 年代。甚至没有想到会有多个显示器，更不用说分辨率和比例不同的了。官方文档甚至 [认为“显示”是整个 Xorg 服务器，包括其所有屏幕的结合体](https://www.x.org/releases/X11R7.7/doc/xproto/x11protocol.html#glossary:Display)。***

***Wayland 通过一种有意的设计决策修复了这个问题，这也使得与其他功能的协作变得更加容易：窗口不知道自己表面外面发生的事情。所以它们只是在特定表面上被告知使用什么比例。***

***[refi64](https://refi64.com) 曾经在过去为 Xwayland 做过一些工作，特别是重新基于和更新 [xwayland: Multi DPI support via global factor rescaling](https://gitlab.freedesktop.org/xorg/xserver/-/merge_requests/733)。这主要工作是因为 X11 属性是窗口本地的，类似于在 [Wayland](https://wayland.app/protocols/wayland#wl_surface:event:preferred_buffer_scale) 上获取比例因子的方式。***

***这与通常获取比例的情况不同，它没有定义的规范。有环境变量、`XSETTINGS`、工具包特定的配置，大多数（或全部）都不支持分数缩放。所以解释X如何缩放的最好方法是…… 它不缩放。***

## ***分数缩放[](#fractional-scaling)***

***Linux上的分数缩放故事一直很奇怪。Xorg和Wayland一样，一开始并没有分数缩放的概念。幸运的是，Wayland已经[修复了这个问题](https://wayland.app/protocols/fractional-scale-v1)，但是Xorg的情况仍然是整数比例。这也影响到Xwayland，在这里我会详细介绍。***

***一个解决方案是创建一个巨大的帧缓冲区，具有最大公共比例，并将其缩小以适应必要的显示器，这或多或少是GNOME在Wayland上进行分数缩放之前所做的事情 - 告诉客户端以最大整数比例渲染，并缩小到所需的分数比例。唯一的区别是您对整个Xorg帧缓冲区执行此操作，这意味着整个桌面都以最高公共比例呈现。显然不是理想的解决方案，但这可以完成工作。***

***就像我在上一节中提到的，大多数配置对分数缩放不起作用。实际上，这比它的价值更痛苦。您最好搞定Wayland，摆脱Xorg。***

***说实话，我还会引用[Kenny Levinsen](https://kl.wtf/)关于缩放的一些讨论片段，用于我正在调查的一个诅咒项目：***

> ***首先跳过缩放，对于演示来说并不重要***
> 
> ***稍后，您可以通过将粗体字体DPI转换为分数比例值或类似的东西来进行黑客攻击 - X11是一个黑客，所以您做的东西也将成为一个黑客。***
> 
> ***没有为此担心的必要 :)***

## ***几个刷新率[](#several-refresh-rates)***

***当您只有一个显示器时，一切都很好 - 您的游戏可以直接显示。但是多个显示器会为您的组合器创建一个巨大的帧缓冲区，并在*所有*显示器上绘制。与“正常”显示服务器不同，Xorg不会像双缓冲那样进行常规翻转，而是连续绘制到帧缓冲区，并期望硬件偶尔读取并显示它。对于60Hz显示器，它每秒读取60次，对于120Hz则每秒读取120次，依此类推。***

***这意味着，当你不只有一个监视器时，至少在一个具有合成器的设置中（通常你希望有这个），情况可能会变得很糟糕。撕裂、缺少像 VRR 这样的现代功能等等，因为 Xorg 需要足够的帧更新来适应*所有*这些监视器，所有在一个巨大的窗口上 - 但它做不到，所以它只更新足够让刷新率最高的监视器工作。这不太可能很快修复。你只需要完全绕过合成器，以便在需要任何这些好功能的情况下使用。***

***我想提一下，[pac85](https://gitlab.com/pac85) 有一个绕过缺乏 VRR 的破解，它只是伪造一个页面翻转 - 内核驱动被明确告知执行翻转，但给出相同的帧缓冲。***

***这就像你不断地在一张纸上画画，并希望人们知道在他们自己看的时候发生了什么。现在再来做同样的事情，用多个画，都在同一张纸上，同时被画出来。要获得垂直同步，你只需在有人看的时候不要在那张纸上画。双缓冲，即在正常的 Wayland 合成器上，合成器在后台另一张纸上绘制，并将其与第一张交换，并在每次更新时重复。***

***通过显式执行（伪造的）页翻转，你可以让多监视器支持 VRR，因为 CRTC 知道何时发生页面翻转。它不会在自己的意愿下随意更新。或者换句话说，告诉人们“嘿，这是我的新图”，而不是给他们一个新图并且他们*知道*它发生了变化。这就是下面的内核补丁所允许的。***

*****这是一个破解，有时会失败。小心处理。这不适合在任何地方提交。*****

> ***显式同步的事物会被破坏，内核跟踪 GPU 工作在哪个帧缓冲上运行。所以你可以想象，事情在那里可能出错。***

***Xorg 补丁：[https://gitlab.com/pac85/xorg-server/-/commit/e2a4d5cf8965f7fcc8f07d04cb1e95f5e62a0094](https://gitlab.com/pac85/xorg-server/-/commit/e2a4d5cf8965f7fcc8f07d04cb1e95f5e62a0094)***

***`|***

```
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17

```

|***

```
diff --git a/drivers/gpu/drm/amd/amdgpu/amdgpu_display.c b/drivers/gpu/drm/amd/amdgpu/amdgpu_display.c
index b702f499f5fb..d5ae05f57054 100644
--- a/drivers/gpu/drm/amd/amdgpu/amdgpu_display.c
+++ b/drivers/gpu/drm/amd/amdgpu/amdgpu_display.c
@@ -213,6 +213,13 @@ int amdgpu_display_crtc_page_flip_target(struct drm_crtc *crtc,
        work->crtc_id = amdgpu_crtc->crtc_id;
        work->async = (page_flip_flags & DRM_MODE_PAGE_FLIP_ASYNC) != 0;

+       if (crtc->primary->fb == fb) {
+               adev->mode_info.funcs->page_flip(adev, work->crtc_id, work->base, true);
+               kfree(work->shared);
+               kfree(work);
+               return 0;
+       }
+
        /* schedule unpin of the old buffer */
        obj = crtc->primary->fb->obj[0];

```

|`***

## ***高动态范围[](#high-dynamic-range)***

***HDR 是一个相当新的东西。除了电视和游戏机之外，它在所有地方都还是一个孩子（实际上更接近青少年）。macOS 可能是在这方面最可用的平台。Windows 也支持，但体验可能不稳定，以及它们的自动 SDR -> HDR 转换。***

***在 Linux 上，它已经存在一段时间了，假设你与显示器之间没有任何中间件。这意味着它应该可以，而且确实可以在 Kodi、SDL 和 Gamescope 上运行，这些可以直接在 KMS 上运行。既然你毫无疑问已经看到并听说过它，特别是如果你研究过 Linux 上的 HDR，这里是 *Gamescope* 是如何做到的 - 我将主要从 [Joshua Ashton](https://github.com/Joshua-Ashton) 复制粘贴一些内容：***

> ***Gamescope 在 Wayland 上运行***
> 
> ***它只是用自己的协议发送额外的元数据***
> 
> ***[https://github.com/ValveSoftware/gamescope/blob/7fffcc813c0f1ae48d9f1d4637a508eace889507/protocol/gamescope-xwayland.xml](https://github.com/ValveSoftware/gamescope/blob/7fffcc813c0f1ae48d9f1d4637a508eace889507/protocol/gamescope-xwayland.xml)***
> 
> ***交换链反馈和 HDR 元数据是唯一需要的东西***
> 
> ***它使用这个协议来实现一个 Vulkan 层，将使用 X11 Vulkan WSI 的应用程序转换为 Wayland Vulkan WSI，并在幕后创建一个覆盖表面***

***至于 OpenGL 的问题，如果你尝试走类似的路线：***

> ***如果你想做与我一样的 XWayland 绕过操作，那么在 GL 上会非常痛苦***
> 
> ***需要更多的 mangohud 风格的挂接***
> 
> ***或者实际上也许不是，我不知道***
> 
> ***我通常试图忘记 GL WSI 的工作方式，因为它太可怕了***

***从这个角度看，Gamescope 上实现 HDR 所需的工作与其他合成器和普通的 Wayland 客户端所需的工作基本相同；目前还没有一个 [上游协议](https://gitlab.freedesktop.org/wayland/wayland-protocols/-/merge_requests/14) 可用。***

***至于 X？不会发生。虽然 NVIDIA 提出了一个 [提案](https://lists.x.org/archives/xorg-devel/2017-July/054112.html)，但没人真的有兴趣在 Xorg 上工作并维护 HDR 支持。再次引用 Joshua Ashton 的一两句话：***

> ***我可以给你一个三个字的引用***
> 
> ***“让它死吧”***
> 
> ***更详细的解释是嗯***
> 
> ***X11 的视觉 ID 已经是一团糟……我们最好不要碰这些狗屎，离得越远越好***

## ***NVIDIA[](#nvidia)***

***当然，NVIDIA 像往常一样喜欢做他们自己的事情。如果你想在 Xwayland 上做任何事情，并且没有多个 GPU，就用 Nouveau 吧。***

***详细解释我懒得搞了，如果你感兴趣的话可以去Google一下。不过我会链接到 [这里](https://gitlab.freedesktop.org/xorg/xserver/-/merge_requests/967)，它允许 Xwayland 在 NVIDIA GPU 上工作时避免之前遇到的许多问题，并且能更轻松地在像 Intel 的新内核驱动和 Asahi Linux 这样的新系统上运行。***

***幸好 [NVK 最近被合并到 Mesa 中](https://www.phoronix.com/news/Mesa-NVK-Vulkan-Merged)，所以我们终于可以摆脱专有驱动，并且不会给 NVIDIA 用户带来更糟糕的用户体验（至少对一半用户来说是这样。如果你有 Maxwell 或 Pascal 卡但不支持重频，请向 NVIDIA 提出 bug）。***

## ***应用开发的视角[](#application-development-pov)***

***地狱。***

***如果你正在为上个世纪八十年代编写复古软件，那就用 X。如果你在写一些不是为几十年前的硬件设计的东西，那就要考虑 Wayland。***

***如果你甚至*尝试*和 X 交互，你将花更多时间蜷缩在胎儿姿势抽泣，而不是做任何有意义的事情。***

***Wayland 的问题稍少一些 - 你可能需要一只情感支持 Blåhaj 来陪伴你。***

***或者使用类似 SDL 的东西，并尽量不直接与显示服务器交互。这样你就可以支持其他显示服务器*并且*在此过程中更容易地将你的软件移植到其他平台，如果你愿意的话。***

***为了获得完整的体验，我将写一个迷你 Wayland 服务器和 X 客户端，这应该能教会我很多关于所有这些工作方式的知识，并且我可以满足一下我的虐待狂心理。***

## ***辅助功能[](#accessibility)***

***关于 Wayland 的一个重要事项是它缺乏相应的辅助软件 - 这并不是 Wayland 本身可以解决的问题，如果你仍然希望有安全保证，或者确实在协议范围内。***

***那么我们打算如何解决这个问题呢？当然是通过一个门户！具体来说，是一个[辅助门户](https://github.com/flatpak/xdg-desktop-portal/issues/1046)。这将允许辅助工具在各种合成器上工作，甚至在 Xorg 本身上也可以使用 InputCapture、RemoteDesktop 和 ScreenCast 门户。主要问题只是要弄清楚这些辅助工具的需求，并制定一个它们可以使用的 API。***

***已经存在标准的 a11y 接口，大部分应该已经可以正常工作 - 就是这样。可能还有改进的空间，但它们大部分都可以工作。***

***Linux 上的普遍辅助功能现状有些令人失望，但幸运的是，像[Lukáš Tyrychtr](https://gitlab.gnome.org/tyrylu)和[Tait Hoyem](https://tait.tech/)这样的人正在帮助改进这一状况。***

## ***常见的投诉[](#common-complaints)***

***总会有人抱怨*某事*不能正常工作。因此，这里基本上是一个包含解释和提示的常见问题解答。***

+   ***屏幕录制已经可以使用。Chrome 和 Firefox 支持它，OBS 已经可以工作，而 Electron 只需应用程序更新到较新版本并解决一些小问题即可。大多数应用程序只需更新其工具包。如果你使用的应用程序无法与其配合使用，有可能通过在浏览器中使用它来解决。***

+   ***屏幕撕裂已经具备了适当的协议，并且有多个问题在跟踪其支持。现在只需要内核驱动程序支持它。***

+   ***X 特定的工具：甭想了。X 特定的工具就是 X 特定的，不是 Wayland 特定或通用的。你最好制作 Wayland 特定的工具，或者能在两者上都能工作的东西。但别指望 xrandr 或 xeyes 能在 Wayland 上工作。***

+   ***Barrier/Synergy/远程桌面：已完成，并且已经在 GNOME 和 xdg-desktop-portal 中添加了支持。Wayland 协议待办事项中会构建在 wlroots 之上。特别是远程桌面，已经有一段时间可以使用了，并且得益于 InputCapture portal（Barrier/Synergy 使用的那个），它得到了一些改进。***

+   ***全局快捷键：已经有一个[可用的门户](https://flatpak.github.io/xdg-desktop-portal/#gdbus-org.freedesktop.portal.GlobalShortcuts)可以处理。***

+   ***网络透明性：使用[WayPipe](https://gitlab.freedesktop.org/mstoeckl/waypipe)。Wayland作为一个协议，并不需要支持网络透明性。X11有很多Wayland没有的功能，同样的想法也适用于其他许多功能。***

***另外，请考虑你的功能是否是Wayland需要处理的；在大多数情况下，它并不是。Wayland不需要告诉合成器如何管理它们的窗口，或者定义合成器作为整体如何实现。将一个合成器的工作流程推到另一个合成器上也不合适——它们是不同的合成器，具有不同的工作流程、设计和意识形态。如果你不喜欢某些功能的工作方式，自己动手让它符合你的口味。***

## ***结论[](#conclusion)***  

***你可能可以向Xorg中添加所有（或者大多数）这些功能，但那要经历一些根本性的变化、重写和扩展。到那时……你已经只是又做了一个Wayland。所以甚至不要试图争论你可以“改进Xorg”。你不能。你最好的选择只是得到Xwayland，它甚至几乎不能正常工作。你已经看到，并且被告知，X生态系统已经沉没得多（更像是已经沉没的船）。***

***如果你仍然选择使用Xorg是完全合理的，因为一些功能目前可能还不太完善，特别是在辅助功能方面。但这艘船也只能浮在水面上那么久。偶尔试试Wayland，如果对你不起作用的话，并且关注相关讨论。最终你将不得不使用它，所以适应它。如果你无法接受现状，试着改进情况以便你可以接受。给你本地的自由开源软件开发者捐几块钱。学习如何提交问题。在你能力所及的地方改进事物，让其他人可以在你力所不能及的地方改进。***

***我想我会以这个结束：***

***Xorg：这全都是hack，而且不是所有的hack都有效。***
