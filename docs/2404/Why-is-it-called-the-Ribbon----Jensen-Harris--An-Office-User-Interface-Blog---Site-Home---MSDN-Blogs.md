<!--yml

category: 未分类

date: 2024-05-27 13:21:18

-->

# 为什么称其为 Ribbon？ - Jensen Harris：一个办公用户界面博客 - 站点主页 - MSDN 博客

> 来源：[https://web.archive.org/web/20160203163107/http://blogs.msdn.com/b/jensenh/archive/2005/10/07/478214.aspx](https://web.archive.org/web/20160203163107/http://blogs.msdn.com/b/jensenh/archive/2005/10/07/478214.aspx)

因为你永远无法预测最终会采用什么名称。

当我的团队在设计[Outlook 2003的UI重新设计](https://web.archive.org/web/20160203163107/http://www.microsoft.com/office/outlook/prodinfo/overview.mspx)时，我们设计的一个区域后来被市场称为“[导航窗格](https://web.archive.org/web/20160203163107/http://office.microsoft.com/en-us/assistance/HP010378651033.aspx)”。但官方的功能命名通常在产品周期的后期（通常是 Beta 1 之后）才确定，因此总是有一些内部名称会在正式名称开始使用后长期保留。

当我们为 Outlook 2003 的功能列表工作时，最终成为导航窗格的那个项目被称为“组合 Outlook 酒吧和文件夹列表”。幸运的是，我的团队成员有远见，意识到这是一个不好的名称，必须用两年的时间去忍受。他建议我们将其命名为“WunderBar”。这个名字很棒，因为它既流畅易说，又具有积极的内涵（奇迹），而且因为我们团队中有很多德语母语者，所以也很有趣。后来，有人发现了一个[WunderBar 糖果棒](https://web.archive.org/web/20160203163107/http://www.mikescandywrappers.com/wunderbar_1204.html)，而且还发现自助餐厅的番茄酱分配器被称为“Wunder-Bar”。这使事情变得更好。即使今天，许多人内部仍然用其原始名称称呼导航窗格，而所有的代码仍然如此。

那么，“[Ribbon](https://web.archive.org/web/20160203163107/http://blogs.msdn.com/jensenh/archive/2005/09/14/467126.aspx)”这个名字是从哪里来的呢？

2003 年秋天，我们努力制作了许多不同的原型，试图确定新 UI 的方向。我们画了很多图，花了很多时间辩论不同方向的优缺点。

我们几个人在办公室里集思广益，我提出了一个“缎带”命令的想法。想象一下像[中世纪卷轴](https://web.archive.org/web/20160203163107/http://courseweb.stthomas.edu/medieval/images/waldensianscroll.gif)，一长条纸绕在两个纺锤上，你可以通过转动其中一个纺锤在纸上前后移动。我想在计算机术语中，它看起来像是一个极长的细条命令，横向排列的 Office 12 缎带选项卡内容，然后加入超快速的双向滚动。

毫无疑问，那是个愚蠢的主意（我甚至不确定我们是否费心去画它），但它在有机地变形成一组名为“缎带”的图片，我们将同样的想法分割成了选项卡。到我们完善这个想法时，我们称之为缎带 — 即使这个名字不再有太多意义了。

我想，那应该是认识到我们需要一个更吸引人的名字（比如 WunderBar）的最佳时机，但遗憾的是，我们没有，在几天后称它为其他名字只会让每个人都困惑。此外，缎带只是许多方案之一，当时还不清楚我们要构建的是哪个方案。

因此，完全偶然间，一个描述完全不同 UI 机制的名称，至今仍是我们使用的代码名。
