<!--yml

类别：未分类

日期：2024-05-27 15:03:09

-->

# 如何和你的室友搞怪 | Max Bernstein

> 来源：[https://bernsteinbear.com/blog/how-to-mess-with-your-roommate/](https://bernsteinbear.com/blog/how-to-mess-with-your-roommate/)

在我解释我对可怜的Logan做了什么之前，我必须解释一下我们公寓的媒体设置。不久你就会明白为什么了。

Logan，如果你正在阅读这篇文章，希望你能更受欢迎。

我们有一台运行Ubuntu桌面版的电脑连接到电视上。它充当我们的媒体服务器。因为它通常也需要联网，所以它还提供了几个网页，一个SSH服务器和一些其他服务。

由于电视是4K而桌面电脑是残破的，它所配备的显卡是不可接受的。Logan决定购买了一张几代前的NVIDIA显卡（比我们现有的好得多），以显示流畅的4K视频。

### 这个恶作剧

安装后不久，我们遇到了一些驱动程序故障，这时我觉得如果有一种方法可以手动启动错误消息，那将会很有趣。

经过一些互联网搜索，明显地远程在电视上显示消息如此简单：

1.  作为显示内容的用户SSH进入到盒子中

1.  `DISPLAY=:0 zenity --info --text 'Hello!'`

（`DISPLAY=:0` 部分是必需的，因为我连接的会话上没有显示器，我希望在主显示器上显示。）

由于我们遇到了NVIDIA显卡的问题，我决定采用更类似于以下的方法：

`DISPLAY=:0 zenity --warning --text 'Display is running in low-graphics mode.'`

我尝试了这个（而且它奏效了），但是每次我想要干扰Logan时都手动从SSH客户端登录到服务器上是一件痛苦的事情。所以我决定想出点儿花样。

我考虑过定期设置一个cron任务来经常骚扰他，但这有两个问题：

1.  就是这样——普通的

1.  我们有其他*真实的*cron任务，这使得这个 sneaky 新任务太容易发现了

其他像SysVInit脚本的选项出于类似的原因也被排除了。

所以我决定改为制作一个公开访问的网页，上面有一个按钮：“干扰Logan”。

### 恶作剧设置

为了执行，我想我需要几件东西：

1.  一些可以处理用户输入的东西

1.  一些能够代表网页用户执行任意命令的东西

结果就是：

1.  NGINX

1.  NGINX FPM 扩展

1.  PHP

1.  PHP FPM 包

因此，我决定在 [http://our.apartment.server](#) 设置一个网站，其中包括一个登录页面（`logan.html`）和一个“操作页面”（`zenity.php`）：

```
<!-- logan.html -->
<html>
    <head>
        <style type="text/css">
            form button {
                font-size: 20px;
            }

            div.explanation {
                width: 400px;
            }
        </style>

        <meta name="viewport"
              content="width=device-width,initial-scale=1,maximum-scale=1,user-scalable=no">
        <meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1">
        <meta name="HandheldFriendly" content="true">
    </head>
    <body>
        <form method="POST" action="/zenity.php">
            <button>Fuck with Logan</button>
        </form>
    </body>
</html> 
```

在那里有一些无意义的 `meta` 标签，以便在手机上更容易使用页面（记住我是在方便在外出时使用）。

对于那些不能在脑海中渲染HTML的人（我想大多数人都是），那个页面看起来像这样：

当按钮被点击时，它会POST到另一个页面来处理繁重的工作：

```
<?php
/* zenity.php */

$messages = Array(
    "An error has occurred.",
    "An error has occurred.",
    "An error has occurred.",
    "An error has occurred.",
    "An error has occurred.",
    "An error has occurred.",
    "This graphics driver could not find compatible graphics hardware.",
    "Display driver stopped responding.",
    "Display driver stopped responding.",
    "Display driver stopped responding.",
    "Display driver stopped responding and has recovered.",
    "The system is running in low-graphics mode.",
    "The system is running in low-graphics mode.",
    "The system is running in low-graphics mode.",
    "The system is running in low-graphics mode.",
    "The system is running in low-graphics mode.",
    "The system is running in low-graphics mode.",
    "NVIDIA driver installer stopped responding.",
    "NVIDIA has stopped this device because it has reported problems. (Code 43)",
    "An error has occurred with iface wlx10bef54d395c."
);

$statuses = Array("error", "warning");

$msg = $messages[array_rand($messages)];
$status = $statuses[array_rand($statuses)];
$timeout = "--timeout 10";

exec("sudo -u thedisplayuser /usr/sbin/zenity --$status --display=:0 --text 'Error: $msg' $timeout > /dev/null &");

include 'logan.html';

?>

<div class="explanation">
A pop-up with a randomly-generated error message just appeared on the home
TV and bothered the shit out of Logan, who knows nothing of this website. He is
now wondering why the heck we have so many graphics card / internet / other
issues.
</div>
<br />
<img src='/logan.jpg' /> 
```

这个页面做了很多事情：

1.  选择一个半随机的错误消息来显示

1.  选择将出现的对话框类型

1.  在一定时间内显示该框（10秒）

1.  渲染按钮以及刚刚发生的解释，还附上洛根本人的有趣照片

对于那些无法在头脑中渲染HTML的人（希望大多数人都不会），该页面看起来像这样：

如果你想知道为什么网页允许以这种方式执行代码，并且Web服务器所有者（`www-data`）如何可能作为显示用户（`thedisplayuser`）执行命令，你可能会高兴地知道，我在sudoers文件中严格限制了它：

```
# /etc/sudoers
www-data ALL=(thedisplayuser) NOPASSWD: /usr/bin/zenity 
```

这个特定的配置允许`www-data`作为`thedisplayuser`执行仅限于`/usr/bin/zenity`的命令，而且无需密码。

我消灭了一些恼人的PHP和NGINX配置无聊之后，部署了它。然后，我把URL发给了校园里认识洛根的几个朋友。

### 恶作剧结果

谢天谢地，洛根的反应如此强烈。如果他只是有点恼火，我会感到失望，觉得我所有的努力都白费了。但没有！他失去了冷静。

我数不清有多少次重新启动，重新安装驱动程序，修改内核。我只希望当他打开VLC后，公寓里有人突然弹出一堆显示图形卡错误的窗口时，我能录下他的愤怒表情。

但我太惊讶了，另一个室友克里斯决定干预…

### 恶作剧者被恶作剧

#### 第一步

有一天我注意到洛根睡觉时出现了一条错误消息：“Max是幕后主使”。什么？？？我被反向恶作剧了！于是我进行了调查，发现有人（克里斯）将该消息编辑到了`zenity.php`中，随机选择。我立即删除了它（洛根暂时还没能察觉到这个恶作剧），并且认为这场趣味已经结束了。但没有。

#### 第二步

过了大约一周，消息又出现了。我以为克里斯注意到了并决定重新添加到列表中。错了。不在那里。仔细检查文件后，我注意到它现在调用的是`/usr/sbin/zenity`，而不是系统默认的`/usr/bin/zenity`，并且在sudoers文件中有相应的条目允许它。`/usr/sbin/zenity`是什么？一个shell脚本：

```
#!/bin/bash

echo '.' >> /tmp/log.txt
if [ 0 -eq $((RANDOM % 100)) ];
then /usr/bin/zenity --error --display=:0 --text "Max is responsible for these." --timeout 10 > /dev/null &
else /usr/bin/zenity "$@"
fi 
```

如果我见过这种东西，那可真是高级别的。百分之九十九的情况下，它做对了事情——而另外百分之一的情况下，它会显示“Max对此负责”。我删除了文件（一个错误，我现在知道了），sudoers条目也改了，并把`zenity.php`改回了原样。消息停止出现了。但后来它们又回来了。

#### 第三步

我检查了`zenity.php`。没有新的内容。`/usr/sbin/zenity`？消失了。我感到困惑。

所以我决定窥视`/usr/bin/zenity`：

```
#!/bin/bash
# --- SOME BINARY JUNK ---
# --- SOME BINARY JUNK ---
# --- SOME BINARY JUNK ---
# --- SOME BINARY JUNK ---
# --- SOME BINARY JUNK ---
if [ 0 -eq $((RANDOM % 70)) ];
then /usr/bin/rpmdb-client --error --display=:0 --text "M""a""x"" ""i""s r""e""s""p""o""n""s""i""b""l""e"" f""o""r ""t""h""e""s""e." --timeout 10 > /dev/null &
else /usr/bin/rpmdb-client "$@"
fi 
```

这个狡猾的家伙。他直接编辑了`zenity`二进制文件，把它变成了一个Bash脚本，而且只有1/70的时间会工作。这是什么鬼。`rpmdb-client`是什么鬼？所以我反击了。我修改了它：

```
# <SNIP>
if [ 0 -eq $((RANDOM % 70)) ];
then /usr/sbin/rpmdb-client --error --display=:0 --text "M""a""x"" ""i""s r""e""s""p""o""n""s""i""b""l""e"" f""o""r ""t""h""e""s""e." --timeout 10 > /dev/null &
else /usr/bin/rpmdb-client "$@"
fi 
```

看到区别了吗？第一个调用 `/usr/sbin/rpmdb-client` 而不是 `/usr/bin/rpmdb-client`，并将 `sbin` 变体变成了无操作的 bash 脚本。幸运的话，他不会注意到这个一个字符的改变，他的消息就永远不会显示出来。

TODO: 弄清楚 `/usr/sbin/zenity` 和 `/usr/bin/rpmdb-client` 之间的 ELF 可执行文件的区别，这些文件是由 Chris 创建的。有一些我还不理解的奇怪二进制差异。

#### 第四步

我决定在 Chris 发现上述一字之差之前再进一步反击。我撤销了所有修改，并决定对 `zenity` 进行补丁。在这里，特别感谢[Tom Hebb](https://tchebb.me/)（就像我写的其他技术文章一样）帮助我。这是我所做的：

1.  配置 `apt` 以下载源包（在这种情况下，添加 `deb-src` 行到 `/etc/apt/sources.list`）

1.  `apt-get source zenity`

1.  使用 `quilt` 制作补丁：

    1.  `quilt new myPatch.diff`

    1.  对 `src/msg.c` 进行补丁，以侦测消息文本中是否使用了 “max” 或 “Max” 这两个单词：

        ```
        Index: zenity-3.18.1.1/src/msg.c
        =================================================================== --- zenity-3.18.1.1.orig/src/msg.c +++ zenity-3.18.1.1/src/msg.c @@ -21,6 +21,8 @@
          * Authors: Glynn Foster <glynn.foster@sun.com>
          */

        +#include <string.h>
        +
         #include "config.h"

         #include "zenity.h"
        @@ -85,6 +87,11 @@ zenity_msg (ZenityData *data, ZenityMsgD
           GObject *text;
           GObject *image;

        +  if (strstr(msg_data->dialog_text, "Max")
        +      || strstr(msg_data->dialog_text, "max")) {
        +      return;
        +  }
        +
           switch (msg_data->mode) {
             case ZENITY_MSG_WARNING:
               builder = zenity_util_load_ui_file ("zenity_warning_dialog", NULL); 
        ```

    1.  `quilt add src/msg.c`

    1.  `quilt pop`

1.  `dpkg-source --commit`

1.  `dpkg-buildpackage -us -uc`

1.  意识到创建和安装新包比仅静默替换二进制文件更为显眼。

1.  静默替换 `rpmdb-client`（他的 `zenity`）二进制文件为我编译的 `zenity` 版本。

1.  对自己的聪明之举感到自豪。

这个补丁修改了 `zenity`，如果它注意到消息包含文本 “max” 或 “Max”，它将悄悄地不做任何操作。

UPDATE: 到目前为止（2018 年三月底），Chris 没有注意到我修改了二进制文件。因此，Logan 也没有注意到任何包含我的名字的消息。这个恶作剧似乎没有尽头，除非我决定在我们毕业时告诉他。

UPDATE: 我们毕业了，各奔东西。虽然 Logan 和我明年将再次共居，但我们可能不会在家里待得很久，以继续这个恶作剧。我决定在进一步捣蛋之前发布这篇文章。
