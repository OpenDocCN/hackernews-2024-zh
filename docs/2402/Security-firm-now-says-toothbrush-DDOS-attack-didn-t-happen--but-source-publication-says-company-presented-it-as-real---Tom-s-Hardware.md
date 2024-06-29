<!--yml

category: 未分类

date: 2024-05-27 14:38:04

-->

# 安全公司现在称牙刷DDOS攻击并未发生，但新闻来源称公司确实将其作为真实事件呈现 | Tom's Hardware

> 来源：[https://www.tomshardware.com/networking/three-million-malware-infected-smart-toothbrushes-used-in-swiss-ddos-attacks-botnet-causes-millions-of-euros-in-damages](https://www.tomshardware.com/networking/three-million-malware-infected-smart-toothbrushes-used-in-swiss-ddos-attacks-botnet-causes-millions-of-euros-in-damages)

**更新2 — 2024年2月9日上午6:30：** 原始报告中提到，三百万支牙刷被用于DDOS攻击的安全公司现已撤回该报道，并声称这是翻译错误的结果 —— 但据刊登初始报道的新闻机构称，[这一说法并不正确](https://www.aargauerzeitung.ch/wirtschaft/cyberangriff-die-gehackten-zahnbuersten-gehen-medial-um-die-welt-und-loesen-fragen-aus-wie-es-dazu-kam-ld.2577182)。据报道，这一事件并不是媒体的翻译错误。该出版物声称Fortinet将此事描述为实际发生，并批准了文章的文本，该文章在发布前已提交给Fortinet。

Aargauer Zeitung（报道的来源）对此事的声明（通过[Google翻译](https://www.aargauerzeitung.ch/wirtschaft/cyberangriff-die-gehackten-zahnbuersten-gehen-medial-um-die-welt-und-loesen-fragen-aus-wie-es-dazu-kam-ld.2577182))：

*加利福尼亚Fortinet总部现在称之为“翻译问题”的事情，在调查期间听起来完全不同：瑞士Fortinet代表在讨论当前威胁的会议上描述了牙刷案件为真正的DDoS攻击-攻击描述。*

*Fortinet提供了具体细节：关于攻击导致瑞士公司网站宕机的时长；造成的损失规模。出于对客户的考虑，Fortinet没有透露是哪家公司。*

*在发布前，文本已提交给Fortinet进行验证。未有异议的声明称这是一个真实发生的案例。*

*Fortinet的全球管理部门现已收回了其向多家国际媒体发布的声明。该公司还未向CH Media发送此声明。我们还未收到Fortinet的进一步声明。"*

***编辑2/7/2024 — 下午3:30：*** Fortinet向我们发送了一份声明，表明牙刷攻击报告不准确：

获取Tom's Hardware最佳新闻和深度评论，直接发送到您的收件箱。

“为了澄清，关于牙刷被用于DDoS攻击的话题是在一次采访中作为某种攻击类型的例子进行讨论的，这并不是基于Fortinet或FortiGuard Labs的研究。由于翻译问题，这个话题的叙述已经被拉伸到了假设性和实际情况模糊的地步。” - Fortinet。

[源报告的原文](https://www.aargauerzeitung.ch/wirtschaft/kriminalitaet-die-zahnbuersten-greifen-an-das-sind-die-aktuellen-cybergefahren-und-so-koennen-sie-sich-schuetzen-ld.2569480?reduced=true)如下所述：

*“她在家里的浴室里，但她却是一场大规模网络攻击的一部分。这款电动牙刷采用Java编程，犯罪分子悄无声息地在上面安装了恶意软件 - 就像其他三百万支牙刷一样。一个命令足以让远程控制的牙刷同时访问一家瑞士公司的网站。网站崩溃，瘫痪了四个小时。造成数百万美元的损失。*

***这个例子，看起来像是好莱坞的情节，实际上发生了。*** 它展示了数字攻击的多功能性。” [强调添加]

一个[德语媒体报道](https://www.golem.de/news/iot-hacker-missbrauchen-zahnbuersten-fuer-ddos-angriffe-2402-181921.html)称这个故事“实际上发生了”，表明翻译是准确的，多位德语讲者已确认所述攻击“实际上发生了”。目前尚不清楚[Aargauer Zeitung](https://www.aargauerzeitung.ch/wirtschaft/kriminalitaet-die-zahnbuersten-greifen-an-das-sind-die-aktuellen-cybergefahren-und-so-koennen-sie-sich-schuetzen-ld.2569480?reduced=true)（原始来源）是否会发布更正。

[原文文章：](https://www.aargauerzeitung.ch/wirtschaft/kriminalitaet-die-zahnbuersten-greifen-an-das-sind-die-aktuellen-cybergefahren-und-so-koennen-sie-sich-schuetzen-ld.2569480?reduced=true)

根据[Aargauer Zeitung](https://www.aargauerzeitung.ch/wirtschaft/kriminalitaet-die-zahnbuersten-greifen-an-das-sind-die-aktuellen-cybergefahren-und-so-koennen-sie-sich-schuetzen-ld.2569480?reduced=true)（参考[Golem.de](https://www.golem.de/news/iot-hacker-missbrauchen-zahnbuersten-fuer-ddos-angriffe-2402-181921.html)）最近发布的一份报告，约有三百万只智能牙刷被黑客感染，并被奴役成为僵尸网络。源报告称这支庞大的联网牙齿清洁工具军团被用于对一家瑞士公司网站的DDoS攻击。据报道，该公司的网站在攻击压力下崩溃，导致数百万欧元的损失。

在这种特殊情况下，由于其基于Java的操作系统，牙刷僵尸网络被认为是脆弱的。源报告未提及任何特定的牙刷品牌。通常情况下，这些牙刷会利用其连接性来跟踪和改善用户的口腔卫生习惯，但在遭受恶意软件感染后，这些牙刷被迫加入了一个僵尸网络。

瑞士分公司的全球网络安全公司[Fortinet](https://www.fortinet.com/)的Stefan Züger向出版物提供了一些提示，人们可以采取措施来保护他们自己的牙刷或其他连接设备，如[路由器](https://www.tomshardware.com/networking/routers/best-wi-fi-routers)、机顶盒、监控摄像头、门铃、婴儿监视器、[洗衣机](https://www.tomshardware.com/networking/your-washing-machine-could-be-sending-37-gb-of-data-a-day)等。

“每一个连接到互联网的设备都是一个潜在的目标，或者可以被滥用进行攻击，” Züger告诉瑞士报纸。这位安全专家还解释说，黑客正在不断地对每一个连接设备进行漏洞探测，因此设备软件/固件制造商和网络犯罪分子之间存在着真正的武器竞赛。Fortinet最近将一台‘未受保护’的PC连接到互联网，并发现仅用了20分钟它就被恶意软件感染。

我们没有关于受到极具代价的DDoS攻击针对的特定瑞士公司的细节。然而，恶意行为者通常会在武器化他们的DDoS僵尸网络之前附加有金钱要求的威胁。也许瑞士公司拒绝支付，或者恶意行为者发动这次攻击以展示他们的实力（牙齿？）在提出任何要求之前。

尽管我们没有 DDoS 攻击的详细情况，但它仍然作为设备所有者的又一警告，要尽力保持设备、[固件](https://www.tomshardware.com/news/gigabyte-firmware-update-backdoor)和软件的更新；监控网络是否有可疑活动；安装和使用安全软件；并遵循[网络安全](https://www.tomshardware.com/reviews/wireless-security-hack,2981-3.html)的最佳实践。

我们已联系Fortinet进行评论，并根据需要更新本故事。

*注：* 本文标题最初为“在瑞士DDoS攻击中使用了三百万感染恶意软件的智能牙刷 —— 僵尸网络造成数百万欧元的损失”，但我们改动以反映新的发展。
