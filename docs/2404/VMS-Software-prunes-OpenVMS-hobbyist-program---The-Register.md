<!--yml

category: 未分类

date: 2024-05-27 13:03:41

-->

# VMS Software 精简 OpenVMS 业余爱好者计划 • The Register

> 来源：[https://www.theregister.com/2024/04/09/vsi_prunes_hobbyist_prog/](https://www.theregister.com/2024/04/09/vsi_prunes_hobbyist_prog/)

对于那些希望在非生产环境中使用 OpenVMS 的人来说，这是个坏消息。旧版本正在消失，条款变得更加严格。

OpenVMS 的持续开发者，VMS Software, Inc.，简称 VSI，宣布了最新的[社区计划更新](https://vmssoftware.com/about/news/2024-03-25-community-license-update/)。消息不太妙：Alpha 和 Itanium 版本不再提供，只有限制的 x86-64 版本。

OpenVMS 是大型重要操作系统的始祖之一。作为启发了 DOS、CP/M、OS/2 和 Windows 的操作系统以及 Unix 首次迈向 32 位的硬件的原生操作系统，VMS 已经存在了近半个世纪。几十年来，其各种所有者提供了各种["业余爱好者计划"](https://www.openvmshobbyist.com/news.php)，在这些计划下，你可以免费获取安装和运行许可证，只要不用于生产。

自 Compaq 收购 DEC，然后 HP 收购 Compaq 以来，其前景看起来乌云密布。HP [正式终止](https://www.theregister.com/2013/06/10/openvms_death_notice/)它于 2013 年，然后在 2014 年 [宽恕它](https://www.theregister.com/2014/07/31/openvms_spared/)并出售它。新所有者 VSI [将其移植到 x86-64](https://www.theregister.com/2016/10/13/openvms_moves_slowly_towards_x86/)，并在 2022 年发布了[新版本 9.2](https://www.theregister.com/2022/05/10/openvms_92/)。

去年大约这个时候，我们报道了 VSI [增加了对 AMD 的支持并开设了一个自己的业余爱好者计划](https://www.theregister.com/2023/04/13/openvms_921_x86_hobby/)。从最新的公告看，他们似乎对市场的反应感到失望：

尽管 HPE 在 2020 年停止提供 OpenVMS 原始 VAX 版本的业余爱好者许可证，但 VSI 仍继续维护 OpenVMS 8（也就是 Alpha 和 Itanium 版本），同时致力于 x86-64 版本 9 的开发。VSI 甚至提供了[学生版](https://training.vmssoftware.com/student-license/)，其中包括一个[免费的 Alpha 模拟器](http://mail.migrationspecialties.com/FreeAXP.html)和一个用于其中运行的 OpenVMS 8.4 的副本。

这些许可证将在 2025 年到期，并且不会续订。如果你有老式的 DEC Alpha 或 HP Integrity 机器，你将无法为它们获取合法的 OpenVMS 许可证，或者续订现有安装的许可证 - 除非你付费。

仍然会有一个[社区许可证](https://vmssoftware.com/community/community-license/)版本，但从现在开始只支持 x86-64。虽然 OpenVMS 9 主要面向虚拟化监控器，但它确实支持在单一型号的 HPE 服务器上进行裸金属操作，即[ProLiant DL380 Gen9](https://www.hpe.com/psnow/doc/c04346247)。如果您有其中一个用来玩耍——好吧，很抱歉。现在社区用户只能获取一个 VM 镜像，以 VMWare 的 `.vmdk` 文件提供。它包含一个准备就绪的“OpenVMS 系统磁盘，其中包含安装了 OpenVMS、编译器和开发工具。”其许可证有效期为一年，到期后您将获得一个全新的副本。

这意味着您将无法配置自己的系统并使其保持运行——您将不得不每年从头开始重新创建它。对于那些拥有较旧系统的人来说，唯一的选择是[申请](https://share.hsforms.com/131UO5x_iSEGxlYs8pQTYSQdi37l)成为 OpenVMS 大使。

这个决定*是*可以理解的。再也没有新的 Alpha 或 Itanium 硬件，也永远不会有了。VSI 是一家企业，而且不是大企业；它需要赚钱。在维护 v8 版本的同时，它继续为其提供业余爱好者许可证，但现在，如果 OpenVMS 有未来，那就是 v9 版本。VSI 在创建新的 x86-64 版本上投入了大量的精力和资金。第 9 版是它唯一的生存机会，所以这是它想要推广的。

但说实话……极少有人对老式企业软件如此热衷，以至于他们想在地下室运行旧大型服务器，并保持它们合法授权。疏远这些人不是什么好事，而且肯定不会有帮助。*The Reg* 自由开源软件桌面怀疑主要结果将鼓励他们升起骷髅和交叉骨旗，成为软件盗版。®
