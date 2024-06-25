<!--yml

分类：未分类

日期：2024-05-27 14:23:40

-->

# PyPy 已经转移到了 Git、GitHub | PyPy

> 来源：[https://www.pypy.org/posts/2023/12/pypy-moved-to-git-github.html](https://www.pypy.org/posts/2023/12/pypy-moved-to-git-github.html)

PyPy 已将其主要仓库和问题跟踪器从 [https://foss.heptapod.net/pypy/pypy](https://foss.heptapod.net/pypy/pypy) 迁移到 [https://github.com/pypy/pypy](https://github.com/pypy/pypy)。显然，这意味着开发现在将在 Git 而不是 Mercurial 中跟踪。

### 动机

我们仍然认为 Mercurial 是一个更好的版本控制系统。命名分支模型和用户界面都更好。但是

+   foss.heptapod.net 在 google/bing/duckduckgo 搜索中的索引不好，所以人们发现在项目中搜索问题更困难。

+   自从 Heptapod 加强了垃圾邮件控制以来，我们收到的报告说用户只是为了发现它们被标记为垃圾邮件而创建问题。

+   开源已经成为 GitHub 的代名词，而我们太小了以至于无法改变这一点。

+   目前的大部分开发都是为了解决问题而进行的。如果所有代码都在同一个平台上，跟踪交错的问题会更容易。

+   [FAQ](https://doc.pypy.org/en/latest/faq.html#why-doesn-t-pypy-use-git-and-move-to-github) 提出了两个反对迁移的论点。[Github notes](https://git-scm.com/docs/git-notes) 解决了第一点的很多问题：发现提交来源的困难，尽管不是完全解决。但主要问题在于第二点，事实证明**不**迁移到 GitHub 是对贡献和问题报告的一种阻碍。

+   希望继续使用 Mercurial 的人可以使用下面相同的方法将代码推送到 GitHub。

+   GitHub 比 foss.heptapod.net 更丰富资源。我们可以添加 CI 任务来替换部分老化的 [buildbot 基础设施](https://buildbot.pypy.org)。

### 方法

迁移需要两部分：迁移代码，然后迁移问题和合并请求。

#### Code migration 1: 代码和笔记

我使用了 [git-remote-hg 的分支](https://github.com/mnauw/git-remote-hg) 来创建一个包含所有变更集的本地 Git 仓库。然后我想给每个提交添加一个 Git 笔记，标明它来自哪个分支。所以我准备了一个文件，有两列：Git 提交哈希和对应的 Mercurial 分支。Mercurial 可以用两种方式描述每个提交：提交哈希或数值索引。我使用 `hg log` 将索引 `i` 转换为 Mercurial 哈希，然后使用 `git-remote-hg` 中的 `git-hg-helper` 将 Mercurial 哈希转换为 Git 哈希：

```
$(cd pypy-git; git-hg-helper git-rev $(cd ../pypy-hg; hg log -r $i -T"{node}\n"))

```

然后我再次使用 `hg log` 来打印索引 `i` 的 Mercurial 分支：

```
$(cd pypy-hg; hg log -r $i -T'{branch}\n')

```

将这两者结合起来，我可以通过它们的数值索引循环遍历所有的提交，以准备好文件。然后我遍历文件中的每一行，并添加 Git 笔记。由于 `git note add` 命令作用于当前 HEAD，我需要依次检出每个提交，然后添加笔记：

```
git checkout -q <hash> && git notes --ref refs/notes/branch add -m branch:<branch>

```

然后我可以使用 `git push --all` 推送到 GitHub。

#### Code migration 2: 准备分支

PyPy 几乎有 500 个未关闭的分支。 代码迁移创建了所有分支 HEAD，但 `git push --all` 没有将它们推送。 我需要检出它们并逐个推送。 因此，我创建了一个包含所有分支名称的文件

```
cd pypy-hg; hg branches | cut -f1 -d" " > branches.txt

```

然后将每个分支推送到 GitHub 存储库

```
while  read  branch; do git checkout branches/$branch && git push origin branches/$branch; done < branches.txt

```

注意，分支由迁移命名为 `branches/XXX`，而不是 `branch/XXX`。 这会使合并请求迁移混乱，稍后会详细说明。

#### 问题和合并请求迁移

我使用了来自[node-gitlab-2-github](https://github.com/piceaTech/node-gitlab-2-github)的解决方案，几乎完美地运行了。 在**私有存储库**上执行转换非常重要，否则每次成功映射的用户名提及都会通知用户有关转移。 对于像 PyPy 这样有 600 个合并请求和 4000 多个问题的存储库来说，这可能相当恼人。 问题转移没有问题：脚本正确保留了问题编号。 但是脚本不会将 Mercurial 哈希转换为 Git 哈希，因此注释中的裸哈希显示时没有指向提交的链接。 合并请求是更大的问题：

+   一旦合并，Mercurial 命名分支就会“消失”，因此对于已合并分支的合并请求在 Git 中找不到目标分支名称。 转换将创建一个带有标签 `gitlab merge request` 的问题。

+   由 `git-remote-hg` 创建的分支之所以被称为 `branches/XXX`，而不是预期的 `branch/XXX`，原因不明。 这使得合并请求/PR 转换混乱。 对于某些分支（开放的 PR 和主目标分支），我手动创建了额外的不带 `es` 的分支。 最终结果是，开放的合并请求变成了开放的 PR，已合并的合并请求变成了问题，并且已关闭但未合并的合并请求未被迁移。

#### 分层转换

PyPy 已经从 Bitbucket 迁移到了 Heptapod。 许多问题反映了多次转换：它们有着像“最初在 Bitbucket 上创建的 XXX”这样的行，来自第一次转换，以及来自此转换的额外行“在 Heptapod 中”。

### 鸣谢

我们要向[Octobus](https://octobus.net/)团队表示感谢，他们支持 Heptapod。 从 Bitbucket 迁移是相当费力的，自那时起，他们一直慷慨地托管我们的开发。 我们祝愿他们一切顺利，并仍然相信 Mercurial 应该“胜出”。

### 下一步

尽管 GitHub 上的存储库已经上线，但我们仍然有一些需要做的事情：

+   文档需要更新为新存储库，并且需要调整来自 readthedocs 的构建自动化。

+   Wiki 应从 Heptapod 复制过来。

+   buildbot.pypy.org 也应查看新存储库。 我希望代码能够与 Git 存储库交互。

+   speed.pypy.org 跟踪变化，它也需要引用新位置

+   为了在新提交上使用 Git 笔记跟踪分支，我激活了 Julian 的 [github action](https://github.com/Julian/named-branch-action) 来为每个提交添加一个 Git 分支注释。请查看那里的 README 以了解如何使用 Git 笔记的说明。

+   一些合并请求没有迁移。如果有人想要，他们可以在解决分支命名问题后进行迁移。

此外，现在是时候让你们所有人证明这次迁移是值得的：

+   给这个仓库加星星，让其他人知道如何找到它，

+   帮助解决一些未解决的问题或提出新问题，

+   利用更熟悉的工作流程参与到项目中，

+   建议改进迁移的方法：我是否遗漏了什么或者可以做得更好？

### 开发将如何改变？

Heptapod 不允许个人分支，因此我们慷慨地将提交权限授予了主要仓库。此外，我们（嗯，我）一直在使用直接提交到主要工作流。我们现在将采用更有结构的工作流程。请分叉仓库并提交拉取请求以进行任何更改。我们现在可以添加一些预合并 CI 来检查 PR 至少是否通过了第一阶段的转换。活跃的分支将是：

+   `main`: 在 Mercurial 中是 "default"，它是 Python2.7 解释器和 RPython 解释器的基础，

+   `py3.9`: Python3.9 解释器，还包括来自 `main` 的所有 RPython 更改。这与 Mercurial 上的情况完全相同，以及

+   `py3.10`: Python3.10 解释器，还包括来自 `main` 的所有 RPython 更改和来自 `py3.9` 的所有错误修复。这与 Mercurial 上的情况完全相同。

#### 在仓库之间工作

##### 查找提交

如果您想弄清楚 Mercurial 提交如何与 Git 提交相关联，您可以使用 `git-hg-helper`。您在 Git 仓库中运行它。它接受来自一个仓库的完整的长哈希，然后给您另一个仓库的相应哈希：

```
$  git-hg-helper  git-rev  d64027c4c2b903403ceeef2c301f5132454491df
4527e62ad94b0e940a5b0f9f20d29428672f93f7
$  git-hg-helper  hg-rev  4527e62ad94b0e940a5b0f9f20d29428672f93f7
d64027c4c2b903403ceeef2c301f5132454491df

```

##### 查找分支

从 Mercurial 迁移的分支将具有 `branches` 前缀，而不是 `branch`。虽然 GitLab 使用 `branch` 作为其前缀，但 `git-remote-hg` 脚本使用 `branches`。新工作应该在针对 `main`、`py3.9` 或 `py3.10` 的 PR 中进行。

感谢您帮助改进 PyPy。

Matti
