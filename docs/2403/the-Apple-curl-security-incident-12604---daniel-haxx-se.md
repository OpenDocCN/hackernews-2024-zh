<!--yml

category: 未分类

日期：2024-05-27 14:46:58

-->

# 苹果的 curl 安全事件 12604 | daniel.haxx.se

> 来源：[https://daniel.haxx.se/blog/2024/03/08/the-apple-curl-security-incident-12604/](https://daniel.haxx.se/blog/2024/03/08/the-apple-curl-security-incident-12604/)

tldr：苹果认为这没问题，但我不这么认为。

在 2023 年 12 月 28 日，[bugreport 12604](https://github.com/curl/curl/issues/12604) 在 curl 问题跟踪器中被提出。我们每天收到很多问题反馈，因此单凭这个事实本身并不算出乎意料。我们阅读报告，调查问题，提出跟进问题，看看我们能学到什么，需要解决什么问题。

本例中问题的标题非常清楚：*[flag –cacert 行为在 macOS 和 Linux 之间不一致](https://github.com/curl/curl/issues/12604)*，由吴悦东提出。

友好的报告者展示了 macOS 捆绑的 curl 版本与完全从开源构建的 curl 二进制文件在同一 macOS 机器上运行时表现不同。

curl 命令行选项`[--cacert](https://curl.se/docs/manpage.html#--cacert)`为用户提供了一种方式，告诉 curl **这是要信任的确切 CA 证书集**，在进行以下传输时。如果 TLS 服务器无法提供可以使用该证书集验证的证书，则应该失败并返回错误。

curl 中的这种特定行为和功能已经存在多年（该选项于 2000 年 12 月添加到 curl 中），当然是为了让用户知道它正在与已知和可信的服务器通信。这实际上是 TLS 所做的一个非常基本的部分。

当在 macOS 上使用 curl 这个命令行选项时，*由苹果提供的版本*，**它似乎会退回并检查系统的 CA 存储，以防所提供的 CA 证书集在验证失败时**。这是一个未请求的*次要检查*，没有文档记录，而且真的让人大吃一惊。因此，当用户使用修剪过的专用 CA 证书文件运行检查时，如果系统的 CA 存储包含可以验证服务器的证书，它将不会失败！

这是一个安全问题，因为现在**突然间通过了本不应该通过的证书检查**。

我在 2023 年 12 月 29 日 08:30 UTC 发送的电子邮件中报告了这个安全问题。这不是一个主要问题，但确实存在问题。

## 苹果说这没问题。

在 2024 年 3 月 8 日，苹果产品安全团队做出了他们的回应：

```
Hello,

Thank you again for reporting this to us and allowing us time to investigate.

Apple’s version of OpenSSL (LibreSSL) intentionally uses the built-in system trust store as a default source of trust. Because the server certificate can be validated successfully using the built-in system trust store, we don't consider this something that needs to be addressed in our platforms.

Best regards,
KC
Apple Product Security
```

事件解决。

## 我不同意。

显然，我持不同意见。这个*未记录的功能*使得在 macOS 上使用 curl 进行 CA 证书验证变得完全不可靠，并且与文档不一致。它会误导用户。

请注意。

由于这不是我们所提供的 curl 版本中的安全漏洞，我们对这个问题没有发布 CVE 或者其他任何信息。严格来说，问题甚至不是在 curl 代码中。它出现在苹果提供并构建 curl 在其平台上使用的 LibreSSL 版本。

## 讨论

[黑客新闻](https://news.ycombinator.com/item?id=39650498)
