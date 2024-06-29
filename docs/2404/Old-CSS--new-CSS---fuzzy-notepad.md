<!--yml

category: 未分类

日期：2024-05-27 13:12:14

-->

# 旧的CSS，新的CSS / 模糊的笔记本

> 来源：[https://eev.ee/blog/2020/02/01/old-css-new-css/](https://eev.ee/blog/2020/02/01/old-css-new-css/)

我是在90年代末才开始涉足网页设计和开发的，直到我敲下这句话的时候才意识到那是多么久远的事了。

这简直太可怕了。我的意思是，能够制作东西并且让其他人在线上看到确实很酷，但我们的工具并不是很多。

我一直认为*大多数*从事网页相关工作的人都还记得那些日子，或者至少记得随之而来的那十年，但我觉得这种假设可能有点过时。不久前，我看到一条[tweet](https://twitter.com/keinegurke_/status/1162309192855822339)，对我们在没有`border-radius`时所做的事感到惊讶。我仍然记得满怀期待地等待它去掉前缀！

不过，我怀疑我也认识一些只在旧日尝试过网页设计的人，他们或许认为一切都没有改变。

我在这里要告诉**所有人**离开我的草坪。这是我记忆中的CSS和网页设计历史。

* * *

（请记住，这篇文章是记忆和研究的结合体，所以我不能保证任何内容是正确的，*特别是*关于因果关系的部分。你可能想看看[W3C关于CSS历史的页面](https://www.w3.org/Style/CSS20/history.html)，这个页面要短得多，更接近现实，也少得多的带有不雅语言。）

（另外，更多的示意图会让这篇文章大有裨益，但光是*写*就已经花了很长时间。）

起初，并没有CSS。

这真的很糟糕。

我最喜欢的那个时代的遗物是教会我HTML的书籍：O’Reilly的[HTML权威指南](https://isbnsearch.org/isbn/9781565924925)，在90年代中后期出版了几个版本。这本书确实是关于*HTML*的，完全没有提到CSS。我已经找不到它了，也很难在网上找到截图，但这里有一张来自HTML与XHTML：权威指南的页面，似乎是修订版（我会稍后提到XHTML），风格基本相同。这里是199X年的尖端网页设计建议：

“*清楚地用水平线条分割页眉和页脚。*”

不，那不是`border-top`。那是`<hr>`。页面标题几乎肯定是用了`<center>`居中。

页面使用默认的文本颜色、背景和字体。部分原因是这本书是一本逐步介绍概念的指南；部分原因是这本书是黑白印刷的；还有部分原因，我敢肯定，是因为任何上色工作都非常让人头疼。

假设你想让整个网站上所有的`<h1>`都是红色的。你必须这样做：

|  |
| --- |

```
`<H1><FONT COLOR=red>...</FONT></H1>` 
```

|

…*每一次都是这样*。希望你永远不要决定改用蓝色！

哦，每个人都以大写字母编写HTML标签。我不记得为什么我们都认为那是个好主意。也许这是因为文本编辑器中的语法高亮还不太普遍（我的意思是：我12岁时在用记事本），大写标签更容易与正文区分开。

因此，保持你的网站一致性有点儿像噩梦。一个解决方案是简单地不对任何东西进行样式化，很多人都这样做了。在某些方面，这很好，因为浏览器允许你更改这些默认设置，所以你可以按自己的方式阅读网络。

一个聪明的替代方案，我记得在很多Geocities网站上看到过，就是给每个页面都赋予完全不同的视觉风格。反正，对吧？每个新页面都随心所欲地做。

那种趋势很可能是网络设计的顶峰。

天啊，我怀念那些日子。当时没有高墙花园，没有Twitter或Facebook。如果你想对任何人说点什么，你得自己搭建网站。那是*太棒了*。没有人知道自己在做什么；我敢打赌，当时绝大多数网页设计师都是毫无头绪的业余爱好者青少年（比如我），都在互相抄袭其他毫无头绪的业余爱好者青少年。一半的网络是关于《变形金刚》的粉丝门户网站，带有令人费解的闪屏页面，警告说如果你的屏幕分辨率是640×480，他们的网站效果最佳。（显然，任何分辨率不足的12岁孩子应该用零用钱买个新显示器。）所有Cool并且在行的人都使用Internet Explorer 3，这是当时最先进的浏览器，但是一些失败者仍然使用Netscape Navigator，所以你必须在闪屏页面上放置一个“最佳IE”动画GIF。

这也是“网络安全颜色”时代的产物 — 一种包含216种颜色的调色板，其中每个通道都是`00`、`33`、`66`、`99`、`cc`或`ff` — 这种存在是因为一些人仍然使用256色的显示器！现在我们理所当然的事物，比如24位色彩。

事实上，我们现在理所当然的*很多*东西当时仍然是一个奇怪而未被开发的问题空间。你想在网站的每个页面上都有相同的导航？好的，没问题：复制/粘贴到每个页面上。当你更新它时，记得更新每个页面 —— 但最有可能你会忘记一些页面，整个网站就变成了自己的考古挖掘，有越来越腐败的页面层。

使用*框架*要简单得多，意思是浏览器窗口分割成网格，每个部分加载不同的页面……但是如果他们登陆没有框架的独立页面，这时人们会感到困惑，这在从AltaVista等搜索引擎跳转时很常见。（我真不敢相信我在解释框架，但是自2001年以来没有人再使用它们了。你知道iframes吗？“i”代表*内联*，以区别于*常规*框架，后者占据整个视口。）

PHP 当时甚至还没有被称为 PHP，没人听说过它。这种奇怪的 “Perl” 和 “CGI” 真的很奇怪和难以理解，而且在你自己的计算机上无法运行，错误也很难找到和诊断，而且 Geocities 不支持它。如果你真的很幸运和聪明，你的网络主机使用的是 Apache，你可以使用其 “服务器端包含” 语法来做类似于这样的事情：

|

```
 1
 2
 3
 4
 5
 6
 7
 8
 9
10
11
12
13
14
15
16
17
```

|

```
`<BODY>
    <TABLE WIDTH=100% BORDER=0 CELLSPACING=8 CELLPADDING=0>
        <TR>
            <TD COLSPAN=2>
                <!--#include virtual="/header.html" --> 
            </TD>
        </TR>
        <TR>
            <TD WIDTH=20%>
                <!--#include virtual="/navigation.html" --> 
            </TD>
            <TD>
                (actual page content goes here)
            </TD>
        </TR>
    </TABLE>
</BODY>` 
```

|

*Mwah.* 美妙。Apache 会看到这些特殊注释，粘贴引用文件的内容，然后你就可以开始操作了。不过，缺点是当你想要在自己的网站上工作时，所有导航都不见了，因为你在没有 Apache 的普通计算机上进行操作，而你的网络浏览器认为这些只是普通的 HTML 注释。当然，安装 Apache 是不可能的，因为你有一台 *电脑*，而不是 *服务器*。

遗憾的是，这一切都已经消失了 —— 被同质化的时间线所掩埋，那些不是这周制作的东西都成了过时和被遗忘的东西。互联网本应使信息永恒，但实际上，很多信息却是短暂的。我怀念几乎我认识的每个人都有自己的网站的时代。拥有 Twitter 和 Instagram 作为你整个在线存在的替代品实在是差强人意。

…

那么，让我们来看看《Space Jam》的网站。

如果你不了解，《Space Jam》是有史以来最伟大的电影。它记录了兔八哥极其短暂的篮球生涯，与现实中的迈克尔·乔丹一起拯救地球免受外星人侵害，原因无关紧要。它后来推出了一系列非常成功和广受好评的 [RPG 衍生作品](https://www.talesofgames.com/related_game/barkley-shut-up-jam-gaiden/)，描述了《Space Jam》的余波，非常符合正史。

我们是真的幸运，因为《Space Jam》发布 24 年后，它的网站依然 [保持在线](https://www.spacejam.com/1996/)。我们可以在这里探索1996年网页设计的巅峰，就在这里，就在现在。

首先，请注意，这个网站的每一页都是静态页面。不仅如此，它是以 `.htm` 结尾而不是 `.html`，因为在 Windows 95 之前的版本上，人们仍然受制于 8.3 文件名。不确定为什么在 URL 中这很重要，好像你要在 Web 服务器上运行 Windows 3.11 一样，但是事实就是如此。

闪屏页的 CSS 看起来是这样的：

| |
| --- |

```
`<body bgcolor="#000000" background="img/bg_stars.gif" text="#ff0000" link="#ff4c4c" vlink="#ff4c4c" alink="#ff4c4c">` 
```

|

哈哈，开个玩笑！CSS 是什么鬼？《Space Jam》比它早了一个月。（我在页面源代码中看到了一行，但我很确定那是后来添加的，用来为一些法律上义务的政策链接设置样式。）

请注意这些导航链接的极其精确的定位。这项技术是1996年人们完成一切工作的方式：使用表格。

实际上，表格在布局上比 CSS 有一个功能优势，这在那些日子非常重要，不仅因为当时还没有 CSS。你看，你可以使用 Ctrl+单击来选择表格*单元格*，甚至拖动来选择所有单元格，这显示了单元格的排列方式，起到了超级复古布局调试器的功能。这非常棒，因为第一个有意义的网络调试工具，[Firebug](https://en.wikipedia.org/wiki/Firebug_%28software%29)，直到 2006 年才发布——整整十年之后！

这个表格的标记溢出了许多令人费解的空行，但如果去掉这些空行，它看起来就像这样：

|

```
 1
 2
 3
 4
 5
 6
 7
 8
 9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
37
38
```

|

```
`<table width=500 border=0>
<TR>
<TD colspan=5 align=right valign=top>
</td></tr>
<tr>
<td colspan=2 align=right valign=middle>
<br>
<br>
<br>
<a href="cmp/pressbox/pressboxframes.html"><img src="img/p-pressbox.gif" height=56 width=131 alt="Press Box Shuttle" border=0></a>
</td>
<td align=center valign=middle>
<a href="cmp/jamcentral/jamcentralframes.html"><img src="img/p-jamcentral.gif" height=67 width=55 alt="Jam Central" border=0></a>
</td>
<td align=center valign=top>
<a href="cmp/bball/bballframes.html"><img src="img/p-bball.gif" height=62 width=62 alt="Planet B-Ball" border=0></a>
</td>
<td align=center valign=bottom>
<br>
<br>
<a href="cmp/tunes/tunesframes.html"><img src="img/p-lunartunes.gif" height=77 width=95 alt="Lunar Tunes" border=0></a>
</td>
</tr>
<tr>
<td align=middle valign=top>
<br>
<br>
<a href="cmp/lineup/lineupframes.html"><img src="img/p-lineup.gif" height=52 width=63 alt="The Lineup" border=0></a>
</td>
<td colspan=3 rowspan=2 align=right valign=middle>
<img src="img/p-jamlogo.gif" height=165 width=272 alt="Space Jam" border=0>
</td>
<td align=right valign=bottom>
<a href="cmp/jump/jumpframes.html"><img src="img/p-jump.gif" height=52 width=58 alt="Jump Station" border=0></a>
</td>
</tr>
...
</table>` 
```

|

这是前两行，包括徽标。你明白了吧。所有内容都用表格单元格上的`align`和`valign`布局；`rowspan`和`colspan`经常使用；还有一些 `<br>` 用来微调每次一个行高的垂直定位。

这个页面上还有其他奇妙的遗物，包括这个头部，其中包含 Apache SSI 语法！这肯定在这些年移动站点时悄悄断掉了；它目前托管在 Amazon S3 上。你知道，亚马逊？那个书店？

|  |
| --- |

```
`<table border=0 cellpadding=0 cellspacing=0 width=488 height=60>
<tr>
<td align="center"><!--#include virtual="html.ng/site=spacejam&type=movie&home=no&size=234&page.allowcompete=no"--></td>
<td align="center" width="20"></td>
<td align="center"><!--#include virtual="html.ng/site=spacejam&type=movie&home=no&size=234"--></td>
</tr>
</table>` 
```

|

好吧，让我们来看看[jam central](https://www.spacejam.com/1996/cmp/jamcentral/jamcentralframes.html)。我已经使用浏览器开发工具将视口缩小到640×480，以获得真实体验（虽然我也会失去标题栏、任务栏和五六个 IE 工具栏的一些垂直空间）。

注意框架：左上角的徽标导航回着陆页，巧妙地节省了重复所有导航的屏幕空间，右上角则是一个该死的广告横幅，已经被七种不同方式拦截了。这三部分都是单独的页面。

还要注意完全无法阅读的红色文本放在纹理背景上，这是90年代网页设计的真正特征之一。“为什么不将那段文字放在易于阅读的背景上？”你可能会问。你这个蠢货。我怎么*可能*做到？只有`<body>`有`background`属性！我可以用表格，但表格只支持纯色背景，那看起来会太无聊了！

但等等，这是什么新的导航小部件？为什么链接都这么对不齐？这又是一个表格吗？不是，尽管用切割图像块填充表格并不罕见。但这是一个*图像映射*，一个被遗忘的 HTML 功能。我来展示一下源代码：

|  |
| --- |

```
`<img src="img/m-central.jpg" height=301 width=438 border=0 alt="navigation map" usemap="#map"><br>

<map name="map">
<area shape="rect" coords="33,92,178,136" href="prodnotesframes.html" target="_top">
<area shape="rect" coords="244,111,416,152" href="photosframes.html" target="_top">
<area shape="rect" coords="104,138,229,181" href="filmmakersframes.html" target="_top">
<area shape="rect" coords="230,155,334,197" href="trailerframes.html" target="_top">
</map>` 
```

|

我认为这基本上是不言自明的。`usemap` 属性附加了一个图像映射，其定义为一堆可点击的区域，美丽地编码为晦涩的坐标列表或其他什么东西。

这些东西仍然有效！这是在 HTML 中！你现在可以使用它！虽然你可能不应该这样做！

让我们再来看看这里的另一页随机页面。我很想看看电影中的一些照片。（等等，*照片*？难道我们还不知道什么是“截图”吗？）

另一个框架集，但这次排列方式不同。

|  |
| --- |

```
`<body bgcolor="#7714bf" background="img/bg-jamcentral.gif" text="#ffffff" link="#edb2fc" vlink="#edb2fc" alink="#edb2fc">` 
```

|

他们在这里做了一件重要的事情：因为他们指定了一个背景图像（是不透明的），他们*还*指定了一个背景颜色。如果没有这个颜色，如果背景图像加载失败，页面将是白色文本在默认的白色背景上，这将是*难以阅读*。

（这仍然是需要牢记的重要事情。我觉得现代网页开发倾向于假设所有内容都会加载，或者将加载视为某种需要解决的不便，但并非每个人都在距离主干网二十英尺的旧金山办公室内使用有线连接。）

但关于页面本身。缩略图网格是网页设计的经典问题，可以追溯到……嗯……至少可以追溯到《Space Jam》时代。主要问题在于你希望*把东西放在一起*，而HTML默认将所有内容堆叠在一个大列中。你可以将所有缩略图放在同一行（自动换行）的文本中，但那不会形成一个真正的网格——而且你通常希望每个缩略图都有某种形式的标题。

《Space Jam》的方法是使用当时所有人都有的唯一真正工具：表格。它的结构是这样的：

|  |
| --- |

```
`<table cellpadding=10>
<tr><td align=center><a href="..."><img src="..."></a></td>...</tr>
<tr>...</tr>
<tr>...</tr>
<table>` 
```

|

一个3×3的缩略图网格，由浏览器来排列。（最后一张图片单独一行，实际上不是表格的一部分。）这种方式无法按比例缩放到你的屏幕大小，但那时每个人的屏幕都非常小，所以*略*不是太大的问题。他们在这里没有添加标题，但由于每个缩略图都包裹在一个表格单元中，他们很容易可以这样做。

这就是1996年缩略图网格的技术水平。我们将会多次回顾这个小UI难题；你可以在[另一页](https://eev.ee/media/2020-02-css/thumbnail-grids.html#tables)上查看实时示例（并查看样本标记的源代码）。

但是让我们花点时间来欣赏一下“全尺寸、全彩色、互联网质量”的电影截图在我当前的显示器上的大小。

嘿，它们不到16 KB！只需九秒就可以下载。

（我想起了嵌入式*视频*的问题，直到几年后HTML5的`<video>`标签才解决了这个问题。在那之前，你必须使用二进制插件，而且所有插件都很糟糕。）

（哦，顺便说一句：默认情况下，链接内的图像周围有一个与链接颜色相同的边框。图像链接通常是不言自明的，因此这通常很烦人，在使用`<img border=0>`之前你必须为每个图像单独禁用它们。）

所以这就是我们开始的地方，而且很糟糕。如果你想要在超过少数页面上保持*任何*一致性，你的选择非常有限，基本上只能大量复制和粘贴。《Space Jam》网站基本上选择了不去烦恼——许多其他网站也是如此。

然后CSS出现了，简直是*个奇迹*。所有内联重复消失了。你想要所有顶级标题都是特定颜色？没问题：

|  |  |
| --- | --- |

Bam！完成了。无论您的文档中有多少个`<h1>`，每一个都会变成令人眼花缭乱的红色，而您无需再费心思。更棒的是，您可以将这段代码放入自己的文件中，并且几乎不费力气地将这种颇具争议的美学选择应用于*整个网站的每一页*！同样适用于您华丽的平铺背景图像，链接的颜色以及表格中字体的大小。

（只需记住在HTML注释中包裹您`<style>`标签的内容，否则不支持CSS的旧浏览器将把它们显示为文本。）

您不仅限于大规模地设置标签的样式，CSS还引入了“类”和“ID”来专门目标标记的元素。像`P.important`这样的*选择器*只会影响`<P CLASS="important">`，而`#header`则只会影响`<H1 ID="header">`。（区别在于ID在文档中是唯一的，而类可以任意使用。）借助这些工具，您可以有效地发明自己的标签，为您的网站提供一个定制版本的HTML！

这是一个巨大的进步，但当时没有人（大概？）在考虑使用CSS来实际*排列*页面。当[CSS 1](https://www.w3.org/TR/2008/REC-CSS1-20080411/)在1996年12月成为推荐标准时，它几乎没有涉及布局。它所做的只是从HTML的*现有*能力中分离出它们所附属的标签。我们有字体颜色和背景色*因为*存在`<FONT COLOR>`和`<BODY BACKGROUND>`。唯一远程影响事物位置的特性是`float`属性，相当于`<IMG ALIGN>`，它将图像拉到侧边，并让文本围绕其周围流动，就像杂志文章一样。远非令人满意。

这并不太令人惊讶。除了表格之外，HTML对于布局并没有真正的答案，而表格属性过于复杂，无法在CSS中进行概括，并且与标签结构纠缠不清，因此CSS 1没有任何继承的内容。它只是简化了我们在`<FONT>`标签等上已经在做的事情 —— 让Web设计更少单调，更少出错，更少噪音，并且更易于维护。这是一个相当不错的进步，每个人都欣然接受它，但表格仍然是排列页面的王者。

尽管如此，你的博客实际上只需要一个页眉和一个侧边栏，而表格完全可以胜任，而且你也不会经常彻底改变这个基本结构。复制/粘贴几行`<TABLE BORDER=0>`和`<TD WIDTH=20%>`也不是什么大不了的事情。

在某个时间段内 —— 我想说是几年，但当你还是个孩子时，时间过得更慢 —— 这就是Web的状态。表格用于布局，CSS用于… 好吧，*样式*。颜色，大小，粗体，下划线。甚至还有这种酷炫的技巧，你可以让链接只在鼠标*指向*它们时才有下划线。*酷帅！*

（有趣的事实：HTML *电子邮件*仍然基本困在这个*时代*。）

（大约在这个时候，我11岁，完全不知道自己在做什么，主要是从其他11岁的孩子那里学习。但那没关系；网络的一大部分是11岁的孩子在制作他们自己的网站，这是美好的。你为什么要去*商业*网站，当你可以窥探到地球另一边某人非常具体的爱好呢？）

一年半后的98年中，我们迎来了[CSS 2](https://www.w3.org/TR/2008/REC-CSS2-20080411/)的礼物。（顺便说一句，我*喜欢*这个页面的背景。）这是一个小小的升级，解决了各个领域的一些缺陷，但最有趣的是增加了几个定位原语：`position`属性，允许您在精确的坐标位置放置元素，以及`inline-block`显示模式，允许您像处理图像一样将元素放在文本行中。

如此诱人的果实，却又触手可及！使用`position`似乎很好，但像素完美的定位与HTML的流动设计严重不符，很难做到在其他屏幕尺寸上不崩溃或有其他严重缺陷。这个谦逊的`inline-block`似乎*足够*有趣；毕竟，它解决了HTML布局的核心问题，即*将东西放在一起*。但至少目前没有浏览器实现它，它基本被忽视了。

我不能确定是引入了定位还是其他因素，但*某些事物*在这个时候启发了人们尝试在CSS中进行布局。理想情况下，你会*完全*将页面的结构与其外观分离。甚至有一个网站将这一原则推向了极端 — [CSS Zen Garden](http://www.csszengarden.com/)至今仍在，展示了通过应用不同样式表将*相同的HTML*彻底转变为完全不同设计的能力。

麻烦的是，早期的CSS支持非常多BUG。回想起来，我怀疑浏览器供应商只是从HTML标签中挑出行为并称之为一天。我很高兴地说，RichInStyle仍然有一个[早期浏览器CSS错误的广泛列表](http://www.richinstyle.com/bugs/)；这里是一些我*最喜欢的*：

+   IE 3会忽略文档中除了最后一个`<style>`标签之外的所有`<style>`标签。

+   IE 3忽略了伪类，所以`a:hover`会被视为`a`。

+   IE 3和IE 4将`auto`边距视为零。实际上，我认为这一点可能一直持续到IE 6。但这没关系，因为IE 6还错误地将`text-align: center`应用于块元素。

+   如果你给一个绝对URL设置背景图像，IE 3会尝试在本地程序中打开图像，就像你已经下载了它一样。

+   Netscape 4理解像`#id`这样的ID选择器，但将`h1#id`视为*无效*。

+   Netscape 4没有继承属性 — 包括字体和文本颜色！ — 到表格单元格中。

+   Netscape 4在`<li>`标签上应用了样式到列表*标记*，而不是内容。

+   如果同一个元素同时设置了`float`和`clear`（这并不奇怪），Netscape 4 for Mac就会崩溃。

这就是我们所面对的情况。而人们想要使用CSS来*布局*整个页面？哈哈。

然而这个想法越来越受欢迎。甚至成为一种精英主义的呐喊口号，一种用来抨击其他人的最佳实践。表格布局只是纯粹糟糕，你会听到这样的说法！它们会让屏幕阅读器混乱，它们在语义上不正确，与CSS定位的交互性差！所有这些说法都是正确的，但当替代方案是——这就是一个更难接受的现实。

嗯，我们马上就会谈到。首先，让我们了解一下2000年左右的网络背景。

简而言之，这家公司Netscape一直在销售其Navigator浏览器（面向企业使用，个人使用免费），然后微软推出了完全免费的Internet Explorer浏览器，*然后*微软竟然胆敢将IE捆绑在Windows系统中。你能想象吗？一个操作系统*带有*一个浏览器？这件事情闹得沸沸扬扬，[微软因此而被起诉](https://en.wikipedia.org/wiki/United_States_v._Microsoft_Corp.)，他们输了，但后果基本上是没有什么变化。

但无论如何，这都不会有任何影响，因为他们已经*做到了*，并且成功了。IE几乎完全击败了Netscape的市场份额。两款浏览器都非常地有bug，而且这些bug还*各有不同*，所以一个网站如果只支持其中一款浏览器，当在另一款浏览器上浏览时很可能会出现大问题——这意味着当Netscape的市场份额下降时，网页设计师对它的关注越来越少，越来越少的网页在它上面运行，从而导致它的市场份额进一步下降。

如果你不使用Windows系统，那你就倒霉了，我猜。有趣的是，Mac版的IE 5.5其实比IE 6要*少*出错。（顺便说一句，比尔·盖茨不是一个天才程序员，而是一个积极而无情的商人，通过有意消灭所有竞争对手来赚取自己的财富，从而导致整体计算机技术恶化。）

到了2001年中期Windows XP发布时，内置了Internet Explorer 6，Netscape已经从一个巨头变成了一个小众玩家。

然后，在完全完全主导之后，微软停了下来。自从成立以来，Internet Explorer每年发布一次左右，但IE 6成为了五年多来的最后一次发布。它仍然存在缺陷，但当没有竞争时这些缺陷就不那么明显了，而且*足够好*。同样，Windows XP也足够好以主导桌面，接下来几年也不会再有新的Windows系统。

W3C，编写标准的团体（不要与 W3Schools 混淆，后者是不正当的 SEO 寄生虫），也停滞了。HTML 在 90 年代中期经历了几次修订，然后在 HTML 4 版本上冻结了。CSS 在短短一年半的时间里得到了更新，然后再也没有了；小更新 [CSS 2.1](https://www.w3.org/TR/CSS21/) 直到 2004 年初才达到候选推荐阶段，并花了另外七年才最终定稿。

随着 IE 6 的主导地位，整个 Web 就像是时间被冻结了一样。标准不再重要，因为实际上只有一个浏览器，无论它做什么都成为了事实上的标准。随着 Web 的普及，IE 的控制也使得除了 Windows 之外的任何平台都难以使用，因为 IE 只能在 Windows 上运行，而实际上一个网站是否能在其他浏览器上工作还是个未知数。

（人们开始怀疑垄断是不好的。应该有一条法律！）

与此同时，Netscape 通过决定对其浏览器引擎进行大规模重写，最终推出了更加符合标准的 Netscape 6，却付出了数年的市场缺席时间，而 IE 则一直占据市场份额高达 96% 的垄断地位。另一方面，新引擎作为 Mozilla 应用套件开源，这在未来几年变得非常重要。

在我们讨论那个问题之前，还有一些其他事情正在发生。

所有早期的 CSS 实现都充斥着 bug，但有一个 bug 特别臭名昭著：*框模型 bug*。

你看，一个框（元素占用的矩形空间）有几个尺寸：它自己的宽度和高度，然后周围的空白称为填充，然后是可选的边框，然后是将其与相邻框分隔开的外边距。CSS 规定这些属性都是累加的。一个具有以下样式的框：

|  |
| --- |

```
 `width:  100px; padding:  10px; border:  2px  solid  black;` 
```

|

…因此从边框到边框将会是 124 像素宽。

而 IE 4 和 Netscape 4 则采取了不同的方法：它们将 `width` 和 `height` 视为从边框到边框的测量，然后*减去*边框和填充以获取元素本身的宽度。在这些浏览器中，同样的框从边框到边框将是 100 像素宽，其中 76 像素留给了内容部分。

这种与规范冲突的情况并不理想，而 IE 6 则试图修复它。不幸的是，简单地进行改变将意味着完全破坏了许多之前在*两个* IE 和 Netscape 中都能正常工作的网站的设计。

所以 IE 团队提出了一个非常奇怪的妥协方案：他们将旧的行为（以及其他几个主要的 bug）声明为“怪异模式”，并将其设为*默认*。新的“严格模式”或“标准模式”必须通过在文档的 `<html>` 标签之前放置“doctype”来选择*进入*。看起来会是这样的：

|  |
| --- |

```
`<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01//EN" "http://www.w3.org/TR/html4/strict.dtd">` 
```

|

多年来，每个HTML文档的顶部都必须粘贴这个该死的一行混乱代码。（HTML5稍后将其简化为`<!DOCTYPE html>`。）回过头来看，这实际上是一种奇怪的选择以实现正确的CSS行为；文档类型自HTML规范的早期就已经存在，那时候它是一个[RFC](https://tools.ietf.org/html/rfc1866)。我猜想的想法是，由于实际上*没有人*费心包含一个，这是一种便捷的方式允许选择，而无需要求专有扩展仅仅是为了避免最初就是错误的行为。对IE团队来说真是太好了！

有趣的是，怪异模式在所有浏览器中依然存在，并且仍然是默认的状态，二十年后！随着时间的推移，具体的怪癖变化了，特别是Chrome和Firefox即使在怪异模式下也不使用IE盒模型，但仍然有[相当多其他的模拟错误](https://developer.mozilla.org/en-US/docs/Mozilla/Mozilla_quirks_mode_behavior)。

现代浏览器还有“准标准”模式，仅模拟一个怪癖，也许是第二个最臭名昭著的：如果一个表格单元格仅包含单个图像，则基线下的空间将被移除。在正常的CSS规则下，图像位于一行（其他情况下为空的）文本中，这需要一些保留在下面用于降部（如y字母的尾巴）的空间。早期的浏览器没有正确处理这个问题，大约在2000年左右的一些本来应该是严格模式的网站却依赖于它——例如，通过将大图切割并排列在表格单元格中，期望它们紧密显示在一起——因此有了中间模式来继续支持它们的使用。

但是回到过去：尽管这显然是标准的一种胜利（因此可以实现互操作性），但也带来了一个新问题。由于IE 6占主导地位，而文档类型（doctypes）是可选的，几乎没有令人信服的理由去费心使用严格模式。其他浏览器最终*模拟*了它，并且非标准行为成为了事实上的标准。关心这类事情的网页设计师（我们有很多）高呼要启用严格模式，因为这是确保与其他浏览器兼容性的绝对最低步骤。

与此同时，W3C对HTML失去了兴趣，转而支持开发XHTML，这是一种尝试用XML语法而非SGML重新设计HTML。

（你问什么是SGML？我也不知道。没有人知道。那是HTML构建的语法，这是唯一让人们听说过它的原因。）

值得肯定的是，当时确实有一些很好的理由这样做。HTML通常是手工编写的（现在依然如此），而任何手工编写的东西都可能偶尔出现错误。浏览器并没有习惯直接拒绝有错误的HTML，所以它们采用了各种错误校正技术 —— 而且和其他一切一样，不同的浏览器对错误的处理方式也不同。在IE 6中，略有格式不规范的HTML可能看起来工作正常（所谓的“工作正常”是指“做了你希望的事情”），但在其他任何浏览器中可能就会变成一团糟。

W3C的解决方案是XML，因为在21世纪初他们几乎对所有问题的解决方案都是XML。如果你不知道，XML在错误处理方面采用了更为明确和激进的方法 —— 如果你的文档包含解析错误，*整个文档*都是无效的。这意味着如果你依赖XHTML并在某处打了一个错字，**什么都不会**呈现。只有一个*错误*。

这很糟糕。表面上听起来还可以，但是请考虑一下：通常使用*库*动态组装通用XML，将文档视为一个你操作的树，然后在完成时将其转换为文本。这在XML作为数据序列化的常见用途中非常棒，其中你的数据已经是树形结构，XML结构的大部分是简单且重复的，并且很容易隐藏在*函数*中。

HTML不是这样的。HTML文档几乎没有可靠的重复结构；即使是这篇博文，主要由`<p>`标签构成，也包含了意外的`<em>`标签和偶尔出现在段落之间的`<h2>`标签。这在树形结构中表达起来不太有趣。这是一件大事，因为同一时期服务器端渲染也变得流行起来，生成的HTML仍然被视为一个带有*模板*的文本流。

如果HTML只是作为完整的静态文档写入，那么XHTML本来可能行得通 —— 你写一个文档，在浏览器中查看，知道它工作正常，没有问题。但是动态生成它，并冒着*特殊边界情况*可能导致整个站点被一个晦涩难懂的浏览器错误所替换的风险？那*太糟糕*了。

那当时我们刚开始听说这个新潮的Unicode事物时，确实没有什么帮助。有时候要弄清楚怎么让它工作还不总是很清楚，而且一个糟糕的UTF-8序列足以让整个XML文档被视为*格式错误*！

因此，经过一些尝试之后，XHTML基本上被遗忘了。它的遗产以两种方式延续着：

+   它让我们都停止使用大写标签名！再见`<BODY>`，你好`<body>`。XML是区分大小写的，所有XHTML标签都定义为小写，因此大写标签根本无法工作。（有趣的事实：直到今天，JavaScript API仍然以大写形式报告HTML标签名。）语法高亮的普及可能也与此有关；我们不再像1997年那样全都在使用记事本了。

+   一些人*仍然*认为自闭标签是必需的。你知道吗，HTML有两种类型的标签：像`<p>...</p>`这样的容器和像`<br>`这样的标记。由于`<br>`不可能包含任何内容，所以没有`</br>`这样的东西。作为一个通用语法，XML没有这种区别；每个标签*必须*关闭，但作为一种快捷方式，你可以写`<br/>`来表示`<br></br>`。

    XHTML已经死了很多年了，但由于某种原因，我仍然看到人们在常规HTML文档中写`<br/>`。在XML之外，这个斜杠什么都不做；HTML5已经为兼容性原因定义了它，但它被默默忽略了。它甚至是积极有害的，因为它可能会让你误以为`<script/>`是一个空的`<script>`标签 —— 但在HTML中，它绝对*不是*！

我确实怀念XHTML的一件事情。你可以将其与XSLT，即XML模板化元语言，结合使用，实现浏览器内模板化（即将页面特定内容插入整体站点布局），而无需脚本。这是*唯一*可能的方式，当它起作用时，它非常酷，但当它不起作用时，缺点就太严重了。另外，XSLT完全*难以理解*。

回到CSS！

你是一名有抱负的网页设计师。出于某种原因，你想尝试使用这个CSS东西来布局整个页面，尽管它*明显*只是用于颜色和其他东西。你会怎么*做*？

如我之前提到的，你的核心问题在于*将事物放在一起*。把事物*放在*另一起并不是问题 —— 这是HTML的正常行为。之所以人们都用表格，是因为你可以把东西随便扔到表格单元中，让它们并排、列出来。

嗯，表格似乎已经过时了。CSS 2添加了一些元素显示模式，这些模式对应于表格的各个部分，但要使用它们，你必须像真正的表格一样三级嵌套：表格本身，然后是行，然后是单元格。这似乎并不是一个巨大的进步，而且无论如何，IE在遥远的*未来*之前都不会支持它们。

有那个`position`东西，但它似乎更多时候让事物*重叠*。嗯。

那还*剩下*什么？

只有一个工具，确实如此：`float`。

我说过`float`是为杂志式的“拉出”图像而设计的，这是真的，但CSS将其定义得相当通用。从*原理*上讲，它可以应用于任何元素。如果你想要一个侧边栏，你可以告诉它向左浮动，宽度占页面的20%，你会得到类似于这样的效果：

|  |
| --- |

```
`+---------+
&#124; sidebar &#124; Hello, and welcome to my website!
&#124;         &#124;
+---------+` 
```

|

唉！浮动还有一个次要行为，那就是文本会围绕它环绕。如果你的页面文本比侧边栏长，它会围绕*在*侧边栏下方，并且这种错觉会破灭。但是嘿，没问题。CSS指定浮动不会互相环绕，所以你所需要做的就是也浮动主体！

|  |
| --- |

```
`+---------+  +-----------------------------------+ &#124; sidebar &#124; &#124; Hello, and welcome to my website! &#124;
&#124;         &#124; &#124;                                   &#124;
+---------+ &#124; Here's a longer paragraph to show &#124;
 &#124; that my galaxy brain CSS float    &#124;
 &#124; nonsense prevents text wrap. &#124;
  +-----------------------------------+` 
```

|

这种方法虽然有效，但其限制比表格的限制明显得多。例如，如果添加页脚，它会尝试适应正文文本的 *右侧* — 记住，所有这些都是“拉”浮动，所以在浏览器看来，“光标”仍然在顶部。现在你需要使用`clear`，将一个元素移到所有浮动元素下方以修复这个问题。如果你将侧边栏设置为20%宽，正文设置为80%宽，那么它们之间的任何边距将增加到100%，使页面比视口更宽，因此现在你会看到一个难看的水平滚动条，所以你还需要进行一些奇怪的数学运算来修复它。如果任一部分有边框或背景，那么它们高度不同就会显得有点显眼，所以你必须进行一些 *真正* 怪异的操作来修复 *那个*。而更加用心的作者注意到，屏幕阅读器会先读完整个侧边栏，然后才会读到正文文本，对盲人访客来说这是一种非常不礼貌的做法，所以他们想出了更 *复杂* 的设置，以在HTML中实现三列布局，其中中间列首先出现。

结果是一个看起来漂亮且工作良好且正确缩放的设计，但支持的是一个奇怪混乱的CSS。你 *编写* 的东西实际上并不对应你 *想要* 的 — 这些是你设计的主要部分，而不是一次性的引用！很难理解布局相关的CSS与屏幕上显示的内容之间的关系，在变好之前，这种困惑只会变得更加严重。

有了新的工具，我们可以改进缩略图网格。原始的基于表格的布局，即使你不关心标签语义，也令人难以忍受。现在我们可以做得 *更好*！

|  |
| --- |

```
`<ul class="thumbnail-grid">
    <li><img src="..."><br>caption</li>
    <li><img src="..."><br>caption</li>
    <li><img src="..."><br>caption</li>
    ...
</ul>` 
```

|

这就是CSS的梦想：你的HTML以一种合理的形式包含页面数据，而CSS描述它的实际 *外观*。

不幸的是，只有`float`作为我们唯一的工具，结果有些粗糙。这个 [新版本](https://eev.ee/media/2020-02-css/thumbnail-grids.html#floats) 对各种屏幕尺寸的适应性更好，但需要一些技巧：单元格必须是固定高度，居中整个网格相当复杂，并且在更宽的元素下网格效果完全崩溃。现在变得明显的是，我们想要的更像是一个具有灵活列数的表格。这只是在 *伪装* 而已。

你还需要这种奇怪的“clearfix”东西，这是在这个时代变得臭名昭著的咒语。记住，浮动不会移动“光标” — 这是我使用的一个虚假概念，但足够接近。这意味着这个 `<ul>`，里面 *只有* 浮动元素，根本没有高度。它的结束点与开始点完全相同，所有浮动的缩略图都会溢出到它的下面。更糟糕的是，因为任何后续元素都没有任何浮动的 *兄弟元素*，它们会完全忽略缩略图，并从空的“网格”下方正常渲染 — 产生一个重叠的 *混乱*！

解决方案是在列表的*末尾*添加一个占用空间但具有 CSS `clear: both` 的虚拟元素 — 将它推到所有浮动元素下面。这有效地将 `<ul>` 的底部推到所有单独缩略图的下方，使其紧密地围绕着它们。

后来的浏览器支持 `::before` 和 `::after` “生成内容” 伪元素，这使我们完全可以避免使用虚拟元素。从 00 年代中期开始的样式表通常都会被这样的东西所填充：

|  |
| --- |

```
`.thumbnail-grid::after  { content:  ''; display:  block; clear:  both; }` 
```

|

不过，它比*表格*好多了。

作为 JavaScript 世界的一个快速插曲，新潮的 `position` 属性确实赋予了我们动态处理布局的能力。我强烈反对这种异端邪说，主要是因为没有人真正做对过，但对于某些人来说，玩具还是挺有用的。

于是开始了 “动态 HTML” 的时代 — 即由 JavaScript 影响的 HTML，一个现在已经完全不流行的术语，因为我们甚至不能再做一个静态的博客而不使用 JavaScript 了。在早期，这种做法要无害得多，青少年们在鼠标光标后面放置了闪光的星星或者小模拟时钟，实时刻度。

这些东西最流行的来源之一是 [Dynamic Drive](http://www.dynamicdrive.com/)，这个神奇的网站依然存在，并且可能有许多从早期到现在都没有更新的玩具。

但如果你不喜欢挖掘，这里有个例子：每年（今年除外，我忘了，哎呀），我喜欢在我的博客上加入彩带和其他无聊的东西来庆祝我的生日。我非常懒，所以开始这个传统时，我使用了 [我从某处找到的这个脚本](http://www.schillmania.com/projects/snowstormv12_20041121a/script/snowstorm.js)，最初是为了雪花而设计的。它通过在页面上放置一堆图像，给它们 `position: absolute`，然后精确地修改它们的坐标，来工作。

与我几年前从头写的[版本](https://c.eev.ee/PARTYMODE/)相比，这个版本只有一点点 JS 来设置图像，然后让浏览器用 CSS 来动画显示它们。功能稍微少一些，但让浏览器完成所有工作，甚至可能使用硬件加速。我们已经走了这么远。

黑暗时代不会永远持续。多种因素的结合使我们朝着光明前进。

其中最重要的之一是 [Firefox](https://www.mozilla.org/en-US/firefox/) — 或者，如果你很酷，最初是 Phoenix，然后是 Firebird — 它于 04 年 11 月发布了 1.0 版，并继续大幅度击败 IE。重新编写的 Netscape 6 浏览器内核，也就是 Mozilla Suite 的核心，被提取为独立的浏览器。它快速、简单，符合更多标准，但完全没有这些都无关紧要。

不，Firefox 确实因为有 *选项卡* 才站稳了脚跟。IE 6 没有选项卡；如果你想打开第二个网页，你就得打开另一个窗口。真是糟糕透了。Firefox 简直是个奇迹。

当然，Firefox 不是第一个支持选项卡的浏览器；完整的Mozilla 套件浏览器也有它们，而鲜为人知（但不畏强权！）的Opera 也有很长时间了。但Firefox因为各种原因才取得了成功，其中至少之一是它没有像Opera 那样在顶部有一个巨大的广告栏。

当然，设计师们确实推动了Firefox 的标准理由；只是这个角度主要吸引了其他设计师，而不是他们的父母。其中最受欢迎和令人惊叹的演示之一是[Acid2 测试](https://en.wikipedia.org/wiki/Acid2)，旨在测试当时的各种现代Web标准功能。它在正确渲染时能够显示一个可爱的笑脸，而在IE 6 中却是一个[可怕的噩梦地狱场面](https://en.wikipedia.org/wiki/File:Ieacid2.png)。早期的Firefox 并不完美，但肯定比较接近，你可以*看到*它在进步，直到Firefox 3 的发布时完全通过了测试。

Firefox 还有一个更快的JavaScript引擎，甚至在JIT技术普及之前就已经非常快。比如，据我回忆，IE 6 通过遍历整个文档来实现 `getElementById`，尽管ID是唯一的。看看一些[旧的jQuery发布公告](https://blog.jquery.com/2011/01/31/jquery-15-released/)吧；它们通常有一些性能图表，而其他所有的东西绝对都*超越*了IE 6 到 8。

哦，还有一件事，IE 6 还是一个巨大的安全漏洞，特别是它对任意二进制组件的本地支持，只需在复杂对话框上点击“是”，就可以完全无限制地访问您的系统。这可能并不有助于它的声誉。

无论如何，除了IE以外的其他浏览器开始占据严肃市场份额，即使是最固执的设计师也不能再只针对IE 6，然后就可以一帆风顺了。现在有一个*理由*来使用严格模式，关心兼容性和标准——而Firefox正努力更好地遵循这些标准，而IE 6则依然停滞不前。

（我认为这一效果为OS X 开辟了一些市场，并且为iPhone 的存在创造了条件。我不是在开玩笑！想想看；如果iPhone浏览器因为大家还在瞄准IE 6 而什么都无法运行，它基本上就是一个更昂贵的Palm。记住，最初苹果甚至不想要原生应用程序；它押注于Web。）

（顺便说一句，Safari 在2003年1月发布，基于KDE Konqueror 浏览器中使用的KHTML引擎的一个分支。我想当时我在使用KDE，所以这对我来说非常激动人心，但其他人真的不太关心OS X及其2%的市场份额。）

另一个重要因素出现在2004年愚人节，谷歌宣布推出Gmail。哈哈！一个有趣的笑话。不糟糕的Web邮件？这真是个好笑的事情，谷歌。

哦。哦，靠。哦他们不是开玩笑的。*这到底是怎么运作的*

答案，正如现在每个网页开发者都知道的那样，是 XMLHttpRequest —— 这个名字是因为没有人曾经用它来请求 XML。显然，它是由 Microsoft 为 Exchange 发明的，早期由 Mozilla 克隆，但我只是从 [Wikipedia](https://en.wikipedia.org/wiki/XMLHttpRequest) 上读到这些，你也可以自己看看。

重要的是，它让你能够从 JavaScript 发起 HTTP 请求。现在，你可以仅仅更新页面的*一部分*，用新数据完全在后台进行，而不需要重新加载。*没人*在此之前听说过这种东西，因此当谷歌基于它推出了一个完整的电子邮件客户端时，简直像是魔法一般。

可以说整个事情都是一个错误，导致了一个地狱般的未来，在这个未来中，静态页面仅仅为了没有任何理由使用 XHR 后台加载了三段文字，但这是一篇[不同的文章](https://eev.ee/blog/2016/03/06/maybe-we-could-tone-down-the-javascript/)。

2006 年 8 月，类似的情况出现了，[jQuery](https://jquery.com/) 的发布就是一个类似的奇迹。它不仅掩盖了IE的“JScript”API与其他所有人采用的标准方法之间的差异（其他库之前也做过），而且使得一次性处理*整组*元素变得非常容易，而历史上这一直是一个巨大的痛苦。现在，你可以相当容易地从 JavaScript 中随处应用 CSS！这其实是个坏主意！但是所有事情都那么糟糕，我们还是这么做了！

等等，我听到你在呼喊。这些事情都与 JavaScript 有关！这不是一篇关于 CSS 的文章吗？

你完全正确！我提到 JavaScript 的崛起，因为我认为它直接导致了 CSS 的现代状态，这要归功于一个重要的因素的增加：

Firefox 向我们展示了，我们可以拥有真正能够*改善*的浏览器 —— Acid2 的每一个新改进都是令人兴奋的。Gmail 向我们展示了，Web 不仅仅能够展示前端的纯文本和雪花。

人们开始渴望变得*时髦*起来。

问题是，浏览器并没有真正变得更好。在某些方面，Firefox 更快，它更加贴近 CSS 规范，但它并没有从根本上做任何浏览器本不应该能做到的事情。只有*工具*有所改进，而这主要影响了 JavaScript。CSS 是一种静态语言，所以你不能写一个库来使它变得更好。用 JavaScript 生成 CSS 是可能的，但这绝对是个坏主意。

另一个问题是，CSS 2 只能很好地处理矩形样式。这在90年代很好，当时每个操作系统都有矩形包含更多矩形的美学。但现在我们已经到了 Windows XP 和 OS X 的时代，所有东西都是光滑和光亮的，用弯曲的塑料制成。在你的*文件浏览器*中拥有圆角和整齐阴影的图案，但在 Web 上却没有这样的东西，这有点尴尬。

于是，一个新的黑暗时代开始了。

设计师们想要的很多东西，CSS 简直不能提供。

+   圆角是一个大问题。方形角落已经过时了，现在每个人都想要圆角按钮，因为它们是未来。 （原生按钮也不再流行，出于某种原因。）然而，CSS 没有办法做到这一点。你的选择是

    1.  制作一个固定大小的圆角矩形背景图，并将其放在一个固定大小的按钮上。也许干脆把文本去掉，直接把整个按钮做成一个图像。 呃。

    1.  制作一个*通用*的背景图像并缩放它以适应。更聪明，但是角落可能不会是圆的。

    1.  制作圆角矩形，剪掉角和边缘，然后将它们放在一个 3×3 的表格中，按钮标签放在中间。更好的是，使用 JavaScript 实时生成这些。

    1.  算了吧，把你整个网站做成一个大 Flash 应用 哈哈

    另一个问题是，IE 6 不理解带有 8 位 alpha 通道的 PNG 图像；它只能正确显示带有 1 位 alpha 通道的 PNG，即每个像素要么完全不透明，要么完全透明，就像 GIF 图像一样。你不得不接受锯齿边缘，把一个纯色背景色嵌入图像，或者应用各种围绕这个该死的垃圾问题的修复方案。

    |  |
    | --- |

    ```
    `filter:  progid:DXImageTransform.Microsoft.AlphaImageLoader(src='bite-my-ass.png');` 
    ```

    |

+   类似的问题还有：渐变和阴影效果！没有这些，你怎么能做出时髦的塑料按钮呢？但是这时你基本上又陷入了制作图像的困境。

+   半透明度曾经是一团糟。大多数浏览器从很早就支持 CSS 3 的 `opacity` 属性…… 除了 IE，它需要另一种怪异的微软专用 `filter` 东西。而且如果你只想让背景半透明，你需要一个半透明的 PNG，这个…… 你懂的。

+   从一开始，jQuery 就内置了诸如 `fadeIn` 等动画效果，它们开始在各处频繁出现。这有点像网络版的那个 Compiz 立方体效果，你懂的，00 年代中期每个 Linux 用户（包括我自己）都在用的那个该死的 [Compiz cube effect](https://youtu.be/4QokOwvPxrE?t=118)。

    显然，你需要 JavaScript 在大多数有趣的情况下触发元素的消失，但是使用它来控制实际的动画有点过火，会对浏览器造成压力。选项卡式浏览进一步加剧了这一点，因为浏览器大多是单线程的，出于各种原因，每个打开的页面都运行在同一个线程上。

+   哦！交替的表格行背景颜色。这已经过时了，但我认为这很遗憾，因为这确实使得表格更容易阅读。但是 CSS 对此没有解决方案，所以你要么为每一行设置一个类似 `<tr class="odd">` 的类（希望表格是通过代码生成的！），要么用一些 jQuery 的垃圾处理方法。

+   CSS 2 引入了 `>` 子选择器，所以你可以写像 `ul.foo > li` 这样的东西，来为特殊列表设置样式，而不会弄乱嵌套列表，然而 IE 6！竟然！不支持！这个！

所有这些只是审美上的问题，然而如果你对布局感兴趣，哦，Firefox 的崛起一方面让你的生活变得更容易，另一方面又让它变得更加困难。

还记得 `inline-block` 吗？实际上，Firefox 2 支持它！虽然它存在 bug，并且被隐藏在供应商前缀之后，但它基本上能用，这让设计师开始尝试使用它。然后 Firefox 3 支持得更加完整，感觉简直奇迹般。我们[缩略图网格的第三版](https://eev.ee/media/2020-02-css/thumbnail-grids.html#inline-block)就像这样简单，只需要设置一个宽度和 `inline-block`：

|  |
| --- |

```
`.thumbnails  li  { display:  inline-block; width:  250px; margin:  0.5em; vertical-align:  top; }` 
```

|

`inline-block` 的一般思想是，*内部*行为类似于块，但块本身被放置在常规流动文本中，就像图片一样。因此，每个缩略图都被包含在一个框中，但这些框都紧邻在一起，并且由于它们具有相等的宽度，它们流入一个网格中。由于它实际上是一行文本，因此您不必像处理 浮动布局 那样解决页面其他部分的奇怪影响。

当然，这种方法也有一些缺点。例如，您无法处理剩余空间，因此在某些特殊的屏幕尺寸下可能会出现右侧的大片空白。您仍然面临着单元格宽度过大而破坏网格的问题。但至少它不是 浮动布局。

有一个小问题：IE 6。它*技术上*支持`inline-block`，但仅限于自然为`inline`的元素 —— 例如 `<b>` 和 `<i>`，而不是 `<li>`。所以，你真的不会（或者不应该）认为要在这些元素上使用 `inline-block`。哎。

幸运的是，某个绝对天才在某个时候发现了 `hasLayout`，这是IE中的内部优化，用于标记一个元素是否具有…… 呃…… 布局。看，我不知道。基本上它会改变元素的渲染路径 —— 使其*不同地*存在 bug，就像每个元素都处于怪癖模式一样！总之，在IE 6中，如果添加几行代码，上述问题就会解决：

|  |
| --- |

```
`.thumbnails  li  { display:  inline-block; width:  250px; margin:  0.5em; vertical-align:  top; *zoom:  1; *display:  inline; }` 
```

|

领先的星号使属性无效，因此浏览器应该忽略整行……但由于某些我无法理解的原因，IE 6 却忽略了星号并接受了规则的其余部分。（几乎任何标点符号都有效，包括连字符或 —— 我个人最喜欢的是下划线。）`zoom` 属性是 Microsoft 的扩展，用于缩放内容，其副作用是为元素授予了“布局”神奇属性。而 `display: inline` *应该* 使每个元素的内容溢出成为一行文本，但是IE将具有“布局”的`inline`元素大致视为`inline-block`。

在这里我们看到了 CSS 混乱的真正潜力。特定于浏览器的规则，故意的糟糕语法，只有一个浏览器会忽略，以复制一个 *现在仍然无法清楚描述的效果*。[整个教程](https://blog.mozilla.org/webdev/2009/02/20/cross-browser-inline-block/)写来解释如何实现像 *网格* 这样简单的东西，但确实在大多数人的浏览器上都能工作。你还会看到 `* html`，`html > /**/ body`，以及各种其他无厘头的东西。之前的 “clearfix” hack？它的 [完整版本](https://css-tricks.com/snippets/css/clear-fix/)，兼容 *每一个* 浏览器，稍微有点*更糟*：

|

```
 1
 2
 3
 4
 5
 6
 7
 8
 9
10
11
12
13
```

|

```
`.clearfix:after  { visibility:  hidden; display:  block; font-size:  0; content:  " "; clear:  both; height:  0; } .clearfix  {  display:  inline-block;  } /* start commented backslash hack \*/ *  html  .clearfix  {  height:  1%;  } .clearfix  {  display:  block;  } /* close commented backslash hack */` 
```

|

难怪人们开始抱怨 CSS 了吧？

这是一个盲目复制/粘贴的时代，希望通过这种方式让该死的东西运行起来。以此为例：有人（我曾经找到过原始来源，但现在找不到了）提出了一个愚蠢的想法，总是设置 `body { font-size: 62.5% }`，因为“相对单位很好”，并希望覆盖看似巨大的默认浏览器字体大小 16px（事实证明，[是正确的](https://www.smashingmagazine.com/2011/10/16-pixels-body-copy-anything-less-costly-mistake/)），以及处理IE的bug。不久后他收回了这个决定，但已经造成了影响，现在**成千上万**的网站都以此为“最佳实践”开始。这意味着，如果你想在浏览器中改变默认字体大小，不管是放大还是缩小，你都很头疼 —— 缩小它，网页中的一大部分文字就会变得微小，放大它，一切仍然比你要求的要小得多，为了弥补放大，那些真正尊重你决定的内容将变得庞大。至少现在我们有更好的页面缩放功能，我猜是这样。

哦，还有一点要记住：当时还没有 Stack Overflow。这些东西完全靠口口相传传播。如果你幸运的话，你可能知道一些关于网站的网站，比如 [quirks mode](https://www.quirksmode.org/) 和 [Eric Meyer 的网站](https://meyerweb.com/)。

实际上，看看 Meyer 的 [css/edge](https://meyerweb.com/eric/css/edge/index.html) 网站，那里展示了一些人们早在 2002 年甚至只用 CSS 1 就能实现的疯狂示例。我仍然认为 [complexspiral](https://meyerweb.com/eric/css/edge/complexspiral/demo.html) 是纯粹的天才，尽管现在你可以用 `opacity` 和一个图片就做到。至于 [raggedfloat](https://meyerweb.com/eric/css/edge/raggedfloat/demo.html) 中的方法，直到几年前才在 CSS 中得到原生支持，通过 [`shape-outside`](https://developer.mozilla.org/en-US/docs/Web/CSS/shape-outside)！他还为我们带来了 [CSS reset](https://meyerweb.com/eric/tools/css/reset/)，消除了浏览器默认样式的差异。

（我无法言表Eric Meyer在CSS方面的*pioneer*身份有多么重要。当他年幼的女儿Rebecca在六年前去世时，她以自己的CSS颜色名`rebeccapurple`被独特地永恒化，这就是Web社区对他的高度评价。现在我得去哭一下那个故事了。）

设计师和开发人员正在推动浏览器的能力极限。浏览器处理这一切的方式有些粗糙。所有的修复和解决方法以及库都很神秘、脆弱、容易出错和/或过于笨重。

显然，浏览器需要一些新功能。但是随便搞点东西进去是没有用的；微软做了很多这样的事情，结果大部分都弄得一团糟。

几个挣扎的尝试开始了。尽管W3C仍然固执己见——甚至明确拒绝了对HTML提出的增强建议，转而支持XML——一些来自（活跃的）浏览器供应商苹果、Mozilla和Opera的人决定自己建立一个俱乐部。WHATWG在2004年6月成立，并开始了HTML5的工作。（它最终非常明确地定义了错误处理，完全取消了需要XHTML的需求，并消除了在处理任意HTML时的一些安全问题。此外，它还带来了一些新好玩的东西，如本地音频、视频和日期、颜色等表单控件，这些东西以前是由JavaScript驱动的自定义控件处理的。而且，呃，现在还经常这样。）

然后是CSS 3。我不确定它是什么时候开始存在的。它慢慢地出现，挣扎着，就像小鸡从蛋中孵化出来，花了一些该死的甜时间才真正被任何地方实现。

我在这里不得不做出很多有根据的猜测，但我*认为*它始于`border-radius`。具体来说，是从`-moz-border-radius`开始的。我不知道它是什么时候首次引入的，但Mozilla的Bug跟踪器中有关它的提及可以追溯到1999年。

看吧，Firefox自己的用户界面是通过CSS来渲染的。如果Mozilla想做一些CSS无法完成的事情，他们会添加自己的属性，以`-moz-`为前缀，表明这是他们自己的发明。而当没有真正的伤害时，他们也会将这个属性留给网站使用。

我的猜想是，推动CSS 3真正开始的时刻是Firefox起飞并且设计师们发现了`-moz-border-radius`。突然之间，内置的圆角就出现了！不再需要在Photoshop里到处摸索；你只需要写一行代码！几乎一夜之间，到处都是圆角。

从那里开始，事情开始迅速发展。新的CSS功能逐个解决常见问题，并被聚集到一个新的CSS版本中：CSS 3。其中的主要问题是解决之前提到的设计问题：

+   圆角，由`border-radius`提供。

+   渐变，由`linear-gradient()`和其它工具提供。

+   多重背景，虽然并非迫在眉睫的问题，但事实证明，它们确实使其他一些事情变得更容易。

+   透明度，由 `opacity` 和带有 alpha 通道的颜色提供。

+   盒子阴影。

+   文本阴影在 CSS 2 中存在，但在 2.1 中被移除，且从未被实现过。

+   边框图片，因此你可以做比简单圆角边框更花哨的事情。

+   过渡和动画，现在可以轻松实现，无需 jQuery（或任何 JavaScript）。

+   `:nth-child()`，用纯 CSS 解决交替行问题。

+   变换。等等，什么？这种功能从 SVG 中泄露出来，浏览器也被期望实现它，并且它建立在变换操作之上。代码已经存在，所以，嘿，现在我们可以用 CSS 旋转东西了！以前是做不到*的*。酷。

+   Web 字体，虽然在 CSS 中已有一段时间，但只在 IE 中实现，并且只支持一些带有版权保护的糟糕字体格式。现在我们不再受限于 Windows 自带的四种糟糕字体，也不再是其他人没有的字体了！

这些阴影非常棒！它们并未解决任何布局问题，但确实解决了设计师们曾经笨拙地通过大量图片和/或 JavaScript 绕过的美学问题。这意味着下载内容更少，而且更多地使用文本代替图片，这两者对于 Web 都是相当不错的。

最大的讽刺是，所有这些功能所能实现的东西几乎立刻就过时了，现在我们又回到了扁平矩形的时代。

哎呀！世界依然不完美。

我相信其中几个新式装置最初由浏览器供应商开发并添加了前缀。一些后来的装置由 CSS 委员会设计，但在设计仍在变动时由浏览器实现，也加入了前缀。

于是开始了*前缀地狱*，至今仍在持续。

Mozilla 有 `-moz-border-radius`，所以当 Safari 实现它时，被命名为 `-webkit-border-radius`（“WebKit”是苹果的 KHTML 分支的名称）。然后 CSS 3 规范将其标准化，并简称为 `border-radius`。这意味着如果你想使用圆角边框，实际上需要给出*三*个规则：

|  |
| --- |

```
`element  { -moz-border-radius:  1em; -webkit-border-radius:  1em; border-radius:  1em; }` 
```

|

前两个使效果在当前浏览器中真正可用，最后一个则是未来的保护措施：当浏览器实现了真正的规则并且放弃了前缀版本时，它将接管过来。

每*一次*都得这样做，因为 CSS 不是编程语言，没有宏或类似函数。有时 Opera 和 IE 会有自己的实现，带有 `-o-` 和 `-ms-` 前缀，这样总共就有五份副本。渐变色情况更加糟糕；语法经历了几次主要的不兼容修订，因此你甚至不能依赖复制/粘贴并更改属性名！

很多人都搞砸了。我不能完全责怪他们；这确实很糟糕。但是足够多的页面仅使用前缀形式，而不是最终形式，以至于浏览器不得不继续支持前缀形式的时间比他们本来想要的更长，以避免破坏一些东西。如果前缀形式仍然有效，并且这是你习惯写的方式，那么也许你还是不会使用未带前缀的那个。

更糟糕的是，*一些*人只会使用适用于他们钟爱的浏览器的形式。随着移动网络浏览器的兴起，情况变得尤为严重。iOS 和 Android 上的内置浏览器分别是 Safari（WebKit）和 Chrome（最初是 WebKit，现在是一个分支），因此你只“需要”使用 `-webkit-` 属性。这让 Mozilla 在发布 [Firefox for Android](https://www.mozilla.org/zh-CN/firefox/mobile/) 时变得更加困难。

嘿，还记得 IE 6 的整个混乱事件吗？我们又回到了这里！Mozilla 最终决定[实现](https://developer.mozilla.org/zh-CN/docs/Web/CSS/WebKit_Extensions#Supported_in_Firefox_with_-webkit-_prefix)一些 `-webkit-` 属性，即使在桌面版 Firefox 中至今仍然支持这些属性。情况变得荒谬到现在 Firefox 甚至仅通过这些属性支持某些效果，例如 [`-webkit-text-stroke`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/-webkit-text-stroke)，这种属性并未被标准化。

更糟糕的是，Chrome 当前的分支引擎被称为 Blink，所以*技术上*它不应该再使用 `-webkit-` 属性了。然而，我们还在这里。至少不像 [用户代理字符串的混乱](https://webaim.org/blog/user-agent-string-history/) 那么糟糕。

现在，浏览器供应商基本上已经放弃了前缀；取而代之的是将实验性功能隐藏在标志（flag）后面（因此它们只在开发者的机器上工作），而新功能理论上被设计得更小更容易稳定。

这种混乱可能是开发 [Sass](https://sass-lang.com/) 和 [LESS](http://lesscss.org/) 的一个巨大动机因素，这两种语言都生成 CSS。或者说… 两个 CSS 预处理器，也许是这样。它们有非常相似的目标：都添加了变量、函数和某种形式的宏到 CSS 中，允许你消除样式表中许多重复和浏览器 Hack 等无聊的东西。嘿，这个博客[仍然使用 SCSS](https://github.com/eevee/eev.ee/tree/988fc2b4547ee41388f29c4bad622c492c4c6f77/theme/static/sass)，尽管随着时间的推移其使用逐渐减少。

但是，就像天使从天而降一样… [flexbox](https://developer.mozilla.org/zh-CN/docs/Web/CSS/CSS_Flexible_Box_Layout)。

Flexbox 已经存在了相当长的时间 —— 据称它在 2006 年 Firefox 2 中就开始部分支持了！它经历了几个不兼容的修订版本并花费了很长时间才稳定下来。然后 IE 也花了很长时间来实现它，你不想依赖于只适用于一半受众的布局工具。直到相对最近的时候（2015 年？之后？），Flexbox 才有了足够广泛的支持可以安全使用。我甚至可以发誓，我现在仍然遇到一些人的当前 Safari 完全不识别它，尽管据说 Safari 五年前就取消了前缀……

总之，flexbox 是一个相当常见的 GUI 布局工具的 CSS 实现：你有一个父元素和一些子元素，父元素有一定的可用空间，并且它会自动地将可用空间分配给子元素。你知道，它*将东西并排放置*。

总体思路是，浏览器计算父元素的可用空间和每个子元素的“初始大小”，确定额外空间的多少，并根据每个子元素的灵活性分配它。想象一下工具栏：你可能希望每个按钮都有固定的大小（flex 为 0），但想要添加均匀分配任何剩余空间的间隔器，所以你会给它们一个 flex 为 1。

当这些都完成之后，你还可以使用一些提升生活质量的选项：你可以在子元素之间分配额外的空间，让子元素拉伸到相同的高度或者以各种方式对齐它们，甚至可以让它们换行显示，以便能够容纳所有的子元素！

有了这个，我们可以再次尝试那个[缩略图网格](https://eev.ee/media/2020-02-css/thumbnail-grids.html#flexbox)：

|  |
| --- |

```
`.thumbnail-grid  { display:  flex; flex-wrap:  wrap; } .thumbnail-grid  li  { flex:  1  0  250px; }` 
```

|

这真是奇迹般的事情。我一夜之间就忘记了 `inline-block`，并且大多数时间都在盼望它被普遍支持。它甚至非常清楚地表达了我想要的。

…几乎。它仍然存在一个问题，即太宽的单元格将破坏网格，因为它仍然是一个水平行包装到几个独立的行中。尽管如此，它确实非常酷，并解决了许多其他布局问题。这肯定已经足够好了。除非……？

我想说，flexbox 的大规模采用标志着现代 CSS 时代的开始。但是仍然存在一个悬而未决的问题……

IE 6 花了很长很长时间才退出舞台。直到 2010 年初左右，它的市场份额才降至不到 10%（仍然是一个相当大的份额）。

Firefox 在 2004 年底发布了 1.0 版本。而 IE 7 则是两年后才发布，虽然它带来了一些小的改进，但却存在与为 IE 6 构建的内容兼容性问题，并且那些一直使用 IE 6 的用户（其中许多并非计算机专业人士）通常认为没有升级的必要。Vista 发布时内置了 IE 7，但 Vista 可以说是个失败作品 —— 我不认为它在整个生命周期内能够超越 XP 的地位。

其他因素包括企业 IT 政策，这些政策通常采取“永远不升级任何东西”的形式 —— 这通常是有充分理由的，因为我听到了无数关于内部应用程序只能在 IE 6 中运行的可怕故事。然后还有*整个韩国*，因为他们在法律中确立了一些[安全要求](https://www.washingtonpost.com/world/asia_pacific/due-to-security-law-south-korea-is-stuck-with-internet-explorer-for-online-shopping/2013/11/03/ffd2528a-3eff-11e3-b028-de922d7a3f47_story.html)，只能通过 IE 6 的 ActiveX 控件实施。

如果你在维护一个网站，它被商业人士或居住在其他国家的人们使用 — 或者更糟，*需要*他们使用 — 你几乎必须支持 IE 6\. 制作个人工具和网站的人们早早就放弃了对 IE 6 的兼容性，并在他们的网站上贴满了越来越令人讨厌的横幅，嘲笑任何敢于使用它的人…… 但如果你是某人的老板，你为什么要告诉他们可以放弃潜在观众的 20%？只能更加努力工作！

多年来，随着 CSS 变得更加强大，IE 6 仍然是个累赘。它甚至仍然不能理解 PNG alpha，而与此同时我们开始获取更关键的功能，如 HTML5 中的本地视频。解决方案变得更加混乱，而我们基本上无法使用的功能列表变得更长。 （我可以向你展示我的博客在 IE 6 中的样子，但我认为它甚至不能连接 —— 它支持的 TLS 版本如此古老和破损，以至于大多数服务器已经将其禁用！）

顺便说一句，致敬 YouTube 团队的一些人，他们在 2009 年 7 月[添加了警告横幅](https://www.theverge.com/2019/5/4/18529381/google-youtube-internet-explorer-6-kill-plot-engineer)，恳请 IE 6 用户切换至*任何*其他浏览器 —— 而不需要任何人的批准。“一个月内…… 全球超过 10% 的 IE6 流量消失了。” 不是所有的英雄都穿斗篷。

我认为末日的开始是 YouTube *实际上*停止支持 IE 6 的那一天 — 2010 年 3 月 13 日，几乎是其发布后的九年。我不知道 YouTube 对企业用户或韩国政府有多大*直接*影响，但一个庞大的网络公司放弃整个浏览器确实传递了一个非常强有力的信息。

当然，IE 还有其他版本，许多版本都是自成一派的头痛。但随着每一个后续版本变得更少问题，如今你甚至不用太过考虑在 IE（现在是 Edge）中进行测试了。正好在微软放弃他们自己的渲染引擎，将他们的浏览器变成 Chrome 的克隆时。

现在的 CSS 真的很棒。您不需要奇怪的破解方法来将东西放在一起。现在浏览器开发工具已经内置，并且非常棒 —— Firefox 已经开始专门警告您，因为某些 CSS 属性的值可能导致其他属性失效！像“堆叠上下文”（不管那是什么）这样的隐式副作用现在可以显式设置，例如 `isolation: isolate`。

实际上，让我列出我现在认为您可以在 CSS 中做的所有事情。这不是关于样式所有可能用途的指南，但如果您的 CSS 知识自 2008 年以来没有更新过，我希望这能激起您的兴趣。而这些东西只是 CSS！现在支持的功能如此之多，曾经不可能或痛苦或需要笨拙插件的事情现在都得到了本地支持 —— 音频、视频、自定义绘图、3D 渲染……更不用说对 JavaScript 的巨大人体工程学改进了。

[Grid](https://developer.mozilla.org/zh-CN/docs/Web/CSS/CSS_Grid_Layout) 容器几乎可以做任何表格能做的事情，包括自动确定可以容纳多少列。这太神奇了。有关更多信息，请参见下文。

[Flexbox](https://developer.mozilla.org/zh-CN/docs/Web/CSS/CSS_Flexible_Box_Layout) 容器可以将其子元素排列成行或列，允许每个子元素声明其“默认”大小以及希望消耗的剩余空间比例。Flexbox 可以换行，重新排列子元素而不改变源顺序，并以多种方式对齐子元素。

[Columns](https://developer.mozilla.org/zh-CN/docs/Web/CSS/CSS_Columns) 将文本分布到多个列中。

[`box-sizing`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/box-sizing) 属性允许您根据每个元素的需求选择 IE 盒模型，当您需要整个元素占用固定空间并且需要填充/边框*减去*时。

[`display: contents`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/display) 将一个元素的内容“倒出”到其父级中，就好像它根本不存在一样。`display: flow-root` 基本上是自动的 clearfix，只是晚了十年。

[`width`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/width) 现在可以设置为 `min-content`, `max-content` 或 `fit-content()` 函数，以实现更灵活的行为。

[`white-space: pre-wrap`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/white-space) 保留空白字符，但在必要时断开行以避免溢出。`pre-line` 也很有用，它将连续的空格折叠成单个空格，但保留字面换行符。

[`text-overflow`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/text-overflow) 在溢出文本时使用省略号（或自定义字符）进行截断，而不是简单地截断文本。规范还指定了淡化文本的能力，但这还没有实现。

[`shape-outside`](https://developer.mozilla.org/en-US/docs/Web/CSS/shape-outside) 改变在浮动文本周围包裹时使用的形状。它甚至可以使用图像的 alpha 通道作为形状。

[`resize`](https://developer.mozilla.org/en-US/docs/Web/CSS/resize) 为任意元素提供一个调整大小的手柄（只要它具有`overflow`）。

[`writing-mode`](https://developer.mozilla.org/en-US/docs/Web/CSS/writing-mode) 设置文本流动的方向。如果您的设计需要适应多种书写模式，一些 CSS 属性提到左/右/上/下的替代项描述了书写模式的方向：[`inset-block`](https://developer.mozilla.org/en-US/docs/Web/CSS/inset-block) 和 [`inset-inline`](https://developer.mozilla.org/en-US/docs/Web/CSS/inset-inline) 用于位置，[`block-size`](https://developer.mozilla.org/en-US/docs/Web/CSS/block-size) 和 [`inline-size`](https://developer.mozilla.org/en-US/docs/Web/CSS/inline-size) 用于宽度/高度，[`border-block`](https://developer.mozilla.org/en-US/docs/Web/CSS/border-block) 和 [`border-inline`](https://developer.mozilla.org/en-US/docs/Web/CSS/border-inline) 用于边框，以及类似的填充和边距。

[过渡效果](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Transitions) 在值变化时平滑地插值，无论是由于像`:hover`这样的效果还是例如从 JavaScript 添加的类。[动画](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Animations) 类似，但会自动播放预定义的动画。两者都可以使用多种不同的[缓动函数](https://easings.net/en)。

[`border-radius`](https://developer.mozilla.org/en-US/docs/Web/CSS/border-radius) 圆角化框的角落。角落可以是不同的尺寸，并且可以是圆形或椭圆形。曲线还适用于边框、背景和任何框阴影。

[盒子阴影](https://developer.mozilla.org/en-US/docs/Web/CSS/box-shadow) 可用于投射阴影的显而易见的效果。您还可以使用多个阴影和`inset`阴影来实现各种巧妙的效果。

[`text-shadow`](https://developer.mozilla.org/en-US/docs/Web/CSS/text-shadow) 就像名字所说的那样，尽管您也可以堆叠几个以粗略近似文本轮廓。

[`transform`](https://developer.mozilla.org/en-US/docs/Web/CSS/transform) 允许您对元素应用任意矩阵变换 — 您可以进行缩放、旋转、倾斜、平移和/或透视变换，而不会影响布局。

[`filter`](https://developer.mozilla.org/en-US/docs/Web/CSS/filter)（与 IE 6 不同）提供了几种特定的视觉滤镜，可以应用于元素。大多数影响颜色，但也有`blur()`和`drop-shadow()`（与`box-shadow`不同，它适用于元素的外观而不是其包含框）。

[`linear-gradient()`](https://developer.mozilla.org/en-US/docs/Web/CSS/linear-gradient)、[`radial-gradient()`](https://developer.mozilla.org/en-US/docs/Web/CSS/radial-gradient)，以及新的支持较少的 [`conic-gradient()`](https://developer.mozilla.org/en-US/docs/Web/CSS/conic-gradient)，以及它们的 `repeating-*` 变体，都生成渐变图像，可以在 CSS 的任何需要图像的地方使用，最常见的是作为 `background-image`。

[`scrollbar-color`](https://developer.mozilla.org/en-US/docs/Web/CSS/scrollbar-color) 可以改变滚动条的颜色，但当前浏览器的一个缺点是将滚动条简化为非常简单的拇指和轨道。

[`background-size: cover` 和 `contain`](https://developer.mozilla.org/en-US/docs/Web/CSS/background-size) 将按比例缩放背景图像，要么足够大以完全覆盖元素（即使被裁剪），要么足够小以确切地适合其中（即使不覆盖整个背景）。

[`object-fit`](https://developer.mozilla.org/en-US/docs/Web/CSS/object-fit) 是一个类似的概念，但适用于非背景媒体，如 `<img>`。相关的 [`object-position`](https://developer.mozilla.org/en-US/docs/Web/CSS/object-position) 类似于 `background-position`。

可以使用多个背景，尤其是与渐变一起使用 —— 可以堆叠多个渐变、其他背景图像和底部的纯色。

[`text-decoration`](https://developer.mozilla.org/en-US/docs/Web/CSS/text-decoration) 比以前更加精致；你现在可以设置线条的颜色并使用多种不同类型的线条，包括虚线、点线和波浪线。

[CSS 计数器](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Lists_and_Counters/Using_CSS_counters) 可以用来以任意方式对任意元素编号，将 `<ol>` 的计数能力暴露给任何你想要的元素集。

[`::marker`](https://developer.mozilla.org/en-US/docs/Web/CSS/::marker) 伪元素允许你样式化列表项的标记框，甚至可以用自定义计数器完全替换它。浏览器支持不太稳定，但正在改进。类似地，[`@counter-style`](https://developer.mozilla.org/en-US/docs/Web/CSS/@counter-style) at-rule 实现了全新的计数样式（如 1 2 3、i ii iii、A B C 等），然后你可以在任何地方使用它，尽管目前只有 Firefox 支持。

[`image-set()`](https://developer.mozilla.org/en-US/docs/Web/CSS/image-set) 提供了候选图像列表，并根据用户屏幕的像素密度选择最合适的一个。

[`@font-face`](https://developer.mozilla.org/en-US/docs/Web/CSS/@font-face) 定义了一个可下载的字体，虽然你可以通过使用 [Google Fonts](https://developers.google.com/fonts/) 避免弄清如何正确使用它。

[`pointer-events: none`](https://developer.mozilla.org/en-US/docs/Web/CSS/pointer-events) 使元素完全忽略鼠标；它无法悬停，点击事件会直接穿过它传递到下面的元素。

[`image-rendering`](https://developer.mozilla.org/en-US/docs/Web/CSS/image-rendering) 可以强制图像使用最近邻插值法而不是插值处理，尽管浏览器支持仍不稳定，可能还需要包含一些特定厂商的属性。

[`clip-path`](https://developer.mozilla.org/en-US/docs/Web/CSS/clip-path) 将元素裁剪为任意形状。还有 [`mask`](https://developer.mozilla.org/en-US/docs/Web/CSS/mask) 用于任意Alpha蒙版，但浏览器支持参差不齐，而且这个功能非常复杂。

[`@supports`](https://developer.mozilla.org/en-US/docs/Web/CSS/@supports) 允许您根据浏览器的支持情况明确地编写不同的CSS，尽管现在它不如2004年那样有用。

`A > B` 选择直接子元素。 `A ~ B` 选择兄弟元素。 `A + B` 选择直接（元素）兄弟。方括号可以根据属性选择 [许多内容](https://developer.mozilla.org/en-US/docs/Web/CSS/Attribute_selectors)；最明显的是 `input[type=checkbox]`，尽管您还可以通过匹配 `<a href>` 的部分进行有趣的操作。

现在有一大堆伪类。其中许多是针对表单元素的：[`:enabled`](https://developer.mozilla.org/en-US/docs/Web/CSS/:enabled) 和 [`:disabled`](https://developer.mozilla.org/en-US/docs/Web/CSS/:disabled); [`:checked`](https://developer.mozilla.org/en-US/docs/Web/CSS/:checked) 和 [`:indeterminate`](https://developer.mozilla.org/en-US/docs/Web/CSS/:indeterminate)（也适用于单选按钮和 `<option>`）; [`:required`](https://developer.mozilla.org/en-US/docs/Web/CSS/:required) 和 [`:optional`](https://developer.mozilla.org/en-US/docs/Web/CSS/:optional); [`:read-write`](https://developer.mozilla.org/en-US/docs/Web/CSS/:read-write) 和 [`:read-only`](https://developer.mozilla.org/en-US/docs/Web/CSS/:read-only); [`:in-range`](https://developer.mozilla.org/en-US/docs/Web/CSS/:in-range)/[`:out-of-range`](https://developer.mozilla.org/en-US/docs/Web/CSS/:out-of-range) 和 [`:valid`](https://developer.mozilla.org/en-US/docs/Web/CSS/:valid)/[`:invalid`](https://developer.mozilla.org/en-US/docs/Web/CSS/:invalid)（用于 HTML5 客户端表单验证）; [`:focus`](https://developer.mozilla.org/en-US/docs/Web/CSS/:focus) 和 [`:focus-within`](https://developer.mozilla.org/en-US/docs/Web/CSS/:focus-within); 以及 [`:default`](https://developer.mozilla.org/en-US/docs/Web/CSS/:default)（选择默认表单按钮和任何预选复选框、单选按钮和 `<option>`）。

用于定位兄弟元素中特定元素的选择器有：[`:first-child`](https://developer.mozilla.org/en-US/docs/Web/CSS/:first-child)，[`:last-child`](https://developer.mozilla.org/en-US/docs/Web/CSS/:last-child)，和 [`:only-child`](https://developer.mozilla.org/en-US/docs/Web/CSS/:only-child); [`:first-of-type`](https://developer.mozilla.org/en-US/docs/Web/CSS/:first-of-type)，[`:last-of-type`](https://developer.mozilla.org/en-US/docs/Web/CSS/:last-of-type)，和 [`:only-of-type`](https://developer.mozilla.org/en-US/docs/Web/CSS/:only-of-type)（其中“type”表示标签名）；以及 [`:nth-child()`](https://developer.mozilla.org/en-US/docs/Web/CSS/:nth-child)，[`:nth-last-child()`](https://developer.mozilla.org/en-US/docs/Web/CSS/:nth-last-child)，[`:nth-of-type()`](https://developer.mozilla.org/en-US/docs/Web/CSS/:nth-of-type)，和 [`:nth-last-of-type()`](https://developer.mozilla.org/en-US/docs/Web/CSS/:nth-last-of-type)（用于选择每第二个、第三个等元素）。

[`:not()`](https://developer.mozilla.org/en-US/docs/Web/CSS/:not) 反转选择器。 [`:empty`](https://developer.mozilla.org/en-US/docs/Web/CSS/:empty) 选择没有子元素和文本的元素。 [`:target`](https://developer.mozilla.org/en-US/docs/Web/CSS/:target) 选择通过URL片段跳转到的元素（例如，如果地址栏显示 `index.html#foo`，则选择ID为 `foo` 的元素）。

[`::before`](https://developer.mozilla.org/en-US/docs/Web/CSS/::before) 和 [`::after`](https://developer.mozilla.org/en-US/docs/Web/CSS/::after) 现在应该使用两个冒号，表示它们创建伪元素，而不仅仅是作用于附加的选择器。 [`::selection`](https://developer.mozilla.org/en-US/docs/Web/CSS/::selection) 自定义选定文本的外观； [`::placeholder`](https://developer.mozilla.org/en-US/docs/Web/CSS/::placeholder) 自定义占位文本（在文本字段中）的外观。

[媒体查询](https://developer.mozilla.org/en-US/docs/Web/CSS/@media)可以根据页面的查看方式进行适应，执行各种操作。[`prefers-color-scheme`](https://developer.mozilla.org/en-US/docs/Web/CSS/@media/prefers-color-scheme) 媒体查询会告诉您用户系统是设置为浅色主题还是深色主题，因此您可以相应地调整而无需询问。

您可以将半透明的[颜色](https://developer.mozilla.org/en-US/docs/Web/CSS/color_value)表示为`#rrggbbaa`或`#rgba`，也可以使用`rgba()`和`hsla()`函数。

角度可以用`turn`单位描述为一个完整圆的分数。当然，也可以使用`deg`和`rad`（以及`grad`）。

[CSS 变量](https://developer.mozilla.org/en-US/docs/Web/CSS/Using_CSS_custom_properties)（官方称为“自定义属性”）允许你指定任意命名的值，可以在任何需要值的地方使用。你可以使用这个功能在 JavaScript 中减少 CSS 调整（例如，通过设置 CSS 变量而不是手动调整多个属性来重新着色页面的复杂部分），或者拥有一个通用组件，该组件会根据祖先设置的变量做出反应。

[`calc()`](https://developer.mozilla.org/en-US/docs/Web/CSS/calc) 计算任意表达式并自动更新（尽管 `box-sizing` 有点重复它）。

[`vw`, `vh`, `vmin` 和 `vmax` 单位](https://developer.mozilla.org/en-US/docs/Web/CSS/length) 允许你将长度指定为视口宽度或高度的一部分，或者两者中较大/较小的那个。

* * *

哦，我确定我遗漏了很多内容，大家在评论中可能有更长的有趣小贴士列表。感谢你帮我节省了一些努力！现在我可以停止浏览 MDN，做这最后有趣的部分。

终于，我们来到构建缩略图网格的最终且客观正确的方式：使用 [CSS 网格](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Grid_Layout)。你可以确定这是正确的选择，因为名字里有“grid”。现代 CSS 功能非常棒，让你说出你想要的事情，然后它就会发生，而不是试图通过巫术来间接发生。

而且，这很简单：

|  |
| --- |

```
`.thumbnail-grid  { display:  grid; grid:  auto-flow  /  repeat(auto-fit,  minmax(250px,  1fr)); }` 
```

|

完成！这样就给你一个[网格](https://eev.ee/media/2020-02-css/thumbnail-grids.html#grid)。你有无数其他小技巧可以使用，就像 flexbox 一样，但基本思想是一样的。你甚至不需要为元素本身设置样式；大部分布局工作都在容器中完成。

[`grid` 缩写属性](https://developer.mozilla.org/en-US/docs/Web/CSS/grid) 看起来有点吓人，但只是因为它非常灵活。它的意思是：一行一行填充网格，生成所需数量的行；创建尽可能多的 250px 宽度列，并平均分配剩余的空间。

CSS 网格对布局 `<dl>` 非常方便，历史上制作这个东西非常痛苦——一个 `<dl>` 包含任意数量的 `<dt>`，后面跟任意数量的 `<dd>`（包括零个），直到出现网格之前唯一的样式方式是浮动 `<dt>`，这意味着它们必须有固定的宽度。现在你可以告诉 `<dt>` 放在第一列，`<dd>` 放在第二列，网格会处理其余的部分。

对页面进行布局？整个侧边栏的事情？看看这是多么简单：

|

```
 1
 2
 3
 4
 5
 6
 7
 8
 9
10
11
12
13
14
15
16
```

|

```
`body  { display:  grid; grid-template: "header         header          header" "left-sidebar   main-content    right-sidebar" "footer         footer          footer" /  1fr  6fr  1fr ; } body  >  header  { grid-area:  header; } #left-sidebar  { grid-area:  left-sidebar; } /* ... etc ... */` 
```

|

完成。简单。标记中出现部分的顺序都不重要。

网页仍然*有点*混乱。很多人甚至不知道flexbox和grid现在已经几乎是[普遍支持的](https://www.caniuse.com/#feat=css-grid)，但考虑到从早期规范工作到广泛实施花了多长时间，我也不能完全责怪他们。昨天我看到一个全新的小网站，主要是一个巨大的各种宽度“缩略图”列表，并且它使用的是浮动布局！甚至不是`inline-block`！我不知道我们是如何设法教导每个人如何使用所有必要的技巧来使其工作的，但却无法普及flexbox的。  

但比这更糟糕的是：我仍然经常遇到使用*JavaScript*来完成整个页面布局的网站。如果你使用[uMatrix](https://addons.mozilla.org/en-US/firefox/addon/umatrix/)，你的第一印象是一堆文本重叠在另一堆文本上。这肯定是一种倒退吧？你到底在做什么，让你的页眉和侧边栏只能通过执行代码才能正确布局？这不像页面加载时*没有*CSS —— 没有纯HTML中的东西会默认重叠！你必须告诉它去做这些！

还有移动网络，尽管每个人的良好意图，但似乎已经失败了。最初的想法是使用CSS媒体查询将正常的网站适应手机屏幕，但大多数主要网站都有完全独立的移动版本。这意味着移动网站要么缺少一些重要功能，我仍然需要在手机上笨拙地导航，要么桌面网站充满了没人真正需要的垃圾。

（同时，Google自己的Android版本的Docs/Sheets等，像是网页版的功能的5%？不太清楚怎么看这个。）

嗯。强烈考虑写点东西，更详细地讲述自Firefox 3时代以来CSS改进的事情，类似于[我为JavaScript写的那篇文章](https://eev.ee/blog/2017/10/07/javascript-got-better-while-i-wasnt-looking/)。但这篇文章已经够长了。

我不知道 CSS 中接下来会发生什么，特别是现在 flexbox 和 grid 已经解决了我们所有的问题。我模糊地意识到一些更深入的数学支持正在进行中，可能还有一些类似 Sass 中颜色修改的函数。有一个 [painting API](https://developer.mozilla.org/en-US/docs/Web/API/CSS_Painting_API) 允许你使用 canvas API 动态生成背景，这是…相当了不起。显然现在规范中允许你使用 [`attr()`](https://developer.mozilla.org/en-US/docs/Web/CSS/attr)（它计算为 HTML 属性的值）作为任何属性的值，这看起来很酷，甚至可能让你完全用 CSS 实现 HTML 表格，但你也可以用变量来做同样的事情。我是说，嗯，自定义属性。我对 [`:is()`](https://developer.mozilla.org/en-US/docs/Web/CSS/:is) 很感兴趣，它匹配任何选择器列表中的一个，以及 [subgrid](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Grid_Layout/Subgrid)，它允许你向网格添加一些嵌套，但保持子元素仍然对齐于它。

要列出一些本来是未来趋势但却失败了的事情要容易得多。

+   [`display: run-in`](https://developer.mozilla.org/en-US/docs/Web/CSS/display) 自 CSS 版本 2（早在 '98 年）就存在，但基本上不被支持。这个想法是一个“run-in”框被插入，内联地，到下一个块中，所以像这样：

    |  |
    | --- |

    ```
    `<h2 style="display: run-in;">Title</h2>
    <p>Paragraph</p>
    <p>Paragraph</p>` 
    ```

    |

    显示如下：

    > **标题** 段落
    > 
    > 段落

    啊，嗯，我开始明白为什么它不被支持了。它*曾经*存在于 WebKit 中，但显然是如此不可行以至于六年前被移除了。

+   “替代样式表” 在 00 年代初非常流行，至少在我几个朋友的网站上是这样。这个想法是你可以为你的网站列出*多个*样式表（大概是为了不同的主题），然后浏览器会给用户一个列表。可惜的是，这个列表总是藏在一个菜单中，没有明显的指示何时实际上被填充，因此最后，所有希望有多个主题的人只好自己实现页面内的主题切换器。

    这个功能 [仍然支持](https://developer.mozilla.org/en-US/docs/Web/CSS/Alternative_style_sheets)，但显然 Chrome 从未费心去实现它，因此它实际上已经死了。

+   更一般地说，原始的 CSS 规范显然期望用户能够为网站编写他们自己的 CSS — 在第二段中明确地说

    > …读者可以有一个个人样式表来调整人类或技术上的障碍。

    嘿，听起来很酷。但它从未作为浏览器功能实现。Firefox 有 [`userContent.css`](http://kb.mozillazine.org/UserContent.css) 和一些 URL 选择器用于编写每个站点的规则，但相对较为晦涩。

    仍然显然有对这个概念的需求，正如Stylish扩展的流行所证明的那样 —— 它正是这样做的（太糟糕了，它被一些蠢货买走，开始用它来窃取浏览器数据并出售给广告商）。改用[Stylus](https://addons.mozilla.org/en-US/firefox/addon/styl-us/)。

+   对我来说，一个常见的问题是，根据复选框的状态为其*标签*设置样式。使用`:checked`伪选择器很容易为复选框本身添加样式。但如果你按照显而易见的方式布置复选框和其标签：

    |  |
    | --- |

    ```
    `<label><input type="checkbox"> Description of what this does</label>` 
    ```

    |

    …然后CSS没有办法同时定位到`<label>`元素或文本节点。jQuery的（最初是自定义的）选择器引擎提供了一个自定义的`:has()`伪类，可以用来表达这一点：

    |  |
    | --- |

    ```
    `/* checkbox label turns bold when checked */ label:has(input:checked)  { font-weight:  bold; }` 
    ```

    |

    早期CSS 3选择器讨论显然想避免这种情况，我想是出于性能原因？一种略显新颖的替代方案是完整地写出整个选择器，但可以用“subject”指示符改变规则影响的部分。起初这是一个伪类：

    |  |
    | --- |

    ```
    `label:subject  input:checked  { font-weight:  bold; }` 
    ```

    |

    后来，他们引入了一个`!`前缀：

    |  |
    | --- |

    ```
    `!label  input:checked  { font-weight:  bold; }` 
    ```

    |

    幸运的是，这被认为是一个坏主意，所以当前规范化的方法是……[`:has()`](https://developer.mozilla.org/en-US/docs/Web/CSS/:has)! 不幸的是，它只允许在JavaScript中查询时使用，并且无论如何都没有实现。20年过去了，我仍在等待一种方法来为复选框的标签设置样式。

+   `<style scoped>`是一个属性，本来可以使`<style>`元素的CSS规则只适用于其直接父元素内的其他元素，这意味着您可以插入任意（可能是用户编写的）CSS而不会影响页面的其他部分。唉，这些已经在一段时间前悄悄地被取消了，而shadow DOM则被建议作为一个极其不合适的替代品。

+   我记得当我第一次听说[Web components](https://developer.mozilla.org/en-US/docs/Web/Web_Components)时，它们是你可以使用的模板，用于减少纯HTML中的重复？但现在我找不到这个概念的任何踪迹，当前的实现需要JavaScript来定义它们，所以没有什么声明性的东西将一个新标签链接到它的实现。这使得它们对于任何没有充分理由依赖JS的东西来说完全无法使用。唉。

+   `<blink>`和`<marquee>`。安息吧。虽然两者都可以很容易地用CSS动画复制。

你还在这里？结束了。回家吧。

并且也许抵制Blink单一文化，使用[Firefox](https://www.mozilla.org/en-US/firefox/)，包括[在你的手机上](https://www.mozilla.org/en-US/firefox/mobile/)，除非你使用iPhone，它禁止其他浏览器引擎，这比微软曾经做过的任何事情更糟糕，但我们只是某种原因接受了它。
