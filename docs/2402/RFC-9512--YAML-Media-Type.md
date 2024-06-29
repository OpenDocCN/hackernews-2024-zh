<!--yml

category: 未分类

date: 2024-05-27 15:03:33

-->

# RFC 9512: YAML媒体类型

> 来源：[https://www.rfc-editor.org/rfc/rfc9512.html](https://www.rfc-editor.org/rfc/rfc9512.html)

本文档中的关键词"MUST"、"MUST NOT"、"REQUIRED"、"SHALL"、"SHALL NOT"、"SHOULD"、"SHOULD NOT"、"RECOMMENDED"、"NOT RECOMMENDED"、"MAY"和"OPTIONAL"，当且仅当它们全部大写出现时（如此处所示），应按照BCP 14 [[RFC2119](#RFC2119)] [[RFC8174](#RFC8174)]中所述进行解释。[¶](#section-1.1-1)

本文档中的术语"content negotiation"和"resource"应按照[[HTTP](#RFC9110)]中的定义解释。[¶](#section-1.1-2)

本文档中的术语"fragment"和"fragment identifier"应按照[[URI](#RFC3986)]中的定义解释。[¶](#section-1.1-3)

本文档中的术语"presentation"、"stream"、"YAML document"、"representation graph"、"tag"、"serialization detail"、"node"、"alias node"、"anchor"和"anchor name"应按[[YAML](#YAML)]中的定义解释。[¶](#section-1.1-4)

包含YAML代码的图形始终以`%YAML`指令开头，以提高可读性。[¶](#section-1.1-5)
