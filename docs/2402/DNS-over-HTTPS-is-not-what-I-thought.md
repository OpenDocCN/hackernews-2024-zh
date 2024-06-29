<!--yml

category: 未分类

date: 2024-05-27 14:31:16

-->

# DNS over HTTPS 并非我所想象的那样。

> 来源：[https://www.petefreitag.com/blog/dns-over-https/](https://www.petefreitag.com/blog/dns-over-https/)

几个月前，我致力于移除我博客上的一些旧的损坏链接。我从2002年开始写博客，所以我二十年前链接的许多网站现在不再活跃，或者不再由同一所有者拥有。我决定通过清除任何不再通过 DNS 解析的域名来开始这项任务。

虽然我知道一些传统的查询 DNS 方法，但我认为这是探索 *DNS over HTTPS* 或 *DoH* 协议的好机会。这是一种通过加密的 HTTPS 连接访问 DNS 记录的相当 *新* 的标准。RFC（8484）发布于2018年10月。

DNS over HTTPS 的酷之处在于，它使用 HTTPS。由于 HTTP 客户端非常丰富且为许多开发人员所理解，*应该* 可以实现一个非常简单的实现。

### 使用 curl 进行 DNS 查询

假设我们要查询 example.com 的 `A` 记录，我们可以像这样使用 curl 进行请求：

```
curl -H 'Accept: application/dns-json' 'https://cloudflare-dns.com/dns-query?name=example.com&type=A'
```

在这种情况下，我正在使用 CloudFlare 的公共 DNS over HTTPS 服务器 `cloudflare-dns.com`，但我们也可以使用 Google 的公共 DNS over HTTPS 服务器：`dns.google`，通过 curl 这样使用：

```
curl -H 'Accept: application/dns-json' 'https://dns.google/resolve?name=example.com&type=A'
```

实际上，唯一稍微棘手的地方是，您必须传递 HTTP 请求标头：

```
Accept: application/dns-json
```

从这里开始，只需传入要查找的名称以及使用查询字符串参数指定的 DNS 记录类型。

我们从服务器收到的响应看起来像这样：

```
{
  "Status": 0,
  "TC": false,
  "RD": true,
  "RA": true,
  "AD": true,
  "CD": false,
  "Question": [
    {
      "name": "example.com",
      "type": 1
    }
  ],
  "Answer": [
    {
      "name": "example.com",
      "type": 1,
      "TTL": 85506,
      "data": "93.184.216.34"
    }
  ]
}

```

如果您请求一个没有 A 记录的主机名，您可能会收到这样的响应：

```
{
    "Status": 3,
    "TC": false,
    "RD": true,
    "RA": true,
    "AD": false,
    "CD": false,
    "Question": [
        {
        "name": "invalid-12345-example.com",
        "type": 1
        }
    ],
    "Authority": [
        {
        "name": "com",
        "type": 6,
        "TTL": 900,
        "data": "a.gtld-servers.net. nstld.verisign-grs.com. 1706288745 1800 900 604800 86400"
        }
    ]
    }

```

在这种情况下，您会注意到没有 `Answer` 数组，并且 `Status` 是 `3` 而不是 `0`。状态 3 对应于 DNS 服务器的 NXDOMAIN（代码 3）响应。以下是一些可能的状态值：

+   `0` (NOERROR): DNS 查询成功

+   `1` (FORMERR): DNS 查询格式无效。

+   `2` (SERVFAIL): 服务器由于问题未能处理 DNS 查询。

+   `3` (NXDOMAIN): 查询的域名不存在。

+   `4` (NOTIMP): 服务器不支持请求的查询类型。

+   `5` (REFUSED): 服务器因策略原因拒绝回答查询。

这样可以很容易地查看 DNS 查询是否成功。

接下来，您可能想知道那些布尔键是什么：

+   `TC` (Truncated) - 指示由于大小限制而截断响应。

+   `RD` (Recursion Desired) - 指示客户端是否请求递归解析。

+   `RA` (Recursion Available) - 服务器是否支持递归解析

+   `AD` (Authentic Data) - 指示响应是否已使用 DNSSEC（域名系统安全扩展）进行验证。

+   `CD` (Checking Disabled) - 客户端是否请求禁用 DNSSEC 验证。

### 这不是 RFC 8484，不是 DoH！

结果发现我使用的实际上并不是RFC 8484 DNS over HTTPS，而是受2013年草案规范启发的一种实现，称为*用于表示DNS数据的JSON格式草案-bortzmeyer-dns-json-01*。

这个实际的RFC 8484 DNS over HTTPS规范要难得多，因为你必须发送符合RFC 1035的二进制编码消息。例如，如果你想要发送对`www.example.com`的A记录查找，你可以这样做：

```
curl 'https://dns.google/dns-query?dns=AAABAAABAAAAAAAAA3d3dwdleGFtcGxlA2NvbQAAAQAB' | cat -v
```

DNS查询是对二进制DNS记录请求的URL安全的Base64编码。

我使用`cat -v`来显示响应，因为二进制响应包含你的终端默认隐藏的字符。

### 总结

发现我所认为的*DNS over HTTPS*实际上并不符合RFC 8484标准是很有趣的。CloudFlare和Google提供的只是基于2013年IETF互联网草案的一个非常方便的JSON API实现。

实际上有一篇题为*在JSON中表示DNS消息*的RFC 8427，但这不是CloudFlare / Google服务器使用的格式。它更接近于将二进制格式映射到JSON表示。

我可能会继续使用JSON API，而不是真正的DNS over HTTPS协议，但知道有所区别还是很好的。
