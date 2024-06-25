<!--yml

category: 未分类

日期：2024 年 5 月 27 日 14:52:53

-->

# WINE 9.0 为 64 位 x86 带来更好的 WoW64 支持 • The Register

> 来源：[`www.theregister.com/2024/01/18/wine_90_is_out/`](https://www.theregister.com/2024/01/18/wine_90_is_out/)

WINE 9.0 带来了更好的 WoW64 支持，适用于 64 位 x86 - *和 Arm* - 工具包，并原生支持 Linux 上的 Wayland。

在过去的六年里，WINE 的发布周期变得令人愉快地规律。[WINE 9.0 已发布](https://gitlab.winehq.org/wine/wine/-/releases/wine-9.0)于周二，差不多一年之后[我们报道了 WINE 8.0](https://www.theregister.com/2023/02/03/wine_80_dxvk_21/)，两年之后我们[报道了 WINE 7.0](https://www.theregister.com/2022/01/19/wine_7/)。

最新版本巩固了我们在之前文章中提到的一些变化。在内部，它将 32 位 Windows API 调用“转换”为 64 位调用，然后再将其转换为 Unix API 调用，开发人员称之为 WoW64，这与[同名的 Windows 功能](https://learn.microsoft.com/zh-cn/windows/win32/winprog64/wow64-implementation-details)类似。正如发布公告所说：

（这里的“PE 格式”是指微软的*可移植可执行*格式，我们在几年前做过[深入解释](https://www.theregister.com/2022/06/20/redbean_2_a_singlefile_web/)。）

公告继续：

没有任何混淆的潜力。

目前，虽然大多数主要的 Linux 发行版只有 64 位版本，但仍然包括对 32 位库和程序的支持...但所有大公司都在考虑放弃这一点。Canonical 在 2019 年考虑过[在 Ubuntu 中放弃对 32 位应用程序的支持](https://www.theregister.com/2019/06/24/steam_wine_ubuntu_32_bit/)，但却收回了这一决定。 Fedora [仍在评估中](https://www.theregister.com/2022/03/10/fedora_inches_closer_to_dropping/)这一点。很有可能，Debian 的下一个版本，[13 "Trixie"版本将不再有 x86-32 版](https://www.theregister.com/2023/12/19/debian_to_drop_x86_32/)，[FreeBSD 15 也不会有](https://www.theregister.com/2023/10/24/freebsd_14_rc2/)。

但 WINE 不仅可以运行在 Linux 和 FreeBSD 上;它也在苹果的 macOS 上运行。最后一个 macOS 10 版本，[10.15 "Catalina"](https://www.theregister.com/2019/10/08/adobe_catalina/)在 2019 年放弃了对 32 位的支持，自那以后的五个版本都不支持 x86-32 代码。

能够在 macOS 11 到 14 上运行 32 位 Windows 二进制文件本身就很有益处，但还有更多：

这并不意味着 WINE 将 x86 代码翻译成 Arm：

Arm64 上的 MacOS 可以通过[罗塞塔 2 翻译器](https://www.theregister.com/2020/11/18/apple_silicon_m1_mac_compatibility/)（甚至可以从 Linux VM 中调用它](https://www.theregister.com/2022/06/09/apple_linux_support_macos/)提供支持，运行 Linux 的 Arm64 的用户将不得不提供自己的翻译器，但 Wine 开发人员指向了[FEX-Emu](https://fex-emu.com/)。

同时，在 Linux 领域，也支持不断增长的 Wayland 显示协议：

与 32 位二进制支持类似，大多数基于 Wayland 的发行版仍然通过 XWayland 支持 X.11 应用程序，但直接的 Wayland 渲染路径有助于性能。这对玩家、流媒体播放器以及与视频工作的人很重要，多亏了 Valve 在 SteamOS 上的辛勤工作，[Windows 应用程序在 Linux 上的性能](https://www.theregister.com/2023/09/27/osseu_steam_os_3/)正在迅速提升。

此版本中还有许多其他变化。还改进了后置处理处理、3D 图形、音频和视频处理、桌面集成、通过 Gecko 支持`MSHTML`渲染、Mon 和.NET 支持等等。

但是，你应该注意，64 位专用和 Wayland 支持，两个关键功能，仍然是可选的：你可以在稳定版本中使用它们，但你必须明确启用它们。WINE 的稳定版本是小心谨慎的东西，大多数情况下，默认设置会避免启用可能会为任何人打破任何东西的东西。*Wine-Stable*是你通常会在 Linux 发行版的存储库中找到的版本。

如果你想要一些仍在积极开发中的花哨功能，有一个单独的分支，名为[Wine-Staging](https://wiki.winehq.org/Wine-Staging)。如果即使这样对你来说还不够新鲜，还有`-development`分支，它大约每两周发布一个新版本，就像[WineHQ FAQ](https://wiki.winehq.org/Wine_User's_Guide#Wine_from_WineHQ)描述的那样。

一如既往，如果你想知道特定的应用程序或游戏是否可用，请先检查[Wine 应用程序数据库](https://appdb.winehq.org/)。还有一些辅助应用程序可以让你更轻松地运行东西，比如[Winetricks 辅助脚本](https://wiki.winehq.org/Winetricks)，它可以自动为特定应用程序安装所需的 Windows 依赖项。

如果所有这些听起来都太麻烦了，你只是想要安装并且工作 - 或者你想要在经济上支持 WINE 的发展 - 那么[像几十年来一样](https://www.theregister.com/2011/01/29/codeweavers_impersonator/)，CodeWeavers' CrossOver 会让事情变得更容易……在 Linux 上，在 ChromeOS 上，以及[在 Mac 上也是如此](https://www.theregister.com/2020/11/19/crossover_apple_m1/)。®
