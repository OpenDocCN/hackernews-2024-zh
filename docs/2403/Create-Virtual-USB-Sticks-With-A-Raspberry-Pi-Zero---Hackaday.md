<!--yml

分类：未分类

日期：2024-05-27 14:46:41

-->

# 使用 Raspberry Pi Zero 创建虚拟 USB 棒 | Hackaday

> 来源：[https://hackaday.com/2024/03/09/create-virtual-usb-sticks-with-a-raspberry-pi-zero/](https://hackaday.com/2024/03/09/create-virtual-usb-sticks-with-a-raspberry-pi-zero/)

如今，从 USB 存储设备播放音乐文件是一个常见功能，并内置在 [Folkert van Heusden] 的 Opel Astra 的信息娱乐系统中。不幸的是，这种 USB 播放功能经常伴随着诸如音频编解码器等方面的各种限制，在 [Folkert] 的车辆中，限制为 1000 个文件。这促使他寻找 [另一种选择](https://vanheusden.com/electronics/virtual-usb/)，以避免在通勤时每周听到同样的歌曲的恐怖。解决方案是？使用 Linux 内核中的 Mass Storage Gadget（[MSG](https://www.kernel.org/doc/html/latest/usb/mass-storage.html)）驱动程序将 Raspberry Pi Zero 制作成虚拟 USB 大容量存储设备。

选择 USB 存储作为理想选择主要来自于信息娱乐系统的年代，该系统缺乏蓝牙，并且音频输入插孔噪声较大。当然，通过 MSG 驱动程序让 Raspberry Pi Zero 模拟成存储设备并不能解决文件限制的问题，因此编写了两个 Python 脚本来解决这个问题：一个从音乐文件夹创建镜像，另一个随机选择零的 SD 卡上可用的镜像并配置 MSG 驱动程序使用它。

至于未来改进的列表，包括将 RPi Zero 的 SD 卡挂载为只读以处理汽车关闭时的断电，以及由于使用回环设备，需要以 root 运行来创建镜像。作为概念验证，它似乎在正确的轨道上。

并非仅仅是老式信息娱乐系统可以享受到所有的乐趣。如果你的运气足够好，可以在仪表板上运行 Linux，你可能只需要一个 [Bash 脚本就能控制系统的自由](https://hackaday.com/2021/01/30/nissan-gives-up-root-shell-thanks-to-hacked-usb-drive/)。
