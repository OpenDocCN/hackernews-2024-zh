<!--yml

类别：未分类

日期：2024-05-27 14:37:05

-->

# 用大约 ~12 行 bash 重拾你的注意力 — 自学者

> 来源：[https://www.autodidacts.io/coldturkey-selfcontrol-freedom-leechblock-alternative-in-bash/](https://www.autodidacts.io/coldturkey-selfcontrol-freedom-leechblock-alternative-in-bash/)

计算机是您可以使用或被使用的工具。[我以前写过关于驯服工具的策略](https://autodidacts.io/intentional-computing/?ref=autodidacts.io)，包括在我的 hosts 文件中阻止令人分心的网站。我的“checking”函数是这一策略的自然演变。

它实现了流行的抗拖延应用程序（如[ColdTurkey](https://getcoldturkey.com/?ref=autodidacts.io)，[LeechBlock NG](https://www.proginosko.com/leechblock/?ref=autodidacts.io)，[Freedom](https://freedom.to/?ref=autodidacts.io)，和[Self Control](https://selfcontrolapp.com/?ref=autodidacts.io)）的核心功能，**仅需大约 ~12 行 bash**。

下面是如何设置“checking”：

1.  备份您现有的 `/etc/hosts` 文件。

1.  将其重命名为 `/etc/hosts-checking`。

1.  在 `/etc/hosts-doing` 创建一份副本，并添加所有令人分心的网站。我的个人弱点：电子邮件、HackerNews、Lobste.rs、网站统计和Spotify for Artists（用于我的乐队）。

1.  将下面的“checking”函数添加到您的 `~/.bashrc` 中。（抱歉，fish/zsh/nushell/powershell 用户们。我相信你们足够聪明，能够将这个简单愚蠢的函数转换为你们选择的 shell 脚本语言！）

1.  运行 `source ~/.bashrc`。

1.  可选：为了避免每次输入密码，编辑您的 sudoers 文件（使用 `sudo visudo /etc/sudoers`），并在末尾添加两行：`user ALL=NOPASSWD: /usr/bin/ln -sf /etc/hosts-checking /etc/hosts` 和 `user ALL=NOPASSWD: /usr/bin/ln -sf /etc/hosts-doing /etc/hosts`（将"user"替换为您的用户名）。

这是如何使用它的方法：

```
checking 15m 
```

您可以传递任何单位的持续时间给 sleep 函数（如 `checking 12s`，`checking 1hr` 等）

实际上就是这个东西：

```
function checking () {
    #set -x
    #set -o errexit
    export DURATION="${1:-1m}"
    echo "Starting a Checking session lasting ${DURATION}"
    sudo ln -sf /etc/hosts-checking /etc/hosts && resolvectl flush-caches
    echo "Distracting websites and comms unblocked!"
    sleep "$DURATION"
    notify-send "Checking session complete! Close your tabs."
    sleep 1m
    sudo ln -sf /etc/hosts-doing /etc/hosts && resolvectl flush-caches
    echo "Done"
    trap "sudo ln -sf /etc/hosts-doing /etc/hosts" INT # Handle Ctrl+C
    #systemctl suspend
} 
```

它非常简单，实际上是自我记录的。

注意1分钟的宽限期，并提醒该收尾了。

最后的可选暂停仅是为了强调一点，并给大脑清理的时间。（如果我实际上注意到并在通知弹出时停下来，我可以按 `Ctrl+C`。）

这里有一些您可以在 `hosts-doing` 中阻止的想法：

1.  新闻网站

1.  分析和仪表板

1.  订阅阅读器

1.  社交媒体

1.  电子邮件和聊天

```
127.0.0.1       news.ycombinator.com lobste.rs  
127.0.0.1       plausible.io googleanalytics.com goatcounter.com umami.is usefathom.com artists.spotify.com  
127.0.0.1       commafeed.com feedly.com inoreader.com
127.0.0.1       reddit.com facebook.com x.com instagram.com bsky.social threads.net craigslist.org
127.0.0.1       gmail.com slack.com discord.com 
```

这只是一个示例。我只会封锁我（过度）使用的服务，每行放一个网站以便阅读。

显然，不要阻止任何真正需要用于工作的内容。

此外，借用Tiny Tiny RSS迷人的免责声明^([1](https://tt-rss.org/?ref=autodidacts.io))：“无担保。如果它坏了，你得自己搞定。”
