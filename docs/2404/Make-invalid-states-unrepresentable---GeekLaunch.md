<!--yml

category: 未分类

date: 2024-05-27 13:32:23

-->

# 使无效状态不可表示 - GeekLaunch

> 来源：[https://geeklaunch.io/blog/make-invalid-states-unrepresentable/](https://geeklaunch.io/blog/make-invalid-states-unrepresentable/)

让我们谈谈类型。

这篇文章适用于大多数编程语言，但在使用功能更强大、受函数启发或数学化类型系统的语言时，应用这些概念会更容易^。

正如你可能知道的那样，Rust 编译器可以输出一些非常高质量的错误消息。

```
error[E0596]: cannot borrow `x` as mutable, as it is not declared as mutable
  --> src\lib.rs:11:5
   |
10 |     let x = vec![];
   |         - help: consider changing this to be mutable: `mut x`
11 |     x.push(1);
   |     ^^^^^^^^^ cannot borrow as mutable

For more information about this error, try `rustc --explain E0596`.
error: could not compile `invalid-state` due to previous error
```

例如，当代码尝试修改一个不可变绑定时，它会建议改变其可变性。

```
error[E0063]: missing field `date_of_birth` in initializer of `Person`
  --> src\lib.rs:11:16
   |
11 |     let john = Person {
   |                ^^^^^^ missing `date_of_birth`

For more information about this error, try `rustc --explain E0063`.
error: could not compile `invalid-state` due to previous error
```

这里，它建议从类型构造函数中缺失一个字段的名称。

```
error: Ferris cannot be used as an identifier
 --> src\lib.rs:6:9
  |
6 |     let 🦀 = 0;
  |         ^^ help: try using their name instead: `ferris`

error: could not compile `invalid-state` due to previous error
```

这里，它建议修改一个无效的标识符（顺便说一句：这可能是我最喜欢的 `rustc` 错误消息之一）。

Rust 编译器很 *智能*，与语言强大的类型系统结合，你将拥有一个实用的工具。

## 合法 vs. 有效状态

类型告诉编译器如何表示数据。这些类型可以来自标准库、第三方库、语言的基本类型，甚至是你自己编写的类型。

类型界定了 *合法可表示状态* 的集合 $\mathcal{R}$ ^(在你的应用中。)

接着，有你的业务逻辑。业务逻辑利用符合你的类型定义的可表示状态来操作这些数据，并生成一些输出。你的业务逻辑可以处理的数据组成了有效状态集合 $\mathcal{V}$（即“可处理”状态），而且关键的是，*有效状态集合未必等于可表示状态集合*。

$$\mathcal{R} \supseteq \mathcal{V}$$

实际上，$\mathcal{R}$ 往往比 $\mathcal{V}$ *显著* 大，即代码可以处理的状态远少于实际可表示的状态。

这两组之间的区别在于无效状态的集合：即程序能够表示但不知如何正确处理的数据。这就是 bug 发生的地方。

为了减少 bug，我们需要尽量缩小这个差距，为此，我们可以选择：

1.  增加代码处理的案例数量，或者，

1.  减少可表示状态的数量。

增加代码处理的案例数量往往会增加其复杂性，复杂的代码如果不小心处理则更容易出 bug。因此，在这篇文章中，我们讨论的是后一种策略：减少可表示状态的数量。

另一种思考方式是，我们尽可能将尽可能多的错误从运行时转移到编译时。

## 示例：颜色

让我们看一个简单的例子：

```
fn accepts_color(color: &str) {
    // ...
}
```

这里，我们有一个接受颜色作为输入的函数。目前，它接受一个字符串。让我们看看这对我们的函数有什么影响：

```
accepts_color("#000000");
accepts_color("rgba(255, 255, 255, 0)");
accepts_color("purple");
accepts_color("sapphire");
accepts_color("5");
accepts_color("白");
accepts_color("Call me Ishmael.");
accepts_color("");
```

现在，可表示的状态集合大于有效状态集合，这意味着我们的函数将必须包含一些解析逻辑和可能一些错误处理。如果函数通过返回`Result`（Rust）或抛出异常（Java等）传播错误，则调用代码也需要执行一些错误处理。

让我们改为引入一个简单的`Color`数据结构，它有两个变体：用于RGB和RGBA颜色，并且我们将更新我们的函数以接受此类型的参数而不是字符串：

```
enum Color {
    Rgb(u8, u8, u8),
    Rgba(u8, u8, u8, u8),
}

fn accepts_color(color: Color) {
    // ...
}
```

现在让我们来看看可表示的状态：

```
accepts_color(Color::Rgb(0, 0, 0));
accepts_color(Color::Rgba(255, 255, 255, 0));
```

结果表明，所有可表示的状态也都是有效状态！这意味着我们的集合$\mathcal{R}$和$\mathcal{V}$是相等的，并且不需要运行时错误处理。

在我继续之前，让我们退后一步，评估一下我们如何从这样编码中获益：

1.  首先，它帮助我们在代码库中分离关注点。在您可以在业务逻辑中使用用户输入之前，它必须被验证并解析为内部数据结构，编译器强制执行此要求。

1.  然后，只要数据存在，解析和验证步骤所提供的保证就会持续存在。这提供了更深入和更早的保证。

1.  如果我们将函数的可能输入限制在某些范围内，例如，这意味着函数主体中的逻辑只需涵盖更少的情况。

1.  如果您的类型反映您的业务逻辑，对这些类型的更改将导致编译时错误，直到相应更新您的代码。这是件好事。

1.  最后，在具有多个贡献者的较大代码库中，良好结构化的数据类型更易于理解和使用。在类型良好的数据中错误使用的可能性更小（尽管不是不可能）。

## 示例：模态文本编辑器

让我们看一个稍微复杂一些的例子吧。

想象一下你正在编写一个类似Vim的文本编辑器。在Vim中，你可以执行不同的文本编辑操作，以及一些其他功能，比如录制宏。宏是一系列可以保存和多次执行的操作组合。

在我们的示例编辑器中，我们将添加一个约束：不允许记录的宏内部保存或记录宏。

（请注意，实际的Vim确实*支持*“递归”宏操作，但这只是一个激发想象的例子，所以就顺着这个想法走吧。）

如果我们要为操作实现一个类型，可能会看起来像这样：

```
pub struct Motion(/* ... */);
pub struct Register(/* ... */);

pub struct SaveMacro {
    register: Register,
    actions: Vec<Action>,
}

pub enum Action {
    Move(Motion),
    Delete(Motion),
    Insert(String),
    SaveMacro(SaveMacro),
    RunMacro(Register),
}
```

操作由枚举表示。保存宏要求我们指定要保存宏的寄存器，以及要保存的动作列表。`Motion`和`Register`结构体是无关的，所以我省略了任何定义。

然而，这组类型不强制执行非递归约束。`SaveMacro::actions`不应*允许*包含`Action::SaveMacro`或`Action::RunMacro`，但这些类型声明允许这样做。

没什么大不了的，我们仍然可以强制执行这一点，没有问题。我们只需将`SaveMacro`的字段设为私有，并提供一个强制执行约束的构造函数：

```
#[derive(Debug)]
pub enum SaveMacroError {
    IllegalSave,
    IllegalRun,
}

impl SaveMacro {
    pub fn new(register: Register, actions: Vec<Action>) -> Result<Self, SaveMacroError> {
        for action in actions.iter() {
            match action {
                Action::SaveMacro(..) => return Err(SaveMacroError::IllegalSave),
                Action::RunMacro(..) => return Err(SaveMacroError::IllegalRun),
                _ => {}
            }
        }

        Ok(Self { register, actions })
    }
}
```

情况并不太糟。我们有一点代码，一个简单且自解释的错误类型，而且我们的构造函数运行得很好。

那么问题出在哪里呢？

那么，每当我们想要使用构造函数时，调用者必须进行一些错误处理。

…这并不是世界末日。错误处理是工作描述的一部分。

```
let save_macro = SaveMacro::new(
    Register(/* ... */),
    vec![
        Action::Insert(String::from("code")),
        Action::Move(Motion(/* ... */)),
        // Action::RunMacro(Register(/* ... */)),
    ],
)
.unwrap();
```

这里我使用了`unwrap`（如果遇到任何错误将停止程序的函数）。

如果我们取消注释掉的那行代码会发生什么？

在编译时，什么也没有！不幸的是，这段代码产生了一个运行时错误。我们的验证代码只在*运行*时运行，因此编译器无法告诉我们出了什么问题。

这不一定是件坏事：运行时错误很正常。有时是不可避免的。然而，在这种情况下，我们可以做得更好。

## 缩小可表示的状态空间

让我们再次看看我们的类型声明，并进行一些重构，以缩小可表示的状态集合，并使其更接近有效状态集合。

首先，重写我们的`Action`和`SaveMacro`类型。

```
pub enum Action {
    Edit(EditAction),
    SaveMacro(SaveMacro),
    RunMacro(Register),
}

pub enum EditAction {
    Move(Motion),
    Delete(Motion),
    Insert(String),
}

pub struct SaveMacro {
    pub register: Register,
    pub actions: Vec<EditAction>,
}
```

你会注意到我已将宏定义的合法操作提取到它们自己的枚举中。

还要注意，在这个版本中，没有构造函数，也没有用于构造函数错误的类型，并且`SaveMacro`的字段可以是公共的。

这是构造一个与前一个版本示例相同功能的`SaveMacro`操作的重写方式。

```
let save_macro = SaveMacro {
    register: Register(/* ... */),
    actions: vec![
        EditAction::Insert(String::from("code")),
        EditAction::Move(Motion(/* ... */)),
        // Action::RunMacro(Register(/* ... */)),
    ],
};
```

然而，这一次，如果我们取消注释掉的那行代码，我们会在编译时得到一个错误！

```
error[E0308]: mismatched types
  --> src\compiletime_validation.rs:32:9
   |
32 |         Action::RunMacro(Register(/* ... */)),
   |         ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^ expected enum `compiletime_validation::EditAction`, found enum `compiletime_validation::Action`

For more information about this error, try `rustc --explain E0308`.
error: could not compile `invalid-state` due to previous error
```

完美！这正是我们想要的。

我们以更接近业务逻辑的方式定义了我们的类型系统，作为回报，Rust 编译器能够在编译时自动验证更多的代码。

比较这一版本与之前的版本，我们需要自己编写验证代码，即使如此，我们仅在运行时才能得到错误。

## 状态机和状态转换

让我们通过实现一个状态机再看一个例子。我们将利用类型系统的能力来在编译时强制执行有效的状态转换。

我们将使用一个简单的 VPN 示例，它有三种状态：`Disconnected`（断开连接）、`Connecting`（连接中）和`Connected`（已连接），以及四种状态转换：

+   `Disconnected` → `Connecting`（断开连接 → 连接中）

+   `Connecting` → `Connected`（连接成功时）

+   `Connecting` → `Disconnected`（连接失败时）

+   `Connected` → `Disconnected`（已连接 → 断开连接）

这是一种建模数据结构的方式：

```
struct Disconnected;
struct Connecting;
struct Connected;

struct Vpn<S> {
    state: S,
}
```

请注意，我们没有将不同的状态放在一个枚举中，而是用其他“`Vpn`状态”结构参数化了`Vpn`结构体。将来，所有状态结构体可能会实现一个共同的特性，但现在这已经足够了。

由于结构体中的状态字段是私有的，我们可以通过在该类型上仅实现一个`new`函数来轻松强制所有`Vpn`的初始状态为`Disconnected`。

```
impl Vpn<Disconnected> {
    pub fn new() -> Self {
        Vpn {
            state: Disconnected,
        }
    }
}

let vpn: Vpn<Disconnected> = Vpn::new(); // ok
let vpn: Vpn<Connected> = Vpn::new(); // compile-time error
```

从这里开始，实现每个状态转换就变得非常简单了。

```
impl From<Vpn<Disconnected>> for Vpn<Connecting> {
    fn from(_value: Vpn<Disconnected>) -> Self {
        Vpn { state: Connecting }
    }
}

impl From<Vpn<Connected>> for Vpn<Disconnected> {
    fn from(_value: Vpn<Connected>) -> Self {
        Vpn { state: Disconnected }
    }
}
```

然而，`Connecting`状态有两种可能性：成功时可以转换为`Connected`，或者失败时转换为`Disconnected`。

我们可以通过`TryFrom`实现来建模这个问题。

```
impl TryFrom<Vpn<Connecting>> for Vpn<Connected> {
    type Error = Vpn<Disconnected>;

    fn try_from(_value: Vpn<Connecting>) -> Result<Self, Self::Error> {
        if can_connect() {
            Ok(Vpn { state: Connected })
        } else {
            Err(Vpn {
                state: Disconnected,
            })
        }
    }
}
```

* * *

我并不希望说服你认为你的类型集只有在不能表示无效状态时才是好的，即 $\mathcal{R} = \mathcal{V}$。然而，我希望展示，对数据结构设计稍加思考*可能*有助于你在开发早期*避免*更多*的*错误。

* * *

如果我没有提到在这个主题上在我之前的工作，那我会感到遗漏。

* * *

我是[NEAR Protocol](https://near.org/)的软件工程师，也是[Tokyo Institute of Technology](https://www.titech.ac.jp/)的研究生。

关注我的[Twitter](https://twitter.com/sudo_build)和[Mastodon](https://infosec.exchange/@hatchet)。
