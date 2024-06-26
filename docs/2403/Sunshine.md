<!--yml

category: 未分类

date: 2024-05-29 12:50:30

-->

# 阳光

> 来源：[https://toaster.llc/blog/sunshine/index.html](https://toaster.llc/blog/sunshine/index.html)

<center-column>从零到光子：

阳光

2024年3月21日

在制作相机的过程中，不可避免地会拍摄很多照片。在使用[Photon](/photon)捕捉图像时，我发现了两个与阳光相关的现象，这让我感到惊讶和有趣。

* * *

## 透明度

第一个现象发生在我访问父亲时的肯塔基州。

在我童年家中的客厅工作时，我从一个原型中直播图像。我拿起原型，PCB的背面捕捉到一缕阳光透过窗户照射。图像立即变换 —— 它不仅显示了客厅，还显示了客厅中叠加的布线迹象：

我非常确定在成像流水线中没有意外实现线迹渲染器...

起初，我以为这些是光子PCB上的金属线迹：

**左侧**

：PCB布局；外部矩形是图像传感器IC的边界。内部矩形是图像传感器的像素阵列区域。

**右侧**：用显微镜观察实际的PCB。

但仔细检查显示PCB上的迹线与神秘的迹线不匹配：

不匹配！

**左侧**：PCB布局；缩放并裁剪至图像传感器像素阵列的边界。

**右侧**：神秘迹线；在盖住镜头的情况下捕捉，以使唯一进入图像传感器的光来自其背面。

如果神秘的迹线不是PCB线路，那它们是什么？

当我回到圣何塞时，我能够在显微镜下查看备用图像传感器的背面：

备用图像传感器的背面

增强：

中了！

**左侧**：图像传感器的背面；去饱和并裁剪至图像传感器像素阵列的边界。

**右侧**：相同的神秘迹线。

完全匹配！神秘的迹线是图像传感器背面的BGA电路。

这种现象似乎对紫外光谱中的光特别敏感。例如，我家客厅开着窗户，仅仅是环境中的紫外线就触发了一个没有背板的设备：

</uv-traces-living-room.06918f22.mp4>

当打开窗户时，即使设备不直接受到阳光照射，迹线也会出现。

注意迹线的明显紫色色调，暗示着紫外线光谱。

相比于可见光谱，我猜紫外线由于其较高的能量，更容易穿透PCB的FR-4。

幸运的是，Photon有铝外壳，可以防止这种现象在实地发生；如果Photon有塑料外壳，则需要光屏蔽。

* * *

## 宇宙故障

第二个现象发生在圣何塞，当我从Photon原型中直播图像时。我一走出门测试在户外拍摄图像时，直播图像停止了。

可能是我的USB堆栈出现了错误，我回到室内并恢复了图像流，但当我立即走到户外时，它再次发生了。尽管当时情况看起来很奇怪，但结果是一致的：户外因素导致原型出现故障。

经过几次实验证实：当原型在户外但处于阴凉处时，问题没有出现；用绝缘胶带包裹电路板也能避免问题出现。（这是在我用金属外壳测试之前的情况。）

为了将问题缩小到特定组件，我制作了一个铝箔掩模，并将其贴在厨房桌子的一侧，然后配置计算机，在发生故障时发出蜂鸣声。现在我可以移动电路板来直接微弱的阳光：

</glitches.0675a26c.mov>

阳光轻易地确定了两个受日光干扰的芯片：一个16 MHz的时钟芯片和一个3.3V的LDO芯片。

启用音频，以便听到故障发生时的声音。

此策略明确了受影响的组件，它们都有一个共同点：[芯片级封装！](https://en.wikipedia.org/wiki/Chip-scale_package) 这些芯片没有用黑色塑料包裹，光线可以进入硅片并干扰其正常操作。

光子显然是一个出色的日光检测器！</center-column>
