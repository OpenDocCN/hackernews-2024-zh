<!--yml

category: 未分类

date: 2024-05-27 13:14:10

-->

# Swift for C++ Practitioners, Part 1: Intro & Value Types | Doug's Compiler Corner

> 来源：[https://www.douggregor.net/posts/swift-for-cxx-practitioners-value-types/](https://www.douggregor.net/posts/swift-for-cxx-practitioners-value-types/)

# Swift for C++ Practitioners, Part 1: Intro & Value Types

有一个[入门指南](https://www.swift.org/getting-started/)适合广泛的受众。然而，我注意到从C++过来的人往往会在Swift的某些设计方面遇到困难，并且可能会陷入困境。我认为我理解其中的原因：这两种语言在感觉上相似到足以让熟悉C++的人将C++的习惯和模式投射到Swift上，但结果并不总是理想的。因此，我想以一种针对C++“从业者”的不同方式教授Swift：那些日常写C++并了解C++语言、标准库和最佳实践的人。亲爱的C++从业者，我想通过将C++的习惯、模式和心理模型映射到Swift中来教授Swift。我希望通过这一系列文章，你不仅学会Swift，还学会如何*优雅地*使用Swift。

作为一名C++程序员，Swift的某些部分会让人感到像魔术一样，例如独立类型检查的泛型和精美组合的值类型，我们会为此感到兴奋。我将展示Swift设计如何解决我们共同认为问题的C++方面，例如错误的默认设置或可避免的问题。Swift的设计方式。Swift的其他部分可能会与C++从业者的理念不合，我们也不会回避这些。相反，我们将解释不同之处，Swift为何以此设计，以及如何应对。我身兼两个世界：我是Swift的设计师、实现者和倡导者，同时我在C++方面也有着悠久的历史，包括是Clang的代码所有者并在ISO C++委员会上度过了十年。我日常大部分编写的代码是在Swift编译器中，虽然大部分是C++，但正向Swift迁移。

> *注意：* 你可能听说过[Swift与C++的互操作性](https://www.swift.org/documentation/cxx-interop/)。这是一个非常棒的工具，可以逐步将C++代码库转向Swift，或者用更好的Swift接口包装C++库。但如果你已经了解C++并想学习Swift，这不是开始的好地方。相反，我建议首先纯粹用Swift构建一些东西，以便了解Swift的感觉，而不是被现有C++代码向更C++-centric的模式拉扯。一旦你对两种语言都有了扎实的理解，就能更好地将Swift整合到现有的代码库中去。

这是一个多部分系列，将详细介绍Swift的各种特性。我们将从必备的“Hello, world”开始，然后直接深入*值类型*。

## Hello, World!

好了，让我们把这事儿过去：这是 Swift 中的 "Hello, World"：

```
print("Hello, world!") 
```

但更重要的是向你问好，亲爱的 C++ 实践者，所以让我们稍微定制一下：

```
let reader = "dear C++ practitioner"
print("""
      Hello, \(reader)!

      Today, we shall embark on learning a new programming language, Swift.
      """) 
```

`let` 关键字用于声明不可变变量，类似于 C++ 中的 `const`，但提供更强的保证：我们稍后会详细讨论。我们省略了类型，因为 Swift 使用类型推断方式类似于 C++ 中的 `auto`，不过我们也可以显式地写出类型，如 `let reader: String`。三重引号描述了[多行字符串字面量](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/stringsandcharacters#Multiline-String-Literals)，其中的 `\(...)` 语法是[字符串插值](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/stringsandcharacters#String-Interpolation)，这是一种常见的脚本语言特性，方便在字符串中插入值。

## 值类型

C++ 提供了对*值类型*的广泛支持，即每个副本都完全独立于原始对象。让我们以 C++ 向量的简单示例来说明：

```
std::vector<std::string> v1 = { "Hello", "original" };
std::vector<std::string> v2 = v1;
v2[1] = "copy"; 
```

在这里，`v1` 是一个包含 `"Hello"` 和 `"original"` 的向量。当我们复制 `v2` 时，该复制完全独立于原始对象：最后一行对 `v2` 进行的更改，使其包含 `"Hello"` 和 `"copy"`，不会修改原始的 `v1`。

许多 C++ 类型都是值类型，从内置类型如整数和浮点类型到标准库容器如 `std::string`、`std::vector` 和 `std::map`。C++ 允许你通过控制类类型的创建、复制和销毁方式来构建自己的值类型，只要你遵循[三五零法则](https://en.cppreference.com/w/cpp/language/rule_of_three)。

Swift 也强调值类型，因为它们有助于*局部推理*，即能够单独查看代码并推断它的作用及其是否正确。当你复制值类型的实例时，你不需要担心你做的任何事情会影响到原始对象。和 C++ 类似，许多 Swift 类型都是值类型，包括 `String`、`Array` 和 `Dictionary`，它们分别类似于 `std::string`、`std::vector` 和 `std::map`：

```
let v1: [String] = ["Hello", "original"] 
var v2 = v1 
v2[1] = "copy"
print(v1) 
print(v2) 
```

这里我们引入了 `var` 关键字：`var` 用于声明可修改的变量（即它们是可变的），而 `let` 用于声明不可修改的变量（它们是不可变的）。在 Swift 中，我们建议尽可能使用 `let`，因为不可变性有助于局部推理：如果某物不在变化，推理它就更容易。

### 在结构体中聚合值

与C++一样，Swift有结构体来聚合数据。而在C++中，`struct`和`class`之间的区别几乎只是装饰性的（仅影响默认是`public`还是`private`），在Swift中它们完全是不同的实体。Swift的`struct`通常是值类型，而Swift的`class`则是面向对象意义上的类，并具有*引用语义*：复制仍然引用相同的底层实例。我们稍后会再次讨论类，因为`struct`是我们从其他值类型构建值类型的一种方式：

```
struct LabeledPoint {
  var x: Double
  var y: Double
  var label: String
} 
```

包含其他值类型的结构体本身也是值类型。例如，让我们使用那个带标签的点：

```
let p1 = LabeledPoint(x: 0, y: 0, label: "origin")
var p2 = p1
p2.label = "center"

print(p1) 
print(p2) 
```

那一行首先创建了一个`LabeledPoint`的新实例，调用一个*初始化器*（这就是Swift称为构造函数的东西），从组件部分产生新值。结果在堆栈上，而不是堆上，就像你在C++中期望的那样。将`p1`的值复制到`p2`会产生一个完全独立的值，就像对应的C++代码所期望的那样。

> **带标签的参数**：在创建新的`LabeledPoint`实例时，请注意每个参数都需要一个标签，例如，`x:`、`y:`和`label:`。默认情况下，所有函数参数必须在调用站点上打上标签，这有助于传达函数将如何处理相应参数的信息，提高可读性。这在与默认参数结合使用时特别有用。当然，函数可以选择不标记某个特定参数，我们稍后会回到这一点。

### 初始化总是通过初始化器进行。

C++有几种不同的方法来初始化`struct`的实例，包括构造函数调用、初始化列表、默认初始化和复制初始化。Swift选择其中一种：调用初始化器。初始化器负责在返回之前初始化结构的所有字段（没有任何借口）。在上一节中创建`LabeledPoint`正在使用Swift为结构体自动提供的*逐成员*初始化器，该初始化器按顺序从相应的参数初始化字段。如果我们愿意，我们也可以直接编写此初始化器，就像这样：

```
struct LabeledPoint {
  var x: Double
  var y: Double
  var label: String

  init(x: Double, y: Double, label: String) {
    self.x = x
    self.y = y
    self.label = label
  }
} 
```

`init`关键字是定义初始化器的内容，并且相当于在C++中重复类名以定义构造函数，但通常不那么冗长。`self`是Swift中相当于`this`的东西，但将其视为类似于C++引用（`ClassName&`）而不是指针，因为在C++中它将是指针（`ClassName*`）。

在 `struct` 中没有初始化字段的特殊语法，就像在 C++ 中一样。相反，它只是对字段的普通赋值，并且编译器检查：(1) 你不能在赋值之前读取一个字段，以及 (2) 在引用 `self` 作为整个对象之前，所有字段都已经赋值。因此，让我们尝试一个破坏这两条规则的语义灾难初始化器：

```
 init(x: Double, y: Double, label: String) {
  self.y = self.x   
  self.x = x
  if Int.random(in: 0..<2) == 1 {
    print(self) 
  }
} 
```

在 Swift 中不会出现未初始化变量的使用，因为有一个语义保证称为 *明确初始化*：编译器会检查在所有执行路径上，在使用之前每个变量都已初始化。这同样适用于所有代码，并且有助于消除在 C++ 中困扰我们的一类错误：

```
let p: LabeledPoint
if y > 0 {
  p = LabeledPoint(x: 0, y: 0, label: "origin")
}
print(p) 
```

由于明确初始化的存在，Swift 没有像 C++ 那样的默认构造函数概念。变量 `p` 在定义的那一行 *未初始化*，就像一个具有非平凡默认构造函数的 C++ 类。相反，你要对其进行赋值，而第一次赋值就是初始化。你不能在初始化之前读取它，因此不存在由于未初始化值而导致的 *未定义行为*。

你可以编写一个不接受任何参数的初始化器，也许对于点来说这是有意义的（比如创建原点），但是 Swift 不会自动调用它：你总是需要显式调用它。让我们写出来，这样我们可以展示 Swift 中等同于 C++ 的 [委托构造函数](https://learn.microsoft.com/en-us/cpp/cpp/delegating-constructors?view=msvc-170)：

```
 init() {
  self.init(x: 0, y: 0, label: "origin")
} 
```

对 `self.init` 的调用将初始化 `self` 的所有字段的责任委托给另一个初始化器。明确初始化的规则也适用于这里：在 `self.init` 调用之前，你不能使用（或初始化）`self` 的任何字段；而在那之后，`self` 就完全初始化了。

### 我的复制构造函数在哪里？

到目前为止，你可能已经注意到，我们可以编写一个看起来非常像复制构造函数的初始化器：

```
 init(_ other: LabeledPoint) {
  self.x = other.x
  self.y = other.y
  self.label = other.label
} 
```

`other` 声明中的 `_` 是一个“未命名”的占位符，这里表示该初始化器的参数未命名。因此，我们可以使用 `LabeledPoint(other)` 这样的语法来调用这个初始化器，就像在 C++ 中一样。然而，Swift 永远不会 *隐式* 调用这样的初始化器，因为它没有任何特殊之处。

Swift 将通过在 `struct` 的每个实例属性上直接执行这些操作来复制、移动和销毁 `struct` 的实例。实质上，Swift 的 `struct` 总是遵循 C++ 的 [zero rule](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rc-zero)，其中依赖于所有特殊构造函数、赋值运算符和析构函数的默认实现。实际上，这意味着不可能观察到 Swift 何时正在复制值类型的实例，因此编译器可以自由地进行必要的复制以实现程序的语义，并在不再需要时优化掉复制。当编译器确定复制源即将消失时，它甚至可以隐式地将“复制”转换为移动。

对于需要保留资源的类型，Swift 有类，我已经注意到将是另一篇文章的主题。Swift 也有“不可复制”类型的概念，但这些将在稍后讨论。现在，我们将更深入地探讨值类型。

### 不可变性在值类型中深入运行。

早些时候，我们介绍了 `let` 作为创建不可变局部变量的一种方式。对于值类型，不可变性是默认的。除非另有规定，否则参数是不可变的，这也包括 `self`：

```
 func badSwapX(_ other: LabeledPoint) {
  let tmpX = other.x
  other.x = self.x  
  self.x = tmpX     
} 
```

通过在 `func` 关键字之前放置 `mutating` 修改器，可以创建可以突变 `self` 的方法。让我们从一个合理的例子开始：

```
 mutating func reflectOverXAxis() {
  y = -y
} 
```

对于其他参数，可以在参数上放置 `inout` 来指示它是由函数修改的意图。形式上，函数上的 `mutating` 等同于 `self` 参数上的 `inout`，但 Swift 使用 `mutating` 因为它读起来更好。通过 `inout` 参数和 `mutating` 方法，我们可以编写 `badSwapX` 方法的工作版本：

```
 mutating func swapX(_ other: inout LabeledPoint) {
  let tmpX = other.x
  other.x = self.x
  self.x = tmpX
} 
```

在调用具有 `inout` 参数的函数时，必须在参数前加上 `&`，以表明我们正在将其（逻辑）地址传递给函数。例如，这里是对 `swapX` 的调用：

```
var p1 = LabeledPoint(x: 0, y: 0, label: "Origin")
var p2 = LabeledPoint(x: 1, y: 1, label: "Upper right unit")
p1.swapX(&p2) 
```

如果我们试图将 `&` 应用于的值是不可变的，Swift 编译器会产生一个错误。请注意，即使在调用`可变`方法时，`p1` 上没有前缀 `&`：因为方法的名称应该[清楚地暗示突变](https://www.swift.org/documentation/api-design-guidelines/)。当然，如果 `p1` 是不可变的，那么仍然会出现错误。

在我们转向下一个值类型 `enum` 之前，还有两件关于不可变性的重要事情要说。

首先，你*无法欺骗不可变性*，就像你无法欺骗死亡一样。 Swift 中没有类似于 C++ 的 `const_cast`。 Swift 中没有可变数据成员；即使是结构体的 `var` 成员，也只能在该结构体的 `var` 实例上进行修改。 没有 `const T&` 参数可以在你的底层值改变时发生变化：不可变值确实是不可变的，并且编译器确保无论如何传递不可变参数（按值传递还是按引用传递），底层值都不会改变。 这可能令人沮丧，因为你失去了控制何时按值传递、按 `const` 引用传递或按 rvalue 引用传递来进行移动。 另一方面，这是解放的：当 `const&` 在你的底层实际上*发生*变化时，没有什么神秘的远程行动，你可以依靠不可变性来更轻松地推理你的代码。

这带我们到第二点：*输入参数没有别名*。 这里所说的别名是指两个不同的按引用传递的参数实际上引用了相同的底层实例。 如果你曾经不得不在 C++ 的复制或移动赋值运算符中添加 `if (this == &other) { ... }` 检查，你就知道参数意外别名对程序语义的影响有多严重。 在 Swift 中，我们有[排他性法则](https://github.com/apple/swift/blob/main/docs/OwnershipManifesto.md#the-law-of-exclusivity)，防止任何这种别名现象发生。

### 内存安全和排他性法则

Swift 的*排他性法则*规定，只有同时访问给定内存中的值的两个访问是读取时，才能同时进行，因此尝试在访问该值时形成一个突变访问（例如传递某些东西给 `inout`）是错误的。 但这不是一些抽象的规则，当你搞砸时会引入未定义的行为：Swift 通过静态检查（如果产生别名会产生编译器错误）和动态检查来强制执行排他性法则。

当被访问的值足够局部时，静态检查排他性法则就会应用。 值类型非常适合此操作，因为值类型的两个独立的 `var` 实例保证不会发生别名。 编译器可以正确确定调用 `p1.swapX(&p2)` 中的两个 `inout` 参数不会别名，因为 `p1` 和 `p2` 是独立的变量。 如果改为 `p1.swapX(&p1)`，编译器将产生描述问题的错误：

```
=== exclusivity.swift:28 ===
   ┆
26 │   var p1 = LabeledPoint(x: 0, y: 0, label: "Origin")
27 │   var p2 = LabeledPoint(x: 1, y: 1, label: "Upper right unit")
28 │   p1.swapX(&p1)
   │   │        ╰─ note: conflicting access is here
   │   ╰─ error: overlapping accesses to 'p1', but modification requires exclusive access; consider copying to a local variable
29 │ 
```

现在，如果我们处理的变量不是局部的——比如说它是一个全局变量（咻！）或者是一个类的成员变量，那么无法推理所有访问。 让我们构建一个小的假设性例子来说明这一点：

```
var globalOrigin = LabeledPoint(x: 0, y: 0, label: "origin")

func swapXWithGlobalOrigin(_ other: inout LabeledPoint) {
  other.swapX(&globalOrigin) 
}

func somewhereElse() {
  swapXWithGlobalOrigin(&globalOrigin) 
} 
```

在 `swapXWithGlobalOrigin` 内部，没有办法知道程序的其他部分是否在运行时访问 `globalOrigin`。因此，Swift 编译器会插入一个运行时检查，跟踪 `globalOrigin` 何时可能被修改，并在同时访问发生时停止程序：

```
Simultaneous accesses to 0x100e93008, but modification requires exclusive access.
Previous access (a modification) started at t`somewhereElse() + 42 (0x100e8e95a).
Current access (a modification) started at:
0    libswiftCore.dylib                 0x00007ff82b380890 swift::runtime::AccessSet::insert(swift::runtime::Access*, void*, void*, swift::ExclusivityFlags) + 444
1    libswiftCore.dylib                 0x00007ff82b380ae0 swift_beginAccess + 66
2    t                                  0x0000000100e8e8d0 swapXWithGlobalOrigin(_:) + 59
3    t                                  0x0000000100e8e930 somewhereElse() + 51
4    t                                  0x0000000100e8ea40 static Main.main() + 9
5    t                                  0x0000000100e8ea50 static Main.$main() + 9
6    t                                  0x0000000100e8ea70 main + 9
7    dyld                               0x00007ff81aaa9c10 start + 1942
Fatal access conflict detected. 
```

大多数 Swift 程序员从不考虑互斥性法则：它的执行是为了防止在 C++ 中引起未定义行为的错误，因此它对 Swift 的内存安全性至关重要。但是 Swift 中大部分的变异都作用于局部值，并且语言帮助你避免大部分这类情况，因此实际上很少遇到运行时检查。

### 枚举是枚举和联合体的结合

枚举是 Swift 最可爱的小功能之一。我们从 [CLU](https://en.wikipedia.org/wiki/CLU_(programming_language)) 借鉴了它们，在 Swift 1.0 之前甚至使用了 `oneof` 关键字。Swift 的枚举是一种类型安全的变体，它包含了 C++ 的 `enum`、`union` 和 `std::variant`，集成在一个小巧的包中。枚举可以表示一组命名的情况，比如通过语义名称表示的字体大小：

```
enum FontSize {
  case title
  case paragraph
  case footnote
} 
```

这个枚举的工作方式与你期望的等效的 C++ `enum class` 一样。例如：

```
let fontSize: FontSize = .paragraph

switch fontSize {
  case .title: print("Title")
  case .paragraph: print("Paragraph")
  case .footnote: print("Footnote")
} 
```

我悄悄地在这里加入了一个 `switch` 语句，因为通常使用 `switch` 语句处理枚举中的每个情况。在 Swift 中，`switch` 语句必须是全面的：如果你没有处理所有可能的情况，你需要添加一个 `default` 子句。这消除了遗漏的意外，比如当有人添加一个新情况时，这在 C++ 编译器中通常是一个警告。如果你一直在为上面缺少的 `break` 语句而苦恼，那么不用担心：Swift 在下一个情况之前会插入一个 `break`，如果你真的想执行下一个情况，你必须显式地写 `fallthrough`。

回到 `FontSize`：`FontSize` 的情况包含在类型内部。如果你想引用 `paragraph` 情况，可以使用 `FontSize.paragraph`。然而，每当有类型信息时，比如在初始化 `FontSize` 类型的变量或者在类型为 `FontSize` 的值上进行切换时，你可以使用*前导点语法*，比如 `.paragraph`，让 Swift 的类型推断确定类型。将上述与相应的 C++ `enum class` 进行比较：

```
enum class FontSize {
  title,
  paragraph,
  footnote
};

auto fontSize = FontSize::paragraph;
switch (fontSize) {
  case FontSize::title: print("Title"); break;
  case FontSize::paragraph: print("Paragraph"); break;
  case FontSize::footnote: print("Footnote"); break;
} 
```

这些小细节确实积累成为更清晰的代码。Swift 的前导点语法与带标签的参数非常好地配合，因为参数标签暗示了参数的类型，从而产生非常可读的代码。例如，让我们想象一个 `Font` 结构体，它使用 `FontSize`、`FontStyle` 和 `FontWeight` 枚举，包括一些默认值：

```
struct Font {
  var style: FontStyle = .sanSerif
  var size: FontSize = .paragraph
  var weight: FontWeight = .regular
} 
```

现在，我们可以这样创建一个新的 `Font`：

```
let font = Font(size: .title, weight: .bold) 
```

注意参数标签 `size` 和 `weight` 如何自然地描述后面的参数，并且类型提供了足够的信息，所以我们不需要在这些参数上写出冗余的 `FontSize` 和 `FontWeight` 类型。此外，即使 `style` 是第一个参数，我们也能使用默认参数：带标签的参数让默认参数工作得非常好。这些对于语言来说是简单的设计决策，但它们加强了可读性的代码。

好了，回到枚举！想象一下，你在 C++ 中把那个 `FontSize` 写成了一个 `enum class`。记住它。它很简单，很有效。现在，有人告诉你需要支持*自定义*的字体大小，可以用点数表示。你美好的 `enum class` 瞬间消失了，因为你不能枚举所有自定义的点数大小。在 C++ 中，我会使用以下模式：

```
class FontSize {
public:
  enum Kind {
    title,
    paragraph,
    footnote,
    custom
  };

private:
  Kind kind;
  int points; 

public:
  FontSize() : kind(paragraph) { }
  FontSize(Kind kind) : kind(kind) { assert(kind != custom); }

  static FontSize forCustom() { 
    FontSize size;
    size.kind = custom;
    size.points = points;
    return size;
  }

  explicit operator Kind() const { return kind; } 

  int getPoints() const {
    assert(kind == custom);
    return points;
  }
}; 
```

这就是*很多*代码。它实现了一种类型安全的联合体，包含了三种简单的 case，再加上一个 `custom` case。每次我在 C++ 中写这种代码时（这种情况经常发生），我都会感到不安，因为这需要大量样板代码，而且很容易犯愚蠢的错误。我还没有找到一种能使这个过程更清晰的 C++ 技术或库。如果你有一个更优雅的解决方案，请随时告诉我。

在 Swift 中，你可以向 `FontSize` 添加一个 case：

```
case custom(points: Int) 
```

Swift 的 case 可以在其中携带值，这就是为什么我之前说它们也像 C++ 中的联合体---但没有所有的未定义行为。考虑到这种情况，我可以创建一个自定义的字体大小：

```
let customFont: FontSize = .custom(points: 32) 
```

并相应地扩展我的 `switch` 语句：

```
switch fontSize {
  case .title: print("Title")
  case .paragraph: print("Paragraph")
  case .footnote: print("Footnote")
  case .custom(let pt): print("\(pt) points")
} 
```

Swift 提供了模式匹配。当我们匹配 `custom` case 时，我们还声明一个新变量 `pt` 来捕获 `points` 的值。变量 `pt` 只在使用自定义字体大小时可用，因此不需要像在 C++ 中那样运行时断言 `kind == custom`。

让我们再为我们的字体大小添加一个 case，这样我们就可以取一个现有的字体大小并按给定的因子进行缩放。它可以这样表达：

```
indirect case scaled(size: FontSize, factor: Double) 
```

这样，就可以构造一个比段落字体大 20% 的字体，例如，

```
FontSize.scaled(size: .paragraph, factor: 1.2) 
```

`indirect` 关键字需要指示与 case 关联的值（在 Swift 中称为*关联值*）需要间接存储，因为关联值本身包含了 `FontSize` 的实例。枚举是值类型，通常使用堆栈存储，所以 `indirect` 表示当 case 的值需要移动到堆上时。如果缺少 `indirect`，编译器会报错，因为 `FontSize` 类型在内存中没有固定大小：

```
=== FontSize.swift:1 ===
 1 │ enum FontSize {
   │      ╰─ error: recursive enum 'FontSize' is not marked 'indirect'
 2 │   case title
 3 │   case paragraph
 4 │   case footnote
 5 │   case custom(points: Int)
 6 │   case scaled(size: FontSize, factor: Double)
   │        ╰─ note: cycle beginning here: (size: FontSize, factor: Double) -> (.0: FontSize)
 7 │ }
 8 │ 
```

间接枚举 case 对于构建递归数据结构（如二叉树）非常有用。我们将在能够适当地使用泛型时回到它们身上。

现在我们可以创建一个很好的成员函数，通过特定因子来缩放我们已有的字体实例：

```
 func scaled(by factor: Double) -> Self {
  .scaled(size: self, factor: factor)
} 
```

这可以称为，例如，`myFontSize.scaled(by: 1.2)`。这里有一些小细节需要注意。首先是枚举类型可以像结构体一样拥有方法。它们也可以有初始化器，必须最终将一个案例分配给`self`。接下来，我们的函数正在返回`Self`，这是“`self`的类型”的简写。最后，看看参数的命名，“`by factor`”：这里，*参数标签* 是`by`（在调用处使用），*参数名* 是`factor`（在函数体内部使用）。这是因为参数标签用于描述调用处的参数，即我们正在“按`by` 1.2缩放”，而参数名是实际参数的名词——用于计算中的`factor`。在此之前，我们通常看到这两个名称通常相同，或者省略了参数标签，但是分开这两个名称可以导致优雅、可读的代码，特别是当参数标签是一个介词时。

## 集合

标准的Swift集合类型`Array`、`Dictionary`和`Set`在存储值类型时是值类型。例如，我们可以有一个`Font`实例的数组，它将表现为值类型：

```
var fonts = [Font(size: .title, weight: .bold), Font(size: .paragraph)] 
var oldFonts = fonts  
fonts.append(Font(size: .footnote))
print(fonts.count)    
print(oldFonts.count) 
fonts[1].weight = .bold 
```

字典和集合的工作方式类似。例如，让我们构建一个列出所有字体名称的字典：

```
var fontsDict = [  
  "Title" : Font(size: .title, weight: .bold), 
  "Paragraph" : Font(size: .paragraph)
]

var oldFontsDict = fontsDict 
fontsDict["Footnote"] = Font(size: .footnote) 
print(fontDict.count)    
print(oldFontsDict.count) 
```

括号语法用于字典字面量（当元素是`key: value`对时）和数组字面量（当元素只是...元素时）。字面量实际上是可扩展的：你可以使用数组字面量初始化一个`Set`，甚至你自己的类型，通过定义一个适当的初始化器并选择通过泛型系统成为该字面量类型的“可表达”类型（我保证在以后的文章中会详细讨论）。

## 正规类型

在C++中，我们有时会谈论由亚历山大·斯捷潘诺夫（Alexander Stepanov）定义的[正规类型](http://stepanovpapers.com/DeSt98.pdf)，这些类型在值语义方面表现可预测：你可以复制它们，复制品等于原始值。它们可以被移动、销毁和交换。C++20引入了[`std::regular`概念](https://en.cppreference.com/w/cpp/concepts/regular)来捕捉这些要求。

Swift值类型默认满足大部分正规类型的要求，并且基于相同的语义契约。Swift值类型始终可复制、可销毁、可分配和可移动。实际上，在Swift中，你甚至无法真正表达这些想法，它只是类型行为的一种方式。

然而，与 Stepanov 或 C++ 标准定义的常规类型有一些显著差异。首先前面已经提到：Swift 根本没有“默认构造”这个概念，因此 Swift 值类型并不“默认可构造”。虽然由于确定性初始化，在 Swift 中你不太需要这种概念。对类型的作者来说这里有一个好处：如果某种状态没有意义，你无需担心发明一个“默认”状态。例如，想象一个始终非空的集合：如果没有做一些奇怪的事情，比如添加一个单一的默认构造元素，你怎么给它一个默认构造函数？所以这样的类型在 Swift 中不可能是正则的。

常规类型也有（不）相等运算符（`==`和`!=`）。Swift 并不免费提供这些，但你可以自己编写，例如，

```
 static func ==(lhs: Font, rhs: Font) -> Bool {
  return lhs.style == rhs.style && lhs.size == rhs.size && lhs.weight == rhs.weight
} 
```

或者让编译器为你做，通过在类型定义中加入`: Equatable`：

```
struct Font: Equatable {
  var style: FontStyle = .sanSerif
  var size: FontSize = .paragraph
  var weight: FontWeight = .regular
} 
```

这里说`Font`类型是`Equatable`，编译器会根据数据成员为你合成`==`和`!=`。你也可以在这里加上`Hashable`，以获取结合数据成员的哈希函数，这样你的类型就可以用作`Dictionary`中的键或`Set`中的值。再次说明，这是我们初步接触 Swift 泛型系统，但目前你可以把`Equatable`想象成类似于 C++ 概念（我们在 Swift 中称之为*协议*），但...更好。

## 接下来是什么？

我们已经大量讨论了值类型--- 如果你在写 Swift，你应该经常使用它们，因为它们提供了出色的局部推理能力和以直观方式建模大多数数据的能力：`struct`用于聚合数据和集合，`enum`用于捕捉不同的选择。

在本系列的下一部分中，我们将讨论引用类型。具体来说，是类和 Swift 如何支持面向对象编程。
