<!--yml

分类：未分类

日期：2024-05-27 14:39:23

-->

# 条件 Git 配置 - Scott's Weblog - 一个专注于云计算、Kubernetes、Linux、容器和网络的 IT 专业人士的博客

> 来源：[`blog.scottlowe.org/2023/12/15/conditional-git-configuration/`](https://blog.scottlowe.org/2023/12/15/conditional-git-configuration/)

# 条件 Git 配置

* 发布于 2023 年 12 月 15 日 · * 存档在 解释 中 · * 304 字（估计阅读时间 2 分钟）*** ***在之前关于 自动转换 Git URL 的文章的基础上，我又回来了，这次是关于 [Git](https://www.git-scm.com) 的另一篇文章——条件性地包含 Git 配置文件的功能（可能很强大）。这意味着你可以配置 Git 根据某些条件不同地进行配置（和行为），只需包含或不包含 Git 配置文件。让我们看一个直接从我的工作流程中取出的相当简单的例子。

这是我的系统范围的 Git 配置的配置段落：

```
[includeIf "gitdir:~/Work/Code/Repos/"]  path = ~/Work/Code/Repos/.gitconfig 
```

关键在于 `includeIf` 关键字。在这种情况下，如果 Git 仓库的位置与 `gitdir` 后面的路径规范匹配，则 Git 将包含指定的配置文件。基本上，这意味着 *所有* 在 `~/Work/Code/Repos` 下的仓库都会触发额外配置文件的包含。

这是额外的配置文件：

```
[user]  email = name@work-domain.com name = Scott Lowe [commit]  gpgsign = false 
```

只要我把所有与工作相关的仓库分组在指定的目录路径中，这些值就会覆盖系统范围的值。这意味着我可以将我的工作邮箱地址指定为与提交到工作相关的仓库相关联的邮箱地址，而所有其他邮箱地址则使用不同的邮箱地址。这个配置还允许我禁用对工作相关的仓库进行 GPG 签名（即，在指定路径中的仓库），因为我没有将 GPG 密钥与我的工作邮箱地址关联。

你能用每个仓库的配置设置来完成这个吗？ *当然可以。* 这种配置机制允许你根据文件系统位置将配置设置应用于一组仓库，而不是每个仓库都要做同样的事情。

希望这些信息对你有用。如果你有任何问题或任何反馈，请随时联系我——[在 Fediverse 上](https://fosstodon.org/@scottslowe)、[在 Twitter 上](https://twitter.com/scott_lowe)，或在任何各种 Slack 社区中！

### 元数据和导航

* Git * CLI

*上一篇文章：[自动转换 Git URL](https://blog.scottlowe.org/2023/12/11/automatically-transforming-git-urls/)

*下一篇文章：[使用 Direnv 动态启用 Azure CLI](https://blog.scottlowe.org/2023/12/18/dynamically-enabling-azure-cli-with-direnv/)**** ****欢迎分享本文！

[](https://www.facebook.com/sharer/sharer.php?u=https%3a%2f%2fblog.scottlowe.org%2f2023%2f12%2f15%2fconditional-git-configuration%2f "在 Facebook 上分享")*[](https://plus.google.com/share?url=https%3a%2f%2fblog.scottlowe.org%2f2023%2f12%2f15%2fconditional-git-configuration%2f "在 Google Plus 上分享")********
