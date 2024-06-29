<!--yml

category: 未分类

date: 2024-05-27 13:17:45

-->

# Rust所需的调用约定 · mcyoung

> 来源：[https://mcyoung.xyz/2024/04/17/calling-convention/](https://mcyoung.xyz/2024/04/17/calling-convention/)

我经常说所谓的“C ABI”是一个非常糟糕的ABI，而在传递复杂类型时相对缺乏想象力。很多人问我：“好的，你会用什么来替代”，我只是指向了[Go寄存器ABI](https://go.googlesource.com/go/+/refs/heads/dev.regabi/src/cmd/compile/internal-abi.md)，但大多数人很难填补我所说的空白。本文详细解释了我的意思。

我曾经讨论过[调用约定](https://mcyoung.xyz//2021/11/09/assembly-1/#the-calling-convention)，但作为提醒：*调用约定*是ABI的一部分，关注如何传递函数的参数以及如何实际调用函数。这包括参数放置在哪些寄存器中，哪些寄存器中返回值，函数序言/尾声的样式，异常展开的工作方式等。

此特定帖子主要讨论x86，但我打算比较通用（使我写的内容同样适用于ARM、RISC-V等）。我假设读者对x86汇编、LLVM IR和Rust有一般的了解（但不了解rustc的内部）。

今天，与许多其他本地编译语言一样，Rust 定义了一个未指定的调用约定，允许它根据喜好调用函数。实际上，Rust 会转换为LLVM内置的C调用约定，LLVM的序言/尾声代码生成会为其生成调用。

Rust相当保守：它试图生成LLVM函数签名，这些签名Clang也可以合理生成。这有两个显著的好处：

1.  良好的概率调试器不会对其产生影响。不过，在Linux上这不是问题，因为DWARF非常通用，且不内置Linux C ABI。我们只关注基于ELF的系统，并假设调试性不是问题。

1.  由于使用了Clang不涵盖的ABI代码生成，因此不太可能触发LLVM的错误。我认为，如果Rust触发LLVM的错误，我们应该实际上去修复它们（实际上只有极少数rustc贡献者这样做）。

然而，我们太保守了。我们得到简单函数的糟糕代码生成：

```
fn extract(arr: [i32; 3]) -> i32 {
  arr[1]
}
```

```
extract:
  mov   eax, dword ptr [rdi + 4]
  ret
```

`arr` 宽度为12字节，所以你可能会认为它会通过寄存器传递，但事实并非如此！它是通过指针传递的！当请求`extern "C"`时，Rust实际上比Linux C ABI规定更为保守，因为它实际上会在寄存器中传递 `[i32; 3]` 。

```
extern "C" fn extract(arr: [i32; 3]) -> i32 {
  arr[1]
}
```

```
extract:
  mov   rax, rdi
  shr   rax, 32
  ret
```

数组通过 `rdi` 和 `rsi` 传递，其中 `i32` 被打包到寄存器中。该函数将 `rdi` 移入 `rax`，作为输出寄存器，并将上半部分向下移动。

Clang 不仅生成明显*糟糕*的代码来通过值传递东西，而且如果你请求标准调用约定，它也知道如何更好地做到这一点！我们可以生成比 Clang 更好得多的代码，但我们没有！

从此处开始，我将描述如何做到这一点。

假设我们保留`extern "Rust"`的当前调用约定^(，但在编译 crate 时添加一个`-Zcallconv`标志来设置`extern "Rust"`的调用约定。支持的值将是`-Zcallconv=legacy`用于当前的调用约定，以及`-Zcallconv=fast`用于我们将要设计的调用约定。我们甚至可以让`-O`自动设置`-Zcallconv=fast`)。

为什么保留旧的调用约定？尽管我没有详细讨论调试性能，但`-Zcallconv=fast`不会有的一个好特性是它不会按照C ABI顺序放置参数，这意味着在x86上依赖“戴安娜的丝绸裙子花了89美元”这样的助记符的读者将会感到相当困惑。

我还假设我们甚至可能不支持对一些目标（如 WASM）启用`-Zcallconv=fast`，在那里没有“寄存器”和“溢出”的概念。在调试构建中甚至可能不值得启用它，因为在关闭优化时它会生成更糟糕的代码。

函数指针也有一个小问题，以及`extern "Rust" {}`块。因为这个标志是每个 crate 的，即使函数可以宣布它们使用哪个版本的`extern "Rust"`，函数指针却没有这种奢侈。但通过函数指针调用是缓慢和罕见的，所以我们可以简单地强制它们使用`-Zcallconv=legacy`。我们可以生成一个 shim 来根据需要转换调用约定。

类似地，原则上我们可以这样调用任何 Rust 函数：

```
fn secret_call() -> i32 {
  extern "Rust" {
    fn my_func() -> i32;
  }
  unsafe { my_func() }
}
```

然而，这种机制只能用于调用未混淆的符号。因此，我们可以简单地强制`#[no_mangle]`符号使用传统的调用约定。

在理想的情况下，LLVM可以提供一种直接指定调用约定的方法。例如，这个参数放在那个寄存器中，这个返回值放在那个寄存器中等等。不幸的是，向LLVM添加调用约定需要编写大量的C++代码。

然而，我们可以通过以下过程设定自己的调用约定，来摆脱这个问题。

1.  首先，确定给定目标三元组的“通过寄存器”传递的最大值。我将在下面解释如何做到这一点。

1.  决定如何传递返回值。它要么适合于输出寄存器中，要么需要通过“引用”返回，这种情况下我们向函数传递一个额外的`ptr`参数（带有`sret`属性的标记），函数的实际返回值是该指针。

1.  决定哪些通过值传递的参数需要降级为通过引用传递。这将是一个启发式方法，但通常大约是“大于通过寄存器空间的参数”。例如，在x86上，这大约是176字节。

1.  决定哪些参数通过寄存器传递，以最大化寄存器空间的使用。这个问题是NP-hard（它是背包问题），因此需要启发式算法。所有其他参数都通过堆栈传递。

1.  在LLVM IR中生成函数签名。这将是作为各种非聚合体传递的所有参数，例如`i64`、`ptr`、`double`和`<2 x i64>`。对于这些非聚合体的有效选择取决于目标，但在64位架构上通常会得到上述内容。在“寄存器输入”后，将传递到堆栈上的参数。

1.  生成函数序言。这是从寄存器输入解码每个Rust级别参数的代码，使得在使用`-Zcallconv=legacy`时存在与其对应的`%ssa`值。这使我们能够生成函数主体的相同代码，无论调用约定如何。冗余的解码代码将被DCE passes消除。

1.  生成函数退出块。这是一个包含单个`phi`指令的块，用于返回类型，就像在`-Zcallconv=legacy`时那样。该块将其编码为所需的输出格式，然后根据需要`ret`。所有函数的退出路径都应该`br`到这个块，而不是直接`ret`。

1.  如果一个非多态、非内联函数可能会被取地址（作为函数指针），要么因为它被导出到创建外，要么因为创建获取了指向它的函数指针，则生成一个使用`-Zcallconv=legacy`并立即尾调用真实实现的适配器。这是为了保留函数指针的相等性是必要的。

主要影响在于我们需要制定启发式算法来确定哪些内容应放入寄存器（因为我们允许重新排序参数以提高吞吐量）。这相当于背包问题；背包启发式算法超出了本文的范围。这应尽早发生，以便将此信息塞入`rmeta`中，避免需要重新计算。我们可能希望根据`-Copt-level`使用不同的、更快的启发式算法。请注意，正确性要求我们禁止链接由多个不同Rust编译器生成的代码，这已经是事实，因为Rust从发布到发布会改变ABI。

假设我们这样做了，我们如何确保LLVM实际上按我们希望的方式传递内容？我们需要确定LLVM允许的最大“按寄存器”传递是什么。以下是在特定版本的LLVM上确定这一点的有用LLVM程序：

```
%InputI = type [6 x i64]
%InputF = type [0 x double]
%InputV = type [8 x <2 x i64>]

%OutputI = type [3 x i64]
%OutputF = type [0 x double]
%OutputV = type [4 x <2 x i64>]

define void @inputs({ %InputI, %InputF, %InputV }) {
  %p = alloca [4096 x i8]
  store volatile { %InputI, %InputF, %InputV } %0, ptr %p
  ret void
}

%Output = { %OutputI, %OutputF, %OutputV }
@gOutput = constant %Output zeroinitializer
define %Output @outputs() {
  %1 = load %Output, ptr @gOutput
  ret %Output %1
}
```

当你将一个聚合体按值传递给LLVM函数时，LLVM会尝试将该聚合体“展开”为尽可能多的寄存器。不同系统上有不同的寄存器类。例如，在x86和ARM上，浮点数和向量共享同一寄存器类（某种意义上）。

上述值适用于x86^(。LLVM将通过寄存器传递六个整数和八个SSE矢量，并通过寄存器返回一半（3个和4个）。增加任何值都会生成额外的加载和存储，这表明LLVM放弃并在堆栈上传递参数。)

对于`aarch64-unknown-linux`，输入和输出分别为8个整数和8个矢量。

这是我们每个类别可以使用的寄存器的最大数量。任何额外的东西都会传递到堆栈上。

我建议*每个函数*都具有相同数量的按寄存器传递的参数。因此，在x86上，每个`-Zcallconv=fast`函数的签名应该如下所示：

```
declare {[3 x i64], [4 x <2 x i64>]} @my_func(
  i64 %rdi, i64 %rsi, i64 %rdx, i64 %rcx, i64 %r8, i64 %r9,
  <2 x i64> %xmm0, <2 x i64> %xmm1, <2 x i64> %xmm2, <2 x i64> %xmm3,
  <2 x i64> %xmm4, <2 x i64> %xmm5, <2 x i64> %xmm6, <2 x i64> %xmm7,
  ; other args...
)
```

在传递指针时，适当的`i64`应该用`ptr`替换，而传递`double`时，它们应该替换为`<2 x i64>`。

但是你可能会说，“米格尔，这太疯狂了！大多数函数的字节数不超过176字节！”如果不是LLVM非常规定的`poison`语义的魔力的话，你是正确的。

如果我们不使用的每个参数都传递`poison`，我们可以摆脱做额外工作。因为`poison`等于“当前时刻最方便的可能值”，当LLVM看到通过寄存器传递的`poison`时，它决定最方便的值是“寄存器中已经存在的任何值”，因此它不必触碰该寄存器！

例如，如果我们想通过`rcx`传递指针，我们将生成以下代码。

```
; This is a -Zcallconv=fast-style function.
%Out = type {[3 x i64], [4 x <2 x i64>]}
define %Out @load_rcx(
  i64 %rdi, i64 %rsi, i64 %rdx,
  ptr %rcx, i64 %r8, i64 %r9,
  <2 x i64> %xmm0, <2 x i64> %xmm1,
  <2 x i64> %xmm2, <2 x i64> %xmm3,
  <2 x i64> %xmm4, <2 x i64> %xmm5,
  <2 x i64> %xmm6, <2 x i64> %xmm7
) {
  %load = load i64, ptr %rcx
  %out = insertvalue %Out poison,
                      i64 %load, 0, 0
  ret %Out %out
}

declare ptr @malloc(i64)
define i64 @make_the_call() {
  %1 = call ptr @malloc(i64 8)
  store i64 42, ptr %1
  %2 = call %Out @by_rcx(
    i64 poison, i64 poison, i64 poison,
    ptr %1,     i64 poison, i64 poison,
    <2 x i64> poison, <2 x i64> poison,
    <2 x i64> poison, <2 x i64> poison,
    <2 x i64> poison, <2 x i64> poison,
    <2 x i64> poison, <2 x i64> poison)
  %3 = extractvalue %Out %2, 0, 0
  %4 = add i64 %3, 42
  ret i64 %4
}
```

```
by_rcx:
  mov   rax, qword ptr [rcx]
  ret

make_the_call:
  push  rax
  mov   edi, 8
  call  malloc
  mov   qword ptr [rax], 42
  mov   rcx, rax
  call  load_rcx
  add   rax, 42
  pop   rcx
  ret
```

如果函数不与毒害参数以任何规定的方式交互，则将毒害传递给函数是完全合法的。正如我们所看到的，`load_rcx()`在`rcx`中接收其指针参数，而`make_the_call()`在设置调用时不承担任何惩罚：将毒害加载到其他十三个寄存器中编译成无事可做^(，因此它只需要加载malloc返回的指针到`rcx`中。)

这几乎给了我们对参数传递的完全控制；不幸的是，它并非完全控制。在理想的世界中，应该使用相同的寄存器作为输入和输出，以便更轻松地对调用进行流水线处理而不引入额外的寄存器流量。这在ARM和RISC-V上是正确的，但在x86上不是。然而，因为寄存器排序对我们来说仅仅是一个建议，我们可以选择以任何顺序分配返回寄存器。例如，我们可以假设输入寄存器的分配顺序为`rdx`、`rcx`、`rdi`、`rsi`、`r8`、`r9`，输出寄存器的分配顺序为`rdx`、`rcx`、`rax`。

```
%Out = type {[3 x i64], [4 x <2 x i64>]}
define %Out @square(
  i64 %rdi, i64 %rsi, i64 %rdx,
  ptr %rcx, i64 %r8, i64 %r9,
  <2 x i64> %xmm0, <2 x i64> %xmm1,
  <2 x i64> %xmm2, <2 x i64> %xmm3,
  <2 x i64> %xmm4, <2 x i64> %xmm5,
  <2 x i64> %xmm6, <2 x i64> %xmm7
) {
  %sq = mul i64 %rdx, %rdx
  %out = insertvalue %Out poison,
                      i64 %sq, 0, 1
  ret %Out %out
}

define i64 @make_the_call(i64) {
  %2 = call %Out @square(
    i64 poison, i64 poison, i64 %0,
    i64 poison, i64 poison, i64 poison,
    <2 x i64> poison, <2 x i64> poison,
    <2 x i64> poison, <2 x i64> poison,
    <2 x i64> poison, <2 x i64> poison,
    <2 x i64> poison, <2 x i64> poison)
  %3 = extractvalue %Out %2, 0, 1

  %4 = call %Out @square(
    i64 poison, i64 poison, i64 %3,
    i64 poison, i64 poison, i64 poison,
    <2 x i64> poison, <2 x i64> poison,
    <2 x i64> poison, <2 x i64> poison,
    <2 x i64> poison, <2 x i64> poison,
    <2 x i64> poison, <2 x i64> poison)
  %5 = extractvalue %Out %4, 0, 1

  ret i64 %5
}
```

```
square:
  imul rdx, rdx
  ret

make_the_call:
  push rax
  mov rdx, rdi
  call square
  call square
  mov rax, rdx
  pop rcx
  ret
```

`square`生成非常简单的代码：输入和输出寄存器都是`rdi`，因此不需要额外的寄存器流量。同样地，当我们有效地执行`@square(@square(%0))`时，函数之间没有设置。这类似于在aarch64上看到的代码，它使用相同的寄存器顺序作为输入和输出。由于这个原因，这个IR的“朴素”版本在aarch64上产生了完全相同的代码。

```
define i64 @square(i64) {
  %2 = mul i64 %0, %0
  ret i64 %2
}

define i64 @make_the_call(i64) {
  %2 = call i64 @square(i64 %0)
  %3 = call i64 @square(i64 %2)
  ret i64 %3
}
```

```
square:
  mul x0, x0, x0
  ret

make_the_call:
  str x30, [sp, #-16]!
  bl square
  ldr x30, [sp], #16
  b square  // Tail call.
```

现在我们已经确立了如何分配寄存器的完全控制权，我们可以转向在Rust中最大化使用这些寄存器。

简单起见，我们可以假设rustc已经将用户的类型处理成基本的聚合和联合；这里没有枚举！然后我们必须就将参数的哪些部分分配给寄存器做出一些决策。

首先是返回值。这相对直接，因为只有一个值需要传递。我们需要返回的数据量*不是*结构体的大小。例如，`[(u64, u32); 2]`宽度为32字节。然而，这八个字节是填充！我们在返回时不需要保留填充，因此我们可以将结构体展平为`(u64, u32, u64, u32)`，并按大小排序为`(u64, u64, u32, u32)`。这没有填充，并且宽度为24字节，适合于x86上LLVM给出的三个返回寄存器。我们将一个类型的*有效大小*定义为它占用的非`undef`位数。对于`[(u64, u32); 2]`，这是192位，因为它排除了填充。对于`bool`，这是一位。对于`char`，这在技术上是21位，但将`char`视为`u32`的别名更简单。

计算位数的原因是为了能够显著压缩。例如，将一个充满布尔值的结构体简单地位压缩为一个单一寄存器中的位。

因此，如果返回值的有效大小小于输出寄存器空间（在x86上，这是三个整数寄存器和四个SSE寄存器，因此我们得到88字节总共，或者704位），则将返回值转换为引用返回。

参数寄存器则更为困难，因为我们遇到了[背包问题](https://zh.wikipedia.org/wiki/背包问题)，这是一个NP难题。以下是一个相对天真的启发式方法，我会从这里开始，但随着时间的推移，可以无限地增强智能化程度。

首先，将任何有效大小大于寄存器总输入空间（在x86上为176字节或1408位）的参数降级为引用。这意味着我们得到一个指针参数。这样做是有益的，因为一个单独的指针可能比庞大的结构体更好地打包。

枚举应该被适当的辨别联合对替换。例如，`Option<i32>`在内部是`(union { i32, () }, i1)`，而`Option<Option<i32>>`是`(union { i32, (), () }, i2)`。使用一个小的非二的整数可以提升我们打包的能力，因为枚举辨别者通常非常微小。

接下来，我们需要处理联合。因为在我们背后弄乱联合的未初始化位是允许的，所以我们需要将它作为一个`u8`数组传递，除非它只有一个非空变体，此时它将被替换为该变体^。

现在，我们可以继续将所有内容展平。所有转换后的参数都展平为它们最原始的组件：指针、整数、浮点数和布尔值。每个字段都不应大于最小的参数寄存器；这可能需要分割大型类型，如`u128`或`f64`。

这个大列表的基本类型按有效大小从小到大排序。我们取这个列表中能够适应可用寄存器空间的最大前缀；其余的放在堆栈上。

如果 Rust 级别的输入的一部分以这种方式发送到堆栈，并且该部分大于指针大小的小倍数（例如，2x），则将其降级为通过堆栈上的指针传递，以最小化内存流量。其他所有输入按照排序前的顺序直接传递到堆栈上。这有助于保持需要复制的区域相对连续，以最小化对 `memcpy` 的调用。

我们选择在寄存器中传递的内容是按照逆大小顺序分配到寄存器中的，例如，首先是 64 位内容，然后是 32 位内容等。这与 `repr(Rust)` 结构体使用的布局算法相同，用于将所有填充移动到尾部。一旦到达 `bool` 类型，这些内容是按位压缩，每个寄存器中有 64 个。

这是一个相对复杂的示例。我的 Rust 函数如下：

```
struct Options {
  colorize: bool,
  verbose_debug: bool,
  allow_spurious_failure: bool,
  retries: u32,
}

trait Context {
  fn check(&self, n: usize, colorize: bool);
}

fn do_thing<'a>(op_count: Option<usize>, context: &dyn Context,
                name: &'a str, code: [char; 6],
                options: Options,
) -> &'a str {
  if let Some(op_count) = op_count {
    context.check(op_count, options.colorize);
  }

  for c in code {
    if let Some((_, suf)) = name.split_once(c) {
      return suf;
    }
  }

  "idk"
}
```

此函数的代码生成非常复杂，因此我只会涵盖开头和结尾。在排序和展开之后，我们的原始参数 LLVM 类型大致如下：

```
gprs: i64, ptr, ptr, ptr, i64, i32, i32
xmm0: i32, i32, i32, i32
xmm1: i32, i1, i1, i1, i1
```

一切都适合寄存器！那么，在 x86 上，LLVM 函数看起来是什么样子呢？

```
%Out = type {[3 x i64], [4 x <2 x i64>]}
define %Out @do_thing(
  i64 %rdi, ptr %rsi, ptr %rdx,
  ptr %rcx, i64 %r8, i64 %r9,
  <4 x i32> %xmm0, <4 x i32> %xmm1,
  ; Unused.
  <2 x i64> %xmm2, <2 x i64> %xmm3,
  <2 x i64> %xmm4, <2 x i64> %xmm5,
  <2 x i64> %xmm6, <2 x i64> %xmm7
) {
  ; First, unpack all the primitives.
  %r9.0 = trunc i64 %r9 to i32
  %r9.1.i64 = lshr i64 %r9, 32
  %r9.1 = trunc i64 %r9.1.i64 to i32
  %xmm0.0 = extractelement <4 x i32> %xmm0, i32 0
  %xmm0.1 = extractelement <4 x i32> %xmm0, i32 1
  %xmm0.2 = extractelement <4 x i32> %xmm0, i32 2
  %xmm0.3 = extractelement <4 x i32> %xmm0, i32 3
  %xmm1.0 = extractelement <4 x i32> %xmm1, i32 0
  %xmm1.1 = extractelement <4 x i32> %xmm1, i32 1
  %xmm1.1.0 = trunc i32 %xmm1.1 to i1
  %xmm1.1.1.i32 = lshr i32 %xmm1.1, 1
  %xmm1.1.1 = trunc i32 %xmm1.1.1.i32 to i1
  %xmm1.1.2.i32 = lshr i32 %xmm1.1, 2
  %xmm1.1.2 = trunc i32 %xmm1.1.2.i32 to i1
  %xmm1.1.3.i32 = lshr i32 %xmm1.1, 3
  %xmm1.1.3 = trunc i32 %xmm1.1.3.i32 to i1

  ; Next, reassemble them into concrete values as needed.
  %op_count.0 = insertvalue { i64, i1 } poison, i64 %rdi, 0
  %op_count = insertvalue { i64, i1 } %op_count.0, i1 %xmm1.1.0, 1
  %context.0 = insertvalue { ptr, ptr } poison, ptr %rsi, 0
  %context = insertvalue { ptr, ptr } %context.0, ptr %rdx, 1
  %name.0 = insertvalue { ptr, i64 } poison, ptr %rcx, 0
  %name = insertvalue { ptr, i64 } %name.0, i64 %r8, 1
  %code.0 = insertvalue [6 x i32] poison, i32 %r9.0, 0
  %code.1 = insertvalue [6 x i32] %code.0, i32 %r9.1, 1
  %code.2 = insertvalue [6 x i32] %code.1, i32 %xmm0.0, 2
  %code.3 = insertvalue [6 x i32] %code.2, i32 %xmm0.1, 3
  %code.4 = insertvalue [6 x i32] %code.3, i32 %xmm0.2, 4
  %code = insertvalue [6 x i32] %code.4, i32 %xmm0.3, 5
  %options.0 = insertvalue { i32, i1, i1, i1 } poison, i32 %xmm1.0, 0
  %options.1 = insertvalue { i32, i1, i1, i1 } %options.0, i1 %xmm1.1.1, 1
  %options.2 = insertvalue { i32, i1, i1, i1 } %options.1, i1 %xmm1.1.2, 2
  %options = insertvalue { i32, i1, i1, i1 } %options.2, i1 %xmm1.1.3, 3

  ; Codegen as usual.
  ; ...
}
```

上面，`!dbg` 元数据应附加到实际材料化其参数的指令上。这确保在要求打印参数值时，gdb 会做一些聪明的事情。

另一方面，在当前的 rustc 中，它给 LLVM 提供了八个指针大小的参数，因此它最终会消耗掉所有六个整数寄存器，再加上两个在堆栈上传递的值。效果不太理想！

这并不是完全超工程的调用约定描述：在某些情况下，我们可能知道我们有额外的寄存器可用（例如 x86 上的 AVX 寄存器）。有时我们可能希望将结构拆分到寄存器和堆栈之间。

这还没有涉及返回值可能看起来像什么。`Result` 经常通过 `?` 传递到几层函数中，这可能导致大量冗余的寄存器移动。通常，`Result` 大到足以不适合寄存器，因此 `?` 堆栈中的每个调用都必须通过从内存加载的 ok 位来检查。相反，`Result` 返回可能作为错误的 out-parameter 指针实现，其 ok 变体的有效载荷及其 ok 位返回为 `Option<T>`。通过 `Into` 调用的细节有些麻烦，但这个想法是可实现的。

现在，因为我们是 Rust，我们也有一个 C 没有（但 Go 有）的技巧！当我们生成所有调用者将看到的 ABI（对 `-Zcallconv=fast`）时，我们可以查看函数体。这意味着一个 crate 可以宣布其函数的 *精确* ABI（按寄存器传递）。

这为基于极端优化的 ABI 打开了大门。我们可以从简单地丢弃未使用的参数开始：如果函数对参数没有任何操作，就不要浪费寄存器。

另一个例子：假设我们知道 `&T` 参数不会被保留（这个问题借用检查器在编译器的这一点可以回答），且永远不会转换为原始指针（或者写入一个取原始指针的内存等）。我们还知道 `T` 相当小，并且 `T: Freeze`。那么，我们可以用值传递直接替换引用的指针。

最明显的候选者是像 `HashMap::get()` 这样的 API。如果键是像 `i32` 这样的东西，我们需要将该整数溢出到堆栈并传递指针！这导致了不必要的、可以避免的内存流量。

基于配置文件的 ABI 是更进一步的步骤。我们可能会知道某些参数比其他参数更热门，这可能会导致它们在寄存器分配顺序中优先考虑。

甚至可以想象一种情况，一个函数通过引用接收一个非常大的结构体，但是三个 `i64` 字段非常热门，所以调用者可以通过寄存器和大结构体的指针同时传递这些字段。调用者不会看到额外的成本：它必须发出这些加载指令。然而，调用者可能已经在寄存器中保存了这些值，这样可以避免一些内存流量。

仪表化配置甚至可能表明，复制整个函数是有意义的，这些函数除了 ABI 外完全相同。也许它们通过寄存器接收不同的参数，以避免昂贵的溢出。

这比我通常的写作要更加高级（和抱怨），但这是我发现 Rust 中令人沮丧的一个方面。我们可以比 C++ 做得更好（因为它们的 ABI 约束）。这些不是新的想法；这 *确实* 是 Go 是如何做到的！

那么我们为什么不这样做呢？部分原因是 ABI 代码生成很复杂，正如我上面描述的那样，LLVM 给我们提供了很少有用的控制选项。这对于 rustc 并不友好，做错事情可能会对可用性产生不好的后果。另一部分原因是缺乏专业知识。截至目前，只有少数几个人在为 rustc 做贡献时具备了足够理解 LLVM 语义（和情绪波动）的能力，以便发出正确的代码，使我们获得良好的代码生成且不会崩溃 LLVM。

另一个原因是编译时间。函数签名越复杂，我们需要生成的序言/尾声代码就越多，LLVM 需要处理的就越多。但是 `-Zcallconv` 旨在仅在打开优化时使用，所以我认为这不是一个有意义的抱怨。我也不认为将项目的编译时间视为度量标准是健康的……但我认为这最终不是一个相关的缺点。

很不幸，我没有空闲时间来深入修复 rustc 的 ABI 代码，但我对 LLVM 很熟悉，我知道这是 Rust 的一个低总线因子的地方。因此，我很高兴为 Rust 编译器团队提供专业知识，帮助 LLVM 做正确的事情，以提升优化代码的速度。
