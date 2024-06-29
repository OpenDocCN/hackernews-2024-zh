<!--yml

category: 未分类

日期：2024-05-27 12:53:04

-->

# LiveView 最适合与 Svelte 配合使用。

> 来源：[https://blog.sequin.io/liveview-is-best-with-svelte/](https://blog.sequin.io/liveview-is-best-with-svelte/)

我们是 [Sequin](https://sequin.io/docs?ref=blog.sequin.io)。我们将第三方 API 数据（例如 Salesforce、AWS）转换为 Postgres 表和事件流。作为基础设施公司，我们质疑是否真的需要 SPA。因此，我们开始使用 LiveView，这帮助我们快速推进，但仍然让我们渴望更多。本文讲述了这段旅程。

Phoenix 的 [LiveView](https://hexdocs.pm/phoenix_live_view/Phoenix.LiveView.html?ref=blog.sequin.io) 分化了我们的团队。与 SPA 相比，我们能够快速构建 2-3 倍的组件和功能。但反过来，有些组件和功能构建起来非常令人沮丧，或者感觉非常不直观。

换句话说，LiveView 让许多事情变得容易。但它也使一些简单的事情变得困难。

这种情况造成了紧张。我们是否继续沿着这条路走？还是屈服并将我们的应用程序转换为 SPA？

幸运的是，我们找到了一个名为 [LiveSvelte](https://github.com/woutdp/live_svelte?ref=blog.sequin.io) 的伴侣库。LiveView 结合 Svelte 提供了一种开发体验，与我之前使用过的任何全栈范式都不同。

团队一致认为：这是构建应用的一种绝佳方式。

为了理解 LiveView+Svelte 的范式，我将从解释 LiveView 的工作原理及其与众不同之处开始。然后，我将详细说明我们在纯 LiveView 方法中遇到的摩擦。到那时，您将能够欣赏到 LiveSvelte 提供了什么。

## 什么是 LiveView

LiveView 提供了一种非常独特的构建 Web 应用程序的方式。

在传统的服务器渲染的 Web 应用程序中，服务器是无状态的。客户端请求页面，服务器渲染页面。所有客户端操作都返回服务器，服务器重新渲染下一个页面。

在单页面应用（SPA）中，客户端负责构建页面。它利用后端 API 来读取和写入数据。客户端应用程序是有状态的（例如在 React 中使用 `useState`）。

在 LiveView 中，服务器负责渲染页面。但它是有状态的。前端的操作由后端处理，但服务器会 *增量* 更新 DOM，就像在 SPA 中一样。

从高层来看，SPA 复杂是因为分布式系统复杂。支持客户端 JS 应用程序就像支持微服务（而且是在一个敌对的、不可信的环境中运行的微服务！）

*理论上*，您的前端应用程序使用后端的 REST API，该 API *可以* 用于支持许多不同的服务和客户端。但实际上，您的前端应用程序的需求是独一无二的。因此，您的后端路由和控制器因服务单个客户端的需求而爆炸增长。

如果没有别的选择，这种复杂性意味着要做很多琐碎的工作。每个请求都需要在前端和后端进行大量的管道工作。调用堆栈很容易超过半打层：

+   `onMount`

+   `await api.fetchUsers`

+   `parseResponse`

+   `Router.handle(/api/users)`

+   `AuthPlug.verify_cookie`

+   `UsersController.index`

+   `Users.list_for_org`

+   `ApiHelpers.prepare_response`

LiveView 的承诺是，您可以创建丰富的客户端体验，而无需前端微服务。您回到了更简单的世界，在那里您可以在渲染表行的函数旁边查询数据库。如果有新的行进来，您只需要将它推送到表中，LiveView 将为您更新客户端。

但此外，您还可以享受使用前端框架的状态化范式构建应用程序的乐趣。通过这种方式比以前的后端范式更容易更快地构建丰富的交互模式，您无需在每次请求时“重建整个世界”。

## LiveView 使易事变难

LiveView 中有很多好东西。但也有真正的障碍。

我们在 LiveView 中遇到困难的两个主要领域是：

### 客户端状态是不可避免的

使用这种方法有（字面上的）光速限制：您的服务器只能与您的用户 *如此* 接近。

不可避免地，您需要在客户端执行一些操作。动画、工具提示、显示/隐藏 DOM 元素、禁用表单字段等等。

例如，我们的应用程序中有一个表单，有两个相互依赖的下拉菜单。在第一个下拉菜单中选择一个选项允许服务器为第二个下拉菜单生成列表。为了获得最佳用户体验，您希望在第一个下拉菜单更改后立即禁用第二个下拉菜单。然后，当服务器重新填充它时，您可以重新启用它：

*模拟客户端和服务器之间 1000ms 的往返延迟。*

据我们所知，要实现这一点，您需要在 LiveView 中使用两个独立的概念：

+   使用[JS 模块](https://hexdocs.pm/phoenix_live_view/Phoenix.LiveView.JS.html?ref=blog.sequin.io)在第一个下拉菜单更改时禁用第二个下拉菜单。

+   使用[hook](https://hexdocs.pm/phoenix_live_view/js-interop.html?ref=blog.sequin.io#client-hooks-via-phx-hook)在第二个下拉菜单上注册事件侦听器。然后，从后端发送一个动作来重新启用第二个下拉菜单。

对于稍微复杂的交互模式，您需要融入*第三*个概念，即 LiveView 状态。例如，也许您只想在特定条件下重新启用第二个下拉菜单。

这三个概念如何组合在一起并不明显（我们仍然不确定这是否是正确的模式！）

因此，虽然服务器负责大量 DOM 更改，但并非全部。您可以在必要时使用 JS 和 hooks 来添加 JavaScript。这些工具感觉像是附加到核心 LiveView 上，因此它们的使用模式并不明显。而且您使用的 JS 和 hooks 越多，您的 DOM 状态现在就存在于 LiveView *之外*。

这与 React 这样的范式形成鲜明对比。在 React 中，一路下来都是状态和动作。有了这个核心概念，您几乎可以做任何事情。而且 DOM 状态和组件状态之间没有模糊的界线。

React能采取这种方法是因为客户端操作和客户端状态之间没有延迟。这意味着你可以让React的状态范式处理每一个操作和转换。因为LiveView的所有状态都在服务器端，它必须应对客户端操作和服务器端状态之间的延迟。这意味着，虽然LiveView的状态*看起来像*其他前端框架，但模型实际上是非常不同的。

以输入字段为例。在React中，不能在不经过状态路由的情况下插入字符到输入字段中。这解锁了一个强大的编程模型，其中你的组件重新渲染——因此对每次按键作出响应。这为状态和操作范式提供了很大的影响力，在这里你可以使用一个核心概念（`useState`）来解决大量的问题空间。

在LiveView中，更准确地说是用户更改了输入字段，*然后*过了一小段时间LiveView才会知道并做出反应。没有延迟时，*它看起来很像React*。但是随着延迟增加，它的范式就大不相同了。

在像React这样的前端框架中，你需要一直应对服务器端延迟。但是*何时*进行高延迟操作是明确的（即你正在从服务器获取数据）。在LiveView中，这个边界就比较模糊。

### 三个组件

LiveView有三种不同类型的组件：LiveViews、LiveComponents和Components。

LiveView和LiveComponent就像React中的有状态组件，而Components则像功能组件。

重要的是，LiveView始终是最上层的父组件。你在LiveView下面作为子元素渲染LiveComponents和Components。

在React中，轻松切换状态组件和功能组件很容易——只需添加或删除`useState`钩子。两者的API是相同的（它们都以相同的方式接受props）。并且除了状态之外，它们具有相同的功能集。例如，它们都可以以相同的方式注册和响应DOM事件。

切换组件类型的简易性很重要。随着应用程序的成熟，你不断地提取组件。你要弄清楚哪些部分应该被重用，什么应该被泛化，状态应该放在哪里等等。

在LiveView中，这三个组件是非常不同的。因此，将LiveView重构为LiveComponent实际上是相当繁琐的。

特别是：

+   渲染和向LiveViews和LiveComponents传递props的语法是不同的。

+   LiveViews和LiveComponents的生命周期是不同的。

+   在LiveView和LiveComponent之间的[通信选项](https://hexdocs.pm/phoenix_live_view/Phoenix.LiveComponent.html?ref=blog.sequin.io#module-unifying-liveview-and-livecomponent-communication)是不同的。例如，你向LiveView发送信息，但向LiveComponent发送更新。

+   LiveComponents不是进程，因此无法像LiveViews那样与系统的其余部分进行交互。

最后一点是使LiveComponents如此不同和令人沮丧的原因。这些限制*是合理的*：LiveView是一个进程。这是LiveView最棒的部分之一，它们“只是进程”，因此可以像其他进程一样适应您的Elixir/OTP系统。例如，您可以在LiveView中使用发布/订阅来订阅系统范围的更改。

LiveComponent不是它自己的进程，它们是由LiveView调用的模块。父LiveView进程持有所有子组件的状态。因此，LiveView有一个`pid`，状态和一个收件箱；LiveComponent没有。这意味着LiveView还必须处理其子LiveComponent的所有消息路由。

这符合Elixir/OTP设计原则：进程是构建块。为了让LiveComponents具有独立状态管理和操作处理的能力，它们每个都需要成为自己的进程。

但是，对我来说，真的很难理解LiveComponents。我经常想给我的LiveComponent发送一个事件/动作，但却没有好的方法去做。我们最终使用了`send_update`，这是一个尴尬的API。我们无法决定：我们是通过`send_update`发送*动作*，还是用它来修补状态？如果我们在我们的`update`条款中使用它来修补状态，我们如何告诉是否正在装载或更新？

## 这就是“LiveView方式”的难以捉摸之处。

LiveView经常让我们感觉好像“缺了点什么”。“LiveView方式”似乎难以捉摸。

或许LiveView处于一种奇异谷状态。它与现代前端框架有很多共同点。因此，我们的“React大脑”和直觉会驱使我们使用旧模式——但这些往往会导致死胡同。更多的陌生感本应迫使我们认识到这些差异，并以不同的方式解决问题。

您可以仅使用LiveView状态和动作做很多事情。但是存在限制，当您遇到它们时，您需要切换范式。

它具有组件，帮助您组织和重用代码。但由于JavaScript和Elixir之间的差异，LiveView无法提供同构组件树，除非进行大量抽象，因此有LiveViews和LiveComponents。

**这就是使LiveSvelte如此有前景的原因**。正如您将看到的那样，它将更多责任转移到前端。它接受了前端将具有自己状态的事实。并且让您利用所有现代JavaScript组件框架的成熟性。

## LiveView + Svelte

LiveSvelte允许您从LiveView中渲染Svelte组件。这是一个很棒的范例。

有几种不同的方式可以从您的LiveViews渲染Svelte，但最基本的方式如下所示：

```
# LiveView component
defmodule Web.SyncLive.Form do
  def render(assigns) do
    assigns = 
      assigns
      |> Map.put(:encoded_collections, Enum.map(assigns.collections, &encode_collection/1))
      |> Map.put(:encoded_errors, encode_errors(assigns.changeset))

    ~H"""
      <.svelte
        name="MyForm"
        props={
          %{
            collections: @encoded_collections,
            credential_options: @credential_options,
            errors: @encoded_errors,
          }
        }
        socket={@socket}
      />
    """
  end
end 
```

这是一个Elixir模块，LiveView。在渲染中，我们首先将我们的Elixir数据结构编码为前端所需的形式。我们喜欢在传递给Svelte之前明确地将Elixir结构等编码为纯映射的模式，就像这样：

```
 defp encode_collection(%Collection{} = collection) do
    %{
      "id" => collection.id,
      "slug" => collection.slug,
      "name" => collection.name
    }
  end 
```

我们能够在Svelte组件上设置属性。这些属性如预期般传递到组件：

```
// Svelte component
<script>
  export let resource;
  export let credential_options = [];
  export let errors = {};
  export let live;
</script> 
```

LiveSvelte 为我们设置的 props 之一是 `live` prop。为了从 Svelte 组件向 LiveView 发送通信，我们可以调用 `live.pushEvent`。例如，看看将变更发送到服务器的过程是多么简单：

```
<script>
  // ...
  $: {
    live.pushEvent("form_updated", { form }, () => {});
  }
</script> 
```

这是 Svelte 中的一个响应式块。只要变量 `form` 发生变化，它就会被执行。（有点像 `useEffect`，其中 `form` 是依赖项。）

LiveView 可以处理并响应 `pushEvent`，使用典型的 Elixir 消息处理语义：

```
# In the LiveView
# ...
  @impl LiveView
  def handle_event("form_updated", %{"form" => form}, socket) do
    params = decode_params(socket, form)
    {:noreply, merge_changeset(socket, params)}
  end

  defp merge_changeset(socket, params) do
    changeset = Collection.create_changeset(socket.assigns.resource, params)

    assign(socket, :changeset, changeset)
  end 
```

我们首先从前端解码参数，逆转我们在出路时做的任何编码/映射。然后，`merge_changeset/2` 更新我们的 changeset。如果在 changeset 中有任何验证错误，这些错误将通过 `errors` prop 返回到前端。

因此，你有来自 Elixir 的数据流通过 props 传递到组件。LiveView 进程可以随时更新 props，导致 Svelte 组件重新渲染。任何其他通信可以通过 websocket 进行。

两者之间的边界非常明确——就像任何单页面应用程序一样清晰。

最具革命性的是，你有一个 *后端，有状态进程* 和一个 *前端，有状态进程* 在协作。

而且这样做 *既* 有趣又高效。

这三个强大的特性：

1.  后端控制前端组件上的 props。

1.  前端 *和* 后端都是有状态的。

1.  在两者之间你有一个私有的、双向通信通道，*其中任意一方都可以向另一方发起消息*。

#1 可以实现多亏了 LiveView 的渲染范式：服务器上的重新渲染会自动推送并应用于客户端。这使得服务器可以像 JS 的父组件一样更新组件上的属性！

这是可能的，因为 LiveView 是一个进程。进程是 Elixir 封装和减少状态的方式。

#3 可以实现多亏了 LiveView 提供的持久 websocket，连接到了前端。

考虑这种范式与单页面应用程序之间的区别：

首先，所有浏览器路由都通过后端进行。这是一个很好的简化器。（在常规单页面应用程序中，你必须维护 *两* 套路由，一个用于浏览器，一个用于 API。）

第二，后端是有状态的。它知道你所在的路由。你正在处理哪个资源。它处理的每个动作都可以更加增量化，因为它是将状态变更应用于自身，而不是从头开始重建状态。

第三，前端和后端之间的通信是私有的和耦合的，正如它应该是的。你不会在服务器的公共路由中使用一堆支持单个组件的 RPC 调用。当你在客户端看到 `pushEvent` 时，你知道处理它的位置——在协作的 Elixir 模块中。

第四，功能被分割到仅有两个文件中。当然，后端模块会调用你的后端函数（例如从数据库获取数据），前端会导入组件和样式。但是两者之间的往返不需要通过一堆API模块、路由器和控制器。

第五，前端和后端之间的通信要简单得多。后端可以简单地更新属性以通知前端的更改。前端可以使用`pushEvent`而不需要处理过期令牌、超时或故障。它是二进制的：要么WebSocket打开，这意味着服务器正在运行，要么关闭，这时LiveView会友好地向用户显示全局“断开连接”横幅。

简单来说，前端微服务被消除了。

最终你会感觉到责任分工得当，几乎没有样板代码。所有的业务逻辑都在后端 - 如何加载数据、*加载哪些*数据、如何排序和过滤数据，以及你的验证器等等。你的前端代码非常简单。在Svelte中，它全部是（1）`if/end`块来有条件地渲染东西（2）动画和（3）几个非常简单的`pushEvent`函数返回到服务器。

最后那部分真是让我大开眼界。典型的SPA前端通常充满了大量逻辑，通常是`map`、`reduce`和`filter`，以便处理服务器数据，为显示准备数据或为服务器准备数据。在LiveSvelte应用中，所有这些都可以在服务器端完成。LiveView可以精确地准备数据，使其与Svelte组件所需的数据完全匹配。这将复杂性留在了你的服务器语言、服务器数据结构和服务器测试套件中。

后端LiveView和前端Svelte组件并不是紧密耦合，而是两半：LiveView仅呈现该Svelte组件，并且该Svelte组件仅由该LiveView呈现。

与“常规”LiveView相比，这种范式：

+   在前端中拥抱状态和状态转换。

+   在前端和后端之间创建了一个清晰的边界层。

+   利用Svelte的组件范式，像其他当代JS框架一样非常成熟和熟悉。

+   总的来说，让出色的前端框架发挥其所长！纯LiveView方法不允许你利用这个庞大的生态系统。（例如，Svelte带有出色的动画基元。）

通过更多地移到前端，我们不再感觉自己处于尴尬的中间地带。

我们选择了LiveSvelte，因为React没有类似完整的LiveView库。与Svelte一起工作的乐趣是一个非常愉快的意外。因为LiveView处理了状态管理的繁重工作，我们在Svelte中的状态管理非常简单。对于基本的状态和响应性，Svelte是我使用过的最轻量、最快的前端框架。我们也更喜欢其模板特性，尤其是可以使用`if/else`而不是三元运算符和其条件属性设置。

进一步说，Svelte 5 即将面世，我们对其 [符文](https://svelte.dev/blog/runes?ref=blog.sequin.io) 感到乐观。我们认为这使得 Svelte 更易上手和理解，这意味着团队中的每个人都能够跨越技术栈。

我现在确信 LiveView 最擅长作为后端-for-前端。通过渲染前端组件，增量更新它们，维护有状态的后端进程，并提供 websocket API，它为前端应用程序创造了一个极具生产力的平台。

如果你正在使用 LiveView 并且对我强调的任何摩擦感到 resonated，你需要试一试。如果你从未使用过 LiveView，你会发现这种范式 *降低* 了学习曲线。这是因为你可以使用大量你熟悉的 JavaScript 框架原语。

* * *

**2024年3月4日更新**：加入 [Hacker News](https://news.ycombinator.com/item?id=39916144&ref=blog.sequin.io) 上的讨论。
