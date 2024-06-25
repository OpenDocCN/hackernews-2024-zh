<!--yml

类别：未分类

日期：2024 年 05 月 27 日 15:24:01

-->

# 卫星正在成为新的手机基站-IEEE Spectrum

> 来源：[`spectrum.ieee.org/satellite-cellphone-starlink`](https://spectrum.ieee.org/satellite-cellphone-starlink)

Starlink 通过其最新一代卫星 v2mini，在本月首次通过 4G/LTE 连接在移动电话之间发送和接收短信，详情请参见[本月首次](https://api.starlink.com/public-files/DIRECT_TO_CELL_FIRST_TEXT_UPDATE.pdf)，此前类似的项目由[亚马逊](https://spectrum.ieee.org/tag/amazon)、苹果、AST SpaceMobile、华为和 Lynk Global 完成。由 SpaceX 运营的卫星星链将向全球至少八个不同移动网络运营商的订户提供短信服务，并且可能在“未来几年”不需要其客户现在使用的地面终端即可提供语音和数据覆盖范围，星链的美国合作伙伴 T-Mobile 在[声明中](https://www.t-mobile.com/news/un-carrier/first-spacex-satellites-launch-for-breakthrough-direct-to-cell-service-with-t-mobile)表示。

Starlink 的成就是卫星和蜂窝基站融合的最新例证。少数公司正在利用更便宜的卫星制造和发射成本，以及改进现有技术，例如[波束成形](https://spectrum.ieee.org/5g-bytes-beamforming-explained)，来弥合手机和轨道卫星之间的几百公里距离。这些公司需要解决的许多新问题之一是，这是首次，基站本身是网络的移动组件：低地球轨道（LEO）卫星的速度达到每小时数万公里，因此它们与地球表面上的任何一个手机的通信时间很短。

目前，正在竞争解决这些问题的公司已经通过商业卫星（华为/中国电信；Lynk Global；苹果/Globalstar）发送和接收了常规手机上的短信，并通过实验卫星（AST SpaceMobile）进行了[5G](https://spectrum.ieee.org/tag/5g)语音和数据通话，正如*[IEEE Spectrum](https://spectrum.ieee.org/)*所报道的。投资者已经注意到：Lynk Global 正在[上市](https://lynk.world/news/lynk-signs-letter-of-intent-to-become-publicly-listed-leading-satellite-to-phone-company-through-a-business-combination-with-slam-corp/)，交易价值高达 8 亿美元，而 AT&T、[谷歌](https://spectrum.ieee.org/tag/google)和沃达丰最近投资了 AST SpaceMobile，该公司的[市值为 6.746 亿美元](https://www.google.com/finance/quote/ASTS:NASDAQ)。

“我记得 10 年前，移动运营商告诉卫星人，‘你们的价格点太高了。’这完全改变了，因为技术更加可用，开发更加灵活，失败的处理方式也不同。” **—Andreas Knopp, 德国联邦国防大学**

直到最近，卫星才能连接下方数百公里的手机。人们带着卫星手机去更偏远的地方探险，需要笨重的天线，需要对多颗卫星有清晰的视线，需要一段时间才能获得信号。集成地面和卫星蜂窝网络并不像在不同的手机塔之间移动并将信号转交给下一个手机塔那样容易。

事实上，这项任务非常困难，以至于一个研究小组[建立了一个实验应用程序](https://ieeexplore.ieee.org/document/10199206)，帮助一辆配有自己的计算机的互联网连接的牲畜卡车在失去蜂窝网络信号时切换到机载的 Starlink 地面站。丹麦奥尔堡大学的无线通信研究员[Melisa López](https://vbn.aau.dk/en/persons/melisa-maria-lopez-lechuga)称，实现地面和非地面网络的无缝集成是最终目标。

Starlink 并没有解释其 4G 连接的许多细节，但现有的商用卫星群揭示了无缝卫星蜂窝连接的一些构建块，研究人员至少发表了一个有前途的未来卫星群的线索。

## 连接手机到卫星的三个关键

公司并不是重新设计手机以使其更像卫星手机，而是在不折不扣地为了迎合手机，重新设计卫星网络。它们正在使卫星上的天线变得更大，以将卫星转变为手机塔。例如，[AST SpaceMobile 的第一颗卫星](https://spectrum.ieee.org/satellite-cellphone)的天线表面积为 64 平方米，第二代卫星的天线为 128 平方米，计划增加到 400 平方米。Starlink 的新 v2mini 卫星天线为 6.21 平方米，但 Starlink 计划使用更大的与手机兼容的卫星，在其更大的 Starship 火箭可用时进行发射。

公司还在让他们的卫星更像手机塔，低于以前的飞行高度。在太空时代的头几十年，通信卫星被放置在地球高得多的地球同步轨道上，可以覆盖地球表面的大部分地区，时间相对较长。然而，那些卫星处理的设备远远少于今天存在的设备。

过去十年左右，更小、更便宜的卫星和更便宜的发射成本的出现，使得依赖于在低地球轨道飞行的许多更便宜的卫星的商业模式成为可能。这些新卫星的寿命不会太长，但能更好地探测地表移动电话发出的微弱信号并处理其不断增长的流量。

SpaceX 的 Starlink 网络可以直接连接到现成的手机，可能消除了庞大卫星电话的需求。

“我记得 10 年前我们进行了许多讨论，移动运营商告诉卫星人士，‘你们的价格点太高了。’这完全改变了，因为技术的可用性增加了，开发更加灵活，而且对失败采取了不同的方法，” 慕尼黑联邦国防大学信号处理工程师[Andreas Knopp](https://www.unibw.de/satcom/chair/team/andreas-knopp)说道。

另一个贡献者是改进的波束成形技术，这是指发送设备计算将其信号定向到特定接收者的最佳方式，而不会干扰其他接收者的方法。这可能涉及将信号从地面塔反弹到建筑物或山坡上，或者可能涉及精确瞄准窄、快速移动的信号，来自每小时数万公里的卫星。

更复杂的波束成形技术可能涉及从多个天线发送相同的信号，以便信号相互增强，有点像声波和谐的方式。调谐家庭扬声器系统以在特定沙发点提供最佳声音的任何人都做过波束成形，很可能在背景中借助了复杂的软件。

未来，可能值得将波束成形任务分配到更多的卫星上。[最近的两篇论文](https://ieeexplore.ieee.org/abstract/document/10355084) 的作者写道。其中一项研究中提出的一种情景是使用超过两打小型卫星以近距离编队飞行，以复制目前由一个与手机兼容的卫星完成的工作。“这些卫星每个都是独立的，拥有自己的组件。主要方面是同步算法，它必须使频率、相位和信号到达的时间能够协调一致，” 慕尼黑联邦国防大学信号处理研究生[Diego Tuzi](https://www.unibw.de/satcom/chair/team/diego-tuzi)说道，他和 Knopp 一起是其中一篇最近论文的合作者之一。

目前，Starlink 及其竞争对手提供的内容很少，尽管这是一个必要的步骤。 "这总比没有好，" 最近一篇论文的另一位共同作者，也是德国联邦国防大学的信号处理工程师[托马斯·德拉莫特](https://www.unibw.de/space-en/members/dr-thomas-delamotte)说道，"但如果你想有一个长远的视角，我们需要新的方法来使[6G](https://spectrum.ieee.org/tag/6g)无处不在。"

来自您网站的文章

相关文章周围的网络
