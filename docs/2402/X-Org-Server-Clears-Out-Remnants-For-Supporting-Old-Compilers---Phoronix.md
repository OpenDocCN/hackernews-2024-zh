<!--yml

category: 未分类

date: 2024-05-29 13:16:07

-->

# X.Org 服务器为支持旧编译器清除遗留物 - Phoronix

> 来源：[https://www.phoronix.com/news/XServer-Clear-Out-Old-Compilers](https://www.phoronix.com/news/XServer-Clear-Out-Old-Compilers)

短期内仍然看不到新的 X.Org 服务器功能发布的迹象，因为大多数主要利益相关者都在除了 XWayland 部分之外的 xorg-server 上撤资。但对于那些关注的人来说，在过去几天里，X.Org 服务器也有一些 NetBSD/OpenBSD 的构建修复，以及清理一些旧编译器支持的遗留物。

特别是，

[旧的 Sun 编译器支持](https://gitlab.freedesktop.org/xorg/xserver/-/commit/6dafe3dbe658b1a2ad927eceb274808a1ac9bc05)

通过 "__SUNPRO_C" 清除了针对 Sun Studio 编译器的各种设置。此外，旧

[USL 编译器](https://gitlab.freedesktop.org/xorg/xserver/-/commit/27b55301077f33d7fd67611bdc003e72cd321d0b)

检查也已经被移除。

虽然一些人可能会因为 X.Org 服务器的重要性和历史而反对放弃对旧和过时编译器的支持，但是这些环境的支持实际上已经死了。由于 X.Org 服务器从 GNU Autoconf 切换到 Meson 构建系统，而 Meson 不支持这些过时的目标，因此它们无法构建。因此，现在只是清理死代码阶段。

代码库中还进行了一些春季清理，例如这个

[有趣的补丁](https://gitlab.freedesktop.org/xorg/xserver/-/commit/a91a862332a6d5d61aa749a8fd8b87985541a602)

由 Oracle 的 Alan Coopersmith 解释。长期从事 X.Org 开发的开发者解释说：

> "unifdef SUNSYSV
> 
> 我不知道这段代码最初是用来做什么的 - 它是在 1988 年添加的，比 Solaris 2.0 的 SysV R4 发布提前了 4 年，我找不到任何定义了 SUNSYSV 的地方。

X.Org 服务器：除了 XWayland 部分的代码库之外，很少有新功能发布。

[正在持续的一系列安全问题](https://www.phoronix.com/news/X.Org-2024-Six-More-Security)

至少导致了新的补丁发布。看看在 XWayland 范围之外，2024 年是否会有任何新的 X.Org 服务器功能发布是件有趣的事情，鉴于开发者兴趣的减弱以及社区中那些反对 Wayland 但又少有行动的人实际上对 X.Org 服务器开发没有实际贡献。
