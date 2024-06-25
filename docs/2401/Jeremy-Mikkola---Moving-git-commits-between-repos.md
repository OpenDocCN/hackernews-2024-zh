<!--yml

类别：未分类

日期：2024 年 5 月 27 日 14:25:49

-->

# Jeremy Mikkola - 将 git 提交移动到不相关的仓库

> 来源：[`jeremymikkola.com/posts/2017_07_15_move_commits_between_git_repos.html`](https://jeremymikkola.com/posts/2017_07_15_move_commits_between_git_repos.html)

# 将 git 提交移动到不同的仓库

发表于 2017 年 7 月 15 日

你知道吗，可以在两个不相关的 git 仓库之间移动提交吗？

这样做依赖于 Git 的一个有趣属性。它没有“相同”的两个仓库的概念，这意味着它允许在任何两个仓库之间推送和拉取，而不管它们是否持有相同的项目。

（公平地说，git 会告诉您两个仓库没有共同提交时的情况，但这是一个警告，您可以忽略。）

通常仅将提交移动到另一个仓库是不够的。通常，将文件移动到该仓库内的不同路径非常重要。幸运的是，git 有一个命令可以做到这一点。`git filter-branch` 允许在 git 历史中移动文件（以及以各种其他方式重写历史）。

我将演示您可能遇到的两种用例：将一个仓库的一部分提取到自己的仓库中，并相反，从另一个仓库拉取提交。

假设您有一个大型项目的某个子目录，这个子目录实际上应该是自己的项目。当然，您可以创建一个新的仓库并复制文件，但这不会保留它们的提交历史。为了保留提交历史，我们需要做更多的工作。

首先，在另一个副本中开始创建您的项目以进行工作：

```
git clone existing_project/ new_project/
cd new_project/
```

这将现有项目从 `existing_project/` 目录克隆到 `new_project/` 目录中。如果您没有 `existing_project` 的本地副本，请替换项目的克隆 URL。

为了安全起见，我们将删除指向原始项目的 git 远程引用。这将防止意外地将修改后的版本推送回原始仓库。

```
git remote rm origin
```

现在，是时候将仓库过滤为您想要的子目录了。

```
git filter-branch --prune-empty --subdirectory-filter my_sub_directory/
```

然后您就完成了！用快速的 `ls` 和 `git log` 检查结果。

## 从另一个仓库拉取提交

对两个仓库进行相反操作并合并它们要复杂一些。在此示例中，提交将从名为 `source_repo` 的项目中移动到名为 `dest_repo` 的项目中。

首先，向目标仓库添加一个指向源仓库的 git 远程引用：

```
git remote add source ../source_repo
git config remote.source.pushurl "don't push"
```

将 `remote.source.pushurl` 设置为某个随机值会导致推送失败，从而使远程变为只读。这可以防止意外地破坏源仓库。

接下来，我们将从源仓库中将提交拉入目标仓库中的一个分支中。

```
git fetch source
git checkout source/master
git checkout -b source_master
```

这将创建一个名为 `source_master` 的分支，其中包含源仓库中 master 分支的完全副本的历史记录。

假设您希望源仓库中的文件位于名为 `source_files/` 的目录中。与其他示例一样，`git filter-branch` 允许将文件移动到新位置。

```
git filter-branch --tree-filter "mkdir -p source_files; git ls-tree --name-only \$GIT_COMMIT | tr '\n' '\0' | xargs -0 -I FILE mv FILE source_files"
```

在这一点上，我喜欢使用 `git log --stat` 来检查提交中文件的位置，以确保一切正常。

最后，是时候将这段历史与该项目的原始历史结合起来了。

```
git checkout -b source-commits
git rebase master
```

如果你愿意，你也可以选择合并而不是变基。

`source-commits` 分支已准备好合并到主分支。
