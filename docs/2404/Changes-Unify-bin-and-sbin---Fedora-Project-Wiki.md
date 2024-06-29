<!--yml

category: 未分类

date: 2024-05-27 13:11:43

-->

# 更改/合并 bin 和 sbin - Fedora 项目维基

> 来源：[https://fedoraproject.org/wiki/Changes/Unify_bin_and_sbin](https://fedoraproject.org/wiki/Changes/Unify_bin_and_sbin)

# 合并 `/usr/bin` 和 `/usr/sbin`

## 摘要

`/usr/sbin` 目录成为 `bin` 的符号链接，这意味着像 `/usr/bin/foo` 和 `/usr/sbin/foo` 这样的路径指向同一个位置。`/bin` 和 `/sbin` 已经是指向 `usr/bin` 和 `usr/sbin` 的符号链接，因此 `/bin/foo` 和 `/sbin/foo` 实际上也指向同一个位置。`/usr/sbin` 将从默认的 `$PATH` 中移除。类似的更改也应用于将 `/usr/local/sbin` 指向 `bin`，从而使 `/usr/local/bin/foo` 和 `/usr/local/sbin/foo` 指向同一位置。`%_sbindir` 的定义将改为 `%_bindir`，因此包在重新构建后将开始使用新目录而无需进一步操作。维护者*可能*停止使用 `%_sbindir`，但无需这样做。

## 拥有者

## 当前状态

## 详细描述

`/bin` 和 `/sbin` 之间的分割既不实用，也未被使用。最初的分割是将在 `/sbin` 中静态链接的“重要”二进制文件用于紧急和救援操作。显然，我们现在不再进行静态链接。后来，这种分割被重新用于隔离只能由管理员使用的“重要”二进制文件。尽管这在理论上看起来很有吸引力，但在实践中，很难对此类程序进行分类，并且普通用户经常从 `/sbin` 调用程序。大多数需要在*某些*操作时具有根权限的程序在无特权操作时也能使用。甚至在需要特权时，通常也会动态获取特权，例如使用 `polkit`。多年来，默认的 `$PATH` 设置为用户包括这两个目录。随着 systemd 的出现，这变得更加系统化：systemd 为所有用户和服务设置了带有这两个目录的 `$PATH`。因此，一般来说，所有用户和程序都会找到这两组二进制文件。

`/bin` 和 `/sbin` 分割的另一个用途是 `consolehelper`。在这种方法中，用户界面程序（`/bin/foo`）是指向 `/bin/consolehelper` 的符号链接，后者是一个 suid 二进制文件，用于提升特权并调用“真正的” `foo`（`/sbin/foo` 或 `/usr/libexec/foo`）。大多数使用 `consolehelper` 的情况已迁移到 polkit ([https://fedoraproject.org/wiki/Features/UsermodeMigration](https://fedoraproject.org/wiki/Features/UsermodeMigration))，但仍有一些用户在使用 ([https://bugzilla.redhat.com/show_bug.cgi?id=502765](https://bugzilla.redhat.com/show_bug.cgi?id=502765))。使用 `/sbin` 存放需要特权的程序与建议的合并不兼容；这些包需要调整以将需要特权的二进制文件移动到 `/usr/lib` 或 `/usr/libexec`（参见范围部分）。

"由于一般所有用户会话和服务都将这两个目录放在 `$PATH` 中，这种分割实际上并没有被*用*到任何地方。它的主要影响是当人们需要使用绝对路径时和目录猜错的混乱。其他发行版将某些二进制文件放在另一个目录中，因此绝对路径通常不具备可移植性。此外，用户很容易在 `$PATH` 中将 `/sbin` 放在 `/bin` 之前，而管理员则很容易将 `/bin` 放在 `/sbin` 之前，这会导致混乱。如果删除此功能，则系统变得稍微简单，这对于不了解分割历史的新用户尤其有用。"

"多年前，我们合并了 `/bin` 和 `/usr/bin` ([https://fedoraproject.org/wiki/Features/UsrMove](https://fedoraproject.org/wiki/Features/UsrMove))。在某些方面，*那个*分割很类似：它有一个历史的理由，但十多年前已经不再适用，将程序清晰地分类是不可能的，因此启动时需要这两个部分，即使这种分割使系统变得更加复杂且没有多少收益，它仍然被继续保留，因为这样做比移除更容易 ([http://www.freedesktop.org/wiki/Software/systemd/TheCaseForTheUsrMerge](http://www.freedesktop.org/wiki/Software/systemd/TheCaseForTheUsrMerge))。*这个*分割虽然不那么显眼，但同样使系统变得更复杂却没有任何收益，移除它是自然的后续步骤。"

## "反馈"

"[https://lists.fedoraproject.org/archives/list/devel@lists.fedoraproject.org/message/TRQ3I7BDTHG6N7WQEQ7PMK3P6PVWXQMI/](https://lists.fedoraproject.org/archives/list/devel@lists.fedoraproject.org/message/TRQ3I7BDTHG6N7WQEQ7PMK3P6PVWXQMI/)"

"/usr/bin 中的程序在手册的第 1 节中有其文档，而 /usr/sbin 中的程序在第 8 节中有其文档。"

"[https://lists.fedoraproject.org/archives/list/devel@lists.fedoraproject.org/message/NALSOW35LXN6HZPULMQ46WXNGZO4SYK4/](https://lists.fedoraproject.org/archives/list/devel@lists.fedoraproject.org/message/NALSOW35LXN6HZPULMQ46WXNGZO4SYK4/)"

> "手册的各节具有历史意义："
> 
> "[https://en.wikipedia.org/wiki/Man_page#Manual_sections](https://en.wikipedia.org/wiki/Man_page#Manual_sections) 例如，1 = 一般命令，8 = 系统管理"
> 
> "因此，除非工具本身正在改变其目的或者现在放错了位置，手册的节应该保持不变。"

## "对 Fedora 的好处"

+   "包维护者无需考虑是否将程序安装在 `%_bindir` 或 `%_sbindir` 中。"

+   "用户无需考虑程序是否在 `%_bindir` 或 `%_sbindir` 中。"

+   Fedora变得更加与其他发行版兼容（例如，我们有`/sbin/ip`，而Debian有`/bin/ip`，我们有`/bin/chmem`和`/bin/isosize`，而Debian有`/sbin/chmem`和`/sbin/isosize`，我们还有`/sbin/{addpart,delpart,lnstat,nstat,partx,ping,rdma,resizepart,ss,udevadm,update-alternatives}`，而Debian将这些放在`/bin`下等等。）

+   Fedora与Arch更加兼容，后者几年前已进行了合并。

+   `execvp`及相关函数遍历较少的目录。这可能对速度没有影响，但在查看日志或`strace`输出时是一种简化。

## 范围

+   提议所有者：

    +   调整`/usr/lib/rpm/macros`（`rpm`包的一部分）中的`%_sbindir`以评估为`%_bindir`。在大规模重建期间，包将自动更新。

    +   向`filesystem`包添加一个`%filetrigger`，为从`/usr/sbin`中卸载的每个`foo`创建指向`../bin/foo`的符号链接。

    +   向`filesystem`包添加一个`%posttrans`触发器，检查`/usr/sbin`只包含符号链接，并执行`ln -fs bin /usr/sbin`。（这些脚本使得过渡更加顺利。在任何时候，旧路径仍然有效。过渡完成后，我们可以删除这些脚本，并在`filesystem`包中提供`/usr/sbin`的符号链接。）

    +   调整systemd包以使用`-Dsplit-bin=no`进行构建。

    +   为打包指南提交一个拉取请求，停止提及`%{_sbindir}`。宏将保持定义以避免使用它的包的中断。

+   其他开发者：

    +   安装到硬编码的`$DESTDIR/usr/sbin`，但在`%files`中使用`%{_sbindir}`的包将需要进行调整。

    +   使用consolehelper并在两个目录下安装相同名称的包需要调整以使用不同的目录。其中一些包可能会被废除。见下面的列表。

+   在两个目录中使用带有二进制文件的usermode的包：

    +   `anaconda-live`

    +   `beesu`

    +   `chkrootkit`

    +   `hddtemp`

    +   `mate-system-log`

    +   `setuptool`

    +   `subscription-manager`

    +   `system-switch-java`

    +   `xawtv`

+   打包一个在`/usr/sbin`中有符号链接的包需要删除：

    +   opensmtpd

    +   rpcbind

    +   policycoreutils

    +   systemd-udev

+   商标批准：不适用（此更改不需要）

+   与社区倡议的对齐：否

## 升级/兼容性影响

对用户来说，这些变化应该基本上是看不见的。过渡期间，两组路径都应该有效，并且用户应该在`$PATH`中有这两个目录。一旦过渡完成，两组路径仍然有效，但用户将只有`/usr/bin`在`$PATH`中。

## 如何测试

## 用户体验

## 依赖

## 应急计划

+   如果特定软件包的二进制移动导致问题，我们可以创建一个兼容的符号链接，例如使用该软件包中的`%postin`脚本创建`/usr/sbin/foo → ../bin/foo`的符号链接。

+   如果更改普遍引起问题并需要回滚，则需要撤消`rpm`中宏定义的更改并重新构建一些或所有包。

+   应急截止日期：原则上可以随时完成，但需要重建部分或全部受影响的软件包。

+   是否阻塞发布？否。如果更改只完成部分，那么可以接受。

## 文档

待定。

## 发布说明
