<!--yml

category: 未分类

date: 2024-05-29 12:12:23

-->

# Linux 6.9将放弃旧的NTFS文件系统驱动程序 - Phoronix

> 来源：[https://www.phoronix.com/news/Linux-6.9-Dropping-Old-NTFS](https://www.phoronix.com/news/Linux-6.9-Dropping-Old-NTFS)

两年前与Linux 5.15合并，使用

[由Paragon Software开发的“NTFS3”驱动程序](https://www.phoronix.com/news/NTFS3-For-Linux-5.15)

具有工作的读写支持和其他改进，以支持微软的NTFS文件系统驱动程序。这个驱动程序是对主线内核中原始NTFS只读驱动程序的重大改进，比使用NTFS-3G FUSE文件系统驱动程序更快。现在，足够的时间过去了，NTFS3驱动程序运行良好，旧的NTFS驱动程序已经准备删除。

在Linux 6.9合并窗口开放之前，这个周末或下一个周末，取决于v6.8周期的进展如何，Christian Brauner提交了一个"

[vfs ntfs](https://lore.kernel.org/lkml/20240308-vfs-ntfs-ede727d2a142@brauner/)

"请求，去除旧的NTFS驱动程序。他主张将其移除为：

> "这删除了旧的ntfs驱动程序。新的ntfs3驱动程序是一个完整的替代品，两年前合并。我们经历了各种用户空间，他们要么使用ntfs3，要么使用ntfs的fuse版本，因此两者都不构建ntfs或ntfs3。我认为这是一个明确的迹象，表明我们应该冒险删除传统的ntfs驱动程序。
> 
> ...
> 
> 除了各种奇怪的修复之外，它没有得到维护。最坏的情况是，如果有人确实有有效的依赖关系，我们可能不得不重新引入它。但是值得一试，看看我们是否可以删除它。"

删除这个传统的NTFS内核驱动程序可以减少Linux源代码树的29,303行。
