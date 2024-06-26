<!--yml

分类：未分类

日期：2024 年 05 月 27 日 14:32:24

-->

# 如何使用 2GB 的 VPS 每月为数百万请求提供服务；或者，对经典网络的赞美 - Alex Cabal

> 来源：[`alexcabal.com/posts/standard-ebooks-and-classic-web-tech`](https://alexcabal.com/posts/standard-ebooks-and-classic-web-tech)

# 如何使用 2GB 的 VPS 每月为数百万请求提供服务；或者，对经典网络的赞美

2022 年 2 月 11 日

[Standard Ebooks](https://standardebooks.org) 是一个项目，它接收公有领域文学的抄本，例如通常在 [Project Gutenberg](https://gutenberg.org) 上可用的那种，然后使用 [详细的样式手册](https://standardebooks.org/manual) 创建美丽、现代的电子书。来自世界各地的志愿者致力于制作这些电子书，然后我们免费发布它们，并且没有版权限制。

美国公有领域中的图书通常是指至少发表了 95 年的图书。（写作时，这意味着在 1927 年之前发表。）你可以想象，我们花了大量时间处理非常古老的书籍。那么为什么不用非常古老的技术创建一个网站呢？

在 2021 年，Standard Ebooks 服务器提供了约 140 万次电子书下载和 1550 万次页面浏览。这意味着每月超过 120 万次页面浏览，这还*不包括对我们的 [OPDS](https://en.wikipedia.org/wiki/Open_Publication_Distribution_System) 和 RSS 订阅的请求*，这也是以百万计的。在过去几年里，它至少 [三次](https://news.ycombinator.com/item?id=25138534) [登上过](https://news.ycombinator.com/item?id=20594802) [Hacker News 的首页](https://news.ycombinator.com/item?id=14570035)，而且毫不费力。我们用一个微小的 VPS 提供了这些服务，它的网站上甚至没有 JavaScript ——甚至没有数据库！

我们获得的流量*并不算*太疯狂。但这证明了如何使用廉价的经典网络技术可以提供出人意料的大量流量，而不依赖于我们许多人在启动新项目时所倚赖的任何技术支撑。

## SE 服务器的功能

SE 使用一台单核心 VPS 和 2GB RAM 运行其全部操作。它负责：

1.  托管我们所有电子书的 Git 仓库，作为我们生成的电子书的真实来源。 (虽然我们也在 GitHub 上托管我们所有的电子书，*GitHub 实际上是一个镜像*，而不是我们电子书的原始来源。我们认为保持本地控制是重要的，并且自主托管一个类似 GitHub 的解决方案，允许公众查看电子书源并提交更正，而不是依赖于 GitHub，这是我们长期路线图上的一部分。)

1.  在电子书发布或更新时运行电子书构建流程。

1.  提供 SE 网站服务，包括 OPDS 和 RSS 订阅。

[Standard Ebooks 网站](https://github.com/standardebooks/web)运行在经典的 LAP 技术栈上。什么？LAP？没错：LAMP 没有 M。*甚至在 SE 网站后面都没有传统的数据库。*

## 为什么选择 PHP？

我现在写 PHP 已经快二十年了。这是一种我熟悉的语言和范例。随着年龄的增长，人对新技术的热情越来越少——毕竟，当人必须钉钉子时，只有这么多种不同的锤子可以让人感到兴奋，然后人就不再在乎，直接拿起最常用的工具。所以当我想要开始一个新的网站时，我选择 PHP。

幸运的是，按照某种标准，PHP 是网络 78%的服务器端语言的选择，所以将你的车厢拴在这样一种流行的语言上意味着未来几十年至少会很容易找到人来维护它。

现代 PHP 并不算太差，而且与一些竞争对手相比，它至少有两个主要优点：

1.  每个请求都是一个完全独立的请求，重新构建整个世界。没有共享状态（除非你希望有）。

1.  它是一种解释性语言，所以在部署时不需要重新编译代码，也不需要重新启动服务器。你在脚本中进行更改，保存文件，刷新浏览器，就完成了。这使得开发和调试*非常快*，部署*非常容易*。

许多人也不知道的是，PHP 本身也是一种相当不错的模板语言！虽然很容易假设任何 PHP 网页项目至少需要一个[大型](https://laravel.com/) [框架](https://symfony.com/)，但事实上，使用基本的 PHP 作为自己简单的模板系统是很容易的。对于许多基本的经典 web 网站，比如 SE，这通常足以完成工作；而最大的好处是，你不必被困在不得不学习和*维护*庞大的第三方框架的困境中。

我认为人们真的低估了在*网站开发*完成后甚至在*网站维护*期间维护第三方框架的负担。即使你的网站只存在短短五年，你将不得不多少次更新框架，即使只是出于安全更新，并跟上 API 更新，以确保没有破坏性变化。对于本质上我永远不想再见到的脚手架来说，一次更新都太多了。

## SE 服务器如何处理所有事务而不崩溃

### 处理 web 请求

请求从 Apache 开始，通常通过 `mod_rewrite` 重写 URL——例如，将电子书路径重写为带有查询字符串的 PHP 脚本。这在某种程度上等同于服务器端框架中的路由器，只不过在 PHP 进入之前就完成了。

一旦 Apache 决定要运行哪个 PHP 脚本，它就会将其转发给 PHP-FPM。[PHP-FPM](https://php-fpm.org/) 是多年前取代了不堪重负的 `mod_php` 的进程管理器，也是现代 PHP 网站能够如此快速的主要原因之一。

一旦请求到达 PHP，我们就开始使用 [非常非常小的基于文件的模板系统](https://github.com/standardebooks/web/blob/master/lib/Template.php) 为浏览器创建输出。它使用原始 PHP 而不是自定义的模板语法。正如我之前提到的，PHP 本身已经是一个令人惊讶的好模板语言。为什么要重新发明轮子呢？

### 没有数据库的动态内容

在请求的这一点上，我们可能需要在将数据呈现给浏览器之前查找一些数据。这就是人们可能会查询数据库的地方，但我们没有数据库。那么我们该怎么做呢？

你可能会记得 SE 服务器还托管了我们所有电子书的 Git 仓库。这些电子书包含丰富的元数据，包括标题、作者、描述、字数、系列信息等等。由于我们的电子书语料库相对较小（在写作时大约有 630 本电子书），*我们能够将它们全部放入内存中*：具体来说，使用 PHP 内置的 [APCu 对象缓存](https://www.php.net/manual/en/book.apcu.php)。

这个缓存在 PHP 进程间是持久的，所以我们只需要在 PHP-FPM 第一次启动时初始化它一次，并且只需要在电子书更新时更新它。如果 PHP 进程崩溃或服务器重新启动，网站会在下次启动时从头开始重建缓存。构建这个缓存确实需要几秒钟，但由于崩溃和重新启动很少发生，这并不是什么大问题。

你可能会觉得这是对动态网站相当愚蠢的方法。为什么不直接使用数据库呢？

我并不完全不同意。但这样做是因为 SE 起初是一个小型爱好项目，而且在很多方面仍然是这样，小型爱好项目希望把时间花在主要事情上（电子书）而不是次要事情上（网站）。

要构建一个网站，我们将 *已经* 必须解析我们的电子书中的 XML，哪怕只是为了将其插入数据库以供以后查找。如果我们已经在 PHP 中创建了电子书对象，为什么不直接跳过数据库、ORM 层、粘合代码、数据模型和所有其他附属内容，而只是简单地缓存电子书对象本身，以节省这项爱好的开发时间呢？

结果证明，对于我们的小型数据集和非常基本的数据处理要求，这种方法表现得非常好。当然，我们 *最终* 会达到需要真正的数据库的地步，但我们可能首先会达到这一点，因为我们希望我们的网站做更复杂的事情，而不是因为我们的电子书语料库的大小需要数据库。

与此同时，我们仍然可以正常服务数百万的页面浏览。

### 不需要 Javascript

如果我们还没有用我们深深的疑问的设计决策震惊你，那么这一点可能会最让年轻的开发人员震惊：我们的网站不使用任何 Javascript。零。 (好吧，除了一个指向 `manifest.json` 文件的链接，因为没有别的办法。)

就内容而言，SE 网站非常符合经典 Web 网站的特点。有一个主页，一个带有分页列表的搜索页面，这些项目的详细视图，在某些地方有一些非常基本的表单，以及一些辅助的、大多是静态页面。碰巧的是，纯 HTML 加上一些巧妙的 CSS 几乎拥有所有这种网站所需的功能：

+   样式、动画、过渡和响应式功能都由纯 CSS 处理。

+   表单是普通的 HTML，通过一个无聊的提交按钮发送常规的 GET 和 POST 请求。我们不需要花哨的 AJAX 或加载旋转器——当你点击提交时，你的浏览器可能已经在某个地方显示了一个加载旋转器。

+   没有客户端跟踪代码，也没有广告。

+   所有的东西在到达您的浏览器之前都是服务器端渲染的。

在短短的几年内，服务器端渲染从几乎普遍的默认值变成了一种古怪的遗物，现在 Javascript 是一个“显而易见”的必需品，这是一个一切都是由巨大的第三方 Javascript 框架渲染为客户端的 API 的地方。但是这种新的范式是 *慢的*：提供数兆字节的 Javascript 是慢的，等待 API 响应是慢的，客户端解析响应并生成页面是慢的。

更令人气愤的是，往往 *你要做两次的工作*：一次在服务器上解析、组装和输出数据，再次在客户端解析、组装和渲染数据。

而且 JS 框架，就像 PHP 框架一样，是另一个你需要无限期维护的第三方依赖项，即使你的网站不再活跃更新。一个曾经流行的 JS 框架多少次会被最新的热门技术所取代？

不仅如此，使用精心选择的纯 HTML 在源头上输出[可访问的网页](https://developer.mozilla.org/en-US/docs/Learn/Accessibility/HTML)比在事后用客户端 JS 组合元素要容易得多。

JS 框架对于一些更大、*非常复杂的应用程序*确实有意义。但是真正合法的 *非常复杂的应用程序* 寥寥无几。往往，它们只是[又一个 CRUD 应用程序](https://en.wikipedia.org/wiki/Create,_read,_update_and_delete)——经典 Web 为之设计和构建的基于文档、异步模型。

而且，尽管通过使用 JS 框架开始开发一个新项目可能会看起来更快，但通过使用你选择的语言的服务器端框架开始开发一个新项目可能会同样快。

我不是一个反对 JS 的狂热者。明智地使用 Javascript 绝对可以增强用户体验。但是整个东西需要用 React 来做吗？或者只需要几行精心放置的 jQuery 代码——甚至是原生 JS——就能完成任务吗？你的 CRUD 应用程序*需要*成为通过 Websockets 传送的单页面 JS 应用程序吗？

对于作为一种非常经典的网络项目的 SE 来说，摒弃 Javascript、服务器端渲染，并依赖于经典网络的静态文档和基本表单的交互模式不仅减轻了我们的维护负担，极大简化了开发，而且加速了用户体验，提高了可访问性。

### 云也不是必需的。

如今，甚至在像 AWS 或 Azure 这样的云服务上启动一个基本的网站非常流行。但是，虽然托管我们 VPS 的公司在技术上是一个云服务——诚然，我们渴望自给自足的愿望仅止于拥有租用机架上的裸金属——但我们并不依赖于像 AWS 或 Azure 这样的任何其他第三方云服务。

其中一部分原因是因为只是瞥一眼[令人眼花缭乱的 AWS 首字母缩写和首字母缩略词列表](https://docs.aws.amazon.com/general/latest/gr/glos-chap.html)就让我头晕目眩。

但更严重的原因是，作为一种经典网络项目，SE 渴望独立，就像经典网络曾经一样。我们当前的一个大型外部依赖是 GitHub，但我们（非常）长期的目标是放弃它，转而采用自托管的 Git 接口。

从云提供商开始会放弃自己的独立性，因为它经常会导致供应商锁定。例如，一旦你开始使用 AWS 及其难以理解的一群神秘服务，你就开始需要专业的、专有的知识——我怎么再次设置 S3 存储桶的权限？——把一个已建立的项目从中解脱出来可能会很困难。如果你想要更改你的托管提供商，或者你的存储或备份解决方案呢？

在我们自己的 VPS 上运行一个基本的 Linux 盒的最大好处是一切都只是通用、众所周知的平台上的文件。在很大程度上，一个主机上的 Ubuntu 会和另一个主机上的 Ubuntu 一样，而 Ubuntu 是 Linux，所以跳到（比如）CentOS 并不是一个巨大的飞跃。这些技能是高度可转移的，使得移动一个项目到其他地方，或者升级一个服务器，如果我们的需求超过了它的能力，都变得更容易。

很多人也选择云服务，因为它们的 CDN 可以提高请求速度。虽然这当然是正确的，但 SE 的经典网络方法使得这个因素变得不那么重要。现代网络应用程序在资源大小上变得如此庞大，并且花费了大量时间来处理 JS，以至于使用 CDN 来节省几毫秒的静态资源获取时间对它们来说变得更加有价值。但是由于我们仔细控制了所提供的静态资源的大小和数量，并且只是将完整页面发送回客户端，而不是依赖于 JS 来渲染页面，突然间节省几毫秒的时间似乎变得不那么重要了。而且，如果网站在我们主机远离的地理位置加载较慢，那么，好吧，我们这里提供的是免费电子书——这不是生死攸关的事情。

云服务提供商在许多计算任务中有其用武之地，一旦项目开始达到*非常*大规模，它们就开始变得更有意义了。但是对于提供基本网站及其所需的基础设施（甚至可能需要处理意外的大量流量）而言，VPS 是一种低成本、简单且无锁定的选择。非常经典的网络方式。

### 它只是文件在文件系统中

对于大多数基本网站来说，除了数据库之外最大的资源消耗者通常是用于组装要提供的网页的服务器端语言。当在文件系统上提供不需要任何服务器端处理的纯文件时，Web 服务器的速度非常快。

考虑到这一点，我们已经做到了 SE 服务器提供的大部分文件都是驻留在常规文件系统上的静态文件。我们的 OPDS 和 RSS 订阅是静态文件，只在书籍更新时重新生成；电子书下载是文件夹中的普通文件，其在线阅读版本也是如此；诸如此类。

使用脚本进行数据库查找并为每个请求重新生成一个 feed 或许是很诱人的。但是，当文件系统上的普通文件如此高效时，这就是一项很大的工作。将网站保持为大部分静态文件不仅是一种非常快速、非常经典的网站组织方式，而且还使我们受益于...

### 简单的部署

因为 PHP 没有预编译，而且我们没有需要转换或压缩的 Javascript 或 CSS，所以部署网站就像使用 [`rsync`](https://linux.die.net/man/1/rsync) 一样容易。

一个 `rsync` 调用将我们的开发文件系统推送到生产环境中，创建、更新和删除文件，使两个文件系统保持一致，然后——我们的更改就生效了。

网站只是文件的这种范例越来越不寻常，因为语言和框架积累了越来越奇特的编译过程和包管理器。我的 NPM 版本是否更新？我们是否编写了正确的 XML 模板以编译正确的 DLL？我们是否转换了 TypeScript 并将其缩小为 Yarn 包？Bower 是否将正确的库克隆到 Jekyll 中，然后将 monorepo 进行组装？谁在乎！我们所要做的就是复制我们的文件系统，然后它就能正常工作。

### 构建电子书

当电子书发布或更新时，SE 服务器也负责构建它们。这是通过我们的 [电子书制作命令行工具集](https://github.com/standardebooks/tools) 来完成的。每当电子书存储库更新时，一个 Git 钩子会触发电子书的重新构建，以便更新的版本可以在网站上提供。

构建电子书可能会因为多种原因而变得相当缓慢和资源密集，在我们的低资源 VPS 上甚至更慢。VPS 可以舒适地一次构建一个电子书同时提供网站服务，但是超过三个电子书同时构建可能会导致服务器喘不过气来，手足无措地抓着自己的胸口。

为了防止这种情况发生，使用 [任务池，或 `ts`](https://manpages.ubuntu.com/manpages/xenial/man1/tsp.1.html) 来排队构建，这是一个管理一系列 Bash 脚本队列、按顺序执行它们并记录它们输出的程序。`ts` 是一个小巧、简单、Unix 风格的方式，确保一系列电子书的频繁更新不会一次性超负荷使用我们有限的资源，同时避免像 Beanstalk 这样的大型一体化解决方案的过度使用。

### 旁注：XHTML

与性能无关的 SE 网站的一个特殊之处是它提供 XHTML5。

XHTML 是 HTML5 的被忽视的兄弟，是 2000 年代初期神秘语义网络的先驱。它带来了 XML 的所有烦人的、令人叹息的包袱，如命名空间和严格的解析器错误，但只带来了很小的好处，大部分都给了渲染引擎，其他方面并不多。

它在网络上从未流行起来，部分原因是因为在网站上下文中，大量 HTML 是由模板和库生成的，很容易在其中的某个地方引入语法错误；而与 HTML 不同的是，XHTML 中最微小的语法错误意味着整个页面被浏览器抛弃，你得到的是“黄色死亡屏幕”。

即使 XHTML 在网络上并不流行，但它仍然在一个非常大的领域里存在：电子书。

你知道，[EPUB](https://en.wikipedia.org/wiki/EPUB)，这个令人惊讶的优秀（你多少次会把一个标准描述为“优秀”？）开放电子书格式被绝大多数电子书制作者和阅读系统使用，基本上就是一组打包在 zip 文件中的 XML/XHTML 文件。你可以解压缩一个电子书并在浏览器中打开其 XHTML 文件，它看起来就像一个普通的网页。

由于我们的主要产品是包含 XHTML 的 zip 文件，我们想为什么不将网站本身也作为 XHTML 提供，作为对我们电子书起源的一种怪念头？所以我们这样做了。

我真的不建议你在开始一个新项目时使用 XHTML。它是不易原谅的，好处很少甚至没有。但对于我们来说，这是一种有趣的实验，一个项目经理在现实生活中不会让你做的事情——向我们所钟爱的 EPUB 格式致敬，并向语义网络的美好梦想举杯致敬。

## 这一切为什么重要呢？

所有这些都表明，你不需要很多东西来构建一个性能优越的、有用的网站，能够在一个小服务器上处理数百万次请求，并且还能处理其他资源密集型任务。

认为需要臃肿的 JS 框架、复杂的后端框架、一大堆包管理器、像数据库或专门的排队软件这样复杂的移动部件，以及专有供应商提供的高资源、昂贵的云实例来创建一个“现代”网站是一个谬论。经典的 Web 技术可以让你以每月几分钱的价格提供数百万次的页面浏览量。*而且，你还能获得速度、可访问性、可维护性和简洁性，这些都是免费的*。

下次你想要添加一个 JS 库来对一个元素进行动画，问问自己：我能用纯 CSS 实现吗？

下次你用 React 开始一个新的网站时，问问自己：如果我只是在服务器端渲染呢？

下次你在专有云提供商上启动一个计算实例时，问问自己：一个 $5 的 VPS 够用吗？

下次你想要提供 XHTML5 时，问问自己：等一下，*什么*？
