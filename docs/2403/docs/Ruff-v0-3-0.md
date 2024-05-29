<!--yml
category: 未分类
date: 2024-05-27 14:30:06
-->

# Ruff v0.3.0

> 来源：[https://astral.sh/blog/ruff-v0.3.0](https://astral.sh/blog/ruff-v0.3.0)

[Ruff v0.3.0](https://github.com/astral-sh/ruff) is available now! Install it from [PyPI](https://pypi.org/project/ruff/), or your package manager of choice:

```
pip install  --upgrade  ruff
```

As a reminder: **Ruff is an extremely fast Python linter and formatter, written in Rust.** Ruff can be used to replace Black, Flake8 (plus dozens of plugins), isort, pydocstyle, pyupgrade, and more, all while executing tens or hundreds of times faster than any individual tool.

In October, we announced the [production-ready Beta of Ruff's formatter](https://astral.sh/blog/the-ruff-formatter). Since then, we've focused on improving compatibility between the linter and formatter, fixing deviations from Black, and implementing Ruff's 2024.2 formatting style. This release promotes the formatter to stable and ships Ruff's 2024.2 style guide.

## The Ruff 2024.2 style guide [#](#the-ruff-20242-style-guide)

The new style guide follows the principle that Ruff is designed as a drop-in replacement for Black. It is heavily inspired by Black's 2024 stable style guide (released as part of [Black 24.1.0](https://github.com/psf/black/releases/tag/24.1.0)). We'd like to offer our thanks to the Black maintainers for their fantastic work leading the code style discussions and defining their new style guide.

The rest of this section covers the highlights of Ruff's new style guide. Head to the [release](https://github.com/astral-sh/ruff/releases/tag/v0.3.0) for a detailed list of all style changes.

### Prefer wrapping an assignment's value [#](#prefer-wrapping-an-assignments-value)

Until now, Ruff preferred breaking an assignment's target over its value. The new style favors wrapping the assignment value before the assignment target or its type annotation.

```
# 2023
query_settings[
 "base_backup"
] =  f"S3('{base_backup}/{shard}', '{aws_key}', '{aws_secret}')"
 # 2024
query_settings["base_backup"] = (
 f"S3('{base_backup}/{shard}', '{aws_key}', '{aws_secret}')"
)
```

### Parenthesize multiple context managers in `with` statements [#](#parenthesize-multiple-context-managers-in-with-statements)

Previously, Ruff formatted context managers in `with` statements on a single line, even if they exceeded the configured line length. Ruff now parenthesizes long context managers when targeting Python 3.9 or later, and wraps them across multiple lines.

```
# 2023, or when targeting Python < 3.9
with make_context_manager1() as cm1, make_context_manager2() as cm2, make_context_manager3() as cm3, make_context_manager4() as cm4:
 pass
 # 2024 when targeting Python 3.9+
with (
 make_context_manager1() as cm1,
 make_context_manager2() as cm2,
 make_context_manager3() as cm3,
 make_context_manager4() as cm4,
):
 pass
```

### More condensed formatting of functions and classes with a dummy body [#](#more-condensed-formatting-of-functions-and-classes-with-a-dummy-body)

The new style reduces the vertical space for dummy implementations by collapsing the ellipsis (`...`) onto the same line. Blank lines between dummy function definitions are also now optional.

```
# 2023
@overload
def  f():
 ...
 def  f(a:  int  |  None):
 ...
 class  Test:
 ...
 # 2024
@overload
def  f(): ...
def  f(a:  int  |  None): ...
 class  Test: ...
```

### Improved multiline string formatting [#](#improved-multiline-string-formatting)

In the 2023 style guide, Ruff always indented multiline strings, even if they were the only arguments to a function call. The new style guide avoids indenting multiline strings if the call's opening parentheses and the string's quotes are on the same line, reducing vertical space.

```
# Input
textwrap.dedent("""
 This is a
 multiline string
""")
 # 2023
textwrap.dedent(
 """
 This is a
 multiline string
"""
)
 # 2024
textwrap.dedent("""
 This is a
 multiline string
""")
```

## Range Formatting [#](#range-formatting)

Ruff v0.2.1 added support for range formatting. IDEs use range formatting to limit formatting to a selected range (known as *Format Selection* in VS Code) or to [modified lines](https://code.visualstudio.com/updates/v1_49#_only-format-modified-text) based on version control information.

Restricting formatting to modified lines is especially useful if:

*   Your project is incrementally adopting a formatter; or,
*   You're working on a project that doesn't use standardized formatting, but you'd like to use Ruff to format a specific region of code to which you've made changes.

</static/WEBM/v0.3/format_modified_lines.webm>

The user adds a new argument to an existing function call. VS Code reformats the file on save using Ruff to format the call expression. 

In the above video, VS Code reformats the modified function call, but leaves the `__init__` function definition unaltered, since it wasn't changed by the user.

To enable formatting of modified lines in VS code, set `editor.formatOnSaveMode` to `"modificationsIfAvailable"`.

## f-string placeholder formatting [#](#f-string-placeholder-formatting)

Ruff v0.2.2 added support for formatting within f-strings (in `--preview`). You no longer need to manually format expressions inside placeholders; Ruff can take care of it for you.

```
 console.print(f"[red]Error!: {e}[/]")
 console.print(
-    f"[yellow]Restart with `--start-from {processed_issues+start_from}` to continue.[/]"
+    f"[yellow]Restart with `--start-from {processed_issues + start_from}` to continue.[/]"
 )
```

We're collaborating with Black to define a standardized style for f-string formatting. Please [share your feedback](https://github.com/astral-sh/ruff/discussions/9785) and help us shape f-string formatting going forward.

## Lint for invalid formatter suppression comments [#](#lint-for-invalid-formatter-suppression-comments)

Ruff's formatter has stricter requirements on [formatter suppression comment](https://docs.astral.sh/ruff/formatter/#format-suppression) placement than Black, motivated in favor of predictable formatting.

However, the difference has confused users, since it was unclear *why* Ruff ignored specific suppression comments that Black did not. Ruff v0.3.0 ships with the new lint rule [`RUF028`](https://docs.astral.sh/ruff/rules/RUF028), which identifies invalid formatter suppression comments; that is, `# fmt: off` comments that Ruff's formatter will not respect.

```
def  fmt_off_between_lists():
 test_list = [
 # fmt: off
 1,
 2,
 3,
 ]
```

`RUF028` generates a diagnostic for the use of `# fmt: off` on the above snippet, because Ruff (unlike Black) doesn't support `# fmt: off` inside expressions:

```
RUF028.py:3:9: RUF028 This suppression comment is invalid because it cannot be in an expression, pattern, argument list, or other non-statement
  |
1 | def fmt_off_between_lists():
2 |     test_list = [
3 |         # fmt: off
  |         ^^^^^^^^^^ RUF028
4 |         1,
5 |         2,
  |
  = help: Remove this comment 
```

## Thank you! [#](#thank-you)

Thank you to everyone that provided feedback regarding the formatter and other changes included in Ruff's `--preview` mode, and especially, to our contributors. It's an honor building Ruff with you!

* * *

View the full changelog on [GitHub](https://github.com/astral-sh/ruff/releases/tag/v0.3.0).

Read more about [Astral](https://astral.sh/about) — the company behind Ruff.