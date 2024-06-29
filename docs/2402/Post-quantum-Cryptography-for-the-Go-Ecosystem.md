<!--yml

category: 未分类

date: 2024-05-27 14:31:05

-->

# 用于 Go 生态系统的后量子密码学

> 来源：[https://words.filippo.io/dispatches/mlkem768/](https://words.filippo.io/dispatches/mlkem768/)

[filippo.io/mlkem768](https://pkg.go.dev/filippo.io/mlkem768?ref=words.filippo.io) 是一款经过优化以确保正确性和可读性的纯 Go 实现的 ML-KEM-768。ML-KEM（前身为 Kyber，因为我们不能拥有好东西而改名）是一个正在由 NIST 标准化并被大多数行业采用的 [后量子](https://en.wikipedia.org/wiki/Post-quantum_cryptography?ref=words.filippo.io) 密钥交换机制。

该软件包总共有约 [~500 行代码](https://github.com/FiloSottile/mlkem768/blob/main/mlkem768.go?ref=words.filippo.io)，加上 200 行注释和 650 行测试代码。除了 golang.org/x/crypto/sha3 外，它没有任何依赖项。它旨在 [上游合并](https://go.dev/cl/550215?ref=words.filippo.io) 到 Go 标准库（最初作为一个仅内部使用的包，用于一个可选的 crypto/tls 实验），并且通过简洁性、易审查性和彻底测试来提供高安全性保证设计。

我在 [Twitch](https://twitch.tv/filosottile?ref=words.filippo.io) 上直播了它的部分开发过程，你可以在 [YouTube 上观看回放](https://www.youtube.com/watch?v=MyB7A93C-V0&ref=words.filippo.io)。

与大多数其他实现不同，此代码并非从参考 pq-crystals 库移植，而是从头开始编写，从未密切阅读其他代码库。这是一种有意的规范验证练习，以展示仅凭规范就可以生成可互操作的实现。

[FIPS 203 文档](https://nvlpubs.nist.gov/nistpubs/FIPS/NIST.FIPS.203.ipd.pdf?ref=words.filippo.io) 原来是一份优秀的实现指南，包含详细的伪代码、详尽的定义以及一致的类型信息。（这是我希望所有大型规范文档都能做到的事情：定义你的类型并使用它们并标记它们！）为了使代码更容易审查和作为学习资源更好，函数和变量名称，甚至操作顺序都精心选择以反映 FIPS 规范。

实际上，规范要求的数学背景相当有限，但为了简化实施者的工作，我撰写了《足够的多项式和线性代数来实现 Kyber》[Enough Polynomials and Linear Algebra to Implement Kyber](https://words.filippo.io/dispatches/kyber-math/)。

除此之外，留给读者作为练习的部分有

1.  实现模数为素数 3329 的算术；

1.  具体实现将压缩和解压功能映射值从 [0, 3329) 到 [0, 2ᵈ)；并且

1.  确保操作的时间保持恒定。

对于模数算术来说相当容易，因为多年来我们通过 RSA 和椭圆曲线实现学到了有限域算术的很多知识。这个小素数使得任务感觉异常简单。

[压缩和解压缩](https://nvlpubs.nist.gov/nistpubs/FIPS/NIST.FIPS.203.ipd.pdf?ref=words.filippo.io#equation.4.5)证明是项目中最困难的部分。规范将它们定义为抽象术语，如分数和舍入规则 —— “只需”计算 (2ᵈ/q)·x 或 (q/2ᵈ)·y 并四舍五入到最接近的整数 —— 但实际上我们需要用常量时间算术和位操作来实现它们！在我的[公开评论](https://groups.google.com/a/list.nist.gov/g/pqc-forum/c/n8dsN_aMsa8/m/NHO2lNyWAAAJ?ref=words.filippo.io)中，我指出让每个实现自行制定策略是冒险且多余。结果证明我比想象中更正确：[参考实现和几乎每个从它移植的实现](https://github.com/pq-crystals/kyber/commit/dda29cc63af721981ee2c831cf00822e69be3220?ref=words.filippo.io)都使用了依赖于编译器优化和平台的 DIV 指令，即使除数固定也是变时的。本软件包未受影响，因为从一开始就使用了[巴雷特约简](https://www.nayuki.io/page/barrett-reduction-algorithm?ref=words.filippo.io)，就像 BoringSSL 一样。

您可以在[pqc-forum邮件列表上阅读我的其余正式公开评论](https://groups.google.com/a/list.nist.gov/g/pqc-forum/c/n8dsN_aMsa8/m/NHO2lNyWAAAJ?ref=words.filippo.io)。

可读性是实现的一个主要目标，特别是对于像压缩和解压缩这样的复杂函数。可读性的实现有两个目的：首先，它允许在代码审查过程中和之后由感兴趣的研究人员进行有效的审查，从而提高安全性；其次，它作为下一代维护者和密码工程师（或者好奇的极客）的教育资源。阅读 Go 加密标准库是我开始走向这里之路的起点，因此对我来说，保护和改进它作为学习资源尤为重要。这显然是主观的，但我相信这是最易理解的公共 ML-KEM/Kyber 实现。例如，比较[我们的压缩/解压缩函数](https://github.com/FiloSottile/mlkem768/blob/344d5ee2c575ca84613bbb119c1d8b1ef6699ea5/mlkem768.go?ref=words.filippo.io#L394-L442)与[参考实现](https://github.com/pq-crystals/kyber/blob/11d00ff1f20cfca1f72d819e5a45165c1e0a2816/ref/poly.c?ref=words.filippo.io#L9-L112)。

有时为了改进可读性和可审查性，会使代码变得更长并且不太可重用：例如对于ML-KEM-768，我们需要以紧凑格式序列化1位、4位、10位和12位整数。编写一个通用的1到12位编码器和解码器确实是一件相当复杂的工作，但是每种大小的专用编码器/解码器实际上都很容易编写。这就是为什么我们有`ringCompressAndEncode1/4/10`等，而不是一个单一的通用函数。这也使得我们能够将一些特殊的必要检查轻松地整合到12位解码器中。

顺便提一下，这仅仅因为我们专注于ML-KEM-768，否则我们必须实现5位和11位编码。ML-KEM在三个安全级别（-512，-768和-1024）上进行了规定。然而，Kyber团队建议选择-768而不是-512以获得更保守的安全边界，而-1024之所以存在仅仅是因为256位安全级别的规定和盲目的[强度匹配](https://www.imperialviolet.org/2014/05/25/strengthmatching.html?ref=words.filippo.io)。大多数正在测试或标准化的协议都集中在ML-KEM-768上，因此仅专注于它不仅提高了可读性，还提高了安全性（因为移动部分更少）和性能（因为我们可以优化分配大小、迭代次数和编码算法），而几乎没有额外成本。

在可读性之后，测试是该软件包高安全保障策略的主要组成部分。除了检查密钥生成、封装和解封装的正确轮回，并保持95%以上的测试覆盖率外，我们

+   确保与[NIST和其他实现获取的测试向量](https://github.com/FiloSottile/mlkem768/blob/344d5ee2c575ca84613bbb119c1d8b1ef6699ea5/testdata/vectors.json?ref=words.filippo.io)的互操作性；

+   对基域算术运算（加法、减法和乘法模3329）的每种输入组合进行详尽测试，以确保结果值与使用变量时间操作轻易计算的预期值一致；

+   对比math/big.Rat对压缩和解压缩进行详尽测试（由David Buchanan贡献）。

+   测试预先计算的常数是否与其定义匹配；

+   检查所有函数的每个输入，确保不正确的长度（无论长短）都会导致适当的错误；

+   运行我们开发的广泛可重用的测试向量集（见下文）；

+   运行由[Sophie Schmieg提供的测试向量](https://groups.google.com/a/list.nist.gov/g/pqc-forum/c/aCAX-2QrUFw/m/hy5gwcESAAAJ?ref=words.filippo.io)，这些测试向量最终将包含在[Wycheproof](https://github.com/google/wycheproof?ref=words.filippo.io)中；

我们的测试向量设计为其他实现可重复使用，并作为[CCTV项目的一部分发布](https://c2sp.org/CCTV/ML-KEM?ref=words.filippo.io)，包括用于测试和调试每个中间步骤和部分算法的详细[中间值](https://github.com/C2SP/CCTV/blob/main/ML-KEM/README.md?ref=words.filippo.io#intermediate-values)。在开发过程中，我们使用了不同的测试向量集，每个都设计为涵盖不同的边缘情况。

+   [负测试向量](https://github.com/C2SP/CCTV/blob/main/ML-KEM/README.md?ref=words.filippo.io#bad-encapsulation-keys)提供无效的封装密钥，其中系数高于3329。这些向量经常被请求，因为Kyber团队和NIST团队的所有测试向量都是针对正常的、正确的输入。这些向量分别测试从3329到2¹²-1的每个值和每个系数位置，剩余系数共享，使其从1–3 MiB压缩到12–28 KiB。

+   [“不幸”向量](https://github.com/C2SP/CCTV/blob/main/ML-KEM/README.md?ref=words.filippo.io#unlucky-ntt-sampling-vector)需要异常大量的XOF读取。Kyber使用*拒绝采样*从公钥的一部分中采样矩阵：它获取一个介于0和2¹²-1之间的随机值，并检查是否小于3329，如果不是，则再次尝试。采样矩阵所需的字节数取决于采样的幸运程度，这是公钥组件随机函数的结果。这些向量是常规公钥，需要从SampleNTT中的SHAKE-128 XOF读取超过575字节，通常的概率为2⁻³⁸。Sophie的向量经过更多的暴力破解，需要多达591字节。

    > 此时我想感谢我们的检测和响应团队，他们没有让我因为大量随机种子的哈希和输出中的零而失去工作。 — Sophie Schmieg

+   特殊向量在ML-KEM.Decaps中使用`strcmp()`时[会失败](https://github.com/C2SP/CCTV/blob/main/ML-KEM/README.md?ref=words.filippo.io#strcmp-vectors)。在ML-KEM.Decaps中，密文与K-PKE.Encrypt的输出进行了隐式拒绝比较。如果实现使用`strcmp()`进行比较，则会在比较早期终止的情况下未能拒绝某些密文。我希望这个问题会悄悄成为一个陷阱多年——谁会在加密代码中使用`strcmp()`呢——然后无情地消除一个漏洞，因为当然会有人这样做。

+   [累积向量](https://github.com/C2SP/CCTV/blob/main/ML-KEM/README.md?ref=words.filippo.io#accumulated-pq-crystals-vectors)（源自参考 pq-crystals 实现）允许测试随机可达的边缘情况，而无需检查大量数据。Kyber 的参考实现包括一个 `test_vectors.c` 程序，用于生成 300MB 的随机向量。我并没有打算检查输出或编译 C 代码，但由于它们只是随机生成的向量，我们可以在测试中使用确定性 RNG（空输入的 SHAKE-128）重新生成它们，并检查它们哈希到预期值。我们甚至可以进一步，为一百万次随机测试生成哈希，超过它们生成的 10k 次。

我很高兴地报告，在 filippo.io/mlkem768 完成后引入的许多测试中，没有发现任何问题。然而，有[至少一个报告的实例](https://github.com/C2SP/CCTV/issues/7?ref=words.filippo.io)显示了负向量在主要实现中识别出的一个缺陷。

性能并非主要目标（无论是该软件包还是[Go 加密软件包](https://go.dev/design/cryptography-principles?ref=words.filippo.io)），但该软件包需要足够快以便有用。幸运的是，ML-KEM 很快，以至于这个简单的实现与我们的汇编优化的 P-256 和 X25519 实现相竞争。

为了进行公平比较，需要注意每一方需要执行的完整操作：对于 ECDH，需要两次标量乘法（其中一次是固定基点的乘法）；对于 KEMs，一方是密钥生成和解封装，另一方是封装。ECDH 是对称的，而 ML-KEM 密钥建立则不是。

下面的 ECDH 基准已经包括了两次标量乘法，而 mlkem768 基准被分成了“Alice” 下的密钥生成和解封装以及“Bob” 下的封装。因为解封装包括完整的加密（用于检查结果密文是否与输入匹配），Alice 的时间比 Bob 长得多：后者只做了一次加密，而前者则进行了一次加密、一次解密和一次密钥生成。

总而言之，“Bob” 的速度与我们的 X25519 或 P-256 相当，而“Alice”的速度不到两倍。与一些最快的 ML-KEM 实现（如 BoringSSL 和 libcrux）相比，此软件包的时间大约是它们的两倍。对于这样一个简单且未经优化的实现，这已经完全令人满意了。

```
goos: darwin
goarch: arm64
cpu: Intel(R) Core(TM) i5-7400 CPU @ 3.00GHz
pkg: crypto/ecdh
                         │   sec/op    │
ECDH/P256-8                49.43µ ± 0%
ECDH/X25519-8              77.46µ ± 0%

pkg: filippo.io/mlkem768
                         │   sec/op    │
RoundTrip/Alice-8          109.4µ ± 0%
RoundTrip/Bob-8            56.19µ ± 0%

goos: linux
goarch: amd64
pkg: crypto/ecdh
                         │   sec/op    │
ECDH/P256-4                78.88µ ± 1%
ECDH/X25519-4              115.6µ ± 2%

pkg: filippo.io/mlkem768
                         │   sec/op    │
RoundTrip/Alice-4          223.8µ ± 2%
RoundTrip/Bob-4            114.7µ ± 1% 
```

性能并不完全免费。总的来说，我遵循了高性能Go编程模式，例如尝试最小化堆分配。接下来，我[重新设计了x/crypto/sha3包](https://go.dev/cl/544817/2?ref=words.filippo.io)，这样可以在没有任何堆分配的情况下使用，这要归功于[中间堆栈内联技巧](https://words.filippo.io/efficient-go-apis-with-the-inliner/)。然而，我尚未合并这些更改，它们也不包含在上述基准测试中，因为它们对Apple M2处理器产生负面影响。目前还不清楚原因。

```
goos: darwin
goarch: arm64
pkg: filippo.io/mlkem768
                  │   sec/op    │   sec/op     vs base                │
RoundTrip/Alice-8   109.4µ ± 0%   121.3µ ± 1%  +10.91% (p=0.000 n=10)
RoundTrip/Bob-8     56.19µ ± 0%   59.94µ ± 2%   +6.66% (p=0.000 n=10)

goos: linux
goarch: amd64
                  │   sec/op    │   sec/op     vs base               │
RoundTrip/Alice-4   223.8µ ± 2%   218.6µ ± 1%  -2.32% (p=0.000 n=10)
RoundTrip/Bob-4     114.7µ ± 1%   109.5µ ± 0%  -4.57% (p=0.000 n=10) 
```

唯一成功的优化是在Gophers Slack的`#performance`频道上抱怨上述令人困惑的结果，这导致Josh Bleecher Snyder贡献了[一些变更](https://github.com/FiloSottile/mlkem768/pulls?q=is%3Apr+author%3Ajosharian&ref=words.filippo.io) :)

还有一些低悬的水果：密钥生成和解包都从相同的值中取样一个矩阵，由于这两者通常在Alice端顺序执行，所以可以存储该矩阵节约大约10%的时间。在sha3读取路径中，也许有保存副本的机会。之后，这就是优化字段实现的问题。

如果你到达这一步，你可能想在Bluesky上关注我[@filippo.abyssdomain.expert](https://bsky.app/profile/filippo.abyssdomain.expert?ref=words.filippo.io)或者在Mastodon上[@filippo@abyssdomain.expert](https://abyssdomain.expert/@filippo?ref=words.filippo.io)。

## 附加曲目：使用ML-KEM实现作为Kyber v3

NIST对Kyber的第3轮提交做了一些小的更改。这些更改总结在FIPS草案的1.3节中。

然而，有几个实验性协议是以Kyber v3（或“draft00”）的术语定义的，包括主要部署的PQ TLS密钥交换。我们是否需要制作一个单独的包来支持它们？

幸运的是，我们不。

一个变化为边缘情况添加了一些验证（公钥中非规范系数的编码），在Kyber中未定义。诚实的实现不会生成这样的密钥，因此我们可以根据FIPS草案的规定拒绝它们。这将使我们的实现可以被指纹识别为Kyber-on-ML-KEM，但其他情况下无害。

一个变化取消了应用于CSPRNG输入的哈希步骤。由于这些字节是随机的，任何一方都无法分辨出区别。

最终的变化是最重要的，也是最棘手的。密文曾经被哈希为共享密钥。这个区别将阻碍互操作性。然而，混合发生作为额外的密钥派生，完全在ML-KEM中移除，而是将值K直接返回。这意味着我们可以运行ML-KEM生成共享密钥K，然后应用

```
SHAKE-256(K || SHA3-256(c))[:32] 
```

生成Kyber共享密钥的Kyber共享密钥共享密码。不需要打破ML-KEM抽象。

有一个小问题：Kyber和ML-KEM在解密时通过将密文与密钥哈希并返回作为共享密钥来隐式拒绝。如果我们在ML-KEM上进行上述密钥派生，将会对隐式拒绝的密文进行两次哈希。这没关系，因为隐式拒绝的输出是设计上不可预测的，不是相互操作的目标。

## 图片

在柏林有一个旧的关闭的机场，[泰普尔霍夫](https://en.wikipedia.org/wiki/Berlin_Tempelhof_Airport?ref=words.filippo.io)，现在是一个公园。沿着出租车道（如图所示）或者09L/27R和09R/27L交叉跑道的中心线走有点令人不安，至少对我来说是这样。（“我应该与地面或塔台联系吗？我可以进入这条跑道吗？”）有趣的是，在2010年，一架单发飞机忘记切换燃油箱，紧急降落在27L跑道上。关闭的跑道毕竟是最好的坏着陆场。

![一条水泥跑道在日落时分拍摄的照片，从黄色中心线的中间看。机场航站楼在地平线上可见，左侧有一块草地。](img/0154902025ef34d014567ae278a1e7e9.png)

这项工作由Google [开源安全补贴](https://bughunters.google.com/about/rules/5891381450768384/open-source-security-subsidies-rules?ref=words.filippo.io) 和我的出色客户资助——[Sigsum](https://www.sigsum.org/?ref=words.filippo.io)，[Latacora](https://www.latacora.com/?ref=words.filippo.io)，[Interchain](https://interchain.io/?ref=words.filippo.io)，[Smallstep](https://smallstep.com/?ref=words.filippo.io)，[Ava Labs](https://www.avalabs.org/?ref=words.filippo.io)，[Teleport](https://goteleport.com/?ref=words.filippo.io) 和 [Tailscale](https://tailscale.com/?ref=words.filippo.io)——通过我们的保持协议获得了面对面时间和无限访问Go和密码学的建议。

以下是他们中一些人的话！

Latacora — [我们写了关于委托密码哈希的文章](https://www.latacora.com/blog/2023/12/22/case-for-password-hashing/?ref=words.filippo.io)，这是一种较少为人知的密码哈希原语。这是一个带有特殊属性的PBKDF，允许将哈希计算外包给潜在不受信任的服务器。在这篇博文中，我们描述了这种原语并讨论了它在端到端加密（E2EE）备份系统中的适用性。

Teleport — 过去五年来，攻击和妥协已从传统恶意软件和安全漏洞转向通过社会工程、凭证盗窃或网络钓鱼识别和妥协有效用户账户和凭证。[Teleport身份治理与安全](https://goteleport.com/identity-governance-security/?utm=filippo&ref=words.filippo.io)旨在通过访问监控消除弱访问模式，通过访问请求最小化攻击面，并通过强制访问审查清理未使用的权限。

[Ava Labs](https://www.avalabs.org/?ref=words.filippo.io) — 我们在[Ava Labs](https://www.avalabs.org/?ref=words.filippo.io)（Ava Labs的维护者），负责维护[AvalancheGo](https://github.com/ava-labs/avalanchego?ref=words.filippo.io)（与[Avalanche Network](https://www.avax.network/?ref=words.filippo.io)交互的最广泛使用的客户端）认为，持续维护和开发开源加密协议对于区块链技术的广泛采用至关重要。我们自豪地通过对Filippo及其团队的持续赞助来支持这一必要且有影响力的工作。
