<!--yml
category: 未分类
date: 2024-05-27 15:02:16
-->

# SAML V2.0 Deployment Profile for Federation Interoperability Version 2.0

> 来源：[https://kantarainitiative.github.io/SAMLprofiles/saml2int.html](https://kantarainitiative.github.io/SAMLprofiles/saml2int.html)

This revision, the first major rewrite of this material, reflects the input of many experienced implementers and deployers of this technology and best practice developed over 15 years of experience with varied approaches. While it has an emphasis on highly-scaled multi-lateral federation deployments involving thousands of Identity Providers (IdPs) and Service Providers (SPs), most of these requirements are applicable to virtually any significant deployment of SAML SSO.

The requirements specified are in addition to all normative requirements of the underlying Web Browser SSO and Single Logout profiles [[SAML2Prof]](#SAML2Prof), as modified by the Approved Errata [[SAML2Err]](#SAML2Err), and readers are assumed to be familiar with all relevant reference documents. Any such requirements are not repeated here except where deemed necessary to highlight a point of discussion or draw attention to an issue addressed in errata, but remain implied.

Nothing in this profile should be taken to imply that disclosing personally identifiable information, or indeed any information, is required from an IdP with respect to any particular SP. That remains at the discretion of applicable settings, user consent, or other appropriate means in accordance with regulations and policies. However, this profile does obligate IdPs to honor certain key signals from an SP in the area of subject identification and requires successful responses to include specific SAML Attributes [[X500SAMLattr]](#X500SAMLattr) under certain conditions. Failure is always an option.

Note that SAML features that are optional, or lack mandatory processing rules, are assumed to be optional and out of scope of this profile if not otherwise precluded or given specific processing rules.

This profile addresses only behavior specific to the direct participants in the covered profiles. It does not include additional processing requirements that may be important when proxying.

### 1.1\. Notation and Terminology

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in BCP 14 [[RFC2119]](#RFC2119) [[RFC8174]](#RFC8174) when, and only when, they appear in all capitals, as shown here.

This specification uses the following typographical conventions in text: `<ns:Element>`, `Attribute`, **Datatype**, `OtherCode`. The normative requirements of this specification are individually labeled with a unique identifier in the following form: **[SDP-EXAMPLE01]**. All information within these requirements should be considered normative unless it is set in *italic* type. Italicized text is non-normative and is intended to provide additional information that may be helpful in implementing the normative requirements.

The abbreviations IdP and SP are used below to refer to Identity Providers and Service Providers in the sense of their usage within the SAML Browser SSO and Single Logout profiles.

Whether explicit or implicit, all the requirements in this document are meant to apply to deployments of SAML profiles and may involve explicit support for requirements by SAML-implementing software and/or supplemental support via application code. Deployments of a Service Provider may refer to both stand-alone implementations of SAML, libraries integrated with an application, or any combination of the two. It is difficult to define a clear boundary between a Service Provider and the application/service it represents, and unnecessary to do so for the purposes of this document.

#### 1.1.1\. References to SAML 2.0 specification

When referring to elements from the SAML 2.0 core specification [[SAML2Core]](#SAML2Core), the following syntax is used:

When referring to elements from the SAML 2.0 metadata specification [[SAML2Meta]](#SAML2Meta), the following syntax is used:

When referring to elements from the SAML 2.0 Metadata Extensions for Login and Discovery User Interface specification [[MetaUI]](#MetaUI), the following syntax is used:

When referring to elements from the SAML 2.0 Metadata Extension for Entity Attributes specification [[MetaAttr]](#MetaAttr), the following syntax is used:

When referring to elements from the SAML V2.0 Subject Identifier Attributes Profile specification [[SAML2SubjId]](#SAML2SubjId), the following syntax is used:

When referring to elements from the SAML V2.0 Asynchronous Single Logout Protocol Extension specification [[SAML2ASLO]](#SAML2ASLO), the following syntax is used:

When referring to elements from the XML-Signature Syntax and Processing Version 1.1 WWWC Recommendation [[XMLSig]](#XMLSig), the following syntax is used: