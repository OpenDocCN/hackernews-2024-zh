<!--yml

category: 未分类

date: 2024-05-29 12:49:11

-->

# 现代Git命令和功能你应该知道 | Martin Heinz | 个人网站和博客

> 来源：[https://martinheinz.dev/blog/109](https://martinheinz.dev/blog/109)

我们所有人 - 软件工程师 - 每天都在使用`git`，然而大多数人只接触最基本的命令，比如`add`、`commit`、`push`或`pull`，就好像还是2005年一样。

然而，Git自那时起引入了许多功能，使用它们可以让您的生活变得更加轻松，让我们探索一些最近新增的现代`git`命令。

## Switch

自2019年以来，或更准确地说，引入了Git版本2.23的新功能`git switch`，我们可以用它来切换分支：

```
 git switch other-branch
git switch -  # Switch back to previous branch, similar to "cd -"
git switch remote-branch  # Directly switch to remote branch and start tracking it 
```

好了，这很酷，但我们一直以来都在使用`git checkout`来切换分支，为什么需要一个单独的命令呢？`git checkout`是一个非常多功能的命令 - 它可以（除其他功能外）检出或恢复特定文件甚至特定提交，而新的`git switch`*只*切换分支。另外，`switch`执行额外的健全性检查，`checkout`则不会，例如如果切换导致本地更改丢失，`switch`会中止操作。

## Restore

另一个在Git版本2.23中新增的子命令/功能是`git restore`，我们可以使用它将文件恢复到上次提交的版本：

```
 # Unstage changes made to a file, same as "git reset some-file.py"
git restore --staged some-file.py

# Unstage and discard changes made to a file, same as "git checkout some-file.py"
git restore --staged --worktree some-file.py

# Revert a file to some previous commit, same as "git reset commit -- some-file.py"
git restore --source HEAD~2 some-file.py 
```

上述片段中的注释解释了各种`git restore`的工作原理。总体而言，`git restore`替代并简化了一些已经负载过重的`git reset`和`git checkout`的使用情况。另请参阅[这部分文档](https://git-scm.com/docs/git#_reset_restore_and_revert)来比较`revert`、`restore`和`reset`。

## Sparse Checkout

下一个是`git sparse-checkout`，这是一个比较隐晦的功能，添加于Git 2.25，发布于2020年1月13日。

假设您有一个庞大的单体库，将微服务分隔到各个单独的目录中，由于仓库的大小，像`checkout`或`status`这样的命令非常慢，但也许您真的只需要处理单个子树/目录。好吧，`git sparse-checkout`来拯救您：

```
 $ git clone --no-checkout https://github.com/derrickstolee/sparse-checkout-example
$ cd sparse-checkout-example
$ git sparse-checkout init --cone  # Configure git to only match files in root directory
$ git checkout main  # Checkout only files in root directory
$ ls
bootstrap.sh  LICENSE.md  README.md

$ git sparse-checkout set service/common

$ ls
bootstrap.sh  LICENSE.md  README.md  service

$ tree .
.
├── bootstrap.sh
├── LICENSE.md
├── README.md
└── service
    ├── common
    │   ├── app.js
    │   ├── Dockerfile
    ... ... 
```

在上面的示例中，我们首先克隆了仓库，而不实际检出所有文件。然后我们使用`git sparse-checkout init --cone`来配置`git`只匹配仓库根目录中的文件。因此，在运行检出后，我们只有3个文件而不是整个树。然后，要下载/检出特定目录，我们使用`git sparse-checkout set ...`。

如前所述，当在本地使用大型仓库时，这种方法非常方便，但在CI/CD中同样有用，可以提高流水线的性能，当您只想构建/部署单个单体库的一部分时，不需要检出所有内容。

想要详细了解`sparse-checkout`的文章，请参阅[这篇文章](https://github.blog/2020-01-17-bring-your-monorepo-down-to-size-with-sparse-checkout/)。

## Worktree

不少人可能需要同时在单个应用（仓库）中工作多个特性，或者可能在处理某个特性请求时出现了一个关键bug。

在这些情况下，你可能需要克隆仓库的多个版本/分支，或者需要在当时存储/丢弃你正在处理的内容。这些情况的答案是`git worktree`，发布于2018年9月24日：

```
 git branch
# * dev
# master

git worktree list
# /.../some-repo  ews5ger [dev]

git worktree add -b hotfix ./hotfix master

# Preparing worktree (new branch 'hotfix')
# HEAD is now at 5ea9faa Signed commit.

git worktree list
# /.../test-repo         ews5ger [dev]
# /.../test-repo/hotfix  5ea9faa [hotfix]

cd hotfix/  # Clean worktree, where you can make your changes and push them 
```

这个命令允许我们同时在同一个仓库中检出多个分支。在上面的例子中，我们有2个分支`dev`和`master`。假设我们正在`dev`分支上开发一个特性，但被告知要进行紧急bug修复。我们可以在`master`分支的`./hotfix`子目录中创建一个新的工作树。然后我们可以切换到那个目录，进行我们的更改，推送它们，然后返回到原始的工作树。

欲了解更详细的内容，请参阅[这篇文章](https://opensource.com/article/21/4/git-worktree)。

## 二分查找

最后但并非最不重要的，`git bisect`，虽然并不是很新（Git 1.7.14，2012年5月13日发布），但大多数人只使用了大约2005年左右的`git`功能，所以我认为还是值得展示的。

正如[文档页面](https://git-scm.com/docs/git-bisect)所描述的：*`git-bisect` - 使用二分查找来找出引入bug的提交*：

```
 git bisect start
git bisect bad HEAD  # Provide the broken commit
git bisect good 479420e  # Provide a commit, that you know works
# Bisecting: 2 revisions left to test after this (roughly 1 step)
# [3258487215718444a6148439fa8476e8e7bd49c8] Refactoring.

# Test the current commit...
git bisect bad  # If the commit doesn't work
git bisect good # If the commit works

# Git bisects left or right half of range based on the last command
# Continue testing until you find the culprit

git bisect reset  # Reset to original commit 
```

我们首先要显式地使用`git bisect start`开始二分查找会话，之后我们提供不工作的提交（最可能是`HEAD`）和最后一个已知的工作提交或标签。有了这些信息，`git`将检出介于*"bad"*和*"good"*提交之间的提交。在这一点上，我们需要测试该版本是否有bug，然后使用`git bisect good`告诉`git`它工作正常或者`git bisect bad`告诉`git`它不工作。我们不断重复这个过程，直到没有提交剩余，`git`会告诉我们引入问题的是哪个提交。

我建议查看[文档页面](https://git-scm.com/docs/git-bisect)，其中展示了几个`git bisect`的更多选项，包括可视化、重播或跳过提交。

## 结论

如果你搜索与`git`相关的问题，很可能会最终看到*StackOverflow*上的一个答案，其点赞数可能达到几千。尽管这个答案可能仍然有效，但它很可能已经过时，因为它是10年前编写的。因此，可能有更好、更简单、更容易的方法来处理。因此，当面对某个`git`问题时，我建议查看[git文档](https://git-scm.com/)获取更新的命令，这些命令都有很多优秀的示例，或者探索`man`页面，查看多年来添加到这些经典命令中的许多标志和选项。
