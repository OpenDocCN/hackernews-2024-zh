<!--yml

-   类别：未分类

-   日期：2024-05-29 12:39:29

-   -->

# -   Single Sign-On（SSO）是什么？ - 由米兰·米兰诺维奇博士讲解

> -   来源：[https://newsletter.techworld-with-milan.com/p/how-does-single-sign-on-sso-work](https://newsletter.techworld-with-milan.com/p/how-does-single-sign-on-sso-work)

-   在这期中，我们将讨论**单点登录（SSO）**。通过单一登录凭证，用户可以安全地访问多个应用程序和服务，这就是SSO，一种用户认证机制。

-   让我们深入了解。

-   * * *

***[POST/CON](https://www.postman.com/postcon/?utm_source=influencer&utm_medium=Social&utm_campaign=POSTCON_2024&utm_term=Milan_Milanovic&utm_content=Conference_Landing_Page)** 是 Postman 史上最大的 API 大会，将于 4 月 30 日至 5 月 1 日在加利福尼亚州旧金山举行！目前，Postman 团队正在提供**[票价 30% 折扣](https://postcon.postman.com/event/5bb77b40-684b-4b79-bb8f-f1420897ee6e/register?utm_source=influencer&utm_medium=Social&utm_campaign=POSTCON_2024&utm_term=Milan_Milanovic&utm_content=ticket_sales_30_discount)**，截至 3 月 26 日。点击这里注册！*

-   *POST/CON 24 是为所有与 API 有关或业务依赖 API 的人士。参与者将获得证书和 Postman 徽章！*

*在两天的时间里，您将听取**[行业领袖的发言](https://www.postman.com/postcon/speakers/?utm_source=influencer&utm_medium=Social&utm_campaign=POSTCON_2024&utm_term=Milan_Milanovic&utm_content=Conference_Speakers)**，参加实践**[工作坊](https://www.postman.com/postcon/workshops/?utm_source=influencer&utm_medium=Social&utm_campaign=POSTCON_2024&utm_term=Milan_Milanovic&utm_content=Conference_Workshops)**，聆听深入的演示，并参与独家对话。如果您与 API 有关或您的业务依赖于 API，POST/CON 是您的场所。*

-   [立即注册](https://postcon.postman.com/event/5bb77b40-684b-4b79-bb8f-f1420897ee6e/regProcessStep1?utm_source=influencer&utm_medium=Social&utm_campaign=POSTCON_2024&utm_term=Milan_Milanovic&utm_content=ticket_sales_30_discount)

-   * * *

**单点登录（SSO）**是一种允许用户使用单一登录访问多个应用程序的认证过程。这是通过一个中央认证服务器实现的，该服务器存储用户的凭证，并为每个应用程序验证它们。

-   在云时代，SSO 的概念并不新鲜。在 1990 年代中后期，使企业能够安全地链接其个人电脑、网络和服务器的本地身份解决方案是 SSO 技术的来源。在这段时间内，公司开始使用专门的系统处理用户 ID，如**轻量级目录访问协议（LDAP）**和**Microsoft Active Directory（AD）**。之后，他们使用本地 SSO 或 Web 访问管理（WAM）产品来保护访问。

-   在 SSO 游戏中有三个关键角色：

+   **身份提供者 (IdP)：** 这是中央认证服务器。这是您输入凭据并得到验证的地方。将其视为高安全性建筑的入口处。

+   **服务提供商 (SP)：** 这些个别应用依赖于 SSO 进行用户登录。您的工作电子邮件、项目管理工具和 CRM 平台都可以作为 SP。想象这些个别办公室位于安全建筑内部。

+   **SSO 服务器**：这是 IdP 和 SP 之间的桥梁。它处理它们之间的通信并安全地传输认证令牌。将其视为连接入口和各个办公室之间的安全走廊。

Google 和其他服务是SSO运行的绝佳示例。让我们以尝试使用您的 Google 账户访问 Trello 为例。您不需要在 Trello 上创建新的用户账户并记住新的用户名/密码。

例如，当您尝试使用您的 Google 账户**登录到 Trello**时，它会将您重定向到托管在 accounts.google.com 上的中央服务。在这里，您将看到一个登录表单来输入您的凭据。如果认证过程成功，Google 将您重定向到 Trello，在那里您可以自动登录并获得访问权限。

通过 Google 账户访问 Trello

如果您想通过您的 Google 账户访问 Trello，下面是发生的步骤：

1.  **用户请求访问**：使用 Trello 登录网页并选择 Google 账户作为登录方式。

1.  **IdP 重定向**：Trello 将用户重定向到 Google 登录页面。

1.  **提供登录页面**：用户会看到 Google 的登录页面。

1.  **输入凭据**：用户输入他们的 Google 凭据。

1.  **SSO 服务器验证**：Google 将认证信息发送到 SSO 授权服务器。

1.  **IdP 上的认证**：如果凭据有效，授权服务器将返回认证令牌（SAML）。

1.  **访问授权**：Google 将认证令牌发送给 Trello。

1.  **验证令牌**：在最后一步，Trello 将令牌发送到 Google 授权服务器以进行验证。

1.  **令牌有效**：如果令牌有效，Trello 将允许用户访问并存储会话以供将来交互使用。

SSO 有多个好处，例如：

+   **改善用户体验**：用户不需要记住多个用户名和密码。

+   **增强安全性**：用户不太可能在应用程序之间重复使用密码。

+   **简化用户访问审计**：确保适当的个人可以访问资源和敏感数据可能具有挑战性。SSO 解决方案可以根据其角色、部门和资历级别配置用户访问权限。

SSO 也带来一些重要的缺点：

+   **单点故障**：最显著的缺点之一是，SSO 会创建单点故障。如果 SSO 系统被攻击者破坏，攻击者可能会访问所有连接的应用和服务。

+   **安全风险**: 如果凭证被泄露，所有连接的应用程序的安全性可能会受到威胁。

+   **应用兼容性**: 有时，应用程序未正确配置以与SSO系统配合工作。使用Kerberos、OAuth或SAML的应用程序提供商应能够执行真正的SSO。如果不能，您的SSO解决方案将无法提供完整覆盖，并成为用户必须记住的另一个密码。

要与SSO系统配合工作，您应了解不同的标准和协议。一些常见的协议类型包括：

+   **SAML**: 这是最常见的SSO类型。它使用SAML协议在SSO服务器和应用程序之间交换身份验证信息。

+   **开放授权（OAuth）2.0**: 它代表资源所有者提供委托访问服务器资源。它规定了令牌的传输方式，允许用户的身份由IDP进行验证，并使用凭证访问API。

+   **Open ID Connect (OIDC)** 是基于OAuth 2.0的新型SSO类型。它比SAML更为简单的协议，并且更易于与Web应用集成。

> *要了解更多关于OAuth 2.0的信息，请查看以下**[链接](https://newsletter.techworld-with-milan.com/i/138606726/how-does-oauth-work)**。请注意，您还可以**[使用Postman进行OAuth 2.0](https://learning.postman.com/docs/sending-requests/authorization/oauth-20/)**。*

其他一些SSO类型，如**Kerberos**和**智能卡身份验证**，并不广泛使用。

+   **Kerberos** 允许用户使用其凭据从KDC获取服务票据。然后将这些票据提供给应用程序以获取访问权限，消除了重复登录的需求。然而，Kerberos依赖于KDC与所有参与者之间的共享密钥，因此不适合面向互联网的SSO，存在因受损服务器暴露凭证等安全问题而使其不适合。

+   **智能卡** 作为身份的持有者与SSO系统（如锁）配合工作，授予对应用程序（门）的访问权限，而无需为每个应用程序单独登录。它为身份验证过程增加了物理元素，使其对未经授权的访问更具抵抗力。然而，用户必须亲自携带它。

在选择**适当**的协议时，应考虑以下因素：

+   **企业与消费者应用**: 由于其对企业身份提供者和复杂身份验证场景的广泛支持和集成能力，SAML通常适用于企业应用。OAuth 2.0和OIDC更适合面向消费者的应用程序，提供了灵活性和与移动和Web应用程序的兼容性。

+   **授权与认证：** 如果您的主要需求是认证（验证用户身份），则SAML或OIDC是您的首选选项。OIDC是建立在OAuth 2.0之上的，为OAuth的授权能力提供了额外的身份层。当您的应用程序需要请求访问用户资源而不暴露用户密码时，请使用OAuth 2.0。

+   **评估应用程序和平台的兼容性：** 检查SSO协议与您现有基础设施和计划集成的应用程序的兼容性。一些传统或企业系统可能更广泛地支持SAML，而现代应用程序通常更青睐于OAuth 2.0和OIDC，因为它们对API友好。

+   **考虑用户体验：** OIDC和OAuth 2.0的现代、基于令牌的方法可以提供更流畅和更整合的用户体验，特别是对于Web和移动应用程序而言。

+   **未来的规划：** 考虑您应用生态系统的未来方向。您是否正在转向基于云的服务、API和移动应用？OAuth 2.0和OIDC可能提供更多的灵活性，通常被认为更适合云和移动服务的未来发展。

+   **合规和监管要求：** 确保所选协议符合您行业相关的任何特定监管要求，如GDPR、HIPAA或其他可能规定特定安全或隐私标准的要求。

在市场上有许多产品可以用于SSO：

+   **[Microsoft Entra ID](https://www.microsoft.com/en-us/security/business/identity-access/microsoft-entra-id)**（以前称为Microsoft Active Directory）。适用于已投资于Microsoft生态系统的组织，它与Office 365、Dynamics CRM和其他Microsoft服务无缝集成。以其强大的安全功能和全面的管理能力而闻名。

+   **[Okta](https://www.okta.com/products/single-sign-on/)** 是一个流行的基于云的SSO解决方案，以其易用性、可扩展性和广泛的应用集成而闻名。对于寻求全面身份和访问管理（IAM）平台的组织来说，它是一个强大的选择。

+   **[Ping Identity](https://www.pingidentity.com/en/platform/capabilities/single-sign-on.html)**。以其灵活性闻名，Ping Identity满足具有复杂安全要求的企业。它提供强大的移动和API安全选项，适合需要高度定制和安全性的组织。

+   **[OneLogin](https://www.onelogin.com/product/sso)**。专注于简单性和集成性，OneLogin提供了一种适合中小型企业的直接SSO解决方案。它提供实时威胁检测和AI驱动的认证，以增强安全性。

+   **[Auth0](https://auth0.com/learn/how-to-implement-single-sign-on)** 的开发者友好方法备受青睐。它提供强大的定制选项，成为需要定制身份验证流程的组织的首选。它支持广泛的编程语言和框架。

* * *

1.  **1:1 辅导：** [与我预约一次工作会议](https://newsletter.techworld-with-milan.com/p/coaching-services)。个人和组织/团队成长话题均可进行1:1辅导。我帮助您成为高效率的领导者 🚀。

1.  **[向超过27,000位订阅者推广自己](https://newsletter.techworld-with-milan.com/p/sponsorship-of-tech-world-with-milan)**，通过赞助本期新闻简报。

* * *
