<!--yml

category: 未分类

date: 2024-05-27 14:31:39

-->

# Mullvad 对凯伯的使用不受 KyberSlash 的影响 | Mullvad VPN

> 来源：[https://mullvad.net/en/blog/mullvads-usage-of-kyber-is-not-affected-by-kyberslash](https://mullvad.net/en/blog/mullvads-usage-of-kyber-is-not-affected-by-kyberslash)

凯伯的某些实现的漏洞，即量子抗性密钥封装机制，[最近被披露](https://groups.google.com/a/list.nist.gov/g/pqc-forum/c/ldX0ThYJuBo/m/uIOqRF5BAwAJ)。Mullvad 的量子抗性隧道不受此漏洞或任何类似漏洞的影响。

名为 KyberSlash1 和 KyberSlash2 的两种基于时间的攻击是建立在某些凯伯实现不以恒定时间执行关键操作的事实上的。如果一个服务允许攻击者向同一个密钥对请求许多此类操作，那么攻击者就可以测量时间差异，并逐渐计算出密钥。

这种基于时间的漏洞在密码学中相当常见。这就是为什么 Mullvad 的量子抗性隧道协议设计成这样的，以至于这整个类别的漏洞都是无法利用的。

Mullvad 应用程序为每个量子抗性隧道连接计算全新的密钥对。在两个隧道或两个不同用户之间永远不会重复使用任何秘密密钥材料。因此，每个密钥只用于单个封装操作，因此不存在可以测量时间差异的情景。因此，无论 Mullvad 应用程序和服务器使用的凯伯实现是否容易受到 KyberSlash1 和 KyberSlash2 的影响，都不存在可以利用的情景。

Mullvad 的量子抗性共享密钥交换的密钥对是在*客户端*上生成的，并且只有客户端要建立连接的 WireGuard 服务器才能向其发送密文。因此，永远不会公开暴露任何可能遭到攻击者接触的密钥封装操作可以请求的端点。一切都发生在客户端和 WireGuard 服务器之间的加密 WireGuard 隧道内。

作为额外的安全层，我们的量子抗性隧道不仅依赖于凯伯。我们使用两种量子安全的密钥封装机制（凯伯和经典的麦克利斯），并混合两者的密钥。这意味着只有在 VPN 隧道的安全性受到影响之前，两种算法都必须具有**可利用的**漏洞。
