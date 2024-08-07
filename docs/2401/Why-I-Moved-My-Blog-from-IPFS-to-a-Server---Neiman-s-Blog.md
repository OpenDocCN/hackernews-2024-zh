<!--yml

类别：未分类

日期：2024 年 5 月 27 日 15:22:53

-->

# 为什么我将我的博客从 IPFS 迁移到服务器 | Neiman 的博客

> 来源：[`neimanslab.org/2024-01-31/why-i-moved-my-blog-ipfs-to-server.html`](https://neimanslab.org/2024-01-31/why-i-moved-my-blog-ipfs-to-server.html)

作者：Neiman

于 2024 年 1 月 31 日

# 为什么我将我的博客从 IPFS 迁移到服务器

可以肯定地说我是 IPFS + ENS 网站的先驱。当我在 2019 年 3 月建立了我的第一个 ENS+IPFS 网站时，其他网站不超过 15 个。从 2019 年到 2022 年，我共同建立了一个 IPFS+ENS 浏览器扩展（Almonit），一个 IPFS+ENS 搜索引擎（Esteroids），当然，我的个人博客只在 IPFS+ENS 中可用。

但是今天我将我的博客迁移到了服务器上，我想讨论一下原因。

令我兴奋的是像 IPFS 这样的点对点网站，在理论上，网站的访问量越多，它就越稳健、抗审查和可扩展。再次强调，这只是理论上的。

你知道种子文件似乎能够永生吗？我希望网站也能如此。我想象了一个网站，难以遭受 DDoS 攻击（稳健），难以被封锁（抗审查），而且读者越多，访问速度越快，因为更多的读者帮助传播内容（可扩展）。

我想象了一个有一个大大的“固定我”按钮的网站（在 IPFS 中固定就像在 BitTorrent 中种子）。如果读者按下该按钮，他们将帮助服务该网站。

实际上，这并没有真正奏效，原因有几个。

1.  IPFS 用户大多不运行自己的节点或软件。相反，他们使用网关。我是根据社区中所见到的情况以及运行自己的 IPFS 节点相当不方便这一事实所作出的合理猜测。但即使你运行自己的节点，你访问一个网站并不意味着你会固定它。一点也不。

    这与 BitTorrent 有很大的不同，BitTorrent 的获取内容的唯一方法是运行自己的软件，并且默认情况下，当你下载某样东西时，你也会分享它。

    因此，大多数读者不会帮助分享一个网站，但即使对于愿意帮助的人，仍然存在额外的复杂性：

1.  网站是动态对象。它们的内容一直在更新。如果你只是固定网站当前版本的内容，那就没什么帮助。

    大多数 IPFS 网站所做的是使用一个指向其内容最新版本的命名系统。通常是 IPNS，IPFS 的内部命名系统，或者是 ENS，以太坊命名系统。但是 IPFS 还没有一个简单的命令始终固定 IPNS 的最新内容，如果有人使用 ENS，这意味着固定它的人还需要监听以太坊区块链事件，这本身就是一个巨大的额外挑战，无法使用集中式服务完成。

1.  更糟糕的是，让 IPFS 内容在浏览器中可靠地可用实际上是相当困难的！

    例如，我希望我的 IPFS 博客能在所有主要网关、所有 IPFS 节点、支持 IPFS 原生的 Brave 浏览器以及 IPFS 的 js-libp2p 和 helia（IPFS 的 js 库）上都可以访问。我没找到一个可靠的方法来实现这一点。

    **长篇抱怨：**我从自己的服务器中固定了内容，并且一直在调整设置和定义，但是却无法使内容随处可用。更糟糕的是 Helia，我根本无法让我的内容在浏览器中通过 Helia 直接访问，而不是直接连接到我的节点。那么 p2p 网络是什么意思呢？

    我发现有一个服务，[cid.contact](https://cid.contact)，称为“内容路由”服务。在 cid.contact 的“关于”部分中写着它与 Filecoin 有关，但出于某种原因，它保存了 IPFS 的路由数据，据我所知，这是一个不同的网络。在我至少使用的版本中，cid.contact 的地址是硬编码到 Helia 的代码中的，并且很明显，如果一个内容被 cid.contact 索引，那么它几乎可以在任何地方访问，但如果没有索引 - 它不总是可以访问的。

    我搞不清楚如何将我的内容索引在 cid.contact 中。老实说，我不确定我是否想要这样做。因为有什么意义呢？似乎这只是增加了对一个中心化服务的依赖。我可以尝试运行自己的索引器，并在我的网站中定义它，但同样是中心化。这些索引器的经济模型是什么？我们期望长期由哪些角色来运行它们？

    cid.contact 中的文本说，在当前 IPFS 的规模下，期望 DHT 单独有效地处理路由是不合理的。这在某种程度上是有道理的，但除此之外，还有什么选择，不会破坏技术的优点呢？

到目前为止，我已经厌倦了为了使我的 IPFS 博客正常运行而不断奋斗。至少在短时间内，我想要一个简单、经典的工作解决方案。你现在正在阅读的博客是用 Jekyll 构建的，并托管在我自己的 10 美元服务器上。

不要误会，我仍然是一个 IPFS 的粉丝。这是一个非常好的项目，管理得很好。只是它还不适合个人博客的需求。

话虽如此，要在不把这变成一份全职工作的情况下跟进 IPFS 或 Filecoin 的持续发展和创新确实很困难。我错过了一些琐碎的解决方案或最近的创新吗？如果是的话，告诉我。这里还没有评论，但你可以通过老式的电子邮件（neiman@hackerspace.pl）或者 Mastodon（@neiman@mastodon.social）找到我。
