<!--yml

分类：未分类

日期：2024-05-29 12:38:11

-->

# debian - 我已经删除了systemd - Unix & Linux Stack Exchange

> 来源：[https://unix.stackexchange.com/questions/772960/i-have-deleted-systemd](https://unix.stackexchange.com/questions/772960/i-have-deleted-systemd)

我假设以下所有命令都是以root用户身份运行的。要么直接以root身份登录，要么如果不能，登录为允许使用`sudo`的用户，然后使用`sudo -i`：应该会有一个以`#`结尾的root提示符。

你应该备份`/var/log/apt`的内容

```
cp -a /var/log/apt /root/log-apt-backup 
```

这样你以后可以找到其他已删除的包需要重新安装。

随着对system V的支持逐渐降低，我可以假设一些图形界面部件依赖于systemd的功能，因此无法启动。

你应该重新安装`systemd`，但这可能还不够：为了触发额外所需的依赖关系，你应该删除提供System V的`/sbin/init`的内容：`sysvinit-core`。就像删除`systemd`触发了替换system V包的安装一样，删除这个内容将会触发systemd生态系统中的替换。

在一个空的LXC容器上测试过这个：

```
root@bookworm-test:~# apt remove systemd
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following additional packages will be installed:
  initscripts insserv orphan-sysvinit-scripts sensible-utils startpar sysv-rc
  sysvinit-core ucf
Suggested packages:
  bootchart2 bootlogd
The following packages will be REMOVED:
  dbus-user-session libnss-systemd libpam-systemd systemd systemd-resolved
  systemd-sysv
The following NEW packages will be installed:
  initscripts insserv orphan-sysvinit-scripts sensible-utils startpar sysv-rc
  sysvinit-core ucf
0 upgraded, 8 newly installed, 6 to remove and 0 not upgraded. 
```

然后重新启动，修复等（不要运行，不够）：

```
# apt install systemd
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following additional packages will be installed:
  systemd-timesyncd
Suggested packages:
  systemd-container systemd-homed systemd-userdbd systemd-boot
  systemd-resolved libqrencode4 libtss2-esys-3.0.2-0 libtss2-mu0 libtss2-rc0
  polkitd | policykit-1
The following NEW packages will be installed:
  systemd systemd-timesyncd
0 upgraded, 2 newly installed, 0 to remove and 0 not upgraded. 
```

不够有用（它会保留sys V init）。

相反：

```
root@bookworm-test:~# apt remove sysvinit-core
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following packages were automatically installed and are no longer required:
  initscripts insserv startpar sysv-rc
Use 'apt autoremove' to remove them.
The following additional packages will be installed:
  dbus-user-session libnss-systemd libpam-systemd systemd systemd-sysv
  systemd-timesyncd
Suggested packages:
  systemd-container systemd-homed systemd-userdbd systemd-boot
  systemd-resolved libqrencode4 libtss2-esys-3.0.2-0 libtss2-mu0 libtss2-rc0
  polkitd | policykit-1
The following packages will be REMOVED:
  sysvinit-core
The following NEW packages will be installed:
  dbus-user-session libnss-systemd libpam-systemd systemd systemd-sysv
  systemd-timesyncd
0 upgraded, 6 newly installed, 1 to remove and 0 not upgraded. 
```

看起来更有用。至少应该安装回一个建议的软件包：`polkitd`（通过在`remove`操作的末尾添加一个`+`，可以在一个`remove`操作内完成）。所以最后运行：

```
apt remove sysvinit-core polkitd+ 
```

这需要网络。看起来OP（实际上我的LXC测试）默认不再具备网络。这应该很容易通过名为`eth0`的有线以太网接口解决（实际名称可以从`ip -br link`的结果中检索，如果需要，用新名称替换下面的`eth0`）：

```
dhclient -v eth0 
```

然后再试一次之前的`apt`命令。

在最小系统上会显示类似以下内容：

```
# apt remove sysvinit-core polkitd+
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following packages were automatically installed and are no longer required:
  initscripts insserv startpar sysv-rc
Use 'apt autoremove' to remove them.
The following additional packages will be installed:
  dbus-user-session libduktape207 libglib2.0-0 libglib2.0-data libicu72
  libnss-systemd libpam-systemd libpolkit-agent-1-0 libpolkit-gobject-1-0
  libxml2 sgml-base shared-mime-info systemd systemd-sysv systemd-timesyncd
  xdg-user-dirs xml-core
Suggested packages:
  low-memory-monitor polkitd-pkla sgml-base-doc systemd-container
  systemd-homed systemd-userdbd systemd-boot systemd-resolved libqrencode4
  libtss2-esys-3.0.2-0 libtss2-mu0 libtss2-rc0 debhelper
The following packages will be REMOVED:
  sysvinit-core
The following NEW packages will be installed:
  dbus-user-session libduktape207 libglib2.0-0 libglib2.0-data libicu72
  libnss-systemd libpam-systemd libpolkit-agent-1-0 libpolkit-gobject-1-0
  libxml2 polkitd sgml-base shared-mime-info systemd systemd-sysv
  systemd-timesyncd xdg-user-dirs xml-core
0 upgraded, 18 newly installed, 1 to remove and 0 not upgraded. 
```

并重新启动，切换回systemd。这可能恢复或不恢复图形界面，但应该已经切换回了`systemd`。要验证，可以运行（作为root）：

```
ldd /proc/1/exe | grep systemd 
```

应该有非空输出。

一旦完成上述操作，请检查`/root/log-apt-backup/history.log`中的日志备份，并搜索与你所执行的命令相似的行：`Commandline: apt remove systemd`和`Commandline: apt autoremove systemd`。检查所有已删除软件包，确保在它们各自的`Remove:`条目下面补充可能还缺少的内容。
