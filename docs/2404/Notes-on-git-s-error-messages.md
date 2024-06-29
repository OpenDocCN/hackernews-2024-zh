<!--yml

类别：未分类

日期：2024-05-27 13:08:40

-->

# Git 的错误消息注记

> 来源：[https://jvns.ca/blog/2024/04/10/notes-on-git-error-messages/](https://jvns.ca/blog/2024/04/10/notes-on-git-error-messages/)

在写关于 Git 的文章时，我注意到很多人都在 struggle with Git 的错误消息。我花了很多年来习惯这些错误消息，所以我花了很长时间才理解为什么人们会感到困惑，但在更深思考之后，我意识到：

1.  有时我实际上*确实*被错误消息弄糊涂了，我只是习惯了被困惑。

1.  当错误消息不太具有信息性时，我有一堆获取更多信息的策略。

因此，在本文中，我将详细介绍一些 Git 的错误消息，列出我认为每个消息令人困惑的一些事情，并谈谈我在消息令我困惑时的做法。

### 改进错误消息并不容易。

在我们开始之前，我想说尝试思考为什么这些错误消息令人困惑让我对 Git 的维护有了很多尊重。我已经思考了几个月的 Git，对于一些消息，我真的不知道如何改进它们。

有些我觉得改进错误消息很难的地方：

+   如果你想到一个新消息的想法，很难说它是否真的更好！

+   改进错误消息之类的工作通常[没有资金支持](https://lwn.net/Articles/959768/)。

+   这些错误消息必须翻译（git 的错误消息已翻译成[19种语言](https://github.com/git/git/tree/master/po)！）

话虽如此，如果你觉得这些消息令人困惑，希望这些注释能有助于稍微澄清一些。

```
$ git push
To github.com:jvns/int-exposed
! [rejected]        main -> main (non-fast-forward)
error: failed to push some refs to 'github.com:jvns/int-exposed'
hint: Updates were rejected because the tip of your current branch is behind
hint: its remote counterpart. Integrate the remote changes (e.g.
hint: 'git pull ...') before pushing again.
hint: See the 'Note about fast-forwards' in 'git push --help' for details.

$ git status
On branch main
Your branch and 'origin/main' have diverged,
and have 2 and 1 different commits each, respectively.

```

有些我觉得这里令我困惑的地方：

1.  无论分支只是**落后**还是分支已经**分叉**，你都会收到完全相同的错误消息。从这条消息中无法判断是哪种情况：你需要运行 `git status` 或 `git pull` 以查找答案。

1.  它说`failed to push some refs`，但不是完全清楚*哪些*引用失败推送了。我相信所有未能推送的内容都在前一行列出了 `! [rejected]` – 在这种情况下只是 `main` 分支。

**如果我感到困惑时我喜欢做的事情：**

+   我会运行 `git status` 以了解当前分支的状态。

+   我认为我几乎从不试图一次推送多个分支，所以我通常完全忽略 git 提到的具体失败推送的分支 – 我只假设这是我的当前分支。

```
$ git pull
hint: You have divergent branches and need to specify how to reconcile them.
hint: You can do so by running one of the following commands sometime before
hint: your next pull:
hint:
hint:   git config pull.rebase false  # merge
hint:   git config pull.rebase true   # rebase
hint:   git config pull.ff only       # fast-forward only
hint:
hint: You can replace "git config" with "git config --global" to set a default
hint: preference for all repositories. You can also pass --rebase, --no-rebase,
hint: or --ff-only on the command line to override the configured default per
hint: invocation.
fatal: Need to specify how to reconcile divergent branches.

```

我认为这里最令人困惑的是，git 向你展示了一种类似于压倒性的选择：它说你可以选择：

1.  在本地配置 `pull.rebase false`、`pull.rebase true` 或 `pull.ff only`。

1.  或者在全局配置它们。

1.  或运行 `git pull --rebase` 或 `git pull --no-rebase`。

很难想象一个 Git 初学者如何能轻松使用这个提示来自己整理所有这些选项。

如果我向朋友解释这个问题，我会说“你可以使用 `git pull --rebase` 或 `git pull --no-rebase` 立即使用重排或合并来解决这个问题，如果你想设置一个永久的偏好，你可以使用 `git config pull.rebase false` 或 `git config pull.rebase true`。

`git config pull.ff only` 对我来说有点多余，因为那本来就是 git 的默认行为（尽管并非总是如此）。

**我在这里喜欢做的事情：**

+   运行 `git status` 查看当前分支的状态

+   可能要运行 `git log origin/main` 或 `git log` 查看分歧的提交是什么

+   通常运行 `git pull --rebase` 来解决它

+   有时我会运行 `git push --force` 或 `git reset --hard origin/main` 如果我想放弃我的本地工作或远程工作（例如因为我不小心提交到了错误的分支，或者因为我在只有我在用的个人分支上运行了 `git commit --amend` 并且想要强制推送）

```
$ git checkout asdf
error: pathspec 'asdf' did not match any file(s) known to git

```

这有点奇怪，因为我本意是要检出一个 **分支**，但 `git checkout` 却在抱怨一个不存在的 **路径**。

发生这种情况是因为 `git checkout` 的第一个参数可以是分支也可以是路径，而 git 无法知道你打算使用哪一个。这似乎很难改进，但我可能会期望看到类似 “没有这样的分支、提交或路径：asdf” 的提示。

**我在这里喜欢做的事情：**

+   理论上使用 `git switch` 会更好，但我仍然会继续使用 `git checkout`

+   通常我只记得我需要将其解码为 “分支 `asdf` 不存在”

```
$ git switch asdf
fatal: invalid reference: asdf

```

`git switch` 只接受一个分支作为参数（除非你传递 `-d`），那么为什么它会说 `invalid reference: asdf` 而不是 `invalid branch: asdf`？

我觉得原因是内部 `git switch` 尝试在其错误消息中提供帮助：如果你运行 `git switch v0.1` 切换到一个标签，它会说：

```
$ git switch v0.1
fatal: a branch is expected, got tag 'v0.1'` 
```

因此 git 尝试用 `fatal: invalid reference: asdf` 传达的是 “`asdf` 不是一个分支，但也不是标签，或者任何其他引用”。从我的各种 [git 调查](https://jvns.ca/blog/2024/03/28/git-poll-results/) 来看，我觉得很多 git 用户实际上不知道 git 中的 “引用” 是什么，所以我不确定它是否传达了这个意思。

**我在这里喜欢做的事情：**

当一个 git 错误消息说 `reference` 时，我只会在脑海中将其替换为 `branch` 的百分之九十时间。

```
$ git checkout HEAD^
Note: switching to 'HEAD^'.

You are in 'detached HEAD' state. You can look around, make experimental
changes and commit them, and you can discard any commits you make in this
state without impacting any branches by switching back to a branch.

If you want to create a new branch to retain commits you create, you may
do so (now or later) by using -c with the switch command. Example:

  git switch -c Or undo this operation with:

  git switch -

Turn off this advice by setting config variable advice.detachedHead to false

HEAD is now at 182cd3f add "swap byte order" button 
```

这是个棘手的问题。显然很多人对这条消息感到困惑，但显然也已经有很多努力去改进它。我对这个问题没有什么聪明的话要说。

**我在这里喜欢做的事情：**

+   我的 shell 提示我是否处于分离 HEAD 状态，通常我会记得在这种状态下不要进行新的提交

+   当我查看完我想看的旧提交后，我会运行 `git checkout main` 或其他来回到一个分支

这不是一个错误消息，但单独看还是有点令人困惑：

```
$ git status
interactive rebase in progress; onto c694cf8
Last command done (1 command done):
   pick 0a9964d wip
No commands remaining.
You are currently rebasing branch 'main' on 'c694cf8'.
  (fix conflicts and then run "git rebase --continue")
  (use "git rebase --skip" to skip this patch)
  (use "git rebase --abort" to check out the original branch)

Unmerged paths:
  (use "git restore --staged ..." to unstage)
  (use "git add <file>..." to mark resolution)
  both modified:   index.html

no changes added to commit (use "git add" and/or "git commit -a")</file> 
```

有两件事我认为这里可以更清晰：

1.  我认为如果“正在将分支'main'重新基于'c694cf8'”在第一行而不是第五行，那会更好——当前第一行没有说明正在重新基的是哪个分支。

1.  在这种情况下，`c694cf8`实际上是`origin/main`，所以我觉得“正在将分支'main'重新基于'origin/main'”可能会更清晰。

**我在这里喜欢做的事情：**

我的Shell提示包含我当前正在重新基的分支，所以我依赖它而不是`git status`的输出。

```
$ git rebase main
CONFLICT (modify/delete): index.html deleted in 0ce151e (wip) and modified in HEAD.  Version HEAD of index.html left in tree.
error: could not apply 0ce151e… wip

```

我仍然觉得这件事令人困惑——`index.html`在`HEAD`中被修改了。但`HEAD`是什么？它是我在开始合并/变基时正在工作的提交，还是来自另一个分支的提交？（答案是“如果你在进行合并，`HEAD`是你的分支；如果你在进行变基，它是“另一个分支”，但我总是觉得很难记住）

我认为如果可能的话，如果消息列出了分支名称，我会觉得更容易理解，例如：

```
CONFLICT (modify/delete): index.html deleted on `main` and modified on `mybranch` 
```

```
$ git status
On branch master
You have unmerged paths.
  (fix conflicts and run “git commit”)
  (use “git merge –abort” to abort the merge)

Unmerged paths:
  (use “git add/rm <file>…” as appropriate to mark resolution)
    deleted by them: the_file</file>

no changes added to commit (use “git add” and/or “git commit -a”)

```

我觉得这个和前一个消息的困惑程度完全相同：它说“由他们删除：”，但“他们”指的是你进行了合并、变基或挑选樱桃。

+   对于合并，`them`是你合并进来的另一个分支

+   对于变基，`them`是你运行`git rebase`时所在的分支

+   对于挑选樱桃，我猜这是你挑选的提交

**如果我感到困惑，我喜欢做的事情：**

+   尝试记住我做过什么

+   运行`git show main --stat`或类似的命令来查看我在`main`分支上做了什么，如果我记不起来了

```
$ git clean
fatal: clean.requireForce defaults to true and neither -i, -n, nor -f given; refusing to clean

```

我只是觉得有点困惑，你需要查找`-i`、`-n`和`-f`是什么才能理解这个错误消息。我个人太懒了，所以即使我可能已经使用`git clean`十年了，我仍然不知道`-i`代表什么（`interactive`），直到我写下这个。

**如果我感到困惑，我喜欢做的事情：**

通常我只是混乱地运行`git clean -f`来删除所有未跟踪的文件，并希望一切顺利，尽管现在我可能会改用`git clean -i`，因为我知道`-i`代表什么了。看起来安全得多。

### 就这些！

希望其中一些对你有帮助！
