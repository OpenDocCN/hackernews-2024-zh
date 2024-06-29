<!--yml

category: 未分类

date: 2024-05-29 12:31:47

-->

# Firefox 124 为 Mac 和 Android 带来更多新的动作 • The Register

> 来源：[https://www.theregister.com/2024/03/19/firefox_124/](https://www.theregister.com/2024/03/19/firefox_124/)

最新版本的 Firefox 在多个硬件类别上都有改进，使其更好地适应各种硬件环境。

如果您使用 Mac 或 Android 设备，您很可能会在 [Firefox 版本 124](https://www.mozilla.org/en-US/firefox/124.0/releasenotes/) 中注意到这些改进。在此版本中，[Android 版本](https://play.google.com/store/apps/details?id=org.mozilla.firefox) 现在允许您通过从屏幕顶部向下拉动来刷新页面——这是大多数平板电脑的标准手势。

在 Mac 上，该程序在全屏模式下的工作方式有所改进。Firefox 多年来一直有全屏模式，但长期以来并未使用官方的 [macOS API](https://developer.apple.com/documentation/appkit/nswindow/stylemask/1644530-fullscreen) 实现。大约四年前通过一个 [可选设置](https://apple.stackexchange.com/questions/305883/firefox-on-macos-use-native-fullscreen-on-videos) 可以使用它，关闭了 [bug #1403085](https://bugzilla.mozilla.org/show_bug.cgi?id=1403085)，但现在默认开启。

(*The Reg* 自由软件开发桌面团队自从 System 6 时代就开始使用 Mac，但近年来发现一些年轻人将 MacBook 当作带键盘的 iPad 使用，始终以全屏运行所有应用程序，并通过触控板手势在虚拟屏幕之间切换，这让这位老辈的灰髯人感到困惑和不解。据推测，这些人可能会对这种变化感到满意。在这里，我们通过非常实用的 [RightZoom 实用工具](https://blazingtools.com/right_zoom_mac.html) 来禁用绿色窗口控件的全屏行为。)

[光标浏览](https://www.makeuseof.com/what-is-caret-browsing/) 允许您使用键盘在网页上导航，包括跟随链接或在后台标签中打开链接。现在在 Firefox 的内置 PDF 阅读器中也可以使用这个功能，这对视觉障碍用户和那些偏好键盘而非指针设备的重度键盘用户来说非常方便。

Firefox 现在可以使用 [Screen Wake Lock API](https://developer.mozilla.org/en-US/docs/Web/API/Screen_Wake_Lock_API) 阻止屏幕保护和屏幕变暗。这对于在 Firefox 中观看电影，以及导航应用程序、阅读电子书等非常有用。

在[开发者发布说明](https://developer.mozilla.org/en-US/docs/Mozilla/Firefox/Releases/124)中有更多信息，但这些内容更专业一些。我们希望网页开发者会因为[AbortSignal 接口](https://developer.mozilla.org/en-US/docs/Web/API/AbortSignal)现在支持[AbortSignal.any](https://developer.mozilla.org/en-US/docs/Web/API/AbortSignal/any_static)，以及[WebSockets](https://developer.mozilla.org/en-US/docs/Web/API/WebSockets_API)现在支持HTTPS和相对URL，而感到高兴，虽然我们对这些变化并不感到激动。

不幸的是，从最终版本中缺失的一个测试版功能是[Cookie Banner Blocker](https://support.mozilla.org/en-US/kb/cookie-banner-reduction)，尽管在某些地区的用户可能在浏览私密模式时可以享受这个功能。另一方面，如果你不介意这种打扰，可以使用[I don't care about cookies 扩展](https://www.theregister.com/2022/09/21/avast_buys_i_dont_care_about_cookies_addon/)，或将[EasyList cookie 列表](https://easylist.to/)添加到 uBlock Origin。

在撰写本文时，程序下载已经可用，比如[macOS 版本](https://www.mozilla.org/en-US/firefox/mac/)和本地的[Debian Linux 软件包](https://ftp.mozilla.org/pub/firefox/releases/124.0/linux-x86_64/en-US/)，这些都是在[Firefox 122 首次亮相](https://www.theregister.com/2024/01/25/firefox_122_is_out/)时发布的®。
