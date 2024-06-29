<!--yml

category: 未分类

date: 2024-05-27 15:04:53

-->

# ads.txt - Wikipedia

> 来源：[https://en.wikipedia.org/wiki/Ads.txt](https://en.wikipedia.org/wiki/Ads.txt)

在线广告管理中使用的文本文件格式

**ads.txt**（**授权数字卖家**）是来自[IAB](/wiki/Interactive_Advertising_Bureau "互动广告局")技术实验室的一项倡议。它规定了公司可以在其[Web服务器](/wiki/Web_server "Web服务器")上托管的[文本文件](/wiki/Text_file "文本文件")，列出了其他被授权销售其产品或服务的公司。这旨在允许在线买家检查其购买产品的卖家的有效性，以进行[互联网欺诈预防](/wiki/Internet_fraud_prevention "互联网欺诈预防")。

## 采用状态[[编辑](/w/index.php?title=Ads.txt&action=edit&section=1 "编辑章节：采用状态"))

到2017年11月，超过44%的出版商使用了`ads.txt`文件。^([[1]](#cite_note-Ad_Ops_Insider-1)) 根据Pixalate的数据，截至2017年9月，超过3,500个网站在使用`ads.txt`文件之后，超过90,000个网站正在使用`ads.txt`文件。在出售程序化广告的前1,000个站点中，57%的站点有`ads.txt`文件，而2017年9月仅有16%。^([[2]](#cite_note-Digiday-2))

根据FirstImpression.io的Ads.txt行业仪表板，最新的采用数据如下：^([[3]](#cite_note-3))

| Websites Tier | Jan 30th, 2020 | Jan 30th, 2019 | Jan 30th, 2018 |
| --- | --- | --- | --- |
| Alexa Top 1,000 | 44.20% | 40.90% | 33.70% |
| Alexa Top 5,000 | 39.58% | 36.00% | 29.06% |
| Alexa Top 10,000 | 36.66% | 32.75% | 25.18% |
| Alexa Top 30,000 | 31.71% | 26.34% | 18.52% |

[Google](/wiki/Google "Google")一直是`ads.txt`的积极支持者，并推动发布者更快、更广泛地采纳该技术。^([[4]](#cite_note-4)) 从2017年10月底起，Google Display & Video 360只购买来自发布者`ads.txt`文件中被确认授权的卖家的库存，如果有的话。`ads.txt`可能会成为Display & Video 360的要求。^([[5]](#cite_note-MarTechToday-5))

## 文件格式[[编辑](/w/index.php?title=Ads.txt&action=edit&section=2 "编辑章节：文件格式")]

IAB的ads.txt规范^([[6]](#cite_note-IABTechLab-6))指定了ads.txt文件的格式，可以包含三种类型的记录：数据记录、变量和注释。一个ads.txt文件可以包含任意数量的记录，每个记录占一行。

由于必须遵守ads.txt文件格式，因此已经有一系列验证、^([[7]](#cite_note-IABTechLab_Resources-7)) 管理和协作工具可用来帮助确保正确创建ads.txt文件。

最初的规范，v1.0，发布于2017年。^([[6]](#cite_note-IABTechLab-6)) v1.0.1版在当年稍后根据社区反馈进行了小的修改。2019年，v1.0.2版进行了进一步的小修订，并建议使用一个占位符记录来指示空的ads.txt文件的意图：^([[8]](#cite_note-IABTechLab-spec-8)) `placeholder.example.com, placeholder, DIRECT, placeholder`。

2021年，v1.0.3版增加了`inventorypartnerdomain`指令，2022年的v1.1版增加了`managerdomain`和`ownerdomain`。

## 另见[[编辑](/w/index.php?title=Ads.txt&action=edit&section=3 "编辑部分：另见")]

## 参考资料[[编辑](/w/index.php?title=Ads.txt&action=edit&section=4 "编辑部分：参考资料")]

## 外部链接[[编辑](/w/index.php?title=Ads.txt&action=edit&section=5 "编辑部分：外部链接")]
