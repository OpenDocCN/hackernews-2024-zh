<!--yml

category: 未分类

date: 2024-05-27 15:02:16

-->

# SAML V2.0 用于联合互操作性的部署配置文件 版本 2.0

> 来源：[https://kantarainitiative.github.io/SAMLprofiles/saml2int.html](https://kantarainitiative.github.io/SAMLprofiles/saml2int.html)

此次修订，这些材料的第一次重大重写，反映了许多经验丰富的实施者和部署者对这项技术的意见，以及在使用多种方法的 15 年经验中发展的最佳实践。虽然它侧重于涉及数千个身份提供者（IdPs）和服务提供者（SPs）的高度扩展的多边联合部署，但这些要求大多适用于几乎任何重要的 SAML SSO 部署。

指定的要求是在基础 Web 浏览器 SSO 和单点注销配置文件[[SAML2Prof]](#SAML2Prof) 的所有规范要求之外，并由已批准的勘误[[SAML2Err]](#SAML2Err) 修改，读者假定熟悉所有相关参考文档。除了在需要突出讨论点或引起注意的问题处重复之外，不重复列出任何此类要求，但仍然暗示。

本配置文件中的任何内容都不应被视为要求 IdP 从某个特定 SP 获取披露个人可识别信息，或者实际上任何信息。这仍然取决于适用的设置、用户同意或其他适当的手段，符合法规和政策的规定。然而，本配置文件确实要求 IdP 遵守来自 SP 的某些关键信号在主体识别领域和特定条件下要求包含特定的 SAML 属性[[X500SAMLattr]](#X500SAMLattr)。失败总是一个选择。

请注意，如果没有明确禁止或给出特定处理规则，则假定 SAML 功能是可选的或缺少强制处理规则，不在此配置文件的范围内。

此配置文件仅涉及所覆盖的配置文件中直接参与者的特定行为。它不包括在代理时可能重要的额外处理要求。

### 1.1\. 符号和术语

本文档中的关键字"MUST"、"MUST NOT"、"REQUIRED"、"SHALL"、"SHALL NOT"、"SHOULD"、"SHOULD NOT"、"RECOMMENDED"、"NOT RECOMMENDED"、"MAY" 和 "OPTIONAL" 在出现在全大写时（如本处所示），应按照 BCP 14 [[RFC2119]](#RFC2119) [[RFC8174]](#RFC8174) 的描述进行解释。

本规范在文本中使用以下排版约定：`<ns:Element>`，`Attribute`，**Datatype**，`OtherCode`。本规范的规范要求通过以下形式进行单独标记：**[SDP-EXAMPLE01]**。所有这些要求中的信息应视为规范，除非以*斜体*方式设置。斜体文本为非规范性的，旨在提供可能有助于实施规范要求的额外信息。

下面使用缩写 IdP 和 SP 来指代 SAML 浏览器 SSO 和单点登出配置中的身份提供者和服务提供者。

无论是显式还是隐式的，本文档中的所有要求都适用于 SAML 配置的部署，并可能涉及 SAML 实现软件明确支持要求和/或通过应用代码提供的补充支持。服务提供者的部署可以涵盖独立的 SAML 实现、集成到应用程序中的库或二者的任意组合。很难为服务提供者及其所代表的应用程序/服务之间定义清晰的边界，在本文档的目的上也没有必要这样做。

#### 1.1.1\. SAML 2.0 规范的参考文献

当引用 SAML 2.0 核心规范中的元素 [[SAML2Core]](#SAML2Core) 时，使用以下语法：

当引用 SAML 2.0 元数据规范中的元素 [[SAML2Meta]](#SAML2Meta) 时，使用以下语法：

当引用 SAML 2.0 Metadata Extensions for Login and Discovery User Interface 规范 [[MetaUI]](#MetaUI) 中的元素时，使用以下语法：

当引用 SAML 2.0 Metadata Extension for Entity Attributes 规范 [[MetaAttr]](#MetaAttr) 中的元素时，使用以下语法：

当引用 SAML V2.0 Subject Identifier Attributes Profile 规范 [[SAML2SubjId]](#SAML2SubjId) 中的元素时，使用以下语法：

当引用 SAML V2.0 Asynchronous Single Logout Protocol Extension 规范 [[SAML2ASLO]](#SAML2ASLO) 中的元素时，使用以下语法：

当引用 XML-Signature Syntax and Processing Version 1.1 WWWC Recommendation [[XMLSig]](#XMLSig) 中的元素时，使用以下语法：
