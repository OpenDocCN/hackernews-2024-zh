<!--yml

类别：未分类

日期：2024-05-27 14:29:10

-->

# A CAP tradeoff in the wild - decomposition ∘ al

> 来源：[`decomposition.al/blog/2023/12/31/a-cap-tradeoff-in-the-wild/`](https://decomposition.al/blog/2023/12/31/a-cap-tradeoff-in-the-wild/)

在复制数据系统的背景下，安全性和活性之间存在经典的折衷，最初由 Eric Brewer 提出，后来由 Seth Gilbert 和 Nancy Lynch 进行了精确定义。其思想是，在一个易受分区影响的服务器网络系统中（即，服务器之间的消息可能会被任意延迟，甚至永远延迟），无法同时确保服务器响应请求时使用的是“正确”的响应（*一致性*，一种安全属性），以及每个请求最终都会收到响应（*可用性*，一种活性属性）。

假设我们有一个由两个服务器组成的系统，称之为 Alice 和 Bob，它们由于网络分区而无法相互通信。就在这时，一个客户端请求 Alice 获取某些信息。与此同时，Bob 可能拥有比 Alice 更为实时的信息副本，但 Alice 无法与 Bob 通信以获取信息。因此，Alice 面临一个困境：她可以等待 - 也许永远等待 - 等待分区恢复，以便查看 Bob 是否拥有比自己更为实时的信息，从而牺牲可用性。或者 Alice 可以告诉客户端她所知道的，这可能是过时的，从而牺牲一致性。在网络分区的情况下，无法避免牺牲其中之一。这就是（臭名昭著的）CAP 定理，[Gilbert 和 Lynch 于 2002 年正式化](https://www.comp.nus.edu.sg/~gilbert/pubs/BrewersConjecture-SigAct.pdf)的。 （我认为他们的[2012 年的跟进文章](https://decomposition.al/CSE232-2023-09/readings/cap-perspectives.pdf)更具启发性。）

这个[2018 年提出并仍然未解决的 Kubernetes bug](https://github.com/kubernetes/kubernetes/issues/59848)在不久前与我的一位博士生讨论中提出，我认为这是一个在现实中非常了不起的 CAP 折衷的例子。

[这篇 HotOS 2021 论文的图 2](https://sigops.org/s/conferences/hotos/2021/papers/hotos21-s11-sun.pdf)由孙旭东等人绘制，说明了这个 bug。图的左列标有“history”，表示 etcd（一种强一致性数据存储）关于集群状态的“真相”。然而，各种 Kubernetes 组件（称为“apiservers”和“kubelets”）并不直接与 etcd 交流，而是通过各种缓存间接看到该状态。图中的中间两列代表 apiservers API-1 和 API-2，它们各自维护这样的缓存。

在示例运行的图中显示，API-2“与 etcd 的网络连接出现问题”，因此其缓存落后。同时，Pod P1 在 kubelet K1 上创建，然后移动到 K2 上，这需要在 K1 上删除它，然后在 K2 上创建它。API-1 知道 P1 已从 K1 移动到 K2，但 API-2 不知道。

然后 K1 崩溃了，重新启动，从 API-2 上读取集群状态（它说 P1 应该仍然存在于 K1 上！），结果在 K1 上重新创建了 P1。现在我们违反了一个安全性属性，即同名的两个 Pod 不应该同时存在。

正如[错误报告解释的](https://github.com/kubernetes/kubernetes/issues/59848)，这应该是一个“HA”（高可用性）设置，这就是为什么 API-1 和 API-2 都存在的原因。但它们（以及至关重要的缓存）*确实*都存在，这导致了 CAP 的权衡，因为[缓存就是复制](https://twitter.com/lindsey/status/1446931576185524225)，当不可避免的网络分区发生时，这种权衡毫不奇怪地出现了。

API-1 和 API-2 在上述例子中类比于我上面的 Bob 和 Alice。这并不是一个完美的比喻，因为在我的例子场景中，分区阻止了 Alice 和 Bob 之间的*直接*通信，而在 Kubernetes 的错误场景中，分区阻止了通过 etcd 的共享状态的*间接*通信。但无论如何，重点是 API-2 和 API-1 无法通信，因为 API-2 无法与 etcd 通信，这意味着它无法获取 API-1 的信息。API-2 因此必须在安全性和活跃性之间选择。它选择了活跃性——一个可以理解的选择！——因此，安全不变量被违反了。

在 GitHub 上的数十条评论中有些人询问是否可以直接从 etcd 中读取，而不是从两个 API 服务器的缓存中读取。这的确会绕过这种权衡——如果只有一个副本，那么你就不可能有不一致的副本！但是，该主题中的其他人指出，这样做会创建一个无法接受的可扩展性瓶颈。来回讨论不断进行。

我不能责怪 Kubernetes 的开发者们还没有能够就此问题达成解决方案，因为没有任何解决方案不会牺牲某种可取之处。该主题中 Josh Berkus 的一条评论在中间很好地总结了这一点[“欢迎来到 CAP 定理的世界，我想？”](https://github.com/kubernetes/kubernetes/issues/59848#issuecomment-960282426)

最后，可能有些引起争议的一点是，我觉得上述[HotOS 论文](https://sigops.org/s/conferences/hotos/2021/papers/hotos21-s11-sun.pdf)中的图 1 非常说明问题：我们坐在 etcd 之上，这是一个强一致性的数据存储，但在它和我们之间有着层层的缓存。如果你只会从可能过时的某个缓存级别读取，那么你真的需要 etcd 吗？
