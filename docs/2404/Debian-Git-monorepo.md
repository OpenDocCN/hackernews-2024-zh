<!--yml

类别：未分类

日期：2024-05-27 12:50:49

-->

# Debian Git 单一代码库

> 来源：[https://blog.liw.fi/posts/2024/monorepo/](https://blog.liw.fi/posts/2024/monorepo/)

[Lars Wirzenius Consulting Ltd](https://liw.fi/) 创建了一个包含 Debian 12（bookworm）源代码的 Git 仓库，其中包括 `main` 组件。这是一个仓库包含所有源代码，只有一个提交。这是一个[单一代码库](https://en.wikipedia.org/wiki/Monorepo)。

要克隆此仓库（但在运行之前请阅读以下内容）：

```
git clone https://monorepo.liw.fi/git/debian
```

请善待概念验证 Git 服务器，如果已经有 42 个或更多人在克隆，请避免再次克隆。检出将需要几个小时，并使用大约 500 GiB 的磁盘空间，有大约 1500 万个文件不包括 `.git`。

这是一个只读仓库，在实际用于协作之前需要移至 Debian 基础设施。

## 动机

单一代码库将极大地简化和改进 Debian 的开发流程：

+   更简单的协作：每个软件包都使用相同的流程和工具，帮助他人的软件包将比以往任何时候都更容易。

+   转换变得更容易：所有更改都将在一个分支中准备并原子地合并，而不是分别上传每个源码包。

+   一般性的整体发行变更：所有软件包的所有源代码放在一个树、一个仓库中，这样可以对 Debian 进行影响许多软件包的变更。例如，以前 Debian 花了七年时间将 `/usr/doc` 迁移到 `/usr/share/doc`，现在可以通过一次提交完成。

+   更容易查看变更内容：特别是发布说明维护者将特别欣赏这一点。未来，Debian 发布说明将变得极其详细。它们将不再仅仅说“修复错误和改进”。

+   更好的非自由管理：如果在主要包中发现包含非自由代码，可以使用 `git filter-branch` 来移除。

+   更好的质量控制：只有 CI 测试通过后，才会进行合并，使用来自 Rust 语言开发者的 [`bors`](https://bors.rust-lang.org/) 工具。目前，软件包仅在上传到 Debian 存档后才进行测试。这一变更将减少在生产环境中运行 Debian 不稳定分支时遇到的问题。不久将不会有不稳定版本中的 bug。

+   更快的开发速度：因为 Debian 开发者将不再需要花时间修复不稳定版中的错误，并维护 Debian 缺陷跟踪器，他们将有更多时间和精力更新软件包到更新版本。这反过来将激励 Debian 的大型企业用户更多地资助 Debian 开发，因为他们对变更速度缓慢的主要抱怨最终将得到解决。

# “.dsc”的未来

将不再需要传统的 `.dsc` 源码包格式。相反，可以发布单一代码库的副本。

# 资源使用

单一仓库并不小。然而，考虑到Debian对其业务的重要性，像苹果、微软、亚马逊和谷歌这样的公司已经迫不及待地愿意捐赠硬件和云资源，以支持单一仓库的使用。一旦细节确定下来，预计会在`debian-devel-announce`上发布公告。

# 实现

程序[unpack-debian-sources](https://app.radicle.xyz/nodes/radicle.liw.fi/rad:zgYpM7b29D6wTMjEUxxzBjcF9EvK)在[Radicle](https://radicle.xyz/)网络上作为仓库ID `rad:zgYpM7b29D6wTMjEUxxzBjcF9EvK` 是可用的。在我家里的电脑上运行它大约花了七个小时。之后，我用以下命令创建了这个仓库：

```
git init .
git add -v .
git commit -m "Debian 12 (bookworm), main"
```

`git add`花了大约3.5小时，`git commit`又花了2.5小时。这需要16 GiB的RAM才能运行。因为我已经完成了这一步，其他人不需要重复。其他人可以直接从我的服务器或彼此克隆。提交是:

```
commit 8ae3129b9f222534d6ed20dd0489fb8f2f1d1a97 (HEAD -> main, origin/main, origin/HEAD)
Author: Lars Wirzenius <liw@liw.fi>
Date:   Tue Mar 19 11:05:57 2024 +0000

    Debian 12 main
```

祝玩得开心！

# 免责声明

这篇博客文章是一个愚人节笑话。Debian不会迁移到单一仓库。我从2018年起就不再是Debian的成员了。我不代表Debian发言。如果Debian放弃了目前以开发者和软件包为中心的开发模式，我会非常惊讶。我甚至觉得Debian有单一仓库不合理。尽管如此，这个单一仓库是真实存在的，尽管它将在四月底被删除。

所有这一切背后真正的动机是，我有时候喜欢对我的工具狠一点。这一次，我对Git狠了：它能处理这么大的仓库吗？在2009年它做不到。到了2024年它可以了。
