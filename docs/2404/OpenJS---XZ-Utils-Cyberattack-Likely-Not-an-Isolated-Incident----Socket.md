<!--yml

category: 未分类

date: 2024-05-27 13:17:22

-->

# OpenJS：“XZ Utils网络攻击可能并非孤立事件” - Socket

> 来源：[https://socket.dev/blog/openjs-xz-utils-cyberattack-likely-not-an-isolated-incident](https://socket.dev/blog/openjs-xz-utils-cyberattack-likely-not-an-isolated-incident)

OpenJS在收到一次针对该组织的社会工程接管尝试后，警告开源项目维护者要保持警惕。

“最近的尝试[XZ Utils后门事件](https://openssf.org/blog/2024/03/30/xz-backdoor-cve-2024-3094/)（[CVE-2024-3094](https://www.cve.org/CVERecord?id=CVE-2024-3094)）可能不是孤立事件，正如OpenJS基金会与全球数十亿网站使用的JavaScript项目的联合[声明](https://openjsf.org/blog/openssf-openjs-alert-social-engineering-takeovers)中所显示的那样，OpenJS基金会执行董事Robin Bender表示，存在类似的可信接管尝试。”

一次可信的接管尝试涉及一系列要求OpenJS采取行动更新其流行的JavaScript项目以“解决任何关键漏洞”的电子邮件，来自各种GitHub相关的电子邮件地址，作者要求被指定为新的维护者。OpenJS注意到这种方法与“贾坦”用来添加XZ/liblzma后门的方法类似。

基金会发布了一系列关于识别这些威胁和可疑模式的提示，以及保护开源项目和关键基础设施的步骤。

## 开源安全需要系统性改革[#](#Open-Source-Security-Needs-a-Systemic-Overhaul)

安全行业仍然因[xz-utils后门事件](https://socket.dev/blog/how-to-use-socket-to-find-out-if-you-were-affected-by-the-backdoored-xz-package)可能带来的潜在破坏而震惊，这一企图在被意外发现后靠纯运气挫败。这并非是一种面对规模化开源软件安全的策略。在下一次可能已经在进行中之前，生态系统只能靠借时间生存。

专家们正在思考宏观问题，并希望摆脱这些零星的反应性措施，转向更为现实和主动的安全战略。社区正在逐步开始将开源软件视为基础设施，这需要一种持续和系统性的变革，将安全性作为基础组成部分进行整合。这是一个难以解决的问题。

在Mastodon的[讨论](https://fosstodon.org/@camdoncady@infosec.exchange/112278085619243206)中，美国航空工程师Camdon Cady将现有物理基础设施面临的挑战与Linux生态系统内的安全问题进行了类比：

> 在美国，最近有一艘船撞上了一座桥，导致桥梁倒塌。技术解决方案（桥墩周围的碰撞保护）几十年来已广为人知，并且广泛应用于新桥上。保护装置尚未在所有现有桥梁上进行更新。
> 
> 我在想，也许Linux生态系统实际上是一堆老化的基础设施。如果是这样，这个事实可能被Linux内核看似健康的贡献所掩盖。然而，除了内核之外，还有大量其他的“基础设施”被随附运送：coreutils、binutils、gcc、make、autotools、cmake、zlib、xz等等等等。

信息安全专家Kurt Seifried [指出](https://fosstodon.org/@kurtseifried@infosec.exchange/112278322554923641)，尽管开源社区知道Linux内核及其周围的所有实用工具将被广泛使用，“我们并没有想到每个人的汽车和灯泡都会运行它”。开源软件已成为支持每个现代设备和系统的隐形基础设施，从家用电器到关键的国家基础设施。

最近的[报告](https://www.atlanticcouncil.org/in-depth-research-reports/report/open-source-software-as-infrastructure/)与开源政策网络及一群开源软件开发者和维护者合作起草，认为开源软件（OSS）像物理基础设施一样，应当加强以确保其可持续广泛使用，以防止在灾难性安全风险下崩溃。报告并非认为OSS本质上不安全，而是强调其与基础设施的平行性，需要政府在生态系统的长期健康方面发挥作用。

> 当政策只关注可怕的潜在结果时，其思想往往会反映出对恐惧的偏见，但这不应成为开源软件的框架。开源使得用户从个人到国家情报机构都能够更多地实现和解决问题，而不是构成威胁。它的安全性不仅是对用户持续价值的保证，从个人到大型用户，甚至是国家情报机构，也是防范恶意意图的堡垒。
> 
> 虽然开源软件作为欧盟（EU）国家政策问题再次引起关注，并且在某些方面首次成为美国的一个问题，源于恐惧和灾难，但机遇远比表面更深远。这种规模和重要性的基础设施得到支持、加强和放大，而不是仅仅在短暂的活动风暴中修复，就像持续提供清洁水、道路和桥梁以及健康的资本市场一样。

不足为奇的是，OpenJS 基金会和被95%以上的所有网站使用的 JavaScript 生态系统，成为寻求被指定为重要项目维护者的威胁行为者的下一个目标。不幸的现实是，这些恶意尝试以惊人的速度发布到公共注册表中。我们在 Socket 团队每周在 npm、PyPI 和 Go 中捕获100多次软件供应链攻击。

软件工程师安德鲁·尼斯比特一直在[Ecosyste.ms](https://advisories.ecosyste.ms/advisories?order=desc&sort=blast_radius)上探讨开源安全通告的“爆炸半径”概念。他使用安全通告的 CVSS 分数乘以依赖于该软件包的仓库数量来确定“爆炸半径”。从这个角度来看，尼斯比特得出结论：在一个非常流行的库中的中等漏洞可能比在不受欢迎的库中的关键 CVE 影响更大。

这个实验在[Ecosyste.ms](http://Ecosyste.ms)上强调了开源软件包及其依赖之间深层次的相互关系，以及安全事件如何从一个单点故障迅速传播到整个生态系统，影响到众多项目和应用。

把开源软件仅视为免费资源而不是关键保护资产的做法，在面对开源软件快速扩展过程中日益复杂的网络威胁时已经证明是不足够的。这些类比传统物理基础设施的思路是系统改革的基础。

OpenJS 正在引发关于他们桌面上出现的威胁的警报，但更广泛的挑战在于识别和减轻尚未被发现的供应链攻击。从根本上保护开发过程是开始的地方。开发者们应该认真对待这个警告，并严格审核他们现有的软件包，确保不会引入潜在的恶意依赖，或者轻率地信任历史上一直安全的软件包或维护者。
