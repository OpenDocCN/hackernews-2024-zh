<!--yml

category: 未分类

date: 2024-05-27 12:55:35

-->

# 一场隐秘的攻击几乎危及了全球的计算机系统。

> 来源：[https://www.economist.com/science-and-technology/2024/04/02/a-stealth-attack-came-close-to-compromising-the-worlds-computers](https://www.economist.com/science-and-technology/2024/04/02/a-stealth-attack-came-close-to-compromising-the-worlds-computers)

2020年，流行的在线漫画《XKCD》刊登了一幅漫画，描述了一个摇摇欲坠的积木堆，标有标签：“所有现代数字基础设施”。在底部岌岌可危地支撑着一块孤零零的、细长的砖块：“一个随意在内布拉斯加州维护自2003年以来的项目。” 这幅插图迅速成为技术人员中的文化经典，因为它揭示了一个残酷的现实：支撑互联网核心的软件并非由巨大的企业或庞大的官僚机构维护，而是由少数默默无闻的志愿者辛勤工作。最近几天的网络安全惊慌表明，其结果可能是接近灾难性的。

2024年3月29日，微软工程师安德烈斯·弗洛因德（Andres Freund）发表了一篇简短的侦探故事。最近几周，他注意到SSH（一种安全登录到互联网上另一设备的系统）的运行速度比预期慢了约500毫秒。经过更仔细的检查发现，恶意代码深藏在XZ Utils中，这是用于在Linux操作系统内部压缩数据的软件，而Linux操作系统则运行在几乎所有公开可访问的互联网服务器上。这些服务器最终支撑互联网，包括重要的金融和政府服务。这段恶意代码本质上充当了一个“主钥匙”，允许攻击者窃取加密数据或植入其他恶意软件。
