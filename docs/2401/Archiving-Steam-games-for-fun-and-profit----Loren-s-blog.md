<!--yml

类别：未分类

date: 2024-05-27 14:31:48

-->

# 为了娱乐和利润存档 Steam 游戏 // Loren 的博客

> 来源：[`lorendb.dev/posts/archiving-steam-games-for-fun-and-profit/`](https://lorendb.dev/posts/archiving-steam-games-for-fun-and-profit/)
> 
> 简而言之：如果你想创建所有 Steam 游戏的存档，请查看我在 GitHub 上的[download-steam-games](https://github.com/LorenDB/download-steam-games)。它有很少的文档，但你应该能够轻松运行它。

我最近决定下载[KSP2](https://www.kerbalspaceprogram.com/games-kerbal-space-program-2)的一些测试版本，这样一年后我可以回顾它们，并笑一笑最初版本中存在多少可怕的 bug。我决定直接将游戏备份下载到我的服务器，所以我通过`ssh`登录了服务器，从系统仓库安装了 Steam，并启动了一个新的`ssh -X`会话。然而，当我启动 Steam 时，我意识到我的服务器连接互联网速度太慢了，以至于无法在任何可用状态下运行`ssh -X`。事实上，它太慢了，以至于似乎在 Steam 端超时了，并且几乎不可能登录 Steam。

不屈不挠，我开始研究[SteamCMD](https://developer.valvesoftware.com/wiki/SteamCMD)。结果发现，虽然 SteamCMD 是为游戏服务器管理员设计的，但它愿意下载任何 Steam 游戏。我很快就安装并使用它下载了一款游戏。但是，为了下载多个游戏，反复在 SteamCMD 中执行一组指令有点麻烦，所以我决定写脚本来完成这项工作。

## 但是你为什么要这么做呢？

为什么我要这么做？如果你认为我疯了，去[r/DataHoarder](https://www.reddit.com/r/DataHoarder/)看看。

说真的，虽然我不指望 Steam 明天就会消失，但游戏被从平台上移除是有先例的；许多最早的游戏已经被移除。按照它们在 Steam 上可用日期排列，最早的游戏是原版[Counterstrike](https://store.steampowered.com/app/10)，游戏 ID 为 10。游戏 1 至 9 已不再在平台上，而且我确信它们远不是唯一被删除的游戏。存档游戏可以让我避免丢失从 Steam 上移除的游戏。此外，有时候爱好者希望能够访问游戏的旧版本；r/DataHoarder 上有史以来最受欢迎的帖子之一是关于[某人曾收藏一个旧版 Minecraft Alpha](https://www.reddit.com/r/DataHoarder/comments/o9cnj3/one_womans_quest_to_never_delete_anything_allowed/)，让 Minecraft 的爱好者有机会更全面地了解游戏的演变过程。

> 更新（2024 年 2 月 23 日）：事实证明我在这一点上是错误的。游戏可能会从 Steam 中下架，但它们并没有被删除。如果你拥有这款游戏，你仍然可以下载它。而且，由于 Steam 通过十位数来编号游戏，原始的反恐精英 *是* Steam 上列出的第一款游戏。对此表示歉意！

## 一个天真的 bash 脚本

我的第一种方法是创建一个 bash 脚本，用一个漂亮的语法包装 SteamCMD。当然，我也想要绕过交互式提示。让我们看看最终我建立了什么：

```
STEAM_ACCOUNT= GAME_NAME= GAME_STEAM_ID= GAME_FORCE_WINDOWS= GAME_BETA= 
```

这些变量控制一些基本的东西。`STEAM_ACCOUNT` 是你在 SteamCMD 中登录的帐户；虽然 SteamCMD 缓存凭据，但你仍然必须在每次启动它时发出 `login <username>` 命令，所以脚本必须知道要使用哪个用户名。`GAME_NAME` 不一定是实际的游戏名称；它只是用于生成最终游戏存档的文件名。`GAME_STEAM_ID` 是你想要下载的游戏的数字 ID；例如，KSP2 是 [`954850`](https://store.steampowered.com/app/954850/Kerbal_Space_Program_2/)。

`GAME_FORCE_WINDOWS` 值得更详细地解释一下。虽然我使用 Linux，但有些游戏没有 Linux 构建可用。对于这些游戏，我想保存 Windows 构建的备份。这将让我稍后通过 Proton 运行游戏。方便的是，SteamCMD 允许你通过设置 `@sSteamCmdForcePlatformType <platform>` 来覆盖你下载游戏的平台。`GAME_FORCE_WINDOWS` 后来被用作 SteamCMD 脚本内容的一部分；因此，如果我想添加一个 Windows 覆盖，我可以将 `GAME_FORCE_WINDOWS` 设置为 `@sSteamCmdForcePlatformType windows`。

最后，我们有 `GAME_BETA`。[SteamDB](https://steamdb.info) 显示游戏的测试版标识符；你可以使用这些来下载一些游戏的早期版本。考虑到这个项目的最初目的是存档早期的 KSP2 构建，这是我想要使用的东西。

脚本的下一部分仅仅是基于命令行参数设置每个变量的逻辑。出于简洁起见，我决定跳过它，因为这是微不足道的 bash 逻辑。继续往下，我们得到这个：

```
cat > ~/.download-$GAME_NAME.txt << EOF $GAME_FORCE_WINDOWS force_install_dir $HOME/Steam/downloads/$GAME_NAME login $STEAM_ACCOUNT app_update $GAME_STEAM_ID $GAME_BETA validate quit EOF 
```

SteamCMD 可以通过脚本进行控制。这就是我们要绕过其交互式界面的方式。

脚本的第一行是前述的平台覆盖。之后，我们使用 `force_install_dir` 强制 SteamCMD 将游戏下载到已知位置。在这个脚本的版本中，我已经将其硬编码为 `~/Steam/downloads`，因为我已经将 SteamCMD 安装到了 `~/Steam`。然后我们登录并发出实际命令来获取应用程序；一旦应用程序下载完成，我们就退出 SteamCMD。

```
PREVIOUS_DIR=$(pwd) cd ~/Steam ~/Steam/steamcmd.sh +runscript $HOME/.download-$GAME_NAME.txt cd $PREVIOUS_DIR rm ~/.download-$GAME_NAME.txt 
```

这里我们实际上执行脚本，然后删除它以防止混乱。

```
PREVIOUS_DIR=$(pwd) cd ~/Steam/downloads tar --use-compress-program=pigz -cf $PREVIOUS_DIR/$GAME_NAME.tar.gz $GAME_NAME cd $PREVIOUS_DIR   rm -r $HOME/Steam/downloads/$GAME_NAME 
```

最后一步是将游戏文件夹压缩成一个单独的`.tar.gz`文件。同样，我们现在将最终归档文件倾倒到`~/Steam/downloads`中。这其中唯一真正不寻常的部分是，我告诉`tar`使用`pigz`来进行 gzip 压缩。这是因为当你归档许多吉字节的游戏数据时，`pigz`会使用所有 CPU 核心，这一点非常重要。

这一切都很好，但你仍然需要手动运行每个游戏的命令。为了减轻这一点，我创建了一个[简单的脚本](https://paste.segfault.foo/loren/ac1cfff462804e70bdf85fb764328b19)，它读取一个模糊类似 CSV 的文件，列出了要下载的游戏。那绝对比什么都不做要好，但仍然感觉很粗糙。

## 正确构建它的方式

我决定将我的两个脚本移植成一个应用程序。我选择用[D](https://dlang.org)编写这个应用程序；回顾起来，Python 可能是一个很好的选择，因为它几乎预装在每个发行版上，但我更喜欢用 D 来做几乎所有的事情。我的移植脚本与上面显示的功能相似；然而，我添加了各种改进。路径不再是硬编码的，可以为任何平台（Windows、macOS 和 Linux）下载游戏，并且游戏数据存储为 JSON 文件，而不是奇怪的自定义格式。

我称之为`download-steam-games`（多么原创）的应用程序现在可以在[GitHub](https://github.com/LorenDB/download-steam-games)上获得。构建它只需运行`dub build`。之后，你需要运行`dub run -- --add-game`来将游戏添加到下载列表中。添加了一些游戏后，你可以使用`dub run`以下载模式运行应用程序。它将为您指定的每个平台下载所有游戏，将它们`.tar.gz`，并将它们放在您选择的输出文件夹中。

## 限制

应用程序目前有些限制：没有重新配置设置的功能，不能用`--add-game`指定 beta 版本，并且下载过程中没有进度报告。但是，我认为它已经足够健壮，可以在生产中实际使用。我希望尽快解决所有这些问题，并且在此期间，我想添加一个功能，可以为归档名称添加时间戳，这样你就可以随着时间下载许多版本的相同游戏。

## 结论

这是一个有趣的项目，正在帮助我上瘾于数据囤积。如果你开始使用我的应用程序来帮助你自己囤积游戏，请在下面留下评论！我很想看到你囤积了多少吉字节（或者是几 TB）的游戏。

## 更新

写完这篇文章后不久，我给应用程序添加了一些日志记录。它不完美，但也不算糟糕。

# 评论

由[Cactus Comments](https://cactus.chat)提供支持
