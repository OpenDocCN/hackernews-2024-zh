<!--yml

类别: 未分类

日期：2024-05-27 14:43:36

-->

# Underjord | 解读 Elixir: Phoenix

> 来源：[https://underjord.io/unpacking-elixir-phoenix.html](https://underjord.io/unpacking-elixir-phoenix.html)

# 解读 Elixir: Phoenix

2024-01-22

Underjord 是一群从事 Elixir 咨询和合同工作的[小而美团队](/team.html)。如果你喜欢这篇文章，真的应该尝试一下代码。查看我们的[服务](/services.html)以获取更多信息。

在本系列中，我一直在解开 [Elixir](https://elixir-lang.org/) 的各个方面。大多数情况下，这意味着试图通过 Elixir 的视角来解释 Erlang 和 BEAM。现在我们正在进入网络框架的领域。在这里，我敢说 Elixir 比 Erlang 有更多发言权。据我所知，Erlang 从来没有完全确定首选的网络框架。Elixir 有 [Phoenix](https://www.phoenixframework.org/)，这篇文章将解读 Phoenix。Elixir 网络框架。

至于 Erlang，[这个精彩的 Erlang 列表](https://github.com/uhub/awesome-erlang)上有很多网络框架。虽然有很多，但我从未发现有关“应该”使用哪个的共识。实际上，当我在会议上喝酒时与 [Robert Virding](https://www.wikidata.org/wiki/Q107596747) 对话时，我问了一些关于这个的问题，他或多或少表示 Elixir 和 Phoenix 应该成为 BEAM 的首选网络框架。确切的问题和答案由于时间和记忆而模糊。我的理解是，他更喜欢 Erlang 用于其他方面，但真的希望人们只使用 LfE ;)

这不应被视为对 Erlang 的批评，而是对 Elixir 建立和保持这种有用的内聚性的赞赏。Elixir 生态系统一直以一种 Erlang 未曾有过的关注度关注网络。

人们在 Elixir 中构建了其他网络框架。Phoenix 仍然是主要参与者。它是 Elixir 生态系统的 Rails/Django/Laravel，尽管我认为它更加轻量级。在很多方面都是好的，可能也有坏的。

## Plug

我们不应该从 Phoenix 开始。那是高级框架。处理网络请求和创建响应的基础知识都交由 [Plug](https://hexdocs.pm/plug/readme.html) 处理。Plug 处理头部、主体、查询参数、URL、路径等，通过提供可组合插件的概念。一个非常简单的插件:

elixir

```
defmodule MyApp.NotFoundPlug do
	alias Plug.Conn

	def init(opts) do
		opts
	end

	def call(conn, opts) do
		conn
		# Send a 404 to the client
		|> Conn.send_response(404, "Not found")
		# Do no further processing of this plug pipeline
		|> Conn.halt()
	end
end
```

Plug 还提供了一个用于路由和基于路径和方法构建路由的模块。这个例子来自文档。

elixir

```
defmodule MyRouter do
  use Plug.Router

  plug :match
  plug :dispatch

  get "/hello" do
    send_resp(conn, 200, "world")
  end

  forward "/users", to: UsersRouter

  match _ do
    send_resp(conn, 404, "oops")
  end
end
```

一个不会暂停的插件会继续到下一个指定的插件。这意味着身份验证、授权、解析等的每个步骤可以进行分层。类似于中间件的方法。它们都与`Plug.Conn`结构体一起工作。

Plug 是一个相当优雅的处理 Web 事务的方式。 良好的构建模块。 当我想做一个[我的图像中的实时访问者计数器](https://underjord.io/live-server-push-without-js.html)（还有[Plug](https://github.com/lawik/mjpeg/blob/master/lib/mjpeg.ex) 和[项目](https://github.com/lawik/mjpeg_example)）进行分块和生成 MJPEG 时，我从 Nerves 项目的 Frank Hunleth 和 Connor Rigby那里偷了一些代码。 它不需要 Phoenix 的其余部分。 直接使用 Plug 的一个好例子。 但同时，Phoenix 控制器只是一个 Plug，也可以很好地实现这一功能。

## Web 服务器

从历史上看，Phoenix 倾向于使用 Erlang 编写的名为[Cowboy](https://ninenines.eu/docs/en/cowboy/2.10/guide/)的 Web 服务器。 长期以来，它一直是一个非常可靠的工具，并在那个角色中表现良好。 通过 `plug_cowboy` 库与 Plug 连接。 我越来越多地看到项目选择使用[Elixir 写的Bandit](https://hexdocs.pm/bandit/Bandit.html) ，这意味着社区对贡献的门槛较低，因为在 Elixir 领域的人比 Erlang 领域的人更多。 这也有更多内容。 如果你感兴趣，我们在[BEAM Radio 的一个节目中](https://www.beamrad.io/53)也谈到了一些。 据说 Bandit 的速度比 Cowboy 略快，这当然是一个不错的好处。

这些 Web 服务器的共同点是它们不是你的 Ruby 或 Python 应用程序 Web 服务器。 不需要反向代理，除非你想要一个。 它们实际上可以被信任来进行真正的前线工作。 Erlang 就是为此而构建的。

## Phoenix

现在我认为我们可以着手解决 Phoenix。 啊是的，Elixir 的“Rails”。 但并不那么相似。 现在我对 Rails 没有任何重要经验，但我和有经验的人交谈过。 我在 Django 上有经验。 Django 和 Rails 都在努力帮助开发人员提高在常见情况下的生产力。 帕累托法则，80/20法则，都在其中。 还有更多。

在 Django 中，这是通过“魔法”实现的。 主要是通过类的继承，对你放进其中的内容进行内省并基于此执行操作。 他们也非常直接地采用元编程，你最终会把一堆东西放进一个叫 Meta 的类中。

这些是非常动态的语言，其中 monkey-patching 和其他有趣的特性都是极其容易获得的。 当然，这意味着你应该尽量限制使用这些有趣的特性。 在这些语言和框架之上，没有纪律的代码库可能会变得非常混乱。

Phoenix 尽量不依赖于“魔法”。 我们称之为[宏](https://hexdocs.pm/elixir/macros.html)。

开玩笑的。 宏是 Phoenix 和 Plug 中较令人困惑的部分之一，但它们通常是为了管理一些固有复杂性而存在的，它们很多时候仍然在“你”的代码中。

通常你会使用`mix phx.new my_app`开始一个Phoenix项目，它会生成一个你拥有的项目。当然，你会有依赖项，其中的代码你并不拥有，但你的MyAppWeb模块有宏，可以引入必要的函数给控制器或LiveView，并且你可以根据自己的方式进行调整。

我听过很多人在生成Phoenix项目时会说“有太多文件”，我同意这个印象。但一旦你了解了它们，大多数文件都有相当明确的目的，并且它们在那里的目的是为了让事情变得明确，并交给你掌控，而不是神秘地和魔法般地继承东西给你。还有一些卫生问题，比如gettext，在你的前几个项目中可能用不上，但它们存在是因为应该包括它们。当你需要它们时，它们不在时你会很气恼。

### 有见解的设计，或者说没有见解的设计

我们经常在Web框架中谈论有见解的设计。通常的原因是，有见解的设计会针对某些情况做出重大的取舍，以支持常见情况。帕累托原则，80/20法则，总之，再次强调。通过提供有见解的设计，你可以消除许多决策的必要性，并在理想情况下提供经过良好验证的足够好的解决方案，或者至少提供有帮助的简化方案。

Phoenix并非非常有见解。我认为Phoenix开发者带着更多的经验，并想要专注于好的基元。此外，函数式编程与Python和Ruby的面向对象编程风格有着不同的抽象方式。函数式编程倾向于用简单的部分实现复杂的事情。

Erlang在基本层面上有着极为独立的见解。它在构建服务时做出了许多选择，而Elixir继承了这些观点。我们放弃了许多我们不关心的东西，换取了一个极好的Web框架基础。我们获得了简单的并发和并行，但放弃了小型二进制和数值计算。我们有一致的延迟，但没有获得可变状态的速度。

我一开始把Phoenix视为类似Django的有见解的框架。除了它们都是Web框架之外，我不知道是什么让我有了这个想法。当然，它带来了一些观点，比如“在一个中心位置指定路由很好”，“这是你应该如何引入用于控制器的帮助函数”。它还抽象了连接池和监控树，并在那里固定了一些观点，但通常人们谈论有见解的框架设计时指的并不是这个。

如果你来自Django或Rails，你可能会错过从模型中派生所有内容的极其方便的基于模式驱动开发。如果你想要更多这样的东西，目前最有趣的选择是[Ash Framework](https://ash-hq.org/)，它非常有自己的见解并且非常迷人。

在Phoenix的核心web功能之外还有一些有见解的部分。Channels是对WebSockets的一种意见，同样也适用于Presence。Phoenix LiveView有很强烈的观点，并且为了提高生产力进行了相当激进的折衷。

Phoenix默认还引入了[Ecto](https://hexdocs.pm/ecto/Ecto.html)，而Ecto是一种相当有见解的关系数据库方法。[Changesets](https://hexdocs.pm/ecto/Ecto.Changeset.html)和[Query DSL](https://hexdocs.pm/ecto/Ecto.Query.html)都相当灵活，但它们会推动你朝着库认为最佳的工作方式前进。

Phoenix最有争议的部分似乎是[Contexts](https://hexdocs.pm/phoenix/contexts.html)。它是Controller、Channel或LiveView与基于Ecto的数据层之间的连接组件。默认情况下生成一个方法，但它被广泛讨论，并通常需要您应用自己的设计和想法。它很开放。没有特殊的Context代码。有一个来自Phoenix生成器的意见，但似乎不是太明确，它并不是框架的核心。

总的来说，我认为Phoenix的层次结构良好，我发现在基本框架中对意见的约束意味着没有对Phoenix的轻量级微框架替代的强烈需求（请参见Python中的Flask、FastAPI），因为如果您只要求它不添加数据库或跳过HTML部分或任何您需要的配置，您将获得一个轻量级的API服务或完全简化的HTML输出机器。

## Phoenix的功能集

我将尝试介绍构成Phoenix的东西。也就是说，主要功能。

### 项目生成

Phoenix做了一些所有权的倒置，正如我所提到的，生成了一堆文件，然后你可以拥有和演进它们。我像在试验某种新的黑客工具一样，每周都会运行`mix phx.new`。生成器有很多选项，选择数据库（postgres、mysql、sqlite）或跳过Ecto和数据库细节。HTML或者不HTML。LiveView或不LiveView。Assets或不Assets。这是正常的起点。

还有其他的初始模板。传说中的、Petal Pro和一些其他的。我不能为它们做保证。我曾在一个Petal Pro项目上工作过，效果不错，它确实在模板和布局周围带来了更多的观点。

### Ecto

数据库层。不会被锁定到Phoenix的任何特定方式，但它默认情况下就包含了。

Ecto为您的应用程序提供了与关系数据库相关的功能，并阻止您犯下一堆可能导致SQL注入攻击等错误。这在不同的方式上影响了Ecto的设计。它相当依赖于在编译时使用宏来构建查询。在你可能需要的地方也有逃生舱口。

一个repo是对管理数据库连接池的监督树的抽象。一个典型的应用程序处理一个名为`MyApp.Repo`的Repo，并提供所有查询函数等。

如果您正在处理两个单独的数据库，您可以轻松设置一个额外的 repo。如果您正在处理多租户或其他多数据库情况，您还有“动态 repo”功能，这将让您以这种方式工作。

Schema 模块为指定数据库表格提供了 DSL，主要用途是与 Django 或 Rails 模型相关联，尽管在性质上不那么神奇。它们归结为 Elixir 结构。

定义模式（和其他数据）的验证规则的方法。这将帮助您产生良好的错误，优雅地集成从数据库返回的实际错误以及其他许多内容。变更集用于插入和更新周围。它们是一个很大的话题，值得研究，因为它们是相当有趣和有用的设计。

### Web 相关

该模块是为您生成的，但当添加到您的监督树中时，它会启动一个包含用于启动任何 Phoenix 拥有的进程的监督树的 Phoenix 端点。无论您是使用 Cowboy 还是 Bandit 的 Plug 集成，它都将设置您的 Web 服务器以侦听传入请求并将其路由到 Plug 和端点。端点定义了一些基本的插件和配置。然后通常交给路由器进一步处理请求。

路由器模块是您使用 Phoenix 和 Plug 插件的组合来处理请求并将其委托给控制器和 LiveView 的地方。这是一个很好的结构化中心，还允许您为需要应用的其他插件定义管道，比如检查身份验证和授权。

#### MyAppWeb

这个文件包含您的控制器、频道和 LiveView 的宏。这些宏主要引入其他 Phoenix 功能，但它们在您的文件中，您可以和应该将它们用作引入您自己工具的扩展点。

控制器处理请求。控制器可能生成 JSON API 响应、HTML、文件块或其他任何内容。都没关系。如果要呈现 HTML，则会涉及模板和组件。实际上，控制器也只是一个 Plug。

#### 模板化（[Heex](https://hexdocs.pm/phoenix/components.html)）

Heex 是与 Elixir 配送的常规 Eex 模板不同的演进。Heex 是 HTML 感知的，会在您弄错闭合标签时提醒您。除了基本的字符串插值等，它还有很好的参数语法。

elixir

```
def render(assigns) do
	~H"""  <div :for={thing <- @items} :if={thing.great?} class={@myclass}>  <.custom_component thingie={thing} />  </div>  """
```

### 创新

我想详细介绍一些我认为提供了超出我们通常在 Web 框架中看到的功能或比大多数 Web 框架做得更好的部分。

功能作为 Phoenix Channels 的基础。PubSub 是一种用于应用程序中松散耦合通信的发布/订阅机制。它使用 Erlang 的进程组和 Erlang 的分布。除非您使用 Heroku，否则您需要 Redis 适配器。您应该进行集群化，否则 Chris McCord 会对您感到不满。

Phoenix PubSub 对于让进程跟踪主题并在发生事件时接收消息非常实用。作为 GenServers，Channels 和 LiveViews 是可以处理消息的进程，因此您可以为许多便利性使用它们。一个常见的用例是通知 LiveView 显示的内容实际上已经更改。然后它可以根据需要执行任何适当的操作来通知用户。

设计用于支持 Channels 中的使用，但也可用作更通用的工具。Phoenix Presence 允许您跟踪事物的短暂存在（通常是用户）。他们在线吗？忙碌？远离并带有一条小消息？他们只在移动设备上吗？它使用 CRDT 方法来最小化需要保留的数据量，同时在不需要单独存储后端的情况下创建世界的最终一致模型。

一种抽象，旨在在 WebSocket 之上运行，尽管如果必要，它将进行长轮询，它提供了连接到通道并在其中加入房间的抽象。每个 WebSocket 在服务器端由一个 GenServer 支持，并允许您保持与连接用户相关的状态。通常，您使用处理失败、重新连接并提供一些 API 便利性的 JavaScript 客户端连接到它。

从根本上说，最重要的部分是 WebSockets 和服务器端的 actor。但其他部分也很好。

大多数人心中的宠儿。LiveView 的重点在于消除大部分 Web 开发任务中编写 JavaScript 的需求。它允许基于服务器状态驱动高度交互式的 Web 用户界面。通常通过类似 Channels 的 WebSocket 实现。您可以在 Heex 模板和组件中使用诸如 `phx-click` 等属性来注解，以便向服务器发送事件。事件被处理以更新服务器状态，并计算出最小差异，然后传回浏览器，浏览器可以使用 morphdom 来更新 DOM。

使用 SPA 和自己的 API 实现很难获得如此精简的数据负载。它还深度依赖于 Erlang 特性：一贯低延迟。

还支持驱动某些简单的 JS/CSS 更新，无需服务器往返，支持动画，流式列表，无状态组件，有状态组件等等，还有更多细节我可能忘了提到。

也可以轻松地将服务器端更改推送到相关的 LiveViews，这得益于 BEAM 和 Phoenix PubSub。

在这里，我们有一个非常主观的设计。它牺牲了离线支持，以减少编写代码或维护接口的需求。这是一个极大的时间节省。关于 LiveView 的更多文章可以在其他地方找到。我们可以就此结束。

### 资产管道

曾经 Phoenix 随附 brunch，我想。然后他们转向了行业标准：Webpack。后来他们厌倦了帮助人们保持他们的 node 和 npm 设置正常运行所带来的巨大支持负担。Phoenix 切换到[Esbuild](https://esbuild.github.io/)，通过 Hex.pm 传递的一个 Elixir 库。

我认为这是一个非常明智的举措。现在我的资产问题少多了。

Phoenix 随附有 Swoosh，这是一个用于电子邮件的抽象框架。它有许多流行的交易性电子邮件提供商的插件。因此，一旦进入生产，你可以轻松地进行密码重置或魔术链接。

它还有一个非常好的小型开发工具，让你检查一个已经“发送”的邮箱。

不太被宣传的酷东西。它是一个可观察性仪表板 web UI，在您的管理界面中默认可用，并可以做一些简单的事情，比如：

+   调查正在运行的进程

+   查看内存使用情况

+   查看操作系统指标

+   查看ETS表的使用情况

+   捕获请求日志

以及更多。你几乎没有任何努力就得到了一个用于调查异常系统的实用的首次查看工具。这正是 Elixir 风范。所有的原始和可能性都来自 BEAM，并且在 Erlang 中一直存在。但 Elixir 使它变得美好，简单，对于我们大多数人来说，因为Phoenix，它默认就在那里。

## 你的应用不仅仅是“网站”

大多数系统除了提供网站流量外，还有更多的职责。通常这些职责被委托给队列/代理、工作人员、其他服务、Redis、数据库、云函数或其他任何东西。Phoenix 应用首先是一个 BEAM 应用。您可以在其中运行许多其他工作负载。你正在构建一个系统，它不必在此运行时下面有几个应用程序。

Phoenix 的一个重要特点是它不以任何特殊怪异的方式感染你的系统。它不会使应用程序偏离正常的 Elixir 应用程序惯例。

## 消除基础设施

我在整个过程中都提到了这一点。但从根本上讲，一个 Phoenix 应用只真正需要一个支持的关系型数据库。通常是Postgres。

没有Redis。没有邮件工作者。没有独立的任务运行者。这一切都可以从你的应用程序中完成。

## 总结

Phoenix 在 Elixir 中绝对是经典的 Web 框架，并且在社区和生态系统中具有巨大的吸引力。我认为我们从中受益匪浅。围绕 Phoenix 和 Ecto 有许多共同的、有方向性的努力。

这是一个非常稳健的 Web 框架，构建在一个传奇般的运行时之上。而你可以用一个非常易于接近的语言在它内部构建。

我们得到了系统设计和开发的可能性，使我们在想做的事情上几乎没有任何限制。

然后，我们得到了以根本独特的方式构建的创新。其他生态系统已经复制了 LiveView，正如他们应该那样。但他们的变体不能做到 LiveView 能做的一切，如果他们尝试的话，他们将会面临挑战。并且这不考虑让应用程序保持在线的所有实际问题。

Underjord 是一个[由 4 人组成的团队](/team.html)，专注于 Elixir 咨询和合同工作。如果你喜欢这些文章，真的应该试试他们的代码。更多信息请查看我们的[服务页面](/services.html)。

注意：或者尝试[YouTube频道上的视频](https://youtube.com/c/underjord)。
