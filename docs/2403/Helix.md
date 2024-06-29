<!--yml

分类：未分类

日期：2024-05-29 12:48:04

-->

# Helix

> 来源：[https://helix-editor.com/news/release-24-03-highlights/](https://helix-editor.com/news/release-24-03-highlights/)

# 发布 24.03 版本的亮点

2024 年 3 月 30 日

Helix 24.03 版本来啦！首先，非常感谢所有参与此次发布的 125 名贡献者。

对 Helix 还不熟悉？Helix 是一个带有内置多选功能、语言服务器协议（LSP）、tree-sitter 支持以及实验性调试适配器协议（DAP）的模态文本编辑器。

让我们看看这个版本的亮点。

## 类似于 Amp 的跳转

在（Neo）Vim 插件空间中，跳转功能非常流行，甚至有插件将相同功能添加到其他工具（如浏览器）中。它们允许您通过输入 "标签" 的方式，有效地在视图的大部分区域中移动选择，就像您通过鼠标点击一样。24.03 版本引入了受 [Amp 编辑器](https://github.com/jmacdonald/amp) 的 [跳转模式](https://amp.rs/docs/usage/#jump-mode) 启发的跳转命令。按下 `gw` 添加跳转标签，然后输入一个标签以跳转到该标签下的单词。在选择模式 (`v`) 中使用 `gw` 可扩展选择。

在过去，Helix 只能切换行注释，如 `//` 和 `#`，而像 OCaml 这样的语言则使用了像 `(*` 这样的 "行" 注释令牌的变通方法。在 24.03 版本中，Helix 现在还可以切换块注释。使用 `C-c` 或 `<space>c`，根据语言的注释令牌配置智能地在当前选择周围添加或删除行或块注释，使用 `<space>c` 在当前选择周围切换块注释，或使用 `<space><A-c>` 在当前行上仅切换行注释。

## tree-sitter 注入的改进

Helix 使用 [`tree-sitter`](https://github.com/tree-sitter/tree-sitter) 增量解析进行语法高亮显示、文本对象、缩进以及一些动作和命令。24.03 版本改进了我们处理 *[injections](https://tree-sitter.github.io/tree-sitter/syntax-highlighting#language-injection)* 的方式——这是一种强大的 tree-sitter 功能，用于解析包含多种语言的文档。例如，在 HTML 的 `<script>` 或 `<style>` 标签中可能包含 JavaScript、CSS 或其他语言。HTML 解析器无需了解如何解析所有这些语言，而是可以 *注入* JavaScript 或 CSS 解析器来处理 `<script>` 或 `<style>` 标签的内容。

其中一项改进是 `:tree-sitter-subtree` 命令，它显示光标下的语法树的 S 表达式。24.03 版本显示选择区域下的注入层，而不仅限于根层。例如，在这个 asciicast 中，我们现在展示文档中 JavaScript 部分的语法树，而过去我们只展示 HTML 部分的语法树。

另一个重大改进是对 tree-sitter 动作的改进。`A-o` (alt + o) 将选择扩展到当前选择所覆盖的语法树节点的父节点。`A-i` 缩小到子节点，`A-n` 和 `A-p` 分别移动到下一个和上一个节点。以前这些命令只在根层（例如 asciicast 中的 HTML）上工作，但现在它们可以找到包含选择的注入层，并沿着该层的语法树移动。在内部，我们将注入层组织成类似树的结构，以便这些动作在需要时可以切换层。

注意未来的改进，比如在注入内容中查找文本对象。

## 内部改进

24.03 版本还看到了重大的内部改进。首先是来自新的“事件系统” - 这是围绕 Tokio 通道和任务构建的系统，允许 Helix 代码库的不同部分进行通信。事件系统还增加了通用的去抖动和在后台线程中运行任务的方式，因此我们可以避免 UI 的锁定。代码库的部分已经迁移到事件系统，如 LSP 完成和签名帮助。在 24.03 版本中，你会注意到，例如使用箭头键在插入模式下导航时，LSP 完成现在不会自动弹出。相反，它现在是钩子到文档更改事件中，因此它可以在你开始更改文档时智能地弹出，而不是每次编辑器空闲时。

另一个令人兴奋的主要改进是将 `regex` crate 替换为 [`regex-cursor`](https://github.com/pascalkuthe/regex-cursor)，这是与 [`ropey`](https://github.com/cessen/ropey) 兼容的流式 regex 实现。`regex-cursor` 能够在不连续的字符串上运行 - 这些输入可能不在内存中相邻。在 rope 中，文档的不同部分可能在内部分开存储。为了使用 `regex` API，我们需要将文档片段转换为常规的 Rust `String`，从而增加了该文档片段的内存占用。`regex-cursor` 相反操作于 rope 的块上，这极大地提高了效率。

## 总结

正如往常一样，这只是 24.03 版本发布的亮点。详情请查看完整的[更新日志](https://github.com/helix-editor/helix/blob/master/CHANGELOG.md#2404-2024-03-30)。

欢迎在 [Matrix 空间](https://matrix.to/#/#helix-community:matrix.org)讨论使用和开发问题，并跟随 Helix 的发展在 [GitHub 仓库](https://github.com/helix-editor/helix/)中。
