<!--yml

category: 未分类

日期：2024-05-27 14:54:46

-->

# 流行的 git 配置选项

> 来源：[https://jvns.ca/blog/2024/02/16/popular-git-config-options/](https://jvns.ca/blog/2024/02/16/popular-git-config-options/)

哈喽！我一直希望命令行工具能够提供它们各种选项的流行程度数据，比如：

+   “基本上没人使用这个”

+   “80% 的人使用这个选项，可能值得一看”

+   “这个选项有6种可能的值，但实际上只有这两个被大多数人使用”

因此，我在 Mastodon 上[询问了人们最喜欢的 git 配置选项](https://social.jvns.ca/@b0rk/111885363143321068)：

> 你们最喜欢设置的 git 配置选项是什么？目前，我只在`~/.gitconfig`中设置了`git config push.autosetupremote true`和`git config init.defaultBranch main`，很好奇其他人都设置了哪些选项。

像往常一样，我收到了很多很好的答案，并了解到了一些非常流行的 git 配置选项，这些选项我之前从未听说过。

我将列出这些选项，大致按照流行程度排序。以下是目录：

所有的选项都在`man git-config`中有文档，或者可以查看[这个页面](https://git-scm.com/docs/git-config)。

### `pull.ff only`或`pull.rebase true`

这两个选项最受欢迎。它们都有相似的目标：在运行`git pull`时避免在上游分支分歧时意外创建合并提交。

+   `pull.rebase true`相当于每次`git pull`时运行`git pull --rebase`

+   `pull.ff only`相当于每次`git pull`时运行`git pull --ff-only`

我很确定同时设置它们不合理，因为`--ff-only`会覆盖`--rebase`。

个人而言，我不使用这两个选项，因为我更喜欢每次都决定如何处理这种情况，现在 git 的默认行为是当你的分支与上游分支分歧时，只是抛出错误并询问你要做什么（与`git pull --ff-only`非常相似）。

### `merge.conflictstyle zdiff3`

接下来：使合并冲突更易读！`merge.conflictstyle zdiff3`和`merge.conflictstyle diff3`都非常流行（“非常重要”）。

主要思想是，共识似乎是“diff3 很棒，而 zdiff3（较新）更棒！”。

那么`diff3`是怎么回事呢？嗯，在 git 中，默认情况下合并冲突看起来是这样的：

```
<<<<<<< HEAD
def parse(input):
    return input.split("\n")
=======
def parse(text):
    return text.split("\n\n")
>>>>>>> somebranch 
```

我应该决定`input.split("\n")`还是`text.split("\n\n")`更好。但是怎么办？如果我记不清`\n`或`\n\n`哪个正确怎么办？这时就要用到 diff3 了！

使用`merge.conflictstyle diff3`设置时，合并冲突看起来是这样的：

```
<<<<<<< HEAD
def parse(input):
    return input.split("\n")
||||||| b9447fc
def parse(input):
    return input.split("\n\n")
=======
def parse(text):
    return text.split("\n\n")
>>>>>>> somebranch 
```

这里有**额外的信息**：现在原始代码的版本在中间！所以我们可以看到：

+   一方面将`\n\n`改为`\n`

+   另一方面将`input`重命名为`text`

因此，正确的合并冲突解决方法应该是`return text.split("\n")`，因为这样结合了双方的更改。

我没用过zdiff3，但很多人似乎认为它更好。这篇博文[使用 zdiff3 改进 Git 冲突](https://ductile.systems/zdiff3/)详细讲解了它。

### `rebase.autosquash true`

对于我来说，Autosquash也是一个新功能。它的目标是更容易地修改旧的提交。

它的工作原理如下：

+   你有一个提交，想要与之前3次提交中的某个提交（比如`add parsing code`）合并。

+   你可以用`git commit --fixup OLD_COMMIT_ID`提交它，这样新的提交会得到`fixup! add parsing code`的提交消息。

+   现在，当你运行`git rebase --autosquash main`时，它将自动将所有`fixup!`提交与它们的目标合并。

`rebase.autosquash true`意味着`--autosquash`将始终自动传递给`git rebase`。

### `rebase.autostash true`

这会在 git rebase 前自动运行`git stash`，在后面自动运行`git stash pop`。它基本上给`git rebase`传递了`--autostash`参数。

就我个人而言，我有点担心这一点，因为它可能导致重播后出现合并冲突，但我猜对于大多数人来说，这种情况并不经常发生，因为这似乎是一个非常受欢迎的配置选项。

### `push.default simple`，`push.default current`，`push.autoSetupRemote true`

这些[`push`](https://git-scm.com/docs/git-config#Documentation/git-config.txt-pushdefault)选项告诉`git push`自动将当前分支推送到同名的远程分支。

+   `push.default simple`是 Git 的默认设置。只有当你的分支已经跟踪一个远程分支时才有效。

+   `push.default current`类似，但它将始终将本地分支推送到同名的远程分支。

+   `push.autoSetupRemote true`有点不同 - 当你第一次推送分支时，它会自动设置跟踪。

我认为我更喜欢`push.autoSetupRemote true`而不是`push.default current`，因为`push.autoSetupRemote true`还允许你从匹配的远程分支**拉取**（尽管你需要先推送才能设置跟踪）。`push.default current`只允许你推送。

我认为要小心的唯一事情是，当`push.autoSetupRemote true`和`push.default current`时，你需要确信你永远不会意外地创建一个与无关的远程分支同名的本地分支。很多人有分支命名约定（比如`julia/my-change`），使这种冲突几乎不可能发生，或者只有少数协作者，分支名称冲突可能性很低。

### `init.defaultBranch main`

创建新的仓库时，请使用`main`分支而不是`master`分支。

### `commit.verbose true`

这在你写提交消息时，会将整个提交差异添加到文本编辑器中，以帮助你记住你在做什么。

### `rerere.enabled true`

这使得 [rerere](https://git-scm.com/book/en/v2/Git-Tools-Rerere) （“**re**use **re**covered **re**solution”）生效，它会记住你在 `git rebase` 过程中如何解决合并冲突，并在可能的情况下自动解决冲突。

### `help.autocorrect 10`

默认情况下，git 的自动更正会检查拼写错误（例如 `git ocmmit`），但不会真正运行更正后的命令。

如果你希望它自动运行建议，可以将 [`help.autocorrect`](https://git-scm.com/docs/git-config#Documentation/git-config.txt-helpautoCorrect) 设置为 `1`（0.1 秒后运行）、`10`（1 秒后运行）、`immediate`（立即运行）或 `prompt`（提示后运行）

“pager” 是 git 用来显示 `git diff`、`git log`、`git show` 等输出的工具。人们将其设置为：

+   [`delta`](https://github.com/dandavison/delta)（一个带语法高亮的高级 diff 查看工具）

+   `less -x5,9`（设置制表位，我猜如果你有很多带制表符的文件会有帮助？）

+   `less -F -X`（对这个不太确定，`-F` 似乎在内容可以在一个屏幕上显示时禁用分页，但我的 git 似乎已经默认这样做了）

+   `cat`（完全禁用分页）

我以前使用 `delta`，但后来关闭了它，因为我在终端中搞乱了颜色方案，然后搞不清楚如何修复。不过我觉得它是一个很棒的工具。

我相信 delta 还建议您设置 `interactive.diffFilter delta --color-only`，以在运行 `git add -p` 时为代码语法高亮。

### `diff.algorithm histogram`

Git 的默认 diff 算法经常对函数重新排序处理得很糟糕。例如看看这个 diff：

```
-.header {
+.footer {
     margin: 0;
 }

-.footer {
+.header {
     margin: 0;
+    color: green;
 } 
```

我觉得这相当令人困惑。但使用 `diff.algorithm histogram`，diff 看起来像这样，我觉得更清晰：

```
-.header {
-    margin: 0;
-}
-
 .footer {
     margin: 0;
 }

+.header {
+    margin: 0;
+    color: green;
+} 
```

有些人也使用 `patience`，但 `histogram` 似乎更受欢迎。[何时使用每种 Git Diff 算法](https://luppeng.wordpress.com/2020/10/10/when-to-use-each-of-the-git-diff-algorithms/) 有更多相关内容。

### `core.excludesfile`: 全局的 .gitignore

`core.excludeFiles = ~/.gitignore` 可以设置一个全局的 gitignore 文件，适用于所有的仓库，比如 `.idea` 或者 `.DS_Store` 这类你永远不想提交到任何仓库的文件。它的默认位置是 `~/.config/git/ignore`。

### `includeIf`: 个人和工作分开的 git 配置

很多人说他们用这个来为个人和工作仓库配置不同的电子邮件地址。你可以设置成这样：

```
[includeIf "gitdir:~/code/<work>/"]
path = "~/code/<work>/.gitconfig" 
```

我经常会不小心克隆仓库的 HTTP 版本而不是 SSH 版本，然后必须手动进入 `~/.git/config` 并编辑远程 URL。这看起来像一个不错的解决方案：它会将远程中的 `https://github.com` 替换为 `git@github.com:`。

这是在 `~/.gitconfig` 中的样子，因为它有点啰嗦：

```
[url "git@github.com:"]
	insteadOf = "https://github.com/" 
```

有人说他们使用 `pushInsteadOf` 替代，只为 `git push` 执行替换，因为他们不想在拉取公共仓库时解锁他们的 SSH 密钥。

还有几个人提到设置`insteadOf = "gh:"`，这样他们可以`git remote add gh:jvns/mysite`来添加一个少打字的远程仓库。

### `fsckobjects`：避免数据损坏

几个人提到了这一点。有人解释说“及早检测数据损坏。很少有影响，但有几次拯救了我们整个团队”。

```
transfer.fsckobjects = true
fetch.fsckobjects = true
receive.fsckObjects = true 
```

### 子模块相关

我从来没有理解过子模块的任何东西，但有几个人说他们喜欢设置：

+   `status.submoduleSummary true`

+   `diff.submodule log`

+   `submodule.recurse true`

我不打算解释这些，但[Mastodon上@unlambda的解释在这里](https://hachyderm.io/@unlambda/111942468084436716#)

### 和更多内容

这是其他至少有两人建议的所有内容：

+   `blame.ignoreRevsFile .git-blame-ignore-revs`允许您指定在`git blame`期间忽略的提交文件，以免巨大的重命名破坏您的责备

+   `branch.sort -committerdate`，使`git branch`按最近使用的分支排序而不是按字母顺序，以便更容易找到分支。`tag.sort taggerdate`对标签类似。

+   `color.ui false`：关闭颜色显示

+   `commit.cleanup scissors`: 可以在提交消息中写入`#include`，而不会将`#`视为注释并删除

+   `core.autocrlf false`：在Windows上与使用Unix的人员良好兼容

+   `core.editor emacs`：使用emacs（或其他编辑器）编辑提交消息

+   `credential.helper osxkeychain`：使用Mac钥匙链进行管理

+   `diff.tool difftastic`：使用[difftastic](https://difftastic.wilfred.me.uk/)（或`meld`或`nvimdiffs`）显示差异

+   `diff.colorMoved default`：使用不同的颜色突出显示差异中已“移动”的行

+   `diff.colorMovedWS allow-indentation-change`：在设置了`diff.colorMoved`的情况下，还会忽略缩进更改

+   `diff.context 10`：在差异中包含更多上下文

+   `fetch.prune true`和`fetch.prunetags` - 自动删除已删除的远程跟踪分支

+   `gpg.format ssh`：允许您使用SSH密钥签署提交

+   `log.date iso`：将日期显示为`2023-05-25 13:54:51`而不是`Thu May 25 13:54:51 2023`

+   `merge.keepbackup false`，以消除Git在合并冲突期间创建的`.orig`文件

+   `merge.tool meld`（或`nvim`或`nvimdiff`），以便您可以使用`git mergetool`来帮助解决合并冲突

+   `push.followtags true`：与推送的提交一起推送新标签

+   `rebase.missingCommitsCheck error`：在变基期间不允许删除提交

+   `rebase.updateRefs true`：可以轻松地一次性变基多个堆叠的分支。[这篇博文介绍了它](https://andrewlock.net/working-with-stacked-branches-in-git-is-easier-with-update-refs/)

### 如何设置这些选项

我通常使用`git config --global NAME VALUE`来设置git配置选项，例如`git config --global diff.algorithm histogram`。我通常将所有选项设置为全局，因为在不同的存储库中具有不同的git行为会让我感到很紧张。

如果我想删除一个选项，我会手动编辑 `~/.gitconfig`，在那里它们看起来像这样：

```
[diff]
	algorithm = histogram 
```

### 在撰写此文章后我做出的配置更改

我的git配置非常简单，我已经有了：

+   `init.defaultBranch main`

+   `push.autoSetupRemote true`

+   `merge.tool meld`

+   `diff.colorMoved default`（由于某些原因实际上对我不起作用，但我还没有找到时间调试）

我在写完这篇博文后还添加了这3个：

+   `diff.algorithm histogram`

+   `branch.sort -committerdate`

+   `merge.conflictstyle zdiff3`

如果现在我的生活中精心制作的拉取请求与多个提交有更大关联，我可能还会设置 `rebase.autosquash`。

我学会了对设置新的配置选项要谨慎——习惯新行为需要很长时间，如果我一次改变太多东西，我会感到困惑。`branch.sort -committerdate` 是我已经在使用的（通过别名），我相当确信 `diff.algorithm histogram` 将使我在重新排序函数时更容易阅读差异。

### 就这些！

我总是惊讶于询问许多人他们喜欢什么东西，然后列出最常提到的那些，就像我几年前整理的这个 [新命令行工具列表](https://jvns.ca/blog/2022/04/12/a-list-of-new-ish--command-line-tools/) 一样。拥有20或30个选项列表比仔细查阅 [所有大约600个git配置选项](https://jvns.ca/data/all-git-options.txt) 要高效得多。

总结这些内容有点令人困惑，因为git的默认选项确实在多年来发生了很大变化，所以人们偶尔会设置8年前很重要但今天已经是默认的选项。还有一些实验性的选项已经被移除，并用不同的版本替换了。

我尽力在2024年准确解释git的工作方式，但我肯定在某处犯了错误，特别是因为我自己并不使用大部分这些选项。如果你看到错误，请在Mastodon上告诉我，我会尽力修复。

我可能稍后会问人们关于别名，因为有一些很棒的别名我没有提到，因为这已经很长了。
