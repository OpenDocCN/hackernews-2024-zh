<!--yml
category: 未分类
date: 2024-05-27 13:33:51
-->

# PEP 686 – Make UTF-8 mode default | peps.python.org

> 来源：[https://peps.python.org/pep-0686/](https://peps.python.org/pep-0686/)

# PEP 686 – Make UTF-8 mode default

Author:

Inada Naoki <songofacandy at gmail.com>

Discussions-To:

[Discourse thread](https://discuss.python.org/t/14737)

Status:

Accepted

Type:

Standards Track

Created:

18-Mar-2022

Python-Version:

3.15

Post-History:

[18-Mar-2022](https://discuss.python.org/t/14435 "Discourse thread"), [31-Mar-2022](https://discuss.python.org/t/14737 "Discourse thread")

Resolution:

[Discourse message](https://discuss.python.org/t/14737/9)

* * *

This PEP proposes enabling [UTF-8 mode](../pep-0540/ "PEP 540 – Add a new UTF-8 Mode") by default.

With this change, Python consistently uses UTF-8 for default encoding of files, stdio, and pipes.

UTF-8 becomes de facto standard text encoding.

*   The default encoding of Python source files is UTF-8.
*   JSON, TOML, YAML use UTF-8.
*   Most text editors, including Visual Studio Code and Windows Notepad use UTF-8 by default.
*   Most websites and text data on the internet use UTF-8.
*   And many other popular programming languages, including Node.js, Go, Rust, and Java uses UTF-8 by default.

Changing the default encoding to UTF-8 makes it easier for Python to interoperate with them.

Additionally, many Python developers using Unix forget that the default encoding is platform dependent. They omit to specify `encoding="utf-8"` when they read text files encoded in UTF-8 (e.g. JSON, TOML, Markdown, and Python source files). Inconsistent default encoding causes many bugs.

Python will enable UTF-8 mode by default from Python 3.15.

Users can still disable UTF-8 mode by setting `PYTHONUTF8=0` or `-X utf8=0`.

Since UTF-8 mode affects `locale.getpreferredencoding(False)`, we need an API to get locale encoding regardless of UTF-8 mode.

`locale.getencoding()` will be added for this purpose. It returns locale encoding too, but ignores UTF-8 mode.

When `warn_default_encoding` option is specified, `locale.getpreferredencoding()` will emit `EncodingWarning` like `open()` (see also [PEP 597](../pep-0597/ "PEP 597 – Add optional EncodingWarning")).

This API was added in Python 3.11.

[PEP 597](../pep-0597/ "PEP 597 – Add optional EncodingWarning") added the `encoding="locale"` option to the `TextIOWrapper`. This option is used to specify the locale encoding explicitly. `TextIOWrapper` should use locale encoding when the option is specified, regardless of default text encoding.

But `TextIOWrapper` uses `"UTF-8"` in UTF-8 mode even if `encoding="locale"` is specified for now. This behavior is inconsistent with the [PEP 597](../pep-0597/ "PEP 597 – Add optional EncodingWarning") motivation. It is because we didn’t expect making UTF-8 mode default when Python changes its default text encoding.

This inconsistency should be fixed before making UTF-8 mode default. `TextIOWrapper` should use locale encoding when `encoding="locale"` is passed even in UTF-8 mode.

This issue was fixed in Python 3.11.

Most Unix systems use UTF-8 locale and Python enables UTF-8 mode when its locale is C or POSIX. So this change mostly affects Windows users.

When a Python program depends on the default encoding, this change may cause `UnicodeError`, mojibake, or even silent data corruption. So this change should be announced loudly.

This is the guideline to fix this backward compatibility issue:

1.  Disable UTF-8 mode.
2.  Use `EncodingWarning` ([PEP 597](../pep-0597/ "PEP 597 – Add optional EncodingWarning")) to find every places UTF-8 mode affects.
    *   If `encoding` option is omitted, consider using `encoding="utf-8"` or `encoding="locale"`.
    *   If `locale.getpreferredencoding()` is used, consider using `"utf-8"` or `locale.getencoding()`.
3.  Test the application with UTF-8 mode.

*   Ruby [changed](https://bugs.ruby-lang.org/issues/16604) the default `external_encoding` to UTF-8 on Windows in Ruby 3.0 (2020).
*   Java [changed](https://openjdk.java.net/jeps/400) the default text encoding to UTF-8 in JDK 18\. (2022).

Both Ruby and Java have an option for backward compatibility. They don’t provide any warning like [PEP 597](../pep-0597/ "PEP 597 – Add optional EncodingWarning")’s `EncodingWarning` in Python for use of the default encoding.

Deprecating the use of the default encoding is considered.

But there are many cases that the default encoding is used for reading/writing only ASCII text. Additionally, such warnings are not useful for non-cross platform applications run on Unix.

So forcing users to specify the `encoding` everywhere is too painful. Emitting a lot of `DeprecationWarning` will lead users ignore warnings.

[PEP 387](../pep-0387/ "PEP 387 – Backwards Compatibility Policy") requires adding a warning for backward incompatible changes. But it doesn’t require using `DeprecationWarning`. So using optional `EncodingWarning` doesn’t violate the [PEP 387](../pep-0387/ "PEP 387 – Backwards Compatibility Policy").

Java also rejected this idea in [JEP 400](https://openjdk.java.net/jeps/400).

To ease backward compatibility issue, using `PYTHONIOENCODING` as the default encoding of PIPEs in the `subprocess` module is considered.

With this idea, users can use legacy encoding for `subprocess.Popen(text=True)` even in UTF-8 mode.

But this idea makes “default encoding” complicated. And this idea is also backward incompatible.

So this idea is rejected. Users can disable UTF-8 mode until they replace `text=True` with `encoding="utf-8"` or `encoding="locale"`.

For new users, this change reduces things that need to teach. Users don’t need to learn about text encoding in their first year. They should learn it when they need to use non-UTF-8 text files.

For existing users, see the [Backward compatibility](#backward-compatibility) section.

This document is placed in the public domain or under the CC0-1.0-Universal license, whichever is more permissive.