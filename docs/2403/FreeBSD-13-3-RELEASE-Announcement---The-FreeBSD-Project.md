<!--yml

类别：未分类

日期：2024-05-27 14:40:53

-->

# FreeBSD 13.3-RELEASE公告 | FreeBSD项目

> 来源：[https://www.freebsd.org/releases/13.3R/announce/](https://www.freebsd.org/releases/13.3R/announce/)

FreeBSD 13.3-RELEASE现已提供amd64、i386、powerpc、powerpc64、powerpc64le、powerpcspe、armv6、armv7、aarch64和riscv64架构。

FreeBSD 13.3-RELEASE可以从可启动的ISO映像或通过网络安装。某些架构还支持从USB存储设备安装。可以按照下面的部分描述下载所需的文件。

在此消息的底部包含了发布ISO、内存棒和SD卡映像的SHA512和SHA256哈希值。

发布镜像的PGP签名校验和也可在以下位置找到：

此公告的PGP签名版本可在以下位置找到：

发布的镜像的目的如下：

dvd1

它包含安装基本FreeBSD操作系统所需的一切、文档、调试分发集和旨在让图形工作站运行的少量预构建软件包。它还支持启动到基于“livefs”的救援模式。如果您可以刻录并使用DVD大小的媒体，这应该是您所需要的全部内容。

此外，可将其写入适用于amd64架构的USB存储设备（闪存驱动器），并用于可从USB驱动器启动的计算机进行安装。它还支持启动到基于“livefs”的救援模式。

作为如何使用memstick映像的示例之一，假设USB驱动器显示为您机器上的/dev/da0，像这样应该可以工作：

```
# dd if=FreeBSD-13.3-RELEASE-amd64-dvd1.iso \
    of=/dev/da0 bs=1m conv=sync
```

要确保正确获取目标（of=）。

disc1

它包含基本的FreeBSD操作系统。它还支持启动到基于“livefs”的救援模式。没有预构建的软件包。

此外，可将其写入适用于amd64架构的USB存储设备（闪存驱动器），并用于可从USB驱动器启动的计算机进行安装。它还支持启动到基于“livefs”的救援模式。没有预构建的软件包。

作为如何使用memstick映像的示例之一，假设USB驱动器显示为您机器上的/dev/da0，像这样应该可以工作：

```
# dd if=FreeBSD-13.3-RELEASE-amd64-disc1.iso \
    of=/dev/da0 bs=1m conv=sync
```

要确保正确获取目标（of=）。

仅启动

此支持使用CDROM驱动器启动计算机，但不包含用于从CD本身安装FreeBSD的安装分发集。您需要在从CD启动后执行基于网络的安装（例如从HTTP或FTP服务器）。

此外，可将其写入适用于amd64架构的USB存储设备（闪存驱动器），并用于可从USB驱动器启动的计算机进行安装。它还支持启动到基于“livefs”的救援模式。没有预构建的软件包。

作为如何使用memstick映像的示例之一，假设USB驱动器显示为您机器上的/dev/da0，像这样应该可以工作：

```
# dd if=FreeBSD-13.3-RELEASE-amd64-bootonly.iso \
    of=/dev/da0 bs=1m conv=sync
```

要小心确保你正确获取目标（of=）。

记忆棒

这可以写入USB存储器（闪存驱动器），并用于能够从USB驱动器引导的机器上进行安装。它还支持引导到基于"livefs"的救援模式。没有预构建的软件包。

作为如何使用记忆棒镜像的一个示例，假设USB驱动器在您的机器上显示为/dev/da0，像这样应该可以工作：

```
# dd if=FreeBSD-13.3-RELEASE-amd64-memstick.img \
    of=/dev/da0 bs=1m conv=sync
```

要小心确保你正确获取目标（of=）。

迷你记忆棒

这可以写入USB存储器（闪存驱动器），并用于引导机器，但不包含介质本身上的安装分发集，类似于仅引导镜像。它还支持引导到基于"livefs"的救援模式。没有预构建的软件包。

作为如何使用迷你记忆棒镜像的一个示例，假设USB驱动器在您的机器上显示为/dev/da0，像这样应该可以工作：

```
# dd if=FreeBSD-13.3-RELEASE-amd64-mini-memstick.img \
    of=/dev/da0 bs=1m conv=sync
```

要小心确保你正确获取目标（of=）。

FreeBSD/arm SD卡镜像

这些可以写入SD卡并用于引导支持的arm系统。SD卡镜像包含完整的FreeBSD安装，并可以安装到至少512Mb的SD卡上。

对于那些无法访问系统控制台的方便起见，默认提供了一个具有密码`freebsd`的`freebsd`用户，用于`ssh(1)`访问。此外，`root`用户的密码设置为`root`，强烈建议在获得系统访问权限后为两个用户更改密码。

要将FreeBSD/arm镜像写入SD卡，使用`dd(1)`实用程序，将*KERNEL*替换为系统的适当内核配置名称。

```
# dd if=FreeBSD-13.3-RELEASE-arm-armv6-KERNEL.img \
    of=/dev/da0 bs=1m conv=sync
```

要小心确保你正确获取目标（of=）。

FreeBSD 13.3-RELEASE也可以从多家供应商购买DVD。将提供基于FreeBSD 13.3的产品的供应商之一是：

也可用于amd64（x86_64）、i386（x86_32）、AArch64（arm64）和RISCV（riscv64）架构的预安装虚拟机映像，以及QCOW2、VHD和VMDK磁盘映像格式的原始（未格式化）映像。

在这些云托管平台上也提供FreeBSD 13.3-RELEASE amd64：

```
  ap-south-2 region: ami-089060a45633ba665
  ap-south-1 region: ami-02dcc1601d891f579
  eu-south-1 region: ami-096876ff505a88735
  eu-south-2 region: ami-0f79768daa8a3ba50
  me-central-1 region: ami-02d3f18699b9044e8
  ca-central-1 region: ami-0ce4529f93eafe727
  eu-central-1 region: ami-0164778b543e5eb63
  eu-central-2 region: ami-0175ef9bad67508c5
  us-west-1 region: ami-08224a1938d8317b7
  us-west-2 region: ami-0249e4148b539d9eb
  af-south-1 region: ami-06779d1b6ac99d8a3
  eu-north-1 region: ami-0eecde4f1b297b2f4
  eu-west-3 region: ami-0e9aecbeff62a6964
  eu-west-2 region: ami-080e8e554b9e7f71a
  eu-west-1 region: ami-0a9749ae5c79baeee
  ap-northeast-3 region: ami-028e5798c648ba909
  ap-northeast-2 region: ami-04597ef6da3c0ecc8
  me-south-1 region: ami-0a84a1af1dfd0d260
  ap-northeast-1 region: ami-031c3791531713063
  sa-east-1 region: ami-08ec9cb0dbb7e607b
  ap-east-1 region: ami-0e175b993757d40e1
  ap-southeast-1 region: ami-074e797559eee5372
  ap-southeast-2 region: ami-09575879989825e87
  ap-southeast-3 region: ami-060d04faa6c867a5c
  us-east-1 region: ami-0dd9c166b4640d88d
  us-east-2 region: ami-027cde133a99701ec
```

可以从每个区域的系统管理器参数存储中检索这些AMI ID，使用以下键：

```
/aws/service/freebsd/amd64/base/ufs/13.3/RELEASE
```

FreeBSD/arm64 Amazon® EC2™：

AMI在以下区域可用：

```
  ap-south-2 region: ami-039f1df5455411b20
  ap-south-1 region: ami-0936dfeedfb80ca34
  eu-south-1 region: ami-01bfd433755596cf7
  eu-south-2 region: ami-018a8a15d1142acf9
  me-central-1 region: ami-087714379c015bce7
  ca-central-1 region: ami-09457792921858205
  eu-central-1 region: ami-010f1ca390ff93032
  eu-central-2 region: ami-09177d81d898f4019
  us-west-1 region: ami-0e8701784b2fc0612
  us-west-2 region: ami-0111db642946be5d5
  af-south-1 region: ami-0a21399a27229150f
  eu-north-1 region: ami-011ec12eea0adafdd
  eu-west-3 region: ami-0f6906e72786bb4a5
  eu-west-2 region: ami-071c9be1f0abbb967
  eu-west-1 region: ami-03e0415f3cb413977
  ap-northeast-3 region: ami-0c9e7434790d501cd
  ap-northeast-2 region: ami-0495294876a8fd7fd
  me-south-1 region: ami-031acbfed308316a7
  ap-northeast-1 region: ami-06a4f41d6812e3bd7
  sa-east-1 region: ami-035f731b7d29f80ef
  ap-east-1 region: ami-0773ec8775ce85f82
  ap-southeast-1 region: ami-08cd1a98b1595a824
  ap-southeast-2 region: ami-076121bdf01ee8cfe
  ap-southeast-3 region: ami-0f54e915d216d5f2c
  us-east-1 region: ami-011496bffb848eee0
  us-east-2 region: ami-0d3d876c2693efbdd
```

可以从每个区域的系统管理器参数存储中检索这些AMI ID，使用以下键：

```
/aws/service/freebsd/arm64/base/ufs/13.3/RELEASE
```

```
      % gcloud compute instances create INSTANCE \
        --image freebsd-13-3-release-amd64 \
        --image-project=freebsd-org-cloud-dev
      % gcloud compute ssh INSTANCE
```

+   Microsoft® Azure™：

    一旦它们完成发布的认证，FreeBSD虚拟机映像将在Azure市场上提供。

+   Hashicorp/Atlas® Vagrant™：

    可以使用`vagrant`实用程序部署实例：

```
      % vagrant init freebsd/FreeBSD-13.3-RELEASE
      % vagrant up
```
