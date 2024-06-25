<!--yml

类别：未分类

日期：2024-05-27 15:22:26

-->

# 鼠标变成相机 | Hackaday

> 来源：[`hackaday.com/2024/01/29/a-mouse-becomes-a-camera/`](https://hackaday.com/2024/01/29/a-mouse-becomes-a-camera/)

如果你的指针设备是鼠标，把它翻过来。如果你不是非常老派，你的鼠标有一个球，那么很可能会看到一个红色的 LED 灯，这个灯充当一个非常简单的摄像头传感器的照明。鼠标电子设备通过查找图像中的运动来完成它们的工作，但是可以提取数据并重新利用传感器作为数字相机。[Doctor Volt]在一个新视频中展示了这一点，展示了一个罗技外设的内部构造。

鼠标包含一个微控制器和相机部分，幸运的是它有一个 SPI 接口。推断出查询传感器信息的正确寄存器，并且像魔术一样，一幅图像出现了。一个 M12 镜头提供了焦距，带有一个方便的 3D 打印支架，板子重新放回鼠标外壳作为一个外壳。这些图片有点像 Game Boy 相机，低分辨率和单色，但仍然是一个很棒的黑客。

如果你想尝试一下，[你可以在 GitHub 存储库中找到代码](https://github.com/michalin/mousecam)。[你可能会发现值得找一个游戏鼠标](https://hackaday.com/2022/05/16/gaming-mouse-becomes-digital-camera/)，因为分辨率更高的传感器。

[`www.youtube.com/embed/qAlpt_XYkXI?feature=oembed`](https://www.youtube.com/embed/qAlpt_XYkXI?feature=oembed)

视频
