<!--yml

类别：未分类

日期：2024-05-27 15:02:36

-->

# [Git示例](https://antonz.org/git-by-example/)：交互式指南

> 来源：[https://antonz.org/git-by-example/](https://antonz.org/git-by-example/)

# Git示例：交互式指南

Git是*当今软件开发中使用的*分布式版本控制系统。它非常强大，但也以其不那么明显的语法而闻名。

我厌倦了一遍又一遍地搜索相同的Git命令。因此，我创建了一个交互式的逐步指南，介绍Git操作，从基础到高级。您可以从头到尾阅读它（希望能）学到更多关于Git的知识，或者跳转到您感兴趣的特定用例。

随意通过更改命令并点击*运行*来尝试这些示例。

[概念](#concepts) • [基础](#basics) • [分支与合并](#branch-and-merge) • [本地与远程](#local-and-remote) • [撤销](#undo) • [高级](#advanced-stuff) • [最终思考](#final-thoughts)

本指南还提供其他格式：

## 概念

这是指南中唯一的理论内容。我将其简化到π == 3的水平，请不要苛求，如果您是Git专家。

### 工作树，暂存区，仓库

```
┌──────────────┐         ┌──────────────┐ │ local        │ push ─> │ remote       │ │ repo         │ <- pull │ repo         │ └──────────────┘         └──────────────┘ check │  ↑↓ commit / reset out   │ ┌──────────────┐  │ │ staging area │ │ └──────────────┘ ▽  ↑↓ add / restore ┌──────────────┐ │ working tree │ │ .            │ │ ├── go.mod   │ │ └── main.go  │ └──────────────┘ 
```

*工作树*是项目在任意时刻的片段（通常是当前时刻）。当你添加或编辑代码时，你改变了工作树。

*暂存区*是你在将工作树的更改永久保存之前暂存这些更改的地方。

*仓库*（repository）是整个项目历史中所有永久更改（*提交*）的集合。通常有一个单一的*远程*仓库（由GitHub/GitLab等管理），以及许多*本地*仓库 — 每个参与项目的开发者都有一个。

当你把暂存区的更改永久保存时，它会从暂存区移除并*提交*到本地仓库。提交是该更改的永久记录。仓库包含所有已经提交的内容。

当你*checkout*一个特定的提交时，工作树将更新以反映该提交时项目的状态。

本地和远程仓库经常同步，以便所有仓库包含所有开发者的所有提交。

### 分支，标签，HEAD

```
 main             ○ v1.1 feat-2 │               │  ╲│               • │ feat-1        │ │╱              ○ v1.0 │               │ 
```

*分支*是项目现实的另一个版本。通常有一个主分支，以及用于开发功能的单独分支。当功能分支上的工作完成时，它会被合并到主分支（或被丢弃）。

*标签*是项目的一种命名状态。通常在主分支上为重要的里程碑（如发布）创建标签。

当前检出的提交（通常是分支中的最新提交）被称为*HEAD*。

现在无聊的内容已经介绍完了，让我们开始做一些实际操作吧！

## 基础

让我们从本地仓库上的基本Git操作开始。

[初始化仓库](#init-repo) • [添加文件](#add-file) • [编辑文件](#edit-file) • [重命名文件](#rename-file) • [删除文件](#delete-file) • [显示状态](#show-current-status) • [显示日志](#show-commit-log) • [显示提交](#show-specific-commit) • [搜索](#search-repo)

### 初始化仓库

创建一个空仓库：

<codapi-snippet id="init-1" sandbox="shell" editor="basic" template="init.sh" output="">```
Initialized empty Git repository in /tmp/repo/.git/ 
```

设置用户姓名和电子邮件（它们是必需的）：

```
git config user.email alice@example.com git config user.name "Alice Zakas" 
```

<codapi-snippet id="init-2" sandbox="shell" editor="basic" template="init.sh" depends-on="init-1" output-tail="" output="">使用 `--global` 标志来在操作系统用户级别而不是仓库级别设置姓名和电子邮件。

显示用户和仓库配置：

```
git config --list --show-origin 
```

<codapi-snippet id="init-3" sandbox="shell" editor="basic" template="init.sh" depends-on="init-2" output-tail="" output="">```
file:/sandbox/.gitconfig	user.email=sandbox@example.com file:/sandbox/.gitconfig	user.name=sandbox file:/sandbox/.gitconfig	init.defaultbranch=main file:.git/config	core.repositoryformatversion=0 file:.git/config	core.filemode=true file:.git/config	core.bare=false file:.git/config	core.logallrefupdates=true file:.git/config	user.email=alice@example.com file:.git/config	user.name=Alice Zakas 
```

### 添加文件

创建文件并将其添加到暂存区：

```
echo "git is awesom" > message.txt git add message.txt 
```

<codapi-snippet id="add-1" sandbox="shell" editor="basic" template="add.sh" output="">查看暂存区中的更改：

<codapi-snippet id="add-2" sandbox="shell" editor="basic" template="add.sh" depends-on="add-1" output-tail="" output="">```
diff --git a/message.txt b/message.txt new file mode 100644 index 0000000..0165e86 --- /dev/null +++ b/message.txt @@ -0,0 +1 @@ +git is awesom 
```

提交到本地仓库：

```
git commit -m "add message" 
```

<codapi-snippet id="add-3" sandbox="shell" editor="basic" template="add.sh" depends-on="add-1" output-tail="" output="">```
[main (root-commit) 3a2bd8f] add message  1 file changed, 1 insertion(+) create mode 100644 message.txt 
```

### 编辑文件

编辑之前提交的文件：

```
echo "git is awesome" > message.txt 
```

<codapi-snippet id="edit-1" sandbox="shell" editor="basic" template="edit.sh" output="">查看本地更改：

<codapi-snippet id="edit-2" sandbox="shell" editor="basic" template="edit.sh" depends-on="edit-1" output-tail="" output="">```
diff --git a/message.txt b/message.txt index 0165e86..118f108 100644 --- a/message.txt +++ b/message.txt @@ -1 +1 @@ -git is awesom +git is awesome 
```

添加修改过的文件并在一条命令中提交：

```
git commit -am "edit message" 
```

<codapi-snippet id="edit-3" sandbox="shell" editor="basic" template="edit.sh" depends-on="edit-1" output-tail="" output="">```
[main ecdeb79] edit message  1 file changed, 1 insertion(+), 1 deletion(-) 
```

注意，`-a` 不会添加新文件，只会将已提交文件的更改添加。

### 重命名文件

重命名之前提交的文件：

```
git mv message.txt praise.txt 
```

<codapi-snippet id="mv-1" sandbox="shell" editor="basic" template="mv.sh" output="">更改已在暂存区中，因此 `git diff` 不会显示它。使用 `--cached`：

<codapi-snippet id="mv-2" sandbox="shell" editor="basic" template="mv.sh" depends-on="mv-1" output-tail="" output="">```
diff --git a/message.txt b/praise.txt similarity index 100% rename from message.txt rename to praise.txt 
```

提交更改：

```
git commit -m "rename message.txt" 
```

<codapi-snippet id="mv-3" sandbox="shell" editor="basic" template="mv.sh" depends-on="mv-1" output-tail="" output="">```
[main d768287] rename message.txt  1 file changed, 0 insertions(+), 0 deletions(-) rename message.txt => praise.txt (100%) 
```

### 删除文件

删除之前提交的文件：

<codapi-snippet id="rm-1" sandbox="shell" editor="basic" template="rm.sh" output="">更改已在暂存区中，因此 `git diff` 不会显示它。使用 `--cached`：

<codapi-snippet id="rm-2" sandbox="shell" editor="basic" template="rm.sh" depends-on="rm-1" output-tail="" output="">```
diff --git a/message.txt b/message.txt deleted file mode 100644 index 0165e86..0000000 --- a/message.txt +++ /dev/null @@ -1 +0,0 @@ -git is awesom 
```

提交更改：

```
git commit -m "delete message.txt" 
```

<codapi-snippet id="rm-3" sandbox="shell" editor="basic" template="rm.sh" depends-on="rm-1" output-tail="" output="">```
[main 6a2d19b] delete message.txt  1 file changed, 1 deletion(-) delete mode 100644 message.txt 
```

### 显示当前状态

编辑之前提交的文件并将更改添加到暂存区：

```
echo "git is awesome" > message.txt git add message.txt 
```

<codapi-snippet id="status-1" sandbox="shell" editor="basic" template="status.sh" output="">创建新文件：

```
echo "git is great" > praise.txt 
```

<codapi-snippet id="status-2" sandbox="shell" editor="basic" template="status.sh" depends-on="status-1" output-tail="" output="">显示工作树状态：

<codapi-snippet id="status-3" sandbox="shell" editor="basic" template="status.sh" depends-on="status-2" output-tail="" output="">```
On branch main Changes to be committed:  (use "git restore --staged <file>..." to unstage)  modified:   message.txt   Untracked files:  (use "git add <file>..." to include in what will be committed)  praise.txt 
```

请注意，`message.txt` 在暂存区，而 `praise.txt` 未被跟踪。

### 显示提交日志

显示提交：

<codapi-snippet sandbox="shell" editor="basic" template="log.sh" output="">```
commit ecdeb79aad4565d8d7d725678ffadc48b3cdec52 Author: sandbox <sandbox@example.com> Date:   Thu Mar 14 15:00:00 2024 +0000   edit message   commit 3a2bd8f0929c605193120bd1ad12732f49457e99 Author: sandbox <sandbox@example.com> Date:   Thu Mar 14 15:00:00 2024 +0000   add message 
```

仅显示提交消息和短哈希：

<codapi-snippet sandbox="shell" editor="basic" template="log.sh" output="">```
ecdeb79 edit message 3a2bd8f add message 
```

显示提交的ASCII图表：

<codapi-snippet sandbox="shell" editor="basic" template="log.sh" output="">```
* commit ecdeb79aad4565d8d7d725678ffadc48b3cdec52 | Author: sandbox <sandbox@example.com> | Date:   Thu Mar 14 15:00:00 2024 +0000 | |     edit message | * commit 3a2bd8f0929c605193120bd1ad12732f49457e99  Author: sandbox <sandbox@example.com> Date:   Thu Mar 14 15:00:00 2024 +0000   add message 
```

显示紧凑的ASCII图表：

```
git log --oneline --graph 
```

<codapi-snippet sandbox="shell" editor="basic" template="log.sh" output="">```
* ecdeb79 edit message * 3a2bd8f add message 
```

### 显示特定提交

显示最后一次提交内容：

<codapi-snippet sandbox="shell" editor="basic" template="show.sh" output="">```
commit ecdeb79aad4565d8d7d725678ffadc48b3cdec52 Author: sandbox <sandbox@example.com> Date:   Thu Mar 14 15:00:00 2024 +0000   edit message   diff --git a/message.txt b/message.txt index 0165e86..118f108 100644 --- a/message.txt +++ b/message.txt @@ -1 +1 @@ -git is awesom +git is awesome 
```

显示倒数第二次提交：

<codapi-snippet sandbox="shell" editor="basic" template="show.sh" output="">```
commit 3a2bd8f0929c605193120bd1ad12732f49457e99 Author: sandbox <sandbox@example.com> Date:   Thu Mar 14 15:00:00 2024 +0000   add message   diff --git a/message.txt b/message.txt new file mode 100644 index 0000000..0165e86 --- /dev/null +++ b/message.txt @@ -0,0 +1 @@ +git is awesom 
```

使用 `HEAD~n` 显示倒数第n次提交，或者使用特定的提交哈希代替 `HEAD~n`。

### 搜索仓库

有3次提交，每次都向 `message.txt` 添加了一行：

<codapi-snippet sandbox="shell" editor="basic" template="grep.sh" output="">```
cc5b883 no debates 2774a8b is great 31abe57 is awesome 
```

当前 `message.txt` 的状态：

<codapi-snippet sandbox="shell" editor="basic" template="grep.sh" output="">```
git is awesome git is great there is nothing to debate 
```

在工作树中搜索（当前状态）：

<codapi-snippet sandbox="shell" editor="basic" template="grep.sh" output="">```
message.txt:there is nothing to debate 
```

搜索项目到倒数第二次提交为止：

<codapi-snippet sandbox="shell" editor="basic" template="grep.sh" output="">```
HEAD~1:message.txt:git is great 
```

您可以使用特定的提交哈希代替 `HEAD~n`。

## 分支和合并

让我们深入了解合并的奇妙世界。

[分支](#branch) • [合并](#merge) • [变基](#rebase) • [压缩](#squash) • [挑选](#cherry-pick)

### 分支

显示分支（现在只有 `main`）：

<codapi-snippet id="branch-1" sandbox="shell" editor="basic" template="branch.sh" output="">创建并切换到新分支：

```
git branch ohmypy git switch ohmypy 
```

<codapi-snippet id="branch-2" sandbox="shell" editor="basic" template="branch.sh" output="">```
Switched to branch 'ohmypy' 
```

显示分支（当前为 `ohmypy`）：

<codapi-snippet id="branch-3" sandbox="shell" editor="basic" template="branch.sh" depends-on="branch-2" output-tail="" output="">添加并提交文件：

```
echo "print('git is awesome')" > ohmy.py git add ohmy.py git commit -m "ohmy.py" 
```

<codapi-snippet id="branch-4" sandbox="shell" editor="basic" template="branch.sh" depends-on="branch-2" output-tail="" output="">```
[ohmypy a715138] ohmy.py  1 file changed, 1 insertion(+) create mode 100644 ohmy.py 
```

仅显示来自 `ohmypy` 分支的提交：

```
git log --oneline main..ohmypy 
```

<codapi-snippet id="branch-5" sandbox="shell" editor="basic" template="branch.sh" depends-on="branch-4" output-tail="" output="">### 合并

显示所有分支的提交（`main` 中有两次提交，`ohmypy` 中有一次）：

```
git log --all --oneline --graph 
```

<codapi-snippet id="merge-1" sandbox="shell" editor="basic" template="merge.sh" output="">```
* ecdeb79 edit message | * a715138 ohmy.py |/ * 3a2bd8f add message 
```

现在我们在`main`分支上，让我们将`ohmypy`分支重新合并到主分支上：

<codapi-snippet id="merge-2" sandbox="shell" editor="basic" template="merge.sh" output="">```
Merge made by the 'ort' strategy.  ohmy.py | 1 + 1 file changed, 1 insertion(+) create mode 100644 ohmy.py 
```

没有冲突，因此git会自动提交。显示新的提交历史：

```
git log --all --oneline --graph 
```

<codapi-snippet id="merge-3" sandbox="shell" editor="basic" template="merge.sh" depends-on="merge-2" output-tail="" output="">```
*   7d5ac4f Merge branch 'ohmypy' |\ | * a715138 ohmy.py * | ecdeb79 edit message |/ * 3a2bd8f add message 
```

### 变基

显示所有分支的提交（`main`分支有两个提交，`ohmypy`分支有一个提交）：

```
git log --all --oneline --graph 
```

<codapi-snippet id="rebase-1" sandbox="shell" editor="basic" template="rebase.sh" output="">```
* ecdeb79 edit message | * a715138 ohmy.py |/ * 3a2bd8f add message 
```

现在我们在`main`分支上，让我们将`ohmypy`分支重新合并到主分支上：

<codapi-snippet id="rebase-2" sandbox="shell" editor="basic" template="rebase.sh" output="">```
Rebasing (1/1) Successfully rebased and updated refs/heads/main. 
```

注意新的提交历史是线性的，不像我们执行`git merge ohmypy`时的情况：

```
git log --all --oneline --graph 
```

<codapi-snippet id="rebase-3" sandbox="shell" editor="basic" template="rebase.sh" depends-on="rebase-2" output-tail="" output="">```
* c2b0c60 edit message * a715138 ohmy.py * 3a2bd8f add message 
```

变基会重写历史。因此最好不要变基已经推送到远程的分支。

### 压缩

显示所有分支的提交（`main`分支有两个提交，`ohmypy`分支有三个提交）：

```
git log --all --oneline --graph 
```

<codapi-snippet id="squash-1" sandbox="shell" editor="basic" template="squash.sh" output="">```
* ecdeb79 edit message | * b9a7d0f ohmy.lua | * 5ca4d55 ohmy.sh | * a715138 ohmy.py |/ * 3a2bd8f add message 
```

如果我们执行`git merge ohmypy`将`ohmypy`分支合并到`main`分支，那么主分支将接收来自ohmypy的所有三个提交。

有时我们更喜欢将所有分支提交“压缩”成一个单独的提交，然后合并到主分支。让我们来做吧。

切换到`ohmypy`分支：

<codapi-snippet id="squash-2" sandbox="shell" editor="basic" template="squash.sh" output="">```
Switched to branch 'ohmypy' 
```

将所有`ohmypy`的更改合并为一个单独的提交在工作目录中：

<codapi-snippet id="squash-3" sandbox="shell" editor="basic" template="squash.sh" depends-on="squash-2" output-tail="" output="">```
Squash commit -- not updating HEAD 
```

提交合并后的更改：

```
git commit -m "ohmy[py,sh,lua]" 
```

<codapi-snippet id="squash-4" sandbox="shell" editor="basic" template="squash.sh" depends-on="squash-3" output-tail="" output="">```
[ohmypy 4f2a17f] ohmy[py,sh,lua]  1 file changed, 1 insertion(+), 1 deletion(-) 
```

切换回`main`分支：

合并`ohmypy`分支到`main`分支：

```
git merge --no-ff ohmypy -m "ohmy[py,sh,lua]" 
```

<codapi-snippet id="squash-6" sandbox="shell" editor="basic" template="squash.sh" depends-on="squash-5" output-tail="" output="">```
Merge made by the 'ort' strategy.  ohmy.lua | 1 + ohmy.py  | 1 + ohmy.sh  | 1 + 3 files changed, 3 insertions(+) create mode 100644 ohmy.lua create mode 100644 ohmy.py create mode 100644 ohmy.sh 
```

注意`main`中的单个提交由`ohmypy`中的三个提交组成：

```
git log --all --oneline --graph 
```

<codapi-snippet id="squash-5" sandbox="shell" editor="basic" template="squash.sh" depends-on="squash-4" output-tail="" output="">```
*   008dce6 ohmy[py,sh,lua] |\ | * 4f2a17f ohmy[py,sh,lua] | * b9a7d0f ohmy.lua | * 5ca4d55 ohmy.sh | * a715138 ohmy.py * | ecdeb79 edit message |/ * 3a2bd8f add message 
```

### 挑选

我在`message.txt`中有一个拼写错误：

<codapi-snippet id="cherry-1" sandbox="shell" editor="basic" template="cherry.sh" output="">而我不小心在`ohmypy`分支而不是`main`分支中修复了它：

```
git log --all --oneline --graph --decorate 
```

<codapi-snippet id="cherry-2" sandbox="shell" editor="basic" template="cherry.sh" output="">```
* 568193c (HEAD -> main) add praise | * bbce161 (ohmypy) ohmy.sh | * cbb09c6 fix typo | * a715138 ohmy.py |/ * 3a2bd8f add message 
```

我还没有准备好合并整个`ohmypy`分支，所以我将挑选这个提交：

<codapi-snippet id="cherry-3" sandbox="shell" editor="basic" template="cherry.sh" output="">```
[main b23d3ee] fix typo  Date: Thu Mar 14 15:00:00 2024 +0000 1 file changed, 1 insertion(+), 1 deletion(-) 
```

`cherry-pick`将注释应用到`main`分支：

```
git log --all --oneline --graph --decorate 
```

<codapi-snippet id="cherry-4" sandbox="shell" editor="basic" template="cherry.sh" depends-on="cherry-3" output-tail="" output="">```
* b23d3ee (HEAD -> main) fix typo * 568193c add praise | * bbce161 (ohmypy) ohmy.sh | * cbb09c6 fix typo | * a715138 ohmy.py |/ * 3a2bd8f add message 
```

修正了拼写错误：

<codapi-snippet id="cherry-5" sandbox="shell" editor="basic" template="cherry.sh" depends-on="cherry-3" output-tail="" output="">## 本地和远程

在本地存储库中工作很有趣，但添加远程存储库更有趣。

[push](#push) • [pull](#pull) • [resolve](#resolve-conflict) • [push branch](#push-branch) • [fetch branch](#fetch-branch) • [tags](#tags)

### 推送

Alice想克隆我们的存储库并做一些更改。

克隆远程存储库：

```
git clone /tmp/remote.git /tmp/alice 
```

<codapi-snippet id="push-1" sandbox="shell" editor="basic" template="push.sh" output="">```
Cloning into '/tmp/alice'... done. 
```

通常在这里你会看到GitHub/GitLab等的URL，但我们的“远程”存储库在同一台机器上的`/tmp/remote.git`。

设置用户名和电子邮件：

```
cd /tmp/alice git config user.email alice@example.com git config user.name "Alice Zakas" 
```

<codapi-snippet id="push-2" sandbox="shell" editor="basic" template="push.sh" depends-on="push-1" output-tail="" output="">做一些更改并提交：

```
echo "Git is awesome!" > message.txt git commit -am "edit from alice" 
```

<codapi-snippet id="push-3" sandbox="shell" editor="basic" template="push.sh" depends-on="push-2" output-tail="" output="">```
[main b9714f2] edit from alice  1 file changed, 1 insertion(+), 1 deletion(-) 
```

将本地提交的更改推送到远程存储库：

<codapi-snippet id="push-4" sandbox="shell" editor="basic" template="push.sh" depends-on="push-3" output-tail="" output="">### 拉取

我想将Alice的更改拉到本地存储库。

Alice还没有提交：

<codapi-snippet id="pull-1" sandbox="shell" editor="basic" template="pull.sh" output="">从远程存储库拉取最新更改：

<codapi-snippet id="pull-2" sandbox="shell" editor="basic" template="pull.sh" output="">```
Updating 3a2bd8f..b9714f2 Fast-forward  message.txt | 2 +- 1 file changed, 1 insertion(+), 1 deletion(-) 
```

本地存储库现在包含来自Alice的提交：

<codapi-snippet id="pull-3" sandbox="shell" editor="basic" template="pull.sh" depends-on="pull-2" output-tail="" output="">```
b9714f2 edit from alice 3a2bd8f add message 
```

### 解决冲突

我有一个本地提交（尚未推送到远程），与Alice的更改（已经推送到远程）冲突，所以我需要解决它。

从远程存储库拉取更改：

<codapi-snippet id="conflict-1" sandbox="shell" editor="basic" template="conflict.sh" output="">```
Auto-merging message.txt CONFLICT (content): Merge conflict in message.txt Automatic merge failed; fix conflicts and then commit the result.From /tmp/remote  3a2bd8f..b9714f2  main       -> origin/main (exit status 1) 
```

`message.txt`中有冲突！让我们展示一下：

<codapi-snippet id="conflict-2" sandbox="shell" editor="basic" template="conflict.sh" depends-on="conflict-1" output-tail="" output="">```
<<<<<<< HEAD
git is awesome
=======
Git is awesome!
>>>>>>> b9714f2c59c7dbd1205cf20e0a99939b7a686d97 
```

我更喜欢Alice的版本，所以让我们选择它：

```
git checkout --theirs -- message.txt # to choose our version, use --ours 
```

<codapi-snippet id="conflict-3" sandbox="shell" editor="basic" template="conflict.sh" depends-on="conflict-1" output-tail="" output="">将解决后的文件添加到暂存区并完成合并：

```
git add message.txt git commit -m "merge alice" 
```

<codapi-snippet id="conflict-4" sandbox="shell" editor="basic" template="conflict.sh" depends-on="conflict-3" output-tail="" output="">```
[main cbb6112] merge alice 
```

### 推送分支

创建本地的`ohmypy`分支：

```
git branch ohmypy git switch ohmypy 
```

<codapi-snippet id="push-branch-1" sandbox="shell" editor="basic" template="push-branch.sh" output="">```
Switched to branch 'ohmypy' 
```

添加并提交一个文件：

```
echo "print('git is awesome')" > ohmy.py git add ohmy.py git commit -m "ohmy.py" 
```

<codapi-snippet id="push-branch-2" sandbox="shell" editor="basic" template="push-branch.sh" depends-on="push-branch-1" output-tail="" output="">```
[ohmypy c64073e] ohmy.py  1 file changed, 1 insertion(+) create mode 100644 ohmy.py 
```

将本地分支推送到远程：

```
git push -u origin ohmypy 
```

<codapi-snippet id="push-branch-3" sandbox="shell" editor="basic" template="push-branch.sh" depends-on="push-branch-2" output-tail="" output="">```
branch 'ohmypy' set up to track 'origin/ohmypy'. 
```

显示本地和远程分支：

<codapi-snippet id="push-branch-4" sandbox="shell" editor="basic" template="push-branch.sh" depends-on="push-branch-3" output-tail="" output="">```
 main * ohmypy  remotes/origin/main remotes/origin/ohmypy 
```

### 获取分支

获取远程分支：

<codapi-snippet id="fetch-1" sandbox="shell" editor="basic" template="fetch.sh" output="">远程有`ohmypy`分支，但本地未检出：

<codapi-snippet id="fetch-2" sandbox="shell" editor="basic" template="fetch.sh" depends-on="fetch-1" output-tail="" output="">检出`ohmypy`分支：

```
git switch ohmypy # or: git checkout ohmypy 
```

<codapi-snippet id="fetch-3" sandbox="shell" editor="basic" template="fetch.sh" depends-on="fetch-1" output-tail="" output="">```
branch 'ohmypy' set up to track 'origin/ohmypy'. 
```

显示分支：

<codapi-snippet id="fetch-4" sandbox="shell" editor="basic" template="fetch.sh" depends-on="fetch-3" output-tail="" output="">为最新提交创建一个标签：

<codapi-snippet id="tag-1" sandbox="shell" editor="basic" template="tag.sh" output="">为倒数第n次提交创建一个标签：

```
git tag 0.1.0-alpha HEAD~1 
```

<codapi-snippet id="tag-2" sandbox="shell" editor="basic" template="tag.sh" depends-on="tag-1" output-tail="" output="">可以使用提交哈希代替`HEAD~n`。

显示标签：

<codapi-snippet id="tag-3" sandbox="shell" editor="basic" template="tag.sh" depends-on="tag-2" output-tail="" output="">显示带标签的简明日志：

```
git log --decorate --oneline 
```

<codapi-snippet id="tag-4" sandbox="shell" editor="basic" template="tag.sh" depends-on="tag-2" output-tail="" output="">```
ecdeb79 (HEAD -> main, tag: 0.1.0, origin/main) edit message 3a2bd8f (tag: 0.1.0-alpha) add message 
```

删除标签：

<codapi-snippet id="tag-5" sandbox="shell" editor="basic" template="tag.sh" depends-on="tag-2" output-tail="" output="">```
Deleted tag '0.1.0-alpha' (was 3a2bd8f) 
```

将标签推送到远程：

<codapi-snippet id="tag-6" sandbox="shell" editor="basic" template="tag.sh" depends-on="tag-5" output-tail="" output="">## 撤销

"该死，我怎么撤销我刚才做的事情？" — 这是 Git 中永恒的问题。让我们一劳永逸地回答它。

[修订提交](#amend-commit) • [撤销未提交的更改](#undo-uncommitted-changes) • [本地撤销](#undo-local-commit) • [远程撤销](#undo-remote-commit) • [重置历史记录](#rewind-history) • [存储更改](#stash-changes)

### 修订提交

编辑文件并提交：

```
echo "git is awesome" > message.txt git commit -am "edit nessage" 
```

<codapi-snippet id="amend-1" sandbox="shell" editor="basic" template="amend.sh" output="">```
[main c0206a0] edit nessage  1 file changed, 1 insertion(+), 1 deletion(-) 
```

显示提交：

<codapi-snippet id="amend-2" sandbox="shell" editor="basic" template="amend.sh" depends-on="amend-1" output-tail="" output="">```
c0206a0 edit nessage 3a2bd8f add message 
```

我打错字了，所以我想改变提交消息：

```
git commit --amend -m "edit message" 
```

<codapi-snippet id="amend-3" sandbox="shell" editor="basic" template="amend.sh" depends-on="amend-1" output-tail="" output="">```
[main ecdeb79] edit message  Date: Thu Mar 14 15:00:00 2024 +0000 1 file changed, 1 insertion(+), 1 deletion(-) 
```

Git 已经替换了最后一次提交：

<codapi-snippet id="amend-4" sandbox="shell" editor="basic" template="amend.sh" depends-on="amend-3" output-tail="" output="">```
ecdeb79 edit message 3a2bd8f add message 
```

要为最后 `n` 次提交之一更改提交消息，使用 `git rebase -i HEAD~n`（交互式），并按屏幕上的说明操作。

如果提交还没有推送到远程仓库，修改只适用于这种情况！

### 撤销未提交的更改

编辑之前提交的文件并将更改添加到暂存区：

```
echo "git is awesome" > message.txt git add message.txt 
```

<codapi-snippet id="undo-local-1" sandbox="shell" editor="basic" template="undo-local.sh" output="">显示工作树状态：

<codapi-snippet id="undo-local-2" sandbox="shell" editor="basic" template="undo-local.sh" depends-on="undo-local-1" output-tail="" output="">```
On branch main Changes to be committed:  (use "git restore --staged <file>..." to unstage)  modified:   message.txt 
```

从暂存区中移除更改：

```
git restore --staged message.txt 
```

<codapi-snippet id="undo-local-3" sandbox="shell" editor="basic" template="undo-local.sh" depends-on="undo-local-1" output-tail="" output="">本地文件仍然被修改，但没有暂存以进行提交：

<codapi-snippet id="undo-local-4" sandbox="shell" editor="basic" template="undo-local.sh" depends-on="undo-local-3" output-tail="" output="">```
On branch main Changes not staged for commit:  (use "git add <file>..." to update what will be committed) (use "git restore <file>..." to discard changes in working directory)  modified:   message.txt   no changes added to commit (use "git add" and/or "git commit -a") 
```

现在让我们彻底放弃这些更改：

```
git restore message.txt # or: git checkout message.txt 
```

<codapi-snippet id="undo-local-5" sandbox="shell" editor="basic" template="undo-local.sh" depends-on="undo-local-3" output-tail="" output="">显示文件内容：

<codapi-snippet id="undo-local-6" sandbox="shell" editor="basic" template="undo-local.sh" depends-on="undo-local-5" output-tail="" output="">这些更改已经消失。

### 撤销本地提交

我改变了关于上次提交的想法，我想撤销它。

显示提交：

<codapi-snippet id="undo-commit-1" sandbox="shell" editor="basic" template="undo-commit.sh" output="">```
ecdeb79 edit message 3a2bd8f add message 
```

撤销上一次提交：

<codapi-snippet id="undo-commit-2" sandbox="shell" editor="basic" template="undo-commit.sh" output="">提交已经被删除：

<codapi-snippet id="undo-commit-3" sandbox="shell" editor="basic" template="undo-commit.sh" depends-on="undo-commit-2" output-tail="" output="">但更改仍然在暂存区：

<codapi-snippet id="undo-commit-4" sandbox="shell" editor="basic" template="undo-commit.sh" depends-on="undo-commit-2" output-tail="" output="">```
On branch main Changes to be committed:  (use "git restore --staged <file>..." to unstage)  modified:   message.txt 
```

要删除提交和本地更改，请使用 `--hard` 而不是 `--soft`：

```
git reset --hard HEAD~ git status 
```

<codapi-snippet id="undo-commit-5" sandbox="shell" editor="basic" template="undo-commit.sh" output="">```
HEAD is now at 3a2bd8f add message On branch main nothing to commit, working tree clean 
```

重置只适用于提交还没有推送到远程仓库的情况！

### 撤销远程提交

我改变了关于上次提交的想法，但提交已经推送到远程仓库。

显示提交：

<codapi-snippet id="undo-remote-1" sandbox="shell" editor="basic" template="undo-remote.sh" output="">```
ecdeb79 edit message 3a2bd8f add message 
```

撤销上一次提交：

```
git revert HEAD --no-edit 
```

<codapi-snippet id="undo-remote-2" sandbox="shell" editor="basic" template="undo-remote.sh" output="">```
[main 9ffa044] Revert "edit message"  Date: Thu Mar 14 15:00:00 2024 +0000 1 file changed, 1 insertion(+), 1 deletion(-) 
```

你可以通过使用 `HEAD~n` 恢复到倒数第 `n` 次提交，或者使用特定的提交哈希代替 `HEAD~n`。

由于提交已经推送，git 无法删除它。而是创建一个“撤销”提交：

<codapi-snippet id="undo-remote-3" sandbox="shell" editor="basic" template="undo-remote.sh" depends-on="undo-remote-2" output-tail="" output="">```
9ffa044 Revert "edit message" ecdeb79 edit message 3a2bd8f add message 
```

将“撤销”提交推送到远程：

<codapi-snippet id="undo-remote-4" sandbox="shell" editor="basic" template="undo-remote.sh" depends-on="undo-remote-2" output-tail="" output="">### 倒带历史

显示提交：

```
git log --oneline --graph 
```

<codapi-snippet id="reflog-1" sandbox="shell" editor="basic" template="reflog.sh" output="">```
*   7d5ac4f Merge branch 'ohmypy' |\ | * a715138 ohmy.py * | ecdeb79 edit message |/ * 3a2bd8f add message 
```

按时间顺序显示所有存储库状态：

<codapi-snippet id="reflog-2" sandbox="shell" editor="basic" template="reflog.sh" output="">```
7d5ac4f HEAD@{0}: merge ohmypy: Merge made by the 'ort' strategy. ecdeb79 HEAD@{1}: commit: edit message 3a2bd8f HEAD@{2}: checkout: moving from ohmypy to main a715138 HEAD@{3}: commit: ohmy.py 3a2bd8f HEAD@{4}: checkout: moving from main to ohmypy 3a2bd8f HEAD@{5}: commit (initial): add message 
```

假设我想回到 `HEAD@{3}`：

```
git reset --hard HEAD@{3} 
```

<codapi-snippet id="reflog-3" sandbox="shell" editor="basic" template="reflog.sh" output="">```
HEAD is now at a715138 ohmy.py 
```

这将重置整个存储库和工作树到 `HEAD@{3}` 时的状态：

```
git log --oneline --graph 
```

<codapi-snippet id="reflog-4" sandbox="shell" editor="basic" template="reflog.sh" depends-on="reflog-3" output-tail="" output="">```
* a715138 ohmy.py * 3a2bd8f add message 
```

### 堆栈更改

编辑先前提交的文件：

```
echo "git is awesome" > message.txt git add message.txt 
```

<codapi-snippet id="stash-1" sandbox="shell" editor="basic" template="stash.sh" output="">假设我们需要切换到另一个分支，但尚未提交更改。

堆栈本地更改（即将它们保存在“草稿”中）：

<codapi-snippet id="stash-2" sandbox="shell" editor="basic" template="stash.sh" depends-on="stash-1" output-tail="" output="">```
Saved working directory and index state WIP on main: 3a2bd8f add message 
```

堆栈是一种堆栈结构，因此您可以将多个更改推送到其中：

```
echo "Git is awesome!" > message.txt git stash 
```

<codapi-snippet id="stash-3" sandbox="shell" editor="basic" template="stash.sh" depends-on="stash-2" output-tail="" output="">```
Saved working directory and index state WIP on main: 3a2bd8f add message 
```

显示堆栈内容：

<codapi-snippet id="stash-4" sandbox="shell" editor="basic" template="stash.sh" depends-on="stash-3" output-tail="" output="">```
stash@{0}: WIP on main: 3a2bd8f add message stash@{1}: WIP on main: 3a2bd8f add message 
```

现在我们可以切换到另一个分支并执行某些操作：

```
...(omitted for brevity)... 
```

切换回主分支并重新应用来自堆栈的最新更改：

```
git switch main git stash pop 
```

<codapi-snippet id="stash-5" sandbox="shell" editor="basic" template="stash.sh" depends-on="stash-3" output-tail="" output="">```
On branch main Changes not staged for commit:  (use "git add <file>..." to update what will be committed) (use "git restore <file>..." to discard changes in working directory)  modified:   message.txt   no changes added to commit (use "git add" and/or "git commit -a") Dropped refs/stash@{0} (96af1a51462f29d7b947a7563938847efd5d5aeb) 
```

`pop` 按“后进先出”的顺序返回堆栈中的更改。

清除堆栈：

<codapi-snippet id="stash-6" sandbox="shell" editor="basic" template="stash.sh" depends-on="stash-5" output-tail="" output="">## 高级内容

尽管 git 高手可能对这些功能了如指掌，但大多数开发者从未听说过。让我们修复这个问题。

[日志摘要](#log-summary) • [工作树](#worktree) • [二分法](#bisect) • [部分检出](#partial-checkout) • [部分克隆](#partial-clone)

### 日志摘要

自 1.0 发布（标签 `v1.0`）以来，我们有来自 3 位贡献者的 6 个提交：

```
git log --pretty=format:'%h %an %s %d' 
```

<codapi-snippet sandbox="shell" editor="basic" template="shortlog.sh" output="">```
7611979 bob ohmy.lua  (HEAD -> main, origin/main) ef4f23e bob ohmy.sh 3d8f700 bob ohmy.py c61962c alice no debates 2ab82f6 alice go is great ecdeb79 sandbox edit message 3a2bd8f sandbox add message  (tag: v1.0) 
```

请注意 `--pretty` 选项，该选项自定义日志字段：

```
%h   commit hash %an  author %s   message %d   decoration (e.g. branch name or tag) 
```

按贡献者分组列出提交：

<codapi-snippet sandbox="shell" editor="basic" template="shortlog.sh" output="">```
alice (2):  go is great no debates   bob (3):  ohmy.py ohmy.sh ohmy.lua   sandbox (1):  edit message 
```

一些有用选项：

+   `-n`（`--numbered`）会根据提交者数量降序排序输出。

+   `-s`（`--summary`）选项会省略提交描述，仅输出计数值。

按提交者及其提交数量列出信息：

<codapi-snippet sandbox="shell" editor="basic" template="shortlog.sh" output="">### 工作目录

我正在`ohmypy`分支进行重要的工作：

```
echo "-- pwd --" pwd echo "-- branches --" git branch echo "-- status --" git status 
```

<codapi-snippet id="worktree-1" sandbox="shell" editor="basic" template="worktree.sh" output="">```
-- pwd -- /tmp/repo -- branches --  main * ohmypy -- status -- On branch ohmypy Changes to be committed:  (use "git restore --staged <file>..." to unstage)  new file:   ohmy.py 
```

突然间，我在`main`分支中发现了一个烦人的拼写错误。我可以用`git stash`暂存本地更改，或者使用`git worktree`在同时暂存多个分支。

将主线支暂存至`/tmp/hotfix`目录下：

```
git worktree add -b hotfix /tmp/hotfix main 
```

<codapi-snippet id="worktree-2" sandbox="shell" editor="basic" template="worktree.sh" output="">```
HEAD is now at 3a2bd8f add message 
```

修正拼写错误并提交更改：

```
cd /tmp/hotfix echo "git is awesome" > message.txt git commit -am "fix typo" 
```

<codapi-snippet id="worktree-3" sandbox="shell" editor="basic" template="worktree.sh" depends-on="worktree-2" output-tail="" output="">```
[hotfix c3485cd] fix typo  1 file changed, 1 insertion(+), 1 deletion(-) 
```

将更改推送到远程`main`分支：

```
git push --set-upstream origin main 
```

<codapi-snippet id="worktree-4" sandbox="shell" editor="basic" template="worktree.sh" depends-on="worktree-3" output-tail="" output="">```
branch 'main' set up to track 'origin/main'. 
```

现在我可以回到`/tmp/repo`并继续在`ohmypy`分支中工作。

### 比塞尔算法

我有5个命名不恰当的提交：

<codapi-snippet id="bisect-1" sandbox="shell" editor="basic" template="bisect.sh" output="">```
2f568eb main.sh 31ed915 main.sh f8b2baf main.sh 5e0cf35 main.sh 8f0f1e4 main.sh 51c34ff test.sh 
```

出现失败测试结果：

<codapi-snippet id="bisect-2" sandbox="shell" editor="basic" template="bisect.sh" output="">我将使用比塞尔算法找出引入bug的提交：

<codapi-snippet id="bisect-3" sandbox="shell" editor="basic" template="bisect.sh" output="">```
status: waiting for both good and bad commits 
```

当前的状况显然存在bug，但在我的印象中，第一个`main.sh`提交无误：

```
git bisect bad HEAD git bisect good HEAD~4 
```

<codapi-snippet id="bisect-4" sandbox="shell" editor="basic" template="bisect.sh" depends-on="bisect-3" output-tail="" output="">```
status: waiting for good commit(s), bad commit known Bisecting: 1 revision left to test after this (roughly 1 step) [f8b2baf93964ec9e0daa87c9ed262bbf5cf66b67] main.sh 
```

Git 已自动暂存中间的提交。让我们测试一下：

<codapi-snippet id="bisect-5" sandbox="shell" editor="basic" template="bisect.sh" depends-on="bisect-4" output-tail="" output="">测试通过。标记该提交为正常：

<codapi-snippet id="bisect-6" sandbox="shell" editor="basic" template="bisect.sh" depends-on="bisect-5" output-tail="" output="">```
Bisecting: 0 revisions left to test after this (roughly 0 steps) [31ed915660c42a00aa30b51520a16e3f48201299] main.sh 
```

Git 已自动暂存中间的提交。让我们测试一下：

<codapi-snippet id="bisect-7" sandbox="shell" editor="basic" template="bisect.sh" depends-on="bisect-6" output-tail="" output="">测试失败。显示提交详情：

<codapi-snippet id="bisect-8" sandbox="shell" editor="basic" template="bisect.sh" depends-on="bisect-7" output-tail="" output="">```
commit 31ed915660c42a00aa30b51520a16e3f48201299 Author: sandbox <sandbox@example.com> Date:   Thu Mar 14 15:00:00 2024 +0000   main.sh   diff --git a/main.sh b/main.sh index 7f8f78c..ce533e0 100644 --- a/main.sh +++ b/main.sh @@ -1,2 +1,2 @@  # sum two numbers -echo $(expr $1 + $2) +echo $(expr $1 - $2) 
```

引入错误的提交（减法替换为加法）！

### 部分暂存

远程代码库如下所示：

```
. ├── go.mod ├── main.go ├── products │   └── products.go └── users  └── users.go 
```

我们将仅选择性地暂存某些目录。

克隆代码库，但不暂存工作区：

```
git clone --no-checkout /tmp/remote.git /tmp/repo cd /tmp/repo 
```

<codapi-snippet id="sparse-checkout-1" sandbox="shell" editor="basic" template="sparse-checkout.sh" output="">```
Cloning into '/tmp/repo'... done. 
```

告诉git仅检出根目录和`users`目录：

```
git sparse-checkout init --cone git sparse-checkout set users 
```

<codapi-snippet id="sparse-checkout-2" sandbox="shell" editor="basic" template="sparse-checkout.sh" depends-on="sparse-checkout-1" output-tail="" output="">检出目录：

<codapi-snippet id="sparse-checkout-3" sandbox="shell" editor="basic" template="sparse-checkout.sh" depends-on="sparse-checkout-2" output-tail="" output="">```
Your branch is up to date with 'origin/main'. 
```

仅检出根目录和用户目录：

<codapi-snippet id="sparse-checkout-4" sandbox="shell" editor="basic" template="sparse-checkout.sh" depends-on="sparse-checkout-3" output-tail="" output="">```
. ├── go.mod ├── main.go └── users  └── users.go   1 directories, 3 files 
```

未检出`products`目录。

### 部分克隆

我们之前尝试的部分检出方法仍会克隆整个存储库。因此，如果存储库本身很大（如果具有悠久历史或大型二进制文件，则通常如此），克隆步骤可能会很慢且耗费流量。

为了在克隆时减少下载数据量，请使用以下命令之一进行*部分克隆*：

```
# Download commits and trees (directories), # but not blobs (file contents): git clone --filter=blob:none file:///tmp/remote.git   # Download commits only, without trees (directories) # or blobs (file contents): git clone --filter=tree:0 file:///tmp/remote.git 
```

在这两种情况下，git将在需要时延迟获取缺失的数据。

请注意，为使此功能正常工作，远程服务器应支持部分克隆（GitHub支持）。

## 最后的思考

我们已涵盖重要的Git操作，从基本编辑到分支和合并，远程同步，撤销更改，以及执行一些适度的魔法。

欲了解更多关于Git的信息，请查看[参考手册](https://git-scm.com/docs)以及由Scott Chacon和Ben Straub撰写的[Pro Git](https://git-scm.com/book)。

愿Git与你同在！

──

P.S. 本文中的交互式示例由[**codapi**](https://codapi.org/)提供支持 —— 这是我正在开发的开源工具。使用它将实时代码片段嵌入到产品文档、在线课程或博客中。

*[* **订阅***](/subscribe/) *以获取最新的帖子。*
