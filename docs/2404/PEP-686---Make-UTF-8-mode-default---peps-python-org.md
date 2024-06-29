<!--yml

分类：未分类

date: 2024-05-27 13:33:51

-->

# PEP 686 – 使 UTF-8 模式成为默认 | peps.python.org

> 来源：[https://peps.python.org/pep-0686/](https://peps.python.org/pep-0686/)

# PEP 686 – 使 UTF-8 模式成为默认

作者：

Inada Naoki <songofacandy at gmail.com>

讨论地址：

[Discourse 主题](https://discuss.python.org/t/14737)

状态：

已接受

类型：

标准跟踪

创建于：

18-Mar-2022

Python 版本：

3.15

发表历史：

[18-Mar-2022](https://discuss.python.org/t/14435 "Discourse 主题"), [31-Mar-2022](https://discuss.python.org/t/14737 "Discourse 主题")

决议：

[Discourse 消息](https://discuss.python.org/t/14737/9)

* * *

此 PEP 建议默认启用 [UTF-8 模式](../pep-0540/ "PEP 540 – Add a new UTF-8 Mode")。

这一变更使得 Python 在文件、stdio 和管道的默认编码上始终使用 UTF-8。

UTF-8 已成为事实上的标准文本编码。

+   Python 源文件的默认编码是 UTF-8。

+   JSON、TOML 和 YAML 使用 UTF-8。

+   大多数文本编辑器，包括 Visual Studio Code 和 Windows 记事本，默认使用 UTF-8。

+   大多数网站和互联网上的文本数据使用 UTF-8。

+   以及许多其他流行的编程语言，包括 Node.js、Go、Rust 和 Java，默认情况下都使用 UTF-8。

将默认编码更改为 UTF-8 使 Python 更易于与它们互操作。

此外，许多使用 Unix 的 Python 开发者忽视了默认编码取决于平台的事实。当他们读取以 UTF-8 编码的文本文件（例如 JSON、TOML、Markdown 和 Python 源文件）时，他们忘记指定 `encoding="utf-8"`。不一致的默认编码导致了许多 bug。

Python 将从 Python 3.15 开始默认启用 UTF-8 模式。

用户仍然可以通过设置 `PYTHONUTF8=0` 或 `-X utf8=0` 来禁用 UTF-8 模式。

由于 UTF-8 模式影响 `locale.getpreferredencoding(False)`，我们需要一个 API 来获取不受 UTF-8 模式影响的区域设置编码。

`locale.getencoding()` 将被添加用于此目的。它也返回区域设置编码，但忽略 UTF-8 模式。

当指定了`warn_default_encoding`选项时，`locale.getpreferredencoding()`会像`open()`一样发出`EncodingWarning`（参见[PEP 597](../pep-0597/ "PEP 597 – Add optional EncodingWarning")）。

此 API 在 Python 3.11 中添加。

[PEP 597](../pep-0597/ "PEP 597 – Add optional EncodingWarning") 添加了`encoding="locale"`选项到`TextIOWrapper`。此选项用于显式指定区域设置编码。当指定了此选项时，`TextIOWrapper` 应该使用区域设置编码，而不管默认文本编码是什么。

但是，即使在 UTF-8 模式下，`TextIOWrapper` 目前也会在指定 `encoding="locale"` 时使用 `"UTF-8"`。这种行为与[PEP 597](../pep-0597/ "PEP 597 – Add optional EncodingWarning")的动机不一致。这是因为我们没有预料到当 Python 更改其默认文本编码时会使 UTF-8 模式成为默认。

在使 UTF-8 模式成为默认之前，应修复此不一致性。即使在 UTF-8 模式下，当传递 `encoding="locale"` 时，`TextIOWrapper` 应该使用区域设置编码。

这个问题在 Python 3.11 中已经修复。

大多数 Unix 系统使用 UTF-8 语言环境，而 Python 在其语言环境为 C 或 POSIX 时启用 UTF-8 模式。因此，这一变更主要影响 Windows 用户。

当 Python 程序依赖于默认编码时，这一变更可能导致 `UnicodeError`、乱码或甚至是静默数据损坏。因此，这一变更应该被大声宣布。

这是解决这一向后兼容性问题的指南：

1.  禁用 UTF-8 模式。

1.  使用 `EncodingWarning` ([PEP 597](../pep-0597/ "PEP 597 – Add optional EncodingWarning")) 来查找所有受 UTF-8 模式影响的地方。

    +   如果省略了 `encoding` 选项，请考虑使用 `"utf-8"` 或 `"locale"` 作为编码。

    +   如果使用 `locale.getpreferredencoding()`，考虑使用 `"utf-8"` 或 `locale.getencoding()`。

1.  使用 UTF-8 模式测试应用程序。

+   Ruby 在 Ruby 3.0（2020 年）中将默认的 `external_encoding` 改为 UTF-8，在 Windows 上进行了更改（https://bugs.ruby-lang.org/issues/16604）。

+   Java 在 JDK 18 中（2022 年）将默认文本编码改为 UTF-8。详见 [JEP 400](https://openjdk.java.net/jeps/400)。

Ruby 和 Java 都有向后兼容的选项。它们不像 Python 中的 [PEP 597](../pep-0597/ "PEP 597 – Add optional EncodingWarning") 的 `EncodingWarning` 那样提供任何警告，用于使用默认编码。

弃用使用默认编码正在考虑中。

但是有许多情况下，默认编码仅用于读写 ASCII 文本。此外，这样的警告对于在 Unix 上运行的非跨平台应用程序并不实用。

因此，强制用户在每个地方指定 `encoding` 是非常痛苦的。发出大量的 `DeprecationWarning` 将导致用户忽略警告。

[PEP 387](../pep-0387/ "PEP 387 – Backwards Compatibility Policy") 要求对不向后兼容的更改添加警告。但并不要求使用 `DeprecationWarning`。因此，使用可选的 `EncodingWarning` 不违反 [PEP 387](../pep-0387/ "PEP 387 – Backwards Compatibility Policy")。

Java 也在 [JEP 400](https://openjdk.java.net/jeps/400) 中拒绝了这个想法。

为了缓解向后兼容性问题，在 `subprocess` 模块的 PIPEs 中使用 `PYTHONIOENCODING` 作为默认编码正在考虑中。

有了这个想法，即使在 UTF-8 模式下，用户也可以为 `subprocess.Popen(text=True)` 使用传统编码。

但这一想法使得“默认编码”变得复杂。并且这一想法也是向后不兼容的。

因此，这个想法被拒绝了。用户可以在将 `text=True` 替换为 `encoding="utf-8"` 或 `encoding="locale"` 之前禁用 UTF-8 模式。

对于新用户，这一变更减少了需要教授的内容。用户在第一年不需要学习关于文本编码的内容。他们在需要使用非 UTF-8 文本文件时再学习即可。

对于现有用户，请参阅 [向后兼容性](#backward-compatibility) 部分。

本文档属于公共领域，或者使用 CC0-1.0-Universal 许可证，以较宽松的那个为准。
