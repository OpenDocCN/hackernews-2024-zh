<!--yml

类别：未分类

日期：2024-05-29 12:48:11

-->

# 用旧手机作为服务器的合理效果 -

> 来源：[https://jarbus.net/blog/phone-server/](https://jarbus.net/blog/phone-server/)

我在 OnePlus6T 上安装了 [linux](https://postmarketos.org)。设置不到一小时，技术问题包括在内。

# 为什么[](#why)

+   8GB RAM，128GB 存储，2.8GHz 8 核 CPU，售价 $90

+   [OnePlus 6T 上的 Linux](https://wiki.postmarketos.org/wiki/OnePlus_6_(oneplus-enchilada)) 文档 [详细完善](blog/take-the-road-most-documented/)

+   低功耗

+   *小巧* 的占地面积

+   没有额外的电缆

+   内置电池备份

+   WiFi、蓝牙、扬声器、屏幕等

+   除去运输之外，环境影响可以忽略不计

+   我想在 ARM 平台上玩 Linux。

我不需要专用电缆来供电或连接它。在 WiFi 上运行时，它只需要插上我的手机充电器。每当我需要给手机充电时，我就拔下服务器，插上手机。它占用的空间非常小，我 *实际上把它留在了床头柜上*。

# 意外问题[](#unexpected-issues)

+   我无法使 Phosh 镜像启动，所以我改用了 Plasma Mobile 镜像。

+   几分钟内没有活动后，手机将会进入挂起状态。这在 SSH 连接中造成了很多问题，花了一段时间才解决，因为服务器通常不会进入挂起状态。

# Docker[](#docker)

解决了挂起问题后，docker 看起来工作正常。令人惊讶的是，许多镜像都支持 ARM！

# Tailscale[](#tailscale)

我安装了 [tailscale](https://tailscale.com)，以便远程访问服务器，而不用直接连接到互联网。Tailscale 真是太棒了，如果需要的话，我奶奶可能不到五分钟就能设置好。

# 作为普通手机的可行性[](#viability-as-a-regular-phone)

OnePlus 6T 比之前使用的 [PinePhone](https://pine64.org/devices/pinephone/) 快得多。但 Linux 还不适合日常使用：

+   每次重启都需要手动连接 WiFi

+   应用生态系统仍在持续发展中，缺少用于导航、相机、流媒体等的精细应用。

+   像 Firefox 这样的传统 Linux 应用无法正确缩放

希望这些问题将来能够解决，但目前，6T 是一台优秀的服务器。

我一直在玩几个容器，但还没有决定具体要运行哪些服务。我在考虑：

随着事情进展，我会发布更新。
