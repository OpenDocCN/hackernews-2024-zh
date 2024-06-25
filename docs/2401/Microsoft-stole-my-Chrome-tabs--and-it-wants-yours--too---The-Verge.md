<!--yml

分类：未分类

日期：2024 年 05 月 27 日 15:18:09

-->

# Microsoft 偷了我的 Chrome 标签页，而且它也想要您的 - The Verge

> 来源：[`www.theverge.com/24054329/microsoft-edge-automatic-chrome-import-data-feature`](https://www.theverge.com/24054329/microsoft-edge-automatic-chrome-import-data-feature)

上周，我打开了我的电脑，安装了一个 Windows 更新，并重新启动，结果发现 Microsoft Edge 自动打开了我在更新之前正在使用的 Chrome 标签页。我不经常使用 Microsoft Edge，并且我已经将 Google Chrome 设置为我的默认浏览器。9 点钟时，我睡眼惺忪，一时之间没意识到 Microsoft Edge 竟然接管了我在 Chrome 上的工作。我简直不敢相信我的眼睛。

我从未将我的数据导入到 Microsoft Edge，也没有确认是否要导入我的标签页。但是这里是 Edge 在 Windows 更新后自动打开，显示我之前正在工作的所有 Chrome 标签页。我起初甚至没有意识到我在使用 Edge，并且对我的所有标签页突然退出登录感到困惑。

震惊之后，我查看确保自己没有意外允许这种行为。我在 Microsoft Edge 中找到了一个设置，该设置在每次启动时从 Google Chrome 导入数据。“每次在 Microsoft Edge 上浏览时都始终可以访问您最近的浏览数据”，Microsoft 在 Edge 中对该功能的描述如此说道。这个设置是禁用的，我从来没有被要求启用它。您可以在 edge://settings/profiles/importBrowsingData 检查该设置。

*Microsoft Edge 打开我的 Chrome 标签页。*

图片由 Tom Warren / The Verge 提供

所以我去在笔记本电脑上安装了相同的 Windows 更新，结果导致失败，我不得不进行系统恢复。一旦系统恢复完成，同样的事情发生了。Edge 自动打开，显示所有我的 Chrome 标签页。我无法在其他电脑上复制这种行为，但一些 X 用户[回复了我的帖子](https://twitter.com/tomwarren/status/1750175894306439601)，称他们过去也经历过同样的情况。

所有这些似乎都与 Microsoft Edge 中一个很少被报道的导入功能有关，我已确认我的系统上已禁用了该功能。隐私和数据供应链研究员 Zach Edwards 在刚安装的 Windows 中[回复了我的帖子](https://twitter.com/thezedwards/status/1750952950598672455)指出，他在安装过程中发现了一个新的提示:

> 在您确认后，Microsoft Edge 将定期从您的 Windows 设备上的其他浏览器中获取数据。这些数据包括您的收藏夹、浏览历史、Cookies、自动填充数据、扩展、设置和其他浏览数据。

Microsoft 表示数据导入在本地完成并在本地存储，但如果您登录并同步浏览数据，它将被发送到 Microsoft。Microsoft 显示一个大大的蓝色接受按钮，鼓励 Windows 用户启用该功能，如果您想退出，则显示一个较暗的“暂不”的按钮。

我不太确定微软 Edge 是如何在没有启用此设置的情况下自动打开并导入我的 Chrome 浏览器数据的。我注意到在那天早晨安装更新后，我的电脑上出现了一个全屏提示，就像微软在 Windows 更新后用来鼓励人们切换到 Edge 和必应的提示一样。提示出现了，但在不到一秒钟的时间里消失了，所以可能这个对话框崩溃了，Edge 决定仍然进行导入。

我已经要求微软就我所见到的情况发表评论，但该公司尚未在出版时作出回应。

我并不是唯一一个遇到这种情况的人。多名 Windows 用户已经报告了这一情况，他们求助于[Reddit](https://www.reddit.com/r/microsoft/comments/17c7w4i/edge_is_secretly_importing_my_data_from_google/)和[微软自己的支持论坛](https://answers.microsoft.com/en-us/microsoftedge/forum/all/how-to-make-edge-stop-importing-from-chrome/4bcb43bb-b54a-4413-b24c-ea43b49cea54)寻求帮助。[发布者已经确认](https://answers.microsoft.com/en-us/microsoftedge/forum/all/how-do-i-stop-edge-from-importing-bookmarks-and/22d7af7f-d111-481d-9ef3-13a703e61c18) 相同的功能已经被禁用，但数据导入仍在发生。

也许这次对我来说只是一个 bug，或者我可能即将经历其他人已经抱怨了几个月的问题。但很明显，微软已经推出了一个功能，可以自动从任何不是 Edge 的 Web 浏览器中导入 Chrome、Firefox 或其他浏览器的数据。对于一些人来说，这可能是一个有用的功能，但如果它有问题，那么微软需要尽快修复这个问题。

考虑到在我遇到问题之前消失的全屏提示，微软如何说服人们打开这个设置也将是很重要的。微软已经在一些 Windows 更新后开始[发布全屏提示](https://twitter.com/tomwarren/status/1464624781471404035)，试图诱使人们将默认浏览器和搜索引擎切换到 Microsoft Edge 和必应。

*微软经常使用全屏提示来试图让 Chrome 用户切换到 Edge。*

Tom Warren / The Verge 提供的屏幕截图

对于消费者来说，他们实际上改变了什么并不总是显而易见，特别是当微软将事情伪装成“推荐”的浏览器设置时。只需点击一个明显较不突出的“现在不是”的按钮，而不是一个大大的蓝色“是”按钮就容易多了。微软有一个历史 使用类似于膨胀软件和间谍软件开发者所采用的策略来推广其网络浏览器。这些策略包括在没有获得许可的情况下启动 Edge 并将其固定到桌面和任务栏，以及在您尝试下载 Chrome 时进行投票或提示。

微软 Edge 实际上是相当不错的，所以我希望构建它的团队不会采取更多的把戏来让 Chrome 用户使用它。
