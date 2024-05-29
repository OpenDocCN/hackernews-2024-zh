<!--yml
category: 未分类
date: 2024-05-27 15:03:33
-->

# RFC 9512: YAML Media Type

> 来源：[https://www.rfc-editor.org/rfc/rfc9512.html](https://www.rfc-editor.org/rfc/rfc9512.html)

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in BCP 14 [[RFC2119](#RFC2119)] [[RFC8174](#RFC8174)] when, and only when, they appear in all capitals, as shown here.[¶](#section-1.1-1)

The terms "content negotiation" and "resource" in this document are to be interpreted as in [[HTTP](#RFC9110)].[¶](#section-1.1-2)

The terms "fragment" and "fragment identifier" in this document are to be interpreted as in [[URI](#RFC3986)].[¶](#section-1.1-3)

The terms "presentation", "stream", "YAML document", "representation graph", "tag", "serialization detail", "node", "alias node", "anchor", and "anchor name" in this document are to be interpreted as in [[YAML](#YAML)].[¶](#section-1.1-4)

Figures containing YAML code always start with the `%YAML` directive to improve readability.[¶](#section-1.1-5)