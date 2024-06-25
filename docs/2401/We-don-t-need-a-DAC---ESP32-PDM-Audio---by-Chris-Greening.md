<!--yml

类别：未分类

日期：2024-05-27 14:31:32

-->

# 我们不需要 DAC - 由 Chris Greening 的 ESP32 PDM 音频

> 来源：[`atomic14.substack.com/p/esp32-s3-no-dac`](https://atomic14.substack.com/p/esp32-s3-no-dac)

所以，在 ESP32-S3 上没有 DAC。

如果你想要将音频输出并使用模拟放大器，你可能会认为这有点令人沮丧。

但是，使用 ESP32 上的 Sigma Delta 调制来输出**脉冲密度调制**音频实际上非常简单，你可以通过低通滤波器来恢复音频信号 - RC 滤波器就足够了。这也是我在视频中使用的方法。

尽管 Espressif 的文档确实建议使用更好的主动滤波器。

在频域中查看 PDM 信号并将其显示出来确实很有趣。以下是一段音频及其频谱图：

这是原始音频的模拟 PDM 版本。PDM 数据的采样率略高于 1MHz，我显示了从 0 到 500KHz 的频谱图

PDM 数据的范围从 -1 到 1，并根据原始信号的值改变密度。

如果我们只看频谱的较低部分，我们可以看到看起来像是我们的原始信号。

因此，要恢复原始音频，我们只需应用一个低通滤波器 - 哈喽，我们的原始音频信号就回来了！

如何在 ESP32 上实现这一点呢？

我们有几个选择可供选择。

[示例代码](https://github.com/espressif/esp-idf/tree/b4268c874a4cf8fcf7c0c4153cffb76ad2ddda4e/examples/peripherals/sigma_delta/sdm_dac)来自 Espressif 的示例代码建议使用定时器来使用 **sigmadelta_set_duty** 输出每个样本（你也可以使用普通的 PWM）。这对音频数据确实有效，我有一些简单的[样本代码](https://github.com/atomic14/esp32-pdm-audio/blob/main/src/audio_output/PDMTimerOuput.cpp)可以实现这一点，但效率不高 - 我们不断被定时器中断以发送下一个样本。如果你想要从其他来源流式传输样本，那么需要相当多的代码。

更好的方法是使用 I2S 外设，它也可以输出 PDM 数据。但其中有两件令人讨厌的事情，这在文档中的时序图中有所体现。

第一个问题是它总是要输出左右声道。如果我们只想将 PDM 信号直接馈送到模拟音频放大器或耳机中，这有点尴尬。但我们可以通过为左右声道输出相同的值来解决这个问题。

第二个问题是它总是要输出一个时钟信号 - 我们其实不需要这个。我对此的解决方法是将时钟分配给 IO45 或 IO46 - 在 S3 上，你不能真的为这些引脚做太多事情，因为它们是针对引线的引脚，最好是让它们保持不变。但一旦 ESP32 启动了，你可以将它们用于输出。

有“合适”的 PDM 放大器 IC 可以接收此信号 - 例如[ MAX98358 ](https://www.analog.com/media/en/technical-documentation/data-sheets/max98358.pdf)或[ SSM2537 ](https://www.analog.com/media/en/technical-documentation/data-sheets/SSM2537.pdf)。

这一切都出奇的顺利，你可以直接从 PDM 信号驱动耳机，大多数模拟放大器都会接受合并的立体声 PDM 信号，并且带宽足够低，它们就可以正常工作。

你甚至可以只用一个非常简单的半桥或全桥驱动扬声器，并获得合理的音频输出（尽管可能会有很多噪音）。

观看视频并告诉我你的想法。
