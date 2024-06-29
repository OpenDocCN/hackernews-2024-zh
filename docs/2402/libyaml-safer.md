<!--yml

category: 未分类

date: 2024-05-27 14:41:56

-->

# libyaml-safer

> 来源：[https://simonask.github.io/libyaml-safer/](https://simonask.github.io/libyaml-safer/)

<main>Simon Ask Ulsnes, 2024-02-08

* * *

# 将 libyaml 移植到安全的 Rust 中：一些想法

下面是在将 [unsafe-libyaml](https://github.com/dtolnay/unsafe-libyaml) 移植为安全、符合 Rust 语法的过程中，我想到的一些松散的想法，这些想法可能对考虑“用 Rust 重写它”的其他人有用，正如古老的行业守旧者祈祷的方式。

*背景信息：* [YAML](https://yaml.org/) 是一种常用的标记语言，类似于 JSON，Rust 程序员主要通过 [serde_yaml](https://crates.io/crates/serde_yaml) 与之交互，这是一个针对 YAML 的序列化/反序列化实现，基于广受欢迎的 Serde 框架，其背后几乎是对 C [libyaml](https://pyyaml.org/wiki/LibYAML) 库的逐字机器翻译，翻译成了 Rust 中的不安全代码，称为 [unsafe-libyaml](https://github.com/dtolnay/unsafe-libyaml)。这一翻译是通过 [c2rust](https://c2rust.com/) 实现的，仅手动应用了少量修正。

在我的情况下，我面临的情况是需要以一种 [serde_yaml](https://crates.io/crates/serde_yaml) 无法满足的方式与 YAML 交互。具体来说，我需要在游戏的上下文中支持诊断和调试，除了错误消息外还需要输入位置标记，因为在该游戏中，AI 行为树是用 YAML 文件定义的。

但实际上，即使将不安全的 libyaml API 翻译成 Rust 代码后，它在 Rust 中的使用起来仍然非常笨拙，因为它的行为与 C 代码完全一样，所以与直接调用实际的 C 库并无二致。因此，我——或许是年轻时充满狂热乐观的一时冲动——决定逐个函数、逐行将 [unsafe-libyaml](https://github.com/dtolnay/unsafe-libyaml) 移植为安全的 Rust。整个过程花了一周多一点的时间。

[GitHub 上的结果可用](https://github.com/simonask/libyaml-safer)。

**更新 2024-02-09：** 现在已经将 libyaml-safer [发布到 crates.io](https://crates.io/crates/libyaml-safer)。

## 我们正在处理什么？

libyaml 主要由两个组件组成：

+   YAML 解析器负责消耗字节流并产生表示文档结构的“事件”。

+   YAML 发射器负责消耗事件流并产生字节流。

字节流（包括输入和输出）可以采用不同的编码方式，其中包括 UTF-8、UTF-16 小端和 UTF-16 大端。大多数 Unicode 代码点都被支持，但并非全部，特别是不支持控制字符。

libyaml 中的事件是文档逻辑结构的线性表示，比解析器标记高级但比映射、序列和标量值低级。事件包括“开始映射”，“开始映射键”，“标量”等。

Libyaml 还包含一个高级抽象，`yaml_document_t`，它可以表示任何 YAML 文档。解析器可以将事件流加载到文档中，而发射器可以从文档中生成事件流。

## 厄尔德里奇恐怖，或者只是 C

[libyaml](https://pyyaml.org/wiki/LibYAML) 明确是一个 C 库，从 Rust 程序员的角度来看，阅读它时会觉得这一切都非常美妙而疯狂。它来自一个更简单的时代，像所有权、可变性和现实的本质这样的概念都在争论之中。

它并不是“错误”的，只是不同。在许多方面不必要地困难，但在其他方面，只要你能够将宇宙恐怖尽量地藏起来，它就是令人耳目一新的直接方式。

### 弦理论

YAML 是一种人类可读的格式，任何解析器都必须处理大量的字符串。幸运的是，libyaml 做了理智的选择，并定义了自己的字符串类型 - 几乎每一个现存的 C 库都这么做了，甚至许多更大的 C++ 库也是如此 - 有时它们甚至并不包含很多错误。我发誓，如果我们尊敬的祖先在标准 C 库中包含了一个理智的字符串类型，世界历史可能会看起来非常不同，但我又岔开了话题。

这就是在 [libyaml](https://pyyaml.org/wiki/LibYAML) 中的字符串抽象：

```
struct yaml_string_t {
  char* start;  char* end;  char* pointer; // <-- ??? }; 
```

`start` 和 `end` 字段似乎很直接：一个指向字符串开头的指针，一个指向字符串结束后的一个位置。到目前为止一切顺利。但是`pointer`呢？在代码中的某些地方，它被用作字符串的实际结束（因此`end`实际上是字符串分配容量的末尾），在其他地方，它被用作字符串的实际开始，因此`start`是底层分配的开始，而`pointer`用于迭代字符串的字节或子字符串。

这里到底发生了什么？

结果表明，这种数据结构同时服务于多个不同的目的：它既是`&str`、`String`，也是`std::slice::Iter`，通通包含其中。有时甚至是`std::str::Chars`迭代器，用于在字符串中迭代完整的 UTF-8 代码点。唯一确定它当前模拟的抽象的方法是查看上下文。

幸运的是 - libyaml 是一个结构良好的 C 项目 - 有大量的上下文线索。当使用 `STRING_DEL()` 宏时，调用者假设拥有所有权。当使用 `STRING_ASSIGN()` 宏时，字符串被借用和/或用作迭代器。

有趣的结论是：对于 `yaml_string_t` 承担的所有不同角色，事实证明 Rust 标准库中有一个精确匹配每个用例的抽象。在 libyaml 源代码中使用字符串的所有点上，用例直接映射到 Rust 中处理字符串的*几种*方法之一。咻！

*快乐的旁白：* 在 libyaml 中有一个类似的类型，称为`yaml_buffer_t`。它比字符串稍微复杂一些，因为它本质上充当环形缓冲区，它几乎非常接近标准库中的`VecDeque`。万岁！

## 自引用类型 - 哦！

YAML 语法可以用多种方式表示相同的内容。例如，序列和映射可以用“流式样式”和“块样式”表示，字符串可以是单引号、双引号、无引号、块引号等。

```
flow_style_mapping: { a: "Hello", b: "World" }, block_style_mapping:
  a: Hello
  b: World flow_style_sequence: [1, 2, 3] block_style_sequence: - 1 - 2 - 3 
```

libyaml 发射器希望为输出选择既好看又正确的格式。这不仅仅是为了美观，还因为某些东西确实需要特定的样式 - 例如，转义序列仅支持双引号字符串，而某些 Unicode 字符需要表示为转义序列。

发射器通过将“事件”推送到队列来工作。事件包括诸如“开始映射”、“开始映射键”、“开始映射值”、“结束映射”等，发射器在具备足够信息以确定适当样式时写入输出。

这在 libyaml 中实现为每个事件在写入到输出之前进行的“分析”步骤。

```
 | 1700 | static int  |
| 1701 | yaml_emitter_analyze_event(yaml_emitter_t *emitter,  |
| 1702 |  yaml_event_t *event)  |
| 1703 | {  |
| 1704 |  emitter->anchor_data.anchor = NULL;  |
| 1705 |  emitter->anchor_data.anchor_length = 0;  |
| 1706 |  emitter->tag_data.handle = NULL;  |
| 1707 |  emitter->tag_data.handle_length = 0;  |
| 1708 |  emitter->tag_data.suffix = NULL;  |
| 1709 |  emitter->tag_data.suffix_length = 0;  |
| 1710 |  emitter->scalar_data.value = NULL;  |
| 1711 |  emitter->scalar_data.length = 0;  |
| 1712 |    |
| 1713 |   switch (event->type)  |
| 1714 |  {  |
| 1715 |   // ...  |
| 1716 |  }  |
| 1717 |   // ...  |
| 1718 | }  | 
```

分析步骤根据事件类型填充`emitter->anchor_data`、`emitter->tag_data`和`emitter->scalar_data`字段。那些可以为`NULL`的字段是 C 字符串，并且它们引用事件中包含的字符串。但是事件来自由`yaml_emitter_t`拥有的队列。这是 C 库中的很好的设计，因为它使得分析步骤非常廉价 - 没有字符串被复制，只是“借用”。

那么我们如何在安全的 Rust 中表示分析字段呢？盲目地将 C 的方法移植到 Rust 中将导致`yaml_emitter_t`变得自引用，因为分析指针将指向事件队列或刚从队列中弹出的事件，这使得在`yaml_emitter_t`本身上表示那个生命周期是不可能的。

**关键洞见：** 每次调用`yaml_emitter_analyze_event()`时，它都会重置任何先前的分析结果。每个事件只调用一次。啊哈！如果分析的结果是从`yaml_emitter_analyze_event()`的返回值，而不是修改发射器上的字段呢？

```
#[derive(Default)] struct Analysis<'a> {
  pub anchor: Option<AnchorAnalysis<'a>>,
  pub tag: Option<TagAnalysis<'a>>,
  pub scalar: Option<ScalarAnalysis<'a>>, }   struct AnchorAnalysis<'a> {
  pub anchor: &'a str,
  pub alias: bool, }   struct TagAnalysis<'a> {
  pub handle: &'a str,
  pub suffix: &'a str, }   struct ScalarAnalysis<'a> {
  /// The scalar value.
  pub value: &'a str,
  /// Does the scalar contain line breaks?
  pub multiline: bool,
  /// Can the scalar be expessed in the flow plain style?
  pub flow_plain_allowed: bool,
  /// Can the scalar be expressed in the block plain style?
  pub block_plain_allowed: bool,
  /// Can the scalar be expressed in the single quoted style?
  pub single_quoted_allowed: bool,
  /// Can the scalar be expressed in the literal or folded styles?
  pub block_allowed: bool,
  /// The output style.
  pub style: ScalarStyle, }   fn yaml_emitter_analyze_event<'a>(
  emitter: &yaml_emitter_t,
  event: &'a yaml_event_t, ) -> Analysis<'a> {
  match event.type_ {  // ...
 } } 
```

精彩！现在只需将`Analysis<'a>`结构体传递给任何需要额外参数作为分析的发射器函数即可。值得庆幸的是，发射器上没有公共 API 函数依赖于先前已进行的分析，因此通过发射器代码传递该参数是相当简单的，并且由于生命周期注解的缘故，保证没有 bug，即使在发射另一个事件期间使用一个事件的分析也是不可能的。

一方面，这种变化是我们受Rust强制执行的，因为没有这样做borrow checker是无法高兴的。另一方面，Rust编译器实际上鼓励我们选择比原始设计更好的设计。

这有什么好处呢？为什么我们要接受如此明显的暴政，由残忍无情的借用检查器所施加的？

嗯，你看，阅读`libyaml`的原始源代码是很困难的。当看到`yaml_emitter_t`的`anchor_data`、`tag_data`和`scalar_data`字段时，任何阅读者或维护者都必须弄清楚这些指针如何使用以及它们何时有效。这需要进行大量的取证工作，而且每当有新的人看代码时都需要再次进行这些操作，而“新的人”也包括“未来的你”。

C代码的原始设计可以逐字移植到Rust中。但这将需要大量的不安全代码，这将使人明显感受到认知负担。通过选择在安全的Rust中运行的设计，这种认知负担已经被转移到静态验证的不变量上，这在源代码中是显而易见的。`yaml_emitter_analyze_event`的新签名清楚地说明`Analysis<'a>`包含从`yaml_event_t`中引用的生命周期为`'a`的内容。

## 错误处理

我在libyaml-safer中的一个目标是与libyaml相比提供接近完美的错误保真度。

幸运的是，C语言没有异常处理。它们毫无疑问位列人类历史上前20大错误之中，千万别跟我辩，因为我会一死到底。

相反，C代码通常通过返回错误码来报告错误，这已经是Rust所做的功能的一个原始版本。在良好结构化的C代码和惯用Rust代码之间的错误处理流程中，有些惊人的相似之处。Rust使用“尝试”运算符`?`来简化事务，而[unsafe-libyaml](https://github.com/dtolnay/unsafe-libyaml)（Rust）相对于[libyaml](https://pyyaml.org/wiki/LibYAML)（C）的一个重要改进是，整数返回码被替换为定制的`Success`结构。将其替换为[`Result`](https://doc.rust-lang.org/stable/std/result/enum.Result.html)事实上是相当简单的。

但等等……我们不能直接使用`?`。它检查是否发生了错误，并在函数中早期返回，但C语言没有RAII，因此在返回之前还需要释放资源。这是C语言中`goto`的唯一剩下的有效用例，而且通常实际上是处理错误的最可持续的方式。更文明的语言（如Rust）甚至没有这种误用特性。

```
void* stupid_c() {
  void* some_memory = malloc(1024);
  if (some_function(some_memory) != ERROR) {  // ...
 } else {
  goto cleanup; }    if (some_other_function(some_memory) != ERROR) {  return some_memory; }   cleanup:
  // We're returning an error, so we need to free what we allocated.
  free(some_memory);
  return NULL; } 
```

因此，为了开始利用Rust的错误处理功能，所有拥有内存的事物必须首先转换为RAII对象，如`String`、`Vec`、`VecDeque`等。

之后还存在实际上报告错误的问题。

Rust的`Result`很丰富：它们可以包含您选择的错误值，因此可以为用户提供最佳可能的消息，为他们提供所有所需的上下文。C没有内置此选项，大多数库选择简单返回错误代码，然后有时允许程序员在事后获取有关错误的更多信息。

在libyaml中，看起来是这样的：

```
struct yaml_parser_t {
 yaml_error_type_t error;  const char* problem; yaml_mark_t problem_mark;  const char* context; yaml_mark_t context_mark;  // ... } 
```

当发生错误时，相关函数（例如`yaml_parser_parse()`）设置错误字段（`yaml_set_parser_error()`），并返回指示发生错误的整数。然后期望用户访问这些字段以确定错误及其发生位置。

在Rust中，我们不希望解析器有一个特殊的"错误"状态。理想情况下，我们希望能够返回一个自包含的错误，然后可以具备所有通常的美好之处，例如`std::fmt::Display`的实现。

还要注意报告的错误不一定是"解析错误"。可能发生许多通常不被视为语法错误的错误：I/O读取错误，无效的Unicode，标记化错误等等。

让我们尝试一些枚举：

```
enum ParserError {
 Problem { problem: &'static str,
 problem_mark: Mark, context: &'static str,
 context_mark: Mark, } Scanner(ScannerError), }   enum ScannerError {
 Problem { problem: &'static str,
 problem_mark: Mark, context: &'static str,
 context_mark: Mark, }, Reader(ReaderError), }   enum ReaderError {
 Problem { problem: &'static str,
 offset: usize,
 value: i32,
 }, Io(std::io::Error), } 
```

很好，完美的保真度和高质量的错误报告。除了...在64位架构上，顶级`ParserError`枚举的大小为88字节，主要是由于`Mark`类型，它表示输入中的位置，看起来像这样：

```
struct Mark {
  pub line: u64,
  pub column: u64,
  pub index: u64, } 
```

24字节，每个错误通常需要2个标记。

为什么大小会成为问题？

1.  即使是最小的`Result<(), ParserError>`也永远无法在寄存器中返回。

1.  每个遇到`Result<T, ParserError>`且想返回`Result<U, ParserError>`的函数都需要重新包装返回值，无论是否发生错误，导致在栈内存中复制大型结构体的大量操作。

1.  虽然LLVM在128字节以下会内联调用`memcpy()`（https://github.com/rust-lang/rust/pull/64302#issuecomment-529840404），但仍会生成复制这些值的指令。

解析器产生大量微小调用，每个调用都可能失败。每次它需要一个新的标记时，可能也需要一个新的字符，这意味着可能还需要从输入缓冲区中获得另一个字节，这可能会导致从输入执行`read()`操作失败，或者出现Unicode解码错误，或者标记化错误，最终是解析器错误。

总而言之，像这样一个非常大的错误类型...并不好。

有几种前进的选择：

1.  模仿libyaml风格的错误，并在解析器对象本身上设置错误代码。这会导致API不太符合人体工程学，或者需要在所有返回值上引入一个不太好的生命周期参数。

1.  对上述枚举进行Codegolf以减小大小。没有太多可以赢得的地方而不失真，但可能可以减少到大约64字节左右。

1.  将错误包装在一个`Box`中，在64位平台上仅占用8字节。堆分配不是最佳选择，但仅在错误场景下发生，因此在错误路径上牺牲了一些效率，以换取更快的“正常”路径。

1.  接受这就是我们现在的生活，继续前进。

所有的妥协，无论是在可用性还是性能上。

我在“正常”路径上将选项（3）和（4）进行了基准测试，它们之间没有太大的区别。但这仍然*感觉不对*，承诺向用户公开API内部工作原理的“低效”公共API是一个semver的风险，因此目前我选择了一个支持两种实现的API，并在底层使用了装箱（3）。

放大来看，值得注意的是，[`serde_yaml::Error`](https://docs.rs/serde_yaml/latest/src/serde_yaml/error.rs.html#12)实际上选择了（3），一个装箱错误，这表明我并不是唯一一个认为这是可以接受的折衷方案的人。

## 免费赠品

Rust作为一种年轻的语言，拥有更丰富的标准库。这意味着在libyaml中手动实现的几件事情可以用标准设施来替代。

+   UTF-8和UTF-16编码/解码。这些并不特别复杂，但Rust标准库可能会更好地优化（例如，在可用时使用SIMD）。

+   而不是用户提供的不安全回调，输入和输出可以由`std::io::Read`和`std::io::Write`特性处理。这使用户能够在其中插入任何内容，例如tty流、网络流等，而无需额外的努力。

+   解析器需要一些输入缓冲区来执行正确的解码，但是libyaml几乎被迫自己执行这一操作，即使输入已经被用户缓冲。通过使用`std::io::BufRead`特性，解析器的整个中间缓冲区被消除，这意味着如果数据已经来自内存缓冲区，则可以避免不必要的复制。

## 测试

`unsafe-libyaml`自动针对[官方YAML测试套件](https://github.com/yaml/yaml-test-suite)进行测试。在转换为安全的Rust时，拥有这个全面的测试套件是绝对必要的，没有它我不敢着手进行这项工作。

对于代码的每一次更改，我都能运行完整的测试套件，并验证一切是否如预期那样。尽管代码是专注于匹配原始语义甚至API的“形状”，许多事情可能会出错 - 特别是在流控制和错误处理方面。看，使用复杂的循环/匹配/中断结构，c2rust模拟了`goto`的使用情况，所有这些结构都完全消失了，被Rust的“try”运算符`?`所替代。即使是微小的更改也可能会导致错误的机会，特别是当更改数量很多时。

尽管在安全的Rust中编写正确的代码比在C中要容易得多，但逻辑错误显然同样可能发生，特别是在进行非常大规模的重构时。

## 性能

为了比较unsafe-libyaml和安全的Rust移植版的性能，我找了一个相对大的YAML文档（~700 KiB），并使用[Criterion](https://docs.rs/criterion/latest/criterion/)构建了一个非常简单的基准测试。

简而言之：

|  | unsafe-libyaml | libyaml-safer |
| --- | --- | --- |
| 解析700 KiB文档 | 38.258 毫秒 | 37.447 毫秒 |
| 发出700 KiB文档 | 18.491 毫秒 | 17.549 毫秒 |

*鼓掌停顿*

这些基准测试是在运行Windows 11的AMD Ryzen 9 3950X 16核CPU（3.5 GHz）上运行的。

### 分析

+   **重要提示：** 虽然我始终得到稍微更快的libyaml-safer时间，但我并不认为这个基准测试具有任何决定性意义。

+   这些数字*确实*表明，向安全、符合惯例的Rust移植并不会产生含义上较慢的代码。

    +   对于“不安全”的C和C++库的辩护通常是，与安全的Rust相比，一些性能被搁置了，主要是由于边界检查。在现代硬件上，这种辩护总是让我觉得有些可疑，我认为这又是（间接的）证据，实际上往往并不重要。

    +   当然可以构建一些边界检查起作用的东西。但这不是那种情况。

+   *如果* libyaml-safer对所有工作负载始终更快，这里有一些可能的解释：

    +   C libyaml需要对一些不变量（如指针不为NULL）进行断言，即使在发布模式下，unsafe-libyaml也会执行这些断言。在安全的Rust中，这些不变量是静态检查的，其中`&T`永远不会是“NULL”。

    +   由于在解析器中使用了`std::io::BufRead`，所以解析器执行更少的内存拷贝。

+   *即使*这个基准测试完全是假的，libyaml-safer的性能仍然在可接受范围内，考虑到维护性和安全性的显著提高。

+   除了不选择显然更慢的安全替代方案之外，没有做任何优化工作。

    +   一个可能有意义的优化是在解析器中实现更有效的前瞻性。当前，在输入缓冲区上，扫描器和解析器执行许多多余的边界检查。

## 结论

这真的很有趣。

对我来说，对于很多来自C++背景的软件工程师，使用Rust不仅仅是关于安全和正确性的模糊承诺。

这是为了让编程重新变得有乐趣。

我们都喜欢设计漂亮、用户友好的API，我们都喜欢制作那些感觉高质量的东西。我们很多人做不到这一点，因为我们行业的现实通常不允许。所以，这能带来快乐吗？

使用Rust编程就像是拿着蛋糕吃一样。当人们在Rust中变得高效时，这是因为它使许多曾经困难的事情——那些导致眼泪、血液和倦怠的事情——变得不再那么困难了。结果不仅更安全、更容易，而且同样高效。所以请——加入我们吧。

+   在这篇文章中，我保留了来自`unsafe-libyaml`的类型名称和函数名称，但作为迁移到Rust惯用写法的一部分，我也按照Rust惯用的命名惯例重新命名了所有类型和枚举。

</main>
