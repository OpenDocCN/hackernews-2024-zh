<!--yml

类别：未分类

日期：2024-05-27 14:30:34

-->

# 查看 HSTS 文件的简要概况 – Terence Eden’s Blog

> 来源：[https://shkspr.mobi/blog/2024/01/a-quick-look-inside-the-hsts-file/](https://shkspr.mobi/blog/2024/01/a-quick-look-inside-the-hsts-file/)

您在浏览器的地址栏中输入 `example.com`，它会自动将您重定向到 https:// 版本。您的浏览器如何知道需要请求网站的更安全版本？

答案是... 一个 *庞大* 的列表。[HTTP 严格传输安全性](https://zh.wikipedia.org/wiki/HTTP%E4%B8%A5%E6%A0%BC%E4%BC%A0%E8%BE%93%E5%AE%89%E5%85%A8%E6%80%A7)（HSTS）列表是一个域名列表，告诉谷歌它们*始终*希望其网站通过 https 提供。如果用户尝试手动请求不安全版本，浏览器将不允许。这意味着用户与其银行之间的连接，例如，不会被劫持。一个不良的 WiFi 网络也不能强制用户访问网站的不安全和欺诈版本。

大约经过十年的使用，该列表现在大小为 14MB，其中大约有 130,000 个条目。您可以[在线查看列表](https://source.chromium.org/chromium/chromium/src/+/main:net/http/transport_security_state_static.json)或[下载列表](https://chromium.googlesource.com/chromium/src/net/+/refs/heads/main/http)。

格式相对简单：

```
{

 "name": "example.com",

 "policy": "bulk-1-year",

 "mode": "force-https",

 "include_subdomains": true 

},
```

当列表更新时，[Chrome 会使用哈夫曼编码压缩创建一个字典树](https://source.chromium.org/chromium/chromium/src/+/main:net/tools/transport_security_state_generator/README.md?q=transport_security_state_static.json&ss=chromium%2Fchromium%2Fsrc&start=11)，这样它就不必每次都解析那个庞大的文件。

最受欢迎（超过 1,000 条目）的顶级域名后缀 / 公共后缀包括：

| 排名 | TLD | 条目 |
| --: | :-: | --: |
| 1 | com | 43,236 |
| 2 | tk | 19,022 |
| 3 | de | 5,216 |
| 4 | org | 4,731 |
| 5 | gov | 4,507 |
| 6 | net | 4,410 |
| 7 | ga | 4,326 |
| 8 | nl | 2,671 |
| 9 | cf | 2,458 |
| 10 | ml | 2,271 |
| 11 | co.uk | 2,139 |
| 12 | fr | 1,714 |
| 13 | ru | 1,516 |
| 14 | eu | 1,283 |
| 15 | com.br | 1,226 |
| 16 | gq | 1,225 |
| 17 | io | 1,215 |
| 18 | com.au | 1,202 |
| 19 | it | 1,103 |
| 20 | cz | 1,004 |

在 `.com` 之后，免费的 `.tk` 域名绝对占据主导地位。我不知道其中有多少是欺诈的？

有 2,676 个 `.uk` 域名 - 其中只有 537 个不在 `.co.uk` 上。

再进一步，有 418 个国际化域名（以 `xn--` 开头）。

还有大约 187 个域名中含有 "porn"。

你不能从这个数据集中推断出*太多*。很多域名似乎已经过期或不再有效。阅读 [https://hstspreload.org](https://hstspreload.org) 周围的内容，指出由于这个列表是*硬编码*到 Chrome 中的，因此可能需要几个月才能将站点添加到列表中。同样，删除也可能需要很长时间。

我无法摆脱这样的感觉，认为应该有更好的方式来管理所有这些。
