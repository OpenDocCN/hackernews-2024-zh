<!--yml

category: 未分类

日期：2024-05-27 14:50:22

-->

# Caddy的SSO插件存在的安全漏洞 | Trail of Bits博客

> 来源：[https://blog.trailofbits.com/2023/09/18/security-flaws-in-an-sso-plugin-for-caddy/](https://blog.trailofbits.com/2023/09/18/security-flaws-in-an-sso-plugin-for-caddy/)

*作者：Maciej Domanski、Travis Peters和David Pokora*

我们在[Caddy Web服务器的caddy-security插件](https://github.com/greenpau/caddy-security)中发现了10个安全漏洞，这些漏洞可能导致Web应用程序中的多种高危攻击，包括客户端代码执行、OAuth重放攻击和未经授权访问资源。

在我们的评估过程中，Caddy被部署为反向代理，用于访问我们内部几个服务。我们探索了一种插件配置，可以让我们使用Google SSO处理认证和授权，这样我们就不必在每个应用程序上实现这些功能。

在本博文中，我们将简要探讨我们在caddy-security插件中发现的安全漏洞，并讨论它们可能带来的影响。与我们典型的安全评估一样，对于我们发现的每个问题，我们提出了立即修复问题的建议，以及更长远的、更具战略性的建议，以预防类似问题并改进整体安全姿态。

作为安全专家，我们的目标不仅仅是识别特定软件中的漏洞，还包括通过分享我们的建议来为更大的社区贡献。我们相信这些建议可以帮助开发人员在其他SSO系统中克服类似的挑战，并提升其安全性。

## Caddy 背景

[Caddy](https://caddyserver.com/)（又名Caddy服务器或Caddy 2）是一个现代化的开源Web服务器，用Golang编写，旨在易于使用且高度可配置。Caddy旨在简化托管Web应用程序的过程，同时优先考虑安全性和性能。它旨在减少配置和部署Web服务器时的复杂性。

caddy-security 插件是Caddy Web服务器的一个中间件插件。它提供各种安全相关的功能，以增强Web应用程序的整体安全性。caddy-security 插件提供的一些关键功能包括用于实现基于表单、基本、本地、LDAP、OpenID Connect、OAuth 2.0、SAML认证的认证插件，以及基于JWT/PASETO令牌的HTTP请求授权插件。

## 发现

### 问题 #1：反射型跨站脚本（XSS）

*严重性：高*

反射性XSS发生在应用程序将不受信任的数据包含在发送给用户浏览器的HTML响应中时。在此情况下，提供的`/admin%22%3E%3Cscript%3Ealert(document.domain)%3C/script%3E/admin/login`或`/settings/mfa/delete/<img%20src=x%20onerror=alert(document.domain)>` API调用会触发警报。攻击者可以利用此漏洞在目标用户的浏览器中执行任意JavaScript代码，可能导致进一步的攻击，如会话劫持。

为了立即解决此问题，战略性地将所有字符串值视为可能不可信，无论它们的来源如何，并适当地进行转义（使用生成输出安全HTML的`safehtml/template`包）。

除了纠正之外，我们还建议采用几种不同的方法来加强深度防御：

+   扩展单元测试，使用潜在恶意的XSS负载。参考[跨站脚本（XSS）防御手册](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet)了解各种攻击向量。

+   考虑在测试环境中为所有API调用使用Burp Suite Professional的Active Scanner。此外，使用[实时任务扫描](https://portswigger.net/burp/documentation/desktop/automated-scanning/live-tasks)策略，在与Web界面交互时自动扫描底层请求。

+   扩展caddy-security文档以提升安全头部，特别是内容安全策略（CSP）头部，它控制浏览器可以加载哪些资源，从而限制潜在的XSS攻击的影响。

### 问题＃2：不安全的随机性

*严重性：高*

caddy-security插件使用基于Unix时间戳的种子来生成应用程序中三个安全关键上下文的字符串，可能可通过暴力搜索预测。攻击者可能利用用于OAuth流程中身份验证目的的可能可预测的一次性值来执行OAuth重放攻击。此外，在数据库包中生成多因素认证（MFA）密钥时使用不安全的随机性。

为了立即缓解此漏洞，请使用密码学安全的随机数生成器生成随机字符串。Golang的`crypto/rand`库专为生成安全随机数而设计。

除了修复之外，我们建议考虑以下长期建议：

+   审查应用程序中其他使用`math/rand`包进行安全上下文的实例。创建安全的包装函数，并在整个代码中使用它们来生成所需长度的密码学安全字符串。

+   避免重复代码。只有一个函数，例如`secureRandomString`，而不是多个重复函数，可以更轻松地审核和验证系统的安全性。它还可以防止将来对代码库的更改重新引入问题。

+   在CI/CD中实施Semgrep。`math-random-used` Semgrep规则将捕捉使用`math/rand`的实例。更多信息请参考我们的[关于Semgrep的测试手册](https://appsec.guide/docs/static-analysis/semgrep/)。

+   阅读《实用密码学》等教材，这是实际密码学考虑的重要资源。

### 问题＃3：通过X-Forwarded-For头部进行IP欺骗

*严重性：中等*

通过操纵`X-Forwarded-For`头部，攻击者可以欺骗用户身份模块(`/whoami` API端点)中使用的IP地址。如果系统信任此伪造的IP地址，则可能导致未经授权的访问。

要解决此漏洞，重新实现应用程序，不依赖于获取用户IP地址时提供的用户提供的头部。如果需要用户提供的头部（例如用于日志记录目的的`X-Forwarded-For`），确保头部经过适当验证（即值与IP地址格式一致，通过正则表达式）或者清理（避免[CRLF日志注入攻击](https://owasp.org/www-community/vulnerabilities/CRLF_Injection)，例如）。

除了这个即时修复，我们建议考虑以下长期建议：

+   在单元测试级别实现适当的检查，以防止潜在的IP欺骗和`X-`头部。考虑[其他可以重写IP来源的头部](https://book.hacktricks.xyz/network-services-pentesting/pentesting-web/special-http-headers#headers-to-change-location)。

+   在Golang的本机模糊测试中覆盖IP欺骗场景和用户提供的头部处理。

+   使用Burp Suite Professional和[Param Miner](https://portswigger.net/bappstore/17d2949a985c4b7ca092728dba871943)扩展的动态测试方法来识别隐藏头部的处理。

+   扩展caddy-security文档，增加用户对此类威胁的认识；展示配置错误的示例，解决方法以及测试方法。

### 问题＃4：基于Referer头部的跨站脚本攻击（XSS）

*严重性：中等*

通过重写`Referer`头部可以触发XSS漏洞。虽然`Referer`头部通过转义一些字符（例如[`&`], [`<`], [`>`], [`"`], [`'`])进行了清理，但未考虑基于JavaScript URL方案的攻击（例如`javascript:alert(document.domain)//`负载）。利用此漏洞可能并不轻松，但可能导致在目标用户浏览器中执行恶意脚本，从而危及用户会话。

此问题的缓解措施与问题＃1相同。

### 问题＃5：开放重定向漏洞

*严重性：中等*

当已登录用户单击具有`redirect_url`参数的特制链接时，用户可以被重定向到外部网站。用户必须采取行动，例如单击门户按钮或使用浏览器的返回按钮来触发重定向。这可能导致钓鱼攻击，攻击者通过制作令人信服的URL来诱使用户访问恶意网站。

要减轻此漏洞，执行适当的`redirect_url`参数验证，确保重定向URL仅允许在同一域内或来自受信任的来源。

此外，我们还建议以下长期修复：

+   实施健壮的单元测试，涵盖了`redirect_url`参数验证的不同绕过场景。参考可能的[URL格式绕过](https://book.hacktricks.xyz/pentesting-web/ssrf-server-side-request-forgery/url-format-bypass)。请注意，不同组件可能使用不同的URI解析器，这可能导致[parsing confusion](https://snyk.io/blog/url-confusion-vulnerabilities/)。

+   使用Burp Suite Professional与启用了以下两个设置的扫描器：

    +   审计覆盖 - 最大：使用最广泛的负载变化和插入点选项

    +   审计覆盖 - 彻底：尝试更多的负载变化

### 问题＃6：X-Forwarded-Host头部操纵

*严重性：中*

caddy-security插件处理`X-Forwarded-Host`头部，这可能导致各种安全漏洞（Web缓存污染、业务逻辑缺陷、基于路由的服务器端请求伪造[SSRF]和经典的服务器端漏洞）。此外，caddy-security插件基于此头部生成QR码，扩展了攻击面。

为了减轻这个问题，请勿在caddy-security插件逻辑中依赖于`Host`和`X-Forwarded-Host`头。而是手动使用配置文件中当前指定的域来生成QR码。

此外，我们建议以下操作：

+   使用Burp Suite Professional与Param Miner扩展来识别隐藏头部的处理。

+   扩展caddy-security文档以增强用户对[HTTP `Host`头部攻击](https://portswigger.net/web-security/host-header/exploiting)的意识。

### 问题＃7：X-Forwarded-Proto头部操纵

*严重性：低*

处理`X-Forwarded-Proto`头部会导致重定向到注入的协议。虽然此场景可能影响有限，但不正确处理此类头部可能导致安全机制绕过或处理TLS时的混乱，可能导致不可预测的安全风险。

要解决此问题，请勿依赖于`X-Forwarded-Proto`头部。如果需要，应针对接受的协议白名单（例如HTTP/HTTPS）验证`X-Forwarded-Proto`头部的值，并拒绝意外的值。

此外，考虑长期建议从问题＃3开始。

### 问题＃8：通过暴力破解验证代码绕过2FA

*严重性：低*

应用程序的当前实现的两因素认证（2FA）缺乏足够的保护，以防范暴力破解攻击。尽管应用程序在多次未能提供2FA代码后阻止用户，攻击者可以通过自动化应用程序的完整多步骤2FA过程来绕过此阻止机制。

为了有效解决这个问题，在MFA配置中强制执行最小六位代码长度。此外，为了减少自动暴力破解的风险，在一定数量的无效2FA代码尝试后实施帐户锁定机制。最后，在涉及敏感帐户信息或安全设置的关键操作中，强制重新验证。对于诸如更改密码或禁用2FA等操作，用户应要求重新验证，可以使用密码或2FA令牌。如果用户在过去的10分钟内已登录，则可以豁免重新验证。更多信息请查看[2019年正确使用2FA](https://blog.trailofbits.com/2019/06/20/getting-2fa-right-in-2019/)在Trail of Bits Blog上。

### 问题 #9：注销时缺乏用户会话失效

*严重性：低*

在点击“登出”按钮后，caddy-security插件缺乏适当的用户会话失效；即使在发送到`/logout`和`/oauth2/google/logout`的请求后，用户会话仍然有效。获取激活但被假定为已注销的会话的攻击者可以代表用户执行未经授权的操作。

为了解决这个问题，审查登出过程以确定意外行为的原因。确保`/oauth2/google/logout`端点正确终止用户会话并使相关令牌失效。

为了更深入的防御，使用[OWASP应用程序安全性验证标准（V3会话管理）](https://github.com/OWASP/ASVS/blob/master/4.0/en/0x12-V3-Session-management.md#v3-session-management)检查实现是否安全处理会话。

### 问题 #10：解析Caddyfile时发生多个Panic

*严重性：低*

多个解析函数在尝试访问元素之前不验证其输入值是否为`nil`，这可能导致Panic（索引超出范围）。在解析Caddyfile期间的Panic可能不被视为即时漏洞，但它们可能表明未正确执行的安全控制（例如，不充分的数据验证），这可能导致其他代码路径中的问题。

为了解决这些问题，在所有相关功能中，在元素访问之前，集成对输入值的`nil`检查。

为了防止类似类型的问题，为Caddyfile解析功能添加Golang原生模糊测试。

## 针对社区的Golang安全性

在Trail of Bits，我们*热爱*编写和审查Golang代码库。事实上，我们一直在处理与Golang相关的[(Semgrep) 资源](https://appsec.guide/docs/static-analysis/semgrep/)、[规则](https://github.com/trailofbits/semgrep-rules/tree/main/go)和[博客文章](https://blog.trailofbits.com/?s=go&submit=Search)，并期待着参与像这样的宠物审计以及[客户项目](https://github.com/trailofbits/publications#security-reviews)，在这些项目中我们会检查Golang代码库。

我们发布发现的目的是帮助保护那些考虑实施类似我们探索的解决方案的其他人，并帮助他们就安全基础设施做出知情决策。

如果您正在积极地实施Golang代码库或者有关于开源软件的问题、疑虑或其他建议，您认为我们应该关注，请[联系我们](https://www.trailofbits.com/contact/)。

## 协调披露时间表

作为披露过程的一部分，我们首先向caddy-security插件的维护者报告了这些漏洞。披露的时间线如下：

+   +   2023年8月7日：我们向caddy-security插件的维护者报告了我们的发现。

    +   2023年8月23日：caddy-security插件的维护者确认目前没有近期计划来处理报告的漏洞。

    +   2023年9月18日：披露博客文章发布，并在原始项目存储库中提交了问题。

### 就像这样：

载入中...

### *相关*
