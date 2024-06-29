<!--yml

category: 未分类

date: 2024-05-29 12:35:09

-->

# 纯文本电子邮件

> 来源：[https://notes.ghed.in/posts/2024/plain-text-email/](https://notes.ghed.in/posts/2024/plain-text-email/)

22/3/2024

# 纯文本电子邮件

在1990年代中期，已经上网的人们的电子邮件收件箱里发生了一场战争。就在这个时候，HTML电子邮件问世，引发了BBS、邮件列表和IRC频道中激烈的讨论。

很可能大多数人 —— 也许，你们 —— 不知道我在说什么。让我们回到原点，这样我们就可以达成一致。

电子邮件可以以纯文本（`text/plain`）的形式发送，就像在记事本中保存文件一样，或者以HTML（`text/html`）的形式发送。在后一种格式（或者[“MIME类型”](https://developer.mozilla.org/en-US/docs/Web/HTTP/Basics_of_HTTP/MIME_Types)）中，邮件被创建得就像是网站的页面，这打开了一个潘多拉的盒子，我是说…许多可能性，比如丰富的格式和图片与文本混合。

HTML电子邮件有[一些明显的缺点](https://subversion.american.edu/aisaac/notes/htmlmail.htm)，比如由于隐藏链接和加载远程媒体而导致的安全性较低。一个偶发的问题是，不像网页浏览器，电子邮件客户端/应用程序不遵循网页标准 —— 每个都以不同方式呈现HTML，这使得例如设计通讯时报版面布局成为一项困难的事情。

HTML的另一个问题是，采用这种格式的消息更重，因为它们有看不见的部分（头部和HTML代码本身）和可见的部分（特别是图片），而纯文本对应件则没有。

如今，由于无处不在的快速互联网连接，这可能不再是一个问题。在20世纪90年代，由于拨号上网的极其缓慢的连接，这一点很重要。

[Mauricio Teixeira](https://hachyderm.io/@badnetmask)，当时是巴西贝伦（PA）ISP的雇员，如此愤怒，以至于与几个朋友一起构想了“ASCII丝带运动”，这是对当时所有他们发送的消息中的HTML电子邮件的抗议的签名。

（[ASCII](https://en.wikipedia.org/wiki/ASCII)是一个旧的字符编码标准。多年前，它被能够处理非英语字符的UTF-8所取代。）

尽管他失去了将近30年前的记录，但他的一条信息，日期为1998年6月17日，被保存下来，[展示了一个网页](http://arc.pasp.de/)，保留了ASCII Ribbon运动的记忆。显然，签名中有一个实际的ASCII丝带的图画，我将在这里尝试复制：

```
/"\
\ /  CAMPANHA DA FITA ASCII - CONTRA MAIL HTML
 X   ASCII RIBBON CAMPAIGN - AGAINST HTML MAIL
/ \
```

我和Mauricio谈过。如今，他在浏览器中使用Gmail并发送HTML消息。

“那时候的我是另一个人，一个更激进的人，”他告诉我。“并不是我反对纯文本电子邮件的基本思想。我反对。我只是不会再像当初那样做了。”

Mauricio很幸运，在巴西商业互联网到来之前，早早就接触到大学的BBS。他说：“我来自一个只有纯文本的环境。”

在ISP工作时（“我能背诵所有的[email]规则，RFCs”），整天沉浸在Linux终端中，以及阅读纯文本电子邮件，这导致他关注ASCII Ribbon运动。

他最大的问题是一些电子邮件没有附带纯文本副本。可以发送带有纯文本副本的HTML电子邮件，这对像Mauricio在1990年代末那样的情况非常有用，即使用无法渲染HTML的电子邮件客户端的人。“我们因此感到愤怒，”他回忆道。

如今，他指出HTML电子邮件的另一个问题是可访问性较低。HTML布局或复杂的格式可能会使那些有运动或视力障碍的人难以阅读。

该活动并没有引起太大反响，尽管在今天仍然在一些不知名的网站上（以及[Wikipedia的一个条目](https://en.wikipedia.org/wiki/ASCII_ribbon_campaign)中）留有相关记录。据Mauricio称，那时传播被视为另类观点的想法更加困难。

当我问及该活动失败的另一个原因可能是因为它处理的是一个非问题时，他表示同意：“对于大多数人来说，他们创建更漂亮的电子邮件的能力比那些不断抱怨错误的疯狂同志们更重要。”

Mauricio在美国的Red Hat工作，当他的专业需求使他离开办公室时，就用Webmail替代了CLI电子邮件客户端，也就是说，当他需要随时随地访问电子邮件时（这是在现代智能手机出现之前）。他说：“如今，我工作的很多内容都在浏览器中。”“我接受了现在生活在浏览器中的事实。”

`***`

尽管HTML电子邮件绝对主导，纯文本仍在抵抗。一些流行的客户端/应用程序，如Gmail、Apple Mail和Outlook，允许以纯文本编写消息，但在可用HTML版本时不允许阅读。

在Thunderbird中，您可以完全生活在纯文本领域。推荐安装类似[Allow HTML Temp](https://addons.thunderbird.net/pt-BR/thunderbird/addon/allow-html-temp/)这样的扩展，当没有纯文本或者纯文本混乱时启用HTML版本。对于Linux的Evolution也提供这种体验。

罕见的是，仍然有一些仅支持纯文本的电子邮件客户端，例如Claws或The Bat。来自KDE项目的KMail甚至允许您查看HTML格式的消息，但默认强制的是纯文本。（将消息转换为HTML的栏/按钮是一种视觉灾难。）

我已经几年用纯文本写电子邮件了。我很希望也能像这样阅读它们。我可以设置我喜欢的字体，无需分心地阅读。

在现实世界中，HTML促成了糟糕的电子邮件营销，以及无法抵抗文本格式化按钮排列的人们最深层的恶味本能。

如果您想加入ASCII Ribbon团队，[使用纯文本电子邮件](https://useplaintext.email/)网站是一个很好的起点。
