<!--yml

类别：未分类

日期：2024-05-27 14:30:06

-->

# Ruff v0.3.0

> 来源：[https://astral.sh/blog/ruff-v0.3.0](https://astral.sh/blog/ruff-v0.3.0)

[Ruff v0.3.0](https://github.com/astral-sh/ruff) 现已发布！您可以从 [PyPI](https://pypi.org/project/ruff/) 或您选择的软件包管理器中安装它：

```
pip install  --upgrade  ruff
```

提醒一下：**Ruff 是一个极快的用 Rust 编写的 Python linter 和 formatter。** Ruff 可以替代 Black、Flake8（加上数十个插件）、isort、pydocstyle、pyupgrade 等工具，同时执行速度比任何单个工具快十倍甚至更多。

十月份，我们宣布了 [Ruff 格式化器的生产就绪 Beta](https://astral.sh/blog/the-ruff-formatter)。自那以后，我们专注于改进 linter 和 formatter 之间的兼容性，修复与 Black 的偏差，并实施 Ruff 2024.2 的格式化样式。此版本将格式化器提升为稳定版，并发布了 Ruff 2024.2 样式指南。

## Ruff 2024.2 样式指南 [#](#the-ruff-20242-style-guide)

新的样式指南遵循这样一个原则，即 Ruff 被设计为 Black 的一个可替换组件。它深受 Black 2024 稳定样式指南的启发（作为 [Black 24.1.0](https://github.com/psf/black/releases/tag/24.1.0) 的一部分发布）。我们要感谢 Black 的维护者们在领导代码风格讨论和定义他们的新样式指南方面做出的出色工作。

本节的其余部分涵盖了 Ruff 新样式指南的亮点。请前往 [发布页面](https://github.com/astral-sh/ruff/releases/tag/v0.3.0) 查看所有样式更改的详细列表。

### 推荐包装赋值的值 [#](#prefer-wrapping-an-assignments-value)

到目前为止，Ruff 更喜欢在赋值目标之前而不是赋值值之前进行换行。新的样式偏好于在赋值目标或其类型注释之前包装赋值值。

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

### 在 `with` 语句中给多个上下文管理器加括号 [#](#parenthesize-multiple-context-managers-in-with-statements)

以前，Ruff 在 `with` 语句中格式化上下文管理器时，即使超过了配置的行长度，也会将它们放在单行上。现在，如果目标是 Python 3.9 或更高版本，则 Ruff 在长上下文管理器上加括号，并跨多行进行包装。

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

### 使用虚拟主体更紧凑地格式化函数和类 [#](#more-condensed-formatting-of-functions-and-classes-with-a-dummy-body)

新样式通过将省略号 (`...`) 折叠到同一行来减少虚拟实现的垂直空间。虚拟函数定义之间的空行现在也是可选的。

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

### 改进的多行字符串格式化 [#](#improved-multiline-string-formatting)

在 2023 年的样式指南中，Ruff 总是缩进多行字符串，即使它们是函数调用的唯一参数。新的样式指南避免缩进多行字符串，如果调用的开括号和字符串的引号在同一行上，则减少垂直空间。

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

## 范围格式化 [#](#range-formatting)

Ruff v0.2.1 添加了对范围格式化的支持。IDE 使用范围格式化来限制格式化到选择的范围内（在 VS Code 中称为 *格式化所选内容*）或者基于版本控制信息的[修改行](https://code.visualstudio.com/updates/v1_49#_only-format-modified-text)。

如果只对修改后的行进行格式化限制，这将特别有用：

+   您的项目正在逐步采用格式化工具；或者，

+   您正在处理一个不使用标准格式化的项目，但希望使用 Ruff 来格式化您已修改代码的特定区域。

</static/WEBM/v0.3/format_modified_lines.webm>

用户向现有函数调用添加了一个新参数。VS Code 使用 Ruff 在保存时重新格式化文件以格式化调用表达式。

在上述视频中，VS Code 重新格式化了修改后的函数调用，但未更改 `__init__` 函数定义，因为用户未更改它。

要在 VS Code 中启用修改行的格式化，请将 `editor.formatOnSaveMode` 设置为 `"modificationsIfAvailable"`。

## f-string 占位符格式化 [#](#f-string-placeholder-formatting)

Ruff v0.2.2 添加了对 f-string 内部（在 `--preview` 中）格式化的支持。您不再需要手动格式化占位符内的表达式；Ruff 可以为您处理。

```
 console.print(f"[red]Error!: {e}[/]")
 console.print(
-    f"[yellow]Restart with `--start-from {processed_issues+start_from}` to continue.[/]"
+    f"[yellow]Restart with `--start-from {processed_issues + start_from}` to continue.[/]"
 )
```

我们正在与 Black 合作，为 f-string 格式化定义一个标准化的样式。请[分享您的反馈意见](https://github.com/astral-sh/ruff/discussions/9785)，帮助我们塑造未来的 f-string 格式化。

## 无效格式化抑制注释的 lint [#](#lint-for-invalid-formatter-suppression-comments)

Ruff 的格式化器在 [格式化抑制注释](https://docs.astral.sh/ruff/formatter/#format-suppression) 的放置上比 Black 更严格，以支持可预测的格式化。

然而，这种差异让用户感到困惑，因为不清楚为什么 Ruff 忽略了 Black 没有忽略的特定抑制注释。Ruff v0.3.0 提供了新的 lint 规则 [`RUF028`](https://docs.astral.sh/ruff/rules/RUF028)，用于识别无效的格式化抑制注释；即 Ruff 格式化器不会尊重的 `# fmt: off` 注释。

```
def  fmt_off_between_lists():
 test_list = [
 # fmt: off
 1,
 2,
 3,
 ]
```

`RUF028` 生成了一个关于上述片段中使用 `# fmt: off` 的诊断，因为 Ruff（不像 Black）不支持在表达式中使用 `# fmt: off`：

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

## 谢谢！[#](#thank-you)

感谢所有提供关于格式化器和其他 Ruff `--preview` 模式中包含的更改反馈的人，特别是感谢我们的贡献者。能与您一起构建 Ruff 是我们的荣幸！

* * *

在 [GitHub](https://github.com/astral-sh/ruff/releases/tag/v0.3.0) 上查看完整的更改日志。

了解更多关于 [Astral](https://astral.sh/about) — Ruff 背后公司的信息。
