<!--yml

类别：未分类

日期：2024-05-27 13:00:27

-->

# 主页 · thmsmlr

> 来源：[https://thmsmlr.com/cheap-infra](https://thmsmlr.com/cheap-infra)

生活中有一件事是确定无疑的：如果你以超低价格得到了什么东西，肯定是其他地方有人在被宰割，而且很可能是你。这不是反资本主义的怒吼，这只是事实。如果公司不赚钱，它们就不存在。除非有某种创新，否则可以肯定地说你在其他地方支付了过高的价格。

上周，在亚马逊看到一些比在Costco列出的更便宜的东西时，我想起了这件事。Costco在行业中拥有最低的成本结构。这是没有意义的。但是，这篇文章不是关于3美元的驱蚊剂和亏本销售。这是关于AWS和云的一篇文章。

众所周知，亚马逊大部分利润来自AWS，然而还有很多人谈论云是如此便宜。这是不可能的。你正在被宰割。

**这里有一些数学。**

### 从第一原理出发

假设你相对雄心勃勃。你想在互联网上创建一个前1000名的网站，比如[businessinsider.com](https://www.businessinsider.com/)。根据[SimilarWeb](https://www.similarweb.com/website/businessinsider.com)的数据，他们是全球前1000名网站（确切地说是第587名），每月大约有2亿访客。每位访客平均访问2页，因此每月需要提供4亿个HTML文档。在Business Insider的几篇报道中抽样，似乎一个标准的故事HTML文档压缩后大约为75KB。将这些乘在一起，你需要每月30TB的带宽来提供HTML内容。

那听起来很多，但请考虑，我们并没有假设HTML使用CDN，因为您的网站可能不是新闻网站。您的HTML可能是根据每个请求动态提供的。仍需考虑的是，Business Insider的文章中有大量内联JavaScript，这些可能可以在您优化的实现中发送到CDN。75KB的压缩HTML实在是太多了。尽管如此，每月30TB只相当于11MB/秒，而在Business Insider的情况下，这只是大约150个请求/秒。

减少HTML大小，增加请求数量，无论如何，通过CDN为您的JS、CSS和图像提供服务，您的应用代码如果能产生11MB/秒的HTML，就可以拥有一个前1000名的网站。这是一个非常低的门槛。即使是最慢的解释性语言在现代硬件上也能做到10倍以上。

最新的AMD服务器处理器拥有64个核心和128个线程。据传言，他们最新的Zen 5、Turin服务器处理器将拥有192个核心。在双插槽服务器上，您将接近400个核心，768个线程。**您可以通过单个盒子为世界提供服务。**为什么我们还需要Docker、无服务器、水平扩展性呢？

也许你会想，这是一些巧妙的数学技巧，但延迟呢？你不能否认光速，托马斯，宇宙的基本物理原理！这是这些云供应商市场部门目前正在谈论的内容吧？他们说你需要处于边缘。靠近你的用户。将延迟最小化。这是真的，我们在这个家庭中信仰物理学，但具体情况更有趣。

### 边缘感觉不错，但不能实现预期效果。

让我们来了解一下延迟的下限。光速下，在全球范围内往返需花费200毫秒。然而，实际情况是，在地球另一边的良好数据中心通常需要约300毫秒。

现在再次假设您的JS、CSS和媒体是从CDN传送的，那么**真正的问题是，如果您在初始渲染中能够节省300毫秒的服务器处理时间，您实际上已经将您的服务器移到了靠近用户的全球位置。**

有了这样的背景，边缘技术开始显得不那么优越了。虽然第二代无服务器技术在解决冷启动问题方面取得了巨大进展，这个问题曾经轻松耗尽了300毫秒的延迟预算，但数据库问题仍然存在。

无论您如何设计您的站点，SPA、SSR、某些混合形式，如果在呈现页面时涉及至少一个数据库查询，您都必须回到您在us-east-1的数据库。从而将延迟从用户转移到起始服务器，再到边缘服务器和起始服务器。

有人可能会争论数据中心之间的连接比用户与数据中心之间的连接更快，这可能是对也可能不是，但这并不意味着你不能将你的网站放在Cloudfront背后，享受所有这些好处，而不必“运行在边缘”。

更糟糕的是，大多数复杂页面需要进行5次以上的数据库查询才能呈现。大多数Web框架是单线程的，每个查询在us-east-1之间串行运行，使延迟比只去那里一次并使所有数据库查询都在数据中心本地运行更糟糕。

你是否注意到 Vercel 的所有边缘演示都只显示时间？那是因为那是边缘上唯一有用的信息。

*没有人会考虑数据库。*

一个很好的经验法则是，数据中心间通信比数据中心内通信慢10倍，而后者比设备内通信慢10倍。鉴于那些192核的AMD服务器，SQLite看起来相当不错。弥补了那300毫秒的大部分时间。

### 我们甚至还没有谈论成本。

[Hetzner.com](https://www.hetzner.com/cloud)。看看这个网站，告诉我你没有被敲诈。

对于一个带有16核心、64GB RAM 和 NVM 驱动器的服务器，Hetzner 每小时收费 $0.34。在 AWS EC2 上相当的 x86 服务器是 m5a.4xlarge，每小时收费 $0.68。不仅如此，对于带宽，Hetzner 给你免费提供20TB 的数据，之后每TB 收费 $1.5。AWS 只给你免费提供100GB，之后每TB 收费高达 $90。

如果你正在使用边缘云供应商之一，价格会更加离谱。例如，Vercel 在超过免费的第一个TB 的带宽后，每TB 收费 $200。这些云供应商给你很多东西是免费的，目的是为了锁住你，但是在你扩展时却会让你感到困扰，但正如我们之前所建立的，你可能并不需要这些。

### 这就是实际情况。

除非你有云计算的某些特定用例。也许你在转码视频，运行自己的 AI 模型，或者在做一些确实会对系统造成压力的事情，你的网站或 SaaS 可能只需一个服务器就能运行。这样更便宜，也更简单维护。在弗吉尼亚州放一个服务器，你就可以在100毫秒内覆盖英语世界。

在本地使用 SQLite，你不需要一个托管的数据库。使用 [Litestream](https://github.com/benbjohnson/litestream) 进行持续备份。使用 CDN 缓存你的 CSS、JS 和图片。在靠近你的 SQLite 的服务器上渲染页面，以减少往返次数，提高性能。

你的 CI 可以直接将代码 SCP 到服务器上，NGINX 支持零停机部署。不要费心使用 Docker 和虚拟化之类的东西，它只会减慢你的代码速度，影响你的 CI/CD。

他们说你需要它来扩展。大多数情况下，你其实并不需要。关于水平扩展的传闻，是我们行业中最违背摩尔定律的事情。

现实情况是服务器的能力增长速度比互联网增长速度更快。如果你真的关心延迟问题，把一个服务器放在德国，一个放在加利福尼亚，把写操作路由到主服务器，使用本地的读取副本进行读取。

它将非常容易扩展，管理起来也更简单，并且成本大大降低。11MB/sec 不必这么难。
