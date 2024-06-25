<!--yml

category: 未分类

date: 2024-05-27 14:48:34

-->

# 微软在Windows 10上修复BitLocker漏洞的更新失败 • The Register

> 来源：[https://www.theregister.com/2024/01/12/microsoft_update_for_bitlocker_vuln/](https://www.theregister.com/2024/01/12/microsoft_update_for_bitlocker_vuln/)

更新的微软本周再次向外发布了另一个有问题的补丁，形式为KB5034441。然而，与处理BitLocker漏洞不同，该补丁无法安装给一些用户。

[补丁](https://support.microsoft.com/en-gb/topic/kb5034441-windows-recovery-environment-update-for-windows-10-version-21h2-and-22h2-january-9-2024-62c04204-aaa5-4fee-a02a-2fdea17075a8)，发布于1月9日，旨在解决一个漏洞，该漏洞允许攻击者通过使用Windows恢复环境（WinRE）来绕过BitLocker加密。它适用于Windows 10 21H2和22H2，但似乎已经受到了微软传奇般的质量控制的影响。

安装更新时，一些用户发现自己面临`0x80070643`错误，一个通用的失败消息。不幸的是，根据微软的说法，"由于错误代码处理程序中的错误"，这可能不是正确的错误。实际错误可能与恢复分区不足有关："Windows恢复环境维护失败。(CBS_E_INSUFFICIENT_DISK_SPACE)。"

有很大的可能性，标准Windows 10安装的计算机可能没有足够大的恢复分区来处理更新，微软在[KB5028997](https://support.microsoft.com/en-gb/topic/kb5028997-instructions-to-manually-resize-your-partition-to-install-the-winre-update-400faa27-9343-461c-ada9-24c8229763bf)中默默承认了这一点，并提供了如何调整分区大小的说明。

这些说明并非适合心脏脆弱的人。第一个链接要求用户以管理员权限打开命令提示符窗口。从那里开始，用户被引导执行命令以禁用WinRE，然后删除并重新创建恢复分区。这是一个风险很大的过程，对于不谨慎的人来说有很多潜在的错误可能。

微软要求遇到磁盘空间问题的用户尝试手动调整分区大小以解决问题，这可能会被错误的安装失败错误`0x80070643`所掩盖。

从[社交媒体](https://www.reddit.com/r/Windows10/comments/192l9kj/comment/kh32u6f/)的一瞥中可以看出，这个问题是普遍存在的，用户不愿意应用微软的解决方法。可以理解的是，有些人称其为"太技术性和可怕"，而另一个则指出："这是微软需要自己纠正的问题。"

另一个人说："我们不应该是在纠正微软的错误。现在暂停更新，微软将在未来推送修复。"

*The Register* 联系了微软，想知道这家软件巨头是否打算更新其补丁，或者在失败后不需要从命令行中切割文件系统。

与此同时，我们不得不同意一位用户的说法：“微软在推送更新之前需要更好地检查它们。”

的确。显示一个无用的错误消息，然后要求用户深入了解命令行来修复问题。这是什么？Linux？®

### 更新于1月16日添加：

微软发言人告诉我们，公司已经更新了有关此事的公共文档。尽管指导没有变化，微软表示：“我们正在解决问题，并将在即将发布的更新中提供更新。”

那就好了。
