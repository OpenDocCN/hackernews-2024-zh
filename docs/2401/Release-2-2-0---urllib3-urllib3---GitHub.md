<!--yml

类别：未分类

日期：2024-05-27 15:20:58

-->

# [urllib3/urllib3 · GitHub](https://github.com/urllib3/urllib3/releases/tag/2.2.0)

> 来源：[`github.com/urllib3/urllib3/releases/tag/2.2.0`](https://github.com/urllib3/urllib3/releases/tag/2.2.0)

## 🖥️ urllib3 现在在浏览器中可用

🎉 **此版本增加了对[在浏览器中使用 Pyodide 的 urllib3 的实验性支持](https://urllib3.readthedocs.io/en/stable/reference/contrib/emscripten.html)！** 🎉

感谢 Joe Marshall（[@joemarshall](https://github.com/joemarshall)）贡献了这个功能。这一变化得益于 urllib3 v2.0 对我们的 API 与 `http.client` 分离的工作。请将所有 bug 报告给[urllib3 问题跟踪器](https://github.com/urllib3/urllib3/issues)。

## 🚀 urllib3 正在为 HTTP/2 支持进行筹款

[urllib3 正在筹集约 40,000 美元](https://sethmlarson.dev/urllib3-is-fundraising-for-http2-support)来发布 HTTP/2 支持，并确保在 2023 年财政支持急剧下降后，项目的长期可持续维护。如果您的公司或组织使用 Python，并且希望在 Requests、pip、云 SDK 和成千上万的其他项目中获得 HTTP/2 支持，请考虑[贡献资金](https://opencollective.com/urllib3)以确保 HTTP/2 支持可持续发展并长期维护。

感谢您的支持。

## 变更

+   增加了对[Emscripten 和 Pyodide](https://urllib3.readthedocs.io/en/latest/reference/contrib/emscripten.html)的支持，包括在启用线程的跨源隔离浏览器环境中的流式支持。（[#2951](https://github.com/urllib3/urllib3/issues/2951)）

+   添加了对 `HTTPResponse.read1()` 方法的支持。（[#3186](https://github.com/urllib3/urllib3/issues/3186)）

+   添加了对 HTTP/2 的基本支持。 ([#3284](https://github.com/urllib3/urllib3/issues/3284))

+   修复了针对带有尾随点的 URL 请求失败的问题，原因是 SSL 错误。

    在使用代理时。 ([#2244](https://github.com/urllib3/urllib3/issues/2244))

+   修复了连接到代理后，`HTTPConnection.proxy_is_verified` 和 `HTTPSConnection.proxy_is_verified` 始终设置为布尔值的问题。在某些情况下，它以前可能是 `None`。 ([#3130](https://github.com/urllib3/urllib3/issues/3130))

+   修复了在请求中传递 `json=` 的头部时会发生突变的问题 ([#3203](https://github.com/urllib3/urllib3/issues/3203))

+   修复了从 HTTPS 代理连接到 HTTP 目标时，`HTTPSConnection.is_verified` 被设置为 `False` 的问题。以前它被设置为 `True`。（[#3267](https://github.com/urllib3/urllib3/issues/3267)）

+   修复了在配置 HTTP 代理为 HTTPS 时 OpenSSL 3.2.0 中出现的新错误消息的处理问题。 ([#3268](https://github.com/urllib3/urllib3/issues/3268))

+   修复了在服务器证书验证被禁用时 TLS 1.3 后握手认证的问题（[#3325](https://github.com/urllib3/urllib3/issues/3325)）。

下游分发商注意事项：为了运行集成测试，现在需要使用`--integration` pytest 标志再次运行测试。 ([#3181](https://github.com/urllib3/urllib3/issues/3181))
