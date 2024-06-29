<!--yml

category: 未分类

date: 2024-05-29 13:19:42

-->

# Fossil：Fossil 同步协议

> 来源：[https://fossil-scm.org/home/doc/trunk/www/sync.wiki](https://fossil-scm.org/home/doc/trunk/www/sync.wiki)

本文描述了用于在两个 Fossil 仓库之间同步内容的传输协议。

## 1.0 概述

一个 Fossil 仓库的全局状态由一个无序的 [文物集合](https://fossil-scm.org/home/doc/trunk/www/fileformat.wiki) 组成。每个文物由其内容的加密哈希标识，以小写十六进制字符串表示。同步是在仓库之间共享文物的过程，以便所有仓库都有所有文物的副本。因为文物是无序的，所以接收文物的顺序不重要。假定文物的哈希名称是唯一的 - 每个文物都有不同的哈希。在第一个近似下，同步通过共享可用文物的哈希列表，然后共享连接的一侧或另一侧缺少的文物内容来进行。在实践中，一个仓库可能包含数百万个文物。这么多文物的哈希名称列表可能很大。因此，通常采用一些优化措施来通常减少需要共享的哈希数到几十个。

每个仓库还具有本地状态。本地状态确定了网页格式化偏好、授权用户、票务格式等各种因仓库而异的信息。通常在同步过程中不传输本地状态。不过，在 [clone](https://fossil-scm.org/home/help?cmd=clone) 过程中会传输一些本地状态，以初始化新仓库的本地状态。此外，管理员可以使用 [config push](https://fossil-scm.org/home/help?cmd=configuration) 和 [config pull](https://fossil-scm.org/home/help?cmd=configuration) 命令同步本地状态。

### 1.1 无冲突复制数据类型

Fossil 使用的“文物包”数据模型显然是特定 [无冲突复制数据类型 (CRDT)](https://en.wikipedia.org/wiki/Conflict-free_replicated_data_type) 的实现，称为“G-Set”或“Grow-only Set”。关于 CRDT 的学术文献大约在 2011 年左右开始出现，而 Fossil 的研究至少早于此四年。但令人高兴的是，理论家们现在已经证明，Fossil 的基础数据模型可以通过仅使用对等通信而不需要任何中央权威来提供强一致性副本。

如果你已经熟悉 CRDT 并且想知道 Fossil 是否使用了它们，答案是“是”。我们只是不以那个名称称呼它们。

## 2.0 传输

客户端和服务器之间的所有通信都通过 HTTP 请求进行。服务器正在监听传入的 HTTP 请求。客户端发出一个或多个 HTTP 请求，并收到每个请求的回复。

服务器可能作为独立服务器运行，使用["fossil server"命令](https://fossil-scm.org/home/help?cmd=server)，或者可能是从inetd或xinetd启动，使用[/help?cmd=http](https://fossil-scm.org/home/wiki?name=%22fossil+http%22+command)。或者服务器可能是从[CGI启动](https://fossil-scm.org/home/doc/trunk/www/server/any/cgi.md)或者从[SCGI启动](https://fossil-scm.org/home/doc/trunk/www/server/any/scgi.md)。（详见"[如何配置Fossil服务器](https://fossil-scm.org/home/doc/trunk/www/server/)"了解详情。）服务器监听传入HTTP请求的具体方式对此协议无关紧要。重要的是服务器正在监听请求，而客户端是请求的发起者。

一次 [推送](https://fossil-scm.org/home/help?cmd=push)，[拉取](https://fossil-scm.org/home/help?cmd=pull)，或者 [同步](https://fossil-scm.org/home/help?cmd=sync) 可能涉及多个HTTP请求。客户端在所有请求之间保持状态。但在服务器端，每个请求是独立的。服务器不会保留关于客户端的任何信息，从一个请求到下一个请求。

注意：在本文中，我们使用术语"服务器"和"客户端"分别表示交互的监听器和发起者。此协议并不要求服务器实际上是数据中心中的后台处理器，也不要求客户端是台式机或手持设备。在本文中，"客户端"仅表示启动对话的存储库，"服务器"是响应的存储库。没有其他含义。

#### 2.0.1 HTTPS传输

HTTPS与HTTP的唯一区别在于HTTPS协议在传输过程中进行了加密。底层协议是相同的。本文档仅描述了客户端和服务器之间以及返回客户端的基础未加密消息。这些消息是否加密在本文档中并不重要。

Fossil在客户端和服务器中都包含内置的[支持HTTPS加密](https://fossil-scm.org/home/doc/trunk/www/ssl-server.md)。

#### 2.0.2 SSH传输

当使用"`ssh:…`" URL进行同步时，使用相同的HTTP传输协议。Fossil仅使用[ssh](https://en.wikipedia.org/wiki/Secure_Shell)在远程机器上启动一个[fossil test-http](https://fossil-scm.org/home/help?cmd=test-http)命令的实例。然后通过SSH连接发送HTTP请求并获取HTTP回复，而不是通过互联网套接字发送和接收。要查看Fossil客户端用于建立连接的具体"ssh"命令，请在"fossil sync"命令行中添加"--httptrace"或"--sshtrace"选项。

此方法依赖于由 SSH 守护进程设置的远程 `PATH`，可能与同一服务器上交互式 shell 的 `PATH` 不同。例如，在后者中常见的 `$HOME/bin` 在前者中可能找不到，导致在非标准位置安装 `fossil` 二进制文件时同步 `ssh:…` URL 失败，例如

```
./configure --prefix=$HOME && make install
```

解决此问题的两种较简单的解决方案之一是在 SSH 守护进程期望找到 Fossil 的地方安装 Fossil，但当这不是选项时，您可以给出类似以下的 URL：

```
fossil clone ssh://myserver.example.com/path/to/repo.fossil?fossil=/home/me/bin/fossil
```

这样可以为本地的 Fossil 实例提供远程机器上二进制文件的绝对路径，以便通过 SSH 隧道调用该 Fossil 实例。

#### 2.0.3 文件传输

当使用 "file:..." URL 进行同步时，仍然使用相同的 HTTP 协议。但是，不再通过套接字或 SSH 发送每个 HTTP 请求，而是将 HTTP 请求写入临时文件中。然后客户端在子进程中调用 [化石 http](https://fossil-scm.org/home/help?cmd=http) 命令处理请求并生成响应。客户端接着从磁盘上的临时文件读取 HTTP 响应，并删除这两个临时文件。若想查看实现 "file:" 传输的具体 "化石 http" 命令，请在 "fossil sync" 命令中添加 "--httptrace" 选项。

### 2.1 服务器标识

服务器由伴随客户端推送、拉取或同步命令的 URL 参数标识。（为方便用户，客户端命令中可以省略 URL，并重复使用最近一次推送、拉取或同步的相同 URL。在客户端多次同步到同一服务器的常见情况下，这样可以节省输入。）

客户端通过在末尾添加方法名 "**/xfer**" 来修改 URL。例如，如果在客户端命令行上指定的 URL 是

```
https://fossil-scm.org/fossil
```

然后用于执行同步的真正 URL 将是：

```
https://fossil-scm.org/fossil/xfer
```

### 2.2 HTTP 请求格式

客户端始终向服务器发送 POST 请求。POST 请求的一般格式如下：

```
POST /fossil/xfer HTTP/1.0
Host: fossil-scm.hwaci.com:80
Content-Type: application/x-fossil
Content-Length: 4216

```

```
*content...*
```

在上述示例中，第一行的 POST 关键字后给出的路径名是 URL 路径名的副本。Host 参数也取自 URL。内容类型始终为 "application/x-fossil" 或 "application/x-fossil-debug"。 "x-fossil" 内容类型是默认的。唯一的区别是 "x-fossil" 内容使用 zlib 压缩，而 "x-fossil-debug" 则未压缩发送。

服务器的典型响应可能如下所示：

```
HTTP/1.0 200 OK
Date: Mon, 10 Sep 2007 12:21:01 GMT
Connection: close
Cache-control: private
Content-Type: application/x-fossil; charset=US-ASCII
Content-Length: 265

```

```
*content...*
```

响应的内容类型始终与请求的内容类型相同。

## 3.0 化石同步内容

客户端和服务器之间的同步请求由一个或多个 HTTP 请求组成，如前一部分所述。本节详细描述了 "x-fossil" 内容类型。

### 3.1 行定向格式

x-fossil内容类型由零个或多个“卡”组成。卡由换行符（"\n"）分隔。卡上的前导和尾随空白将被忽略。空白卡将被忽略。

每张卡被分成零个或更多个以空格分隔的标记。每张卡上的第一个标记是操作符。后续标记是参数。服务器理解的操作符集与客户端理解的略有不同，尽管两者非常相似。

### 3.2 登录卡

每个从客户端到服务器的消息都以一个或多个登录卡开头。每个登录卡的格式如下：

```
login  *userid  nonce  signature*
```

用户ID是请求服务器服务的用户名称。Nonce是消息剩余部分的SHA1哈希 - 所有跟随终止登录卡的换行符的文本。签名是Nonce和用户密码连接的SHA1哈希。

对于每个登录卡，服务器查找用户并验证nonce是否与消息剩余部分的SHA1哈希匹配。然后检查签名哈希以确保签名匹配。如果一切正常，则客户端被授予指定用户的所有权限。

权限是累积的。可以有多个成功的登录卡。会话权限是所有登录卡中所有权限的并集。

### 3.3 文件卡

通过“file”卡、或“cfile”或“uvfile”卡传输工件。名称“file”卡源于大多数工件对应于版本控制下的文件。名称“cfile”是“压缩文件”的缩写。名称“uvfile”是“未版本化文件”的缩写。

#### 3.3.1 普通文件卡

对于同步协议，使用“file”卡传输工件。文件卡有两种不同的格式，取决于工件是直接发送还是作为来自其他工件的[增量](https://fossil-scm.org/home/doc/trunk/www/delta_format.wiki)。

```
file *artifact-id size* \n *content*
file *artifact-id delta-artifact-id size* \n *content*

```

文件卡后面是内联的“有效载荷”数据。工件或工件增量的内容是紧随终止文件卡的换行符后的x-fossil内容的前 *size* 字节。

文件卡的第一个参数是正在传输的工件的ID。工件ID是工件名称哈希的小写十六进制表示。文件卡的最后一个参数是紧随文件卡之后的有效载荷字节数。如果文件卡只有两个参数，这意味着有效载荷是工件的完整内容。如果文件卡有三个参数，则有效载荷是一个[增量](https://fossil-scm.org/home/doc/trunk/www/delta_format.wiki)，第二个参数是另一个工件的ID，它是增量的来源。

文件卡片在双向发送：客户端到服务器和服务器到客户端。可能会在增量的源之前发送增量，因此客户端和服务器都应该记住增量，并且能够在源到达时应用它们。

#### 3.3.2 压缩文件卡片

当客户端发送克隆协议版本“3”或更高版本时，将在克隆过程中作为“cfile”卡片接收工件。为了通过直接从服务器数据库向客户端发送压缩工件来提高内容传输速度而引入了这张卡片。

压缩文件卡片与文件卡片类似，共享相同的内联“payload”数据特性，并且也采用直接内容或增量内容的相同处理方式。Cfile卡片有两种不同的格式，取决于工件是直接发送还是作为其他工件的增量发送。

```
cfile *artifact-id usize csize* \n *content*
cfile *artifact-id delta-artifact-id usize csize* \n *content*

```

cfile卡片的第一个参数是正在传输的工件的ID。工件ID是工件名称哈希的小写十六进制表示。cfile卡片的第二个参数是工件的原始大小（以字节为单位）。cfile卡片的最后一个参数是紧随其后的有效负载的压缩字节数。如果cfile卡片只有三个参数，这意味着有效负载是工件的完整内容。如果cfile卡片有四个参数，则有效负载是一个增量，第二个参数是另一个工件的ID，该工件是增量的源，第三个参数是增量工件的原始大小。

与文件卡片不同，克隆协议版本为“3”或更高版本时，cfile卡片仅在服务器到客户端的克隆期间单向发送。

#### 3.3.3 私有工件

“私有”内容包括通常不同步的工件。但是，当 [fossil sync](https://fossil-scm.org/home/help?cmd=sync) 命令包含“--private”选项时，私有内容将被同步。

私有内容由“private”卡片标记：

```
private
```

私有卡片没有参数，并且必须直接在包含私有内容的文件卡片之前出现。

#### 3.3.4 未版本化文件卡片

未版本化内容在双向（客户端到服务器和服务器到客户端）使用“uvfile”卡片以以下格式发送：

```
uvfile *name mtime hash size flags* \n *content*
```

*name* 字段是未版本化文件的名称。*mtime* 是文件自1970年以来的最后修改时间（秒）。*hash* 字段是未版本化文件内容的哈希值，或者对于删除的内容是 "**-**"。*size* 字段是内容的（未压缩）字节大小。*flags* 字段是一个整数，被解释为位数组。*flags* 的0x0004位表示要省略*content*。如果内容太大而无法传输，或者发送方仅想要更新文件的修改时间而不更改文件内容，则可能会省略内容。*content* 是文件的（未压缩）内容。

接收方只应在哈希和大小与内容匹配且mtime更新于接收方保留的同一文件的任何现有实例时接受uvfile卡片。发送方通常不会传输uvfile卡片，除非所有这些约束条件为真，但接收方应再次检查。

只有登录用户具有"y"写入未版本化权限时，服务器才会接受uvfile卡片。

服务器响应从客户端收到的uvgimme卡片而发送uvfile卡片。客户端在根据先前从服务器收到的uvigot卡片确定服务器需要内容时发送uvfile卡片。

### 3.4 推送和拉取卡片

客户端到服务器消息的第一张卡片包括推送和拉取卡片。推送卡片告知服务器客户端正在推送内容。拉取卡片告知服务器客户端希望拉取内容。在同步事件中，这两张卡片都会被发送。格式如下：

```
push *servercode projectcode*
pull *servercode projectcode*

```

*servercode*参数是客户端的仓库ID。*projectcode*是客户端仓库包含的软件项目的标识符。为了使事务继续进行，客户端和服务器的projectcode必须匹配。

在克隆过程中，服务器还会向客户端发送推送卡片。这是客户端确定新建立的仓库中应放置的项目代码的方式。

*servercode*参数目前未使用。

### 3.5 克隆卡片

克隆卡片类似于拉取卡片，以便从客户端发送到服务器，告知服务器客户端希望拉取内容。克隆卡片有两种格式。旧版本客户端使用无参数格式，而新版本客户端使用两参数格式。

```
clone
clone *protocol-version sequence-number*

```

#### 3.5.1 协议 3

最新的客户端发送一个带有协议版本为"3"的两参数克隆消息。（未来版本的Fossil可能会使用更大的协议版本号。）协议版本"3"通过引入旨在加速克隆操作的"cfile"卡片来增强了版本"2"。服务器将发送"cfile"卡片而不是"file"卡片。

#### 3.5.2 协议 2

发送的序列号是迄今为止接收到的工件数量。对于第一个克隆消息，序列号为0。服务器将响应并发送文件卡片，数量最多达到最大消息大小。

服务器还会向客户端发送单个"clone_seqno"卡片，以便客户端知道服务器停止的位置。

```
clone_seqno  *sequence-number*

```

后续HTTP请求中的克隆消息将使用前一回复的clone_seqno的序列号。

响应初始克隆消息时，服务器还向客户端发送推送消息，以便客户端发现此项目的projectcode。

#### 3.5.3 旧协议

旧版本的客户端发送一个空的克隆卡片。服务器对空的克隆卡片的响应是向仓库中的每个工件发送一个 "igot" 卡片。然后客户端会发出 "gimme" 卡片来拉取其需要的所有内容。

旧协议对于较小的仓库（50MB，含有 50,000 个工件）效果良好，但对于较大的仓库则太慢且笨重。版本 2 协议是为了提高性能而做出的努力。在 Fossil 的未来版本中，更高编号的克隆协议可能会进一步提升性能。

### 3.6 Igot 卡片

igot 卡片可以从客户端发送到服务器，也可以从服务器发送到客户端，以指示发送方持有特定工件的副本。其格式如下：

```
igot *artifact-id* ?*flag*?

```

igot 卡片的第一个参数是发送方拥有的工件的 ID。接收到 igot 卡片的接收方通常会检查是否也持有相同的工件，如果没有，则会在回复或下一个消息中使用 gimme 卡片请求该工件。

如果第二个参数存在且为 "1"，则由第一个参数标识的工件在发送方是私有的，并且应该被忽略，除非正在进行 "--private" [同步](https://fossil-scm.org/home/help?cmd=sync)。

名称 "igot" 来自于英语俚语表达 "I got"，意思是 "我有"。

#### 3.6.1 未版本化的 Igot 卡片

在同步未版本化内容时，服务器向客户端发送零个或多个 "uvigot" 卡片。uvigot 卡片的格式如下：

```
uvigot *name mtime hash size*

```

*name* 参数是未版本化文件的名称。*mtime* 是自 1970 年以来未版本化文件的最后修改时间（秒）。*hash* 是未版本化文件内容的 SHA1 或 SHA3-256 哈希值，如果文件已删除则为 "**-**"。*size* 是文件未压缩的大小（字节）。

当服务器看到 "pragma uv-hash" 卡片的哈希值不匹配时，它会为其持有的每个未版本化文件发送 uvigot 卡片。客户端将使用此信息来确定需要同步哪些未版本化文件。当服务器收到 uvgimme 卡片但其回复消息大小已经超出正常的 uvfile 回复大小时，也可能会发送 uvigot 卡片。

当客户端收到一个 "uvigot" 卡片时，它会检查文件是否需要从客户端传输到服务器或从服务器传输到客户端。如果需要从客户端到服务器的传输，客户端会安排在后续的 HTTP 请求中进行传输。如果需要从服务器到客户端的传输，则客户端会向服务器发送一个 "uvgimme" 卡片以请求文件内容。

### 3.7 Gimme 卡片

gimme 卡片可以从客户端发送到服务器，也可以从服务器发送到客户端。gimme 卡片要求接收方将特定的工件发送回发送方。gimme 卡片的格式如下：

```
gimme *artifact-id*

```

gimme 卡片的参数是发送方想要的工件的 ID。接收方通常会通过其回复或下一条消息中的文件卡片来响应 gimme 卡片。

"gimme" 名称意为 "给我"。在一些英语方言（包括 Fossil 的原始作者所使用的方言）中，这个祈使句 "give me" 的发音就像一个单词 "gimme"。

#### 3.7.1 无版本 Gimme 卡片

同步非版本化内容时，客户端可以向服务器发送 "uvgimme" 卡片。uvgimme 卡片请求服务器向客户端发送非版本化内容。uvgimme 卡片的格式如下：

```
uvgimme *name*

```

*名称* 是客户端希望获取的服务器上找到的非版本化文件的名称。当服务器看到 uvgimme 卡片时，通常会用 uvfile 卡片回应，尽管如果 HTTP 回复已经超出大小限制，也可能发送另一个 uvigot 卡片。

### 3.8 Cookie 卡片

服务器可以使用 cookie 卡片在客户端上记录少量状态信息。服务器向客户端发送 cookie。客户端在下一次请求时将相同的 cookie 发送回服务器。cookie 卡片有一个参数，即其有效载荷。

```
cookie *payload*

```

客户端不需要在下一次请求时将 cookie 返回给服务器。或者客户端可能在下一次请求时发送来自另一个服务器的 cookie。因此，服务器不应依赖 cookie，并且服务器必须以这样的方式结构化 cookie 负载，以便它可以确定看到的 cookie 是自己的还是来自另一个服务器的。（通常，服务器将其服务器代码嵌入 cookie 的一部分。）

### 3.9 请求配置卡片

请求配置或者 "reqconfig" 卡片是客户端发送给服务器的请求，以请求服务器返回 "配置" 数据。 "配置" 数据是关于用户或者网站外观或其他管理细节的信息，这些信息不是项目的持久和有版本状态的一部分。例如，项目的 "名称"，web 界面的默认层叠样式表（CSS），以及显示在 web 界面上的项目 logo 都属于配置数据元素。

reqconfig 卡片通常作为响应 "fossil configuration pull" 命令而发送。格式如下：

```
reqconfig *configuration-name*

```

截至 2018-06-04，配置名称必须是以下值之一：

|

+   css

+   header

+   footer

+   details

+   logo-mimetype

+   logo-image

+   background-mimetype

+   background-image

+   index-page

+   timeline-block-markup

+   timeline-max-comment

+   timeline-plaintext

+   adunit

+   adunit-omit-if-admin

+   adunit-omit-if-user

|

+   th1-docs

+   th1-hooks

+   th1-setup

+   tcl

+   tcl-setup

+   project-name

+   short-project-name

+   project-description

+   index-page

+   manifest

+   binary-glob

+   clean-glob

+   ignore-glob

+   keep-glob

+   crlf-glob

|

+   crnl-glob

+   encoding-glob

+   empty-dirs

+   ~~allow-symlinks~~

+   dotfiles

+   parent-project-code

+   parent-projet-name

+   hash-policy

+   mv-rm-files

+   ticket-table

+   ticket-common

+   ticket-change

+   ticket-newpage

+   ticket-viewpage

+   ticket-editpage

|

+   ticket-reportlist

+   ticket-report-template

+   ticket-key-template

+   ticket-title-expr

+   ticket-closed-expr

+   xfer-common-script

+   xfer-push-script

+   xfer-commit-script

+   xfer-ticket-script

+   @reportfmt

+   @user

+   @concealed

+   @shun

|

新的配置名称可能会在未来的 Fossil 发布版本中添加。如果服务器收到了它不理解的配置名称，则整个 reqconfig 卡将被静默地忽略。如果用户缺乏足够的权限访问请求的信息，则也可能会忽略 reqconfig 卡。

以字母开头的配置名称引用服务器数据库中“config”表中的值。例如，“logo-image”配置项指的是在 [web-interface](https://fossil-scm.org/home/doc/trunk/www/webui.wiki) 的管理页面上配置的项目标志图像。配置项的值通过“config”卡返回给客户端。

如果配置名称以“@”开头，则表示它引用值的一个类别，而不是单个值。这些配置项的内容在包含纯 SQL 文本的“config”卡中返回，供客户端评估。

@user 和 @concealed 配置项包含敏感信息，对于没有足够权限的客户端会被忽略。

### 3.10 配置卡

“config” 卡用于从客户端向服务器发送配置信息（响应于“fossil configuration push”命令）或从服务器向客户端发送配置信息（响应于“fossil configuration pull”或“fossil clone”命令）。格式如下：

```
config *configuration-name size* \n *content*

```

只有具有“Admin”权限的用户才能接受配置卡。客户端只有在其请求中发送了相应的 reqconfig 卡时，才能接受配置卡。

配置项的内容用于覆盖接收方中相应的配置数据。

### 3.11 编译指示卡

客户端可以通过发出编译指示卡来影响服务器的行为：

```
pragma *name value...* 
```

“pragma” 卡至少有一个参数，即编译指示名称。编译指示名称定义编译指示的作用。根据编译指示名称的不同，编译指示可能具有零个或多个“值”参数。

不时地可能会向 Fossil 协议中添加新的编译指示符名称，以增强其功能。未知的编译指示符会被静默地忽略，以保持向后兼容性。

截至 2019-06-30，以下是已知的编译指示名称：

1.  **send-private** 发送私有编译指示指示服务器向客户端发送其所有私有工件。服务器仅在用户拥有“x”或“Private”权限时才会遵循此请求。

1.  **send-catalog** 发送目录编译指示指示服务器为每个已知工件传输 igot 卡。这可以帮助客户端和服务器在之前的协议错误后重新同步。“--verily”选项可通过 [fossil sync](https://fossil-scm.org/home/help?cmd=sync) 命令传输 send-catalog 编译指示。

1.  **uv-hash** *HASH* 客户端向服务器发送`uv-hash` pragma以触发未版本化内容的同步。*HASH* 是客户端所有未版本化文件的名称、修改时间和各自哈希值的SHA1哈希值。如果客户端的未版本化内容哈希与服务器上的未版本化内容哈希不匹配，那么服务器将回复一个"pragma uv-push-ok"或"pragma uv-pull-only"卡片，后跟每个服务器当前持有的未版本化文件的一个"uvigot"卡片。作为对"uv-hash" pragma的响应发送的"uvigot"卡片的集合称为"未版本化目录"。客户端将使用未版本化目录来确定需要在客户端和服务器之间同步的文件（如果有的话），并在下一个HTTP请求中发送适当的"uvfile"或"uvgimme"卡片。

    如果客户端发送了一个`uv-hash` pragma，并且没有收到`uv-pull-only`或者`uv-push-ok` pragma的回复，这意味着服务器上的内容与客户端完全匹配，不需要进一步的同步操作。

1.  **uv-pull-only** 服务器在响应具有不匹配内容哈希参数的`uv-hash` pragma时向客户端发送`uv-pull-only` pragma。此pragma表示客户端和服务器之间存在未版本化内容的差异，但只能从服务器传输内容到客户端。服务器不愿接受客户端的内容，因为客户端登录缺少"write-unversioned"权限。

1.  **uv-push-ok** 服务器在响应具有不匹配内容哈希参数的`uv-hash` pragma时向客户端发送`uv-push-ok` pragma。此pragma表示客户端和服务器之间存在未版本化内容的差异，并且可以在两个方向上传输内容。服务器愿意接受客户端的内容，因为客户端登录具有"write-unversioned"权限。

1.  **ci-lock** *CHECKIN-HASH CLIENT-ID* 客户端向服务器发送"ci-lock" pragma，表示它即将将一个新的检入添加为CHECKIN-HASH检入的子检入，并且与CHECKIN-HASH在同一个分支上。如果其他某个客户端已经指示它也试图对CHECKIN-HASH进行提交，这表示即将发生分叉，服务器将以"ci-lock-fail" pragma回复（见下文）。检入锁定将在检入实际发生后或超时（目前为一分钟，但可能会更改）后自动失效。

1.  **ci-lock-fail** *LOGIN MTIME* 当服务器收到两个或更多针对相同检入的"ci-lock" pragma消息，但来自不同的客户端时，第二个及后续的ci-lock将在回复中引发一个"ci-lock-fail" pragma，以告知客户端如果继续进行检入，可能会生成一个分叉。LOGIN和MTIME参数旨在为客户端提供信息，帮助其生成一个更有用的错误消息。

1.  **ci-unlock** *CLIENT-ID* 客户端在成功提交后向服务器发送“ci-unlock”指令。这指示服务器释放该客户端先前持有的任何检查点的锁定。ci-unlock指令有助于避免如果检查点被中止然后在分支上重新启动可能会引发的错误正锁定警告。

任何以"#"（ASCII 0x23）开头的卡片是注释卡片，会被静默忽略。

### 3.13 消息和错误卡片

如果服务器发现请求存在问题，它会在回复中生成错误卡片。客户端看到错误卡片时，会向用户显示错误消息并中止同步操作。错误卡片的格式如下：

```
error *error-message*

```

错误消息是编码为单个令牌的英文文本。空格（ASCII 0x20）表示为"\s"（ASCII 0x5C, 0x73）。换行符（ASCII 0x0a）表示为"\n"（ASCII 0x6C, x6E）。反斜杠（ASCII 0x5C）表示为两个反斜杠 "\\"。除了空格和换行符外，错误消息中不允许其他空白字符或不可打印字符。

服务器还可以发送消息卡片，该卡片在客户端控制台上打印消息，但不是错误：

```
message *message-text*

```

消息文本使用与错误消息相同的格式。

### 3.14 未知卡片

如果客户端或服务器看到不符合上述描述的卡片，则会生成错误并中止操作。

## 4.0 幻影和聚类

当存储库知道某个工件存在并知道该工件的ID，但不知道工件内容时，它将该工件存储为“幻影”。存储库通常在接收到一个不持有的工件的igot卡片或引用不持有的增量源的文件卡片时创建幻影。当服务器生成其回复或客户端生成新请求时，通常会为其持有的每个幻影发送gimme卡片。

聚类是一种特殊的工件，用于说明其他工件的存在。符合聚类的语法规则的存储库中的任何工件都被视为聚类。

聚类是以行为单位的。聚类的每一行是一张卡片。卡片之间由换行符（"\n"）分隔。每张卡片由单个字符的卡片类型、一个空格和一个参数组成。不允许额外的空白字符，也不允许尾随或前导空白。所有聚类中的卡片必须严格按字典顺序排列。

一个聚类包含一个或多个"M"卡片，后跟一个单一的"Z"卡片。每个M卡片包含一个参数，该参数是存储库中工件的工件ID。Z卡片有一个参数，该参数是所有前面的M卡片（包括前面的换行符）的MD5校验和的小写十六进制表示。

任何不完全符合集群规格的工件都不是集群。工件中不能有额外的空白。必须有一个或多个 M 卡片。必须有一个带有正确 MD5 校验和的单个 Z 卡片。并且所有卡片必须严格按字典顺序排列。

### 4.1 未集群表

每个存储库维护一个名为“**未集群**”的表，记录其持有但未在集群中提到的每个工件和幻影的身份。未集群表中的条目可以被视为工件树上的叶子。未集群的某些工件将是其他集群。这些集群可能包含其他集群，这些集群可能还包含更多的集群，依此类推。从未集群表中的工件开始，可以跟随集群链找到存储库中的每个工件。

## 5.0 同步策略

### 5.1 拉取

典型的拉取操作如下所示。实际实现的细节可能会略有不同，但拉取的主要步骤如下所示：

1.  客户端发送登录和拉取卡片。

1.  客户端如果之前收到过 cookie，就会发送一个 cookie 卡片。

1.  客户端为其持有的每个幻影发送 gimme 卡片。

    * * *

1.  服务器检查登录密码，如果用户没有拉取权限，则拒绝会话。

1.  如果服务器未集群表中的条目数大于 100，则服务器会构造一个新的集群工件来覆盖所有这些未集群的条目。

1.  服务器为从客户端收到的每个 gimme 卡片发送文件卡片。

1.  服务器为其未集群表中的每个非幻影工件发送 igot 卡片。

    * * *

1.  客户端将文件卡片的内容添加到其存储库中。

1.  客户端为服务器回复中提到但客户端不拥有的工件创建幻影。

1.  当 delta 源是客户端不拥有的工件时，客户端为文件卡片的 delta 源创建一个幻影。

这十个步骤代表一个单独的 HTTP 往返请求。前三个步骤是客户端生成请求时发生的处理。中间四个步骤是服务器处理请求和生成回复时发生的处理。最后三个步骤是客户端解释回复时发生的处理。

在拉取过程中，客户端将持续发送 HTTP 请求，直到它拥有服务器上存在的所有工件。

请注意，服务器会尝试将其回复消息的大小限制在一个合理的范围内（通常约为 1MB），以便在回复变得过大时，可能会像第（6）步描述的那样停止发送文件卡片。

第（5）步是创建新集群的唯一方法。通过仅在服务器上创建集群，我们希望在单个服务器和多个客户端的常见配置中最小化集群之间的重叠量。即使存在多个服务器或服务器和客户端有时改变角色，相同的同步协议仍将继续工作。这些不寻常安排的唯一负面影响是可能生成多于最小数量的集群。

### 5.2 推送

典型的推送操作大致如下所示。与拉取一样，实际实施可能略有不同。

1.  客户端发送登录和推送卡。

1.  客户端为其持有的从本地签入获得的尚未推送过的构件发送文件卡。

1.  如果这是推送中的第二轮或更晚的轮次，则客户端会发送任何服务器在上一个周期发送的送我卡的文件卡。

1.  客户端为其未集群化表中的每个构件发送送我卡，这些构件不是虚拟的。

    * * *

1.  服务器检查登录和推送卡，如果有任何问题则发出错误。

1.  服务器接受来自客户端的文件卡，并将这些构件添加到其代码库中。

1.  服务器为提及其不拥有的构件或提及其不拥有的增量源构件的文件卡创建虚拟。

1.  服务器为所有虚拟发出送我卡。

    * * *

1.  客户端记住了来自服务器的送我卡，以便在下一个周期中可以回复文件卡。

与拉取一样，推送操作的步骤会重复，直到服务器了解客户端上存在的所有构件。此外，与拉取一样，一旦请求大小达到1MB，客户端会通过抑制文件卡来尝试保持请求的大小不要增长太大。

### 5.3 同步

同步只是同时进行的拉取和推送。拉取的前三个步骤与推送的前五个步骤结合在一起。拉取的第（4）到（7）步与推送的第（5）到（8）步结合在一起。拉取的第（8）到（10）步与推送的第（9）步结合在一起。

### 5.4 未版本化文件同步

"未版本化文件"是仓库中仅保留文件的最新版本而不是整个更改历史的文件。未版本化文件用于存储临时内容，例如最新发布的编译二进制文件。

未版本化文件由名称和时间戳（mtime）标识。仅保留每个文件的最新版本（具有最大mtime值的版本）。

使用[fossil未版本化同步](https://fossil-scm.org/home/help?cmd=unversioned)命令来同步未版本化文件。

一个未版本化文件同步的示意图如下：

1.  客户端向服务器发送 "pragma uv-hash" 卡片。uv-hash pragma 的参数是客户端持有的所有未版本化文件的文件名、修改时间和内容哈希的哈希值。

    * * *

1.  如果客户端的未版本化内容哈希与服务器上的未版本化内容哈希匹配，则服务器不需要执行任何操作（no-op）。但如果哈希不同，则服务器会回复 uv-pull-only 或 uv-push-ok 指令，接着为服务器上所有未版本化文件发送 uvigot 卡片。

    * * *

1.  客户端检查从服务器接收到的 uvigot 卡片，并确定需要交换的未版本化文件，以使客户端和服务器同步。然后客户端将适当的 "uvgimme" 或 "uvfile" 卡片发送回服务器。

    * * *

1.  服务器通过接收的 "uvfile" 卡片更新其未版本化文件存储，并在回复中的 "uvgimme" 卡片中以 "uvfile" 卡片作答。

如果有更多的未版本化内容需要传输，而一次 HTTP 请求无法轻松完成，则最后两个步骤可能会重复多次。

## 6.0 总结

这里是同步协议的关键点：

1.  客户端向服务器发送一个或多个 PUSH HTTP 请求。请求和回复的内容类型为 "application/x-fossil"。

1.  HTTP 请求内容使用 zlib 进行压缩。

1.  请求和回复的内容包含一行一张卡片。

1.  卡片格式为：

    +   **登录** *userid nonce signature*

    +   **推送** *servercode projectcode*

    +   **拉取** *servercode projectcode*

    +   **克隆**

    +   **clone_seqno** *sequence-number*

    +   **文件** *artifact-id size* **\n** *content*

    +   **文件** *artifact-id delta-artifact-id size* **\n** *content*

    +   **cfile** *artifact-id size* **\n** *content*

    +   **cfile** *artifact-id delta-artifact-id size* **\n** *content*

    +   **uvfile** *name mtime hash size flags* **\n** *content*

    +   **私有**

    +   **igot** *artifact-id* ?*flag*?

    +   **uvigot** *name mtime hash size*

    +   **gimme** *artifact-id*

    +   **uvgimme** *name*

    +   **cookie** *cookie-text*

    +   **reqconfig** *parameter-name*

    +   **config** *parameter-name size* **\n** *content*

    +   **pragma** *name* *value...*

    +   **错误** *error-message*

    +   **消息** *text-messate*

    +   **#** *arbitrary-text...*

1.  幻影是仓库知道存在但并未拥有的构件。

1.  集群是包含其他构件 ID 的构件。

1.  在拉取期间服务器会自动创建集群。

1.  仓库跟踪所有未在任何集群中命名的所有构件，并为这些构件发送 igot 消息。

1.  仓库跟踪其持有的所有幻影并为这些构件发送 gimme 消息。

## 7.0 故障排除和调试提示

如果您运行 [fossil sync](https://fossil-scm.org/home/help?cmd=sync) 命令（或 [pull](https://fossil-scm.org/home/help?cmd=pull) 或 [push](https://fossil-scm.org/home/help?cmd=push) 或 [clone](https://fossil-scm.org/home/help?cmd=clone)），并使用 --httptrace 选项，Fossil 将在名为的文件中保留每个 HTTP 请求和回复的副本：

+   `http-request-`*N*`.txt`

+   `http-reply-`*N*`.txt`

在上述情况中，*N* 是一个整数，每次往返增加。如果你在服务器端遇到问题，可以在调试器中使用一个 "http-request-N.txt" 文件运行 "[fossil test-http](https://fossil-scm.org/home/help?cmd=test-http)" 命令，并逐步执行服务器执行的处理。

"--transport-command CMD" 选项在 [fossil sync](https://fossil-scm.org/home/help?cmd=sync)（和类似命令）中导致外部程序 "CMD" 用于将同步消息移动到服务器并检索同步回复。CMD 被给予三个参数：

1.  服务器的 URL

1.  一个临时文件的名称，其中包含输出绑定的同步协议文本，带有 HTTP 头部

1.  一个临时文件的名称，用于 CMD 写入回复的同步协议文本，同样不包含任何 HTTP 头部

在复杂的调试情况下，你可以运行命令 "fossil sync --transport-command ./debugging_script"，其中 "debugging_script" 是你自己的脚本，用于调用你试图调试的异常行为。
