<!--yml

category: 未分类

date: 2024-05-27 14:47:39

-->

# 使用 Arduino GIGA R1 WiFi 控制 3.6kW 太阳能电动车充电 | Arduino 博客

> 来源：[https://blog.arduino.cc/2024/03/04/controlling-3-6kw-of-solar-ev-charging-with-an-arduino-giga-r1-wifi/](https://blog.arduino.cc/2024/03/04/controlling-3-6kw-of-solar-ev-charging-with-an-arduino-giga-r1-wifi/)

### 使用 Arduino GIGA R1 WiFi 控制 3.6kW 太阳能电动车充电

Arduino 团队 — 2024 年 3 月 4 日

EV（电动车）与ICE（内燃机）的辩论可能比看上去更加复杂，但有一个事实很简单：在家里发电要比提炼化石燃料容易得多。这意味着在初始投资后，完全可以免费驱动车辆。但这需要大量硬件支持，这也是为什么 [Shawn Murphy 开发了这个充电系统](https://www.hackster.io/racingtogreen/solar-powered-ev-charging-with-agrivoltaics-0973c0)，并通过 Arduino GIGA R1 WiFi 控制。

Murphy 拥有一辆福特闪电电动皮卡，按电动车的标准来说并不高效，因为它的重量。但即使每千瓦时只有两英里的续航能力，他估计他可以在四到五年内通过太阳能充电系统收支平衡。之后，用来驱动福特的电力基本上就是免费的。任何多余的能量都可以为他的家提供电力或反馈到电网上。

单单用于驱动卡车本身就需要大量电力，所以 Murphy 购置了 10 块二手 360 瓦太阳能电池板。这些电池板供电到一个备用电池组，然后供电到福特充电站。

Murphy 希望最大化效率，因此希望太阳能电池板在一个轴上转动以跟随太阳。他估计这将在全天内增加它们的输出量约 20-25%，对于这么大规模的太阳能电池阵列来说，这是相当可观的能量。一个 [Arduino GIGA R1 WiFi board](https://store.arduino.cc/products/giga-r1-wifi) 通过线性执行器控制板块的倾斜。最初，Murphy 使用的是“哑”执行器，但现在他正准备转向来自 Progressive Automations 的“智能”型号，这些型号通过霍尔效应传感器提供位置反馈。

一个 [GIGA Display Shield](https://store.arduino.cc/products/giga-display-shield) 提供了 Murphy 接入界面的途径，他也可以通过 [Arduino Cloud](https://cloud.arduino.cc) 进行访问。除了控制线性执行器外，Arduino 还监控发电和能耗情况。

[这还在进行中](https://www.hackster.io/racingtogreen/solar-powered-ev-charging-with-agrivoltaics-0973c0)，Murphy 在继续改进中，但他已经在为卡车实现“免费”能源的道路上取得了不小的进展。
