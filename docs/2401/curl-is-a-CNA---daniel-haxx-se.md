<!--yml
category: 未分类
date: 2024-05-27 14:56:33
-->

# curl is a CNA | daniel.haxx.se

> 来源：[https://daniel.haxx.se/blog/2024/01/16/curl-is-a-cna/](https://daniel.haxx.se/blog/2024/01/16/curl-is-a-cna/)

[The curl project](https://curl.se/) has been accepted as a [CVE Numbering Authority](https://www.cve.org/ProgramOrganization/CNAs) (CNA) for vulnerabilities in all products directly made or managed by the project. If I’m counting correctly, we are the 351st CNA.

[The official announcement from Mitre](https://www.cve.org/Media/News/item/news/2024/01/16/curl-Added-as-CNA) states: *curl is now a CVE Numbering Authority (CNA) for all products made and managed by the curl project. This includes curl, libcurl, and trurl.*

In plain English, this means that we will reserve and manage our own CVEs in the future directly against the CVE database with no middle man, and also that we have a scope for CVEs that is our territory: curl and libcurl. No one else can now register CVEs for our products – without involving us. (There’s an appeals process so someone can still actually file CVEs for issues even if we say no, but at least there’s a process where both sides will argue their points.)

We do not particularly want to be a CNA but we hope that this move will make it harder to file more [stupid curl CVEs](https://daniel.haxx.se/blog/2023/08/26/cve-2020-19909-is-everything-that-is-wrong-with-cves/) in the future.