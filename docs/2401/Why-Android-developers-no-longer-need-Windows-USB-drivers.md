<!--yml
category: 未分类
date: 2024-05-27 14:27:22
-->

# Why Android developers no longer need Windows USB drivers

> 来源：[https://fabiensanglard.net/android_windows_driver/](https://fabiensanglard.net/android_windows_driver/)

December 30, 2023

Why Android developers no longer need Windows USB drivers

* * *

In the early days of the platform, Android developers working from Linux and Mac OS X connected their device via an USB cable and were good to go. Windows users had to find and then install drivers. That was super-annoying.

Nowadays there is no more need for drivers. What happened?

USB Driver/OS 101

* * *

When you plug an USB device into a port, the operating system needs to load drivers for its interface(s). This is done by inspecting the USB Descriptors hierarchy.

```
Device Descriptor
├─ Configuration Descriptor
│         └─ Interface Descriptor
│                 └─ Endpoint_Descriptors
├─ String Descriptors
└─ Device Qualifier Descriptor      

```

Oftentimes, reading the Device Descriptor [Vendor ID (VPI) and Product ID (PID) fields](https://www.keil.com/pack/doc/mw/USB/html/_u_s_b__device__descriptor.html)[^([1])](#footnote_1) is enough to decide which drivers should be loaded.

All three major OSs ship with "default" Device Class drivers so most USB devices work auto-magically[^([2])](#footnote_2). If I connect my ErgoDox EZ keyboard, Windows attaches `hidusb.sys.` to it[^([3])](#footnote_3) and I don't need a driver.

If Windows cannot find a built-in Device Class driver (or if they are better matches), it loads user-installed drivers. e.g. an Apple Magic Trackpad will also get `hidusb.sys` unless Bingxing Wang's [awesome driver](https://github.com/imbushuo/mac-precision-touchpad/tree/master) is installed, in which case Windows loads `AmtPtpDeviceUm.dll` (with support for right-click and multi-finger gestures).

Why it did not work

* * *

If no driver can be found, Linux loads `usbfs` which lets user-space programs access the device. Mac OS does the same, with `IOKit`. Windows reports an error. No driver is loaded and the Android device cannot be accessed.

Why it worked with an android driver

* * *

To understand what is going on when a driver is installed, let's inspect the "brain" from [Google USB Driver](usb_driver_r13-windows.zip), `android_winusb.inf`.

```
;Google Nexus One
%SingleAdbInterface%        = USB_Install, USB\VID_18D1&PID_4E11

[USB_Install]
Include = winusb.inf
Needs   = WINUSB.NT

```

In `INF` parlance, it instructs "When VID=0x18D1 and PID=0x4E11 (a.k.a: a Google Nexus One), load `winusb` driver".

What is WinUSB? Alike Linux's `usbfs` and Mac OS's `IOKit` it is an USB driver which allows user-space programs to enumerate interfaces and write/read into/from the endpoints[^([4])](#footnote_4). That is what `adb` (the [Android Debug Bridge](https://en.wikipedia.org/wiki/Android_Debug_Bridge)) uses to chat with an Android device.

Why Android devices no longer need a windows driver

* * *

The method previously described has an obvious flaw. If an Android device VID/PID are not listed, `winusb` is not loaded.

Windows 8 has a better way than `INF` files to discover which driver an interface needs. It asks directly to the device!

When a device is connected, the OS issues a String Descriptors request at index `0xEE`. If the device is Microsoft OS Descriptors (MOD) compatible, it returns the string `M\0S\0F\0T\01\000\0\0`. In this case, Windows requests the [Extended Compat ID OS Feature Descriptor](https://learn.microsoft.com/en-us/windows-hardware/drivers/usbcon/microsoft-defined-usb-descriptors).

We can inspect this descriptor on a Pixel 6 with `libusb`'s `xusb`.

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

This device was set up with both Media Transfer Protocol and developer mode enabled. For each of these interfaces, the Feature Descriptor instructs which driver it needs. Respectively `mtp.sys` for the first interface and `winusb.sys` on the second interface.

With `winusb.sys` loaded, user-space executables such as `adb` can open the device, claim the interface, and get the developer rolling. No need for drivers anymore!

Which devices support Microsoft OS Descriptors?

* * *

Josh Gao (@jmga, adb developer at the time) pointed out that he introduced MOD support in Android 10 (API 29)[^([5])](#footnote_5). And he also wrote a CTS test for it so all Android devices shipping after API 29 should have it.

Extended Properties OS Feature Descriptor

* * *

More recent devices like the Pixel 8 have an Extended Properties OS Feature Descriptor. It can feature GUID, help pages, URLs, and even icons.

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

References

* * *

* * *

*