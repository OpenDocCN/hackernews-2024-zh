<!--yml

category: 未分类

date: 2024-05-27 13:02:01

-->

# KDE6 发布：D-Bus 和 Polkit 盛宴 | SUSE 安全团队博客

> 来源：[https://security.opensuse.org/2024/04/02/kde6-dbus-polkit.html](https://security.opensuse.org/2024/04/02/kde6-dbus-polkit.html)

# 目录

# 简介

SUSE 安全团队限制在 openSUSE 发行版和衍生的 SUSE 产品中安装系统范围的 D-Bus 服务和 Polkit 策略。在这些功能的包被添加到生产库之前，需要先由我们进行审查。

在十一月，openSUSE KDE 打包者向我们提出了一个[长长的 KDE 组件列表](https://bugzilla.suse.com/show_bug.cgi?id=1217076)，这些组件将用于即将到来的 KDE6 主要发布版。由于接口重命名或其他破坏性变更，这些包需要调整 D-Bus 和 Polkit 的白名单设置。一次性处理这么多组件是一次独特的经历，也带来了新的见解，这些见解将在本文中讨论。

对于对 D-Bus 和/或 Polkit 新手读者，以下部分提供了摘要，以更好地了解这些系统。

## D-Bus 概述

[D-Bus 消息总线系统](https://www.freedesktop.org/wiki/Software/dbus/)提供了一种在应用程序中实现远程过程调用的定义方式。在 Linux 上通常仅在本地使用，尽管 D-Bus 规范也允许通过网络操作。

D-Bus 服务是提供一个或多个接口的程序，客户端可以调用这些接口来获取信息、触发操作等。D-Bus 规范定义了可以传递给和从 D-Bus 方法调用返回的一组数据类型。

D-Bus 应用通过连接到一个共享总线来相互通信，其中有两种预定义类型：系统总线和会话总线。执行系统范围任务的服务连接到系统总线。这些服务通常以 `root` 或专用服务用户的身份运行。另一方面，会话总线为每个（图形）用户会话创建，只有以登录用户特权运行的应用程序可以连接到它。会话总线不涉及特殊权限。它的主要目的是为会话范围的服务提供定义的 API，例如桌面搜索引擎。

## Polkit 概述

[Polkit](https://www.freedesktop.org/software/polkit/docs/latest/polkit.8.html) 是一个授权框架，允许（特权）应用程序决定系统中的用户是否被允许执行特定操作。与普通的 `root` vs. 非 `root` 决策相比，这些操作允许更精细化的授权模型。例如，可以是启用系统中的蓝牙设备，或者挂载可移动存储设备的操作。

一个 Polkit 策略配置文件声明了某个应用领域使用的操作及其认证要求。当系统中的角色请求一个使用 Polkit 的应用执行操作时，该应用会向系统范围内的 Polkit 守护程序询问该角色是否有权限这样做。根据情况，例如，在用户的图形会话中可能会显示密码提示以授权操作。

Polkit 独立于 D-Bus，但两者结合使用是非常常见的模式。Polkit 的其他使用方式包括在 setuid-root 二进制文件中或通过类似 sudo 的 `pkexec` 实用程序。

## D-Bus 和 Polkit 的安全相关性

D-Bus 和 Polkit 的典型设置如下：一个系统守护程序以完全 root 权限运行，并在 D-Bus 系统总线上注册一个服务。登录到图形会话的非特权用户通过 D-Bus 请求守护程序执行某个活动。这触发 Polkit 认证过程，以确定调用者是否被允许这样做。

从安全角度来看，在这种场景中可能会出现很多问题。接下来的部分将调查可能出现的典型问题。

### 覆盖所有特权代码路径

系统守护程序实际上需要为其提供的每个敏感 D-Bus 方法正确实现 Polkit 授权检查。Polkit 不是什么神奇的东西，而是特权组件需要识别需要由其保护的所有代码路径。

一些应用故意提供未经认证和经过认证的混合 D-Bus 方法。在这些情况下，有时很难记住所有可能的副作用和结果，当忽略某些情况时可能导致安全问题。

### 代表非特权用户作为 root 执行操作

特权 D-Bus 服务组件经常需要代表非特权客户端执行操作。例如，可以是在调用者的主目录中挂载文件系统，或处理由调用者提供的文件。这是经典的权限边界交叉。这类服务的开发者通常不清楚可能出现的问题，特别是在以 `root` 访问用户控制的路径时。

类似地，如果一个特权的 D-Bus 服务将多个用户的数据存储在共享系统目录中，则可能会通过以过于开放的权限存储文件或混淆不同用户上下文而导致信息泄露。

### 集成 Polkit 可能很困难

Polkit 有其自己的命名约定和设计原则，需要深入了解才能完全理解它。除此之外，即使 Polkit 正确请求权限，特权服务仍然需要正确评估结果。在这个领域可能发生的典型错误是特权服务正确向 Polkit 请求认证，但结果被简单地忽略，特权操作继续进行。

### 所有人都可以访问 D-Bus 系统总线。

默认情况下，所有本地用户都可以访问 D-Bus 系统总线并与大多数特权服务通信。个别 D-Bus 服务配置文件可以限制允许调用 D-Bus 服务方法的用户范围。然而，这种设置是例外，因为大多数 D-Bus 服务对所有用户都是可访问的。

这显著增加了攻击面，因为不仅仅是运行授权本地会话的交互式用户帐户可以与这些服务通信，例如 `nobody` 用户帐户也可以。如今，许多在网络上运行的系统守护进程仅具有有限权限，甚至使用 systemd 提供的动态分配用户。如果可以利用具有低权限的这些网络守护进程中的一个，那么特权 D-Bus 系统服务中的弱点可能提供进一步提升权限的可能性。

这也是为什么 SUSE 安全团队作为深入防御策略的一部分，密切关注与网络没有直接连接的这些组件之一的原因。

# KDE KAuth 框架

KDE 桌面环境是 D-Bus 服务的重度用户，无论是在系统总线还是会话总线上。它在 D-Bus 和 Polkit 之上添加了进一步的抽象。这一切的基础组件是 [KAuth 框架](https://api.kde.org/frameworks/kauth/html/)。KAuth 生成 D-Bus 配置文件和一些粘合代码，将 D-Bus 和 Polkit 集成到 KDE 应用程序中。在 KAuth 中，以 `root` 身份运行的特权 D-Bus 服务被称为 KAuth 辅助程序。

我们对 KDE6 发布进行了[专门的跟进审查](https://bugzilla.suse.com/show_bug.cgi?id=1217178)。SUSE 安全团队的一位前成员在 2017 年发现了[一个重大的安全漏洞](https://bugzilla.suse.com/show_bug.cgi?id=1036244)。由于当时的审计是全面的，因此我们不再期望在核心授权逻辑中发现任何重大问题，事实上也确实如此。

## QVariantMap 序列化数据的问题使用有问题。

KAuth 的一个特殊之处在于，在 D-Bus 层级上仅传输基于 Qt 框架提供的 `QVariantMap` 数据类型的二进制 blob 对象，而不是本地 D-Bus 数据类型。

在审查过程中，我们注意到 KAuth 中此功能的实现[有些不稳定](https://invent.kde.org/frameworks/kauth/-/blob/4e94f01d3a191861c8095f673d70291dc225f23e/src/backends/dbus/DBusHelperProxy.cpp#L218)，因为在实际调用 D-Bus 函数回调之前，会处理潜在由攻击者控制的数据进行 Qt 数据类型反序列化。2019年，上游作者[已经识别出这个问题](https://invent.kde.org/frameworks/kauth/-/commit/fc70fb0161c1b9144d26389434d34dd135cd3f4a)，可能导致诸如图像数据被反序列化等副作用，实际上只期望字符串和整数。目前，KAuth 代码介入 Qt 框架的内部状态，以防止这类副作用发生。

## 关于生成的 D-Bus 附加配置片段的问题

仅在我们的审查工作后期，我们意识到 KDE6 版本的 KAuth 引入的变更导致[生成过于开放的 D-Bus 配置文件](https://bugzilla.suse.com/show_bug.cgi?id=1220215)。每个软件包的 D-Bus 配置片段安装在“/usr/share/dbus-1/system.d”。这些配置文件充当了 D-Bus 系统总线的一种防火墙配置。它们定义了谁被允许为特定接口注册 D-Bus 服务，以及谁被允许与之交流。

这里有一个从 systemd-network 的“org.freedesktop.network1.conf”中提取的适当示例：

```
<busconfig>
        <policy user="systemd-network">
                <allow own="org.freedesktop.network1"/>
        </policy>

        <policy context="default">
                <allow send_destination="org.freedesktop.network1"/>
                <allow receive_sender="org.freedesktop.network1"/>
        </policy>
</busconfig> 
```

这仅允许专用服务用户“systemd-network”注册“org.freedesktop.network1” D-Bus 接口，而系统中的其他用户可以与之交流。

KAuth KDE6 版本候选发布生成了这样的配置：

```
<policy context="default">
  <allow send_destination="*"/>
</policy> 
```

这可能会被轻易忽视：这表明每个人都被允许与 D-Bus 系统总线上的所有内容交流。它还影响了其他不应受到个别 KDE 软件包附带的 D-Bus 附加配置片段影响的 D-Bus 服务。尽管大多数在系统总线上运行的 D-Bus 服务是“公共”的，即每个人都可以与它们交流，但有些服务遵循不同的安全模型，只允许特定用户与服务交互。我们确定[ratbagd](https://github.com/libratbag/libratbag/blob/v0.17/ratbagd/org.freedesktop.ratbag1.conf) 是这样一种会受到 KAuth 缺陷负面影响的 D-Bus 服务。这显示了无关软件包的安全状况正在受到威胁。幸运的是，在 KDE6 发布完成之前我们及时发现了这个问题，并在它影响生产系统之前修复了该问题。我们还检查了在 openSUSE Tumbleweed 上安装的任何非 KDE 的 D-Bus 配置文件，以确保没有进一步包含此问题的文件。

这些副作用在某种意义上也是 D-Bus 配置方案的缺点，因为特定 D-Bus 服务的开发者不希望他们的配置文件具有全局影响。类似的问题也存在于 [logrotate](https://bugzilla.suse.com/show_bug.cgi?id=1180525)，其中“/etc/logrotate.d” 中的 drop-in 配置文件的设置可能会影响影响整个系统的全局设置。在这两种情况下，都可能导致难以发现的 bug，因为结果也取决于配置文件片段解析的顺序。

# Legacy fontinst D-Bus 服务

我们已被要求在 KDE6 发布中查看的大多数 KDE 组件，在最近几年已由我们进行了审核。但有些是传统的包，因为它们在我们引入 D-Bus 和 Polkit 的打包限制时已经存在。当时我们没有足够的资源一次性检查所有这些包。

在查看 KDE6 发布时，我们遇到了一个这样的传统组件 [（我们遇到的）](https://bugzilla.suse.com/show_bug.cgi?id=1217186) 是“org.kde.fontinst.service”，它是“plasma6-workspace”包的一部分。我们在那里发现了一个单独的 D-Bus 方法“org.kde.fontinst.manage”，实际上多路复用了一整套子方法，基于一个“method”字符串输入参数。这是不好的设计，因为它破坏了 D-Bus 协议，使得各个方法调用不够显著和不易管理。这一点还加强了只使用一个 Polkit 动作来验证所有子方法的事实。这样，对于隐藏在这个单一 D-Bus 方法调用背后的各个代码路径，只有全有或全无的设置。

该服务中可用的子方法几乎构成一个通用的文件系统 I/O 层，特别是当我们记得此服务以完全 root 权限运行时：

+   install: 可用于将任意文件路径复制到任意位置，新文件的权限为 0644。

+   uninstall: 允许删除任意文件路径，只要它们的父目录有写权限位设置。

+   move: 允许移动任意路径，并完全采用新的所有者 UID 和组 GID 到任意新位置。

+   toggle: 这个操作接受原始 XML，似乎也指定要启用或禁用的字体路径。

+   removeFile: 按标签上说的做；另一种删除文件的方式。

+   configure: 保存修改后的字体目录并调用一个小的 bash 脚本 `fontinst_x11`，准备字体目录并在 X 服务器上触发字体刷新。

字体安装服务的核心业务逻辑应该是管理系统提供的系统范围字体。为了实现这一目标，理想情况下应仅提供必要的高级逻辑操作，例如：从提供的数据安装字体，按名称删除系统字体。复制、删除和移动任意文件明显超出了此服务应执行的范围。

单个Polkit操作“org.kde.fontinst.manage”默认需要`auth_admin_keep`授权，即任何想要调用此方法的人都需要提供管理员凭据。然而，如果管理员决定降低这些要求，因为用户应该能够在系统中安装新字体，那么这个接口不仅允许这样做，还允许通过复制任意文件来获取完全的root权限（例如创建新的“/etc/shadow”文件）。

这项服务需要进行较大的重新设计。KDE上游未能及时为KDE6发布提供这样的解决方案。我们希望它仍然会发生，因为API处于相当令人担忧的状态。

## 对“意外”Polkit设置的困扰

关于`auth_admin`设置在fontinst服务中的情况，这是我们在审查D-Bus服务和Polkit操作时经常看到的一个常见模式。开发人员认为，要求Polkit操作使用`auth_admin`身份验证足以证明过于通用的API或者以`root`身份执行的不安全文件系统操作是合理的。在某些情况下，可以认为一个操作绝不应比`auth_admin`有更弱的身份验证要求，因为否则会导致无法控制的安全问题。不要忘记，Polkit是一个*可配置*的身份验证框架。应用程序会提供默认设置，但系统集成者和管理员可以更改这些要求。

我们知道，(open)SUSE Linux发行版是唯一提供管理员通过配置文件和覆盖来覆盖单个操作的Polkit默认设置的发行版。详细信息请参见[这里](https://en.opensuse.org/openSUSE:Security_Documentation#Configuration_of_Polkit_Settings)。这是通过[polkit-default-privs](https://github.com/openSUSE/polkit-default-privs)包实现的。我们的经验表明，上游开发人员大多既不考虑降低Polkit身份验证要求的安全后果，也不测试为加固目的提高身份验证要求时会发生什么。提高的身份验证要求会导致额外的密码提示，并且一些应用程序实现的工作流程涉及Polkit操作，在这些情况下会导致非常不幸的行为。

一个常见的例子是类似 Flatpak 的软件包管理器，它尝试在图形会话登录时获取 Polkit 认证以进行存储库刷新操作。开发者仅在默认的`yes` Polkit 认证需求下进行测试，这使得认证过程对用户来说是不可见的。当将其提升到`auth_admin`时，用户登录时突然会弹出密码提示，让用户感到困惑和恼火。有方法可以解决这个问题：例如，使用 Polkit 的服务可以询问是否可以在没有用户交互的情况下授权操作。如果不行，那么软件包管理器可以选择现在*不*刷新存储库。还可以使用“org.freedesktop.policykit.imply”注解来分组认证多个操作，以避免单个工作流程中出现多个密码提示。

可以理解的是，对许多不同 Polkit 配置的配置管理对上游开发者来说是难以测试的。增强对该领域一般问题的认识将有助于首次避免这些问题。看起来开发者只是想要在他们的软件中“塞进”认证，然后一旦完成就不再考虑它。当然，对于新手来说，Polkit 和 D-Bus 都不是简单的。尽管如此，每个认证过程都应该经过深思熟虑。对于实施 Polkit 的开发者来说，应该牢记以下要点：

+   Polkit 是一个*可配置*的认证框架，开发者设想的设置可能与运行时实际发生的情况并不相符。

+   当建模 Polkit 动作时，应充分利用细粒度的可能性，允许用户为各个活动进行精细调整要求。

+   在应用程序中发生的每个 Polkit 授权都应在两个方向上进行深思熟虑：如果降低认证要求会发生什么，如果提高认证要求会发生什么？

+   还未讨论的另一个方面是显示给用户的认证消息。它们应清楚地说明授权的具体内容，以便非技术用户能够理解。Polkit 还支持消息中的占位符，用于填充运行时信息，例如正在操作的文件。可惜，这个功能在实际应用中很少被使用。

# 在 sddm-kcm6 中的问题性文件系统操作

此组件是 SDDM 显示管理器的 KDE 配置模块（KCM）。它包含一个名为“org.kde.kcontrol.kcmsddm.conf”的 D-Bus 服务。我们已经在过去对其进行了审查，并且[再次进行了审查](https://bugzilla.suse.com/show_bug.cgi?id=1217188)以适应 KDE6 的发布。该服务存在两个主要问题，将在下文中讨论。

## 由不受特权 D-Bus 客户端提供的文件系统路径上的不安全操作

sddm-kcm6 KAuth 助手提供的多个 D-Bus 方法期望将文件系统路径作为输入参数。将路径传递给特权 D-Bus 服务是另一种常见的问题模式。在 [`openConfig()`](https://invent.kde.org/plasma/sddm-kcm/-/blob/Plasma/6.0/sddmauthhelper.cpp?ref_type=heads#L30) 函数中，如果需要，助手将创建一个[SDDM 主题配置文件](https://invent.kde.org/plasma/sddm-kcm/-/blob/Plasma/6.0/sddmauthhelper.cpp?ref_type=heads#L280)的路径。如果路径已存在，则会对路径进行`chmod()`操作，模式为`0600`，同时也会跟随符号链接。为了了解这种方法可能存在的问题，可以考虑一下如果将“/etc/shadow”作为主题配置路径传递会发生什么情况。

以 `root` 身份在受非特权用户控制的文件上操作，是非常难以正确处理的，并需要谨慎使用较低级别的系统调用。通常开发者甚至没有意识到这个问题。KDE 组件在过去在这个领域有过一些问题。我们认为这个问题有更深层次的根源，即在 Qt 框架的文件系统 API 设计中，一方面不能完全控制低级系统调用（因为 Qt 也是一个平台抽象层），另一方面在这方面也没有确切的文档说明可以期望其 API 的内容。此外，Qt 框架本身并不知道它是作为 root 运行的，可能操作其他用户拥有的文件。Qt 库设计用于实现功能丰富的 GUI 应用程序，并没有真正考虑处理不受信任的输入、以提升权限操作和跨权限边界操作。

避免路径访问问题的一个优雅方法是首先不要传递文件路径，而是通过 D-Bus 传递已打开的文件描述符。这是可能的，因为 D-Bus 内部使用 UNIX 域套接字，它们可以用来传递文件描述符。因此，客户端不是传递一个字符串给服务端，建议“打开这个文件，相信我，没问题”，而是传递一个文件描述符，使用自己的低特权打开，给特权服务。这样，许多路径访问问题一瞬间消失。然而，仍有一些需要注意的情况，例如如果需要执行递归文件系统操作。

不幸的是，KDE 使用的 KAuth 框架在这个领域显示出一些限制。由于 KAuth 助手的 D-Bus API 只能传输从序列化的 `QVariantMap` 得到的二进制 blob，目前无法传递打开的文件描述符。

## 更改由`sddm`服务用户拥有的配置文件

另一个问题并非出现在 D-Bus API 中，而是在 `sync()` 和 `reset()` D-Bus 方法的实现中。一旦来自客户端的任何输入参数被处理，辅助工具将在属于 `sddm` 服务用户的家目录中操作。以下是从 reset() 和 sync() 函数中提取的精简代码：

```
// from SddmAuthHelper::reset()
QString sddmHomeDirPath = KUser("sddm").homeDir();
QDir sddmConfigLocation(sddmHomeDirPath + QStringLiteral("/.config"));
QFile::remove(sddmConfigLocation.path() + QStringLiteral("/kdeglobals"));
QFile::remove(sddmConfigLocation.path() + QStringLiteral("/plasmarc"));
QDir(sddmHomeDirPath + "/.local/share/kscreen/").removeRecursively(); 
```

```
// from SddmAuthHelper::sync()
QString sddmHomeDirPath = KUser("sddm").homeDir();

QDir sddmCacheLocation(sddmHomeDirPath + QStringLiteral("/.cache"));
if (sddmCacheLocation.exists()) {
    sddmCacheLocation.removeRecursively();
}

QDir sddmConfigLocation(sddmHomeDirPath + QStringLiteral("/.config"));

if (!args[QStringLiteral("kscreen-config")].isNull()) {
    const QString destinationDir = sddmHomeDirPath + "/.local/share/kscreen/";
    QSet<QString> done;
    copyDirectoryRecursively(args[QStringLiteral("kscreen-config")].toString(), destinationDir, done);
} 
```

受损的 `sddm` 服务用户可以利用这些操作来获得其利益：

+   例如通过在目录中放置符号链接，可以导致 D-Bus 服务在完全不同的文件系统位置运行，从而造成拒绝服务。尽管如此，这种攻击是有限的，因为在删除调用中使用的最终路径组件需要匹配，如 `kscreen`。

+   通过在“~/.local/share/kscreen”中放置符号链接，它可以导致“kscreen-config”被复制到任意位置。

为了使这些操作变得安全，最好是暂时降低到 `sddm` 用户的权限。

## 从这里开始继续前进

KDE 上游在 KDE6 发布之前未能及时重新设计此 D-Bus 服务。在这种情况下，`sddm` 用户的家目录中的不安全操作甚至可以正式地证明需要分配 CVE。由于所有的 D-Bus 方法都受到 `auth_admin` Polkit 认证要求的保护，至少在默认安装中无法利用这些问题。

# KWalletManager：伪认证保护配置

KWalletManager 是 KDE 的密码管理器。它具有 GUI，并且如人们所期望的那样，在已登录用户的图形用户会话上下文中运行。它提供一个名为“org.kde.kcontrol.kcmkwallet5.save”的 D-Bus 方法的“savehelper”服务。那么一个以 `root` 身份运行的服务辅助程序在这里需要保存什么？让我们看看[实现](https://invent.kde.org/utilities/kwalletmanager/-/blob/master/src/konfigurator/savehelper.cpp)：

```
ActionReply SaveHelper::save(const QVariantMap &args)
{
    Q_UNUSED(args);
    const qint64 uid = QCoreApplication::applicationPid();
    qDebug() << "executing uid=" << uid;
    return ActionReply::SuccessReply();
} 
```

仔细研究这段代码的各个方面会让人意识到它实际上 *什么也没有做*。我们[向上游询问](https://invent.kde.org/utilities/kwalletmanager/-/issues/4)，希望移除这个未使用的辅助工具，但被告知这不是一个错误，而是故意的。他们想要防止以下攻击场景发生：用户离开电脑且未锁定，一个陌生人经过并试图更改 KWalletManager 的设置。为了防止这种情况发生，GUI 会要求服务辅助进行身份验证，需要 Polkit 的 `auth_self` 授权，如果验证失败则不继续进行。

然而，这无法阻止真正的攻击者，因为 KWalletManager 的配置存储在非特权用户的主目录中，仍然可以直接编辑，或者使用一个不要求此身份验证的修改版本的 KWalletManager。更不用说在这种情况下攻击者可能做的*所有其他事情*。那么应该在哪里划清界限才能停止？我们甚至认为这不是加固，而是虚假的安全和混淆。如果确实需要这种虚假的认证，那么至少应该找到一种方法来实现它，而不需要作为 `root` 运行且无所作为的认证助手。虽然上游似乎不同意，但我们要求我们的打包者通过补丁从我们的打包中删除此逻辑。

# DrKonqi 的改进

DrKonqi 是 KDE 的崩溃处理实用程序。最近，它与 systemd-coredump 交互，以访问应用程序的核心转储。我们之前对其的[2022年回顾](https://bugzilla.suse.com/show_bug.cgi?id=1203493)导致了对[systemd-coredump 本身的发现](https://www.openwall.com/lists/oss-security/2022/12/21/3)。与此同时，DrKonqi 获取了额外的 D-Bus 服务逻辑，用于将私有核心转储（例如来自以 `root` 用户身份运行的进程）复制到非特权用户的会话中进行分析。

对于一个 KDE 组件来说，这种[实现方式](https://invent.kde.org/plasma/drkonqi/-/blob/Plasma/6.0/src/coredump/polkit/main.cpp?ref_type=heads#L34)是不寻常的，因为它不依赖于 KAuth：它直接使用 Qt 框架的 D-Bus 和 Polkit 设施。这样做的可能原因是 KAuth 在传递文件描述符方面的不足，正如上文所讨论的。单个的 `excavateFromToDirFd()` D-Bus 方法实际上接受一个文件描述符。这应该是指一个由非特权调用者控制的目录的文件描述符，用于将所选的核心转储复制到其中。即使这意味着 DrKonqi 无法从 KAuth 的常见框架特性中受益，从安全角度来看，这是一个改进以 `root` 运行并在文件系统中操作的 D-Bus 服务鲜为人知的好例子。

不幸的是，即使使用文件描述符也可能会出现问题，正如这个例子所显示的。对于目录的权限处理与常规文件不同。通常只能以只读模式 (`O_RDONLY`) 打开目录。写权限仅在尝试写入时才会检查，比如在 DrKonqi 助手调用 `renameat()` 时。这太晚了。非特权调用者可以打开任何具有读权限的目录，并将其传递给 D-Bus 服务。即使调用者对其没有写权限，以 `root` 运行的 D-Bus 服务现在也会愉快地在目录中创建新文件。

目前正在与上游进行一场[建设性讨论](https://bugzilla.suse.com/show_bug.cgi?id=1220190#c6)，导致此 D-Bus 服务中各种详细改进[即将合并](https://invent.kde.org/plasma/drkonqi/-/merge_requests/217)。关于 dir 文件描述符的问题是在过程的后期才发现的，但希望很快能找到解决方案。

# 结论

D-Bus 和 Polkit 都有其需要理解和良好管理的复杂性。即使超越 Linux 系统的本地安全性，这对于深度防御措施至关重要。像 KAuth 框架这样在其上叠加额外的层次可能会导致长期问题，正如当前 KAuth API 不支持传递文件描述符的情况所示。

我们的 KDE 打包人员和上游早早与我们接触 KDE6 发布候选版本及其引入的更改是有帮助的。在一些领域，如糟糕生成的 D-Bus KAuth 配置文件，上游迅速做出反应并应用修复，从而避免问题代码被释放到 KDE6 生产版本中。在其他领域，如遗留的 fontinst D-Bus 服务或 sddm-kcm D-Bus 服务，修复 API 问题的复杂性显然对上游来说过高，无法及时提出更好的解决方案。对于这些服务中发现的问题，我们决定不要求 CVE 分配，因为在默认的 Polkit 配置中，攻击向量对普通用户是不可达的。

到目前为止，大多数 KDE6 软件包应该已经达到 openSUSE Tumbleweed，并可用于生产环境。

# 参考文献

# 变更历史
