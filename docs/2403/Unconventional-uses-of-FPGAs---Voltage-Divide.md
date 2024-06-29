<!--yml

category: 未分类

date: 2024-05-29 12:29:22

-->

# FPGAs的非传统用途 | 电压分割

> 来源：[https://voltagedivide.com/2024/03/18/unconventional-uses-of-fpgas/](https://voltagedivide.com/2024/03/18/unconventional-uses-of-fpgas/)

我在数字电路中最喜欢的现象是数字再次威胁变为模拟。例如，许多人对超频FPGA以获得更高性能的想法感兴趣，并且通常被鼓励不要这样做。然而，思考超频失败的原因并学习了解超频问题可以帮助您实现稳定的超频或至少帮助您更好地了解您的芯片。这只是在超越建议极限时探索FPGA奇怪行为的一个例子。

每个传感器都是温度传感器，几乎一切都可以通过努力成为电阻器或导体，如果您尝试足够努力，任何东西都可以成为天线。数据表仅仅是建议，最后，我们经常假装事物是理想的，当它们实际上并非如此时。

# 为什么？

能够理解常见建议背后的推理，偶尔在极端情况下推动极限，并通常知道可能性并可能尝试其中一些技巧将有助于您理解部件和工具的故障条件和实际极限，这将总体上使您成为一位更好的开发者。

我个人认为，常见建议不应仅仅被遵循，而是应理解并适用异常情况。

请注意，以下某些技术可能导致FPGA芯片损坏。您已经被警告。

# 背景

在一天结束时，您FPGA芯片中的数字电路实际上是模拟电路。当它们在自己的范围内运行时，它们会像理想的数字电路一样正常工作。通常您希望您的数字部件以数字方式运行，但一些模拟特性可能对您有用，正如我们后面将看到的。

## 工艺、电压和温度变化

每个硅芯片都表现出工艺、电压和温度的变化。

工艺变化与芯片制造相关的效果，这就是为什么FPGA通常以速度等级或速度档次出售，速度更快的芯片卖得更贵。用于超频的CPU可能被称为幸运样本或黄金芯片，以表示在将芯片推到极限时其卓越性能。在速度等级或产品类别内，芯片仍然表现出差异。速度档次只是不能捕捉到制造过程的所有可能结果。良好的样本可能表现出较低的漏电，导致较低的静态功耗，或者较低的传播延迟。

运行中的电压变化通常也会导致数字电路的传播延迟变化，当电压增加时，传播延迟减少。硅片的功耗也与电压的平方成正比，因此，电压的小幅增加可能导致功耗的显著增加，从而产生热量。当电压降低时，情况则相反。电子迁移随着电压的升高而增加，可能会导致介质击穿，这可能导致芯片的损坏。

温度变化通常也会导致传播延迟增加，因为温度升高。泄漏电流经常上升，导致在升高温度时功耗增加。

## 传播延迟

在任何数字电路中，由于寄生电感、电容和电阻，信号传播延迟是一种常见现象。这通常限制了电路的最大时钟速度。

传播延迟在FPGA综合、布置和布线工具中被建模和用于静态时间分析，以预测行为并确保在实际应用中的可靠功能。

电路中的传播延迟是数字电路中利用的模拟效应之一，用于实现诸如延迟线或环振荡器等复杂功能，我们将会看到。

### 组合逻辑故障

查找表（LUTs）和布线中的传播延迟可以产生非常短的脉冲，称为电路中的故障。这些脉冲的持续时间可以达到几十到几百皮秒。通常，这些故障是不希望的，会导致FPGA设计中功耗增加。然而，高速脉冲可以在波联合时间到数字转换器中使用，我们将会看到。

## [亚稳态](链接)

FPGA中边沿敏感型触发器的亚稳态是一个问题，当触发器的设置保持时序不被尊重时。亚稳态发生时，触发器可能会随机设定为1或0输出，而不考虑其输入值。这通常是需要避免的问题，特别是在不同时钟域之间切换时。这通常是不希望的行为，必须避免以确保数字电路的可靠运行，也可能在超频或采样非传统FPGA电路中发生。在某些情况下，它可以作为随机数生成器中的熵源。

## 超频

超频是指以超出制造商建议的时钟频率运行芯片以提升性能。通常会增加电压以减少内部传播延迟，以避免亚稳态，并增加冷却以防止芯片损坏。

有一些确定性高的超频设计方法，其中一些方法包括使用环振荡器测量传播延迟，如下所示。

有时有些应用不需要100%的可靠性，或者无法满足应用程序所需的吞吐量或延迟阈值，而定制ASIC也不是一个选择。有段时间，当FPGA上的加密挖矿更流行时，对FPGA进行超频是相当常见的。

示例

1.  [https://tspace.library.utoronto.ca/bitstream/1807/101178/3/Ahmed_Ibrahim_%20_202006_PhD_thesis.pdf](https://tspace.library.utoronto.ca/bitstream/1807/101178/3/Ahmed_Ibrahim_%20_202006_PhD_thesis.pdf)

## 设备原语

设备原语是FPGA内部的个体构建模块，用于构建您描述的逻辑。其中包括查找表、触发器（寄存器）、乘法器、加法器、传输链、可变延迟线、时钟生成器、高速串行收发器、时钟缓冲器、IO驱动器和接收器等。深入理解这些个体模块并了解它们固有的行为是在FPGA上构建非常规电路的关键。对于日常工作，深入理解这些模块是作为FPGA开发人员的重要基石之一。

# 环振荡器

环振荡器是使用查找表或其他延迟元件的组合回路构建的。一串数字反相器将振荡，创建一个随着单个反相器的传播延迟变化的时钟。延迟越低，时钟频率越高。频率也会随着PVT和其他影响因素而变化。

注意，高频率下工作的环振荡器可能会导致高能耗和局部加热，可能损坏FPGA。

## 动态传播延迟测量

环振荡器的一个用途是通过比较环振荡器的频率与稳定振荡器的频率来测量反相器链的传播延迟。这种传播延迟的测量可以用于动态电压和频率缩放方案，以实现跨PVT的稳定超频。

示例

1.  [https://www.eecg.utoronto.ca/~vaughn/papers/apec2016_dvs.pdf](https://www.eecg.utoronto.ca/~vaughn/papers/apec2016_dvs.pdf)

## 应变计

FPGA中的环振荡器可以用作应变计。如果在FPGA上实现一个环振荡器，然后弯曲FPGA，环振荡器的频率将发生变化。我不知道这是为什么。有人提出是硅的[压阻效应](https://en.wikipedia.org/wiki/Piezoresistive_effect)。

示例

1.  [https://hackaday.com/2015/09/27/mystery-fpga-circuit-feels-the-pressure/](https://hackaday.com/2015/09/27/mystery-fpga-circuit-feels-the-pressure/)

# 时间至数字转换器

时间数字转换器（TDC）是用于时间戳事件的电路。其中最简单的就是带有逻辑的常规计数器，用于记录观察到输入信号上升沿或下降沿时的计数。这种方法的问题在于定时精度受电路的时钟频率限制。500 MHz时钟只能将事件的定时精度限制在约2纳秒内。能够将事件定时精确到皮秒级别的TDC是激光测距、超声波测距、粒子物理实验和斜坡ADC中使用的重要仪器。

在我看来，存在几种TDC设计，但是我认为使用分析FPGA的模拟特性的敲击延迟线设计是最有趣的之一。

## 敲击延迟线

一种产生大约10皮秒分辨率事件可靠定时的方法被称为时间内插法。时间内插法利用FPGA中设备基元的传播延迟生成敲击延迟线（TDL）。对敲击延迟线进行采样以更精确地测量事件发生在时钟边沿之前的时间。

时间内插法的敲击延迟线方法可以通过生成和测量从一个事件中生成的多个事件来改进。这被称为Wave Union TDC（WU-TDC）。这种方法需要快速脉冲发生器，我们将在下文中介绍。

# 短脉冲发生器

在敲击延迟线中，非常短的脉冲可能很有用。通过利用组合逻辑失效产生数百皮秒长的脉冲，在创建上述Wave Union TDC时非常有用。

这些超短脉冲也可能用于超宽带通信（UWB）。

示例

1.  UWB脉冲发生器，[https://www.edn.com/build-a-uwb-pulse-generator-on-an-fpga/](https://www.edn.com/build-a-uwb-pulse-generator-on-an-fpga/)

1.  Wave Union TDC，[https://www.mdpi.com/2079-9292/11/1/30](https://www.mdpi.com/2079-9292/11/1/30)

# 真随机数生成

真随机数是加密的重要组成部分。有几种尝试在FPGA中制作真随机数发生器（TRNG）。许多使用环振荡器或亚稳态作为随机性的源。

示例

1.  基于环振荡器的TRNG，[https://github.com/stnolting/neoTRNG](https://github.com/stnolting/neoTRNG)

1.  基于亚稳态的TRNG，[https://people.csail.mit.edu/devadas/pubs/ches-fpga-random.pdf](https://people.csail.mit.edu/devadas/pubs/ches-fpga-random.pdf)

1.  商业上可用的核心，[https://www.bertendsp.com/products/trng-p200/](https://www.bertendsp.com/products/trng-p200/)

1.  商业上可用的核心，[https://www.latticesemi.com/products/designsoftwareandip/intellectualproperty/ipcore/xipheracores/xip8001b](https://www.latticesemi.com/products/designsoftwareandip/intellectualproperty/ipcore/xipheracores/xip8001b)

# 物理不可克隆功能

物理不可克隆功能（PUF）就像是指纹，它是一种为单个比特流和FPGA晶片组合提供唯一ID的设计。两个FPGA不应具有相同的ID，如果更改比特流，则所有ID都应更改。

这是一个特别棘手的问题。电路设计必须能够抵抗温度和电压变化，但不能抵抗处理变化，以便PUF在单个FPGA晶片上能够可靠一致，但在FPGA晶片之间是可靠不一致的。

如果它们可以可靠制造，PUF可以用作无需电池备份的密码密钥源。在这种情况下，建议将PUF作为挑战/响应系统的一部分，而不是直接将ID用作密钥。

示例

1.  [https://github.com/stnolting/fpga_puf](https://github.com/stnolting/fpga_puf)

# 数字到模拟转换

有许多方法可以使用FPGA上的逻辑和常规IO引脚来生成数字到模拟转换器（DAC）。

1.  PWM和外部低通滤波器

    1.  不是最有效的，但实施最简单，可能适用于基本音频输出。

1.  低通增量ΣΔ调制器和外部低通滤波器

    1.  比PWM DAC好得多，很可能可以达到约10-12位的分辨率，并且具有非常高的过采样率。

    1.  [https://github.com/hamsternz/second_order_sigma_delta_DAC](https://github.com/hamsternz/second_order_sigma_delta_DAC)

1.  带通增量ΣΔ调制器与外部带通滤波器

    1.  将低通增量ΣΔ调制器的积分器替换为谐振器，您可以制作带通增量ΣΔ调制器，它可以使用常规IO引脚达到低射频频率，在600 MHz的频率下，只需一个IO引脚使用输出串行器即可发送FM信号，它可以轻松地落在广播频段。

1.  使用多千兆位传输器的射频DAC

    1.  应用与上述类似的技术，可以使用多千兆位传输器达到更高的载波频率和更高的性能。

    1.  [https://ieeexplore.ieee.org/document/6927248](https://ieeexplore.ieee.org/document/6927248)

    1.  [https://ieeexplore.ieee.org/document/8421247](https://ieeexplore.ieee.org/document/8421247)

# 模拟到数字转换

使用FPGA进行模拟到数字转换（ADC）要比数字到模拟转换复杂得多。然而，在这一领域已经取得了许多成功。

在低功耗的冷却应用（如量子计算）中，ADC在FPGA中很受欢迎。如果您需要在接近绝对零度的冷却头附近使用ADC，则任何产生的热量都会使设计变得更加困难。

LVDS接收器引脚的模拟特性通常被用作模拟比较器。使用外部电容器的斜坡ADC或仅使用引脚的寄生电容可以用于生成高速或高精度ADC。

示例

1.  600 MSPS ADC，有效位数为7，适用于射频应用，[https://github.com/LukiLeu/FPGA_ADC](https://github.com/LukiLeu/FPGA_ADC)

1.  使用多个 LVDS 引脚进行时间交替的超过 1 GSPS，[https://ieeexplore.ieee.org/document/7593301](https://ieeexplore.ieee.org/document/7593301)

1.  LVDS 引脚 AM 接收器，[https://github.com/dawsonjon/FPGA-radio](https://github.com/dawsonjon/FPGA-radio)

# 频率合成

虽然 FPGA 上有用于时钟生成的 PLL，但也可以使用一些外部组件在 FPGA 上创建可用于射频前端的 PLL。理论上，在某些射频前端中，这可以替代外部频率合成器。

例子

1.  38-76 MHz 分数-N 合成器，[http://www.aholme.co.uk/Frac3/Main.htm](http://www.aholme.co.uk/Frac3/Main.htm)

# 天线

FPGA 上的路由基本上只是铜。通过手工路由或生成合适布局的自定义工具，可以将 FPGA 用作天线。

理论上，这可以用于某种数据外泄以传输数据。接收将会更加困难，因为通常需要在天线后面使用放大器。如果可以在其输入操作范围的中间某处使其处于尽可能亚稳态，那么可以将触发器用作增益为单比特 ADC。

例子

1.  探索用于设计非常规天线的 FPGA 互连，[https://dl.acm.org/doi/10.1145/1950413.1950455](https://dl.acm.org/doi/10.1145/1950413.1950455)
