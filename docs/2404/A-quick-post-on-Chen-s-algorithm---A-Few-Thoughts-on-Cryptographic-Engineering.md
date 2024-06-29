<!--yml

category: 未分类

date: 2024-05-27 13:14:30

-->

# 陈氏算法简评 – 密码工程的几点思考

> 来源：[https://blog.cryptographyengineering.com/2024/04/16/a-quick-post-on-chens-algorithm/](https://blog.cryptographyengineering.com/2024/04/16/a-quick-post-on-chens-algorithm/)

**更新** **（4月19日）：** *陈一磊[宣布发现了一个漏洞](http://www.chenyilei.net/)，他不知道如何修复。这一发现也被洪逊吴和托马斯·维迪克独立验证。目前，该论文未提供解决LWE问题的多项式时间算法。*

如果你是一个普通人 — 指的是不痴迷于最新密码学消息的人 — 你可能错过了上周的密码学重大新闻。这则消息来自陈一磊的新论文，“[格问题的量子算法](https://eprint.iacr.org/2024/555)”，这篇论文已经在密码学研究界掀起波澜。现在，专家们正在评估这一结果，尤其是在格与量子算法设计方面（*我不是专家！*），但如果证明是正确的，那对应用密码学界将是非常不利的一天/一周/一个月/一年。

不多说了，以下是对陈氏算法的背景介绍的五个要点。

(1) 密码学家喜欢基于被认为是“难”的数学问题构建现代公钥加密方案。在实践中，我们需要具有特定结构的问题：对于持有秘钥或“陷阱门”的人，我们可以构建*高效*的解决方案，但对于没有秘钥的人来说，却没有*高效*的解决方案。虽然许多问题已被考虑过（通常被丢弃），但今天我们使用的大多数方案都基于三个问题：[因子分解](https://en.wikipedia.org/wiki/Integer_factorization)（RSA密码系统）、[离散对数](https://en.wikipedia.org/wiki/Discrete_logarithm)（Diffie-Hellman、DSA）和[椭圆曲线离散对数问题](https://en.wikipedia.org/wiki/Elliptic-curve_cryptography)（EC-Diffie-Hellman、ECDSA等）。

(2) 虽然我们愿意相信我们喜欢的问题在根本上是“难”的，但我们知道这并不完全正确。研究人员[已经设计出算法](https://en.wikipedia.org/wiki/Shor%27s_algorithm)，相当高效地解决了所有这些问题（即在多项式时间内） —— 只要有人找出如何构建足够强大的量子计算机来运行攻击算法。*幸运的是，目前还没有建成这样的计算机！*

(3) 尽管量子计算机尚未足够强大，无法破解我们的公钥加密，但*未来*量子攻击的威胁已经激励行业、政府和学术界，像《指环王》中的联盟一样，[加入力量](https://csrc.nist.gov/projects/post-quantum-cryptography)，立即解决这个问题。这不仅仅是未来保护我们系统的问题：即使量子计算机需要几十年才能建成，未来的量子计算机也能破解我们*今天*发送的加密消息！

(4) 这项奖学金显著的成果之一是[NIST的后量子密码学（PQC）竞赛](https://www.nist.gov/news-events/news/2022/07/nist-announces-first-four-quantum-resistant-cryptographic-algorithms)：这是一场旨在标准化*“后量子”密码方案*的公开竞赛。至关重要的是，这些方案必须基于不同的数学问题 —— 尤其是那些*看起来*不适合高效量子解决方案的问题。

(5) 在这一系列新方案中，最流行的一类方案基于与称为*[格](https://en.wikipedia.org/wiki/Lattice-based_cryptography)*的数学对象相关的问题。NIST批准的基于格问题的方案包括[Kyber](https://en.wikipedia.org/wiki/Kyber)和[Dilithium](https://pq-crystals.org/dilithium/)（我最近在[博客](https://blog.cryptographyengineering.com/2023/11/30/to-schnorr-and-beyond-part-2/)中有过相关描述）。格问题也是几种高效的[全同态加密](https://blog.cryptographyengineering.com/2012/01/02/very-casual-introduction-to-fully/)（FHE）方案的基础。

这些背景为新结果的呈现铺平了道路。

陈的（尚未经同行评审的）[预印本](https://eprint.iacr.org/2024/555)声称提出了一种新的量子算法，能够高效解决具有特定参数的格中的“最短独立向量问题”（SIVP，以及GapSVP）。如果验证通过，这个结果可能（在许多重要的条件限制下）允许未来的量子计算机破解依赖于这些问题特定实例的方案。好消息是，即使结果正确，易受攻击的参数也是非常具体的：陈的算法并不*立即*适用于[最近标准化的NIST算法](https://www.nist.gov/news-events/news/2022/07/nist-announces-first-four-quantum-resistant-cryptographic-algorithms)如Kyber或Dilithium。此外，算法的具体复杂性并不立即清楚：即使量子计算机变得可用，它可能也是不切实际的。

但是在我们的领域有一句话：*攻击只会变得更好*。如果陈的结果可以改进，那么量子算法可能会使整个“后量子”基于格的方案过时，迫使密码学家[和](https://security.apple.com/blog/imessage-pq3/) [行业](https://signal.org/blog/pqxdh/) [重新](https://bughunters.google.com/blog/5108747984306176/google-s-threat-model-for-post-quantum-cryptography)设计。

换句话说，这既是一项重大的技术成果，也可能是一个轻微的灾难。

如前所述：我既不是格基密码学的专家，也不是量子计算的专家。那些*是*这些领域的人非常忙，试图验证这篇**写作的正确性**，而一些重要的结果在详细检查后已经崩溃了。对于那些寻找最新进展的人，这里有一个[nice writeup by Nigel Smart](https://nigelsmart.github.io/LWE.html)，不讨论量子算法的正确性（请查看底部的更新），但确实谈到了对全同态加密（FHE）和后量子密码（PQC）方案可能影响的可能后果（总结：对某些FHE方案不利，但实际影响取决于算法运行时间的具体细节）。这里还有另一个[brief note on a "bug" that was found in the paper](https://eprint.iacr.org/2024/583.pdf)，似乎已经被作者[addressed by the author](http://www.chenyilei.net/)快速解决。

直到这周，我本打算再写一篇关于复杂性理论、格子及其对应用密码学意义的冗长文章。但现在希望你能原谅我，如果我再稍微持有一下这篇。
