<!--yml

category: 未分类

日期：2024-05-27 14:36:55

-->

# 远程访问巨头AnyDesk在遭受黑客攻击后重置密码并吊销证书 | TechCrunch

> 来源：[https://techcrunch.com/2024/02/05/remote-access-giant-anydesk-resets-passwords-and-revokes-certificates-after-hack/](https://techcrunch.com/2024/02/05/remote-access-giant-anydesk-resets-passwords-and-revokes-certificates-after-hack/)

远程桌面软件提供商AnyDesk确认，上周五晚上发生的一次网络攻击使黑客能够访问该公司的生产系统，导致公司进入了近一个星期的封锁状态。

AnyDesk的软件被数百万IT专业人士使用，快速远程连接到其客户设备，通常用于解决技术问题。在[其网站](https://anydesk.com/en-gb/case-studies)上，AnyDesk声称拥有超过17万客户，包括Comcast、LG、三星和泰雷兹。

这款软件也是威胁行为者和勒索软件团伙的热门工具，他们长期以来一直使用这款软件来获取和维持对受害者计算机和数据的访问权限。美国网络安全局CISA在一月份表示，[黑客利用合法远程桌面软件攻击联邦机构](https://techcrunch.com/2023/01/26/us-federal-agencies-hacked-remote-access-tools/)，其中包括AnyDesk。

当AnyDesk宣布[已更换其代码签名证书](https://anydesk.com/en/changelog/windows)后，关于疑似泄漏的消息开始在上周一传播开来。在数日的停机后，AnyDesk在上周五晚间通过一份声明[确认公司已发现生产系统受到了入侵的证据](https://anydesk.com/en/public-statement)。

AnyDesk称，作为其事件响应的一部分，公司吊销了所有安全相关的证书，必要时进行了系统的修复或替换，并使AnyDesk客户网站的所有密码失效。

“我们将很快吊销我们二进制文件的先前代码签名证书，并已经开始用新的证书替换它，”该公司在周五补充道。

AnyDesk表示，此事件与勒索软件无关，但未透露网络攻击的具体性质。

AnyDesk发言人Matthew Caldwell未回应TechCrunch的电子邮件。与AnyDesk合作以纠正网络攻击的CrowdStrike在周一联系时拒绝回答TechCrunch的问题。

AnyDesk没有回答有关是否有任何客户数据被访问的问题，尽管公司在声明中表示，“没有证据表明任何最终用户系统受到了影响。”

“我们可以确认，目前情况已经得到控制，使用AnyDesk是安全的，”AnyDesk说。“请确保您使用的是最新版本，并带有新的代码签名证书。”

AnyDesk因其迄今为止处理网络攻击的方式而受到批评。正如[德国博主Günter Born最初报道的那样](https://www.borncity.com/blog/2024/02/01/anydesk-und-die-stoerungen-es-ist-womoeglich-was-im-busch/)，AnyDesk最初声称从1月29日开始的四天中断（期间公司阻止用户登录）是“维护”。资深事件响应者Jake Williams在[X上的一篇文章中](https://twitter.com/MalwareJake/status/1753550354958967113)指责AnyDesk在周末前向客户披露网络攻击是“公关行动”。

安全研究人员称，黑客正在向据称受到攻击影响的AnyDesk账户出售访问权限，这些账户详细信息可能源自先前在用户计算机上存在的窃取密码的恶意软件感染。

* * *

*你是否有关于这起事件的更多信息？您可以通过Signal安全地联系Carly Page，电话为+441536 853968，或者通过[电子邮件](mailto:carly.page@techcrunch.com)联系她。您也可以通过[SecureDrop联系TechCrunch](https://techcrunch.com/tips)。*
