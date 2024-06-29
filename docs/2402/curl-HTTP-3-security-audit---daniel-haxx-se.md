<!--yml

category: 未分类

date: 2024-05-29 13:18:59

-->

# curl HTTP/3 安全审计 | daniel.haxx.se

> 来源：[https://daniel.haxx.se/blog/2024/02/23/curl-http-3-security-audit/](https://daniel.haxx.se/blog/2024/02/23/curl-http-3-security-audit/)

最近，[Trail of Bits](https://www.trailofbits.com/) 对 curl 的 HTTP/3 组件及其相关源代码进行了外部安全审计，特别是关注使用和接口化 ngtcp2 和 nghttp3 库的 HTTP/3 相关 curl 代码，这是迄今为止仅有的非实验性 HTTP/3 后端。审计由 [Sovereign Tech Fund](https://www.sovereigntechfund.de/) 通过 [OSTIF](https://ostif.org/) 赞助。

审计 **未发现重大发现或安全问题**，但导致改进了模糊测试，并指出了几个可以进一步改进的领域。特别是在模糊测试部门。 （如果你正在寻找想要为 curl 做贡献的地方，这里就是答案！）

审计发现我们曾在一段时间内意外地显著减少了模糊测试的覆盖范围，甚至没有注意到 - 我们当然立即纠正了这一问题。修复后，幸运的是我们没有遇到问题爆炸（哦！），这证实了在模糊测试能力受限的期间，我们并未出现特定的失误。但仍然需要指出：数周的专业代码检查并未发现严重缺陷。我对这一事实感到非常激动。

由于 curl 使用第三方库执行 QUIC 和 HTTP/3，报告建议应对涉及库的跟进审计。这是一个公平的建议，但这当然超出了我们作为项目可以做的范围。

Trail of Bits 是一支专业且愉快合作的团队。现在已经第二次合作，我对我们合作的团队只有好话。

从 curl 方面，我还想特别强调和感谢 Stefan Eissing 和 Dan Fandrich 参与了这个过程。

**完整报告可在 [curl 网站](https://curl.se/docs/audits.html) 上找到。**

## 第三

这是（非常恰当地，因为它是针对 HTTP/3 的）第三次对 curl 源代码执行的外部安全审计，即使这次的范围比 2016 年和 2022 年的前几次有所限制。相当合适地，每次新的审计中检测到的重要问题数量都在减少。我们热衷于审查，并且我们认真对待安全性。我认为这在审计报告中得到了体现。

## Related

[OSTIF 对审计的博客](https://ostif.org/curl-audit-complete/)。

## Image

顶部的图片是官方 curl 标志和官方 IETF HTTP/3 标志的混搭。由我完成。
