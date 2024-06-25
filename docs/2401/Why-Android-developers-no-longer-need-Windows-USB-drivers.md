<!--yml

类别：未分类

日期：2024-05-27 14:27:22

-->

# 为什么安卓开发者不再需要 Windows USB 驱动程序？

> 来源：[`fabiensanglard.net/android_windows_driver/`](https://fabiensanglard.net/android_windows_driver/)

2023 年 12 月 30 日

为什么安卓开发者不再需要 Windows USB 驱动程序？

* * *

在平台的早期阶段，使用 Linux 和 Mac OS X 的安卓开发者通过 USB 线连接设备就可以开始工作了。而 Windows 用户则必须寻找并安装驱动程序。那真是超级烦人。

如今不再需要驱动程序。发生了什么？

USB 驱动程序/操作系统 101

* * *

当你将 USB 设备插入端口时，操作系统需要为其接口加载驱动程序。这是通过检查 USB 描述符层次结构完成的。

```
Device Descriptor
├─ Configuration Descriptor
│         └─ Interface Descriptor
│                 └─ Endpoint_Descriptors
├─ String Descriptors
└─ Device Qualifier Descriptor      

```

通常，仅仅读取设备描述符的[厂商 ID（VPI）和产品 ID（PID）字段](https://www.keil.com/pack/doc/mw/USB/html/_u_s_b__device__descriptor.html)[^([1])](#footnote_1)就足以决定应加载哪些驱动程序。

三大主要操作系统都自带“默认”设备类驱动程序，因此大多数 USB 设备都能自动运行[^([2])](#footnote_2)。如果我连接我的 ErgoDox EZ 键盘，Windows 会将`hidusb.sys.`连接到它[^([3])](#footnote_3)，而我不需要驱动程序。

如果 Windows 找不到内置的设备类驱动程序（或者有更好的匹配项），它会加载用户安装的驱动程序。例如，如果安装了王冰星的[出色驱动程序](https://github.com/imbushuo/mac-precision-touchpad/tree/master)，则连接苹果魔术触控板时，Windows 会加载`AmtPtpDeviceUm.dll`（支持右键单击和多指手势）而不是`hidusb.sys`。

为什么它不起作用

* * *

如果找不到驱动程序，Linux 会加载`usbfs`，让用户空间程序访问设备。Mac OS 也是如此，使用`IOKit`。Windows 则报告错误。没有加载驱动程序，无法访问安卓设备。

为什么它使用安卓驱动程序

* * *

要理解驱动程序安装时发生的情况，让我们检查来自 Google USB 驱动程序的“大脑”，`android_winusb.inf`。

```
;Google Nexus One
%SingleAdbInterface%        = USB_Install, USB\VID_18D1&PID_4E11

[USB_Install]
Include = winusb.inf
Needs   = WINUSB.NT

```

在`INF`术语中，它指示“当 VID=0x18D1 且 PID=0x4E11（即：Google Nexus One）时，加载`winusb`驱动程序”。

什么是 WinUSB？类似于 Linux 的`usbfs`和 Mac OS 的`IOKit`，它是一种 USB 驱动程序，允许用户空间程序列举接口并读写端点[^([4])](#footnote_4)。这就是`adb`（[Android 调试桥](https://en.wikipedia.org/wiki/Android_Debug_Bridge)）用来与安卓设备通信的方式。

为什么安卓设备不再需要 Windows 驱动程序？

* * *

先前描述的方法存在一个明显的缺陷。如果安卓设备的 VID/PID 没有列出，`winusb` 就不会加载。

Windows 8 有比`INF`文件更好的方法来发现接口需要哪个驱动程序。它直接向设备询问！

当设备连接时，操作系统会发出一个索引为 `0xEE` 的字符串描述符请求。如果设备兼容 Microsoft OS 描述符（MOD），它将返回字符串 `M\0S\0F\0T\01\000\0\0`。在这种情况下，Windows 会请求 [扩展兼容性 ID OS 特征描述符](https://learn.microsoft.com/en-us/windows-hardware/drivers/usbcon/microsoft-defined-usb-descriptors)。

我们可以使用 `libusb` 的 `xusb` 在 Pixel 6 上检查此描述符。

```
$ ./xusb 18d1:4ee2

[...]

Reading string descriptors:
   String (0x01): "Google"
   String (0x02): "AOSP on Oriole"
   String (0x03): "0123456789ABCD"

Reading OS string descriptor:

Reading Extended Compat ID OS Feature Descriptor (wIndex = 0x0004):

  00000000  40 00 00 00 00 01 04 00 02 00 00 00 00 00 00 00  @...............
  00000010  00 01 4d 54 50 00 00 00 00 00 00 00 00 00 00 00  ..MTP...........
  00000020  00 00 00 00 00 00 00 00 01 01 57 49 4e 55 53 42  ..........WINUSB
  00000030  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................

```

该设备已设置为同时启用媒体传输协议和开发者模式。对于这两个接口，特征描述符指示需要哪个驱动程序。分别是第一个接口的 `mtp.sys` 和第二个接口的 `winusb.sys`。

使用加载了 `winusb.sys` 的用户空间可执行文件，如 `adb`，可以打开设备，声明接口，并使开发者投入使用。不再需要驱动程序！

哪些设备支持 Microsoft OS 描述符？

* * *

乔希·高（@jmga，当时是 adb 开发者）指出他在 Android 10（API 29）[^([5])](#footnote_5) 中引入了 MOD 支持。他还为此编写了一个 CTS 测试，因此 API 29 后的所有 Android 设备都应该具有该功能。

扩展属性 OS 特征描述符

* * *

像 Pixel 8 这样的较新设备具有扩展属性 OS 特征描述符。它可以包含 GUID、帮助页面、URL，甚至图标。

```
Reading Extended Properties OS Feature Descriptor (wIndex = 0x0005):

  00000000  8e 00 00 00 00 01 05 00 01 00 84 00 00 00 01 00  ................
  00000010  00 00 28 00 44 00 65 00 76 00 69 00 63 00 65 00  ..(.D.e.v.i.c.e.
  00000020  49 00 6e 00 74 00 65 00 72 00 66 00 61 00 63 00  I.n.t.e.r.f.a.c.
  00000030  65 00 47 00 55 00 49 00 44 00 00 00 4e 00 00 00  e.G.U.I.D...N...
  00000040  7b 00 46 00 37 00 32 00 46 00 45 00 30 00 44 00  {.F.7.2.F.E.0.D.
  00000050  34 00 2d 00 43 00 42 00 43 00 42 00 2d 00 34 00  4.-.C.B.C.B.-.4.
  00000060  30 00 37 00 44 00 2d 00 38 00 38 00 31 00 34 00  0.7.D.-.8.8.1.4.
  00000070  2d 00 39 00 45 00 44 00 36 00 37 00 33 00 44 00  -.9.E.D.6.7.3.D.
  00000080  30 00 44 00 44 00 36 00 42 00 7d 00 00 00        0.D.D.6.B.}...
```

参考资料

* * *

* * *

*
