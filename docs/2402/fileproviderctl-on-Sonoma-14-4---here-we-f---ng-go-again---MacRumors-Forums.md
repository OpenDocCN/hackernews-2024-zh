<!--yml

分类: 未分类

日期：2024-05-27 14:35:24

-->

# Sonoma 14.4 上的 `fileproviderctl` - 又是我们又要去搞砸了 | MacRumors 论坛

> 来源：[https://forums.macrumors.com/threads/fileproviderctl-on-sonoma-14-4-here-we-f-ng-go-again.2418353/](https://forums.macrumors.com/threads/fileproviderctl-on-sonoma-14-4-here-we-f-ng-go-again.2418353/)

它们之前没有被弃用过 —— 它们在 macOS 14.3 中仍然存在。

这些命令用于操作使用 FileProvider API 的任何软件 —— 熟知的例子有 Google Drive、OneDrive、Dropbox、iCloud Drive、CloudMounter、Strongsync 等等。已删除的命令在排除这些应用程序的同步问题时非常有用，特别是 `evict` 和 `domain` 子命令。
