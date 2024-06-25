<!--yml

类别：未分类

日期：2024-05-27 15:20:35

-->

# Wayland 上的 Wine：年度回顾（和展望）

> 来源：[`www.collabora.com/news-and-blog/news-and-events/wine-on-wayland-a-year-in-review-and-a-look-ahead.html`](https://www.collabora.com/news-and-blog/news-and-events/wine-on-wayland-a-year-in-review-and-a-look-ahead.html)

Alexandros Frantzis

2024 年 1 月 30 日

2023 年对于 Wine 的 Wayland 驱动程序来说是一个很棒的一年。我们的目标是从实验阶段迈向将驱动程序作为完整的上游组件。一年后，在[多个合并请求](https://gitlab.winehq.org/wine/wine/-/merge_requests?scope=all&state=merged&search=winewayland)之后，现在已经有很多人能够使用[最新的 Wine 发布](https://gitlab.winehq.org/wine/wine/-/releases/wine-9.0)在一个完全无 X11 环境中享受一些他们最喜爱的 Windows 应用程序！

到目前为止，我们在上游已经拥有了如下功能：

+   基本窗口管理（全屏，最大化，调整大小等）

+   软件渲染（即 GDI）

+   鼠标支持，包括鼠标视角

+   键盘支持，包括键位映射处理

+   Vulkan，包括通过 WineD3D/Vulkan 或 DXVK 支持的 Direct3D

+   对 HiDPI 的基本支持

然而，我们的工作尚未完成。我们将继续在 2024 年进行上游努力，重点放在：

+   通过合成器缩放来模拟显示模式更改

+   OpenGL 支持

+   改进瞬态窗口的定位（弹出窗口，菜单等）

+   更多窗口管理（例如最小化）

+   剪贴板和拖放

+   全面的鲁棒性改进，错误修复，代码改进

最终希望拥有的一些其他功能：

+   在 Wine 核心中支持系统 DPI 自动检测和理想情况下每个监视器的 DPI 处理

+   与即将到来的 Wayland 颜色管理（和 HDR）协议的集成

+   跨进程渲染

在过去的每次驱动程序更新中，我都包括了一个展示我们取得进展的视频。然而，今年已经有几个人使用了 Wayland 驱动程序制作了几部视频（这看起来非常令人兴奋），所以我会让这些视频说服你们！

祝您愉快！
