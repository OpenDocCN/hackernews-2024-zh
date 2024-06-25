<!--yml

类别：未分类

日期：2024-05-27 14:28:41

-->

# Solene'%：OpenBSD 工作站加固

> 来源：[https://dataswamp.org/~solene/2023-12-31-hardened-openbsd-workstation.html](https://dataswamp.org/~solene/2023-12-31-hardened-openbsd-workstation.html)

# 1\. 介绍 [§](#_Introduction)

我想分享一些您可以在 OpenBSD 工作站上进行的加固，并解释每个更改的威胁模型。

[OpenBSD 官方项目网站](https://www.openbsd.org)

随意选择您发现对您的用例有用的任何调整，对于大多数人来说，许多调整肯定是过度的，但根据上下文，这些更改对其他人可能是有意义的。

# 2\. 用户配置 [§](#_User_configuration)

在用户配置中可以做一些调整以提高安全性。

## 2.1\. 最小权限 [§](#_The_Least_privileges)

为了防止程序提升权限，请将自己从 wheel 组中移除，并且不要设置任何 doas 或 sudo 权限。

如果需要 root 权限，请使用 root 用户切换到 TTY。

## 2.2\. 多因素身份验证 [§](#_Multiple-factor_authentication)

在某些情况下，希望使用多因素身份验证，这意味着为了登录您的系统，您需要除了常规密码之外的 TOTP 生成器（通常是手机应用，或者密码管理器，如 KeePassXC）。

这将防止附近的人可能猜测您的系统密码。

我已经写了一篇指南，解释了如何将 TOTP 添加到 OpenBSD 登录中。

[博客文章：OpenBSD 上的多因素身份验证](https://dataswamp.org/~solene/2021-02-06-openbsd-2fa.html)

## 2.3\. 主目录权限 [§](#_Home_directory_permission)

用户目录的权限应该是 700，因此只有所有者和 root 可以浏览它。

理想情况下，您应该将 `umask 077` 添加到您的用户环境中，这样每个新目录或文件的权限都将限制为仅限于您的用户。

# 3\. 防火墙 [§](#_Firewall)

有一些有趣的策略可以通过 OpenBSD 防火墙 Packet Filter 进行配置。

## 3.1\. 阻止入站 [§](#_Block_inbound)

默认情况下，禁用除了对已建立会话的响应之外的所有传入流量是一个良好的做法（这样服务器就可以回复您的请求）。 这样可以防止您本地网络/ VPN 中的某人访问正在监听网络接口上的网络服务。

在 `/etc/pf.conf` 中，您将不得不替换默认值：

```
block return
pass 
```

通过以下方式：

```
block all
pass out inet
# allow ICMP because it's useful
pass in proto icmp 
```

然后，如果您需要允许网络上的某个端口，请使用 `pfctl -f /etc/pf.conf` 重新加载，并在文件中添加相应的规则。

## 3.2\. 过滤出站 [§](#_Filter_outbound)

阻止出站流量可能是有用且有效的，但只有当您确切地知道您需要什么时，因为您将需要手动允许主机和远程端口。

它将防止程序尝试使用非允许的端口/主机外泄数据。

# 为桌面用户禁用网络[§](#_Disabling_network_for_the_desktop_user)

禁用网络是一项重要的缓解措施，我认为。这将防止任何程序试图行事的恶意行为，如果它们无法确定是否存在代理，它们将无法连接到互联网。

这也可以使您免受误操作的命令，这些命令会从网络拉取诸如pip、npm等内容。我认为能够严格控制哪些程序应该进行网络连接，哪些程序不应该进行网络连接是非常重要的。在Linux上，这实际上很容易实现，但在OpenBSD上，我们无法限制单个程序，因此代理是唯一的解决方案。

可以通过创建一个名为`_proxy`（或者您喜欢的其他名称）的新用户来实现。使用`useradd -s /sbin/nologin -m _proxy`命令并将您的SSH密钥添加到其authorized_keys文件中。

在文件`/etc/pf.conf`的末尾添加此规则，然后使用`pfctl -f /etc/pf.conf`重新加载：

```
block return out proto {tcp udp} user solene 
```

现在，如果您希望允许某个程序使用网络，您需要：

+   使用命令`ssh -N -D 10000 _proxy@localhost`将代理切换为打开状态，这仅在您的SSH私钥已解锁时才可能。

+   在程序中配置一个SOCKS5代理

### 一些网络修复[§](#_Some_network_fixes)

大多数程序会对命名为`http_proxy`、`https_proxy`或`all_proxy`的变量中配置的代理做出反应，但是为您的用户全局定义这些变量并不是一个好主意，因为程序更容易自动使用代理，这与这个代理的本质相违背。

#### SSH[§](#_SSH)

默认情况下，除了本地用户外，您将无法ssh到任何地方，我们需要通过本地_proxy用户代理每个远程ssh连接。

在`~/.ssh/config`中：

```
Host localhost
User _proxy
ControlMaster auto
ControlPath ~/.ssh/%h%p%r.sock
ControlPersist 60

Host *.*
ProxyJump localhost 
```

#### Chromium[§](#_Chromium)

如果您没有配置GNOME代理设置，Chromium / Ungoogled Chromium将不会使用代理，除非您添加命令行参数`--proxy-server=socks5://localhost:10000`。

我尝试手动修改dconf数据库，其中包含“GNOME”设置以配置代理，但我没有成功（以前我成功过，但现在我无法成功）。

#### 同步[§](#_Syncthing)

如果您使用syncthing，您需要将其所有流量通过SSH隧道代理。这通过在程序环境中设置环境变量`all_proxy=socks5://localhost:10000`来完成。

# 居住在临时文件系统中[§](#_Live_in_a_temporary_file-system)

您可以将大部分家目录设置为驻留在内存中的临时文件系统，只有少数目录具有持久性。

此更改将阻止任何人使用前一会话遗留的临时文件或缓存。

实现这一点最有效的方法是使用我为此用例编写的程序home-impermanence，它处理应该持久存在的文件/目录列表。

[博客文章：在OpenBSD上使用impermanence实现可重复的干净$HOME目录](https://dataswamp.org/~solene/2022-03-15-openbsd-impermanence.html)

如果您只想使用一个模板来开始（在使用过程中不会发生变化），您可以检查`mount_mfs`的标志`-P`，它允许使用现有目录填充基于新鲜内存的文件系统。

[OpenBSD man页面：mount_mfs(8)](https://man.openbsd.org/mount_mfs)

# 6\. 禁用网络摄像头和麦克风 [§](#_Disable_webcam_and_microphone)

好消息！我在这里抓住机会提醒您，OpenBSD默认禁用各种可用设备的视频和音频录制，而是它们将表现出正常工作的样子，但录制的是空的数据流。

当您需要使用它们时，可以通过更改sysctls `kern.audio.record` 或 `kern.video.record` 为1来手动启用它们。

一些笔记本电脑制造商提供了一个物理开关选项，可以禁用麦克风和网络摄像头，这样您就可以放心它们的状态（Framework）。一些其他制造商还允许不安装任何网络摄像头和麦克风（NovaCustom，Nitropad）。最后，像Coreboot这样的开源固件可以提供一个BIOS设置来禁用这些外围设备，我认为这是可信的。

# 7\. 禁用USB端口 [§](#_Disabling_USB_ports)

如果您需要保护系统免受恶意USB设备的攻击（通常是在办公环境中），则应尽可能在BIOS/固件中禁用它们。

如果不可能，那么您仍然可以在启动时使用此方法禁用内核驱动程序。

创建文件`/etc/bsd.re-config`并添加以下内容：

```
disable usb
disable xhci 
```

这将禁用对USB 3和2控制器的支持。在台式计算机上，在这些情况下，您可能想使用PS/2外设。

# 8\. 系统范围的服务 [§](#_System-wide_services)

## 8.1\. Clamav防病毒软件 [§](#_Clamav_antivirus)

虽然这可能会让你笑一笑，但如果有机会能够救你一次，我认为这仍然是任何加固措施中的一项有价值的补充。来自电子邮件的下载附件，或者恶意的JPG文件仍然可能损害您的系统。

OpenBSD提供了一个完全可用的clamav服务，请不要忘记启用freshclam，病毒数据库更新程序。

## 8.2\. 自动更新 [§](#_Auto-update)

我已经在之前的一篇关于anacron的文章中讨论过这个问题，但我认为，在计算机上每天自动更新软件包和基本系统是到处都应该做的最低限度。

[Anacron：有用的OpenBSD示例](https://dataswamp.org/~solene/2023-06-28-anacron.html#_Useful_examples)

# 9\. 系统配置 [§](#_System_configuration)

## 9.1\. 内存分配加固 [§](#_Memory_allocation_hardening)

OpenBSD malloc系统允许您启用一些额外的检查，例如使用之后释放，堆溢出或者保护页，它们都可以一次性全部启用。这对于安全性非常高效，因为大多数安全漏洞都依赖于内存管理问题，但它可能会破坏具有内存管理问题的软件（这样的软件有很多）。使用这种模式也会对性能产生负面影响，因为系统需要对每个分配的内存块进行更多的检查。

为了启用它，请添加以下内容到 `/etc/sysctl.conf`：

```
vm.malloc_conf=S 
```

可以立即通过 `sysctl vm.malloc_conf=S` 启用，通过设置无值 `sysctl vm.malloc_conf=""` 来禁用。

程序 `ssh` 和 `sshd` 始终使用此标志运行，即使系统范围内已禁用。

# 10\. 更深入的一些想法 [§](#_Some_ideas_to_go_further)

## 10.1\. 专用代理 [§](#_Specialized_proxies)

可能存在不同的代理用户，每个用户限制到允许的远程端口，我们可以想象像这样的代理：

+   http / https / ftp

+   仅限 ssh

+   imap / smtp

+   等等....

当然，这比多用途代理更加麻烦，但至少，程序很难猜出要使用哪个代理，尤其是如果您不同时连接它们。

## 10.2\. 使用专用用户运行进程 [§](#_Run_process_using_dedicated_users)

我以前写过一点关于这个问题，对于命令行程序来说，在 SSH 上以专用本地用户运行它们是有道理的，只要这仍然实际可行。

[使用专用用户运行进程](https://dataswamp.org/~solene/2019-11-12-dedicated-users-processes.html)

但是如果需要运行图形程序，则变得棘手。使用 `ssh -Y` 让远程程序完全访问您的显示服务器，该服务器可以访问正在运行的所有其他内容，这并不理想... 您仍然可以依赖 `ssh -X`，它启用了 X11 安全扩展，但必须信任其实现，并且存在问题，如没有共享剪贴板、性能不佳以及尝试访问由安全协议阻止的合法资源时程序崩溃...

在我看来，实现图形程序隔离的最佳方法是在本地用户中运行专用的 VNC 服务器，并从您自己的用户连接。这比在本地运行 X 要好。

## 10.3\. 带 USB 解锁的加密家目录 [§](#_Encrypted_home_with_USB_unlocking)

在计算机被多人使用的设置中，系统加密可能很麻烦，因为每个人都必须记住主密码短语，你无法保证没有人会把它写在便签上... 在这种情况下，为每个用户创建一个个人加密卷可能更好。

我还没有实现，但我有一个好主意。为用户添加一个卷看起来像这样：

+   为此用户获取一个专用的 USB 存储器，它将用作解锁其数据目录的“密钥”

+   使用随机数据覆盖存储器

+   在系统上创建一个空磁盘文件，它将包含加密虚拟磁盘，使用 USB 磁盘的随机部分作为密码短语（您将必须记下长度 + 偏移量）

+   编写一个 `rc` 文件，如果检测到 USB 磁盘卷，则尝试在启动时解锁并挂载分区

这样，当系统启动时，您只需要将USB存储设备插入，它应该会自动解锁和挂载您的个人加密卷。请注意，如果您想切换用户，如果您不想用命令行操作，您将不得不重新启动以解锁他们的驱动器。

# 11\. 结论 [§](#_Conclusion)

在系统中增加安全性是完全可能的，但是实际世界中安全性和可用性之间的平衡应该始终被研究。

如果一个系统过于硬化，没有人会高效地使用它；另一方面，用户希望他们的系统能保护他们免受大多数常见威胁的侵害。

根据个人的环境和威胁模型，重要的是相应地配置他们的系统。