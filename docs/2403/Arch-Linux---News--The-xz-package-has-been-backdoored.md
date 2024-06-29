<!--yml

类别：未分类

日期：2024-05-29 12:46:01

-->

# Arch Linux - 新闻：xz 软件包已被植入后门

> 来源：[https://archlinux.org/news/the-xz-package-has-been-backdoored/](https://archlinux.org/news/the-xz-package-has-been-backdoored/)

**更新：** 据我们所知，通过发布 tarball 分发的恶意代码从未进入 Arch Linux 提供的二进制文件，因为构建脚本被配置为仅在基于 Debian/Fedora 的包构建环境中注入恶意代码。因此，下面的新闻项大部分可以忽略。

我们正在密切监视情况，并将根据需要更新包和新闻。

* * *

简短来说：立即升级您的系统和容器镜像 **现在**！

正如许多人可能已经阅读的那样（[one](https://www.openwall.com/lists/oss-security/2024/03/29/4)），版本 `5.6.0` 和 `5.6.1` 中的上游发布 tarballs 包含了添加后门的恶意代码。

此漏洞在 Arch Linux 安全跟踪器中跟踪（[two](https://security.archlinux.org/ASA-202403-1)）。

在版本 `5.6.1-2` 之前的 `xz` 包（具体是 `5.6.0-1` 和 `5.6.1-1`）包含此后门。

以下发布物件包含了受损的 `xz`：

+   安装媒介 `2024.03.01`

+   虚拟机镜像 `20240301.218094` 和 `20240315.221711`

+   创建于 *2024-02-24* 和 *2024-03-28* 之间的容器镜像

受影响的发布物件已从我们的镜像站点中移除。

我们强烈建议不要使用受影响的发布物件，而是下载目前可用的最新版本！

## 升级系统

如果您的系统当前安装了版本为 `5.6.0-1` 或 `5.6.1-1` 的 `xz`，强烈建议立即进行完整的系统升级：

`pacman -Syu`

## 升级容器镜像

要确定是否使用了受影响的容器镜像，请使用以下其中之一

`podman image history archlinux/archlinux`

或者

`docker image history archlinux/archlinux`

取决于您使用 `podman` 还是 `docker`。

任何早于 `2024-02-24` 和晚于 `2024-03-29` 的 Arch Linux 容器镜像都受到影响。

运行以下之一

`podman image pull archlinux/archlinux`

或

`docker image pull archlinux/archlinux`

将受影响的容器镜像升级到最新版本。

之后，请确保重新构建基于受影响版本的任何容器镜像，并检查任何正在运行的容器！

## 关于 sshd 认证绕过/代码执行

来自上游报告（[one](https://www.openwall.com/lists/oss-security/2024/03/29/4)）：

> openssh 并不直接使用 liblzma。但是 Debian 和其他几个发行版对 openssh 进行了补丁以支持 systemd 通知，而 libsystemd 确实依赖于 lzma。

Arch 不会直接将 openssh 链接到 liblzma，因此此攻击向量是不可能的。您可以通过执行以下命令来确认这一点：

`ldd "$(command -v sshd)"`

然而，出于谨慎起见，我们建议用户通过升级的方式从系统中移除恶意代码。这是因为可能存在其他尚未发现的利用后门的方法。
