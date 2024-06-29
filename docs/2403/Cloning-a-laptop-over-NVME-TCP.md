<!--yml

类别：未分类

日期：2024年5月27日 14:52:46

-->

# 通过NVME TCP克隆笔记本电脑

> 来源：[https://copyninja.in/blog/clone_laptop_nvmet.html](https://copyninja.in/blog/clone_laptop_nvmet.html)

发布于2024年3月10日，由devops下的copyninja发布

最近，我买了一台新笔记本电脑，需要设置它才能使用。但我真的不想重复我之前在这篇[帖子中解释的同样的旧步骤](https://copyninja.in/blog/live_install_debian.html)。我向我的同事抱怨了这一点，他建议为何不将整个硬盘复制到新笔记本电脑上。虽然这对我来说听起来像一个有趣的点子，但我有我的疑虑，所以这就是我回答他的内容。

1.  我没有工具可以打开我的旧笔记本电脑，并通过USB将新硬盘连接到我的新笔记本电脑上。

1.  我使用的是完整的磁盘加密，我的旧笔记本电脑有一个512GB的硬盘，而新笔记本电脑有一个1TB的NVME，我不太熟悉调整LUKS。

他迅速建议可以通过两步完成。第1步，只需使用NVME通过TCP公开硬盘并通过网络连接并进行全硬盘复制，其余的都很容易实现。简而言之，他建议如下：

1.  从旧笔记本电脑使用nvmet-tcp导出磁盘。

1.  将磁盘复制到新笔记本电脑。

1.  调整分区以使用完整的1TB。

1.  调整LUKS

1.  最后，调整BTRFS根磁盘。

## 通过NVME TCP导出磁盘

我同事建议我做这个最简单的方法是使用[systemd-storagetm.service](https://www.freedesktop.org/software/systemd/man/latest/systemd-storagetm.service.html)。可以通过简单地引导到*storage-target-mode.target*并指定*rd.systemd.unit=storage-target-mode.target*来调用此服务。但他建议不要使用这个，因为我需要调整dracut initrd映像以涉及网络服务以及在此模式下配置WiFi是一件痛苦的事情。

或者，我简单地用GRML rescue CD启动了我的两台笔记本电脑。然后在我的当前笔记本电脑上，使用Linux的nvmet-tcp模块导出NVME硬盘完成了以下步骤：

```
modprobe  nvmet-tcp
cd  /sys/kernel/config/nvmet
mkdir  ports/0
cd  ports/0
echo  "ipv4"  >  addr_adrfam
echo  0.0.0.0  >  addr_traaddr
echo  4420  >  addr_trsvcid
echo  tcp  >  addr_trtype

cd  /sys/kernel/config/nvmet/subsystems
mkdir  testnqn
echo  1  >testnqn/allow_any_host
mkdir  testnqn/namespaces/1

cd  testnqn
# replace the device name with the disk you want to export
echo  "/dev/nvme0n1"  >  namespaces/1/device_path
echo  1  >  namespaces/1/enable

ln  -s  "../../subsystems/testnqn"  /sys/kernel/config/nvmet/ports/0/subsystems/testnqn 
```

这些步骤确保设备现在通过NVME通过TCP导出。下一步是在新笔记本电脑上检测此设备并连接该设备：

```
nvme  discover  -t  tcp  -a  <ip>  -s  4420
nvme  connectl-all  -t  tcp  -a  <>  -s  4420 
```

最后，`nvme list`显示了连接到新笔记本电脑的设备，并且我们可以继续下一步，即进行磁盘复制。

## 复制磁盘

我只是使用`dd`命令将根磁盘复制到我的新笔记本电脑上。由于新笔记本电脑没有以太网端口，我只能依赖WiFi，花了大约7个半小时将整个512GB复制到新笔记本电脑上。我复制的速度约为18-20MB/s。另一种选择是创建一个初始分区和文件系统并对根磁盘进行rsync复制，或者使用BTRFS本身进行文件系统传输。

```
dd  if=/dev/nvme2n1  of=/dev/nvme0n1  status=progress  bs=40M 
```

## 调整分区和LUKS容器

最后一部分非常简单。当我启动`parted`时，它检测到分区表与磁盘大小不匹配，并询问是否可以修复，我选择了是。接下来，我必须安装`cloud-guest-utils`来获取`growpart`以修复第二个分区，然后以下命令将分区扩展到完整的1TB：

接下来，我使用`cryptsetup-resize`来增加LUKS容器的大小。

```
cryptsetup  luksOpen  /dev/nvme0n1p2  ENC
cryptsetup  resize  ENC 
```

最后，我重新启动进入磁盘，一切正常。登录系统后，我调整了BTRFS文件系统的大小。BTRFS要求系统被挂载才能调整大小，所以我无法在实时启动中尝试。

```
btfs  fielsystem  resize  max  / 
```

## 结论

整个过程唯一的好处是我有了一台新笔记本，但我仍然觉得自己在使用现有的笔记本。通常，设置新笔记本需要一到两周的时间才能完全适应，但在这种情况下，整个过程的时间都被节省了。

一项额外的好处是，由于我的同事的帮助，我学会了如何使用NVME通过TCP导出磁盘。这些新知识增加了经验的价值。
