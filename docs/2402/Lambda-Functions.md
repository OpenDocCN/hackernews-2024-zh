<!--yml

category: 未分类

date: 2024-05-27 14:53:41

-->

# Lambda 函数

> 来源：[https://www.lambdafunctions.com/articles/elixir-and-rust](https://www.lambdafunctions.com/articles/elixir-and-rust)

# 在Elixir中管理可变数据与Rust

Elixir的核心优势之一，也是其健壮性和可伸缩性的秘密，是其建立在不可变数据基础上。不过，有时不可变性并不适合某个特定任务——但该任务仅是大项目的一部分。在其他所有地方都享受Elixir数据模型的好处，但为一个区域创造一个小小的可变例外，是否可能呢？

是的！

我参与的一个长期运行的Elixir项目正面临这个问题。该项目通过网络交付，因此完全放弃Elixir并不在考虑之中，因为[Phoenix](https://www.phoenixframework.org/)简直就是Web开发的秘密武器。我们*可以*将可变部分分离成微服务，但这将需要显著的架构和管理开销。我们真正需要的只是一个小小的逃逸口，用于有限的代码块，同时仍然在同一个VM中，并能够正常地与服务的其余部分进行交互。

这就是[Rustler](https://github.com/rusterlium/rustler)提供的功能。

Rustler是“用安全的Rust代码编写Erlang NIFs的库”——换句话说，你可以编写看起来像标准Elixir函数的代码，但实际上是在Rust中实现的。

[NIFs](https://www.erlang.org/docs/17/tutorial/nif)是Erlang虚拟机的特性，在Elixir或Rust出现之前就已存在。Rust和Rustler增加的是：

+   safety—this is critical since a crash in a NIF will bring down the whole VM

+   许多额外的磨练和接口粘合使得可以编写更加雄心勃勃的集成，这种集成可能比你想尝试的C和Erlang标准NIF支持更有可能。

+   可以访问所有Rust的库

不幸的是，网络上大多数Rustler示例都集中在速度优势上，并展示了一个简单的`add`函数的实现，然后就此打住了。虽然这对于演示最低集成要求足够了，但对我来说，Rustler的有趣之处在于以一种受控的方式逃离不可变世界——我想探索如何以安全的方式管理少量可变内存块。尽管Rustler当然能够做到这一点，但在教程或示例方面可用的资源非常有限。

希望本文能有所帮助。

## 目标

如上所述，我希望探索内存管理。更具体地说，我希望能够在Rust世界中保持一块数据，这些数据可以在不同的“Elixir”（Rustler）函数调用之间保持持久。这些函数应该允许Elixir世界将数据传递到Rust世界中，改变那里保存的数据，然后检索结果。

为了提供一个实质性的示例，并避免为此演示实现自己的数据存储，我将使用Oxigraph。

[Oxigraph](https://github.com/oxigraph)是一个实现SPARQL标准的Rust图数据库库。假设我们想要封装它，这样我们就可以从Elixir内部访问一个快速的图数据库。我们将称之为`FeGraph`。

我们想要能够：

1.  创建一个新的内存数据库

    ```
    db = FeGraph.new()
    ```

1.  向其中添加数据

    ```
    FeGraph.set(db, "http://foo.bar.com")
    FeGraph.set(db, "http://foo.baz.com")
    ```

1.  将数据库导出为[Turtle](https://en.wikipedia.org/wiki/Turtle_(syntax))字符串：

    ```
    FeGraph.dump_db(db) |> IO.puts()
    ```

为了让这个长度合理，我们不打算实现需要暴露Oxigraph所有功能所需的一切；只需要足够来展示在Rust中保存数据并从Elixir中进行操作。

## 实现

（如果你想跟着做，我建议你在进入代码之前，先按照我之前提到的许多Rustler `add` 教程之一，这样你就可以运行一个基本的项目，并且了解了Rust和Elixir函数如何联系在一起。后面的一切都假设你已经到了那一点。）

整个方法的关键在于能够在两个世界之间传递一个[Resource](https://erlang.org/doc/man/erl_nif.html#resource_objects)。这充当了一块内存的句柄；它可以从NIF返回，然后再传递回另一个调用。正是我们所需要的。

一个BEAM `Resource`在Rustler中由一个`rustler::resource::ResourceArc<T>`结构表示。让我们从我们的实现开始，定义一个我们可以用作表示图存储状态的句柄的新类型。

在生产环境中，我们可能想管理更多的状态，但现在只需定义一个`MyGraph`结构，它只包含（通过互斥锁）Oxigraph数据存储；这代表了我们想要在Elixir外部管理的可变数据。将来，根据需要，可以向`MyGraph`添加更多的字段。

为了将其转换为可以在Elixir和Rust之间传递的东西，我们需要将其包装在`ResourceArc`中。为了使我们的函数签名更加可读，我们将定义一个新类型的`GraphArc`，表示在`ResourceArc`中的`MyGraph`结构。

在`lib.rs`中：

```
use std::sync::Mutex;
use oxigraph::store::Store;
use rustler::resource::ResourceArc;
use rustler::OwnedBinary;
use rustler::{Env, Term};

struct MyGraph { store: Mutex<Store> }

type GraphArc = ResourceArc<MyGraph>;
```

需要一点额外的管道工作告诉Rustler，`MyGraph`是可以用作`Resource`的东西：

```
fn on_load(env: Env, _info: Term) -> bool {
    rustler::resource!(MyGraph, env);
    true
}

rustler::init!("Elixir.FeGraph", [], load = on_load);
```

有了这些定义，我们可以编写一个`new`函数，该函数分配一个新的数据存储并返回一个句柄：

```
#[rustler::nif]
fn new() -> GraphArc {
    ResourceArc::new(
        MyGraph {
            store: Mutex::new(Store::new().unwrap()),
        }
    )
}
```

在Elixir端，在`fe_graph.ex`中：

```
defmodule FeGraph do
  use Rustler, otp_app: :myapp, crate: "fegraph"

  def new(), do: :erlang.nif_error(:nif_not_loaded)
end
```

在这一点上，我们可以在`iex`中进行测试，看到上面的Elixir存根已被我们定义的Rust NIF替换，我们可以运行它，并得到一个引用：

```
iex(1)> FeGraph.new
#Reference<0.2659174309.2607677441.115195>
```

虽然我们还无法*做*任何操作，但我们已经在Rust中定义了一个数据存储，并在Elixir中看到了它的迹象；在幕后，BEAM和Rustler正在为我们处理所有繁重的工作。

我们来写一个简单的函数向我们的新存储中添加一些数据怎么样？以一种对Elixir代码非常熟悉的方式，它将需要接受并返回一个`GraphArc`句柄。我们还将接受一个字符串以用于三个三元组部分的存储（当然，通常我们会为主语、谓语和宾语部分使用不同的字符串，但我们此处的重点不是SPARQL——我们只想存储一些数据。）

```
#[rustler::nif]
fn set(state: GraphArc, iri: &str) -> GraphArc {
    let store = state.store.lock().unwrap();

    let ex = NamedNode::new(iri).unwrap();
    let quad =
        Quad::new(ex.clone(), ex.clone(), ex.clone(), GraphName::DefaultGraph);
    (*store).insert(&quad).unwrap();

    drop(store);

    state
}
```

在函数内部，我们可以使用我们的`GraphArc`状态参数来获取我们在`new`函数中创建的Oxigraph存储的句柄。一旦获取到，我们可以像往常一样向图中添加一些测试数据，然后返回未更改的`state`。

最后一步是从我们的存储中检索一些数据。与其运行一个查询（这将需要更多SPARQL的介入），我们只需倾倒整个数据库并将其作为字符串返回。与以往一样，我们的新函数将需要接受一个`GraphArc`，但这次我们将返回一个`OwnedBinary`，这使我们能够将二进制数据发送回BEAM，然后摆脱它。

```
#[rustler::nif]
fn dump_db(state: GraphArc) -> OwnedBinary {
    let store = state.store.lock().unwrap();

    let mut buffer = Vec::new();
    (*store)
        .dump_graph(
            &mut buffer,
            GraphFormat::Turtle,
            GraphNameRef::DefaultGraph,
        )
        .unwrap();

    let mut result = OwnedBinary::new(buffer.len()).unwrap();
    result.as_mut_slice().copy_from_slice(&buffer);

    result
}
```

这个函数的大部分内容实际上是将数据从Oxigraph中获取并存入缓冲区，然后从缓冲区转换为`OwnedBinary`；Rustler包装器已经变得几乎无需关注，这正是我最初所希望的。

有了这个功能，我们现在可以演示在Rust中分配一些内存，返回该内存的句柄，然后使用它在BEAM内存模型之外存储数据，并最终将数据取回到Elixir世界中：

```
iex(1)> db = FeGraph.new
#Reference<0.2749498138.3684302852.140448>
iex(2)> FeGraph.set(db, "http://foo.com")
#Reference<0.2749498138.3684302852.140448>
iex(3)> FeGraph.dump_db(db)
"<http://foo.com> <http://foo.com> <http://foo.com> .\n"
```

注意重要部分；尽管数据更改后引用相同，我们在`set`调用后并*不*存储它；通常我们需要像`db = FeGraph.set(db, "http://foo.com")`这样做。`set`返回引用仅为了便于与管道运算符或类似操作的使用。

## 结论

尽管上述代码跳过了大部分错误处理，但希望很明显Rustler使得将Rust代码链接到Elixir项目变得非常容易，从而允许您充分发挥两者的优势。

Rustler对Elixir生态系统是一项重大增强，它不仅可以更快地计算，还可以打开更多的机会。能够选择退出标准BEAM内存模型以用于代码特定部分，可以为自定义数据存储和其他通常不适合Elixir的功能打开大门，同时仍然可以利用Phoenix强大的功能来构建应用程序的大部分部分……几乎无缝集成。
