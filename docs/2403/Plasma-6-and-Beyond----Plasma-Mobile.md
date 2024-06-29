<!--yml

类别：未分类

日期：2024-05-29 11:59:31

-->

# Plasma 6及更多！- Plasma Mobile

> 来源：[https://plasma-mobile.org/2024/03/01/plasma-6/](https://plasma-mobile.org/2024/03/01/plasma-6/)

Plasma Mobile团队很高兴地宣布为移动设备发布了Plasma 6的版本！

自上一个主要版本发布以来，我们进行了长达一年的开发和移植工作，包括移动应用程序。本文将概述许多亮点！

## 管理工作

为了迎接此版本的发布，网站进行了更新。特别是，[安装](https://plasma-mobile.org/get)页面已经增强，以更好地概述提供Plasma Mobile的各种发行版的信息。

在Plasma 6开发期间，[postmarketOS](https://postmarketos.org/)慷慨地为我们提供了一个跟踪git存储库的“nightly”软件包存储库。对于开发工作，这被证明是非常重要的。[您可以在这里找到如何使用此存储库的说明](https://wiki.postmarketos.org/wiki/Nightly)。

## Plasma

[Devin撰写了一篇博客文章，涵盖了一些技术细节](https://espi.dev/posts/2023/12/plasma-mobile-towards-6/)，涵盖了将所有shell组件移植到Qt和Plasma 6 API的主要工作方面。还进行了大量的功能性工作。

### 主屏幕

使用Plasma 6，我们再次将Folio主屏幕切换为默认。主屏幕将再次具有可自定义的页面来放置应用程序和小部件，以及一个应用抽屉和搜索功能。Devin花时间从头开始重写它，解决了Plasma 5中存在的一些限制，特别是自定义现在与每个页面绑定，不再因屏幕旋转而引起重新排序问题。

重要功能：

+   可自定义的页面，用于放置应用程序和小部件

+   文件夹

+   拖放定制

+   小部件（仍然存在一些已知问题，请参见下文）

+   应用抽屉（向上滑动）

+   KRunner搜索（向下滑动）

+   屏幕旋转的行列翻转

+   可自定义的行数和列数

+   可自定义的页面过渡效果

+   导入和导出主屏幕布局作为文件

+   等等！

### 互操作性

在Plasma 5中，Plasma Mobile需要在系统上安装几个配置文件，以设置Plasma中所需的一些设置。对于Plasma 6，Devin创建了一个新的设置服务，自动处理这种情况。这允许在系统上同时安装Plasma Desktop，并消除了安装自定义配置的需要。

### 首次设置

Devin添加了一个初始设置窗口，引导用户配置系统的一些基本方面，如Wi-Fi和蜂窝网络设置。

### 认证对话框

Devin致力于移植认证对话框，使其具备移动端的形态。

### 停靠模式

Devin致力于添加“停靠模式”快速设置，启用后可以显示窗口装饰和最小化/最大化/关闭按钮，并停止强制应用窗口全屏显示。

### 导航面板

Devin添加了一个设置，始终显示键盘切换按钮。他还使得在平板电脑上，如果有足够的屏幕空间，面板会放在底部而不是侧面。

### 任务切换器

为了提高代码可维护性，任务切换器已从Plasma Shell移至KWin。

### 振动

在Plasma 5中，默认振动速度被硬编码为PinePhone的马达速度，这导致其他设备的振动效果过于强烈。现在，振动默认为其他设备的可接受水平，并且仍然可以在shell设置模块中配置。

### 手电筒

在Plasma 5中，手电筒快速设置被硬编码为PinePhone。Florian已经做了工作，现在它可以在Plasma 6中适用于所有手机！

### 设置

Devin改进了移动网络设置模块，使交互任务异步执行，不会冻结UI。他还修复了模块首次打开时显示不必要密码提示的问题。

Devin将Wi-Fi设置模块移植到移动端组件，使其与其他模块一致。他还更新了时间设置模块到新的组件，使得设置时间和日期更加直观。他还做了一些工作，确保应用设置时不会冻结UI。

Joshua重构了Push通知KCM的外观和感觉，并在NeoChat中添加了UnifiedPush支持，因此即使在NeoChat关闭时，您也不会错过消息（当然，如果您想收到通知的话！）

Mathis为该应用程序创建了一个新图标。

![](img/1a20cce536f6c3fbdc2070f0d49f0aeb.png)

## 已知问题

遗憾的是，我们团队的资源有限，因此我们仍有几个未能在此版本发布前解决的重要问题：

+   **在Qt6中，触摸屏滚动的惯性比Qt5低得多**，我们无法更改平台默认设置，参见[此处](https://invent.kde.org/teams/plasma-mobile/issues/-/issues/273)，在Qt的上游报告[这里](https://bugreports.qt.io/browse/QTBUG-121500)。

+   **一些桌面小部件每次移动时都会打开弹出窗口**

+   **受限的小部件选择**。目前我们只有Plasma桌面中的现有小部件。

+   **目前已暂时移除屏幕录制快速设置**。需要进一步研究将其移植到新的API。

+   **目前已暂时移除仅手势模式**。我们需要调查如何使KWin的手势基础设施更符合我们的需求。

+   **有时应用窗口会延伸到导航栏下方**。我们需要调查这个与KWin相关的回归问题。

+   **在PinePhone上，任务切换器特别慢**。我们需要调查KWin效果的性能。

我们的[问题跟踪器](https://invent.kde.org/plasma/plasma-mobile/-/wikis/Issue-Tracking)中还有其他问题正在跟踪。

## 应用程序

在将所有应用程序大力迁移到Qt6和KF6的庞大工作背后，做了大量工作！我们要向所有参与其中的人员表示衷心的感谢！

应用程序的许多重要公告都在[Megarelease](https://kde.org/announcements/megarelease/6)帖子中，所以一定要在那里检查！

### Photos (Koko)

尽管应用程序仍然称为Koko，但用户界面文本现在将其称为“Photos”，以便您更容易找到它（Devin Lin，KDE Gear 24.02，[链接](https://invent.kde.org/graphics/koko/-/merge_requests/133)）

应用程序的新图标已经创建，替换了大猩猩（Mathis Brüchert & Devin Lin，KDE Gear 24.02，[链接](https://invent.kde.org/graphics/koko/-/merge_requests/134)）

![](img/ce7c33ecdd4c404530ecd7a0b8144aaa.png)

### Clock

当闹钟或计时器开始响铃时，时钟应用现在会暂停MPRIS媒体源，并在解除闹钟后恢复（之前暂停的）源（Luis Büchi，KDE Gear 24.02，[链接](https://invent.kde.org/utilities/kclock/-/merge_requests/109)）

### Calculator

计算器应用程序新增了一个新的配置页面，允许设置小数位数，角度单位和解析模式（Michael Lang，KDE Gear 24.02，[链接](https://invent.kde.org/utilities/kalk/-/merge_requests/83)）

我们改进了历史视图以允许删除条目，并用滑动视图替换了额外功能的抽屉样式区域（Michael Lang，KDE Gear 24.02，[链接 1](https://invent.kde.org/utilities/kalk/-/merge_requests/78)，[链接 2](https://invent.kde.org/utilities/kalk/-/merge_requests/79)）

添加了许多新功能和生活质量改进，包括在输入长表达式时自动调整字体大小，更好地渲染指数，并增加了更多的实际计算功能：随机数，基数n的根/对数，微分函数，标准偏差等... 哦！我们还将数学引擎切换到了libqalculate！（Michael Lang，KDE Gear 24.02，[链接 1](https://invent.kde.org/utilities/kalk/-/merge_requests/77)，[链接 2](https://invent.kde.org/utilities/kalk/-/merge_requests/75)，[链接 3](https://invent.kde.org/utilities/kalk/-/merge_requests/80)，[链接 4](https://invent.kde.org/utilities/kalk/-/merge_requests/73)）

## Kasts

除了与Qt6和KF6更新相关的一些轻微视觉变化和几个有关图像和播放控制的修复之外，Kasts现在通过不解析未更改的RSS源极大地加快了播客的更新速度（Bart De Vries，KDE Gear 24.02，[链接 1](https://invent.kde.org/multimedia/kasts/-/merge_requests/141)）

现在可以手动选择颜色主题（例如Breeze亮，Breeze暗）（Bart De Vries，KDE Gear 24.02，[链接 2](https://invent.kde.org/multimedia/kasts/-/merge_requests/140)）

现在可以完全禁用对计量连接的网络检查。这在不能可靠报告连接状态的系统上非常有用（Bart De Vries，KDE Gear 24.02，[链接 3](https://invent.kde.org/multimedia/kasts/-/merge_requests/146)）

## Hash-o-Matic

Hash-o-Matic是一个新的应用程序，可以通过使用文件的MD5、SHA-256和SHA-1哈希或它们的PGP签名来验证文件的真实性。

Hash-o-Matic还可以为文件生成哈希并比较两个文件。

## 贡献

您想要帮助开发Plasma Mobile吗？我们是一群业余时间做这个的志愿者，急需新的贡献者，**欢迎初学者加入**！

查看我们的[社区](https://plasma-mobile.org/join)页面以获取联系方式！

我们始终乐意获得更多的帮助之手，不管您想做什么，但我们特别需要以下支持：

即使您没有兼容的手机或平板电脑，您也可以通过桌面帮助我们进行应用程序开发！

当然，除了编码之外，您还可以帮助我们做其他事情！例如，尝试使用Plasma Mobile来帮助我们测试错误并分类报告！查看每个发行版的[设备支持情况](https://www.plasma-mobile.org/get/)，找到适合您手机的版本。

另一个选择是帮助我们再次使这些博客文章更加定期。我们真的不希望再有6个月的沉默，但撰写这些文章需要出乎意料的时间和精力。

如果您有任何进一步的问题，请查阅我们的[文档](https://invent.kde.org/plasma/plasma-mobile/-/wikis/home)，考虑加入我们的[Matrix频道](https://matrix.to/#/#plasmamobile:matrix.org)。告诉我们您想要做什么或者在哪里需要支持以便开始！

我们的[问题跟踪文档](https://invent.kde.org/plasma/plasma-mobile/-/wikis/Issue-Tracking)也提供了报告问题的方法和位置信息。

## 最后一点…

我们想要感谢在KDE全体贡献者中使这一大版本发布成为可能的每一位！没有你们，我们无法完成这一切。谢谢！

![](img/a1cd68ab1db3b952e06d8f812117c646.png)
