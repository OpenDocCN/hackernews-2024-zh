<!--yml

category: 未分类

日期：2024-05-29 13:20:38

-->

# Solene'%：一些并不广为人知的 OpenBSD 特性

> 来源：[https://dataswamp.org/~solene/2024-02-20-rarely-known-openbsd-features.html](https://dataswamp.org/~solene/2024-02-20-rarely-known-openbsd-features.html)

# 1\. 介绍 [§](#_Introduction)

在本博客文章中，您将了解一些可能有用但不常见的 OpenBSD 功能。

这些功能通常具有小众用途，但重要的是知道它们的存在，以免重复发明轮子 :)

[OpenBSD 官方项目网站](https://www.openbsd.org)

# 2\. 特性 [§](#_Features)

下面列出的功能并非全部特定于 OpenBSD，因为一些功能也可以在其他 BSD 系统上找到。大多数知识对 Linux 用户来说并不实用。

## 2.1\. 安全级别 [§](#_Secure_level)

安全级别是一个名为 `kern.securelevel` 的 sysctl，它有4个不同的值，从级别 -1 到级别 2，只能增加级别。默认情况下，系统在多用户模式下进入安全级别 1（正常安装时的默认模式）。

然后可以升级到最后的安全级别（2），这将启用以下额外的安全措施：

+   所有原始磁盘都是只读的，因此无法尝试对存储设备进行更改

+   时间几乎被锁定，只能通过小幅度的步骤慢慢调整时钟（可能是每隔一段时间最多调整1秒）

+   PF 防火墙规则不能被修改、清空或更改

这个功能对于专用防火墙特别有用，其规则很少改变。阻止时间改变对于远程日志记录非常有用，因为它确保了“事件发生的时间”，并且您可以确信过去的日志没有被修改。

默认安全级别1已经启用一些额外的安全功能，例如无法删除“不可变”和“仅追加”文件标志，这些被忽视的标志（可以使用 chflags 应用）可以锁定文件，防止任何人修改它们。仅追加标志对于日志非常有用，因为无法修改内容，但这并不阻止添加新内容，因此无法通过这种方式修改历史。

[OpenBSD 手册页面：securelevel](https://man.openbsd.org/securelevel)

[OpenBSD 手册页面：chflags](https://man.openbsd.org/chflags)

这个功能也存在于其他 BSD 系统中。

OpenBSD 的内存分配器可以进行调整，系统范围内或每个命令，以增加额外的检查。这可以用于安全原因或在程序中查找与内存分配相关的错误（这是非常常见的...）。

有两种方法可以应用这些更改：

+   通过在 `/etc/sysctl.conf` 中使用 sysctl `vm.malloc_conf` 进行系统范围的设置（确保在那里引用其值，某些字符如 `>` 否则会造成问题，已经遇到过...）

+   在命令行上，通过在运行程序之前加入 `env MALLOC_OPTIONS="flags"` 来设置

man 页面列出了一些选项作为选项使用的标志列表，最容易使用的是 `S`（用于安全检查）。在 man 页面中指出，一个程序如果除了 X 之外的任何标志都会导致其出现错误，因此如果您使用 malloc 选项并且程序崩溃了，这不是您的错（除非您编写了代码;-)）。

[OpenBSD 手册页面：malloc（搜索 MALLOC OPTIONS）](https://man.openbsd.org/malloc)

## 2.3\. 文件标志 [§](#_文件标志)

您肯定习惯于文件属性，如权限或所有权，但在许多文件系统（包括 OpenBSD ffs）中，也存在标志！

可以使用 `chflags` 命令更改文件的标志，有几个可用的标志：

+   nodump：防止文件被 `dump` 命令保存（除非您使用 dump 中的标志来绕过此操作）

+   sappnd：文件只能以写入追加模式使用，只有 root 可以设置/移除此标志。

+   schg：文件不能更改，变为不可变，只有 root 可以更改此标志。

+   uappnd：与 sappnd 模式相同，但用户可以更改该标志。

+   uchg：与 schg 模式相同，但用户可以更改该标志。

如上面的安全级别部分所述，在安全级别 1（默认情况下！），不能移除 sappnd 和 schg 标志，您需要在单用户模式下启动以移除这些标志。

提示：使用 `chflags 0 file [...]` 在文件上移除标志。

您可以使用 `ls -ol` 检查文件的标志，这看起来像这样：

```
terra$ chflags uchg get_extra_users.sh
terra$ ls -lo get_extra_users.sh        
-rwxr-xr-x  1 solene  solene  uchg 749 Apr  3  2023 get_extra_users.sh

terra$ chflags 0 get_extra_users.sh     
terra$ ls -lo get_extra_users.sh     
-rwxr-xr-x  1 solene  solene  - 749 Apr  3  2023 get_extra_users.sh 
```

[OpenBSD 手册页面：chflags](https://man.openbsd.org/chflags)

过去几年，OpenBSD crontab 格式增加了一些新功能。

+   时间字段的随机数：您可以在字段中使用 `~` 而不是数字或 `*` 来生成一个随机值，该值在重新加载 crontab 之前会保持稳定。类似 `~/5` 的事情也可以。您可以使用 `20~40` 强制在范围内获取随机值，以获取 20 到 40 的值。

+   仅在 cron 作业的返回代码不为 0 时发送电子邮件：在时间和命令之间添加 `-n`，如 `0 * * * * -n /bin/something`。

+   仅运行一个作业实例：在时间和命令之间添加 `-s`，如 `* * * * * -s /bin/something`。对于不应同时运行两次的 cron 作业非常有用，如果作业持续时间长于通常时间，则确保在前一个实例完成之前不会启动新实例。

+   无日志记录：在时间和命令之间添加 `-q`，如 `* * * * -q /bin/something`，效果是此 cron 作业将不会记录在 `/var/cron/log` 中。

可以使用 `-ns` 等标志的组合。随机时间对于具有多个系统的情况非常有用，如果不希望它们同时在远程服务器上触发大量 I/O，则可以在某种情况下运行命令。这是为了防止通常运行 `0 * * * * sleep $(( $RANDOM % 3600 )) && something` 的睡眠命令，在运行命令之前运行一个最多一个小时的随机时间。

[OpenBSD 手册页面：crontab](https://man.openbsd.org/crontab.5)

OpenBSD的一个很酷的功能是能够轻松创建带有预配置答案的安装媒体。这通过在`bsd.rd`安装内核中注入特定文件来实现。

有一个名为upobsd的简单工具，是由semarie@创建的，用于轻松修改这样的bsd.rd文件，以包含autoinstall文件，我分叉了该项目以继续维护它。

除了自动安装OpenBSD与用户、ssh配置、要安装的集等...，还可以添加一个site.tgz归档文件，以及通常的集归档文件一起，其中包含您想要添加到系统的文件，这可以包括在首次启动时运行的脚本以触发一些自动化！

如果您在生产环境中运行OpenBSD，并且有很多设备要管理，自动将新设备加入到设备群中应尽可能自动化。

[GitHub 项目页面：upobsd](https://github.com/rapenne-s/upobsd)

[OpenBSD 手册页面：autoinstall](https://man.openbsd.org/autoinstall)

## 2.6\. apmd守护程序挂钩 [§](#_apmd_daemon_hooks)

Apmd 在大多数运行OpenBSD的笔记本电脑和台式机上肯定正在运行，但它具有与其命令行标志无关的功能，因此您可能已经错过了它们。

不同的文件名可以包含在某些事件（如挂起、恢复、休眠等）发生时运行的脚本。

在挂起时，在X会话中运行`xlock`是OpenBSD的一个经典用法，因此系统将在恢复时需要密码。

[旧博客文章：从apmd挂起脚本到xlock](https://dataswamp.org/~solene/2021-07-30-openbsd-xidle-xlock.html#_Resume_/_Suspend_case)

手册页面解释了所有内容，但基本上当您将笔记本连接到电源插座时运行备份程序，就像这样：

```
# mkdir -p /etc/apm
# vi /etc/apm/powerup 
```

您需要编写一个常规脚本：

```
#!/bin/sh

/usr/local/bin/my_backup_script 
```

然后，将其设为可执行。

```
# chmod +x /etc/apm/powerup 
```

当您将系统连接到交流电源时，守护程序apmd将自动运行此脚本。

方法对以下情况相同：

+   休眠

+   恢复

+   暂停

+   待机

+   休眠

+   powerup

+   powerdown

这使得在这些事件上安排任务非常容易。

[OpenBSD 手册页面：apmd（FILES部分）](https://man.openbsd.org/apmd#FILES)

## 2.7\. 使用hotplugd处理设备事件挂钩 [§](#_Using_hotplugd_for_hooks_on_devices_events)

类似于apmd通过事件运行脚本的方式，hotplugd是一个服务，允许在添加/移除设备时运行脚本。

在系统插入USB存储器时，自动挂载USB存储器是一个典型的用法，或者在打开USB打印机电源时启动cups守护程序。

脚本接收两个参数，代表设备类和设备名称，因此您可以在您的脚本中使用它们来了解连接了什么。手册页中提供的示例是一个很好的起点。

这些脚本确实不容易编写，您需要制定一个期望的硬件精确列表以及每个硬件的运行内容，并且不要忘记跳过未知硬件。不要忘记将脚本设为可执行，否则它将无法工作。

[OpenBSD 手册页：hotplugd](https://man.openbsd.org/hotplugd)

## 2.8\. Altroot [§](#_Altroot)

最后，有一个看起来相当酷的功能。在每日脚本中，如果 OpenBSD 分区 `/altroot/` 存在于 `/etc/fstab` 中，并且每日脚本环境具有变量 `ROOTBACKUP=1`，则根分区将被复制到此分区。这允许保持额外的根分区与主根分区同步。显然，如果 altroot 分区在另一个驱动器上，它会更有用。复制是通过 `dd` 完成的。你可以通过检查脚本 `/etc/daily` 查看确切的代码。

然而，如果你没有安装引导加载程序或在磁盘上创建 EFI 分区，如何从这个分区引导是不清楚的...

[OpenBSD 手册页：hier（hier 代表文件系统层次结构）](https://man.openbsd.org/hier)

[OpenBSD 手册页：daily](https://man.openbsd.org/daily)

[OpenBSD FAQ：根分区备份](https://www.openbsd.org/faq/faq14.html#altroot)

## 2.9\. talk：终端中的本地聊天 [§](#_talk:_local_chat_in_the_terminal)

OpenBSD 自带一个名为 "talk" 的程序，这个程序可以创建一个与另一个用户的一对一聊天，无论是在本地系统还是远程系统（设置更复杂）。这不是异步的，两个用户必须同时登录系统才能使用 `talk`。

这个程序不仅限于 OpenBSD，并且可以在 Linux 上使用，但它非常有趣、高效且易于设置，我想写写它。

安装很简单：

```
# echo "ntalk		dgram	udp	wait	root	/usr/libexec/ntalkd	ntalkd" >> /etc/inetd.conf
# rcctl enable inetd
# rcctl start inetd 
```

通信发生在本地主机的 UDP 端口 517 和 518 上，不要将它们开放到互联网！如果你想允许远程系统，使用 VPN 加密流量，并仅允许 VPN 的端口 517/518。

使用很简单，如果你希望 alice 和 bob 互相交谈：

+   alice 输入 `talk bob`，bob 也必须已经登录。

+   bob 在他们的终端收到消息，alice 想要交谈。

+   bob 输入 `talk alice`

+   为两个用户显示终端用户界面，他们写的内容将出现在界面的上半部分，并且接收者的消息将出现在下半部分。

这有点古老，但是它很有效，并且随同基本系统提供。当你只想和某人交谈时，它可以胜任。

# 3\. 结论 [§](#_Conclusion)

OpenBSD 上有一些有趣的特性，我想稍作介绍，也许你会觉得它们有用。如果你知道可以添加到此列表的酷功能，请联系我！
