<!--yml

category: 未分类

日期：2024-05-27 14:35:23

-->

# 使用 resolved、vlc 和 bash 向 Chromecast 流式传输

> 来源：[`linderud.dev/blog/stream-to-chromecast-with-resolved-vlc-and-bash/`](https://linderud.dev/blog/stream-to-chromecast-with-resolved-vlc-and-bash/)

Chromecast 是我经常使用的设备之一。它们小巧实用，使我能够从多个设备向电视机传输视频或音乐。但它也需要你拥有支持的浏览器或视频播放器。显然，这有点无聊。

多年来已经有多个命令行 Chromecast 流式传输工具了。但它们对于 ffmpeg 的使用程度往往不佳，没有硬件解码支持，并且通常实现得相当糟糕。

而不是使用这些工具，我最终编写了两个 bash 脚本，将命令行程序组合在一起，向 Chromecast 流式传输，并且效果出奇地好。

这的第一部分是我们如何向 Chromecast 进行流式传输。幸运的是，使用 `vlc` 很容易解决这个问题。你只需打开 `vlc file://....`，选择 `播放 -> 渲染器`，通常会在那里找到正确的 Chromecast。但这很无聊。我们必须手动打开 GUI 并选择设备！

我可以在手机上查看应用程序，并了解到我的 Chromecast 的本地 IP 地址为 `192.168.1.231`。使用 `clvc`，你还可以传递 `sout` 和 `sout-chromecast-ip` 来自动启动一个无头 VLC 播放器，该播放器将连接到你的 Chromecast。

```
IP="192.168.1.231"
cvlc --sout "#chromecast" --sout-chromecast-ip="$IP" "file://...."
```

这本身就非常有效。你还可以将 `yt-dlp` 管道传递到 `clvc` 并支持大多数媒体网站，如 YouTube 或 Twitch。这使你能够快速编写一些脚本来从这些网站流式传输，并且非常灵活。

```
IP="192.168.1.231"
yt-dlp "https://twitch.tv/sovietwomble" -o - | \ cvlc --sout "#chromecast" --sout-chromecast-ip="$IP" -
```

不过，存在一个问题，即该 IP 不是静态的。如果 Chromecast 重新连接到网络，它的 IP 可能会更改，而我永远不会费心设置静态设备。

幸运的是，Chromecast 通过 [DNS 服务发现](http://www.dns-sd.org/) 自报告，这使得设备可以通过 DNS 名称在网络上进行广播。为了查询这些设备，我们将使用 `systemd-resolved` 和 `resolvectl`。你可能也可以使用其他 DNS 查询工具，但 `systemd-resolved` 还支持 `mDNS`，这对于在网络上查找服务很有用。

首先，你需要通过 `systemctl enable --now systemd-resolved` 启用 `systemd-resolved`。你的情况可能有所不同，但在某些发行版中，默认情况下已启用此功能。值得注意的是，你不需要将 `resolved` 设为默认的存根解析器才能使所有这些功能正常运行。

如果你阅读 [`resolvctl(1)` 手册页](https://man.archlinux.org/man/resolvectl.1#COMMANDS)，你会看到它通过 `resolvectl service` 原生支持 dns-sd。

```
$ resolvectl service _googlecast._tcp local
```

但是我还没有让它工作，因为它似乎会尝试向您的网络 DNS 解析器发送一个组合的 DNS 查询，这似乎取决于您拥有的路由器是否支持。所以我们必须自己实现这个功能。因此，我们将使用 [RFC6763](https://www.ietf.org/rfc/rfc6763.txt) 中概述的 `PTR+SRV+TXT` 过程。

要查找网络上可用的 Chromecast，我们首先会在 `_googlecast._tcp.local` 上查询 PTR 记录。

```
$ resolvectl --type=PTR query _googlecast._tcp.local
_googlecast._tcp.local IN PTR Chromecast-Ultra-13ccb56dd0ea6ce65dbe15b39c573856._googlecast._tcp.local
```

在这里我看到一个 Chromecast。然后我们需要查询服务记录本身，SRV 被定义为 [RFC2782](https://tools.ietf.org/html/rfc2782) 的一部分。

```
$ resolvectl --type=SRV query Chromecast-Ultra-13ccb56dd0ea6ce65dbe15b39c573856._googlecast._tcp.local
Chromecast-Ultra-13ccb56dd0ea6ce65dbe15b39c573856._googlecast._tcp.local IN SRV 0 0 8009 13ccb56d-d0ea-6ce6-5dbe-15b39c573856.local
```

最后，我们只需查询服务的 TXT 记录以获取设备本身的 IP 地址，这是我们起初正确的 IP 地址 :)！

```
$ resolvectl query 13ccb56d-d0ea-6ce6-5dbe-15b39c573856.local
192.168.1.231
```

为了将来更容易处理，我们可以在 bash 脚本中使用 `read` 轻松解析这个空格限制的输出^。

```
$ ~/.local/bin/dns-sd
#!/usr/bin/bash
set -e
read -r _ _ _ ptr _ < <(resolvectl --type=PTR query "$1")
read -r _ _ _ _ _ _ srv _ < <(resolvectl --type=SRV query "$ptr")
read -r _ ip _ < <(resolvectl query "$srv")
printf "$ip"
```

我们还可以制作我们自己的 `chromecast` 脚本，调用 `dns-sd`。

```
$ ~/.local/bin/chromecast
#!/usr/bin/bash
IP="$(dns-sd _googlecast._tcp.local)"
echo "Using $IP"
yt-dlp "$1" -o - | cvlc --sout "#chromecast" --sout-chromecast-ip="$IP" -
```

有了这个，我们有了一个很好的方法将 web 内容流式传输到我们的 chromecast，而不必通过 GUI。

### 附加内容：qutebrowser

如果你和我一样喜欢 qutebrowser，但发现缺少 chromecast 支持很烦人，我们可以用上面的脚本解决这个问题。

我们将使用 qutebrowser 中的当前 chromecast 脚本：

[`github.com/qutebrowser/qutebrowser/blob/main/misc/userscripts/cast`](https://github.com/qutebrowser/qutebrowser/blob/main/misc/userscripts/cast)

并在这里应用差异，只删除我们实际不需要的代码。

```
diff --git a/misc/userscripts/cast b/misc/userscripts/cast index ec703d5fb..57c8f95f9 100755 --- a/misc/userscripts/cast +++ b/misc/userscripts/cast @@ -139,37 +139,6 @@ printjs() {
 }
 echo "jseval -q $(printjs)" >> "$QUTE_FIFO"

-tmpdir=$(mktemp -d) -file_to_cast=${tmpdir}/qutecast -cast_program=$(command -v castnow) - -# pick a ytdl program -for p in "$QUTE_CAST_YTDL_PROGRAM" yt-dlp youtube-dl; do -    ytdl_program=$(command -v -- "$p") -    [ "$ytdl_program" == "" ] || break -done - -if [[ "${cast_program}" == "" ]]; then -    msg error "castnow can't be found" -    exit 1 -fi -if [[ "${ytdl_program}" == "" ]]; then -    msg error "youtube-dl or a drop-in replacement can't be found in PATH, and no installed program " \ -        "specified in QUTE_CAST_YTDL_PROGRAM (currently \"$QUTE_CAST_YTDL_PROGRAM\")" -    exit 1 -fi - -# kill any running instance of castnow -pkill -f -- "${cast_program}" - -# start youtube download in stream mode (-o -) into temporary file -"${ytdl_program}" -qo - "$1" > "${file_to_cast}" & -ytdl_pid=$! - +pkill -f -- chromecast
 msg info "Casting $1" >> "$QUTE_FIFO"
-# start castnow in stream mode to cast on ChromeCast -tail -F "${file_to_cast}" | ${cast_program} - - -# cleanup remaining background process and file on disk -kill ${ytdl_pid} -rm -rf "${tmpdir}" +chromecast "$1" 
```

然后在 `config.py` 中添加一个键绑定。

```
config.bind(',c', 'spawn --userscript cast')
```

哒哒，我们的脚本支持 chromecast 了。

[返回文章](https://linderud.dev/blog/)
