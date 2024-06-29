<!--yml

分类：未分类

日期：2024-05-27 15:05:35

-->

# 如何将 Raspberry Pi 5 超频到 3 GHz 以上！| 汤姆硬件

> 来源：[https://www.tomshardware.com/raspberry-pi/how-to-overclock-the-raspberry-pi-5-beyond-3-ghz](https://www.tomshardware.com/raspberry-pi/how-to-overclock-the-raspberry-pi-5-beyond-3-ghz)

[Raspberry Pi 5](https://www.tomshardware.com/reviews/raspberry-pi-5) 众所周知，它是目前最快的 [Raspberry Pi](https://www.tomshardware.com/raspberry-pi)，毕竟它是新的旗舰产品。最初我们成功将 Raspberry Pi 5 超频到了 3 GHz，比默认的 2.4 GHz 有了显著提升。虽然有人声称他们将其超频到更高的速度，但我们经过与树莓派的确认，3 GHz 是极限。

好吧，曾经是这样，但是一个新的 [固件](https://github.com/raspberrypi/firmware/issues/1876) 似乎打破了这个速度限制。我们能否让 Raspberry Pi 运行得更快？答案是可以，但具体能达到多快还要看“硅神”的心情，因此效果可能有所不同。

在我们的测试中，我们成功稳定地将频率超频到了 3.1 GHz。当我们尝试达到 3.2 GHz 时，在运行 Geekbench 测试时出现了稳定性问题。继续增加到 3.3 GHz，系统经常比正常运行的次数还要多。

YouTuber 和 Raspberry Pi 专家 Jeff Geerling 成功将频率超频到了 3.14 GHz，但是超过这个速度后就失败了。

你的 Raspberry Pi 5 能跑多快？如果你愿意尝试，请记住，如果你搞坏了它，就得自己买单。

## 对于这个项目，你将需要

+   [Raspberry Pi 5](https://www.tomshardware.com/reviews/raspberry-pi-5)

+   主动冷却解决方案

+   最新的 Raspberry Pi OS 安装在 micro SD 卡上

+   备用 micro SD 卡

+   运行 Windows、MacOS 或 Linux 的计算机

在我们尝试超频之前，我们需要一个良好的冷却系统。至少你需要官方的 Raspberry Pi 主动散热器、Argon 的 THRML 主动散热器，或者来自 EDATEC 等的被动散热器。不要在没有散热的情况下尝试这个过程，否则可能会损坏你的 Raspberry Pi。如果你搞坏了你的 Raspberry Pi 5，汤姆硬件概不负责。

我们选择 Argon THRML 60-RC 作为散热器，仅仅因为它价格为 $20，而且它是一款非常强大的散热器。你也可以选择 52Pi 的水冷套件，但是以 $120 的价格只能多提供几度的冷却效果，与 $100 的差价相比。

获取汤姆硬件（Tom's Hardware）的最新新闻和深度评测，直接发送到您的收件箱。

1\. 在你的 Windows / Apple / Linux PC 上，[**下载这个实验性固件**](https://github.com/raspberrypi/firmware/files/14604820/rpi-eeprom-recovery.zip)**。** 请注意，根据树莓派的 Alasdair Allan 在 Mastodon 上的消息，这个固件不被推荐使用，使用时需自行承担风险。

2\. **下载、安装并打开 Raspberry Pi Imager。**

3\. **对于 Raspberry Pi 设备选择“无过滤”，** 然后 **对于操作系统选择“使用自定义”。** 然后对于存储 **选择你的 micro SD 卡并点击下一步** 开始写入过程。

（图片来源：Tom's Hardware）

4\. **当提示时弹出卡片，并将其插入 Raspberry Pi 5。** 确保你的 Raspberry Pi 5 关机，并连接了键盘、鼠标和屏幕。

5\. **开启 Raspberry Pi 5，等待屏幕变绿，然后关闭电源**并**取出 micro SD 卡。**

6\. **插入你的 Raspberry Pi OS micro SD 卡**，然后**启动 Pi 到桌面。** 你需要 [最佳的 Raspberry Pi microSD 卡](https://www.tomshardware.com/best-picks/raspberry-pi-microsd-cards) 来给你的 Pi 提速。

7\. **更新可用的软件库**，然后**升级你的 Raspberry Pi 5**。这将确保我们拥有最新的软件可用。这不是必须的，但始终保持你的 Raspberry Pi 更新是明智的。

```
sudo apt update && sudo apt dist-upgrade
```

8\. **打开 config.txt 进行编辑**。它位于 /boot 目录中。

```
sudo nano /boot/config.txt
```

9\. **在文件底部添加一行，并添加这些行** 来将 CPU 超频到 3.1 GHz。

```
arm_freq=3100
```

10\. **一个可选的步骤。使用 force turbo 将 CPU 和 GPU 运行到最大速度。** 这会覆盖任何缩放管理器，并使 CPU 和 GPU 运行在 100%。主动冷却对于此操作的正常工作至关重要。

```
force_turbo=1
```

11\. **添加另一行以增加 CPU 的电压**。这会替换 over_voltage=X 以为 Pi 提供额外电压的方法。Delta 方法在当前电压水平上增加电压，在这里我们使用 50000 来增加 0.05V。旧的 over_voltage 方法已被弃用于 Raspberry Pi 5。

```
over_voltage_delta=50000
```

12\. **按 CTRL + X，Y 然后 ENTER 保存文件。**

13\. **重新启动 Raspberry Pi 5。** 如果 Raspberry Pi 启动失败，请关闭电源。按住空格键。这将绕过超频配置并使用默认配置启动 Pi。

14\. **打开一个新的终端并使用此命令** **来查看 Pi 的当前 CPU 速度**。按 CTRL + C 停止读取 CPU 速度。

```
watch -n 1 vcgencmd measure_clock arm
```

你的 Raspberry Pi 现在已经超频到 3.1 GHz！如果你足够勇敢，想要进一步推动它，我们向你致敬，不要忘记发布 [Geekbench](https://browser.geekbench.com/user/biglesp) 分数！试试我们的 [压力测试基准工具](https://www.tomshardware.com/how-to/raspberry-pi-benchmark-vcgencmd)，它会记录你的 CPU 温度到一个可以用来制图的 CSV 文件中。
