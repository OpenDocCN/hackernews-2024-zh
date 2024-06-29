<!--yml

类别：未分类

日期：2024年5月27日 14:55:13

-->

# IRC的简单性 - Susam Pal

> 来源：[https://susam.net/simplicity-of-irc.html](https://susam.net/simplicity-of-irc.html)

<main>

# IRC的简单性

-   作者 **Susam Pal**，2022年1月9日

在与朋友和同事的讨论中，每当聊到聊天协议的话题时，我经常提到互联网中继聊天（IRC）协议的简单性，以及这种简单性如何促进了许多成长于上世纪90年代末和21世纪初的年轻电脑爱好者生活中的创造力。对于我们许多在那个时候初次接触互联网的人来说，编写一个IRC机器人成为了我们的首个非平凡的业余编程项目之一，涉及到网络套接字，执行有意义的任务，并为实际用户提供服务。

## 简单性

IRC服务器和客户端在IRC会话期间交换的底层负载非常容易手动阅读和理解。尽管实现IRC服务器仍然需要大量工作来跟踪用户、频道以及在服务器之间交换网络状态和消息，但实现IRC客户端通常相对简单。借助便捷的编程语言，人们可以相当快速地开发各种有趣的工具和机器人。唯一的限制就是创造力！

在IRC的早期阶段，有基本编程技能的人通常能够在几个小时内编写一个简单的IRC机器人是很常见的。这类IRC机器人通常会响应用户的请求，回答常见问题，举办知识竞赛等。协议的简单性使得直接编写能与IRC服务器交互的程序变得非常吸引人。事实上，许多人选择编写代码来从头开始解析和创建IRC负载。使用像Wireshark或Tcpdump这样的数据包分析器观察TCP/IP数据包就足以了解各种负载格式。此外，[RFC 1459](https://www.rfc-editor.org/rfc/rfc1459)作为学习IRC规范的良好参考。

由于IRC协议的简单性，有时当我需要加入IRC频道，比如寻求一些技术帮助时，我会经常直接在没有安装IRC客户端的系统上启动`telnet`，`nc`或`openssl`连接，然后手动输入IRC协议命令以加入所需的频道并与频道用户交流。

## 会话

为了说明IRC协议的简单性，这里有一个涉及加入频道和发布消息的最小IRC会话示例：

```
$ `nc irc.libera.chat 6667`
:strontium.libera.chat NOTICE * :*** Checking Ident
:strontium.libera.chat NOTICE * :*** Looking up your hostname...
:strontium.libera.chat NOTICE * :*** Couldn't look up your hostname
:strontium.libera.chat NOTICE * :*** No Ident response
`NICK humpty`
`USER humpty humpty irc.libera.chat :Humpty Dumpty`
:strontium.libera.chat 001 humpty :Welcome to the Libera.Chat Internet Relay Chat Network humpty
:strontium.libera.chat 002 humpty :Your host is strontium.libera.chat[204.225.96.123/6667], running version solanum-1.0-dev
:strontium.libera.chat 003 humpty :This server was created Sat Oct 30 2021 at 17:56:22 UTC
:strontium.libera.chat 004 humpty strontium.libera.chat solanum-1.0-dev DGQRSZaghilopsuwz CFILMPQSTbcefgijklmnopqrstuvz bkloveqjfI
:strontium.libera.chat 005 humpty MONITOR=100 CALLERID=g WHOX FNC ETRACE KNOCK SAFELIST ELIST=CMNTU CHANTYPES=# EXCEPTS INVEX CHANMODES=eIbq,k,flj,CFLMPQSTcgimnprstuz :are supported by this server
:strontium.libera.chat 005 humpty CHANLIMIT=#:250 PREFIX=(ov)@+ MAXLIST=bqeI:100 MODES=4 NETWORK=Libera.Chat STATUSMSG=@+ CASEMAPPING=rfc1459 NICKLEN=16 MAXNICKLEN=16 CHANNELLEN=50 TOPICLEN=390 DEAF=D :are supported by this server
:strontium.libera.chat 005 humpty TARGMAX=NAMES:1,LIST:1,KICK:1,WHOIS:1,PRIVMSG:4,NOTICE:4,ACCEPT:,MONITOR: EXTBAN=$,ajrxz :are supported by this server
:strontium.libera.chat 251 humpty :There are 66 users and 48644 invisible on 25 servers
:strontium.libera.chat 252 humpty 35 :IRC Operators online
:strontium.libera.chat 253 humpty 11 :unknown connection(s)
:strontium.libera.chat 254 humpty 21561 :channels formed
:strontium.libera.chat 255 humpty :I have 3117 clients and 1 servers
:strontium.libera.chat 265 humpty 3117 4559 :Current local users 3117, max 4559
:strontium.libera.chat 266 humpty 48710 50463 :Current global users 48710, max 50463
:strontium.libera.chat 250 humpty :Highest connection count: 4560 (4559 clients) (301752 connections received)
:strontium.libera.chat 375 humpty :- strontium.libera.chat Message of the Day -
:strontium.libera.chat 372 humpty :- Welcome to Libera Chat, the IRC network for
:strontium.libera.chat 372 humpty :- free & open-source software and peer directed projects.
:strontium.libera.chat 372 humpty :-
:strontium.libera.chat 372 humpty :- Use of Libera Chat is governed by our network policies.
:strontium.libera.chat 372 humpty :-
:strontium.libera.chat 372 humpty :- To reduce network abuses we perform open proxy checks
:strontium.libera.chat 372 humpty :- on hosts at connection time.
:strontium.libera.chat 372 humpty :-
:strontium.libera.chat 372 humpty :- Please visit us in #libera for questions and support.
:strontium.libera.chat 372 humpty :-
:strontium.libera.chat 372 humpty :- Website and documentation:  https://libera.chat
:strontium.libera.chat 372 humpty :- Webchat:                    https://web.libera.chat
:strontium.libera.chat 372 humpty :- Network policies:           https://libera.chat/policies
:strontium.libera.chat 372 humpty :- Email:                      support@libera.chat
:strontium.libera.chat 376 humpty :End of /MOTD command.
:humpty MODE humpty :+iw
`JOIN #test`
:humpty!~humpty@178.79.176.169 JOIN #test
:strontium.libera.chat 353 humpty = #test :humpty susam coolnickname ptl-tab edcragg
:strontium.libera.chat 366 humpty #test :End of /NAMES list.
`PRIVMSG #test :Hello, World!`
:susam!~susam@user/susam PRIVMSG #test :Hello, Humpty!
`PART #test`
:humpty!~humpty@178.79.176.169 PART #test
`QUIT`
:humpty!~humpty@178.79.176.169 QUIT :Client Quit
ERROR :Closing Link: 178.79.176.169 (Client Quit)

```

在上述会话中，用户使用昵称`humpty`连接到Libera Chat网络，加入了名为`#test`的频道，并发布了一条消息。

请注意，上述会话未加密。按照惯例，IRC端口6667用于明文连接。另有一个端口，如6697端口，用于加密连接。以下是使用OpenSSL命令行工具建立的加密IRC会话示例：

```
$ `openssl s_client -quiet -connect irc.libera.chat:6697 2> /dev/null`
:strontium.libera.chat NOTICE * :*** Checking Ident
:strontium.libera.chat NOTICE * :*** Looking up your hostname...
:strontium.libera.chat NOTICE * :*** Couldn't look up your hostname
:strontium.libera.chat NOTICE * :*** No Ident response
NICK humpty
USER humpty humpty irc.libera.chat :Humpty Dumpty
:strontium.libera.chat 001 humpty :Welcome to the Libera.Chat Internet Relay Chat Network humpty
...

```

省略号表示出于简洁起见而省略的行。会话的其余部分与本文的第一个示例非常相似。

值得注意的是，尽管 IRC 协议的载荷格式非常简单，但一旦开始编写 IRC 客户端，你会遇到一些关于协议的微小细节需要注意，例如，向网络进行身份验证，响应来自服务器的 `PING` 消息以避免 Ping 超时，将消息分割成较短的消息，以确保总载荷不超过512个字符的消息长度限制等。对于一个严肃的 IRC 客户端，依赖于一个能够解决这些问题并准确实现 IRC 规范的合适库显然是很有用的。但对于想要理解协议并编写一些有趣工具的业余爱好者来说，IRC 协议的文本性质和简单性提供了一个富有实验和创造性的肥沃土壤。

## 加入

如果你从未使用过 IRC，但是这篇文章引起了你的兴趣并且想要尝试，你可能不想手动输入 IRC 载荷。你会想要一个好的 IRC 客户端。让我分享一些方便的方法来连接到 Libera Chat 网络。比如，你想要加入 Libera Chat 网络的 `#python` 频道。以下是几种方法：

+   通过 Web 界面加入：[web.libera.chat/#python](https://web.libera.chat/#python)。

+   在 macOS 上，运行 `brew install irssi` 安装 Irssi。在 Debian、Ubuntu 或基于 Debian 的 Linux 系统上，运行 `sudo apt-get install irssi`。然后输入 `irssi -c irc.libera.chat` 连接到 Libera Chat。在 Irssi 中输入 `/join #python` 加入频道。

还有许多其他加入 IRC 网络的方法。有桌面 GUI 客户端、Web 浏览器插件、Emacs 插件、Web 服务、弹跳客户端等，让用户以各种方式连接到 IRC 网络。在 Libera Chat 上，有许多开源项目的频道（`#emacs`、`#linux` 等）、特定主题的社区（`##math`、`#physics` 等）、编程语言（`#c`、`#c++`、`#commonlisp` 等）。输入 `/join` 命令，后跟空格和频道名称，加入频道并开始发布和阅读消息。也可以通过频道名称搜索频道。例如，在 Libera Chat 上，要搜索所有包含 "python" 的频道，输入 IRC 命令：`/msg alis list python`。

尽管上面的例子中使用了 Libera Chat，还有许多其他 IRC 网络，如 EFNet、DALNet、OFTC 等。Libera Chat 是开源项目和主题社区非常流行和活跃的网络之一。我每天都在使用它，所以我选择了它作为这里的例子。在 Libera Chat 上有许多紧密联系的社区。我最喜欢的一些是 `#commonlisp`、`#emacs`、`#python` 等。所有这些都有非常友好和活跃的社区，对初学者有很好的态度。

</main>
