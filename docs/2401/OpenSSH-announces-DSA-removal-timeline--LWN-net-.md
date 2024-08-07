<!--yml

category: 未分类

date: 2024-05-27 14:43:18

-->

# OpenSSH 宣布 DSA 算法移除时间表 [LWN.net]

> 来源：[`lwn.net/Articles/958048/`](https://lwn.net/Articles/958048/)

对于仍在使用 SSH 的 DSA 密钥的用户们：该项目已宣布计划在 2025 年初删除对该算法的支持。

> 目前唯一剩下的使用 DSA 的情况应该是非常老旧的设备。因此，我们不再认为在 OpenSSH 中维护 DSA 的成本是合理的。此外，我们希望 OpenSSH 最终移除这种不安全的算法能够加速其他 SSH 实现中对其的废弃，并使密码库的维护者也能将其移除。

* * *

| **From**: |   | Damien Miller <djm-AT-mindrot.org> |
| --- | --- | --- |
| **To**: |   | openssh-unix-dev-AT-mindrot.org |
| **Subject**: |   | 公告：OpenSSH 移除 DSA 支持时间表 |
| **Date**: |   | Thu, 11 Jan 2024 15:40:37 +1100 |
| **Message-ID**: |   | <c857fb2e-a1a8-cfaf-92ab-ea603cb125c1@mindrot.org> |

```

Hi,

OpenSSH plans to remove support for DSA keys in the near future. This
message describes our rationale, process and proposed timeline.

Rationale
---------

DSA, as specified in the SSHv2 protocol, is inherently weak - being
limited to a 160 bit private key and use of the SHA1 digest. Its
estimated security level is <=80 bits symmetric equivalent[1][2].

OpenSSH has disabled DSA keys by default since 2015 but has retained
optional support for them. DSA is the only mandatory-to-implement
algorithm in the SSHv2 RFCs[3], mostly because alternative algorithms
were encumbered by patents when the SSHv2 protocol was designed and
specified.

Since then, the world has moved on. RSA is unencumbered and support
for it is ubiquitous. ECDSA offers significant performance and
security benefits over modp DSA, and EdDSA overs further performance
and security improvements over both again.

The only remaining use of DSA at this point should be deeply legacy
devices. As such, we no longer consider the costs of maintaining DSA
in OpenSSH to be justified. Moreover, we hope that OpenSSH's final
removal of this insecure algorithm accelerates its deprecation in
other SSH implementations and allows maintainers of cryptography
libraries to remove it too.

Process and timeline
--------------------

The next release of OpenSSH (due around 2024/03) will make DSA
optional at compile time, but still enable it by default. Users and
downstream distributors of OpenSSH may use this option to explore the
impact of DSA removal in their environments, or to hard-deprecate it
early if they desire.

Around 2024/06, a release of OpenSSH will change this compile-time
default to disable DSA. It may still be enabled by users/distributors
if needed.

Finally, in the first OpenSSH release after 2025/01/01 the DSA code
will be removed entirely.

In summary:

2024/01 - this announcement
2024/03 (estimated) - DSA compile-time optional, enabled by default
2024/06 (estimated) - DSA compile-time optional, *disabled* by default
2025/01 (estimated) - DSA is removed from OpenSSH

Questions
---------

 * What if I have devices that only support DSA?

Removing DSA from OpenSSH will not remove endpoints that require DSA
from the world and users may still need to connect to them. Although
new releases of OpenSSH will no longer support DSA, past releases and
alternate SSH implementations will continue to do so.

We recommend that users with an ongoing need to connect to DSA-only
endpoints maintain a legacy release of an OpenSSH client for this
purpose, similar to what was recommended when support for the SSHv1
protocol was removed.

For example, Debian maintains a "openssh-client-ssh1" package built
from OpenSSH 7.5 for the purpose of connecting to SSHv1 endpoints.
This package or something similar is likely to be sufficient for
DSA-only endpoints too.

 * Doesn't this make OpenSSH non-compliant with RFC4253?

Practically, no more than we've been since 2015 when we stopped
offering DSA by default.

 * Why make this change now? Why not earlier/later?

We feel like enough time has passed since DSA was disabled by default
for the overwhelming majority of users to have abandoned use of the
algorithm. We are also likely to start exploring a post-quantum
signature algorithm soon and are mindful of the overall size and
complexity of the key/signature code.

 * I want to discuss this change further

The https://lists.mindrot.org/mailman/listinfo/openssh-unix-dev
mailing list is the best place to discuss this. Alternately you can
email the OpenSSH developers at openssh@openssh.com.

Thanks,
Damien Miller, on behalf of the OpenSSH project

[1] https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIS...
[2] https://www.rfc-editor.org/rfc/rfc9142.html#section-1.1
[3] https://www.rfc-editor.org/rfc/rfc4253.html#section-6.6
```

* * *

(

[登录](https://lwn.net/Login/?target=/Articles/958048/)

以发表评论)
