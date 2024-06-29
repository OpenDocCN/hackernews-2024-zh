<!--yml

category: 未分类

date: 2024-05-27 13:14:42

-->

# usbredir

> 来源：[https://www.spice-space.org/usbredir.html](https://www.spice-space.org/usbredir.html)

# usbredir

usbredir 是用于通过网络连接发送 USB 设备流量的网络协议的名称。它也是提供解析库、usbredirhost 库和几个实现该协议的实用程序的软件包的名称。

协议的文档在[这里](https://gitlab.freedesktop.org/spice/usbredir/-/blob/main/docs/usb-redirection-protocol.md)，该文档还解释了 usbhost 和 usbguest 的含义。

usbredir 是为与 Spice 配合而创建的，这也是它托管在 spice-space.org 上的原因，但协议和 usbredirhost 完全独立于 Spice，例如它们也可以用于创建将 USB 设备通过 VNC 连接重定向到 qemu 虚拟机的 VNC 扩展。

usbguest 目前仅为 qemu 实现，并作为 qemu 的一部分提供（在构建 qemu 时，需要可用的 libusbredirparser 库来启用此功能）。

usbredir 支持通过过滤字符串可配置的 USB 设备过滤。

## 主机配置

为了使重定向正常工作，虚拟机必须有支持的 USB 控制器。Qemu 支持 USB2 和 USB3，但在撰写本文时，USB3 的测试较少。对于 USB2 的支持，客户机必须有 EHCI 控制器和伴随的 UHCI 控制器（伴随的 UHCI 控制器还需要支持 USB1.x 设备）。对于 USB3 的支持，需要 XHCI 控制器。它还需要有 Spice 通道用于 USB 重定向。此类通道的数量决定了同时能够重定向的 USB 设备数量。

有关 qemu 中 USB 控制器的更多信息可以在[这里](http://git.qemu-project.org/?p=qemu.git;a=blob_plain;f=docs/usb2.txt;hb=HEAD)找到。

### 使用 virt-manager

使用 virt-manager 创建的虚拟机默认应该具有 USB 控制器。在虚拟机详情中，选择左侧窗格中的“控制器 USB”。如果只需支持 USB2 设备，请确保其模型设置为“USB2”。要支持 USB 3.0，请选择模型类型为“USB3”。

您可以然后单击“添加硬件”，并添加“USB 重定向”项目，设置您想要同时能够重定向的 USB 设备数量。

### 使用 libvirt

以下 libvirt XML 将配置一个支持 USB2 并能够同时重定向 3 个设备的客户机：

```
<controller  type='usb'  index='0'  model='ich9-ehci1'/>
<controller  type='usb'  index='0'  model='ich9-uhci1'>
  <master  startport='0'/>
</controller>
<controller  type='usb'  index='0'  model='ich9-uhci2'>
  <master  startport='2'/>
</controller>
<controller  type='usb'  index='0'  model='ich9-uhci3'>
  <master  startport='4'/>
</controller>
<redirdev  bus='usb'  type='spicevmc'/>
<redirdev  bus='usb'  type='spicevmc'/>
<redirdev  bus='usb'  type='spicevmc'/> 
```

对于 USB3 的支持，配置可以简化为：

```
<controller  type='usb'  index='0'  model='nec-xhci'/>
<redirdev  bus='usb'  type='spicevmc'/>
<redirdev  bus='usb'  type='spicevmc'/>
<redirdev  bus='usb'  type='spicevmc'/> 
```

### 使用 QEMU

以下 qemu 选项将配置一个支持 USB2 并能够同时重定向 3 个设备的客户机

```
-device  ich9-usb-ehci1,id=usb  \
-device  ich9-usb-uhci1,masterbus=usb.0,firstport=0,multifunction=on  \
-device  ich9-usb-uhci2,masterbus=usb.0,firstport=2  \
-device  ich9-usb-uhci3,masterbus=usb.0,firstport=4  \
-chardev  spicevmc,name=usbredir,id=usbredirchardev1  \
-device  usb-redir,chardev=usbredirchardev1,id=usbredirdev1  \
-chardev  spicevmc,name=usbredir,id=usbredirchardev2  \
-device  usb-redir,chardev=usbredirchardev2,id=usbredirdev2  \
-chardev  spicevmc,name=usbredir,id=usbredirchardev3  \
-device  usb-redir,chardev=usbredirchardev3,id=usbredirdev3 
```

对于 USB3 的支持，配置可以简化为：

```
-device  nec-usb-xhci,id=usb  \
-chardev  spicevmc,name=usbredir,id=usbredirchardev1  \
-device  usb-redir,chardev=usbredirchardev1,id=usbredirdev1  \
-chardev  spicevmc,name=usbredir,id=usbredirchardev2  \
-device  usb-redir,chardev=usbredirchardev2,id=usbredirdev2  \
-chardev  spicevmc,name=usbredir,id=usbredirchardev3  \
-device  usb-redir,chardev=usbredirchardev3,id=usbredirdev3 
```

## 客户端配置

客户端需要支持USB重定向。在远程查看器中，一旦建立Spice连接，您可以在“文件->USB设备选择”菜单下选择要重定向的USB设备。还有各种命令行重定向选项，下面将对其进行描述，并在使用--help-spice时运行远程查看器。

要使Windows客户端上的USB重定向工作，您需要安装[UsbDk](/download/windows/usbdk/)。

## 过滤器字符串格式

使用一个过滤器来设置USB设备的阻止或自动连接规则。它由一个或多个规则组成，其中每个规则的形式为：

**<类别>,<厂商>,<产品>,<版本>,<允许>**

使用-1作为类别/厂商/产品/版本的值以接受任何值。

规则本身以以下方式连接：

**规则1|规则2|规则3**

客户端的默认自动连接过滤器字符串是0x03,-1,-1,-1,0|-1,-1,-1,-1,1

这将过滤掉HID（类0x03）USB设备的自动连接，并自动连接其他任何设备。请注意，在末尾显式添加允许规则是必要的，因为默认情况下，所有没有匹配过滤规则的设备都不会自动连接。

## 客户端过滤

设置一个字符串来指定一个过滤器，以确定当插入时哪些USB设备被允许/阻止将USB流量**自动重定向**到客户端（客户端窗口必须处于焦点状态）。

```
remote-viewer  --spice-usbredir-auto-redirect-filter="0x03,-1,-1,-1,0|-1,-1,-1,-1,1" 
```

设置一个字符串来指定一个过滤器，以确定哪些USB设备，已经插入，将在建立Spice连接后**连接时重定向**。

```
remote-viewer  --spice-usbredir-redirect-on-connect="0x03,-1,-1,-1,0|-1,-1,-1,-1,1" 
```

## 主机过滤

设置一个字符串来指定一个过滤器，以确定哪些USB设备被**允许/阻止**将USB流量重定向到客户端。

### 使用QEMU

```
-device  usb-redir,filter='0x03:-1:-1:-1:0|-1:-1:-1:-1:1',chardev=usbredirchardev1,id=usbredirdev1 
```

请注意，在QEMU命令中，过滤器字符串应该使用**':'**字符作为规则内的分隔符。

### 使用libvirt

```
...
<devices>
  ...
  <redirfilter>
  <usbdev  class='0x08'  vendor='0x1234'  product='0xbeef'  version='2.56'  allow='yes'/>
  <usbdev  allow='no'/>
  </redirfilter>
</devices>
... 
```

## 下载

最新版本：[usbredir-0.14.0.tar.xz](/download/usbredir/usbredir-0.14.0.tar.xz)

较旧版本：[0.13](/download/usbredir/usbredir-0.13.0.tar.xz)，[0.12](/download/usbredir/usbredir-0.12.0.tar.xz)，[0.11](/download/usbredir/usbredir-0.11.0.tar.xz)，[0.10](/download/usbredir/usbredir-0.10.0.tar.xz)，[0.9](/download/usbredir/usbredir-0.9.0.tar.xz)，[0.8](/download/usbredir/usbredir-0.8.0.tar.bz2)，[0.7](/download/usbredir/usbredir-0.7.tar.bz2)，[0.6](/download/usbredir/usbredir-0.6.tar.bz2)，[0.5.2](/download/usbredir/usbredir-0.5.2.tar.bz2)，[0.5.1](/download/usbredir/usbredir-0.5.1.tar.bz2)，[0.5](/download/usbredir/usbredir-0.5.tar.bz2)，[0.4.3](/download/usbredir/usbredir-0.4.3.tar.bz2)，[0.4.2](/download/usbredir/usbredir-0.4.2.tar.bz2)，[0.4.1](/download/usbredir/usbredir-0.4.1.tar.bz2)，[0.4](/download/usbredir/usbredir-0.4.tar.bz2)，[0.3.3](/download/usbredir/usbredir-0.3.3.tar.bz2)，[0.3.2](/download/usbredir/usbredir-0.3.2.tar.bz2)，[0.3.1](/download/usbredir/usbredir-0.3.1.tar.bz2)，[0.3](/download/usbredir/usbredir-0.3.tar.bz2)

### 源代码

您可以在这里找到官方usbredir git仓库的链接：[here](https://gitlab.freedesktop.org/spice/usbredir)。
