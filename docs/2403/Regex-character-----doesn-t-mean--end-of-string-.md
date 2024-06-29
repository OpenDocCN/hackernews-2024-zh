<!--yml

category: 未分类

date: 2024-05-29 12:31:57

-->

# 正则表达式字符“$”并不表示“字符串的末尾”

> 来源：[https://sethmlarson.dev/regex-$-matches-end-of-string-or-newline](https://sethmlarson.dev/regex-$-matches-end-of-string-or-newline)

这篇文章讨论了我最近在使用Python的正则表达式模块（`re`）开发[CPython的SBOM工具](https://github.com/python/release-tools/pull/92#discussion_r1484470272)时发现的一些令人惊讶的行为。

以前使用过正则表达式的人可能知道`^`表示“字符串的开头”，并相应地将`$`看作“字符串的结尾”。因此，模式`cat$`将匹配字符串`"lolcat"`，但不会匹配`"internet cat video"`。

字符`^`的行为让我以为`$`类似，但它们并不总是对称的，且行为*依赖于平台*。特别是对于禁用多行模式的Python，字符`$`可以匹配字符串的结尾或者字符串结尾前的*尾随换行符*。

因此，如果你尝试在没有换行符的字符串末尾匹配，**在Python中只用'$'是不行的！** 我的期望是禁用多行模式不会有这种匹配换行符的行为，但事实并非如此。

下一个合理的问题是如何在Python中匹配没有换行符的字符串末尾？

在对[Python](https://docs.python.org/3/library/re.html#regular-expression-syntax)和[其他正则表达式语法](https://www.regular-expressions.info/anchors.html)进行更多研究后，我还发现`\z`和`\Z`可以作为“字符串末尾”字符的候选项。

在Python中启用多行模式时，可以使用[`re.MULTILINE`](https://docs.python.org/3/library/re.html#re.MULTILINE)，文档中有以下描述：

> 当指定`re.MULTILINE`时，模式字符'$'会在字符串的末尾和每行的末尾（紧跟每个换行符）匹配。默认情况下，'$'仅在字符串的末尾和紧跟字符串末尾的换行符（如果有）之前匹配。

让我们看看这些功能如何在多个平台上一起工作：

| 匹配 `"cat\n"` 的模式？ | `"cat$"` 多行模式 | `"cat$"` 无多行模式 | `"cat\z"` | `"cat\Z"` |
| --- | --- | --- | --- | --- |
| PHP | ✅ | ✅ | ❌ | ✅ |
| ECMAScript | ✅ | ❌ | ⚠️ | ⚠️ |
| Python | ✅ | ✅ | ⚠️ | ❌ |
| Golang | ✅ | ❌ | ❌ | ⚠️ |
| Java 8 | ✅ | ✅ | ❌ | ✅ |
| .NET 7.0 | ✅ | ✅ | ❌ | ✅ |
| Rust | ✅ | ❌ | ❌ | ⚠️ |

+   ✅: 模式匹配字符串`"cat\n"`

+   ❌: 模式不匹配字符串`"cat\n"`

+   ⚠️: 模式无效或字符不支持。

总结上表，如果匹配尾随换行符是可接受的，那么在所有平台上，具有多行模式的`$`就能保持一致，但如果我们不希望匹配尾随换行符，则情况会变得更复杂。

在除了Python和ECMAScript以外的所有平台上，要匹配末尾的换行符，请使用`\z`；在Python和ECMAScript中，你需要分别使用`\Z`或不带多行模式的`$`。希望你今天对正则表达式学到了一些东西！

注意：数据表格的数据来自于[regex101.com](https://regex101.com)，我没有使用实际的运行时进行测试。

> **感谢阅读！** ♡ 您觉得这篇文章有帮助吗？想要更多类似内容？通过订阅[RSS feed](/feed)或[电子邮件通讯](https://buttondown.email/sethmlarson)来获取新文章的通知。
