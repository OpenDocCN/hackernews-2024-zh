<!--yml

category: 未分类

date: 2024-05-27 13:10:30

-->

# 浏览器中的非安全性问题：JavaScript在PDF中 – text/plain

> 来源：[https://textslashplain.com/2024/04/10/browser-security-bugs-that-arent-javascript-in-pdf/](https://textslashplain.com/2024/04/10/browser-security-bugs-that-arent-javascript-in-pdf/)

一种相当常见的安全漏洞报告形式是：“*我可以将JavaScript放入PDF文件中并执行它！*”

例如，使用Chrome打开[此PDF文件](https://webdbg.com/test/pdf/contains-script.pdf)，您可以看到`alert(1)`消息显示：

JavaScript在PDF中的支持是按设计预期的PDF渲染软件开发人员支持的功能，包括常见的浏览器如Chrome和Edge。就像HTML一样，PDF文件是一种[活动内容](https://chromium.googlesource.com/chromium/src/+/master/docs/security/faq.md#Are-PDF-files-static-content-in-Chromium)类型，可能包含JavaScript。

偶尔，*经验不足的*安全研究人员会兴奋地针对**浏览器**提交此问题，而这些报告很快会被“[按设计](https://chromium.googlesource.com/chromium/src/+/master/docs/security/faq.md#Are-PDF-files-static-content-in-Chromium)”解决。

偶尔，*经验丰富的*安全研究人员会兴奋地针对**站点和应用程序**提交此问题，这些站点和应用程序愿意托管或传输不受信任的PDF文件，认为这代表了“[存储型跨站脚本漏洞](https://owasp.org/www-community/attacks/xss/#stored-xss-attacks)”。

他们在这里的困惑更加可以理解——如果一个网站允许用户上传包含脚本的**HTML文档**，然后从其域中提供该HTML文档，其中的任何脚本将在服务域的安全上下文中运行。*这*描述了经典的存储型XSS攻击，并且它存在安全威胁，因为嵌入的脚本可以窃取或操作cookie（通过访问`document.cookie`属性），操作Web平台存储（IndexedDB、`localStorage`等），进行来自第一方源的请求伪造攻击等。

PDF文档的情况则大不相同。

[Chrome安全FAQ](https://chromium.googlesource.com/chromium/src/+/master/docs/security/faq.md#does-executing-javascript-in-a-pdf-file-mean-there_s-an-xss-vulnerability)简要描述了这一限制，指出PDF文件提供给PDF的绑定集合比HTML文档提供给DOM的更有限，并且PDF不会根据其服务域获得任何环境权限。

这是什么意思？这意味着，虽然 PDF 中的 JavaScript 确实运行，但脚本运行的*宇宙*是受限的：无法访问 cookie 或存储，极其有限地进行请求（例如，可以将文档的窗口导航到其他位置，但仅此而已），也无法利用 HTML 文档对象模型（DOM）对象如 `document`、`window`、`navigator` 等所暴露的 Web 平台强大功能。虽然 PDF 中的 JavaScript 能力非常有限，但并非不存在，PDF 引擎软件必须小心，避免引入新的功能，从而破坏 PDF 处理代码的安全性假设。

#### 限制 PDF 中的 JavaScript

工程师们应该注意，在处理 PDF 中的 JavaScript 时，要尊重应用程序/用户的设置。例如，如果用户在某个网站上关闭了 JavaScript，那么托管在该网站上的 PDF 文件 [不应允许脚本](https://source.chromium.org/chromium/chromium/src/+/main:chrome/browser/resources/pdf/main.ts;l=40;drc=1076746b494b84eeaf625c7a475ee501de270f17)。这在 Chrome 和 Edge 中都能正常工作。

Firefox 中全局的 `javascript.enabled` 切换设置并不影响其 PDF 查看器中的脚本：

相反，Firefox 提供了一个名为 `pdfjs.enableScripting` 的个别偏好设置，可以从 `about:`config 页面进行配置。

Chromium 目前 [忽略](https://issues.chromium.org/issues/41494478) PDF 响应中的 `Content-Security-Policy` 头部，因为它使用 web 技术来渲染 PDF 文件，这可能与 `CSP` 不兼容，导致用户困惑和开发者不便。
