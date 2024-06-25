<!--yml

category: 未分类

date: 2024-05-27 15:15:22

-->

# Windows 11设计失误 | Brain Baking

> 来源：[https://brainbaking.com/post/2024/01/windows-11-design-mistakes/](https://brainbaking.com/post/2024/01/windows-11-design-mistakes/)

我岳母买了一台预装了Windows 11的新笔记本电脑。我以为这是个好主意，因为我们正在寻找一个易于使用的操作系统。我错了。这篇文章是提醒我自己的，下次有人需要介绍数字官僚世界时，我应该安装一个Linux发行版。

我想知道微软从何时开始失误？可能是在Windows XP之后吧？一层又一层不必要的垃圾最终成就了名为Windows 11的成就，其中AI辅助的Bing，荒谬的Microsoft Store，安全“增强”以及Xbox娱乐的不受欢迎的整合让我连续咒骂了近两个小时，直到最终成功安装了一个地方政府的扩展以使eID读卡器正常工作。

一旦你启动Windows并点击开始按钮，我注意到有许多上次我经常使用Windows时不存在的移动组件。好吧，没错，那是17年前了，但仍然如此。为什么用户必须忍受开始菜单中的热门新闻文章？或者Xbox账户激活问题？或者Bing？或者今天是什么样的天气？屏幕上闪烁的东西越多，我岳母就越困惑，这是理所当然的。所以我试图隐藏、禁用、卸载、恢复、删除、删除、杀死和销毁我认为不必要的一切。

但是我却做不到：Windows根本不让我这样做。要么我无法立即找到配置的地方——荷兰安装更糟糕——要么它根本是“核心Windows”体验的一部分，无法更改。

然后我不得不执行一个`.EXE`安装文件来让一个USB供电的读卡器工作。

但是我却做不到：Windows根本不让我这样做。从什么时候开始双击可执行文件变得不可能了呢？相反，Microsoft Store打开了，并且友好地告诉我我不应该做任何可能是恶意的事情：安装程序不是来自已验证的商店。最近的macOS版本也让安装新软件的体验变得越来越糟糕，但至少在那里我可以简单地按下“允许”，或者在最坏的情况下更改隐私和安全设置以允许从App Store以外的其他来源下载的应用程序。

结果发现Windows安装了一种叫做‘S-Mode’的东西，里面做坏事是*被禁止*的。顺便说一句，这包括按下`WIN+R`然后输入`regedit.exe`，这也是我在另一次试图通过摆弄希望正确的`HKEY_LOCAL_MACHINE`键来告诉Windows闭嘴的方法。

然后我尝试禁用 S 模式。你猜对了：我做不到。到了这一点，每分钟的诅咒数量开始呈指数增长。最简单的禁用方式是通过微软商店…下载一个特殊的软件来删除它？但那需要一个我们没有的微软账户。另一个选择是在选择“重新启动”时按住 `SHIFT` 键以访问修复模式，在那里您可以启动到 UEFI/BIOS 模式以禁用安全引导，但那也会永久性地禁用硬盘加密。而且不起作用。另一种选择是使用修复模式运行一个命令行*确实*启动 `regedit.exe` 并按照网上的可疑文章中找到的步骤进行操作。但那也不起作用。

到了这个时候，我准备放弃，只是简单地创建了一个愚蠢的微软账户来访问那个降级软件。终于，有所进展！S 模式消失后，我终于能够让那个读卡器工作了。但等等，为什么 Edge 浏览器的起始页面突然改变了？为什么我们不断地登录到那个新账户？在注销并解除与该账户的关联后，Windows 也决定丢弃了已经在 Edge 中配置的个人资料信息，例如我婆婆用来浏览网页的起始页面上的快捷链接。又是一串诅咒声接踵而至。

然后我尝试卸载 Edge，但我做不到（我觉得这里有某种模式）。就像真正的 Internet Explorer 一样，该浏览器与 Windows 硬件绑定在一起。Edge 令人分心的闪烁按钮和令人讨厌的 AI 辅助搜索栏很快让我感到恼火。幸运的是，用另一个基于 Chromium 的浏览器替换它对偶尔上网的人来说并不是太大的文化冲击。火狐不是一个选择，因为一些比利时的在线银行系统对该浏览器的优化很差，而改变浏览器对于他们来说太过分了。

禁用自动更新——或者至少是频繁且相当烦人的消息——导致了另一个*让我不能*。我能做的最好的就是将其禁用 5 周。我很清楚自动安全补丁的优势（和劣势），但对于没有经验的计算机用户来说，“重新启动您的计算机！”的消息是引起恐慌而不是放心的原因。

* * *

现代的即插即用操作系统版本带有令人惊讶的多余功能，这实在是令人沮丧。微软并不是唯一有罪的：苹果的 Sonoma macOS 也带有许多无用的功能，这些功能只会占用宝贵的固态硬盘空间。我不在乎小部件和 Siri，我希望我的操作系统能够完成它的职责：帮助操作系统运行。我不想让我的操作系统监视我，也不想让它在我不做任何操作的情况下向 `microsoft.com` 或 `apple.com` 发送网络数据包。我不想要一个强制性的复杂应用商店。我不想要一个充满广告的开始菜单/栏。我不想要创建一个账户并“登录”。

我当然不想要你的 AI 辅助帮助，非常感谢。

下次，我将从格式化并安装Ubuntu开始。这次小小的Windows冒险让我开始怀疑是否应该从macOS切换回Linux，[就像Seb那样](https://seblog.nl/2024/01/05/1/een-nieuw-begin)。

<svg class="icon icon-text"><title>标签图标</title></svg> [`设计`](https://brainbaking.com/tags/design "标签: 设计")