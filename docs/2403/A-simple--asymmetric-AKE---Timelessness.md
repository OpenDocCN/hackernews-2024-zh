<!--yml

类别：未分类

date: 2024-05-29 12:41:26

-->

# 一个简单的，非对称 AKE · 永恒性

> 来源：[https://dannyvanheumen.nl/post/simple-asymmetric-ake/](https://dannyvanheumen.nl/post/simple-asymmetric-ake/)

# 一个简单的，非对称的 AKE

Thu, Mar 28, 2024 ❝一个简单的，（迄今为止未确定的）非对称认证密钥交换❞ <details><summary>内容</summary></details>

在一个实验中，我遇到了对简单认证密钥交换的需求。它起初只是一个密钥交换的权宜之计，没有考虑任何攻击者。只是使用标准构建块的最小工作解决方案。从那里开始，我扩展了解决方案以保护免受攻击者的攻击。

## 介绍

*使用案例* 是某种用户和“服务提供者”，在我的情况下是一个设备。该设备响应请求，在一个单独的计算环境中执行计算，并且在这种特定情况下通过 USB 端口连接。涉及敏感信息。然而，设备本身没有存储能力。计算基于用户输入并由用户与设备进行交互而启动。主动权在用户手中；是否正确建立会话完全由用户的判断决定风险。

*从设备的角度* ，它只需根据请求执行操作。我们可以假设，如果用户愿意与其交互，响应肯定是安全的。或者从另一个角度来看：如果用户愿意发送他们的安全敏感信息，那么执行计算和响应就是可以的。一个保密的会话，另外还保证（唯一）另一方是发起会话的一方，这就是所需的全部。这个另一方提供数据并接收结果。

*从用户的角度* ，还有额外的考虑：用户必须确保设备是他们所期望的实际设备，即不能出现误认的身份，并且在建立安全会话后，如果决定继续，必须确保通信是安全的，即完成 AKE。

### 设备：TKey

*注意：一些反馈是由于对实际涉及的设备类型混淆所导致的。虽然我更喜欢确立一个扎实的抽象定义，但这一节应该使描述和要求更加具体。*

我为此协议考虑的具体设备是 [tillitis TKey](https://tillitis.se/products/tkey/ "TKey - Tillitis"). 这些设备包含一个不可访问的设备秘密（*UDS*），并加载任何提供的程序。设备本身有一个小型固件，能够初始化并接收作为程序二进制的字节。当程序加载时，会生成一个特定于设备 + 程序二进制 + 用户提供的秘密的派生密钥（*CDI*）。然后固件跳转到程序的入口点以启动执行，并随后将控制权交给程序。

*TKey*没有存储能力。程序有几个（硬件）功能可用，并且*CDI*作为唯一的秘密。*CDI*的值是[*UDS*、*program-binary*和*user-supplied secret*的哈希计算摘要](https://dev.tillitis.se/intro/#measured-boot--secrets "Introduction - TKey Developer Handbook")。因此，程序必须是字节精确的二进制，这为防止恶意程序窃取此秘密提供了有趣的机会。使用*CDI*和哈希函数，例如固件中提供的*Blake2s*，程序可以推导出多个秘密。如协议所述的*identity*将是这样的一个秘密值。因此，*identity*极有可能证明设备、程序二进制和用户提供的秘密值是预期的值。

关于设备安全、秘密安全、编程中的潜在漏洞等方面的任何关注都是有效的批评。然而，本文的范围仅限于协议本身。

## 设计

**要求**：

+   客户端和设备之间建立了两方安全会话，

+   受到中间人攻击的保护，

+   临时会话密钥，

+   设备已经过身份验证，

+   客户端不需要认证，

+   不需要会话ID，

**前提条件：客户端身份（假设）不存在**：设备仅从自身的程序状态开始，这对其自身（内部）操作是必要的。设备处理的任何数据都是由客户端提供的。这有一个好处，即它不会错误地暴露敏感数据或者在建立会话时误认用户的身份。设备与发起会话的另一方提供的任何数据一起工作。客户端仅在安全会话确保仅有对方能够在建立的会话中通信时（隐含地）得到验证。

**前提条件：会话范围状态**：设备的状态必须是会话范围的，以确保在会话之间没有数据泄露或者在适用的情况下没有溢出到下一个会话。

## 协议

下文建立在Diffie-Hellman密钥交换基础上，对于存在攻击者的情况进行了最小的实际适应。该协议已被证明适用于被动攻击者，试图证明针对主动攻击者的尝试当前因技术问题而停滞。

使用：

+   前缀`pk_`表示公钥，以及

+   前缀`sk_`表示秘密/私钥

+   `digest = HASH(key, content)`

+   `ciphertext = ENCRYPT(key, plaintext)`

+   `plaintext = DECRYPT(key, ciphertext)`

+   `signature = SIGN(secret_key, content)`

+   `VALIDATE(public_key, signature, content)`

**先决条件**：先前获取的身份`pk_identity`。

```
User                                                              Device
========================================================================
knows `pk_identity`                   knows `sk_identity`, `pk_identity`
generates `sk_user`, `pk_user`

User ----------------------------[pk_user]----------------------> Device

                                      generates `sk_device`, `pk_device`
                         sessionkey = HASH(pk_user^sk_device, pk_device)
                                  signature = SIGN(sk_identity, pk_user)
                                enc_sig = ENCRYPT(sessionkey, signature)

User <----------------------[pk_device,enc_sig]------------------ Device

sessionkey = HASH(pk_device^sk_user, pk_device)
proof = DECRYPT(sessionkey, enc_sig)
VALIDATE(pk_identity, proof, pk_user) 
```

当验证成功时，执行才算成功。任何损坏都将导致无法验证签名。鉴于所有风险均由用户承担，任何验证失败都是终止的理由。

**执行成功后**：

+   建立了一个带有共享的短暂会话密钥的机密会话。

+   我们确信 `pk_device` 是真实的。

+   我们确信 `pk_user` 未经更改地收到了。

+   我们确信这是真实的设备，即证明了对身份的所有权。

**执行失败**：

+   收到的 `pk_user` 是无效的/损坏的。

+   收到的 `pk_device` 是无效的/损坏的。

+   `sessionkey` 由于 `pk_device` 的缘故而无法生成。

+   由于验证 `pk_user` 失败：

    +   `proof` 不是真实的 `signature`：

        +   不正确的 `sessionkey`。

        +   由于损坏的 `enc_sig`。

        +   对损坏的 `pk_user` 进行签名。

    +   `device` 没有拥有 `sk_identity`。

注意：协议不反映使用前缀/上下文来防止签名用于除识别之外的其他目的。在使用时可能需要考虑这一点。

### 洞察

+   用户的 D-H 公钥同时作为挑战-随机数，因此没有机会用已知签名交换出随机数，并且不是自由选择的，因为那需要相应的私钥。

+   挑战-响应签名使用建立的会话密钥加密，从而防止响应消息的（原始）重放。

+   设备的 D-H 公钥还用作键控散列的输入，D-H 的结果共享秘密作为密钥。因此，共享秘密和设备的公钥都是固定的。在两方设置中，如果希望计算相同的共享秘密，则用户的公钥必须也是固定的。

+   鉴于没有持久性且客户端预计要启动并提供数据，设备只需依赖安全会话来保持数据的保密性。

## 分析

此时，我尚未对协议进行全面分析。以下内容仅基于直觉和非正式推理，受 eCK 安全模型概念的指导，并在符号推理器的部分结果后（部分）确认。

根据扩展的 Canetti-Krawczyk（eCK）模型进行认证密钥交换的安全属性。

+   *(隐含) 密钥协定*

    +   `client`/`device`：通过使用短暂密钥交换建立双方安全会话。

+   *密钥确认*

    +   `client`：通过成功的对 `pk_user` 的签名验证的暗示。

    +   `device`：由客户端跟进的暗示。

+   *已知密钥的安全性*

    +   会话密钥完全基于随机数据，双方都有贡献。

+   *防止未知密钥共享攻击的安全性*

    +   只有两个参与者参与密钥交换。正确使用 D-H 可以防止中间人攻击。只有两个参与方，因此不需要共享或传递密钥。

    +   `device`：假设已建立安全会话的任何对应部分有效。进一步证明了所有数据都是由会话对应方提供的。适当的密钥交换可以防止任何第三方的干扰。

    +   `client`：不需要共享密钥。每个会话都是在一个 `device` 和一个 `client` 之间建立的。

+   *防止密钥泄露身份冒充（KCI）攻击的安全性*

    +   身份密钥对仅用于签署发送给客户端的挑战nonce。因此，不存在签名可能被误解的机会。

+   *前向保密*

    +   AKE是以基于随机数据的输入启动的，预共享的*身份公钥*是唯一的例外。

尽管eCK模型（大概率）在精神上而非严格上得到遵循，因为这是一个非对称AKE，在*密钥确认*的情况下，例如，`device`没有得到具体的确认。实际上，这些要求不会带来问题。

## 符号证明器

以下[Verifpal](https://www.verifpal.com/ "Verifpal: Cryptographic Protocol Analysis for Students and Engineers")证明脚本对被动攻击者通过。如果协议在没有失败的情况下完成，则客户端与设备建立的会话是成功的。

请注意，在完成时刻，只有`client`知道结果，并且是否可以安全继续。这是可以接受的，因为`client`必须确信`device`是真实的，并且通信是安全的，然后才能请求`device`的服务并发送数据。

Verifpal证明脚本：

```
attacker[passive]
principal Device[
	knows private identity_secret
	identity = G^identity_secret
]
// Assume knowledge of identity public key prior to initiation of the AKE.
// (Modeled here as a guarded/guaranteed transfer: '[identity]')
Device -> User: [identity]
principal User[
	generates dhe_user_secret
	dhe_user_public = G^dhe_user_secret
]

// Initiation of a secure session.
User -> Device: dhe_user_public
principal Device[
	generates dhe_device_secret
	dhe_device_public = G^dhe_device_secret
	ss_device = dhe_user_public^dhe_device_secret
	sessionkey_device = HASH(ss_device, dhe_device_public)
	signature = SIGN(identity_secret, dhe_user_public)
	proof_encrypted = ENC(sessionkey_device, signature)
]
Device -> User: dhe_device_public,proof_encrypted
principal User[
	ss_user = dhe_device_public^dhe_user_secret
	sessionkey_user = HASH(ss_user, dhe_device_public)
	proof = DEC(sessionkey_user, proof_encrypted)
	_ = SIGNVERIF(identity, dhe_user_public, proof)?
]

queries[
	confidentiality? identity_secret
	confidentiality? dhe_device_secret
	confidentiality? dhe_user_secret
	confidentiality? ss_device
	confidentiality? ss_user
	confidentiality? sessionkey_device
	confidentiality? sessionkey_user
	freshness? dhe_user_secret
	freshness? dhe_device_secret
	freshness? ss_device
	freshness? ss_user
	freshness? sessionkey_device
	freshness? sessionkey_user
	equivalence? ss_user, ss_device
	equivalence? sessionkey_user, sessionkey_device
	equivalence? proof, signature
	// Authentication-queries provide no significance with a passive attacker.
	authentication? User -> Device: dhe_user_public
	authentication? Device -> User: dhe_device_public
	authentication? Device -> User: proof_encrypted
] 
```

当前与“*主动攻击者*”分析的问题是，符号证明器将在成功操纵中间值（例如`ss_device`，`ss_user`等）时过早失败。论点是机密性被破坏。

在协议完成（成功）之后应执行查询。`SIGNVERIF(identity, dhe_user_public, proof)?`是一个已检查的（`?`）验证语句，对于`dhe_user_public`的任何操作都应该失败，因为签名是针对由`client`生成的原始/真实`dhe_user_public`进行验证。然而，它没有。因此，它继续列出所有可能由于破坏传输的`dhe_user_public`而导致中间变量的机密性违规。

通过我的（非正式）推理，如果攻击者破坏了任何数据，`client`的最终签名验证必须失败。因此，如果最终验证成功，必须使用了真实数据，并且攻击者没有操纵建立的安全会话。

### 开放问题

[问题已报告](https://lists.symbolic.software/pipermail/verifpal/2023/000454.html "[Verifpal] Checked SIGNVERIF passed, but checked ASSERT fails (was: Mistake or missed case? SIGNVERIF with original of sent value)")给了Verifpal。不幸的是，在写作时，还没有任何跟进。作为电子邮件对话的一部分，使用了简化的示例来进一步缩小问题范围。

当分析存在主动攻击者时，协议会产生一个永远不应该发生的结果。主张是：使用`nil`（损坏的）公钥，`nil`消息和`nil`签名，验证将通过。这当然是正确的。然而，由于`client`本身生成`pk_user`，因此没有必要插入任何接收到的（潜在损坏的）值。因此，失败的情况根本不会发生。

## 身份挑战的理由

我没有讨论过的一个特定设计选择是使用Diffie-Hellman公钥进行身份挑战。这似乎是一个不太常见的选择。其背后的理由如下；让我们讨论一下可能发生的各种情况：

注意，这里的“*身份的证明*”是对`client`的D-H公钥的签名。

1.  一个合法的设备及其（真实的）密钥对会生成身份的证明。

    *这是预期的使用情况，没有恶意活动。*

1.  对手（Mallory）冒充了预期的设备，却没有访问*身份*密钥对。

    *Mallory冒充`device`，使`client`以为自己在与`device`交流。由于没有访问密钥对的权限，Mallory只能访问以前在他与`device`自己的会话中生成的签名。鉴于传输时签名是加密的，Mallory只能访问到在他自己与`device`会话中生成的签名。在没有访问身份密钥对的情况下提供正确的签名，类似于拥有D-H密钥对的相应私钥，考虑到密钥空间的大小。*

1.  在`client`和`device`之间的会话中进行的中间人攻击。

    *如果Mallory能够执行这种攻击，那么他拥有与`client`相同的临时密钥对。因此，他可以用相同的密钥对与`device`建立自己的会话，并将从`device`收到的签名证明转发给`client`。为此，Mallory需要访问此时`client`生成的密钥对，通过解决计算Diffie-Hellman问题或侵入`client`进程的内存等类似侵入性操作来获取。*

1.  身份密钥对已知/暴露；Mallory拥有`device`的密钥对。

    *Mallory可以完全冒充`device`。由于控制了密钥对，Mallory可以响应来自`client`的任何D-H公钥挑战并提供证明签名。*

第一个情况是正常的使用情况，正如预期的那样。最后两种情况涉及到破坏基础加密机制或主动攻击，这两者都是在协议之外采取的行动。这两种情况都涉及到非对称加密，因此特别容易受到量子计算攻击的影响。然而，目前这些攻击还不可能。

剩下的情况（2.）意味着我们让证明身份与破解密钥交换的安全性同样复杂化。确实，如果合法客户端与`device`使用相同的密钥连接两次，我们可以用第一次会话的证明签名作为响应。然而，“*新鲜性*”的临时会话密钥也是一个要求。请注意，不保存签名对`client`是有利的，攻击者没有权限（加密）签名，没有能力窃听会话。理想情况下，我们永远不会两次使用相同的密钥对。对于攻击者来说，除非他能操纵`client`的选择，他们的公钥选择应是不可预测的，因此窃听是不可能的。他不能计算相应的私钥，也不能收集相应的签名。（请注意，这意味着临时D-H公钥对于攻击者和`device`都应该是不可预测的，这是互补的。）

## 反馈

这篇文章在[Hacker News](https://news.ycombinator.com/item?id=39813537 "一个简单的，（迄今为止未识别的）不对称认证密钥交换（Hacker News）")上讨论。

到目前为止，大多数反馈都源于对情况的最初误解。因此，涉及*认证*、困惑于设备如何在没有任何存储的情况下工作以及对身份验证的需求等主题就会出现。我已经通过添加关于特定设备部分的内容来回应对介绍不清楚的直接评论。我期望这足以使读者对抽象描述和需求与实际用例的期望保持一致。

接下来是几条评论，解决了那些有类似疑问的读者的反馈。

+   *身份*用于识别特定设备实例，而不是物理硬件的一部分。出于同样的原因，即使是对*认证*的批评也不适用于这里。*身份*比认证过程的目标更具体。

+   类似地，对*身份*有多独特，什么会导致它改变，以及当这个*身份*泄露时的风险和影响，也存在困惑。同样，到目前为止，这些困惑/信息不足并不意味着对协议本身的批评。

+   检查真实硬件的方法，很可能比完整的认证过程更简单，并且在理解正确的情况下不适用完整的证书链，已经提供。所描述的协议解决了一个假设硬件是可信的问题。（至少在一开始。替换/移除硬件的攻击仍然被认为是有效的攻击。）

+   为什么不使用已建立的加密技术？这是一个合理的问题。最为普遍的建议是使用TLS，这是一个相当复杂的协议，假设PKI证书基础设施是可用的。另一个建议是Noise协议，我会再次仔细看看。

+   关于缺乏随机数生成器的担忧。这进一步澄清了。没有随机数生成器。固件中已经有一个“真随机数生成器”作为熵源，以及一个可用的哈希函数。至少可以近似实现一个CSPRNG。

+   对于缺少证书链的担忧。这源于对设备信息不足。设备的验证已经是一个提供的功能。

+   其他建议暗示着与设备制造商协调认证。然而，这偏向于集中化、协调的方法和/或资格过程，因此使得几个假设和权衡成为了使建议可行的基础。

+   对于第一次使用即信任（TOFU）的批评同样根源于对源密钥何时可用以及其唯一性程度的误解。

+   对于Verifpal是否是符号证明器的良好选择的批评：显然，如果问题没有解决或其他问题仍然存在，它并不是一个好的选择。可以切换到其他求解器。然而，并没有理由简单因为它不为人知就驳回任何求解器。此外，证明用作确认，旁边是手动分析。

综合我所阅读的反馈，目前尚无迹象表明协议本身存在问题。许多评论严格来说与协议无关。我决定增加一段解释预期设备的具体细节。

## 变更日志

如果有必要，本文将会更新。

+   *2024-03-28* 添加了有关具体设备描述、身份挑战理由、讨论反馈的部分。

+   *2024-03-23* 拼写错误、格式。

+   *2024-03-23* 初始版本。
