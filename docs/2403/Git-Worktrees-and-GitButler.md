<!--yml

category: 未分类

date: 2024-05-27 14:35:09

-->

# Git工作树和GitButler

> 来源：[https://blog.gitbutler.com/git-worktrees/](https://blog.gitbutler.com/git-worktrees/)

上周，我在DevWorld上做了一个演讲，涉及更多类似"[你以为你了解Git](https://blog.gitbutler.com/git-tips-and-tricks/)"风格的技巧。在这个新演讲中，我涵盖了`git worktree`的内容，因为最近听到它被提到很多次，我想是时候写一篇小文章来解释一下了。

"[工作树](https://git-scm.com/docs/git-worktree?ref=blog.gitbutler.com)"工具在Git中帮助你同时处理多个分支。

由于GitButler解决了相同的基本问题，我将花一分钟解释`git worktree`对于那些没有听说过它的人意味着什么，然后还要比较一下GitButler如何解决相同的核心问题。

## 同时处理分支

所以，这里的核心问题是什么？基本上，就是试图同时处理代码中的两个主题。

让我们举个例子。比如说你正在开发一个功能。你的老板打断你（🙄，老板，对吧？）并说“我们需要修复这个bug”。或者可能是在工作时注意到了bug。无论如何，你需要在进行某事的过程中切换上下文。

现在你有几个选择。

+   你可以修复bug并将其提交到你的功能分支，然后尝试一起部署它们。

+   你可以把所有内容藏起来，创建一个新分支，切换到它，修复bug，提交并推送，然后切换回你之前正在工作的内容。

+   你*可以*在本地克隆仓库到一个新目录中，在那里处理另一个分支，然后将其推送回主分支，再从*那里*向上游推送。 🙄

肯定有更好的方法！

## Worktrees

这是`git worktree`的基本用例。工作树允许你为每个正在积极工作的分支拥有单独的工作目录（和暂存区）。

假设我们正在`my-project`目录中进行项目工作，并决定要创建一个名为`bugfix-123`的新分支。我们可以运行`git worktree add`命令，并提供一个目录来创建并检出该分支。

```
❯ git worktree add -b bugfix-123 ../my-project-branches/bugfix-123
Preparing worktree (new branch 'bugfix-123')
HEAD is now at 5b67eb2 first commit
```

现在我们有两个链接的目录。当我们在`my-project-branches/bugfix-123`目录中编辑文件时，它位于`bugfix-123`分支上。

现在，很酷的一点是，这两个检出都共享同一个核心Git数据库。因此，如果我们在其中任何一个中提交，这些分支上的提交对*所有*都可见。

```
❯ pushd ../my-project-branches/bugfix-123/
~/my-project-branches/bugfix-123 ~/my-project

❯ echo 'new commit content' >> README.md; git commit -am 'new commit'
[bugfix-123 2ff5902] new commit
 1 file changed, 1 insertion(+)

❯ popd
~/my-project

❯ git log --oneline bugfix-123
2ff5902 (bugfix-123) new commit
5b67eb2 (HEAD -> main) first commit
```

这对许多分支都适用，而`git worktree`会记住你项目中它们的所有位置及其当前所在的分支：

```
❯ git worktree list
~/my-project                              5b67eb2 [main]
~/my-project-branches/bugfix-123          2ff5902 [bugfix-123]
~/my-project-branches/feature-foobar-baz  b9b9280 [foobar]
~/my-project-branches/feature-new-widget  daf62e4 [new-widget]
```

你可以使用`git worktree remove`删除你不再使用的工作树，或者你也可以只删除目录并运行`git worktree prune`来将其从列表中删除。

```
❯ rm -Rf ../my-project-branches/feature-foobar-baz

❯ git worktree list
~/my-project                              5b67eb2 [main]
~/my-project-branches/bugfix-123          2ff5902 [bugfix-123]
~/my-project-branches/feature-foobar-baz  5b67eb2 [foobar] prunable
~/my-project-branches/feature-new-widget  5b67eb2 [new-widget]

❯ git worktree prune

❯ git worktree list
~/my-project                              5b67eb2 [main]
~/my-project-branches/bugfix-123          2ff5902 [bugfix-123]
~/my-project-branches/feature-new-widget  5b67eb2 [new-widget]
```

现在，这种方法也有一些缺点。

+   如果你正在使用像VS Code这样的编辑器，你需要为每个目录/分支打开一个新窗口。

+   如果您有任何被忽略的文件，如构建产物，您必须为每个工作树重新构建它们。当您仅在一个目录中进行存储或切换分支时，这就不是问题。

+   工作树是分开的，因此您可以在它们之间轻松地创建合并冲突，而无需知道具体情况。

然而，在某些情况下，工作树可能是一种不错的方式，让您在不混淆当前上下文的情况下在新上下文中工作。

## GitButler vs Worktrees

那么这与 GitButler 的虚拟分支有什么不同？它们也提供了一种同时在多个分支上工作的方式，对吧？

嗯，主要的区别在于虚拟分支使用*单一*工作目录来处理*所有*分支。

让我们举一个非常简单的例子。假设我们要为我们的 README 添加西班牙语和法语的翻译，并且我们希望为每一种语言打开一个 PR，以便由不同的人在可能不同的时间进行审查。

### 如何在 Worktrees 中操作

使用工作树，我们可以为每个新建一个工作树，然后在每个工作树中进行提交并推送。

首先，我们创建两个新的工作树。

```
❯ git worktree add -b sp ../my-project-branches/spanish
Preparing worktree (new branch 'sp')
HEAD is now at 5b67eb2 first commit

❯ git worktree add -b fr ../my-project-branches/french
Preparing worktree (new branch 'fr')
HEAD is now at 5b67eb2 first commit
```

现在，我们进入每个目录，进行更改并提交它们。

```
❯ cd ../my-project-branches/spanish

❯ echo 'Léeme' > README.es.md

❯ git add README.es.md; git commit -am 'spanish readme'
[sp 4d57bfb] spanish readme
 1 file changed, 1 insertion(+)
 create mode 100644 README.es.md

❯ cd ../french/

❯ echo 'Lis-moi' > README.fr.md

❯ git add README.fr.md; git commit -am 'french readme'
[fr 48df685] french readme
 1 file changed, 1 insertion(+)
 create mode 100644 README.fr.md
```

现在如果我们回到主目录，我们可以看到那里有两个分支，并且可以从中创建 PR。

```
❯ cd ../../my-project

❯ git log --oneline --graph --all
* 48df685 (fr) french readme
| * 4d57bfb (sp) spanish readme
|/  
* 5b67eb2 (HEAD -> main) first commit
```

### 如何在 GitButler 中进行操作

在 GitButler 中，这简单得多。您可以简单地编辑两个文件，然后将它们拖放到不同的分支通道中，同时提交和推送它们。

假设我们只是编辑这些文件：

```
❯ echo 'Léeme' > README.es.md
❯ echo 'Lis-moi' > README.fr.md
```

然后我们转到 GitButler UI，看到已自动创建一个匿名虚拟分支，因为它检测到了新的更改。我们可以将其中一个文件拖放到一个新的虚拟分支通道中，然后同时提交和推送两个文件。

注意看这里的摩擦力要少得多。您不必创建新分支或切换目录。这是处理这种类型变更管理的一种更快更顺畅的方式。

这究竟是如何工作的？

本质上，我们执行 `git diff` 查看所有已更改的内容，然后我们有自己的映射来确定哪些更改属于哪个分支。然后当您点击“提交”按钮时，我们对您开始的内容运行一种 `git apply` 类型的应用。

这有点像 `git add -p` 所做的，不过我们可以一遍又一遍地做，保持事物分开。

例如，使用正常的 Git 工具，我们可以在这里添加一个 README 并提交和推送，但是为了第二个文件，我们需要重置然后添加另一个，然后更改活动分支并提交等等。这并不简单。

## 一个工作目录统治它们所有

因此，根据您想要做什么，这两种方法各有优缺点。

由于 GitButler 只有一个单一的工作目录，您*无法*应用具有冲突工作的两个分支。如果每个文件只有一份副本，您就不会有冲突。

这可能被视为优势或局限，但我们发现它相当不错。基本上，你从合并后的产品开始，然后从中提取分支。

例如，假设`french`和`spanish`分支最终被合并。合并的顺序无关紧要，我们知道它们不仅会干净地合并，而且知道它们合并后，我们最终得到的是开始时工作目录的状态。

因此，这就是工作树和虚拟分支以相当不同方式解决同一基本问题的方法。希望这对你有帮助！
