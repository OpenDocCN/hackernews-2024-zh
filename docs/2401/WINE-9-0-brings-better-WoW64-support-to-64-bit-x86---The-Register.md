<!--yml

category: 未分类

日期：2024年5月27日14:52:53

-->

# WINE 9.0 为64位x86带来更好的 WoW64 支持 • The Register

> 来源：[https://www.theregister.com/2024/01/18/wine_90_is_out/](https://www.theregister.com/2024/01/18/wine_90_is_out/)

WINE 9.0 带来了更好的 WoW64 支持，适用于64位x86 - *和Arm* - 工具包，并原生支持 Linux 上的 Wayland。

在过去的六年里，WINE的发布周期变得令人愉快地规律。[WINE 9.0已发布](https://gitlab.winehq.org/wine/wine/-/releases/wine-9.0)于周二，差不多一年之后[我们报道了WINE 8.0](https://www.theregister.com/2023/02/03/wine_80_dxvk_21/)，两年之后我们[报道了WINE 7.0](https://www.theregister.com/2022/01/19/wine_7/)。

最新版本巩固了我们在之前文章中提到的一些变化。在内部，它将32位Windows API调用“转换”为64位调用，然后再将其转换为Unix API调用，开发人员称之为 WoW64，这与[同名的Windows功能](https://learn.microsoft.com/zh-cn/windows/win32/winprog64/wow64-implementation-details)类似。正如发布公告所说：

（这里的“PE格式”是指微软的*可移植可执行*格式，我们在几年前做过[深入解释](https://www.theregister.com/2022/06/20/redbean_2_a_singlefile_web/)。）

公告继续：

没有任何混淆的潜力。

目前，虽然大多数主要的Linux发行版只有64位版本，但仍然包括对32位库和程序的支持...但所有大公司都在考虑放弃这一点。Canonical 在2019年考虑过[在Ubuntu中放弃对32位应用程序的支持](https://www.theregister.com/2019/06/24/steam_wine_ubuntu_32_bit/)，但却收回了这一决定。 Fedora [仍在评估中](https://www.theregister.com/2022/03/10/fedora_inches_closer_to_dropping/)这一点。很有可能，Debian的下一个版本，[13 "Trixie"版本将不再有x86-32版](https://www.theregister.com/2023/12/19/debian_to_drop_x86_32/)，[FreeBSD 15也不会有](https://www.theregister.com/2023/10/24/freebsd_14_rc2/)。

但WINE不仅可以运行在Linux和FreeBSD上;它也在苹果的macOS上运行。最后一个macOS 10版本，[10.15 "Catalina"](https://www.theregister.com/2019/10/08/adobe_catalina/)在2019年放弃了对32位的支持，自那以后的五个版本都不支持x86-32代码。

能够在 macOS 11 到 14 上运行32位Windows二进制文件本身就很有益处，但还有更多：

这并不意味着 WINE 将 x86 代码翻译成 Arm：

Arm64上的MacOS可以通过[罗塞塔2翻译器](https://www.theregister.com/2020/11/18/apple_silicon_m1_mac_compatibility/)（甚至可以从Linux VM中调用它](https://www.theregister.com/2022/06/09/apple_linux_support_macos/)提供支持，运行Linux的Arm64的用户将不得不提供自己的翻译器，但Wine开发人员指向了[FEX-Emu](https://fex-emu.com/)。

同时，在Linux领域，也支持不断增长的Wayland显示协议：

与32位二进制支持类似，大多数基于Wayland的发行版仍然通过XWayland支持X.11应用程序，但直接的Wayland渲染路径有助于性能。这对玩家、流媒体播放器以及与视频工作的人很重要，多亏了Valve在SteamOS上的辛勤工作，[Windows应用程序在Linux上的性能](https://www.theregister.com/2023/09/27/osseu_steam_os_3/)正在迅速提升。

此版本中还有许多其他变化。还改进了后置处理处理、3D图形、音频和视频处理、桌面集成、通过Gecko支持`MSHTML`渲染、Mon和.NET支持等等。

但是，你应该注意，64位专用和Wayland支持，两个关键功能，仍然是可选的：你可以在稳定版本中使用它们，但你必须明确启用它们。WINE的稳定版本是小心谨慎的东西，大多数情况下，默认设置会避免启用可能会为任何人打破任何东西的东西。*Wine-Stable*是你通常会在Linux发行版的存储库中找到的版本。

如果你想要一些仍在积极开发中的花哨功能，有一个单独的分支，名为[Wine-Staging](https://wiki.winehq.org/Wine-Staging)。如果即使这样对你来说还不够新鲜，还有`-development`分支，它大约每两周发布一个新版本，就像[WineHQ FAQ](https://wiki.winehq.org/Wine_User's_Guide#Wine_from_WineHQ)描述的那样。

一如既往，如果你想知道特定的应用程序或游戏是否可用，请先检查[Wine应用程序数据库](https://appdb.winehq.org/)。还有一些辅助应用程序可以让你更轻松地运行东西，比如[Winetricks辅助脚本](https://wiki.winehq.org/Winetricks)，它可以自动为特定应用程序安装所需的Windows依赖项。

如果所有这些听起来都太麻烦了，你只是想要安装并且工作 - 或者你想要在经济上支持WINE的发展 - 那么[像几十年来一样](https://www.theregister.com/2011/01/29/codeweavers_impersonator/)，CodeWeavers' CrossOver会让事情变得更容易……在Linux上，在ChromeOS上，以及[在Mac上也是如此](https://www.theregister.com/2020/11/19/crossover_apple_m1/)。®
