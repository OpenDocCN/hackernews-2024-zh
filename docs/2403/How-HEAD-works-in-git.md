<!--yml

category: 未分类

date: 2024-05-27 14:47:17

-->

# `HEAD`在git中的工作原理

> 来源：[https://jvns.ca/blog/2024/03/08/how-head-works-in-git/](https://jvns.ca/blog/2024/03/08/how-head-works-in-git/)

你好！前几天我在Mastodon上发布了一项调查，询问人们对于理解Git中`HEAD`的工作方式有多自信。结果（共1700票）对我来说有些令人惊讶：

+   10% “100%”

+   36% “相当自信”

+   39% “有点自信？”

+   15% “完全没有概念”

我对人们对其理解如此缺乏信心感到惊讶 - 我一直认为`HEAD`是一个相当简单的主题。

通常当人们说某个主题令人困惑时，而我认为并不是时，原因是实际上存在一些隐藏的复杂性，我并未考虑到。经过一些后续对话，我发现`HEAD`实际上比我所认为的更复杂一些！

这里是快速目录：

### HEAD实际上是几个不同的东西

谈到`HEAD`时，我意识到它实际上有几个紧密相关的含义：

1.  文件`.git/HEAD`

1.  `HEAD`，如在`git show HEAD`中所示（git称其为“修订参数”）

1.  在各种命令的输出中，git使用`HEAD`的所有方式（`<<<<<<<<<<HEAD`，`(HEAD -> main)`，`分离HEAD状态`，`在分支main上`等）

这些内容彼此之间非常密切相关，但我认为对于刚接触git的人来说，这种关系可能并不明显。

### 文件`.git/HEAD`

Git有一个非常重要的文件叫做`.git/HEAD`。这个文件的工作方式是它包含以下内容之一：

1.  **分支**的名称（例如`ref: refs/heads/main`）

1.  **提交ID**（例如`96fa6899ea34697257e84865fefc56beb42d6390`）

这个文件决定了在Git中你的“当前分支”是什么。例如，当你运行`git status`时，会看到这个：

```
$ git status
On branch main 
```

这意味着文件`.git/HEAD`包含`ref: refs/heads/main`。

如果`.git/HEAD`包含提交ID而不是分支，git称之为“分离HEAD状态”。我们稍后会讨论这个问题。

（有时人们会说HEAD包含**引用**或提交ID的名称，但我相当确定引用必须是**分支**。您*可以*在手动编辑`.git/HEAD`时将其包含为不是分支的引用的名称，但我不认为您可以通过常规git命令来执行此操作。如果有一种常规git命令的方法可以使.git/HEAD成为非分支引用，我很想知道，如果确实如此，您可能为什么要这样做！）

### `HEAD`，如在`git show HEAD`中所示

在git命令中，非常常见地使用`HEAD`来引用提交ID，例如：

+   `git diff HEAD`

+   `git rebase -i HEAD^^^^`

+   `git diff main..HEAD`

+   `git reset --hard HEAD@{2}`

所有这些（`HEAD`，`HEAD^^^`，`HEAD@{2}`）被称为“修订参数”。它们在[man gitrevisions](https://git-scm.com/docs/gitrevisions)中有文档记录，并且Git将尝试将它们解析为提交ID。

（老实说，我以前从未听说过“修订参数”这个术语，但这是可以让你查阅此概念文档的术语）

`git show HEAD` 中的 HEAD 具有一个非常简单的含义：它解析为你当前检出的**当前提交**！ Git 通过以下两种方式之一解析 `HEAD`：

1.  如果 `.git/HEAD` 包含一个分支名称，它将是该分支上的最新提交（例如从 `.git/refs/heads/main` 中读取）

1.  如果 `.git/HEAD` 包含一个提交 ID，那么它将是该提交 ID

### 接下来：所有的输出格式

现在我们已经讨论了文件 `.git/HEAD` 和“修订参数” `HEAD`，比如在 `git show HEAD` 中。我们留下了 git 在输出中使用 `HEAD` 的各种方式。

### `git status`: “在分支 main 上”或“HEAD 分离”

当你运行 `git status` 时，第一行将总是看起来像以下两种之一：

1.  `在分支 main 上`。这意味着 `.git/HEAD` 包含一个分支。

1.  `HEAD 分离于 90c81c72`。这意味着 `.git/HEAD` 包含一个提交 ID。

我之前承诺过要解释什么是“HEAD 分离”，现在让我们来做吧。

### 分离 HEAD 状态

“HEAD 分离”或“分离 HEAD 状态”意味着你没有当前分支。

没有当前分支有点危险，因为如果你创建新的提交，这些提交将不会附加到任何分支 - 它们将是孤立的！ 孤立的提交有两个问题：

1.  这些提交更难找到（你不能运行 `git log somebranch` 来找到它们）

1.  孤立的提交最终将被 git 的垃圾收集删除

就个人而言，我非常注意避免在分离 HEAD 状态下创建提交，尽管有些人[更喜欢以这种方式工作](https://github.com/arxanas/git-branchless)。不过，退出分离 HEAD 状态非常简单，你可以：

1.  返回到分支（`git checkout main`）

1.  在该提交处创建一个新的分支（`git checkout -b newbranch`）

1.  如果你因为在重新基础的过程中而处于分离 HEAD 状态，请完成或中止重新基础（`git rebase --abort`）

好的，回到其他具有 `HEAD` 的 git 命令！

### `git log`: `(HEAD -> main)`

当你运行 `git log` 并查看第一行时，可能会看到以下三种情况之一：

1.  `commit 96fa6899ea (HEAD -> main)`

1.  `commit 96fa6899ea (HEAD, main)`

1.  `commit 96fa6899ea (HEAD)`

如何解释这些并不完全明显，所以这里的解释如下：

+   在 `(...)` 中，git 列出了指向该提交的每个引用，例如 `(HEAD -> main, origin/main, origin/HEAD)` 意味着 `HEAD`、`main`、`origin/main` 和 `origin/HEAD` 都指向该提交（直接或间接）。

+   `HEAD -> main` 意味着你当前的分支是 `main`

+   如果该行显示为 `HEAD,` 而不是 `HEAD ->`，这意味着你处于分离 HEAD 状态（没有当前分支）

如果我们使用这些规则来解释上述三个示例：结果如下：

1.  `commit 96fa6899ea (HEAD -> main)` 意味着：

    +   `.git/HEAD` 包含 `ref: refs/heads/main`

    +   `.git/refs/heads/main` 包含 `96fa6899ea`

1.  `commit 96fa6899ea (HEAD, main)`的意思是：

    +   `.git/HEAD`包含`96fa6899ea`（HEAD处于“分离”状态）

    +   `.git/refs/heads/main`还包含`96fa6899ea`

1.  `commit 96fa6899ea (HEAD)`的意思是：

    +   `.git/HEAD`包含`96fa6899ea`（HEAD处于“分离”状态）

    +   `.git/refs/heads/main`要么包含不同的提交 ID，要么不存在

### 合并冲突：`<<<<<<< HEAD`只是令人困惑

当您解决合并冲突时，您可能会看到类似这样的内容：

```
<<<<<<< HEAD
def parse(input):
    return input.split("\n")
=======
def parse(text):
    return text.split("\n\n")
>>>>>>> somebranch 
```

我发现在这种情况下的`HEAD`极其令人困惑，基本上我只是忽略它。原因如下。

+   在执行**合并**操作时，冲突中的`HEAD`与运行`git merge`时的`HEAD`相同。简单来说。

+   在执行**变基**操作时，冲突中的`HEAD`则完全不同：它是您要变基的**其他提交**。因此，它与运行`git rebase`时的`HEAD`完全不同。这是因为变基首先检出其他提交，然后重复地在其上挑选提交。

类似地，“ours”和“theirs”在合并和变基中是颠倒的。

在我进行rebase或merge时，`HEAD`的含义变化太令我困惑了，我觉得完全忽略`HEAD`并使用另一种方法来弄清代码的哪一部分更简单。

### 一些关于一致术语的想法

我认为如果 Git 在`HEAD`周围的术语稍微更内部一致，那么`HEAD`会更直观。

例如，Git 谈论“分离的 HEAD 状态”，但从不谈论“附加的 HEAD 状态” - Git 的文档从不使用“附加”一词来指代`HEAD`。Git 谈论“在”一个分支上，但从不谈论“不在”一个分支上。

因此很难猜测“在分支main上”实际上与“HEAD分离”相反。用户如何猜测“HEAD分离”与分支有任何关系，或者“在分支main上”与`HEAD`有任何关系？

### 就这些！

如果我能想到 `HEAD` 在 Git 中的其他用法（特别是在 Git 的输出中出现的方式），我可能会在以后的帖子中补充它们。

如果你觉得HEAD很困惑，希望这有点帮助！
