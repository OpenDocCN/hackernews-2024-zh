<!--yml

category: 未分类

date: 2024-05-27 14:28:54

-->

# cr.yp.to: 2024.01.02: 双重加密

> 来源：[https://blog.cr.yp.to/20240102-hybrid.html](https://blog.cr.yp.to/20240102-hybrid.html)

# cr.yp.to [blog](index.html)

* * *

<details><summary>目录（Index页的Access-I）</summary>

| **2024.01.02: 双重加密：** 分析NSA/GCHQ 对混合加密方案的论点。#nsa #quantification #risks #complexity #costs |
| --- |
| [**2023.11.25: Kyber-512安全分析的另一种搞砸的方式：**](20231125-kyber.html) 回应最近的一篇博客文章。#nist #uncertainty #errorbars #quantification |
| [**2023.10.23：减少Kyber-512"门"计数：**](20231023-clumping.html) 从第一原理出发对两种算法进行分析，与 NIST 的计算相矛盾。#xor #popcount #gates #memory #clumping |
| [**2023.10.03: 不能正确计数：**](20231003-countcorrectly.html) 揭穿 NIST 对于Kyber-512 安全级别的计算。#nist #addition #multiplication #ntru #kyber #fiasco |
| [**2023.06.09: Turbo Boost：**](20230609-turboboost.html) 如何持续化安全问题。#overclocking #performancehype #power #timing #hertzbleed #riskmanagement #environment |
| [**2022.08.05: NSA、NIST 和后量子密码：**](20220805-nsa.html) 宣布我对美国政府的第二起诉讼。#nsa #nist #des #dsa #dualec #sigintenablingproject #nistpqc #foia |
| [**2022.01.29: 抄袭作为专利放大器：**](20220129-plagiarism.html) 理解后量子密码的延迟推出。#pqcrypto #patents #ntru #lpr #ding #peikert #newhope |
| [**2020.12.06: 为错误的指标进行优化，第1部分：Microsoft Word：**](20201206-msword.html) 对 Knauff 和 Nejasmic 的论文“学术研究和开发中使用的文件准备系统的效率比较”进行审查。#latex #word #efficiency #metrics |
| [**2019.10.24：EdDSA 相对于 ECDSA 更能抵抗 Minerva 的原因：**](20191024-eddsa.html) 密码系统设计者成功预测并保护免受实施失败的影响。#ecdsa #eddsa #hnp #lwe #bleichenbacher #bkw |
| [**2019.04.30: 向量化简介：**](20190430-vectorize.html) 了解高速软件生态系统中最重要的变化之一。#vectorization #sse #avx #avx512 #antivectors |
| [**2017.11.05: 重构ROCA：**](20171105-infineon.html) 一个从有限披露中迅速开发攻击的案例研究。#infineon #roca #rsa |
| [**2017.10.17：查找碰撞的量子算法：**](20171017-collisions.html) 对碰撞问题和相关的多目标预影问题的几种算法进行分析。#collision #preimage #pqcrypto |
| [**2017.07.23: 快速键擦除随机数发生器：**](20170723-random.html) 同时清理几个混乱。#rng #forwardsecrecy #urandom #cascade #hmac #rekeying #proofs |
| [**2017.07.19: 后量子密码学的基准测试:**](20170719-pqbench.html) 有关SUPERCOP基准测试系统的新闻，以及对NIST的更多建议。 #benchmarking #supercop #nist #pqcrypto |
| [**2016.10.30: 后量子标准化中的一些挑战:**](20161030-pqnist.html) 我对NIST关于他们的征集提交的第一稿的评论。 #standardization #nist #pqcrypto |
| [**2016.06.07: 正当程序的消亡:**](20160607-dueprocess.html) 关于技术驱动的围攻行为对原告和被告的规范化的一些笔记。 #ethics #crime #punishment |
| [**2016.05.16: 欧洲“量子宣言”中的安全欺诈:**](20160516-quantum.html) 量子密码学家如何从欧洲委员会手中窃取25亿欧元。 #qkd #quantumcrypto #quantummanifesto |
| [**2016.03.15: 托马斯·杰斐逊与苹果对抗FBI:**](20160315-jefferson.html) 政府能够审查如何书写吗？如果一些读者是罪犯怎么办？如果这些书可以被计算机理解呢？为软件出版商介绍言论自由。 #censorship #firstamendment #instructions #software #encryption |
| [**2015.11.20: 破解一打秘钥，再免费获取一百万:**](20151120-batchattacks.html) 批量攻击往往比单目标攻击更具成本效益。 #batching #economics #keysizes #aes #ecc #rsa #dh #logjam |
| [**2015.03.14: 优化编译器的消亡:**](20150314-optimizing.html) 我在ETAPS 2015年的教程摘要。 #etaps #compilers #cpuevolution #hotspots #optimization #domainspecific #returnofthejedi |
| [**2015.02.18: 跟踪打印:**](20150218-printing.html) Equitrac的营销部门如何歪曲事实并干扰你的工作。 #equitrac #followyouprinting #dilbert #officespaceprinter |
| [**2014.06.02: Saber集群:**](20140602-saber.html) 我们如何建立一个能够每年计算3000000000000000000000次乘法的集群，成本仅为50000欧元。 #nvidia #linux #howto |
| [**2014.05.17: 关于英特尔指令集的一些建议:**](20140517-insns.html) 低成本的CPU架构改变将使加密更安全更快速。 #constanttimecommitment #vmul53 #vcarry #pipelinedocumentation |
| [**2014.04.11: NIST的密码标准化流程:**](20140411-nist.html) 改进的第一步是承认之前的失败。 #standardization #nist #des #dsa #dualec #nsa |
| [**2014.03.23: 如何设计椭圆曲线签名系统:**](20140323-ecdsa.html) 有很多选择的椭圆曲线签名系统。标准选择ECDSA，如果你不关心简单性、速度和安全性，这是合理的选择。 #signatures #ecc #elgamal #schnorr #ecdsa #eddsa #ed25519 |
| [**2014.02.13: 对理想格的次域对数攻击:**](20140213-ideal.html) 计算代数数论解决基于格的密码学。 |

| [**2014.02.05：熵攻击！**](20140205-entropy.html) 传统观点认为哈希输出是无法控制的；传统观点是错误的。 |</details>

* * *

## 2024.01.02：双重加密：分析 NSA/GCHQ 对混合方案的论点。#nsa #量化 #风险 #复杂性 #成本

在 2019 年，Google 和 Cloudflare 进行了一个实验，将许多 ["真实用户的连接"](https://blog.cloudflare.com/towards-post-quantum-cryptography-in-tls/) 升级为使用后量子加密的 HTTPS。然后，他们报告了 [统计数据](https://blog.cloudflare.com/the-tls-post-quantum-experiment/)，说明这是多么实惠。Google 也在 2016 年进行了类似的 [实验](https://security.googleblog.com/2016/07/experimenting-with-post-quantum.html)："虽然量子计算机仍处于起步阶段，但我们很高兴开始为它们做准备，并帮助确保我们用户的数据长期保持安全。"

听起来很不错！除了，哎呀，2016 年的实验遇到了 [专利问题](20220129-plagiarism.html)。还有，糟糕的是，2019 年实验中一半的后量子连接使用了 SIKE，而在 2022 年被 [公开](https://eprint.iacr.org/2022/975) [攻击](https://eprint.iacr.org/2022/1026) 展示出可以有效破解。

将连接升级到 SIKE 不仅未能保护这些连接免受未来量子计算机的攻击。而且还使用了一种被 *今天的* 计算机攻破的算法来加密连接。这并不会将用户数据提供给今天的攻击者的唯一原因是这些实验使用了 "混合 "：继续使用椭圆曲线密码学进行加密，同时 *添加* 了一层新算法的加密。椭圆曲线密码学是 [X25519](https://ianix.com/pub/curve25519-deployment.html)，在 2019 年已经是 ["最常用的密钥交换算法"](https://blog.cloudflare.com/towards-post-quantum-cryptography-in-tls/) 之一，而今天则用于 TLS 连接的 ["绝大多数"](https://eprint.iacr.org/2023/734.pdf) 中。

现今的后量子部署已经远远超出了实验阶段。OpenSSH 在 2022 年 4 月升级为默认的 ["混合 Streamlined NTRU Prime + x25519 密钥交换方法"](https://www.openssh.com/txt/release-9.0)。Google 在 2022 年 11 月将其内部通信升级为 ["NTRU-HRSS 和 X25519"](https://cloud.google.com/blog/products/identity-security/why-google-now-uses-post-quantum-cryptography-for-internal-comms) 的混合， 并在 2023 年 8 月升级 Chrome 以支持 ["X25519Kyber768"](https://blog.chromium.org/2023/08/protecting-chrome-traffic-with-hybrid.html) 在 TLS 中的使用。

所有这些升级都使用后量子层来 *尝试* 保护今天的用户数据免受未来量子计算机的攻击，并同时保留一个前量子层，以防后量子系统出现问题时不 *失去* 安全性。

这不应该是有争议的。然而，[NSA](https://nsarchive2.gwu.edu/NSAEBB/NSAEBB441/) 在2022年5月表示，它["不打算批准"](https://web.archive.org/web/20220524232249/https://twitter.com/mjos_crypto/status/1433443198534361101/photo/1) 混合密码系统。在2022年7月SIKE破解后（另请参见我的2022年8月博客文章["NSA，NIST和后量子密码学"](20220805-nsa.html)中有关混合密码系统的部分），NSA发布了一个[2022年9月常见问题解答](https://web.archive.org/web/20220908002357/https://media.defense.gov/2022/Sep/07/2003071836/-1/-1/0/CSI_CNSA_2.0_FAQ_.PDF)，从"不打算批准"略微退缩到"不会要求"。NSA的常见问题解答继续*不鼓励*使用混合密码系统。

意识到美国国家安全局（NSA）的公开立场对加密市场有着重大影响是很重要的。美国的[军事预算](https://en.wikipedia.org/wiki/Military_budget_of_the_United_States)每年接近一万亿美元，而NSA [控制](https://web.archive.org/web/20221022163808/https://www.jcs.mil/Portals/36/Documents/Library/Instructions/CJCSI%206510.02F.pdf?ver=qUEnOsWpGPcGGMFTb4yYVA%3D%3D) 其中的加密部分采购。

这并不意味着NSA有无限的权力。例如，参见上面关于TLS中X25519部署的引用[#x25519deployment]。[几乎](https://www.ssllabs.com/ssltest/viewClient.html?name=Chrome&version=80&platform=Win%2010&key=170) [所有](https://www.ssllabs.com/ssltest/viewClient.html?name=Firefox&version=73&platform=Win%2010&key=171) [的](https://www.ssllabs.com/ssltest/viewClient.html?name=Safari&version=12.1.2&platform=MacOS%2010.14.6%20Beta&key=161) [浏览器](https://www.ssllabs.com/ssltest/viewClient.html?name=Edge&version=18&platform=Win%2010&key=160) 和 [服务器](https://www.ssllabs.com/ssltest/) 都支持X25519并默认使用它，尽管也有例外，例如[nsa.gov](https://www.ssllabs.com/ssltest/analyze.html?d=nsa.gov) 更倾向于P-256。至于混合密码系统，NIST和其他标准化组织可以**要求**升级使用混合密码系统，例如对于密钥封装机制（KEMs）使用以下语言："从前量子加密升级到此KEM应保留前量子加密并添加此KEM作为额外的防御层，而不是移除前量子加密。"

在这篇博客文章中，我将探讨NSA反对混合密码系统的论点。我还将查看NSA在其[朋友](https://www.theguardian.com/uk-news/2013/aug/01/nsa-paid-gchq-spying-edward-snowden) GCHQ 的[声明](https://web.archive.org/web/20231104161635/https://www.ncsc.gov.uk/whitepaper/next-steps-preparing-for-post-quantum-cryptography) 中反混合密码系统的论点。

**NSA反混合论点。** 让我们考虑一下NSA 2022年9月常见问题解答中的以下引用：

+   混合方案给协议增加了复杂性，因为设计者需要纳入额外的协商和错误处理。

    这不是真的。一个使用签名系统的协议并不在意签名系统是否 (1) 内部使用预量子签名系统和后量子签名系统以及 (2) 内部要求这两个签名都通过验证。同样，一个使用 KEM 的协议并不在意 KEM 是否在内部将预量子和后量子会话密钥进行哈希。

    人们*可以*在协议层支持混合，并且这甚至可能有一些优势；但这不需要额外的协商，也不需要额外的错误处理，最重要的是，根本就不需要。

+   混合部署引入了额外的互操作性问题，因为现在所有参与通信的方都必须具备所有算法加上混合化方法的特性。

    这也不是真的。升级需要注意互操作性，但升级到混合 KEM 与升级到非混合 KEM 的方式相同。OpenSSH 升级到 `sntrup761x25519-sha512` 的方式与升级到假设的非混合 `sntrup761-sha512` 的方式相同。

+   “外部正在开发的许多混合解决方案并不促进向严格的抗量子解决方案的过渡，因此实施者需要预见从混合到 QR 的额外重大过渡。”

    “QR”是“抗量子”的行话，而这又是营销部门错误地声称*目标*是已经确立的*事实*。但让我们将其解读为“后量子”并关注“过渡”论点。

    想象一下以下场景。假设公开演示低成本量子攻击，表明 X25519 几乎没有安全价值。假设社区随后决定从 `sntrup761x25519-sha512` 移动到 `sntrup761-sha512`。从 `sntrup761x25519-sha512` 初始过渡到 `sntrup761-sha512`，是*两次*过渡！

    但是什么让这第二次过渡“重大”呢？这不是一个安全升级。是的，软件更改需要时间进行测试和部署，但如果这第二次过渡是一个如此严重的问题，以至于我们应该坚持只进行一次过渡，那么为什么这一次过渡应该是到`sntrup761-sha512`而不是到`sntrup761x25519-sha512`？

    NSA在这里的论点是一个令人困惑的组合，(1) 主张尽量减少过渡的数量和 (2) 主张如果过渡到混合然后随后过渡*离开*混合将很重要。除非你假设混合本身有问题，否则这是毫无意义的。换句话说，这个双过渡论点不是针对混合的独立论点。

+   "也许最重要的是，混合方案使得实现变得更加复杂，因此必须权衡在日益复杂的实现中出现缺陷的风险与密码分析突破的风险。"

    这里的情景更为紧急。假设您设计了一个使用X25519进行加密的加密应用程序。您决定从预量子加密升级到后量子加密。您明智地使用了混合方案。除了现有的X25519代码之外，还有新代码用于任何KEM——让我们称其为UltraKEM。然后，您计算一个混合哈希：这意味着对X25519会话密钥、UltraKEM会话密钥、X25519公钥和UltraKEM密文进行哈希。您使用此哈希的输出作为新的会话密钥。

    跳过混合肯定会在长远减少代码量：一旦所有客户端和服务器都支持UltraKEM，您就可以消除X25519代码和混合哈希代码。

    X25519代码可能会有bug吗？是的。与NSA使用P-256的ECDH相比，X25519更容易被正确实现，但这并不意味着有人实现X25519的风险为零。

    但是为什么NSA没有提到UltraKEM软件中的bug风险？这难道不*显然*是一个更大的风险吗？

    UltraKEM软件比X25519软件更新且更复杂。任何合理的风险评估都将得出结论：在UltraKEM软件中出现可利用的算术错误或时间泄漏或随机性使用错误的概率高于在X25519软件中出现这样的漏洞的概率。此外，如果第一种类型的漏洞发生，其影响是破坏UltraKEM的安全性（用户数据今天就暴露了），而X25519+UltraKEM混合降低了损害（用户数据暴露给了未来的量子计算机）。如果第二种类型的漏洞发生，其影响仅仅是将X25519+UltraKEM的安全性降低到UltraKEM的安全性。

    考虑一下[KyberSlash](https://kyberslash.cr.yp.to)，这是在2023年12月初大多数Kyber库中的一种时序变化。我在2023年12月30日发布了一个[demo](https://groups.google.com/a/list.nist.gov/g/pqc-forum/c/ldX0ThYJuBo/m/uIOqRF5BAwAJ)，从2023年12月初Kyber参考代码的时序中恢复了Kyber的完整密钥。我们应该相信这是后量子软件中的最后一个漏洞吗？

    让我们明确NSA需要支持其所谓的*混合逆转*的场景，即X25519+UltraKEM软件比UltraKEM软件*更*不安全。在X25519软件中需要有一个程序划分灾难，例如可利用的缓冲区溢出；或者在混合哈希代码行中需要有一些类似的毁灭性bug，例如从哈希输入中省略UltraKEM会话密钥。

    我们有工具自动检查X25519输入数据流是否会影响到任何分支条件或数组索引，所以缓冲区溢出到底会出现什么情况呢？至于混合哈希的错误，这又会怎样在互操作性测试中被忽略呢？

    我并不是在说混合逆是不可能实现的。但是NSA所用的措辞（"风险"增加复杂实现可能出现的缺陷）让读者思考X25519+UltraKEM混合的代码总量，错误地暗示任何这些代码的缺陷都是反对混合模式的论据。NSA没有承认UltraKEM代码中可能存在的缺陷会使UltraKEM比混合模式不安全，就像KyberSlash所说的那样。NSA也没有区分（1）X25519代码中可能存在的缺陷降低X25519+UltraKEM的安全性到UltraKEM的安全性，和（2）混合逆的可能性。

    或许NSA会回应说，哦，抱歉，我们没有考虑到后量子软件中的错误，并且不知道KyberSlash，而且真的，这种事有多常见呢？但是另一份文件显示，对于一些"国家安全"数据，NSA使用两个独立的加密层来"减少对手利用单一密码实现的能力"。NSA怎样将这一点与其FAQ中甚至没有提到后量子实现风险的说法相一致呢？

    与此同时，NSA的FAQ中的"突破"一词暗示后量子系统的数学突破很少见；NSA的"平衡"一词暗示这些突破风险与混合逆的风险一样。当然，加密软件中出现的问题可以和数学突破一样毁灭性（而且有些情况下修补[软件问题](https://cr.yp.to/papers.html#cachetiming)比转换到不同的加密系统[更困难](https://eprint.iacr.org/2019/996)）；但是我们为什么要认为混合逆的*概率*与后量子系统被攻破的概率一样呢？

+   "更多的安全产品因实现或配置错误而失败，而不是由于其基础加密算法的失败。因此，投入有限资源增加加密复杂性可能潜在地削弱安全性。"

    这需要澄清。究竟“产品”包括什么？NSA是否声称非密码性代码的漏洞在某种程度上算作反对混合系统的论据？如果重点是密码学：NSA是否不包括，例如，1990年代，当时["95%"](https://link.springer.com/chapter/10.1007/3-540-45539-6_1)的SSL连接使用的RSA-512，无论实施如何都是不可靠的？此外，失败是否按严重程度加权？例如，发现于2012年并利用了MD5漏洞的[Flame恶意软件](https://en.wikipedia.org/wiki/Flame_(malware))被分配了多少权重？此外，保留现有的X25519代码如何成为“花费有限资源增加密码复杂性”的例子？NSA是否在谈论编写混合哈希代码的工作？

**量化密码风险。** 让我们更仔细地看看后量子系统的数学破解会被认为是一个“突破”这一概念。

类比一下，如果有人用“突破”这个词来描述发现已发布软件中的缺陷，你会怎么想？无知得令人震惊，不是吗？

无可辩驳，程序设计是一项易出错的活动。有充分的数据来量化这一点。大型软件项目跟踪他们的[错误](https://bugzilla.mozilla.org/describecomponents.cgi?product=Firefox)。跨重点研究[研究](https://www.vuminhle.com/pdf/issta16.pdf)了多个项目中的错误，[研究](https://www.alexopoulos.ch/files/TOPS2020.pdf)了bug在不同时间段内的发现情况，[研究](https://arxiv.org/pdf/2103.07189.pdf)了不同软件工程实践对错误率的影响等等。程序员的信念常常被研究所[反驳](https://www.cs.ucdavis.edu/~devanbu/belief+evidence.pdf)。

是否有人广泛意识到设计和选择后量子密码系统的过程容易出错？读者偶尔可能会听到关于破解密码系统的新闻，或者可能会碰到一篇描述破解的论文，但读者应该如何了解这种事情有多么*普遍*？有多少研究探讨了密码系统的失败频率呢？

NSA在2021年表示["NSA对NIST PQC流程有信心"](https://datatracker.ietf.org/meeting/112/materials/slides-112-lamps-hybrid-non-composite-multi-certificate-00)。阅读NSA的2022 FAQ的典型读者最终会得出印象，即混合系统正在处理一种假设的失败模式而不是一种真实的失败模式。一个“突破”听起来不像是一个可以预料的事情。NSA没有提到SIKE。已经单独听过SIKE的读者可能很容易认为它是一个孤立的例子。

也许你会想到这样的情况：“实际上这不是一个孤立的例子吗？几乎所有的后量子提案都表现得非常出色吗？否则，我不应该听到更多有关破解的新闻吗？”

我有一篇新论文，题为[“量化密码选择过程中的风险”](https://cr.yp.to/papers.html#qrcsp)，以NIST的后量子竞赛为案例研究：

+   在2017年的竞赛中有69份第一轮提交：**48%** 现在已经被破解了，这意味着最小建议参数现在已知比AES-128更容易被破解。（AES-128是竞赛中允许的最低安全级别。）

+   在第一轮中未被破解的48份提交中：**25%** 现在已经被破解了。

+   在2019年由NIST在第二轮中*选择的28份提交*中：**36%** 现在已经被破解了。

这些故障率非常高。是的，SIKE并不是一个孤立的例子。

不要被听起来自信的推销员愚弄了。如果你遇到声称选择机制S可可靠地选择在10年内不会被破解的密码系统的人，请尝试要求提供对机制S明确定义的出版物的参考文献，以及至少10年后对机制S的定量故障率研究的出版物。如果人们仅仅指出一个未被破解的系统并声称其安全性的原因，那么他们并没有展示任何预测能力。请记住，一个*可破解*的系统在足够的时间内还未被发现攻击算法，而与此同时对其安全性的不正确声明可能只是纯粹的确认偏见所导致的。

**量化密码代码复杂性。** NSA文本中另一个未量化的方面是其声称“混合解决方案使实现更加复杂”。

如果纯粹从字面上理解这些话，那么它们显然是正确的，但显然对于做出决定是没有用的。是的，X25519+UltraKEM软件比UltraKEM软件更复杂；那又怎样？UltraKEM软件比NullKEM软件更复杂；这是否应该成为NullKEM的一个论点？（NullKEM具有空密钥、空密文和会话密钥，其中每个比特位都是0。）

读者理解NSA所说的是X25519+UltraKEM软件与UltraKEM软件之间存在*重要*的复杂性差异。 GCHQ的声明，在[下文](#gchq)中详细介绍，明确表示存在“显著”更多的复杂性。这引出了一个定量问题：我们在谈论多少软件？

我有另一篇新论文，题为[“分析后量子软件的复杂性”](https://cr.yp.to/papers.html#pqcomplexity)，以以下基于格的KEM（密钥交换机制）的参考软件为案例研究：`kyber512`、`kyber768`、`kyber1024`、`ntruhps2048509`、`ntruhps2048677`、`ntruhps4096821`、`ntruhrss701`、`sntrup653`、`sntrup761`、`sntrup857`、`sntrup953`、`sntrup1013` 和 `sntrup1277`。

该论文对每个KEM的参考软件应用一致的规则进行了简化，例如，`ntruhps4096821`为381行，`ntruhrss701`为385行，`kyber1024`为472行，`sntrup1277`为478行。还有用于哈希的子程序，例如一个67行的`hash/sha3256`，以及用于恒定时间排序（在`ntruhps`和`sntrup`的情况下），例如一个33行的`sort/uint32`。该论文还包括了进一步的度量标准（例如圈复杂度，这是一种傲慢的说法，用来讨论计算函数加上分支的方式），多个KEM尺寸的组合（例如`kyber512`加上`kyber1024`为574行），以及对代码行数如何花费的深入分析。

举例来说，从[TweetNaCl](https://tweetnacl.cr.yp.to)中提取X25519软件，并按照新论文的格式重新排版，得到了156行，这比任何一个格子KEM的大小都小了一半以上。当然，我还应该计算混合哈希的代码行数，但这些数字很容易证明我在[上文](#morecomplicated)中对于通用后量子UltraKEM的写法是正确的：“UltraKEM软件比X25519软件更新，也更复杂。”

**量化快速代码的复杂性。** 有人批评这种代码大小比较只适用于*简单*代码。那么切换到*快速*代码的应用程序呢？

对于X25519，[lib25519](https://lib25519.cr.yp.to/speed.html)中提供了许多微架构特定的优化选项，但让我们假设应用程序对约翰·哈里森的英特尔/AMD汇编语言[软件](https://github.com/awslabs/s2n-bignum/blob/main/x86/curve25519/curve25519_x25519.S)的性能满意，该软件为2197行，或者是ARM的等效[软件](https://github.com/awslabs/s2n-bignum/blob/main/arm/curve25519/curve25519_x25519.S)。这些软件在HOL Light中具有形式化验证证明，证明该软件[正确地](https://github.com/awslabs/s2n-bignum/blob/main/arm/proofs/curve25519_x25519.ml) [计算](https://github.com/awslabs/s2n-bignum/blob/main/x86/proofs/curve25519_x25519.ml)了所有输入的X25519，因此代码长度不是正确性问题：它只是反映了首次编写（和验证）代码的难度。

如果我数得没错，[libjade](https://github.com/formosa-crypto/libjade/tree/main/src/crypto_kem/kyber)中的Kyber-768 AVX2软件加上哈希子程序是4589行。（这在[大部分](https://eprint.iacr.org/2023/215)已验证。）由于代码风格略有不同，因此行数并不直接可比，但我认为直接比较会再次显示快速X25519代码的复杂性不到单个Kyber参数集快速代码的一半。

软件正确性的正式验证改变了混合系统的风险分析：在使用的程度上，它降低了软件错误的几率，因此增加了必须放在数学突破上的相对权重。（TweetNaCl中的X25519代码也经过了[验证](https://eprint.iacr.org/2021/428)。）

**GCHQ反混合论点。** 让我们继续考虑GCHQ在2023年11月的声明中的引文：

+   “PQ/T混合方案的成本比单一算法的成本更高。PQ/T混合方案将更复杂地实现和维护，并且也会更不高效。”

    “‘更复杂’部分是对NSA的回应，并且已在[上文](#complexity)中讨论过。我将在[下文](#efficiency)中更多地谈论‘效率低下’部分。‘PQ/T’是‘后量子/传统’的行话。”

+   “PQ/T混合密钥建立机制应谨慎设计，以确保混合机制不会引入额外攻击。截至2023年11月，ETSI正在开发如何做到这一点的建议。[下一段：]为了确保两个签名以健壮的方式验证，提出的PQ/T混合方案的认证可能会比用于保密性的方案复杂得多。与保密性相比，对PQ/T方案的认证的研究活动也显著较少，并且目前尚无关于如何以安全方式执行此操作的指导或共识。”

    嗯，什么？

    使用两个签名系统对消息进行签名。在验证者一侧，仅当两个签名都通过验证时才接受消息。

    如果“尚未提供指导”一词意味着这尚未写下来：例如，参见2018年[PQCRYPTO交付2.5](https://www.pqcrypto.eu/deliverables/d2.5.pdf)的第3.3节。这里没有研究活动的原因是这是一个可以轻易解决的问题。

    对于加密，["KEM combiners"](https://eprint.iacr.org/2018/024)中也提供了简单的哈希解决方案，这也是来自2018年的。我们不需要等待“ETSI的开发”。

+   “在PKI内进行PQ/T混合认证需要一个可以生成和签署传统和后量子数字签名的PKI，或者两个平行的PKI（一个用于传统数字签名，另一个用于后量子数字签名）。这种额外的复杂性以及PKI迁移的困难意味着单次迁移到完全后量子PKI比采用中间的PQ/T混合PKI更可取。”

    将PKI从例如Ed25519签名迁移到Ed25519+UltraSign签名的工作方式与将PKI从Ed25519签名迁移到非混合UltraSign签名的方式相同。关于“单次迁移”，请参见[上文](#transition)关于NSA的“过渡”论点。

+   "在未来，如果存在CRQC，传统PKC算法将不会对具有CRQC的攻击者提供额外的保护。此时，PQ/T混合方案将提供的安全性不会比单一后量子算法更多，但复杂性和开销显着增加。"

    假设存在CRQC（这是"密码学相关的量子计算机"的行话）并不足以证明经典前量子密码学提供的"没有额外保护"和"没有比单一后量子算法更安全"的结论。

    具体来说，想象一下一个演示，花费十亿美元进行量子计算可以破解一千个X25519密钥。哎呀！我们应该争取更高的安全性！我们甚至不希望十亿美元的攻击能够破解*一个*密钥！关心其数据安全性的用户将会因我们部署了后量子密码学而感到满意。但用户会说"让我们关闭X25519，使每个会话便宜一百万美元攻击"吗？我持怀疑态度。我认为用户需要看到更便宜的攻击才会同意X25519的安全价值微不足道。

    另一方面，"复杂性"在[上文](#complexity)已经讨论过，我将在[下文](#efficiency)更多地谈论"开销"。

+   "考虑到这一点，技术系统和风险所有者应该权衡PQ/T混合方案的赞成和反对理由，包括互操作性、实现安全性和协议约束，以及更复杂系统的维护成本，以及需要完成两次迁移（一次到PQ/T混合方案，再次到PQC-only算法作为未来的终结状态）。"

    这里没有新的反混合论证。

**量化密码成本。** 现在让我们考虑GCHQ未量化的效率声明。

显然，继续使用X25519会有*一些*开销：公钥多32字节，密文多32字节，以及一些CPU周期。但真正的问题是这些成本对应用是否重要。

我有另一篇新论文，题为["预测后量子加密文件系统的性能"](https://cr.yp.to/papers.html#pppqefs)，它着眼于当前使用公钥加密来加密存储的文件（例如，在微软的EFS中），并预测了该应用中各种后量子KEM的成本。

文章的一部分是收集存储、通信和CPU时间的购买成本数据，以便能够将字节和周期转换为美元。特别是，互联网数据传输成本大约是每字节2^(−40)美元，计算成本大约是每CPU周期2^(−51)美元。该论文专注于比较一个应用中各种后量子KEM增加的成本，但字节和周期的购买成本是与应用无关的，也对混合成本是否成为问题的问题有所启发。

服务器发送 32 字节并接收 32 字节用于 X25519 密钥交换会产生大约 2^(−34) 美元的数据通信成本。约翰·哈里森上面提到的软件需要大约 2^(17) 个周期来生成密钥和共享密钥，因此大约也需要 2^(−34) 美元，总成本大约为 2^(−33) 美元。

说服务器可以用一美元完成数十亿次 X25519 密钥交换并不结束分析：这引出了应用程序正在进行多少次密钥交换的问题。但让我们将这个成本与 Kyber 密钥交换的成本进行比较。

对于最小的 Kyber 选项 Kyber-512，发送 800 字节密钥并接收 768 字节密文的成本大约为 2^(−29) 美元。包括 X25519 大约增加了 7% 的成本。

我们应该相信一个应用程序可以负担得起 Kyber-512 的成本，但是*不能*负担将该成本乘以 1.07 吗？嗯，好的，这是可以想象的，但证据在哪里呢？有多少读者会期望“不那么高效”的词指的是如此小的性能差距？如果要对如此小的差距进行*任何*评论，那么怎么样*说明具体数字*，以便读者不会被误导呢？

也许 GCHQ 会试图通过指向较慢的 X25519 实现来为其主张辩护，例如上文提到的 TweetNaCl 实现。但这些实现是为了那些慢速并没有问题的应用而设计的：想象一下每个用户仅启动“仅有”一百万个会话的应用。这些应用使得 GCHQ 的成本论点变得更加脆弱。

**量化的重要性。** 我将用一些关于 NSA/GCHQ 论点中未量化主张的观察来结束这篇博客文章。

读者从 NSA 的文本中获得的最重要的信息是后量子系统的破解是罕见的。我提供了相反的数字。NSA 可以回应说，嗯，每一个这样的破解都是一个“突破”（我在 [词典](https://www.merriam-webster.com/dictionary/breakthrough) 中找到的第一个定义：“突破”是指“知识或技术上的突然进步”），而且 NSA 的文本实际上并没有声称关于*这种情况发生的频率*的任何内容。

换句话说，因为“突破”这个词没有量化，它是不可证伪的。但这个词在 NSA 的 FAQ 中起着核心作用：它影响了后量子系统被破解的风险的感知，并影响了关于混合系统的决策过程。

读者的一个有用的防御机制是留意那些*本来可以*量化但*没有*的陈述。如果 NSA 真诚地认为后量子系统很少被破解，为什么 NSA 不对此进行量化呢？如果 NSA 真诚地认为混合系统存在重要的复杂性问题，为什么 NSA 不对复杂性进行量化呢？如果 GCHQ 真诚地认为混合系统存在重要的成本问题，为什么 GCHQ 不对这些成本进行量化呢？

当然，这些是价值数十亿美元的间谍机构，他们有动机削弱加密，但这不是重点。如果你看到一位学者声称后量子系统很少被破解，问同样的问题是很有用的：为什么这个声明没有量化呢？

关于破解、复杂性和成本的实际信息对于加密决策显然很重要。不幸的是，这些信息散布在加密文献中。大多数加密系统的破解并未得到中央跟踪。有许多关于特定操作的字节计数和周期计数的论文，但在上下文中量化成本的情况却少见得多。有时会有关于代码复杂性的评论，但这离系统化还差得很远。对于怀疑的读者来说，收集 NSA 和 GCHQ 正在省略的数据是*可能*的，但这需要工作。

在 NIST 后量子竞赛初期，Tanja Lange 和我准备了一张概述幻灯片，展示了哪些提交已经被破解。我们在一系列讲座中包括了[更新版本](https://cr.yp.to/talks/2017.12.28/slides-dan+nadia+tanja-20171228-latticehacks-16x9.pdf)[（链接）](https://cr.yp.to/talks/2018.12.14/slides-dan+tanja-20181214-pqcrypto-4x3.pdf)[的](https://cr.yp.to/talks/2018.12.28/slides-dan+tanja-20181228-pqcrypto-16x9.pdf)[幻灯片](https://cr.yp.to/talks/2019.06.10/slides-dan+tanja-20190610-pqcrypto-4x3.pdf)。在 2023 年，当新的后量子签名系统提交给 NIST 时，Thom Wiggers 组建了一个[签名动物园](https://pqshield.github.io/nist-sigs-zoo/)，跟踪了其中哪些已经被破解（以及密钥大小等）。还有其他跟踪加密系统的例子。我希望这可以更加系统化，以便读者可以轻松找到关于，例如，一个加密系统在 N 年内被破解的几率的数据。必须明确规定什么算是破解，而且显然不能仅限于攻击*演示*；目标是抵御大规模攻击者，而不仅仅是抵御学术实验。必须有清晰的追踪，以便数据收集可以进行抽查。

我有幸能够花时间调查我认为重要的主题，而不必担心这是否会产生符合现有会议或期刊范围的论文。所以我有上述论文，其中包括量化加密[风险](https://cr.yp.to/papers.html#qrcsp)、[代码复杂性](https://cr.yp.to/papers.html#pqcomplexity)和[成本](https://cr.yp.to/papers.html#pppqefs)的案例研究。但我希望看到更多关于这些主题的研究工作。我希望看到密码学界向初级研究人员清楚地表明，密码学界关心这些主题，并提供了发表论文的场所。

代码复杂性和成本是传统的工程主题，应该很容易适应现有的密码工程场所，例如[CHES](https://ches.iacr.org/)和[密码工程杂志](https://link.springer.com/journal/13389)。但是加密风险分析呢？风险分析已经是软件工程文献中的常见主题，但我认为典型的软件工程场所不会将加密系统的数学破解视为范围内的内容。

一个可能性是《密码学杂志》。我在该杂志上发表了一篇有关[密码竞赛](https://cr.yp.to/papers.html#competitions)的风险分析论文。另一个可能性是IACR的新杂志，《[密码学通信](https://cic.iacr.org/)》，该杂志明确包括了其范围内的“密码学应用方面”。

最重要的是，正如NSA/GCHQ关于混合体的争论所说明的那样，制定加密决策的过程引发了定量问题。我们有责任仔细调查这些问题，以保护用户的利益。

[2024.01.09编辑：修复了到[https://kyberslash.cr.yp.to/libraries.html](https://kyberslash.cr.yp.to/libraries.html)的链接。]

* * *

**版本：** 这是20240102-hybrid.html网页的2024.01.09版本。
