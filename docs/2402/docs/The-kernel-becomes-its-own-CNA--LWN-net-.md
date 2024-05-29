<!--yml
category: 未分类
date: 2024-05-27 14:51:40
-->

# The kernel becomes its own CNA [LWN.net]

> 来源：[https://lwn.net/Articles/961961/](https://lwn.net/Articles/961961/)

# The kernel becomes its own CNA

[Posted February 13, 2024 by corbet]

Greg Kroah-Hartman has

[announced](http://www.kroah.com/log/blog/2024/02/13/linux-is-a-cna/)

that the kernel project has been accepted as a CVE numbering authority (CNA). The way that CVE numbers will be handled by the kernel is described in

[this documentation patch](/ml/linux-kernel/2024021314-unwelcome-shrill-690e@gregkh/)

:

> As part of the normal stable release process, kernel changes that are potentially security issues are identified by the developers responsible for CVE number assignments and have CVE numbers automatically assigned to them. These assignments are published on the linux-cve mailing list as announcements on a frequent basis.
> 
> Note, due to the layer at which the Linux kernel is in a system, almost any bug might be exploitable to compromise the security of the kernel, but the possibility of exploitation is often not evident when the bug is fixed. Because of this, the CVE assignment team are overly cautious and assign CVE numbers to any bugfix that they identify. This explains the seemingly large number of CVEs that are issued by the Linux kernel team.

* * *

(

[Log in](https://lwn.net/Login/?target=/Articles/961961/)

to post comments)