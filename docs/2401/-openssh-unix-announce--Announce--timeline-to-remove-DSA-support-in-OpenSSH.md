<!--yml

类别：未分类

日期：2024-05-27 14:41:47

-->

# [openssh-unix-announce] 宣布：移除 OpenSSH 中对 DSA 的支持时间表

> 来源：[https://lists.mindrot.org/pipermail/openssh-unix-announce/2024-January/000156.html](https://lists.mindrot.org/pipermail/openssh-unix-announce/2024-January/000156.html)

# [openssh-unix-announce] 宣布：移除 OpenSSH 中对 DSA 的支持时间表

**djm at openssh.com** [djm at openssh.com](mailto:openssh-unix-announce%40mindrot.org?Subject=Re%3A%20%5Bopenssh-unix-announce%5D%20Announce%3A%20timeline%20to%20remove%20DSA%20support%20in%0A%20OpenSSH&In-Reply-To=%3Cfac472a8f9544382%40cvs.openbsd.org%3E "[openssh-unix-announce] 宣布：移除 OpenSSH 中对 DSA 的支持时间表")

*Thu Jan 11 15:31:18 AEDT 2024*

* * *

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
email the OpenSSH developers at openssh at openssh.com.

Thanks,
Damien Miller, on behalf of the OpenSSH project

[1] https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-57pt1r5.pdf
[2] https://www.rfc-editor.org/rfc/rfc9142.html#section-1.1
[3] https://www.rfc-editor.org/rfc/rfc4253.html#section-6.6

```

* * *

* * *

[有关 openssh-unix-announce 邮件列表的更多信息](https://lists.mindrot.org/mailman/listinfo/openssh-unix-announce)
