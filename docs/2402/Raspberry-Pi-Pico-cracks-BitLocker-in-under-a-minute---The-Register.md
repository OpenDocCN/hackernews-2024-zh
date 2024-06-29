<!--yml

类别：未分类

日期：2024-05-27 14:40:32

-->

# 树莓派Pico在不到一分钟内破解BitLocker • The Register

> 来源：[https://www.theregister.com/2024/02/07/breaking_bitlocker_pi_pico/](https://www.theregister.com/2024/02/07/breaking_bitlocker_pi_pico/)

我们非常熟悉许多项目中使用树莓派硬件的情况，从使旧计算机重获新生，到运行零售商钟爱的动画显示。但是破解BitLocker？我们怀疑公司不会对这个特定应用太过自豪。

这一技术在周末的一个[YouTube视频](https://youtu.be/wTl4vEednkQ)中有详细记录，展示了如何利用树莓派Pico在不到一分钟的时间内获取对使用BitLocker加密的设备的访问权限，前提是你能够物理接触到该设备。

[YouTube视频](https://youtu.be/wTl4vEednkQ)

在视频中使用了联想笔记本，用户stacksmashing发布了视频，尽管其他硬件也会受到影响。该技术还依赖于一个独立于CPU的可信平台模块（TPM）。在许多情况下，这两者会合并在一起，此时所展示的技术则无法使用。

然而，如果你获得了一个类似受BitLocker保护的易受攻击的设备，获取对加密存储的访问似乎非常简单。问题的关键在于截取从TPM到CPU传递的设备密钥。这个密钥可帮助你毫不费力地获取。

这台特定的笔记本有可以与自定义连接器一起使用的连接，以访问芯片间的信号。加上在树莓派Pico上运行的分析仪，不到10美元的组件成本，你就可以获取到笔记本硬件的主密钥。

微软长期以来一直[承认](https://learn.microsoft.com/en-gb/windows/security/operating-system-security/data-protection/bitlocker/countermeasures)此类攻击是可能的，尽管它将其描述为“有针对性的攻击，需要大量时间；攻击者打开机箱，进行焊接，并使用复杂的硬件或软件。”

在例子中不到一分钟的时间里，我们对“大量时间”的说法表示异议，而树莓派Pico无疑是以不到10美元的价格令人印象深刻，硬件支出既不昂贵也不特定。

如果你的硬件存在漏洞，可以通过使用PIN码来进行缓解。

这足以让管理员匆忙地检查他们的库存清单，以确保他们本来认为已经安全加密的硬件。

正如有人评论的那样：“恭喜你！你找到了FBI的后门。” ®
