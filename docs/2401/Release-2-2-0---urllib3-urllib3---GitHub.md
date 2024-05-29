<!--yml
category: 未分类
date: 2024-05-27 15:20:58
-->

# Release 2.2.0 · urllib3/urllib3 · GitHub

> 来源：[https://github.com/urllib3/urllib3/releases/tag/2.2.0](https://github.com/urllib3/urllib3/releases/tag/2.2.0)

## 🖥️ urllib3 now works in the browser

🎉 **This release adds experimental support for [using urllib3 in the browser with Pyodide](https://urllib3.readthedocs.io/en/stable/reference/contrib/emscripten.html)!** 🎉

Thanks to Joe Marshall ([@joemarshall](https://github.com/joemarshall)) for contributing this feature. This change was possible thanks to work done in urllib3 v2.0 to detach our API from `http.client`. Please report all bugs to the [urllib3 issue tracker](https://github.com/urllib3/urllib3/issues).

## 🚀 urllib3 is fundraising for HTTP/2 support

[urllib3 is raising ~$40,000 USD](https://sethmlarson.dev/urllib3-is-fundraising-for-http2-support) to release HTTP/2 support and ensure long-term sustainable maintenance of the project after a sharp decline in financial support for 2023\. If your company or organization uses Python and would benefit from HTTP/2 support in Requests, pip, cloud SDKs, and thousands of other projects [please consider contributing financially](https://opencollective.com/urllib3) to ensure HTTP/2 support is developed sustainably and maintained for the long-haul.

Thank you for your support.

## Changes

*   Added support for [Emscripten and Pyodide](https://urllib3.readthedocs.io/en/latest/reference/contrib/emscripten.html), including streaming support in cross-origin isolated browser environments where threading is enabled. ([#2951](https://github.com/urllib3/urllib3/issues/2951))
*   Added support for `HTTPResponse.read1()` method. ([#3186](https://github.com/urllib3/urllib3/issues/3186))
*   Added rudimentary support for HTTP/2\. ([#3284](https://github.com/urllib3/urllib3/issues/3284))
*   Fixed issue where requests against urls with trailing dots were failing due to SSL errors
    when using proxy. ([#2244](https://github.com/urllib3/urllib3/issues/2244))
*   Fixed `HTTPConnection.proxy_is_verified` and `HTTPSConnection.proxy_is_verified` to be always set to a boolean after connecting to a proxy. It could be `None` in some cases previously. ([#3130](https://github.com/urllib3/urllib3/issues/3130))
*   Fixed an issue where `headers` passed in a request with `json=` would be mutated ([#3203](https://github.com/urllib3/urllib3/issues/3203))
*   Fixed `HTTPSConnection.is_verified` to be set to `False` when connecting from a HTTPS proxy to an HTTP target. It was set to `True` previously. ([#3267](https://github.com/urllib3/urllib3/issues/3267))
*   Fixed handling of new error message from OpenSSL 3.2.0 when configuring an HTTP proxy as HTTPS ([#3268](https://github.com/urllib3/urllib3/issues/3268))
*   Fixed TLS 1.3 post-handshake auth when the server certificate validation is disabled ([#3325](https://github.com/urllib3/urllib3/issues/3325))

Note for downstream distributors: To run integration tests, you now need to run the tests a second time with the `--integration` pytest flag. ([#3181](https://github.com/urllib3/urllib3/issues/3181))