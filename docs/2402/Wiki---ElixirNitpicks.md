<!--yml

分类： 未分类

日期：2024年5月27日14:40:43

-->

# Wiki - ElixirNitpicks

> 来源：[https://wiki.alopex.li/ElixirNitpicks](https://wiki.alopex.li/ElixirNitpicks)

因此，在2023年中期，我使用 Elixir 写了一个[小而非平凡的 web 应用](https://linkrot.cc/)，这也是[学习这种语言的练习之一](ElixirForCynicalCurmudgeons)。这是我在使用这种语言时遇到的问题和怪异之处的列表。这主要是与 Rust 进行比较，因为这是我目前考虑的首选工具，这一切都是使用 Elixir 1.14 来完成的。

最后更新于2024年4月。

# 错误处理 

Elixir 的错误处理基本上是两个不同世界的结合：异常风格的“raised”错误和 Rust 风格的返回的元组`{:ok，val}`或`{:error, e}`。问题就在于这些东西相互作用的地方。

大多数可能失败的函数返回的是元组 `{:ok，val}`或 `{:error，thing}`。一些东西在失败时会`raise`一个异常，大多数返回错误元组的函数都有一个与相同函数名称加上`!`的版本抛出一个错误。这在交互使用中非常方便，这样你就不必不停地模式匹配来获取运行函数的结果，这会变得很烦人。就像`DB.update`返回`{:ok，val}`或`{:error，e}`，`DB.update!`会返回`val`或抛出`e`的异常。到目前为止还可以吧。我想你总是可以写`try do thing!() rescue e -> {:error，e} end`将抛出的错误转换为返回的错误，你也可以像`{:ok，val} = thing()`这样相当快速和肮脏的方式来引发一个模式匹配错误如果发生了错误。

问题在于对这些事物进行排序和链接，而 Rust 恰好有更好的习语。虽然 Elixir 的 `with` 语句做得 *相当* 不错，但它不太适合嵌套，并且会导致在其中嵌套其他块（如`if`表达式）时出现问题。但由于 Elixir 没有提前返回，你实际上无法像 Rust 的`?`或 `try!()`那样编写一个早期退出的单一表达式。（读者的练习：证明我错了。）当你将多个返回错误值的步骤链接在一起时，也存在使用 Elixir 的 `with` 函数的问题，但是它们的错误值看起来很相似，你想知道到底哪一个在你的错误处理中实际上失败了。[https://dev.to/martinthenth/using-elixirs-with-statement-5e36](https://dev.to/martinthenth/using-elixirs-with-statement-5e36) 在标题为“Differentiating non-matching clauses” 的部分中提供了一个相当巧妙的解决方案，其中你可以像这样处理：

```
with true <- is_email_address?(email),
 true <- EmailAddresses.is_available(email) do
 ...
else
 false ->
 # Return either '{:error, :bad_request} or '{:error, :conflict}'
end
```

并通过编写类似于这样的内容来处理：

```
with {:is_email, true} <- {:is_email, is_email_address?(email)},
 {:is_available, true} <- {:is_available, EmailAddresses.is_available(email)} do
 ...
else
 {:is_email, false} ->
 {:error, :bad_request}

 {:is_available, false} ->
 {:error, :conflict}
end
```

这将在系列中调用两个函数，每个函数返回一个`bool`表示成功或错误，并将它们转换为两种不同的返回类型，你可以在模式匹配中加以区分。老实说，这不是可怕的，但感觉很奇怪和脆弱。我不确定我是否会认为它能达到以下功能：

```
is_email_address(email).map_err(bad_request)?;
is_available(email).map_err(conflict)?;
```

这样做只是为了应该是常见的东西。 

另外一个函数可能会返回 `{:ok, thing}`，或者如果它没有值可返回，则可能只会返回 `:ok`。你必须知道是否需要执行 `:ok = thing()` 还是 `{:ok, _val} = thing()` 因为如果你选择错误的方式会导致错误。因此，在 Elixir 中编写通用错误组合代码有点棘手，因为它必须检查这两种可能性。如果一个函数必须返回 `{:ok, thing1, thing2}` 怎么办？算了吧，没有错误组合器会试图处理每一个可能的元组长度。只需返回 `{:ok, {thing1, thing2}}` 并继续生活，这在思考时更有意义，但也不是那么明显。

另一个错误微妙之处是，“让它失败”在需要向用户呈现良好错误消息时不适用于验证用户输入。有时候验证也不能发生在边缘，因此你必须明确处理代码的深度调用链中的错误，以及一些其他相邻的深度调用链，它们只是记录一些东西然后终止。(嗯，如果验证总是发生在边缘，那可能是我的问题的一部分...)

# 状态管理

在 Elixir 程序中几乎任何全局状态变化都可以并且将被隐藏在一个函数调用之后。没有真正的方法来表达这一点，不像 Rust 中你几乎总是要把 `&mut something` 传递给函数。对于一个纯不可变的函数式语言来说，Elixir 程序中确实有 *很多* 全局可变状态。

一般的常见习语是将你的可变状态隐藏在一个进程中，然后不是直接向它发送消息，而是编写包装函数来为你完成。一个很好的例子是数据库连接。在 Rust 中，你往往会有像这样的东西：

```
let db = Db.open("...")?;
db.query("stuff");
db.insert("other stuff");
stash_into_mutable_global(db);

// in some other function
let local_db_reference = get_db_from_mutable_global()
local_db_reference.query("...")
// local_db goes out of scope
```

在 Elixir 中，你有：

```
DB.open("...")
DB.query("stuff")
DB.insert("other stuff")

// in some other function
DB.query("...")
```

有一个全局进程注册表，你可以给它们附加任意名称，并且库经常使用这个 *很多* 。因此，即使你个人不写它，你最终也会有 *大量* 的全局可变状态。这非常奇怪并且有点令人不安。确实，状态都封装在进程中，但这些进程又隐藏在一个使它们不可见的抽象层后面，所以实际上你只是在触及全局变量。

# 导入

太多了，有各种各样的方式可以导入一个模块。你可以：

+   不要导入它，并通过完全限定的名称引用其内容，`Foo.Util.frobnosticate()`

+   `import` 它，将其所有符号都装入本地命名空间。因此 `import Foo` 允许你调用 `Util.frobnosticate()`，而 `import Foo.Util` 允许你调用 `frobnosticate()`

+   `alias` 它，让你将某个模块路径重命名为另一个，可能更短的模块路径。因此，`alias Foo.Util, as: U` 允许你调用 `U.frobnosticate()`

+   `require` 让你能够调用它里面的宏，通常情况下不能调用。所以在执行 `require Foo.Util` 之前，你不能直接调用 `Foo.Util.frob_macro()`，之后可以用 `Foo.Util.frob_macro()` 调用。尽管显然如果你使用 `import Foo.Util` 也可以直接调用 `frob_macro()`，因此在这里存在重叠行为，我显然没有完全理解。

+   `use` 会调用它里面的回调，可能是生成代码的宏。所以如果你写了 `use Foo.Util`，相当于调用了 `require Foo.Util; Foo.Util.__using__()`，并将其返回的代码插入当前模块。

与 Rust 相比，它使 `use` 执行所有这些操作：

+   没有导入会导致没有导入，就像 Elixir 一样，你可以通过完全限定的名称引用一个东西。

+   `import Foo` 使用 `use foo;` 完成。

+   `import Foo.Util` 使用 `use foo::util::*;` 完成。

+   `alias Foo.Util, as: U` 使用 `use foo::util as u;` 完成。

+   更常见的 `alias Foo.Util, as: Util` 等同于 `use foo::util;`

+   `require Foo.Util` 自 2018 年以来，宏命名空间与函数命名空间在导入方面基本上被一视同仁。Rust 用感叹号结尾视觉上区分宏，因此没有调用意外执行宏的风险。

+   `use Foo.Util` 是唯一没有真正等价的方式；最接近的可能就是过程宏…… 在语言中和其他宏一样对待。

`use` 足够奇怪，很容易与其他不同，但是 `import`、`alias` 和 `require` 之间的确切区别是一个持续的头痛。你可以让一件事情做所有这些：

+   `import Foo` -> 可以调用 `Util.whatever()`

+   `import Foo.Util` -> 可以调用 `whatever()`

+   `import Foo.Util, as: U` -> 可以调用 `U.whatever()`

+   `import Foo.Util, macros: true` -> 可以调用 `whatever_macro()`（可以与 `all:` 或 `as:` 关键字结合使用）。

+   让 `use` 保持原样，它是一个足够奇特的特例，值得有所不同。

我尝试编写一个宏来实现类似的东西，但其实我对这个问题并不太感兴趣，所以没有花太多时间在上面。而且，要使这个宏存在，你必须在每个模块开始时写上 `require CoolMacro`！ 烦人！要求 `require` 实际上相当难以掩盖，除非改变语言的工作方式，我认为。

我曾考虑建议去掉一些显式宏导入的必要性，我仍认为这是个好主意，但Elixir的宏比Rust更加重，所以可能有很好的理由保持现状。尽管如此，Elixir的宏相当……泄漏，这让人非常沮丧。例如，`Integer.is_odd()`就是一个宏，因此如果直接调用它，编译器会友好地提醒你首先调用`require Integer`。我欣赏这一点，但通常如果编译器足够聪明以告诉你问题的确切解决方案，它应该足够聪明以直接为你做出调整。你可能会问，为什么他们首先将`is_odd()`设为宏？因为这样它可以用作匹配守卫，而函数不能，因为匹配守卫是语言的有限子集，没有副作用。这一切都说得通，但*太惊人了*。我们陷入了一个四层深的非明显因果链中，涉及到任意代码生成的危险，而你*真正*想要的是要么有一种方法编写可以用作模式守卫的纯函数，要么一种方式以视觉上区分函数和宏，如Rust的符号，以便更容易看出发生了什么。

# 混合信息

我在Elixir Discord聊天中向新手分发的常见传说包括：

+   如果可以避免，不要使用Umbrella项目

+   如果可以避免，不要使用代码的实时升级 - 例如，更喜欢通过重新启动节点来升级，而不是在迁移状态时热重载新代码

+   如果可以避免，不要使用宏

+   Mnesia数据库更像是Redis的替代品，而不是PostgreSQL的替代品。这更多关于Erlang的消息传递而不是Elixir的，因为Mnesia与Erlang运行时分布式相关，但仍然相关。我也听说过Mnesia被用作控制平面/配置数据库，而不是用户数据数据库，它在这方面可能也非常擅长。

如果Elixir教程或FAQ包含更多这样的智慧，而不是列出它们应该告诉你的漂亮特性，那将是很好的。看起来有很多人运行`mix help new`，看到单行提示“可以使用--umbrella选项生成Umbrella项目”，然后陷入兔子洞，了解Umbrella项目是如何工作的，而不是做点有用的事情。甚至**描述**什么是Umbrella项目并说“除了最大的项目外，大多数项目都不需要这样做”会有所帮助。

# 单元测试更让人头疼

…尽管我真的很难表达为什么。

# 领域特定语言

这更多是与库而不是核心语言本身的问题，但这些核心库（如 Phoenix、Plug、Ecto 等）往往是你首先遇到并且经常使用的东西。然后其他人写的程序 tend to follow 这些核心库教授的习惯用法。基本上，这些库有很多新语法，它们是由宏构建的，这些语法总是遇到宏 DSL *总是* 遇到的问题，即定义不清晰的语义。

例如，使用 Phoenix 时，我有一个稍微奇怪的问题，一个组件作为函数实现了几个私有子组件... 基本上我有：

```
defmodule LinkrotWeb.Components.Navbar do
  use Phoenix.Component
  ...

  attr :page_title, :string, required: false, default: ""
  defp navitem(%{title: _, path: _, name: _} = assigns) do: ...
  defp rnavitem(%{title: _, path: _, name: _, label: _, first: _} = assigns) do: ...

  def navbar(assigns) do
    links = ...
    assigns = assign(assigns, :links, links)
    ~H"""
    <ul class="navbar">
      <%= for {name, path} <- @links do %>
        <.navitem title={@title} name={name} path={path}/>
      <% end %>

      <.rnavitem title={@title} path={~p"/users/register"} name="Register" label="Register" first={true} />
      <.rnavitem title={@title} path={~p"/users/log_in"} name="Log in" label="Log in" first={false} />
    </ul>
    """
  end
end
```

一切都正常工作，但我对 navitem 函数收到警告：

```
warning: undefined attribute "name" for component LinkrotWeb.Components.Navbar.navitem/1
  lib/linkrot_web/components/navbar.ex:61: (file)

warning: undefined attribute "path" for component LinkrotWeb.Components.Navbar.navitem/1
  lib/linkrot_web/components/navbar.ex:61: (file)

warning: undefined attribute "title" for component LinkrotWeb.Components.Navbar.navitem/1
  lib/linkrot_web/components/navbar.ex:61: (file)
```

但我不明白为什么在 rnavitem 函数中没有这些，而且一切都能正确渲染，所以我困惑于这些到底是从哪里来的。结果 `attr` 宏适用于紧接其后的函数，而不是其所在的模块。从某种角度讲这是有道理的，因为这与 Elixir 的普通类型注解一样，但 Elixir 的文档也倾向于将“组件”视为整个模块，而不是该模块内的特定函数。文档中有说明，所以在某种程度上是我没有仔细阅读文档的错，但对我来说当时也不是显而易见，因为 Phoenix 有很多其他像 `plug`、`pipeline` 和 `embed_templates` 这样的宏 DSL，它们只是简单地插入到封闭它们的模块中并像魔法一样处理事物。

“让我们只需将宏创建为声明性 DSL 以使生活更轻松，会出什么问题？” 它实际上并不是声明性的，并涉及对代码结构的隐含假设，这就是问题所在。你可以像这样编写代码：

```
 def navbar(assigns) do
    attr(assigns, [
      [:page_title, :string, required: false, default: ""],
      [:current_user, :string, required: false, default: nil],
    ])
    links = ...
```

而且它将是 *完全没问题*。并且还清楚地表明 `attr` 与函数关联，而不是模块。我还没有看到任何使用 `attr` 的情况，它不能通过正常的 Dialyzer `@spec` 声明来替代，尽管可能存在一些例外情况。也许是默认值？我不知道，总体来说，整个 `assigns` 系统在我看来有点过于神奇。Assigns 有其存在的理由，但是我试图同时处理 Elixir、HEEx 模板和与它们相关的各种 DSL 片段时会遇到问题。

# Misc

+   Ecto，事实上的标准数据库查询构建器，需要更多的关注。它并不差到令人无法想象的地步，但与其他项目相比，我在文档和 API 设计中遇到的许多粗糙之处都是在 Ecto 中。`Repo.one` 返回 `value | nil`，而 `Repo.insert` 返回 `{:ok, value} | {:error, changeset}`，`Repo.transaction` 函数 [真的需要一些相当基本的改进](https://tomkonidas.com/repo-transact/)，我仍然不确定做连接的最佳方法是什么，在两个单独的配置文件中保持迁移和模式同步有点痛苦，等等。然而，所有这些问题也都是相当表面的问题。在操作上，Ecto 表现迅速、灵活，生成良好的查询，并且据我有限的数据库经验看来，倾向于引导您朝着良好的设计模式前进。它的查询 DSL 是我学习 DSL 完全值得且非常有用且低泄漏的抽象的少数几次之一。

+   热代码重新加载有时候有点讨厌。有时候你保存文件后，它会立即自动进行热代码重新加载，而其他时候可能需要几秒钟。有些错误会在系统自动重新加载文件时修复，而有些则不会。这有点讨厌。总体上，监视文件变化比看起来要困难得多，但我真希望它能更好一些。

+   Erlang/Elixir 声称的可靠性基本上只涉及语言本身。大多数时候，我觉得在 Elixir 或 Erlang 中编写健壮、容错的代码并不比其他以不可变优先的函数式语言如 OCaml 或 Clojure 更容易或更难（它可能是它们最接近的近亲，因为它是动态类型的）。错误会发生，并以丑陋的方式导致失败，有时候你真的希望有 Rust 的 `Result` 类型和 `?` 运算符。BEAM/OTP 给你的是工具，用来在错误发生时*处理*它们，而生态系统广泛使用这些东西。我有一段时间以来的直觉是，`systemd` 和 Unix 进程树实际上只是 OTP 的监督树的一种弱化和实现不良的版本，现在我真正使用了一段时间后，这一直是正确的。至少目前是这样。

+   Erlang 和 Elixir 之间存在一种文化分裂，这比必要的生活更加艰难。实际上，在技术上，这两种语言通过 BEAM 互操作得非常好，所以这种分裂*不存在*。例如，由于 Erlang 库存在许多功能，Elixir 库中缺少这些功能，从 Elixir 调用 Erlang 代码（反之亦然）几乎不费吹灰之力。但是，Elixir 有 `mix` 构建工具，Erlang 有 `rebar3`，在同一项目中包含两种语言的交叉构建并不像应该那样毫不费力，据我所知，你不能真正从 Erlang 调用 Elixir 编译器，等等。据说，当 Elixir 刚开始时，它们与 Erlang 社区之间存在一些嫌隙，这是这种裂痕的根源。但似乎没有人再抱怨，两种语言和社区似乎相处得很好。只是在构建100%兼容的工件时，拥有两种不同且大部分不兼容的构建工具真的很烦人。

+   由于某种奇怪的原因，Elixir Discord 社区中缺少穿程序员袜子的酷儿毛茸茸动物，至少与 Rust 或其他大多数我见过的技术 Discord 服务器相比如此。这引起了一些奇怪的认知 dissonance。为什么我在网上与所有这些善良、知识渊博、友好和富有同情心的技术兄弟姐妹一起时感到有些奇怪呢？然后我看到一个我在别处认识的名字，我的大脑后部说“哦，感谢众神，我确实知道她在空闲时间是一只雪豹”。好吧，这只是一个**明确**的开玩笑，但 Rust 用户群体继续是一个有趣的案例研究，展示了当你*非常明确地*说可以做怪人时，你可以在一个地方聚集多少怪人。这并不是抱怨；友好的技术兄弟姐妹也需要他们的位置，但这是一个有趣的对比。在 COVID 前我参加过的最后一次 Rustbelt Rust 活动中，“你对其他语言有兴趣吗”的最常见答案是“Elixir”，远远超过其他语言，因此我觉得 Elixir 在时间和认知空间上与 Rust 有很大的重叠，所以这是一个奇怪的文化分裂。我在 Elixir 社区遇到的任何人都没有一点不友好或排外的感觉，但氛围明显更具企业专业性，所以我想我们怪人并不像让面具下来那么舒服。也许这是因为 Elixir 更多地面向相当主流的 web 技术，而我对其他大多数我感兴趣的事物（编程语言、图形、机器人技术、游戏开发）则更为小众？

# 正面的笔记

以积极方式让我感到惊讶的事情：

+   所以，Elixir 在任何地方都是动态类型的。有一个叫做 Dialyzer 的程序会为你检查可选的类型注解，但有时它感觉有点像是一个二等公民，至少与 Rust 的类型系统和错误消息相比。但有点令我惊讶的是，实际上它表现得*非常好*。如果你允许它，Dialyzer 会帮你捕捉很多类型错误，我使用过的几乎所有库都对所有内容有完整正确的类型注解。Dialyzer 并非百分之百可靠，如果给出不完整的信息，它可能会以相当神秘的方式失败，但总体而言，从第一天开始使用它是非常值得的。（Erlang/Elixir+Dialyzer 是一个非常有趣的参照点，与像 TypeScript 或 Python 这样的逐渐类型化语言形成鲜明对比，我可以长篇大论。）

+   同样地，Elixir 在很大程度上是动态*绑定*的。如果我理解正确，大多数函数调用在语义上被视为按符号名称调用，与运行 `apply(Modulename, :function_name, [args])` 相同。然后编译器会在编译时优化并将其内联为更直接的调用，如果名称在编译时始终已知。这导致了一些对 Rust 程序员来说感觉非常奇怪的事情，比如对不存在的函数或模块的调用只是编译器的*警告*而不是错误。事实上，Elixir 编译器几乎*从不*直接给出错误，基本上只有在无法解析文件时才会失败。这感觉像是一种灵异的感觉……但它的警告基本上总是正确的，很少会漏掉任何东西，所以实际上你只需像对待 `rustc` 的错误一样对待 Elixir 的警告，一切都会很好。

+   当谈到在 Elixir/Erlang 中编写可靠的代码和构建系统时，任何文档首先都会告诉你的是监督树。但很多时候，你大部分时间都不需要去碰它们。每个启动长期运行进程的库都有自己的监督树，Phoenix 会在自己的监督树中启动你的应用程序，并有一个钩子让你编写来连接它们。实际上几乎不需要任何配置或调整，你只需让它们做它们要做的事情，直到你扩展到足够大，真正需要开始增加自己的子进程树以进行并行处理或错误隔离。至少在使用 Phoenix 时，你的程序在实际上变得相当庞大之前是不需要考虑将其拆分成不同的进程的，因为很多工作已经完成了。

# 结论

Elixir 相当酷炫。你应该使用它。

Phoenix 在某种程度上让我有点心烦，但它的初衷是好的，并且在实践中运行良好。它只是遭遇了 Web 框架病，试图使简单的事情过于简单，结果使复杂的事情变得过于复杂。

Ecto是一个非常好用的查询构建器，实际上使得SQL中许多痛点变得不那么痛苦，而且不会引入更多的尖锐边缘。不过，我确实希望迁移只是从模式中生成，就像Django那样。

生态系统中还有很多其他优秀的工具；Oban、Mix、Telemetry/Logger，以及大部分的Erlang标准库都在“10/10，会再次使用”的清单上。许多更为专业的工具虽然功能基本完备，但细节处理可能稍显粗糙。

许多其他工具在“一流”抛光级别上稍显不足；Image/Vix、ExAws和其他更小的工具都完全可用，但它们的API表面可能没有被充分或者强烈地测试过，因此，如果你在一个不熟悉的领域开始或者做一些奇怪的事情，它们可能需要比应有的更多的摩擦才能达到你想要的效果。

我非常激烈地避开了Phoenix LiveView，但或许我不应该这样；它似乎是对[HTMX](https://htmx.org/)采用了很多相同原理的一个非常有趣的尝试。它只是让我对时髦的技术术语和闪光的Web技术充满了畏惧。也许有一天我会写一篇“供愤世嫉俗者使用的LiveView”。虽然我不确定这是我特别想追求的品牌……

我需要用Elixir写一个视频游戏。总有一天！

# 参考资料

一些次要的事情和类似的对话。
