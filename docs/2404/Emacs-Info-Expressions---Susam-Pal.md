<!--yml

category: 未分类

date: 2024-05-27 13:13:25

-->

# Emacs Info 表达式 - Susam Pal

> 来源：[https://susam.net/emacs-info-expressions.html](https://susam.net/emacs-info-expressions.html)

<main>

# Emacs Info 表达式

由**Susam Pal**在 2024 年 4 月 12 日发布

在 `#emacs` IRC 或 Matrix 频道中，我们经常通过 Elisp 表达式分享内置的 Emacs 文档，看起来像这样：

```
(info "(emacs) Basic Undo")
```

这是另一个例子：

```
(info "(emacs) Word Search")
```

尽管 Emacs 手册在全球网络上也是可以获得的，但在 Emacs 频道中这是一种常见的做法。例如，上述表达式引用的章节在这里可以找到：[GNU Emacs 手册：单词搜索](https://www.gnu.org/software/emacs/manual/html_node/emacs/Word-Search.html)。分享 Elisp 表达式的原因很可能部分是传统，部分是便利。许多 Emacs 用户通过 Emacs 自身登录到 IRC 网络，因此一旦接收者在其聊天缓冲区中看到类似上述的 Elisp 表达式，访问相应的手册页面只是在光标移到右括号之后，输入 `C-x C-e` 的简单操作。

但是对发送者来说，仅仅为了与他人分享手册的某个部分而输入 Elisp 表达式不是很笨拙吗？事实证明并非如此。这是 Emacs！所以当然有快捷键可以做到这一点。

## 复制 Info 节点名称

比如，当我们帮助另一个 Emacs 用户时，我们键入 `M-x info-apropos RET version control RET`，并跳转到章节“Branches”，意识到这是我们试图帮助的人应该阅读的章节。现在当我们位于此章节时，我们只需键入 `c`，Emacs 将复制当前 Info 节点的名称到剪贴板。这个名称看起来像这样：

```
(emacs) Branches
```

现在我们可以进入 `*scratch*` 缓冲区（或任何缓冲区），复制节点名称，并手动完成 `info` 表达式。例如，我们可以在新的一行上键入以下键序列来创建 Elisp 表达式并复制到剪贴板：

```
" " C-b C-y C-a C-SPC C-e M-( info C-a C-k C-/
```

在原始的 Emacs 中，上述相对较长的键序列首先会连续输入两个双引号 (`" "`)，然后将光标移回双引号内部 (`C-b`)，接着粘贴来自剪贴板的文本 `(emacs) Branches` (`C-y`)，然后选择粘贴的文本 (`C-a C-SPC C-e`)，接着在括号内部环绕它 (`M-(`)，然后在开括号后插入文本 `info`，最后复制生成的表达式到剪贴板 (`C-a C-k C-/`)。复制到剪贴板的表达式看起来像这样：

```
(info "(emacs) Branches")
```

我们能避免手动构建 `info` 表达式并让 Emacs 为我们做吗？事实证明我们可以，我们将在下一节中看到。

## 复制 Info 表达式

最近我从[Karthink](https://karthinks.com/)和[Mekeor Melire](https://mastodon.social/@mekeor@catgirl.cloud)那里学到，我们可以请求Emacs自动为我们创建整个`info`表达式。我们所需做的就是在`c`键前使用零前缀参数。所以当我们在“Branches”章节时，如果输入`C-0 c`，则以下表达式将复制到删除环中：

```
(info "(emacs) Branches")
```

我本应该知道这一点，因为确实当我们在Info文档浏览器中时，如果键入`C-h k c`来描述键序列`c`，我们会看到以下文档：

```
c runs the command Info-copy-current-node-name (found in
Info-mode-map), which is an interactive byte-compiled Lisp function in
‘info.el’.

It is bound to c, w, <menu-bar> <info> <Copy Node Name>.

(Info-copy-current-node-name &optional ARG)

Put the name of the current Info node into the kill ring.
The name of the Info file is prepended to the node name in parentheses.
With a zero prefix arg, put the name inside a function call to ‘info’.

  Probably introduced at or before Emacs version 22.1.

[back]

```

确实，Emacs确实有一个方便的按键序列来为当前Info节点创建完整的`info`表达式。接收此`info`表达式的人只需评估它即可简单地访问手册的相应部分。例如，在Emacs中复制表达式后，他们可以简单地输入`C-y C-x C-e`将表达式粘贴到缓冲区并立即评估它。或者，他们可能想要输入`M-: C-y RET`来打开`eval-expression`迷你缓冲区，粘贴表达式并进行评估。

</main>
