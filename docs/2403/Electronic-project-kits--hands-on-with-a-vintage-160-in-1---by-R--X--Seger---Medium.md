<!--yml

category: 未分类

date: 2024-05-29 12:39:10

-->

# 电子项目套件：与复古160合一 | R. X. Seger | Medium

> 来源：[https://medium.com/@rxseger/electronic-project-kits-hands-on-with-a-vintage-160-in-1-eea39e6193f4](https://medium.com/@rxseger/electronic-project-kits-hands-on-with-a-vintage-160-in-1-eea39e6193f4)

这个特定的集成电路在现代并不太有趣，它相当隐晦，而且有许多其他放大器可以替代。这些电路中的IC可以替换成[闭环](https://en.wikipedia.org/wiki/Operational_amplifier#Closed_loop)配置的运算放大器吗？

使用非反向运算放大器：Vout = Vin*(1 + Rf/Rg)。

运算放大器芯片：[741](https://en.wikipedia.org/wiki/Operational_amplifier#Internal_circuitry_of_741-type_op-amp)（虽然流行但有更新的），LM324N四路运算放大器（我从一个[不间断电源](/@rxseger/building-an-h-bridge-from-a-salvaged-uninterruptible-power-supply-ab1576ec313f#.7tcvsnkwy)中回收的）

TODO：测试LM324N在这些电路中的替换效果。

## 旋钮：控制和调谐

“控制”是一个位于扬声器附近的电位器，用于音量或频率调整。零件清单中指定为50 kΩ可变电阻，我在两个电阻端测量到43.38 kΩ。调节游标到每个数字的粗略测量：

+   0: 45 Ω

+   1: 160 Ω

+   2: 5.4 kΩ

+   3: 10 kΩ

+   4: 17 kΩ

+   5: 24 kΩ

+   6: 31 kΩ

+   7: 36 kΩ

+   8: 40.04 kΩ

+   9: 43.55 kΩ

+   10: 43.39 kΩ

电阻从9到10几乎没有变化，可能是设计如此或者是年代久远的产物。

“调谐”是无标签电台电路中的一个可变电容器：

零件清单中指定为265 pF，我测量到… 0–1 nF，以及 ∞ Ω。坏了吗？

这可能是一个可行的替代品：[Fudan brand single joint air medium variable capacitor 12–365PF](https://www.aliexpress.com/item/Fudan-brand-single-joint-air-medium-variable-capacitor-12-365PF/32406520331.html?ws_ab_test=searchweb0_0%2Csearchweb201602_1_116_10065_117_10068_114_115_10069_113_10084_10083_10017_10080_10082_10081_10060_10061_10062_10056_10055_10054_10059_10078_10079_10073_10070_421_420_10052_10053_10050_10051%2Csearchweb201603_3&btsid=6c95a6db-565e-4c59-8b5b-97eea3f717ea). TODO：测试使用可变电容的电路

## 扬声器

扬声器是这个套件中最重要的组件之一。它提供了重要的输出：给建造电路的听众听到的听觉反馈。当用于振荡器时，我仍然认为低容量陶瓷电容器是“高音”，因为从这个扬声器听到的声音。

它始终连接到输出变压器（除了“103.继电器和扬声器蜂鸣器”中使用继电器生成交流波形外），尽管输出变压器有时单独使用。

一个很好的机会来测试我周围一些旧扬声器，使用同样的机关枪振荡器电路：

在部件清单中，扬声器只规定为“57毫米S-4565”，但我测量为8.2 Ω，看起来差不多正确。上面显示的三个额外扬声器标签均为8 Ω，在这个电路中都可以正常工作。

还有一套耳机，列为“耳机，高阻抗晶体型（无直流路径）E-0007”，可能会有用，遗憾的是这套我现在没有了。

TODO：可以用手机/电脑耳机作为替代吗？

## 变压器

两个小型变压器：

+   输入变压器（黄色），TD-0097 / 11106069

+   输出变压器（红色），TD-0136 / 11106077

这些零件号似乎是专用套件的，找不到太多信息。但是，[维基百科：变压器类型](https://en.wikipedia.org/wiki/Transformer_types#Audio_transformer)有一些信息：

> 音频变压器专门设计用于音频电路中传输音频信号。它们可以用于阻断无线电频率干扰或音频信号的直流分量，分割或组合音频信号，或在高阻抗和低阻抗电路之间提供阻抗匹配，例如在高阻抗电子管（真空管）放大器输出和低阻抗扬声器之间。

在红色（输出）音频变压器上进行一些测量：

+   1.3 Ω次级线圈（输出，连接扬声器）

+   56 Ω主线圈

+   从主线圈中心分接到两侧的28 Ω

估计为56/1.3 = 43:1的变比。

还有黄色（输入）变压器：

+   次级线圈为156.3 Ω

+   主线圈（输入，连接电路）为190.9 Ω

+   从中心分接约96.1 Ω

输入变压器估计为1.2:1的变比，远低于以上。

使用冲击式机枪脉冲振荡器项目产生声音时，测量输出变压器次级线圈上的交流电压，读数为数百毫伏，交流电流为数十毫安。另一方面，输入电压从几伏到约十伏不等。现在清楚了这个输出变压器的目的：将高电压/低电流变为低电压/高电流以驱动扬声器。

我们如何找到一个合适的替代变压器呢？这是我有的信息：

最大的变压器来自[一个微波炉](/@rxseger/emerson-mw8675w-microwave-oven-teardown-salvaging-the-led-display-a27cd86580b3#.jrq0ahif8)，次大的来自[一个UPS](/@rxseger/building-an-h-bridge-from-a-salvaged-uninterruptible-power-supply-ab1576ec313f#.18iyulrq8)，其余的来自早已失传的各种来源。但这里重要的是变比。我们希望主线圈（带有中心分接）比次级线圈多约40倍的匝数，或至少足以驱动扬声器。

我拥有的大多数变压器具有低变比（包括1.0，用于[隔离变压器](https://en.wikipedia.org/wiki/Isolation_transformer)），或者没有中心分接。低变比变压器可能适用于替换160合1的“输入变压器”。对于输出变压器，我找到了一个合适的：

+   主线圈61.5 Ω，次级线圈0.8 Ω，变比= 76:1

这是底部的小黄色变压器，有五根引线从中出来（这些引线是细小的绞线，表明这是用于音频的）。将其连接到电路中，工作正常：

## 开关

在主板上有一个滑动的[双极双切](https://zh.wikipedia.org/wiki/%E5%88%87%E6%8D%A2%E5%99%A8#%E8%81%94%E7%BB%9D%E5%99%A8%E6%9C%AF%E8%AF%AD)开关。该套件还配备了莫尔斯键，简单地是一个带有[摩尔斯电码](https://zh.wikipedia.org/wiki/%E6%91%A9%E5%B0%94%E6%96%AF%E7%A0%81)印刷在底座上的瞬时开关，但是我的二手套件上缺少这个。这里是手册上的一张图片：

## 灯

灯被列为“灯，红色2.5伏特，300毫安L-0541”，但在3伏特（2xAA电池）上表现良好，如图所示（手册警告如果用9V电源会烧毁）。比我预期的要亮，并且多年来也没有烧毁，或者可能是前任所有者更换了。测量灯泡两端的电阻为2.5 Ω。

这个套件上还有另一组灯，LED数字显示器：

感谢[Jeff Keyzer为提供的这张照片](https://www.flickr.com/photos/mightyohm/2729475946/in/photostream/)（来自Flickr，使用带有署名，裁剪的方式），他的相机或器材比我的更好。在低光条件下，每个段都照亮：

这个分段显示器是共阴极（负极），这里由3伏特电池供电。列出如下：

*发光二极管显示器（每段最小1.6伏特，25毫安最大直流电流，每段最大3伏特反向电压）L-0541*

硬连线电阻器为360 Ω，组装在“LED显示器电路板X-7159”上。如果您有兴趣，可了解更多关于我在[*Emerson MW8675W微波炉拆解：拆解LED显示器*](/@rxseger/emerson-mw8675w-microwave-oven-teardown-salvaging-the-led-display-a27cd86580b3#.9qg0aqsvc)中的经验。现在LED已经很常见，像300合1或者像各种树莓派套件一样的新套件（正如我在[*Raspberry Pi 3上的中断驱动IO：使用RPi.GPIO进行上升/下降沿检测*](/@rxseger/interrupt-driven-i-o-on-raspberry-pi-3-with-leds-and-pushbuttons-rising-falling-edge-detection-36c14e640fef#.r5lh29dw2)中使用的那样）包含了大量的LED，但这个套件只有一个白炽灯和这个普通的7段LED显示器。

TODO：可以给*左侧*的小数点通电吗？

## 继电器

电磁继电器工作在9伏特，有225Ω线圈（R-8158）。它是常见的[单刀双掷](https://en.wikipedia.org/wiki/Relay#Pole_and_throw)（SPDT）类型，线圈有两个引线，三个触点：常闭、公共和常开。这是一个好的继电器，在从已拆解的UPS中回收的继电器中，我用了四个而不是两个。当然，这个小继电器的触点没有220V的额定电压，不像UPS中的那些继电器。

有更现代化的组件用于开关（= [晶体管](https://en.wikipedia.org/wiki/Transistor)），但继电器仍然是一种易于理解、具有教育意义的简单组件，用于理解开关电路的操作，并具有其他一些好处。160个项目中有37个使用了继电器，从第一个项目（1\. 电子蜡烛）到逻辑电路（54\. 逻辑“NOR”电路等），甚至是高压发生器（43\. 高压发生器和44\. 高压发生器 II）。我测试了将9V电池接触其线圈，它仍然正常工作，并发出令人满意的点击声。

## 天线线圈

谈到线圈，"无线电电路"部分还有另一个线圈：

它只列为“天线线圈（带5根引线）CA-0619”，除了它有固体铁氧体芯之外，找不到更多信息。可能已损坏，需要在无线电电路中测试。TODO：测量电感，[LCR表](https://en.wikipedia.org/wiki/LCR_meter)

TODO：是否可行用从[*拆开Winsson 62–1225电源变压器*](/@rxseger/opening-a-winsson-62-1225-power-transformer-a66c4afd25f0#.bahska9fr)中回收的电线自己绕制天线线圈？

## 仪表

0–250 µA，650 Ω表针（“蓝色刻度与表电流成比例。黑色为记录参考。”）模拟表随着数字显示器（7段显示或其他）的出现已经大量不再使用。套件中包含的这个用于微安的仪表，我碰巧还有另一个用于安培：

但尚未找到其实际用途。

## CdS电池和太阳能电池

[图片来源：Jeff Keyzer](https://www.flickr.com/photos/mightyohm/2729475080/in/photostream/) via Flickr

[CdS硫化镉电池光敏电阻](https://en.wikipedia.org/wiki/Photoresistor)（KC-4S，CS-0100）是一种光传感器。它配有一个拇指形状的“光遮蔽罩”（缺失）：

类似的组件可用，比如来自[Adafruit：光电池CdS光电阻](https://www.adafruit.com/product/161)。光电阻根据光强度变化其电阻，但还有其他感应光的替代品，比如[光电二极管](https://en.wikipedia.org/wiki/Photodiode)，它将光转换为电流，就像我在[*SPI接口实验：EEPROMs，Bus Pirate，ADC/OPT101与树莓派*](/@rxseger/spi-interfacing-experiments-eeproms-bus-pirate-adc-opt101-with-raspberry-pi-9c819511efea#.9yky4ya96)中使用的那样。

“太阳能电池，开路电压2.5V典型值，短路电流300勒克斯条件下29 µA典型值”是零件清单如何描述太阳能电池的方式。这套电路中的一些电路完全可以由太阳能供电。对于那些不能的电路：

## 电池

标准的9V电池夹和2xAA电池盒供3V使用。尽管电池供电电路并不罕见，但现代爱好者通常使用交流电源，通常是通过USB适配器供电5伏特。但这套电路中的所有电路都可以由这些电池之一供电（有些可以同时供电），除了“20. 硬币电池”及相关项目，其本身是关于利用镀锌金属和一枚美分创造电池：
