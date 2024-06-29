<!--yml

category: 未分类

date: 2024-05-29 12:48:36

-->

# xz 库后门声明 - UnitedBSD

> 来源：[https://www.unitedbsd.com/d/1321-statement-on-backdoor-in-xz-library](https://www.unitedbsd.com/d/1321-statement-on-backdoor-in-xz-library)

最近，[在 xz 压缩库中发现了一个后门](https://www.openwall.com/lists/oss-security/2024/03/29/4)。xz/liblzma 作为 NetBSD 的一部分包含，并用于发布新版本和软件包。

NetBSD 所使用的所有稳定（和不稳定）版本中的 xz 版本均早于后门作者的任何代码更改。因此，NetBSD 是安全的，**未受影响**最近的发现。据信，攻击仅针对 Linux/glibc，但检查此事使我们能够排除作者对库进行任何其他妥协的尝试。

不过，安装在 pkgsrc 中的 xz 受到影响。在 NetBSD 上使用 pkgsrc 中的 xz 是非默认设置，并且需要显式选择加入。大多数 NetBSD 用户不会安装来自 pkgsrc 的 xz，因为基础系统中的版本更受欢迎。然而，在其他平台上使用 pkgsrc 的用户需要采取预防措施。

无论 NetBSD 是否受影响，后门的发现都是一个警钟，我们将在内部进一步讨论如何进行处理。
