<!--yml

分类：未分类

date: 2024-05-27 14:36:08

-->

# OpenBSD KDE Plasma 桌面 · rsadowski.de

> 来源：[https://rsadowski.de/posts/2024-01-09-openbsd-kde/](https://rsadowski.de/posts/2024-01-09-openbsd-kde/)

新年快乐，黑客们！我以我的一个个人里程碑开始了新的2024年。[The KDE Plasma 5.27](https://kde.org/plasma-desktop/) 已经在OpenBSD -current 上可用，并将成为下一个版本7.5的一部分。我相信并不是很多人会理解这项工作有多大。它开始了很多年前，以至于我甚至不知道我已经在这上面工作了多长时间。肯定是几年而不仅仅是1或2年。我记得在加入OpenBSD社区之前，瓦迪姆·朱科夫在Qt/KDE领域做了很多工作。在早期，有人可以交谈，或者只是知道“你并不孤单”。这已经很久没有了。多年来，我一直在做这件事，就像瓦迪姆多年来一直在做这件事一样。这可能是项目的负担，我们是一个极小的OpenBSD内核和端口开发者团队。

应该提到，去年kn@ - Klemens Nanni在很多审查工作中提供了帮助，因此使得我们有了KDE作为软件包的可能。

当我写作时，我开始思考。现在会发生什么？接下来会发生什么？我有计划吗？当然我可以继续维护整个东西。绝对有趣的是KDE从Qt5切换到Qt6。这对于OpenBSD来说不应该是一个大挑战。所有依赖项，如所有Qt6模块，都已经移植过来了。所以这只是一项艰苦工作的任务。

这里有一些关于我们可以做什么的想法。

或许在这样一个重大成就之后，一个反思的时期开始了。让我们拭目以待2024年会带来什么。

OpenBSD KDE Plasma 5.27 截图：

你可以非常容易地在OpenBSD上安装KDE Plasma和KDE Gear：

```
# KDE Gear $ pkg_add kde # KDE Plasma $ pkg_add kde-plasma # KDE Plasma extras like power management $ pkg_add kde-plasma-extra 
```

特别感谢所有在[GitHup](https://github.com/sponsors/sizeofvoid/)上用小额捐赠支持我的人。
