<!--yml

分类：未分类

日期：2024 年 05 月 27 日 14:56:33

-->

# [curl 是一个 CNA](https://daniel.haxx.se/) | daniel.haxx.se

> 来源：[`daniel.haxx.se/blog/2024/01/16/curl-is-a-cna/`](https://daniel.haxx.se/blog/2024/01/16/curl-is-a-cna/)

[curl 项目](https://curl.se/)已被接受为漏洞的 [CVE 编号管理机构](https://www.cve.org/ProgramOrganization/CNAs) (CNA)，适用于由该项目直接制造或管理的所有产品。如果我数得没错，我们是第 351 个 CNA。

[来自 Mitre 的官方公告](https://www.cve.org/Media/News/item/news/2024/01/16/curl-Added-as-CNA) 表示：*curl 现在是所有由 curl 项目制造和管理的产品的 CVE 编号管理机构（CNA）。这包括 curl、libcurl 和 trurl。*

用通俗的话来说，这意味着我们将来将直接对 CVE 数据库保留和管理我们自己的 CVE，并且我们的 CVE 范围是我们的领土：curl 和 libcurl。现在，没有其他人可以为我们的产品注册 CVE - 而不涉及我们。 (虽然有上诉程序，所以即使我们说不，某人仍然可以提交关于问题的 CVE，但至少有一个过程，双方将辩论他们的观点。)

我们并不特别想成为 CNA，但我们希望这一举措能让未来更难再提出更多 [愚蠢的 curl CVEs](https://daniel.haxx.se/blog/2023/08/26/cve-2020-19909-is-everything-that-is-wrong-with-cves/)。
