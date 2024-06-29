<!--yml

分类：未分类

日期：2024-05-29 12:35:00

-->

# git 中的“当前分支”

> 来源：[https://jvns.ca/blog/2024/03/22/the-current-branch-in-git/](https://jvns.ca/blog/2024/03/22/the-current-branch-in-git/)

你好！我知道我刚刚写了 [一篇关于 git 中 HEAD 的博文](https://jvns.ca/blog/2024/03/08/how-head-works-in-git/)，但我对 git 中“当前分支”这个术语的含义有了更深的思考，它比我想象的要奇怪一些。

### “当前分支”的四种可能定义

1.  它位于文件**`.git/HEAD`**中。这是 [git 术语表](https://git-scm.com/docs/gitglossary#def_HEAD) 给出的定义。

1.  这是 **`git status`** 在第一行上显示的内容

1.  这是你最近用`git checkout`或`git switch`检出的内容。

1.  这是你的 shell 中的 **git 提示**。我使用的是 [fish_git_prompt](https://fishshell.com/docs/current/cmds/fish_git_prompt.html)，所以我将在此进行讨论。

我最初以为这 4 个定义大体上是一样的，但在 Mastodon 上与一些人聊过后，我意识到它们比我想象的更不同。

因此，让我们讨论一些 git 场景以及在每个场景中如何应用这些定义。我使用的是 git 版本 `2.39.2 (Apple Git-143)` 进行了所有这些实验。

### 场景 1：刚刚执行了 `git checkout main`

这里是最普通的情况：你检出了一个分支。

1.  `.git/HEAD` 包含 `ref: refs/heads/main`

1.  `git status` 显示 `当前分支为 main`

1.  我最近检出的是：`main`

1.  我的 shell 中的 git 提示显示：`(main)`

在这种情况下，这 4 个定义都是匹配的：它们都是 `main`。很简单。

### 场景 2：刚刚执行了 `git checkout 775b2b399`

现在让我们假设我检出了一个特定的提交 ID（这样我们处于“分离 HEAD 状态”）。

1.  `.git/HEAD` 包含 `775b2b399fb8b13ee3341e819f2aaa024a37fa92`

1.  `git status` 显示 `HEAD 处于 775b2b39 的分离状态`

1.  我最近检出的是 `775b2b399`

1.  我的 shell 中的 git 提示显示 `((775b2b39))`

再次强调，这些基本上都匹配了 – 有些已经截断了提交 ID，有些则没有，但就是这样。我们继续。

### 场景 3：刚刚执行了 `git checkout v1.0.13`

如果我们检出了一个标签，而不是分支或提交 ID 呢？

1.  `.git/HEAD` 包含 `ca182053c7710a286d72102f4576cf32e0dafcfb`

1.  `git status` 显示 `HEAD 处于 v1.0.13 的分离状态`

1.  我最近检出的是 `v1.0.13`

1.  我的 shell 中的 git 提示显示 `((v1.0.13))`

现在事情开始变得有点奇怪了！`.git/HEAD` 与其他 3 个指示器不一致：`git status`、git 提示和我检出的内容都是相同的 (`v1.0.13`)，但 `.git/HEAD` 包含一个提交 ID。

这是因为 git 试图帮助我们：提交 ID 有点不透明，所以如果有一个标签对应当前提交，`git status` 将显示该标签。

关于这点有一些注意事项：

+   如果我们通过其ID（`git checkout ca182053c7710a286d72`）检出提交，`git status`和我的shell提示符显示的内容完全相同 – Git实际上并不“知道”我们检出了一个标签。

+   看起来你可以通过运行`git describe HEAD --tags --exact-match`来找到与`HEAD`匹配的标签（这里是[fish git提示代码](https://github.com/fish-shell/fish-shell/blob/a5156e9e0e89bff2bd81ac945a019bad34f14346/share/functions/fish_git_prompt.fish#L521-L527)）

+   你可以看到`git-prompt.sh`在2008年的提交[27c578885](https://github.com/git/git/commit/27c578885a0b8f56430c5a24f558e2b45cf04a38)中增加了按标签描述提交的支持。

+   我不知道标签是否有注释会有什么区别。

+   如果有两个标签的提交ID相同，情况会有点奇怪。例如，如果我将标签`v1.0.12`添加到此提交中，使其同时有`v1.0.12`和`v1.0.13`，你可以在这里看到我的git提示符发生了变化，然后提示符和`git status`在显示哪个标签时存在分歧：

```
bork@grapefruit ~/w/int-exposed ((v1.0.12))> git status
HEAD detached at v1.0.13 
```

（我的提示显示`v1.0.12`，`git status`显示`v1.0.13`）

### 情景4：在rebase过程中

现在：如果我检出`main`分支，进行rebase，但在rebase过程中出现了合并冲突怎么办？情况如下：

1.  `.git/HEAD`包含`c694cf8aabe2148b2299a988406f3395c0461742`（我正在rebase到的提交的提交ID，在这种情况下是`origin/main`）

1.  `git status`显示`interactive rebase in progress; onto c694cf8`

1.  我最近检出的是`main`

1.  我的shell的git提示符显示`(main|REBASE-i 1/1)`

关于这个的一些说明：

+   我认为从某种意义上说，“当前分支”是`main` – 这是我最近检出的，这是我们在rebase完成后将返回的地方，如果我运行`git rebase --abort`，也是我们将返回的地方。

+   从另一个意义上说，我们处于分离的HEAD状态在`c694cf8aabe2`。但这并不具有通常分离HEAD状态的含义 – 如果你提交，它不会被孤立！相反，假设你完成了rebase，它将被吸收到rebase中，并放置在你的分支中间。

+   看起来在rebase过程中，旧的“当前分支”（`main`）存储在`.git/rebase-merge/head-name`中。虽然我对此并不完全确定。

### 情景5：就在`git init`后面

当我们用`git init`创建一个空仓库时会怎样呢？

1.  `.git/HEAD`包含`ref: refs/heads/main`

1.  `git status`显示`On branch main`（且“No commits yet”）

1.  我最近检出的东西是，呃，什么都没有

1.  我的shell的git提示符显示：`(main)`

所以这里的一切基本上都对得上，除了我们从未运行过`git checkout`或`git switch`。基本上，Git会自动切换到`init.defaultBranch`配置的分支。

### 情景6：一个裸Git仓库

如果我们用`git clone --bare https://github.com/rbspy/rbspy`克隆一个裸仓库会怎样呢？

1.  `HEAD`包含`ref: refs/heads/main`

1.  `git status`说`fatal: this operation must be run in a work tree`

1.  我最近检出的东西是，好吧，什么也没有，裸仓库中甚至不工作`git checkout`

1.  我的Shell的git提示是：`(BARE:main)`

所以#1和#4匹配（它们都同意当前分支是“main”），但`git status`和`git checkout`甚至都不工作。

关于这一点的一些说明：

+   我认为裸仓库中的`HEAD`主要影响一件事：这是克隆仓库时检出的分支。当你运行`git log`时也会使用它。

+   如果你真的想的话，你可以用`git symbolic-ref HEAD refs/heads/whatever`在裸仓库中将`HEAD`更新到另一个分支。不过我从来没需要这样做过，而且感觉有点奇怪，因为`git symbolic ref`不会检查你指向的`HEAD`实际上是否是一个存在的分支。不确定有没有更好的方法。

### 所有的结果

下面是一个包含所有结果的表格：

|  | `.git/HEAD` | git status | checked out | prompt |
| --- | --- | --- | --- | --- |
| 1\. `checkout main` | `ref: refs/heads/main` | `On branch main` | main | `(main)` |
| 2\. `checkout 775b2b` | `775b2b399...` | `HEAD detached at 775b2b39` | 775b2b399 | `((775b2b39))` |
| 3\. `checkout v1.0.13` | `ca182053c...` | `HEAD detached at v1.0.13` | v1.0.13 | `((v1.0.13))` |
| 4\. 在rebase内部 | `c694cf8aa...` | `interactive rebase in progress; onto c694cf8` | main | `(main\&#124;REBASE-i 1/1)` |
| 5\. 在`git init`之后 | `ref: refs/heads/main` | `On branch main` | n/a | `(main)` |
| 6\. 裸仓库 | `ref: refs/heads/main` | `fatal: this operation must be run in a work tree` | n/a | `(BARE:main)` |

### “当前分支”似乎并不完全被明确定义

当我谈论git时最初的直觉是同意git词汇表并说`HEAD`和“当前分支”意思完全相同。

但这似乎不像我之前认为的那么牢固了！一些想法：

+   `.git/HEAD`绝对是格式最一致的一个 – 它总是一个分支或者提交ID。其他的都要复杂得多

+   多亏了git的帮助，我比以前更能理解“当前分支是你上次检出的任何分支”的定义了（即使你当前正在进行二分查找或合并或其他暂时将HEAD移出该分支的操作），忽视它感觉很奇怪。

+   `git status`提供了很多有用的上下文信息 – 这5条状态信息不仅仅是`HEAD`当前设置的内容。

    1.  `on branch main`

    1.  `HEAD detached at 775b2b39`

    1.  `HEAD detached at v1.0.13`

    1.  `interactive rebase in progress; onto c694cf8`

    1.  `on branch main, no commits yet`

### 更多关于“当前分支”的定义

我打算尝试收集一些我从Mastodon上听到的有关“当前分支”术语的其他定义，并写一些注释。

1.  “如果我提交了一次，将会更新的分支”

    +   大多数情况下，这与`.git/HEAD`相同

    +   有人可能会争论，如果你在rebase过程中，它和`HEAD`是不同的，因为最终那个新提交会出现在`.git/rebase-merge/head-name`分支上。

1.  “大多数git操作作用的分支”

    +   这与`.git/HEAD`中的情况类似，不过某些操作（如`git status`）在某些情况下会有不同的行为，比如在裸仓库中，`git status`不会告诉你当前的分支。

### 关于孤立提交

我注意到的一件事是，所有这些都没有捕捉到当前提交是否**孤立**的信息——`git status`消息（`HEAD detached from c694cf8`）无论你当前的提交是否孤立，都是一样的。

我想这是因为在大型仓库中确定给定提交是否是孤立的可能需要很长时间：你可以通过`git branch --contains HEAD`找出当前提交是否是孤立的，这个命令在有70,000个提交的仓库中大约需要500ms。

当你切换到不同的分支时，Git会警告你提交是否是孤立的（“警告：你正在留下1个提交，不连接到任何你的分支…”）。

### 就是这样！

对于所有这些，我没有特别聪明的话要说。我越想git，就越能理解人们为什么感到困惑。
