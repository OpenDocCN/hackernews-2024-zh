<!--yml

category: 未分类

date: 2024-05-27 14:48:28

-->

# **Etherify：通过 Raspberry Pi 以太网 RF 泄漏传输摩尔斯电码**

> 来源：[`www.rtl-sdr.com/etherify-transmitting-morse-code-via-raspberry-pi-ethernet-rf-leakage/`](https://www.rtl-sdr.com/etherify-transmitting-morse-code-via-raspberry-pi-ethernet-rf-leakage/)

2020 年 11 月 10 日

# **Etherify：通过 Raspberry Pi 以太网 RF 泄漏传输摩尔斯电码**

在他的博客上，SQ5BPF 一直在记录一个 TEMPEST 实验，他已经能够[通过树莓派的以太网连接泄漏的 RF 传输数据](https://lipkowski.com/etherify/)。这个想法诞生于他发现他的树莓派 4 从以太网电缆泄漏出强烈的 125 MHz 的 RF 信号。他继续发现，通过使用 "ethtool" 命令行工具简单地改变以太网链接速度，很容易将一个音调打开和关闭。一旦知道了这一点，创建一些摩尔斯电码的 bash 脚本就是一件简单的事情了。

令人惊讶的是，以太网 RF 泄漏非常强。当树莓派距离 10 米远，中间隔着一堵钢筋混凝土墙时，SQ5BPF 能够通过连接到 PC 的 RTL-SDR 接收到生成的摩尔斯电码。进一步的实验表明，使用 Yagi 天线，他能够在 100 米之外接收到信号。

他的[文章](https://lipkowski.com/etherify/)解释了一些关于数据突发的进一步实验，并提供了他创建的脚本的链接，这样你就可以在家里尝试这个。

更新 - SQ5BPF 还指出以下内容：

> 使用的硬件不同，泄漏程度也不同。树莓派 4 是例外，也可以快速切换链接速度，因此是一个不错的演示候选者，但其他硬件也可以工作。
> 
> 最初的测试是在我周围的一些旧笔记本电脑上进行的，它们也泄漏了。也许有一天我会公布这个，但是它们每个都有不同的行为。
