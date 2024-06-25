<!--yml

类别：未分类

日期：2024-05-27 14:38:17

-->

# 离线使用 Git

> 来源：[https://www.gibbard.me/using_git_offline/](https://www.gibbard.me/using_git_offline/)

一些公司使用隔离网络甚至完全缺乏网络作为安全措施，以防止未经授权的访问。在这些系统上工作可能会很艰难，但仍然有可能，并且也许更重要的是，使用像 Git 这样的正确版本控制工具。

按设计，Git 完全可以在没有远程仓库的情况下正常工作。你可以像平常一样分支、暂存和提交文件。

```
`mkdir testRepo
cd testRepo
git init
touch test.txt
git add --all
git commit -m "Initial Commit"` 
```

如果只使用一台机器进行开发，那么这种方法非常有效，但实际情况往往并非如此。

## 使用多台机器工作 — 使用 USB 记忆棒/硬盘

当安全策略允许在记忆棒或便携式硬盘上进行读/写访问时，可以在此设备上创建远程仓库。

在一台开发机器上挂载记忆棒。

```
`cd /path/to/memory/stick
mkdir repoName.git
cd repoName.git
git init --bare` 
```

转到要共享的仓库，将记忆棒上的远程仓库添加为远程仓库，并推送更改。

```
`cd /path/to/local/repo/
git remote add origin /path/to/memory/stick/repoName.git
git push origin master` 
```

注：远程仓库可以取任何名称。它不必被称为“origin”。

卸载记忆棒，然后将其挂载到另一台开发机器上。

如果开发机器上还没有仓库的副本，那么可以使用 git clone。

```
`git clone /path/to/memory/stick/repoName.git` 
```

如果机器上已经有了仓库的副本，请将记忆棒添加为远程，并获取/拉取更改。

```
`cd /path/to/local/repo/
git remote add origin /path/to/memory/stick/repoName.git
git pull origin` 
```

从现在开始正常使用 Git，但确保每次执行 git pull、fetch 或 push 时，记忆棒都已挂载在机器上。

确保记忆棒是备份例程的一部分。

## 使用多台机器工作 — 使用 CD/DVD

在受限的开发环境中，可能会阻止记忆棒。仍然可以使用 Git，但可能会更加不便。

Git 会愉快地从一个本地仓库的副本中获取更改到另一个本地仓库的副本中。然后一个选择是简单地通过 CD 或其他媒体复制包含本地 Git 仓库的目录到另一台计算机上，并在两台计算机上像平常一样进行更改和提交。当你想要合并更改时，选择一台机器执行合并并将另一个仓库复制到这台机器上。要将所有更改拉到当前分支，请使用：

```
`git pull /path/to/other/repo` 
```

或者，您可以获取更改并创建一个新分支来存储它们：

```
`git fetch /path/to/other/repo
git checkout -b new_branch FETCH_HEAD` 
```

在此时创建一个包含合并的新仓库副本，并将其移动到其他机器上。将最新更改拉到其他仓库中，或者如果需要的话，简单地用新副本替换整个仓库。

显然，这远非最佳方案。复制整个仓库目录将包含个人设置和在 `.gitignore` 文件中排除的文件。为了缓解这个问题，可以使用 Git 克隆来复制仓库，而不仅仅是复制它，但更好的选择是使用 git bundle。

### Git bundle

Git bundle 允许将仓库的部分或全部压缩成一个 git 能够克隆和从中获取的单个文件中。

工作流程与以前非常相似，但是不再复制整个仓库目录，而是创建git捆绑包。在第一台机器上使用以下命令创建一个捆绑包：

```
`git bundle create repoName.bundle --all` 
```

`-- all`选项捆绑整个仓库，包括所有分支和标签。可以使用`-b branchName`或`-t tagName`选择特定分支或标签。

将repoName.bundle文件复制到另一台计算机。要克隆仓库，只需使用：

```
`git clone repoName.bundle` 
```

可以在任何计算机上进行更改和提交，然后像以前一样选择一个机器来执行合并。在不合并的机器上，确保所有更改都已提交，并使用以下命令创建一个捆绑包：

```
`git bundle create repoName.bundle --all` 
```

对于更大的仓库，最好只捆绑仓库的一部分，以避免传输比需要的更多的数据。例如，只包括主分支上最后5个提交的话，可以使用：

```
`git bundle create repoName.bundle -5 master` 
```

非常重要的是，在捆绑包的提交与将进行合并的仓库的提交之间没有间隙，否则过程将失败。

将捆绑文件复制到将进行合并的计算机上，并使用以下命令拉取更改：

```
`git pull /path/to/repoName.bundle` 
```

一旦合并/重新设置完成，使用以下命令创建另一个捆绑包：

```
`git bundle create repoName.bundle --all` 
```

在上述命令中，`--all`可以替换为所需的仓库/提交的子集。

将捆绑文件移动到另一台机器，并在那里更新更改：

```
`git pull /path/to/repoName.bundle` 
```

### 创建一个本地的远程仓库

捆绑解决了在没有网络的情况下同步Git仓库的问题，但我们仍然面临着多台计算机都很可能略有不同步的问题。如果新的开发人员加入团队，他们应该从哪里复制仓库？最好的选择是选择一台开发机器作为“服务器”。可以在这台开发机器上创建一个裸的Git仓库，另外还可以在开发人员实际工作的本地克隆仓库中创建一个本地仓库。

```
`cd /path/to/store/main/repo
mkdir remoteRepoName.git
cd remoteRepoName.git
git init --bare` 
```

然后导航到本地git仓库或创建一个新的仓库，将remoteRepoName.git仓库添加为远程仓库。

```
`cd /path/to/local/repo/
git remote add origin /path/to/store/main/repo/remoteRepoName.git
git push origin branchName` 
```

然后可以在本地仓库进行更改，或者从其他开发机器上创建的捆绑包中拉取。每当有更改时，都可以使用以下命令将其推送到远程：

```
`git push origin branchName` 
```

## 摘要

Git的分布式特性使其可以在没有中央服务器的情况下很好地工作。虽然所提供的选项永远不会像只需推送到github那样方便，但它们肯定比`main_v1_final version_with_bobs_extra_patch finalfinal_version`这样的替代方案好得多。
