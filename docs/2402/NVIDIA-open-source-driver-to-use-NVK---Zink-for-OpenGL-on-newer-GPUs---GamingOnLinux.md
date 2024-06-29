<!--yml

category: 未分类

日期：2024-05-29 13:19:00

-->

# NVIDIA开源驱动计划在较新的GPU上使用NVK + Zink来处理OpenGL | GamingOnLinux

> 来源：[https://www.gamingonlinux.com/2024/02/nvidia-open-source-driver-to-use-nvk-zink-for-opengl-on-newer-gpus/](https://www.gamingonlinux.com/)

最近在Mesa Git存储库上的[合并请求](https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/27628)添加了支持让驱动程序选择Zink作为处理OpenGL的翻译层的初始支持。这基本上是通过通用实现在Vulkan上运行OpenGL，使新型显卡更容易支持OpenGL，减少重复代码，并使得在现代游戏中保持OpenGL存在更为容易，尽管尚未被弃用，但在现代游戏中的使用减少。

该合并允许拥有NVIDIA GeForce RTX 20xx系列GPU及更新设备的用户选择使用Zink，而不是依赖默认的NVC0 Gallium3D实现来处理OpenGL请求。如果您想尝试一下，只需在更新Mesa 24.1后设置`NOUVEAU_USE_ZINK=1`环境变量即可，目前此更新仅在Mesa存储库的主分支上可用。

自从从`airlied/zink-driver-choice`分支合并到`main`分支 [2 天前](https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/27628#note_null)，大多数发行版可能尚未分发这些补丁。对于那些使用基于源码的发行版或像AUR这样的任何源码包构建系统的用户，`mesa-git`包 [可用](https://aur.archlinux.org/packages/mesa-git) 用作跟踪主分支并测试这些最新特性的手段。

虽然NVK作为一项技术仍在开发中，还有很多工作要做，但在过去的一年中已经有了许多改进。相关文章：

此外，开发者Mike Blumenkrantz最近在其[博客文章](https://www.supergoodcode.com/woof/)中谈到了NVK的工作，并感谢一些最近的改进，提到“因此，所有的GL游戏现在都在NVK上运行。没有夸张。它们只是运行。”

文章来自[GamingOnLinux.com](https://www.gamingonlinux.com/)。
