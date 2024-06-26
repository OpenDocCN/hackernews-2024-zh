<!--yml

category: 未分类

date: 2024-05-27 15:03:05

-->

# 有用的cat用法

> 来源：[https://two-wrongs.com/useful-uses-of-cat](https://two-wrongs.com/useful-uses-of-cat)

当我编写转换某些文件内容的Shell单行命令时，它们经常看起来像这样

```
cat access.log | head -n 500 | grep mail | perl -e …

```

这是很多人本能地称之为*无用的cat用法*¹，而更深思熟虑的人会称之为对`grep`和`head`的无用用法，因为Perl脚本当然可以做这两件事。因为`head`可以接受文件名作为参数，我们不需要额外的管道和`cat`命令。实际上，几乎所有命令都可以直接接受文件名²。对于那些不能的命令，我们可以使用输入重定向，例如`< access.log command`。只有当我们想要连接文件内容时，我们才真正需要`cat`。

但我仍然有个理由这样做。

我目前正在重读David Parnas的经典论文之一，关于模块化的《为了易于扩展和收缩而设计软件》³；Parnas；ieee软件工程交易；1979年。每个软件工程师都应该阅读这些东西 - 它很精彩。对于本文，我们将专注于一件事：我们都知道代码更改应该是隔离的。例如，我们应该能够通过仅添加代码来添加新功能，而不是进入并更改现有代码。Parnas用一个有趣的方式表达了这一点。

> 我们必须认识到[…]总是可以从程序中删除代码并得到可运行的结果，[而且]任何软件系统都可以扩展。问题在于这些子集和扩展不是我们如果从一开始就着手设计那个产品会设计出来的程序。此外，为了获得产品所需的工作量似乎与变更的性质不成比例。

他对理想设计的理念是，我们可以添加或删除代码，而看起来程序仍然像是为当前正在执行的任务设计的；即，你无法辨别后来添加或删除的东西，一切看起来都像是原始设计的一部分。

Parnas列出了我们在尝试进行更改时经常遇到的四类问题。对于本讨论，第二类问题是相关的。

> 许多程序被构建为组件链，每个组件从前一个组件接收数据，处理数据（并更改格式），然后将数据发送到链中的下一个程序。如果这条链中的一个组件不需要，那么很难删除该代码，因为其前一个组件的输出与其后继组件的输入要求不兼容。必须替换一个仅仅改变格式的程序。
> 
> 一个例子将是一个假设未排序输入的工资程序。系统的一个组件接受未排序的输入并产生按某些键排序的输出。如果公司采用导致排序输入的办公程序，这个处理阶段是不必要的。为了消除该程序，可能需要添加一个程序，将输入格式文件中的数据转移到适合下一阶段的文件格式中。

如果我们回到我们的 shell 单行命令的例子并稍微眯起眼睛，那么字符串 `access.log` 是一个输入格式（描述具有相关内容的文件），而访问日志的内容是一个不同的输入格式。这些是基本上相同事物的两种表示。

如果我们然后消除无用的 `cat` 使用并改为写入

```
head -n 500 access.log | grep mail | perl -e …

```

我们发现 `head` 执行了两个职责：

1.  将字符串 `access.log` 转换为文件的内容；和

1.  提取前 500 条记录的内容。

当我们对我们的 Perl 脚本感到满意时，认为我们可能希望运行它跨整个访问日志，而不仅仅是前 500 条记录并不是不合理的。如果我们只删除 `head` 处理步骤，我们将没有一个将字符串 `access.log` 转换为访问日志内容的步骤。我们可以将这个责任移到 `grep` 调用中，但这意味着我们必须改变一些现有的组件以删除另一个 - 这样不行！

自然的解决方案是无用的 `cat`。通过单独的处理步骤，将文件名转换为文件内容，我们可以删除任何中间处理步骤，并且仍然保留一个有效的流水线。⁴ 我们也可以将源数据更改为例如 `zcat` 或 `curl` 命令。我经常尝试使用 `cat canned_response.json`，然后一旦对这个单行命令感到满意，就切换到 `curl`。换句话说，过程到过程的管道与输入重定向相比更灵活和解耦，后者意味着特定类型的数据源。人们可以抱怨它，但我将继续编写模块化的代码。即使只是 shell 的一行命令。
