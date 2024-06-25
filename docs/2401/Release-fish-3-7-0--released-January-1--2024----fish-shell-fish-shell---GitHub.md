<!--yml

类别：未分类

日期：2024-05-27 14:24:26

-->

# 发布 fish 3.7.0（发布于 2024 年 1 月 1 日） · fish-shell/fish-shell · GitHub

> 来源：[`github.com/fish-shell/fish-shell/releases/tag/3.7.0`](https://github.com/fish-shell/fish-shell/releases/tag/3.7.0)

这个版本的 fish 相比 fish 3.6.4 有许多改进，详见下文。尽管 fish 内部的移植工作继续使用 Rust 编程语言进行，但该工作未包含在此版本中。fish 3.7.0 和 3.7 系列中的任何未来版本仍然是 C++ 程序。

## 显著改进和修复

+   历史记录分页器的改进，包括：

    +   历史记录分页器现在还会尝试子序列匹配（[#9476](https://github.com/fish-shell/fish-shell/issues/9476)），因此您可以通过搜索`gitInt`找到类似 `git log 3.6.1..Integration_3.7.0` 这样的命令行。

    +   打开历史记录分页器现在会在搜索字段中填入搜索字符串，如果您已经处于搜索状态（[#10005](https://github.com/fish-shell/fish-shell/issues/10005)）。这样做可以更轻松地使用`↑`搜索内容，然后稍后决定切换到完整的分页器。

    +   通过回车关闭历史记录分页器现在会将搜索文本复制到命令行，如果没有匹配项，因此您可以立即继续编辑您尝试查找的命令（[#9934](https://github.com/fish-shell/fish-shell/issues/9934)）。

+   对命令补全和通配符匹配的性能进行了改进，如果操作系统支持的话，特别是在像 NFS 这样的慢文件系统上（[#9891](https://github.com/fish-shell/fish-shell/issues/9891)，[#9931](https://github.com/fish-shell/fish-shell/issues/9931)，[#10032](https://github.com/fish-shell/fish-shell/issues/10032)，[#10052](https://github.com/fish-shell/fish-shell/issues/10052)）。

+   现在可以配置 fish 等待一段指定的时间以完成多键序列，而不是无限期等待。例如，这使得将`kj`绑定到 vi 模式中的切换模式成为可能。

    超时可以通过新的 `fish_sequence_key_delay_ms` 变量进行设置（[#7401](https://github.com/fish-shell/fish-shell/issues/7401)），并且可能在未来版本中默认设置。

## 弃用和删除的功能

+   `LS_COLORS` 不再由 `ls` 函数自动设置（[#10080](https://github.com/fish-shell/fish-shell/issues/10080)）。用户

    设置 `.dircolors` 的脚本应该使用其他方式手动导入它。通常情况下，这将是`set -gx LS_COLORS (dircolors -c .dircolors | string split ' ')[3]`

## 脚本改进

+   使用负数运行 `exit` 不再导致 fish 崩溃（[#9659](https://github.com/fish-shell/fish-shell/issues/9659)）。

+   `fish --command` 现在如果解析失败将返回非零状态（[#9888](https://github.com/fish-shell/fish-shell/issues/9888)）。

+   `jobs` 内置功能现在会转义它打印的命令（[#9808](https://github.com/fish-shell/fish-shell/issues/9808)）。

+   如果计数是块大小的倍数，`string repeat` 现在不会溢出（[#9900](https://github.com/fish-shell/fish-shell/issues/9900)）。

+   `builtin` 内建现在将以无效参数正确报错，而不是什么都不做并返回 true（[#9942](https://github.com/fish-shell/fish-shell/issues/9942)）。

+   在管道中再次允许 `command time`，以及 `command and` 和 `command or`（[#9985](https://github.com/fish-shell/fish-shell/issues/9985)）。

+   `exec`现在也会应用变量覆盖，因此`FOO=bar exec`将正确设置`$FOO`（[#9995](https://github.com/fish-shell/fish-shell/issues/9995)）。

+   `umask` 现在将正确处理空符号模式，例如 `umask u=,g=rwx,o=`（[#10177](https://github.com/fish-shell/fish-shell/issues/10177)）。

+   改进了命令替换中发生的错误的错误消息（[#10054](https://github.com/fish-shell/fish-shell/issues/10054)）。

## 交互式改进

+   `read` 现在不再启用括号粘贴，因此它不会在组合的命令行中保持启用状态，例如 `mysql -p(read --silent)`（[#8285](https://github.com/fish-shell/fish-shell/issues/8285)）。

+   Vi 模式现在使用 `fish_cursor_external` 来为外部命令设置光标形状（[#4656](https://github.com/fish-shell/fish-shell/issues/4656)）。

+   在 vi 模式下打开历史搜索会正确切换到插入模式（[#10141](https://github.com/fish-shell/fish-shell/issues/10141)）。

+   Vi 模式下的光标形状现在在 iTerm2 中启用了（[#9698](https://github.com/fish-shell/fish-shell/issues/9698)）。

+   对于 iTerm2 启用了工作目录报告（[#9955](https://github.com/fish-shell/fish-shell/issues/9955)）。

+   作为 root 完成命令包括不属于 root 的命令，修复了 fish 3.2.0 中引入的退化（[#9699](https://github.com/fish-shell/fish-shell/issues/9699)）。

+   选择使用 `fish_color_selection` 设置前景和背景颜色，如预期，而不仅仅是背景（[#9717](https://github.com/fish-shell/fish-shell/issues/9717)）。

+   当移动长列表时，完成分页器不再有时会跳过最后一个条目（[#9833](https://github.com/fish-shell/fish-shell/issues/9833)）。

+   交互式 `history delete` 界面现在允许指定索引范围，例如“1..5”（[#9736](https://github.com/fish-shell/fish-shell/issues/9736）），`history delete --exact` 现在正确保存历史记录（[#10066](https://github.com/fish-shell/fish-shell/issues/10066)）。

+   命令完成现在将在 macOS 上调用原生的 `manpath`，而不是潜在的 Homebrew 版本。这样可以避免尴尬的错误消息（[#9817](https://github.com/fish-shell/fish-shell/issues/9817)）。

+   新的绑定函数 `history-pager-delete`，默认绑定到 `Shift` + `Delete`，将从历史记录中删除当前选择的历史页条目（[#9454](https://github.com/fish-shell/fish-shell/issues/9454)）。

+   `fish_key_reader`现在会直接使用可打印字符，因此按下“ö”不再导致它告诉你要绑定`\\u00F6`（[#9986](https://github.com/fish-shell/fish-shell/issues/9986)）。

+   `open`现在可以再次用于启动终端程序，因为已修复了`xdg-open`的错误，并删除了一个解决方法（[#10045](https://github.com/fish-shell/fish-shell/issues/10045)）。

+   `repaint-mode`绑定现在只会在需要重新绘制时移动光标。这修复了 vi 模式中的`Alt`组合绑定（[#7910](https://github.com/fish-shell/fish-shell/issues/7910)）。

+   默认情况下，使用新的`clear-screen`绑定函数清除屏幕和重新绘制现有提示，用于`Ctrl` + `l`。

    因此，除非终端非常慢，否则它会消除可见的闪烁（[#10044](https://github.com/fish-shell/fish-shell/issues/10044)）。

+   `alias`便利函数对具有不寻常字符的命令（如`+`）提供了更好的支持（[#8720](https://github.com/fish-shell/fish-shell/issues/8720)）。

+   已修复了一个长期存在的问题，即分页器中的项目有时会显示不正确的格式（[#9617](https://github.com/fish-shell/fish-shell/issues/9617)）。

+   `Alt` + `l`绑定，用于列出光标下标记的目录，正确展开波浪号（`~`）到主目录（[#9954](https://github.com/fish-shell/fish-shell/issues/9954)）。

+   现在，如果`PAGER`环境变量未设置，或者没有可用的分页器，则使用外部分页器的各种 fish 实用程序将尝试一些常见的分页器进行尝试，或者将输出写入屏幕而不使用分页器（[#10074](https://github.com/fish-shell/fish-shell/issues/10074)）。

+   特定命令的标签补全现在可以提供以句点开头的结果。例如，现在可以为具有前导句点的文件进行`git add`的标签补全。默认的文件补全会隐藏这些文件，除非令牌本身具有前导句点（[#3707](https://github.com/fish-shell/fish-shell/issues/3707)）。

### 改进了提示

+   默认主题现在仅使用命名颜色，因此它将跟踪终端的调色板（[#9913](https://github.com/fish-shell/fish-shell/issues/9913)）。

+   Dracula 主题现在已经与上游同步（[#9807](https://github.com/fish-shell/fish-shell/issues/9807)）； 使用`fish_config`重新应用它以获取这些更改。

+   `fish_vcs_prompt`现在也支持 fossil（[#9497](https://github.com/fish-shell/fish-shell/issues/9497)）。

+   使用`prompt_pwd`函数显示工作目录的提示正确显示以破折号开头的目录（[#10169](https://github.com/fish-shell/fish-shell/issues/10169)）。

### 补全

+   增加了补全：

+   `zfs`补全不再打印关于设置只读变量的错误（[#9705](https://github.com/fish-shell/fish-shell/issues/9705)）。

+   `kitty`补全已删除，以保留它们在上游的状态（[#9750](https://github.com/fish-shell/fish-shell/issues/9750)）。

+   `git` 补全现在支持引用其他别名的别名（[#9992](https://github.com/fish-shell/fish-shell/issues/9992)）。

+   `gw` 和 `gradlew` 补全现在可以正确加载（[#10127](https://github.com/fish-shell/fish-shell/issues/10127)）。

+   对许多其他补全进行改进。

+   改进手动页完成生成器（[#9787](https://github.com/fish-shell/fish-shell/issues/9787)、[#9814](https://github.com/fish-shell/fish-shell/issues/9814)、[#9961](https://github.com/fish-shell/fish-shell/issues/9961)）。

## 其他改进

+   对文档进行改进和修正。

+   网页配置现在在打印时使用更可读的样式，例如用于键绑定参考（[#9828](https://github.com/fish-shell/fish-shell/issues/9828)）。

+   更新德语翻译（[#9824](https://github.com/fish-shell/fish-shell/issues/9824)）。

+   Nord 主题的颜色更加匹配其官方风格（[#10168](https://github.com/fish-shell/fish-shell/issues/10168)）。

## 对于分发商

+   一些分发的 fish 衍生代码的许可信息不完整。虽然许可信息存在于源分发中，但未包含在文档中。这已得到修正（[#10162](https://github.com/fish-shell/fish-shell/issues/10162)）。

+   CMake 配置步骤现在也会寻找 libterminfo 作为 libtinfo 的替代名称，就像在 NetBSD curses 中使用的那样（[#9794](https://github.com/fish-shell/fish-shell/issues/9794)）。

* * *

*下载链接：为了下载 fish 的源代码，我们建议使用名为 "fish-3.7.0.tar.xz" 的文件。从 "源代码 (tar.gz)" 下载的文件将无法正确构建。此文件的 SHA-256 摘要为 `df1b7378b714f0690b285ed9e4e58afe270ac98dbc9ca5839589c1afcca33ab1`。David Adam（密钥 ID `0x7A67D962D88A709A`）的 GPG 签名可作为 "fish-3.7.0.tar.xz.asc" 获取。*
