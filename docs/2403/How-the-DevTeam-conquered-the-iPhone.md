<!--yml

category: 未分类

date: 2024-05-27 14:50:54

-->

# 开发团队如何征服iPhone

> 来源：[https://fabiensanglard.net/iSummer/](https://fabiensanglard.net/iSummer/)

2024年1月31日

开发团队如何征服iPhone

* * *

2007年的夏天对手机爱好者来说是一个多事之秋。一切始于6月29日，当时苹果公司宣布推出iPhone。如果你从未花时间观看完整的发布会，我强烈建议你去看一看。

<MacWorld%20keynot%202007.webm>

您的浏览器不支持视频标签。

对这一宣布的反应褒贬不一。一些人对这款设备不屑一顾，而另一些人则渴望尝试。

我属于第二类别。然而，我住在加拿大，这里并没有看到iPhone的发布日期。事实上，iPhone直到一年后的2008年7月11日才与Rogers达成协议推出3G[^([2])](#footnote_2)。

iPhone 开发团队

* * *

对于被遗忘的国家而言，出现了希望。一群人聚集在一起，目标是通过软件使设备能够在任何运营商下运行。他们称自己为“iPhone 开发团队”[^([3])](#footnote_3)。

通过**iphone.fiveforty.net**的博客承诺开放性。他们定期报告他们的进展。在高峰期，每小时更新一次（7月3日从凌晨12点到下午9点[^([4])](#footnote_4)共有八次更新）。

毫无疑问，在2007年的夏天，我的网络浏览器总是关注他们的网站。然而，我并没有技术能力完全理解他们在做什么。参与者们技术娴熟，术语频繁，我从未敢提出问题。

对我来说，无法理解这样一个历史性时刻的技术细节总是让我感到困扰。今天，我决定借助Wayback Machine回到过去，看个究竟。也许你会喜欢和我一起来体验。

iphone.fiveforty.net状态栏

* * *

为了跟踪进展，**iphone.fiveforty.net**在其主页上设有一个状态栏。它展示了DevTeam为实现目标而确定的所有里程碑。这些成就分为三个级别，从红色到绿色不等。

我记不清是否完全显示过红色，我也不确定它是否曾经如此。Wayback机器最早抓取的日期是[2007年7月6日](https://web.archive.org/web/20070706064445/http://iphone.fiveforty.net/wiki/index.php?title=Main_Page)，当时完成了六个里程碑中的两个。

| 突破DMG密码 | 绕过激活 | 获得写入权限 | 获得工作工具链 | 解锁手机 | 启用第三方应用 |
| --- | --- | --- | --- | --- | --- |

这段旅程完成于9月12日，然而Wayback机器“仅”在13天后（[2007年9月25日](https://web.archive.org/web/20070925035357/http://iphone.fiveforty.net:80/wiki/index.php?title=Main_Page)）才抓取到。

| 解密固件 | 绕过激活 | 获得写入权限 | 获得工作工具链 | 启用第三方应用 | 解锁手机 |
| --- | --- | --- | --- | --- | --- |

我们将按顺序研究每个阶段。

预期的方式

* * *

在深入每个里程碑之前，让我们提醒自己iPhone预计如何使用。

1.  可以从苹果商店购买设备，价格为$499（4GB存储版本）和$599（8GB版本）[^([5])](#footnote_5)。

1.  解封揭示了一个[无法使用的](activate.jpeg) iPhone 展示了一个[连接到iTunes](itunes.jpg)的屏幕。

1.  客户不得不打开iTunes并订阅（[视频](https://www.youtube.com/watch?v=SAhRFE4D6W0)，[文章](https://archive.is/UjS8d)）AT&T的[会员服务](activationtips.pdf)。

1.  在此之后，手机激活。它仍然绑定（锁定）在AT&T网络，但客户可以正常使用。

里程碑

* * *

早期由DevTeam定义的六个里程碑沿着一个逻辑路径，将一个惰性的电子设备变回智能手机。

+   获得读取权限以理解系统 -> **破解DMG密码**。

+   使设备摆脱其惰性 -> **绕过激活**。

+   获得写入权限以修改系统 -> **获得写入权限**。

+   修改系统以使用自定义可执行文件 -> **获取工作工具链**。

+   做一些事情让基带连接到任何运营商 -> **解锁**。

+   创建一个应用程序来自动化整个过程 -> **启用第三方应用程序**

解密里程碑

* * *

为了将设备恢复到出厂状态，iTunes下载了一个**iPhone软件**（扩展名`.ipsw`）存档。这一步的目标是理解其中的每一个文件。方便地，存档仍然可以在ipsw.me[^([6])](#footnote_6)找到，并且它使用zip格式，因此我们可以查看它。

```
% unzip -l iPhone1,1_1.0_1A543a_Restore.ipsw
Archive:  iPhone1,1_1.0_1A543a_Restore.ipsw
  Length      Date    Time    Name
---------  ---------- -----   ----
 15725706  06-26-2007 18:41   694-5259-38.dmg
 86257664  06-04-2007 19:32   694-5262-39.dmg
        0  05-22-2007 22:25   Firmware/all_flash/
    14474  05-22-2007 22:21   Firmware/all_flash/applelogo.img2
    73866  05-22-2007 22:21   Firmware/all_flash/batterycharging.img2
    65674  05-22-2007 22:21   Firmware/all_flash/batterylow0.img2
    73866  05-22-2007 22:21   Firmware/all_flash/batterylow1.img2
    43146  05-22-2007 22:21   Firmware/all_flash/DeviceTree.m68ap.img2
   145546  05-22-2007 22:21   Firmware/all_flash/iBoot.m68ap.RELEASE.img2
    51338  05-22-2007 22:21   Firmware/all_flash/LLB.m68ap.RELEASE.img2
      175  05-22-2007 22:25   Firmware/all_flash/manifest
    34954  05-22-2007 22:21   Firmware/all_flash/needservice.img2
    26762  05-22-2007 22:21   Firmware/all_flash/recoverymode.img2
   103562  05-22-2007 22:25   Firmware/dfu/iBSS.m68ap.RELEASE.dfu
     9354  05-22-2007 22:25   Firmware/dfu/WTF.s5l8900xall.RELEASE.dfu
  3073247  06-19-2007 13:51   kernelcache.restore.release.s5l8900xrb
     1594  06-26-2007 18:41   Restore.plist
---------                     -------
105700928                     20 files // All iOS is a mere 105 MiB!!

```

那里有许多文件，它们的目的在很早之前就被确定了[^([7])](#footnote_7)[^([8])](#footnote_8)[^([9])](#footnote_9)。其中我们发现在恢复过程中显示在屏幕上的`img2`图像、包含基带的所有内容的`Firmware`文件夹以及iOS内核(`kernelcache`)。

更重要的是，我们看到两个巨大的`dmg`存档。第一个`694-5259-38.dmg`（称为ramdisk[^([10])](#footnote_10)）只有在iTunes恢复手机时使用。令人惊讶的是，它并没有加密。通过简单的`dd`命令可以挂载它。

```
$ dd if=694-5259-38.dmg of=ramdisk.dmg bs=512 skip=4 conv=sync
$ hdiutil attach ramdisk.dmg
/dev/disk4                                            /Volumes/ramdisk
$ find /Volumes/ramdisk > ramdisk.txt

```

这并不是iOS文件系统（仅有[100条目](ramdisk.txt)），但它允许DevTeam探查`/private/etc/master.passwd`，找到用户`mobile`（运行应用程序）和用户`root`（运行所有其他进程）的密码。

更重要的是，ramdisk的内容使得访问第二个dmg文件`694-5262-39.dmg`变得可能，该文件包含正常操作期间使用的iOS文件系统。尽管存档被加密，但通过查看ramdisk中的`/usr/sbin/asr`找到了其密钥。

请注意，他们找到了一个密钥，而不是密码。他们无法使用 `hdiutil`，必须编写自己的解密器 `vfdecrypt.c`[^([11])](#footnote_11)。解密后，此映像可以被挂载，他们获得了对完整运行时文件系统的读访问权限（文件列表[在此](https://web.archive.org/web/20071005010951/http://iphone.fiveforty.net/wiki/index.php/SystemFileAndDirectoryList#System.2FLibrary.2FExtensions.2F_Source_Files)）。

| 突破 DMG 密码 | 绕过激活 | 获取写入访问权限 | 获取工作工具链 | 解锁手机 | 启用第三方应用程序 |
| --- | --- | --- | --- | --- | --- |

激活里程碑

* * *

开箱即用的 iPhone 是一块砖。它尚未激活。用户订阅 AT&T 后，发生了以下事件[^([12])](#footnote_12)。

1.  iTunes 将收集设备的设备 ID、IMEI 和 ICCID。

1.  这三个字段将被连接成一个令牌。

1.  令牌将被发送到 Apple 服务器（`albert.apple.com`），在那里将使用 Apple 的私钥进行签名。

1.  签名的令牌随后将被发送回设备。

1.  一个名为 [`lockdownd`](https://iphonedev.wiki/Lockdownd) 的守护程序通过 USB 监听并使用 Apple 的公钥验证令牌。

1.  随着证明令牌来自于 Apple，并匹配设备 ID、IMEI 和 ICCID，`lockdownd` 将设备状态更新为“已激活”。

1.  用户随后可以访问 iPhone [主屏幕](Apple-iPhone.webp) 和应用程序。

```
┌─────────┐
│ iPhone  │
├─────────┤              ┌──────┐      ┌────────────────┐
│lockdownd│              │iTunes│      │albert.apple.com│
└────┬────┘              └──┬───┘      └───────┬────────┘
     │                      │                  │
     │   1.dID,IMEI,ICCID   │                  │
     ├─────────────────────►│                  │
     │                      │   2.Send token   │
     │                      ├─────────────────►│
     │                      │                  │
     │                      │   3\. f(token)    │
     │                      │◄─────────────────┤
     │                      │                  │
     ├──────────────────────┤                  │
     │ 4.f(iD, IMEI, ICCID) │                  │
     │         ==           │                  │
     │      f(token) ?      │                  │
     │                      │                  │
     ▼                      ▼                  ▼
 ACTIVATED!

```

至于激活的第一次突破来自臭名昭著的开发者 dvdjon，当他发布了 PhoneActivationServer[^([13])](#footnote_13)。

> dvdjon 通过修改 iTunes，使其使用 HTTP 而不是 HTTPS 连接激活服务器，并重定向激活请求到 PhoneActivationServer，创造了一个激活方法。
> 
> PhoneActivationServer 随后会向 iPhone 发送一个有效的账户令牌。然而，账户令牌对应的却是不同的 IMEI、ICCID 和设备 ID。这种方法导致手机处于不匹配 ICCID 的状态，但允许访问用户界面。
> 
> - iPhone Elite Dev Team
> 
> [^([14])](#footnote_14)

解释的关键部分是“有效令牌”。如果没有私钥，您如何生成一个呢？事实证明您根本不需要。PhoneActivationServer 总是返回相同的已签名令牌，该令牌来自成功激活（dvdjon 的手机？），无论输入的令牌是什么。这是一个简单的重播技巧。

George Hotz 的演示“Hacking the iPhone”进一步概述了这一点。

> 将激活记录重新发送到另一部手机
> 
> lockdownd 在响应中并未检查（iD、IMEI、ICCID）是否实际匹配任何内容。

DevTeam 编写了一个名为 `tools`[^([16])](#footnote_16) 的命令行工具，从 plist 文件加载硬编码的已签名令牌，并通过 iTunesMobileDevice.dll 函数将其发送到任何 iPhone（后来通过独立的 `iPhoneInterface` 工具改进，无需 iTunes）。如果您想自己查看，该工具附带了 [源代码](https://att.newsmth.net/nForum/#!article/Apple/178321)。

| 打破 DMG 密码 | 绕过激活 | 获得写入权限 | 获取工作工具链 | 解锁手机 | 启用第三方应用程序 |
| --- | --- | --- | --- | --- | --- |

写入权限里程碑

* * *

激活的手机将出现在 iTunes GUI 中，用户可以上传文件，如音乐和照片。因此存在“某些”写入权限。但处理文件上传的进程（`acfd`）被 `chroot` 禁闭在 `/root/Media`[^([17])](#footnote_17)。此外，只有用户分区被挂载为“rw”。系统分区仅挂载为“r”。因此，双重目标是打破 `chroot` 禁锢（这就是“越狱”一词的由来）并在系统分区中某种方式写入。

```

                ┌──────┐
                │iPhone│
┌──────┐        ├──────┤
│iTunes│        │ acfd │
└──┬───┘        └──┬───┘
   │               │
   │  write(file)  │
   ├──────────────►│
   │               │
   │               │
   │             chroot
   │               │
   │               ▼
   ▼               /root/Media

```

该团队似乎在2007年7月8日左右找到了一个解决方案[^([18])](#footnote_18)，并描述了一个涉及 ramdisk[^([19])](#footnote_19) 的过程。

要理解它的工作原理，我们需要了解 iPhone 启动的两种方式。手机启动时运行的第一条指令来自 BootROM。从那里开始，加载程序引导更复杂的阶段。请注意，在启动新阶段之前，会检查其签名，因此只有苹果签名的内容才能运行。这个过程建立了一条从 Darwin 内核到运行应用程序的信任链。

```
 Normal operation boot chain:

┌───────┐     ┌───┐      ┌─────┐      ┌──────┐      ┌───────────┐
│BootROM├────►│LLB├─────►│iBoot├─────►│Kernel├─────►│Normal Mode│
└───────┘     └───┘      └─────┘      └──────┘      └───────────┘

```

有第二种启动模式，允许 iTunes 将手机从糟糕的状态恢复到良好状态。当手机进入恢复模式时，它会停留在 iBoot 阶段[^([20])](#footnote_20)。从那里开始，手机期望从内存中加载下一个阶段（现在我们理解了名为“ramdisk”的 dmg 存档，它是一个用于上传到内存的磁盘）。

```
 Recovery mode boot chain:
                                    ┌──────────────────────────────────┐
                                    │              iTunes              │
                                    └───┬───────────┬───────────┬──────┘
                                        │           │           │ push files to
                                        ▼           ▼           ▼ restore iOS
┌───────┐     ┌───┐      ┌─────┐    ┌───────┐   ┌──────┐    ┌───────────┐
│BootROM├────►│LLB├─────►│iBoot├───x│ramdisk├──►│Kernel├───►│RestoreMode│
└───────┘     └───┘      └─────┘    └───────┘   └──────┘    └───────────┘

```

DevTeam 查看了 `iTunesMobile.dll` 内部以了解 iTunes 如何写入文件系统以执行恢复。他们发现了诸如 `mount`、`umount` 和 `ditto`（用于复制文件）等命令，并编写了一个名为 [iPHUC](https://github.com/svn2github/iphuc)，能够通过 `iTunesMobile.dll` 的私有方法与恢复模式下的设备进行通信[^([21])](#footnote_21)。

`iPHUC` 的源代码后来被公开，我们可以看看它的工作原理。

1.  用户将手机置于恢复模式。

1.  [发送 ramdisk 到手机（grestore 命令）](https://github.com/svn2github/iphuc/blob/master/RecoveryInterface.cpp#L77C8-L77C24)[^([22])](#footnote_22)[^([23])](#footnote_23)。

1.  [将 ramdisk 加载到手机 RAM 中。](https://github.com/svn2github/iphuc/blob/df564835edd8a65f6c9e08fc5e837815bd546775/RecoveryInterface.cpp#L87)

1.  [发送 kernelcache。](https://github.com/svn2github/iphuc/blob/df564835edd8a65f6c9e08fc5e837815bd546775/RecoveryInterface.cpp#L100C5-L100C25)

1.  [引导内核指向 ramdisk。](https://github.com/svn2github/iphuc/blob/df564835edd8a65f6c9e08fc5e837815bd546775/RecoveryInterface.cpp#L115)

1.  现在手机处于恢复模式。

恢复模式中的 iBoot 有一个有趣的属性。在加载内核之前，它不会检查其签名。虽然这与主题无关，但后来允许 DevTeam 加载一个修补的内核，允许执行未签名可执行文件的 `execl`。

现在我们掌握了所有的知识，可以理解详细的越狱步骤，详见 [这里](https://web.archive.org/web/20071005150518/http://iphone.fiveforty.net/wiki/index.php/How_to_Escape_Jail)。

1.  将手机放入恢复模式

1.  使用恢复模式 `mount` 挂载系统和用户分区。

1.  使用 `ditto` 将 `/etc/fstab` 和 `/System/Library/Lockdown/Services.plist` 复制到 `/root/Media`。

1.  使用 iTunes 将这些文件复制到工作站。

1.  使用工作站文本编辑器修改这些文件如下。

    1.  [修改 fstab](https://www.theiphonewiki.com/wiki//private/etc/fstab) 以在系统分区中使用“rw”模式而不是“r”模式。

    1.  [创建第二个 afcd 服务](https://web.archive.org/web/20071005150518/http://iphone.fiveforty.net/wiki/index.php/How_to_Escape_Jail#:~:text=string%3E%2Dd%3C/string%3E-,%3Cstring%3E/%3C/string%3E,-%3C/array%3E%0A%09%3C/dict%3E) 在 `Services.plist` 中，不是在 `/root/Media` 而是在 `/`。

1.  使用 iTunes 将这些文件推送回 `/root/Media`。

1.  `ditto` 将修改后的文件复制回系统分区的原始位置。

1.  重新启动

重新启动后，iTunes 由于 `acfd2` 可以看到整个文件系统。由于系统和用户分区都以“读写”方式挂载，“DevTeam” 已经实现了对系统的完全读写访问。

```

                ┌──────┐
                │iPhone│
┌──────┐        ├──────┤
│iTunes│        │ acfd2│
└──┬───┘        └──┬───┘
   │               │
   │  write(file)  │
   ├──────────────►│
   │               │
   │               │
   │               ▼
   ▼               /

```

| 解密固件 | 绕过激活 | 获得写入权限 | 获得工作工具链 | 解锁手机 | 启用第三方应用程序 |
| --- | --- | --- | --- | --- | --- |

后来，激活和写入权限都自动化到了名为“INdependence”的桌面 MacOS X 应用程序中。

工具链 / 启用第三方里程碑

* * *

关于如何实现这一点的具体信息不多，只知道至少有十二人参与其中[^([24])](#footnote_24)。到 2007 年 7 月 19 日，完成了能够针对 ARM 的 `binutils` 工具链，详见 [这里](https://web.archive.org/web/20070904020127/http://iphone.fiveforty.net:80/wiki/index.php/Past_Progress_Reports#:~:text=After%20many%2C%20many%20hours,%2D%20the%20dev%20team)。这使得 DevTeam 能够在设备上运行他们编写的程序。

> 经过多个小时的紧张工作，夜巡完成了第一个独立的“Hello World”* 应用程序，成功在 iPhone 上编译并启动。这得益于“ARM/Mach-O 工具链”，夜巡过去几周认真开发的“特别项目”。工具链的某些部分（如汇编器）正在进行优化和测试，尽快发布。
> 
> 值得注意的是，夜巡在创建这些工具方面发挥了重要作用，几乎是独自工作，确保项目完成。夜巡还负责开发了“越狱漏洞”，从他和团队其他成员发现的信息中开发出来。
> 
> 请加入我们一起感谢 Nightwatch、Tmiw、Darkten 和 Daeken 让这一切成为可能。
> 
> - iPhoneDevTeam Wiki/div>
> 
> > mach-o 和 ARM：此前从未在苹果之外完成；我们需要自己编写（也就是惊叹地看着夜巡干的事）
> > 
> > - Geohotz<
> > 
> > [^([25])](#footnote_25)
> > 
> > /div>
> > 
> > 工具链的另一个目标是重建一个头文件（MobileTerminal.h），可以从`iTunesMobile.so`中公开私有函数，并与`afc`通信，而无需运行 iTunes。
> > 
> > 几次演示提到内核在允许`execl`前检查可执行文件的签名。第一代 iPhone 并没有做这件事，这可能是在 v1.1.1 中引入的。
> > 
> > 解锁里程碑
> > 
> > * * *
> > 
> > 最后，我们达到了最后一项和整个努力的目标。DevTeam 大约在 [2007年8月14日](https://web.archive.org/web/20070814011425/http://iphone.fiveforty.net/wiki/index.php/Main_Page) 达到了这个目标。请注意，颜色是橙色，而不是红色。看来他们知道解锁即将发生。
> > 
> > | 解密固件 | 绕过激活 | 获取写入访问权限 | 获取工作工具链 | 启用第三方应用 | 解锁手机 |
> > | --- | --- | --- | --- | --- | --- |
> > 
> > 在继续之前，简要介绍一下 iPhone 的结构。智能手机实际上由两部分组成。智能部分是 iOS 和手机/调制解调器（其固件称为“基带”）。这是两个不同的系统，有各自的RAM、CPU、存储、固件甚至自己的振荡器。它们通过 UART 线（安装在`/dev/tty.baseband`上）使用 AT 命令（例如：[使用 AT 命令发送短信](https://www.smssolutions.net/tutorials/gsm/sendsmsat/)）进行通信。
> > 
> > ```
> >         ┌─────────────────────────────────────────────────────────┐
> >         │                        iPhone                           │
> >         │                                                         │
> >         │   ┌────────────┐                     ┌───────────────┐  │
> >         │   │            │         UART        │               │  │
> >         │   │    iOS     │◄───────────────────►│    Baseband   │  │
> >         │   │            │  /dev/tty.baseband  │               │  │
> >         │   └────────────┘                     └───────────────┘  │
> >         └─────────────────────────────────────────────────────────┘
> > 
> > ```
> > 
> > 解锁过程的工作原理几乎从第一天起就是[众所周知的](https://web.archive.org/web/20070904020127/http://iphone.fiveforty.net:80/wiki/index.php/Past_Progress_Reports#:~:text=We%20even%20know,to%20AT%26T.)（详细内容[在这里](https://www.theiphonewiki.com/wiki/Unlock#:~:text=At%20%2B0x400%20in%20the%20seczone%2C%20a%20token%20is%20stored%20encrypted%20with%20(NCK%20%2B%20NORID%20%2B%20HWID)）。
> > 
> > > 我们甚至知道解锁的 AT 命令。它是 'AT+CLCK="PN",0,"xxxxxxxx"'。
> > > 
> > > 但是祝你好运找到那些 x。它们被称为 NCK，即网络控制密钥，据信每个人的手机都是唯一的。忘记暴力破解（时间不实用）和显而易见的条目。如果你仍然认为暴力破解是个好主意，[阅读这篇文章](https://web.archive.org/web/20070825072344/http://lpahome.com/iPhone/youarestupid.txt)。
> > > 
> > > 此外，每部手机解锁尝试的上限为 3-10 次，在此之后固件将“硬锁”到 AT&T。
> > > 
> > > - iPhoneDevTeam wiki
> > > 
> > 基带也有一个 BootROM，以及一系列的加载程序建立了一条信任链，所以一切最终都要根据签名进行验证。这些密钥当然只在运行时可用，无法提取。此外，与基带的工作也很困难。
> > 
> > > 现在关于基带的重要事情，也是最令人恼火的事情，是没有 DFU/恢复模式，我一直对 planetbeing、wizdaz 和 pumpkin 等人很羡慕，因为他们总是有一个故障安全模式，基本上可以让他们为手机做的一切事情无所顾忌。
> > > 
> > > 我们中的一些人在几个时间点完全擦除了 NOR，完全使 LLB 失效之类的事情。如果你的设备中有无效的 LLB，这是第二阶段，你的手机基本上会在黑屏中快速闪烁，并且屏幕上会出现可怕的伤痕，非常可怕，让你觉得它完全报废了，我们将其昵称为“圣诞树模式”。
> > > 
> > > 但是，虽然当时看起来很糟糕，只要你掌握了好的时间控制，你总是可以进入 DFU 模式并从中恢复。在基带中没有这样的事情；有些操作可能会永久性地使你的手机变砖。
> > > 
> > 到了[2007年7月](https://gizmodo.com/iphone-reverse-engineering-opens-new-door-to-total-unlo-284614)，DevTeam 已从 ipsw 存档中逆向工程出基带。此外，他们还研究了负责更新基带的可执行文件 `/usr/local/bin/bbupdater`，该文件位于 ramdisk 中。他们知道了所有上传新基带的命令。
> > 
> > 这导致了第一个命令行 [`iUnlock`](https://web.archive.org/web/20111121110900/http://gizmodo.com/assets/resources/2007/09/iunlock_src.zip)，其中包括许多文件（例如转储的固件 "nor" 和 secpack 文件 "ICE03.12.06_G.fls"）。不久后，他们发布了一个更简单的 `anySIM` 应用程序，只需点击一个按钮即可上传并运行到手机上。
> > 
> > 由于[源代码](https://code.google.com/archive/p/devteam-anysim/source/default/source)也已发布，我们可以查看内部并了解其工作原理。
> > 
> > 1.  打开 `/dev/tty.baseband` 并设置调制解调器参数（例如：波特率）。
> > 1.  
> >     ```
> >      fd = open("/dev/tty.baseband");
> >     ```
> >     
> > 1.  转储基带（即 NOR，大小为 4MiB）到 /tmp。
> > 1.  
> > 1.  将基带加载到 RAM 中。
> > 1.  
> > 1.  加载 Secpack（来自 ramdisk 中名为 ICE03.12.06_G.fls 的文件）。
> > 1.  
> > 1.  在 RAM 中修补基带指令（使任何 NCK 允许解锁）。
> > 1.  
> >     ```
> >       int offset = 0x216f28;
> >     
> >       Buffer[offset + 0] = 0x01;
> >       Buffer[offset + 1] = 0x00;
> >       Buffer[offset + 2] = 0xa0;
> >       Buffer[offset + 3] = 0xe3;
> >     ```
> >     
> > 1.  推回基带。
> > 1.  
> >     ```
> >       SendBeginSecpack(fd, secpack); \\ secpack explanation here 
> >       free(secpack);
> >     
> >       SendErase(fd, 0xA0020000, 0xA03bfffe);
> >       Seek(fd, 0xA0020000 - 0x400); // Firmware must be at 0xA0020000\. WTH?
> >       
> >       unsigned char foo[0x400];
> >       memset(foo, 0, 0x400);
> >       SendWrite(fd, foo, 0x400, false);
> >       
> >       SendWrite(fd, fw, fwsize, true);
> >       SendEndSecpack(fd);
> >       ValidateFW(fd, 0xA0020000, fw, 0x800);
> >     
> >     ```
> >     
> > 1.  解锁。
> > 1.  
> >     ```
> >       AT+CLCK="PN",0,"00000000"
> >       AT+CLCK="PN",2
> >     
> >     ```
> >     
> > 总结一下，`anySIM`的工作原理是在手机运行时候，转储基带固件（相当酷），然后进行修补 [^([27])](#footnote_27)（使用原始字节写入，同样相当酷！），最后上传回去。但这实际上不应该奏效，因为上传时会检查固件签名，而固件已经被修改过。固件上传应该会因为签名检查失败而失败。
> > 
> > 看起来魔法似乎在于 [minus 0x400](https://www.theiphonewiki.com/wiki/Minus_0x400) 的偏移量。为什么有效并不是很清楚，即使 GeoHotz 解释过。
> > 
> > > 在签名验证之前，前 0x400 字节不会被写入。所以，开始写入前 0x400 字节 :-)
> > > 
> > > - Geohotz
> > > 
> > 在本文发布后，Hackernews 的善良之人们加入了解释 [^([28])](#footnote_28) `-0x400` 的过程。具体如下。
> > 
> > 基带以最多`0x800`字节的块接收新固件。它无法将整个4MiB存储在RAM中，然后进行校验和最后写入闪存。相反，字节在接收时即写入闪存，除了前`0x400`字节以外，这些字节在RAM中缓冲。当固件完全上传时，基带执行校验和。如果测试失败，则缓冲的字节将被丢弃，而不会写入闪存（基带固件损坏，将无法启动）。如果固件通过测试，则`0x400`字节将写入固件开头。
> > 
> > `-0x400`技巧的原理是首先在应写入固件的位置前写入垃圾0x400字节。然后发送4MiB固件（偏移量与应写入固件的确切位置匹配）。当校验和测试不可避免地失败时，垃圾字节将被丢弃。但整个新固件已经正确地刷新了！
> > 
> > | 解密固件 | 绕过激活 | 获取写入权限 | 获取工作工具链 | 启用第三方应用程序 | 解锁手机 |
> > | --- | --- | --- | --- | --- | --- |
> > 
> > 把所有东西放在一起
> > 
> > * * *
> > 
> > [解锁说明](https://web.archive.org/web/20071011194202/http://iphone.fiveforty.net/wiki/index.php/Software_Unlock)的完整列表于[2007年9月12日](https://gizmodo.com/the-complete-iphone-unlock-star-wars-timeline-304310)发布。其中包括按大洲划分的证言[^([29])](#footnote_29)，特别是加拿大[^([30])](#footnote_30)的证言。
> > 
> > 结语
> > 
> > * * *
> > 
> > 2007年9月27日，苹果迅速发布了iPhone固件V1.1.1。进度条被重置[^([31])](#footnote_31)，猫鼠游戏由此开始，至今仍在进行。
> > 
> > | 解密 1.1.1 | 获取写入权限 1.1.1 | 激活 1.1.1 | 解锁 1.1.1 | 启用第三方应用程序 1.1.1 |
> > | --- | --- | --- | --- | --- |
> > 
> > 深入了解
> > 
> > * * *
> > 
> > 如果你对2007年解锁的更多考古学感兴趣，这里是一些链接供参考。
> > 
> > 一更事情
> > 
> > * * *
> > 
> > 本文得以实现，感谢互联网档案馆及其WayBack机器的卓越性能。请花一分钟[捐赠](https://archive.org/donate)。
> > 
> > 参考资料
> > 
> > * * *
> > 
> > * * *
> > 
> > *
