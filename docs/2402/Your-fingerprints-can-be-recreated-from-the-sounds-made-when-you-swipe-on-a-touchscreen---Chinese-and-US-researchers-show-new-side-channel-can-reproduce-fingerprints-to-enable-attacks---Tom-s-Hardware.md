<!--yml

category: 未分类

date: 2024-05-27 15:01:48

-->

# 您的指纹可以通过触摸屏上滑动时产生的声音重现——中国和美国研究人员展示新的侧信道可以重现指纹以实施攻击 | Tom's Hardware

> 来源：[https://www.tomshardware.com/tech-industry/cyber-security/your-fingerprints-can-be-recreated-from-the-sounds-made-when-you-swipe-on-a-touchscreen-researchers-new-side-channel-attack-can-reproduce-partial-fingerprints-to-enable-attacks](https://www.tomshardware.com/tech-industry/cyber-security/your-fingerprints-can-be-recreated-from-the-sounds-made-when-you-swipe-on-a-touchscreen-researchers-new-side-channel-attack-can-reproduce-partial-fingerprints-to-enable-attacks)

一组来自中国和美国的研究人员提出了对生物特征[安全](https://www.tomshardware.com/tag/security)的有趣新攻击。[PrintListener：揭示指纹认证的脆弱性通过手指摩擦声](https://www.ndss-symposium.org/wp-content/uploads/2024-618-paper.pdf) [PDF] 提出了对先进自动指纹识别系统（[AFIS](https://www.aratek.co/news/automated-fingerprint-identification-system-afis-an-overview)）的侧信道攻击。该攻击利用用户在[触摸屏](https://www.tomshardware.com/news/nokia-lumia-iphone-ipad-touchscreen,16351.html)上滑动的声音特征来提取指纹模式特征。经过测试，研究人员声称他们能够在最高安全假接受率设置为0.01%的情况下，“成功攻击高达27.9%的部分指纹和9.3%的完整指纹，仅需五次尝试”。据称，这是首个利用滑动声音推断指纹信息的研究。

[生物特征指纹安全](https://www.tomshardware.com/news/raspberry-pi-pico-bank-locker)广泛应用并深受信任。如果事态继续发展，预计到2032年，指纹认证市场将价值近1000亿美元。然而，组织和个人越来越意识到攻击者可能想窃取他们的指纹，因此一些人开始小心保护指纹不被看到，并对展示手部细节的照片变得敏感。

没有接触印迹或手指详细照片，攻击者如何希望获取任何指纹数据以增强对用户指纹的MasterPrint和DeepMasterPrint字典攻击结果呢？一个答案是：PrintListener论文中提到，“攻击者可以在线捕捉到指纹滑动摩擦声，具有很高的可能性。”指纹滑动声的来源可以是像[Discord](https://www.tomshardware.com/news/discord-throttles-nvidia-gpu-memory-clock-speeds)、Skype、WeChat、[FaceTime](https://www.tomshardware.com/news/apple-facetime-glitch-bug-eavesdropping-lawsuit,38527.html)等流行应用。任何用户在设备麦克风处于活动状态时轻率地在屏幕上进行滑动动作的对话应用。因此，PrintListener得名于侧信道攻击。

PrintListener内部运作背后有一些复杂的科学，但如果您已经阅读了上面的内容，您对研究人员为了不断改进他们的AFIS攻击所做的工作已经有了一个大致的了解。然而，有三个主要挑战被克服，以使PrintListener达到今天的水平：

+   从指纹摩擦的微弱声音分离：基于谱分析的摩擦声事件定位算法得到发展。

+   将手指图案对声音的影响与用户的生理和行为特征分离。为了解决这个问题，研究人员使用了最小冗余最大相关性（mRMR）和自适应加权策略。

+   进展从使用统计分析初级到二级指纹特征的推测，设计启发式搜索算法

图片1/2

（图片来源：华中科技大学、武汉大学、清华大学、科罗拉多大学丹佛分校）

（图片来源：华中科技大学、武汉大学、清华大学、科罗拉多大学丹佛分校）

为了证明这一理论，科学家们实际上将他们的攻击研究作为PrintListener来实际开发。简而言之，PrintListener使用一系列算法对原始[音频信号](https://www.tomshardware.com/tech-industry/artificial-intelligence/audacity-gets-ai-transcription-and-noise-suppression-courtesy-of-intel-openvino-plug-ins)进行预处理，然后用于生成PatternMasterPrint的有针对性的合成（由具有特定模式的指纹生成的MasterPrint）。

重要的是，PrintListener在“现实世界场景中”经历了大量实验，并且如引言中所述，可以在四分之一以上的情况下成功地进行部分指纹攻击，并在接近十分之一的情况下进行完整指纹攻击。这些结果远远超过未辅助的MasterPrint指纹词典攻击。

获取Tom's Hardware的最新消息和深度评论，直接发送到您的收件箱。
