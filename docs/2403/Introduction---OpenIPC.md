<!--yml

category: 未分类

date: 2024-05-27 14:31:43

-->

# Introduction - OpenIPC

> 来源：[https://openipc.org/](https://openipc.org/)

### OpenIPC 是您 IP 摄像机的替代开源固件。

OpenIPC 是一个面向多个制造商的 ARM 和 MIPS 处理器 IP 摄像机的开源操作系统，旨在取代那些由供应商预装的闭源、不透明、不安全、经常被抛弃和无法支持的固件。

OpenIPC 固件提供二进制预编译文件，方便最终用户安装。此外，我们为任何有能力的程序员提供完整的源文件访问，以便进一步开发和改进项目。OpenIPC 的源代码根据最简单的开源许可协议之一，[MIT 许可证](https://opensource.org/licenses/MIT)，给用户明确的权限，即使作为专有软件的一部分也可以重用代码。我们只是礼貌地要求您将您的改进回馈给我们。我们将感谢任何反馈和建议。

[安装说明](https://github.com/OpenIPC/wiki/blob/master/en/installation.md) [预编译的二进制文件](/supported-hardware) [GitHub 上的源代码](https://github.com/OpenIPC)

OpenIPC 固件使用 [Buildroot](https://buildroot.org/) 构建其 Linux 发行版，并使用 Majestic、[Mini](https://github.com/OpenIPC/mini) 或 [Venc](https://github.com/OpenIPC/silicon_research) 流媒体器。

Majestic code 虽然不是开源的，但为各种硬件提供了前所未有的性能和功能。Majestic 流媒体的作者正在考虑在他筹集到足够资金支持进一步开发后，开源代码库的可能性。你可以通过 [支持开源](/support-open-source) 来加快这一进程。

### 为什么选择 OpenIPC 固件？

首先，OpenIPC 固件为您带来自由。使用 OpenIPC，您可以完全控制摄像头的系统，以及您希望摄像头如何行为的细节。没有后门、没有僵尸网络、没有加密挖矿恶意软件、没有键盘记录器、没有密码嗅探器，没有任何您在无法访问源代码的封闭二进制系统中可能期望的东西。

至于固件的功能，我们致力于为各种摄像头提供通用支持，因此我们首先关注基本功能，使最终用户能够升级其摄像头的固件，并在支持的情况下流视频和音频，而无需太多麻烦。

尽管如此，我们有一些有趣的点。我们的固件支持外部 IPEYE 云存储，向 YouTube 和 Telegram 流视频，使用 SOCKS5 代理，设置虚拟隧道等功能。

此外，还有几个专注于专用设备的项目，如无人机上安装的摄像机，安装在建筑头盔上的摄像机，用于测量工具的使用，用于视觉器官的医学研究，水下捕鱼等等。
