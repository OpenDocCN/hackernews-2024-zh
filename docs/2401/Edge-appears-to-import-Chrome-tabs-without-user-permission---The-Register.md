<!--yml

category: 未分类

date: 2024-05-27 15:23:52

-->

# Edge 看起来在未经用户许可的情况下导入 Chrome 标签页 • The Register

> 来源：[https://www.theregister.com/2024/01/30/microsoft_edge_tabs/](https://www.theregister.com/2024/01/30/microsoft_edge_tabs/)

更新的 Windows 用户请注意：据称，微软的 Edge 浏览器正在未经许可的情况下主动导入来自谷歌浏览器的开放式标签页和其他数据，即使已禁用导致此情况发生的“功能”。

嵌在 Windows PC 上的 Edge 浏览器设置中的代码远不止一次性导入收藏夹和存储的密码，并且至少自 [2022 年中期](https://www.windowslatest.com/2022/05/30/microsoft-edge-new-feature-will-constantly-pull-data-from-chrome/) 以来以某种形式存在。它赋予 Edge 能力，使其每次启动时都能导入来自 Chrome 的[几乎](https://support.microsoft.com/en-us/microsoft-edge/what-s-imported-to-microsoft-edge-ab7d9fa1-4586-23ce-8116-e46f44987ac2#:~:text=What%27s%20imported%20from%20Google%20Chrome%3F) 所有浏览器数据。

表面上是微软简化让 Windows 用户转换到 Edge 的过程的一种方式，但是根据用户的说法，该功能目前在没有完全许可的情况下进行。正如 Windows 制造商惯常做的那样，它还会将这些数据[同步](https://support.microsoft.com/en-us/microsoft-edge/change-and-customize-sync-settings-in-microsoft-edge-be529080-f2e9-b642-538f-976956b8da6b)到云端，只要用户登录了 Microsoft 账户 - 如果你打算保持 Chrome 和 Edge 环境分开，这并不是好消息。

[更多数据](https://www.theregister.com/2023/11/15/google_amazon_microsoft_mozilla/)供 Redmond 使用；正如我们过去报道的那样，用户的隐私权更少。

自去年以来，即使那些不常使用 Edge 的 Windows 用户也一直在报告导入功能的问题和意外惊喜。其他媒体已经[报道过](https://www.ghacks.net/2023/10/13/how-to-stop-microsoft-edge-from-importing-google-chrome-browsing-data-at-launch/)该功能的贪婪本质，并建议采取措施阻止其发生（例如通过打开 `edge://settings/profiles/importBrowsingData` 并确保关闭），但读者对这些报道的回应表明设置并不重要，导入仍然会发生。

此问题最近由 The Verge 的 Tom Warren 再次提出，他[表示](https://www.theverge.com/24054329/microsoft_edge-automatic-chrome-import-data-feature)他注意到 Edge 在系统重新启动时以与 Chrome 重新启动前相同的状态打开。Warren声称他既没有将 Chrome 数据导入 Edge，也没有授权其从 Chrome 导入开放的浏览器标签页，但情况却是如此。Warren说他从未切换过此设置，并且在他检查时它是关闭的。

快速在互联网上转一圈就会发现，这种情况发生在很多人身上——默认情况下，数据被拉入 Edge 而没有得到许可——尽管这似乎并非普遍现象。

[过去几个月](https://answers.microsoft.com/en-us/microsoftedge/forum/all/how-to-make-edge-stop-importing-from-chrome/4bcb43bb-b54a-4413-b24c-ea43b49cea54)的多个 Microsoft [支持](https://answers.microsoft.com/en-us/microsoftedge/forum/all/why-edge-keep-syncing-all-data-from-chrome-even/1efde161-5c8b-411b-a59a-1bfa2b8cedcc) [帖子](https://answers.microsoft.com/en-us/microsoftedge/forum/all/how-do-i-stop-edge-from-importing-bookmarks-and/22d7af7f-d111-481d-9ef3-13a703e61c18) 都声称存在同样的问题——即使切换了设置，Edge 仍然意外地从 Chrome 导入数据。Reddit 上的几篇帖子也[提到](https://www.reddit.com/r/MicrosoftEdge/comments/17xglpc/edge_keeps_importing_chrome_bookmarks_on_startup/)了同样的情况。

来自 Microsoft 员工和独立顾问的回复建议检查 `ImportBrowsingData` 设置，并手动通过在组策略设置中手动添加注册表键来禁用自动导入，但回复都是一样的：似乎没有办法关闭它。

有报告称 Edge 在启动时打开了以前重启前打开的 Chrome 标签，这一情况似乎在 Windows 11 更新 KB5034204（https://support.microsoft.com/en-au/topic/january-23-2024-kb5034204-os-builds-22621-3085-and-22631-3085-preview-7652acf2-56dc-430e-b8ef-ec8f56ec1028）发布后引起了轰动。这次更新似乎以某种方式强制用户使用该功能。

[其他人](https://www.reddit.com/r/Windows11/comments/19e118i/january_23_2024kb5034204_os_builds_226213085_and/)报告了在安装更新后 Edge 以 Chrome 数据的身份出现的类似问题，尽管安全研究员 Zach Edwards 在 X/Twitter 上[指出](https://twitter.com/thezedwards/status/1750952950598672455)，这次微软可能有借口。更新安装步骤中的屏幕提到，Edge 将“定期从您 Windows 设备上的其他浏览器中导入数据”。

这一切都是公开的——在某种程度上——因为微软在臭名昭著的 KB5034204 更新安装时简要提到了这一点。

除了Edge的问题之外，KB5034204似乎存在着一些影响许多Windows 11用户的问题，这可以从其发布几天后的一个[Reddit帖子](https://www.reddit.com/r/WindowsHelp/comments/1abedvs/update_kb5034204_fails_to_install/)中看出。用户报告了安装失败和启动问题，而其他网站则[报道](https://windowsreport.com/windows-11-kb5034204-issues/)了应用程序崩溃、文件资源管理器和任务栏的[问题](https://www.windowslatest.com/2024/01/28/windows-11-kb5034204-is-crashing-file-explorer-taskbar-and-wont-install-for-some/)以及其他问题。我们再次联系了微软，询问了KB5034204的更广泛问题，但是至今也没有得到回复。

无论微软是否在该更新中征求了Edge从其他浏览器导入数据的许可，问题并不新鲜——在论坛帖子和帮助请求中已经有了充分的记录。我们已经向微软寻求解释，虽然我们被告知他们正在调查此事，但我们还没有收到实际的回复。

同时，在微软修补补丁之前，最好先暂停安装最新一轮的Windows 11更新。或者解释一下。啊，我们在开玩笑。我们已经要求评论。®

### 更新添加

微软目前拒绝发表评论。一位熟悉这一混乱情况且对Windows巨头有了解的人士，尽管不愿透露身份，告诉我们似乎“如果用户在其他设备上选择了Edge的连续导入体验，那么这种状态可能会在他们的设备之间同步错误。这并不是预期的功能体验。”我们得到的保证是，微软将在下一次Edge稳定版发布时解决这个问题。
