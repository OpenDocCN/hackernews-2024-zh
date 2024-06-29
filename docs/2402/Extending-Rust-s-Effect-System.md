<!--yml

类别：未分类

日期：2024-05-27 14:44:47

-->

# 扩展 Rust 的效果系统

> 来源：[https://blog.yoshuawuyts.com/extending-rusts-effect-system/](https://blog.yoshuawuyts.com/extending-rusts-effect-system/)

# 扩展 Rust 的效果系统

— 2024-02-09

1.  [介绍](https://blog.yoshuawuyts.com/extending-rusts-effect-system/#introduction)

1.  [没有泛型的 Rust](https://blog.yoshuawuyts.com/extending-rusts-effect-system/#rust-without-generics)

1.  [为什么要使用效果泛型？](https://blog.yoshuawuyts.com/extending-rusts-effect-system/#why-effect-generics)

1.  [第一阶段：效果泛型 trait 定义](https://blog.yoshuawuyts.com/extending-rusts-effect-system/#stage-i-effect-generic-trait-definitions)

1.  [第二阶段：效果泛型的边界、实现和类型](https://blog.yoshuawuyts.com/extending-rusts-effect-system/#stage-ii-effect-generic-bounds-impls-and-types)

1.  [什么是效果？](https://blog.yoshuawuyts.com/extending-rusts-effect-system/#what-are-effects)

1.  [第三阶段：更多效果](https://blog.yoshuawuyts.com/extending-rusts-effect-system/#stage-iii-more-effects)

1.  [结束语](https://blog.yoshuawuyts.com/extending-rusts-effect-system/#outro)

*这是我在 RustConf 2023 的演讲《扩展 Rust 的效果系统》的文字记录，演讲于 2023 年 9 月 13 日在新墨西哥州阿尔伯克基举行，并通过网络直播。*

## Introduction

自 2015 年发布版本 1.0 以来，Rust 不断发展。我们添加了诸如尝试操作符 (`?`)、常量泛型、泛型关联类型（GATs）和当然：`async/.await` 等重要功能。在这四个特性中，有三个可以被视为“效果”。尽管我们已经花了很长时间在这些特性上，但它们仍然在积极进行中。

你好，我叫 Yosh，在 Microsoft 担任 Rust 的开发者倡导者。在过去的五年里，我一直在 Rust 自身工作，此外还是 Rust Async 工作组的成员，并且是 Rust Effects Initiative 的联合负责人之一。

本次演讲的论点是，我们在 Rust 1.0 中无意中包含了一个效果系统作为语言的一部分。自那以来，我们已经开始添加许多新效果，为了将它们完全整合到语言中，我们需要支持*效果泛型*。

在这次演讲中，我将解释效果是什么，以及什么使它们难以集成到语言中，以及我们如何克服这些挑战。

## Rust Without Generics

当我刚接触 Rust 时，我花了一些时间弄清楚如何使用泛型。我习惯于编写 JavaScript，而在那里我们没有泛型。所以我发现自己大部分时间都在编写针对*具体类型*的函数。我记得我的代码感觉相当笨拙，这并不是一次很好的经历。与标准库提供的代码相比，更是如此。

一个泛型标准库函数的例子是 `io::copy` 函数。它从读取器中读取字节，并将它们复制到写入器中。我们可以给它一个文件，一个套接字，或者两者的任何组合，它都会愉快地将字节从一个复制到另一个。只要我们给它正确的类型，这一切都会正常工作。

但是如果 Rust 实际上没有泛型呢？如果我一开始使用的 Rust 实际上是我们全部拥有的呢？我们会如何编写这个 `io::copy` 函数呢？好吧，考虑到我们试图在套接字和文件类型之间复制字节，我们可能会手动编写这些函数。对于我们这里的两种类型，我们可以编写四个独特的函数。

但是对我们来说，不幸的是，这只能解决我们面前的问题。但是标准库不仅仅有两种实现读写的类型。它有 18 种实现读取的类型，以及 27 种实现写入的类型。因此，如果我们想要覆盖整个标准库的 API 空间，我们总共需要 486 个函数。如果这是我们实现 `io::copy` 的唯一方式，那将会导致一种相当糟糕的语言。

现在幸运的是，Rust 确实具有泛型，并且我们所需的只是一个 `copy` 函数。这意味着我们可以自由地继续向标准库中添加更多类型，而无需担心需要实现更多函数。我们只需要一个 `copy` 函数，编译器会为我们提供任何我们给它的类型生成正确的代码。

## 为什么要使用泛型？

在 Rust 中，我们希望泛型的不仅仅是类型。我们还有 "const 泛型"，允许函数在常量值上泛型化。以及 "值泛型"，允许函数在不同值上泛型化。这就是我们可以编写可以接受不同值的函数的方式 - 这是大多数编程语言中存在的一个特性。

```
fn by_value(cat: Cat) { .. } fn by_reference(cat: &Cat) { .. } fn by_mutable_reference(cat: &mut Cat) { .. } 
```

但是，并非导致 API 重复的所有因素都可以泛型化。例如，根据我们是以拥有的方式、引用的方式，还是可变引用的方式来获取值，创建不同方法或类型是非常常见的。我们还经常为常量值和运行时值创建重复的 API，以及根据 API 是否需要线程安全性创建重复的结构。

但是，在导致 API 重复的所有因素中，效果可能是最大的之一。当我谈论 Rust 中的效果时，我的意思是诸如 `async/.await` 和 `const` 这样的关键字；还有 `?`，以及诸如 `Result` 和 `Option` 这样的类型。所有这些都与语言有深刻的语义连接，并以其他关键字和类型所不具备的方式改变了我们代码的含义。

有时我们会写出效果不对的代码，导致*效果不匹配*。这也被称为*函数着色问题*，由罗伯特·奈斯特罗姆描述过。一旦你意识到*效果不匹配*，你会发现它们到处都有，不仅仅是在 Rust 中。

这些效果不匹配的结果是，在Rust中使用效果基本上会让您陷入二流的体验中。无论您使用const、async、Result还是Error - 几乎肯定会在某个地方遇到兼容性问题。

```
let db: Option<Database> = ..; let db = db.filter(|db| db.name? == "chashu"); 
```

以`Option::filter` API为例。它接受一个引用类型并返回一个布尔值。如果我们尝试在其中使用`?`操作符，我们会得到一个错误，因为该函数不返回`Result`或`Option`。无法在任意闭包内部使用`?`是效果不匹配的一个例子。

但是像这样的简单函数只是表面上的。标准库中几乎每个特性都存在效果不匹配的问题。以常见的`Debug`特性为例，几乎每种Rust类型都实现了它。

我们可以为我们虚构的类型`Cat`实现`Debug`特性。这里的参数`f`实现了`io::Write`并代表字节流。使用`write!`宏，我们可以将字节写入该流中。但如果由于某种原因，我们希望将字节异步地写入，比如说异步套接字。哦，我们做不到。`fn fmt`不是异步函数，这意味着我们无法在其中等待。

一种解决方法可能是创建某种中间缓冲区，并同步地将数据写入其中。然后我们可以异步地从该缓冲区中写出数据。但这会涉及到我们之前没有的额外复制。

如果我们想要与以前完全相同，解决方案就是创建一个*新的*`AsyncDebug`特性，它*可以*异步地将数据写入流中。但现在我们有了重复的特性，这正是我们试图避免的问题。

或许我们应该只是添加一个`AsyncDebug`特性然后结束。然后我们也可以添加`Read`、`Write`和`Iterator`的异步版本。也许还有`Hash`，因为它也会写入输出流。那`From`和`Into`呢？或许还有`Fn`、`FnOnce`、`FnMut`和`Drop`，因为它们是内置的？等等。事实上，效果不匹配是结构性的，为每个效果不匹配复制API界面会导致API的指数增长。这与我们之前看到的数据类型泛型类似。

让我试着举个例子来说明这一点。假设我们取现有的`Fn`特性系列，并引入它们的效果版本。也就是说：与`unsafe`、`async`、`try`、`const`和生成器一起工作的版本。一个效果就有六个独特的特性。两个效果就有十二个。所有五个效果，我们突然看到有96个不同的特性。

标准库中的问题空间非常广泛。从分析 Rust 1.70 标准库，据我估计约有 75% 的标准库将与 const 效果交互。大约 65% 将与 async 效果交互。大约 30% 将与 try 效果交互。确切的数字不精确，因为各种效果的部分仍在进行中。这在实践中将产生多大影响，很大程度上取决于我们最终如何设计语言。

如果你比较这些数字，那么接近 100% 的标准库将与一种或多种效果交互。约 50% 将与两种或更多效果交互。如果考虑到效果之间相互作用可能导致指数级增长，这应该提醒我们，聪明的一次性解决方案行不通。我认为处理这个问题的最佳方式是允许 Rust 使项目能够对效果进行通用化。

## 第一阶段：效果通用特征定义

现在我们已经仔细看了当我们无法对效果进行通用化时会发生什么，是时候开始讨论我们可以采取的措施了。不出所料的答案是将效果通用性引入语言中。为了涵盖所有用途，需要采取几个步骤，所以让我们从第一个、可能也是最重要的一步开始：效果通用特征定义。

这很重要，因为它将允许我们将效果特征引入作为标准库的一部分。这将在其他方面有所帮助，帮助标准化围绕标准库的各种异步生态系统。

```
pub trait Into<T>: Sized {     
  fn into(self) -> T; } 
```

```
impl Into<Loaf> for Cat {     
  fn into(self) -> Loaf {
  self.nap()
 } } 
```

让我们用一个简单的例子来说明：`Into` 特征。`Into` 特征用于将一种类型转换为另一种类型。它对类型 `T` 进行泛型化，并具有一个函数 "into"，它消耗 `Self` 并返回类型 `T`。假设我们有一个类型 `Cat`，当它打盹时会变成一个可爱的小面包。我们可以通过在函数体中调用 `self.nap` 来为 `Cat` 实现 `Into<Loaf>`。

```
pub trait AsyncInto<T>: Sized {     
 async fn into(self) -> T; } 
```

```
impl AsyncInto<Loaf> for Cat {     
 async fn into(self) -> Loaf {
  self.nap().await
 } } 
```

但如果猫不立即打盹怎么办？也许`nap`实际上应该是一个异步函数。为了在特征实现内部等待`nap`，`into`方法必须是异步的。如果我们从头开始编写一个异步特征，我们可以通过公开一个新的具有异步`into`方法的`AsyncInto`特征来实现这一点。

但我们不仅仅想向标准库中添加一个新特征，而是想要*扩展*现有的 `Into` 特征以支持异步效果。我们可以通过使异步效果*可选*来扩展带有 `async` 效果的 `Into` 特征。与其要求特征总是同步或异步，实现者应该能够选择要实现的特征版本。

```
#[maybe(async)] impl Into<Loaf> for Cat {     
 #[maybe(async)]
  fn into(self) -> Loaf {
  self.nap()
 } } 
```

这将通过在特征上添加一个新的符号来实现："maybe async"。我们还不知道我们想要为"maybe async"使用什么语法，所以在本次讨论中我们将使用属性。"maybe async"符号的工作原理是，我们将所有希望成为"maybe async"的方法标记为这样。然后也将我们的特征本身标记为"maybe async"。

```
impl Into<Loaf> for Cat {     
  fn into(self) -> Loaf {
  self.nap()
 } } 
```

```
impl async Into<Loaf> for Cat {     
 async fn into(self) -> Loaf {
  self.nap().await
 } } 
```

然后实现者可以选择他们想要实现的同步或异步版本的特性。根据选择的版本，方法最终变成同步或异步。这个系统完全向后兼容，因为实现 `Into` 的同步版本与今天保持相同。但是希望实现异步版本的人可以通过在实现中添加一些额外的 `async` 关键字来实现。

```
impl async Into<Loaf> for Cat {
 async fn into(self) -> Loaf {
  self.nap().await
 } } 
```

```
impl Into<Loaf, true> for Cat {
  type ReturnTy = impl Future<Output = Loaf>;
 fn into(self) -> Self::ReturnTy { async move {
  self.nap().await
 } } } 
```

在底层，实现会转换为我们今天已经能够写的普通 Rust 代码。同步实现返回类型 `T`。但是异步实现返回 `T` 的 `impl Future`。在底层，它只是一个单一的 const bool 和一些关联类型。

+   良好的诊断

+   渐进式稳定化，

+   向后兼容性

+   清晰的推断规则

为什么我们要费力去研究一种语言特性，如果它的“脱糖”变得如此简单，这是一个合理的问题。原因在于：效果无处不在，我们希望确保效果泛型感觉像语言的一部分。这不仅意味着我们希望严格控制诊断。我们还希望能够逐步引入它们，有明确的语言规则，并且向后兼容。

但是如果你记住这些，把效果泛型视为主要是对 const bools + 关联类型的语法糖可能也是可以接受的。

## 第二阶段：效果泛型约束、实现和类型

声明效果泛型特质只是一个开始。标准库不仅公开特质，还公开各种类型和函数。而效果泛型特质并不能直接帮助解决这个问题。

```
pub fn copy<R, W>(
  reader: &mut R,  writer: &mut W ) -> io::Result<()> where
 R: Read, W: Write; 
```

让我们再次看看我们之前的 `io::copy` 示例。正如我们所说，`copy` 接受一个读取器和写入器，然后将字节从读取器复制到写入器。我们已经看到了这一点。

```
pub fn async_copy<R, W>(
  reader: &mut R,  writer: &mut W ) -> io::Result<()> where
 R: AsyncRead, W: AsyncWrite; 
```

现在，如果我们尝试在标准库中添加这个的异步版本，它会是什么样子呢？好吧，我们需要从命名上给它一个不同的名称，这样它就不会与现有的 `copy` 函数冲突。对于特性约束也是一样，所以这个函数将取代 `Read` 和 `Write`，取而代之的是 `AsyncRead` 和 `AsyncWrite`。

```
pub fn async_copy<R, W>(
  reader: &mut R,  writer: &mut W ) -> io::Result<()> where
 R: async Read, W: async Write; 
```

一旦我们有了效果泛型特质定义，情况会变得稍微好一些。不再需要异步版本的 `Read` 和 `Write` 特性的副本，函数可以选择现有 `Read` 和 `Write` 特性的异步版本。这已经好了一些，但这仍意味着我们有两个版本的 `copy` 函数。

```
#[maybe(async)] pub fn copy<R, W>(
  reader: &mut R,  writer: &mut W ) -> io::Result<()> where
 R: #[maybe(async)] Read, W: #[maybe(async)] Write; 
```

相反，理想的解决方案是允许 `copy` 本身在异步效果上泛化，并使其决定我们想要哪些 `Read` 和 `Write` 的版本。这些就是我们所说的“效果泛型约束”。函数的效果和它所带的约束的效果变得一致。在文献中，这也被称为“行多态”。

```
copy(reader, writer)?; // infer sync copy(reader, writer).await?; // infer async copy::<async>(reader, writer).await?; // force async 
```

因为函数本身现在是异步效果的通用类型，我们需要在调用站点确定我们打算使用哪个变体。这个系统将利用*推断*来确定。这是一种说法高明的方式，说明编译器将对程序员意图使用的效果进行猜测。如果他们使用了`.await`，他们可能想要异步版本。否则，他们可能想要同步版本。但是就像任何猜测一样：有时我们会猜错，因此出于这个原因，我们希望提供一种逃逸通道，允许程序作者强制选择变体。我们还不知道这个的确切语法，但是我们假设这可能会使用turbofish符号。

```
struct File { .. } impl File {
  fn open<P>(p: P) -> Result<Self>
  where
 P: AsRef<Path>; } 
```

但是效果通用并不仅仅适用于函数。如果我们希望使标准库能够很好地处理效果，那么类型也将需要效果通用。这乍看起来可能有些奇怪，因为“异步类型”可能并不直观。但是例如在 Windows 上的文件需要初始化为同步或异步。这意味着它们是否是异步的不仅仅是函数的属性，而是类型的属性。

让我们以stdlib的`File`类型作为我们的示例。为简单起见，假设它只有一个方法：`open`，它返回错误或文件。

```
struct AsyncFile { .. } impl AsyncFile {
 async fn open<P>(p: P) -> Result<Self>
  where
 P: AsRef<AsyncPath>; } 
```

如果我们想提供`File`的异步版本，我们再次需要复制我们的接口。这意味着一个新类型`AsyncFile`，它有一个新的异步方法`open`，它以异步版本的`Path`作为参数。而`Path`本身需要是异步的，因为它本身在它上有异步的文件系统方法。正如我之前所说：一旦你开始寻找，你会注意到效果无处不在。

```
#[maybe(async)] struct File { .. }   #[maybe(async)] impl File {
 #[maybe(async)]
  fn open<P>(p: P) -> Result<Self>
  where
 P: AsRef<#[maybe(async)] Path>; } 
```

与其创建第二个`AsyncFile`类型，使用类型上的效果通用性，我们将能够将`File`打开为异步。这样我们可以仅保留一个`File`定义，用于同步和异步变体。

```
#[maybe(async)] fn copy<R, W>(reader: R, writer: W) -> io::Result<()> {
  let mut buf = vec![4028];
  loop {
  match reader.read(&mut buf).await? {  0 => return Ok(()),
 n => writer.write_all(&buf[0..n]).await?,
 } } } 
```

现在我有点含糊地忽略了`copy`函数和`File`类型的内部实现。它们的工作方式在两者之间有些不同。就`copy`函数而言，异步和非异步变体之间的实现将是相同的。如果函数编译为异步，一切都按原样工作。但是如果函数编译为同步，那么我们只需移除`.await`，函数应该按预期编译。

由于“可能是异步”的函数只能调用同步或其他“可能是异步”的函数。但对于大多数情况来说，这应该是可以接受的。

```
impl File {
 #[maybe(async)]
  fn open<P>(p: P) -> Result<Self> {
  if IS_ASYNC { .. } else { .. }
 } } 
```

像`File`这样的具体类型就有点棘手了。它们通常希望根据拥有的效果运行不同的代码。幸运的是，像`File`这样的类型已经根据平台有条件地编译不同的代码，因此引入新的类型条件不应该是一个很大的飞跃。我们需要的关键是在函数体内检测代码是异步编译还是非异步编译 - 基本上是一种花哨的布尔。

我们已经可以对 const 效果使用 `const_eval_select` 内部函数。它目前是不稳定且有些冗长，但它可靠地工作。我们应该能够轻松地将其类似地适应到 async 和其他效果上。

## 什么是效果？

有关效果的系统研究在计算机科学领域已经有将近 40 年的历史了。这大约与 C++ 的年龄相当。最近在编程语言领域，像 Koka、Eff 和 Frank 这样的研究语言展示了效果可以有多么有用，因此它已经成为一个热门话题。而像 Scala 和较少程度上的 Swift 则采纳了效果特性。

当人们谈论效果时，通常会广泛提到以下两种情况之一：

+   **代数效果类型**：这是对函数和上下文的语义标注，授予执行某些操作的许可。

+   **代数效果处理器**：这是一种类型化的控制流原语，允许人们定义自己版本的 `async/.await`、`try..catch` 和 `yield`。

许多具有效果的语言同时提供效果类型和效果处理器。这些可以一起使用，但实际上它们是不同的特性。在这次讲话中，我们只讨论效果类型。

```
pub async fn meow(self) {} pub const unsafe fn meow() {} 
```

在这次讲话中，我们一直称之为“效果”的东西实际上是*效果类型*。Rust 在历史上并没有这样称呼它们，我认为这可能是为什么直到最近我们才开始关注效果泛型的原因。但事实证明，将我们的某些关键词重新解释为效果类型确实是完全合理的，并为我们提供了一个强大的理论框架来进行推理。

我们还有 `unsafe`，它允许您调用 `unsafe` 函数。不稳定的 try 块特性不要求您将返回类型包装在 `Ok` 中。不稳定的生成器闭包语法使您可以访问 `yield` 关键字。当然，还有 `const` 关键字，它允许您在编译时评估代码。

```
async { async_fn().await }; // async effect unsafe { unsafe_fn() }; // unsafe effect const { const_fn() }; // const effect try { try_fn()? }; // try effect (unstable) || { yield my_type }; // generator effect (unstable) 
```

在 Rust 中，我们目前有五种不同的效果：`async`、`unsafe`、`const`、`try` 和生成器。所有这六种效果都处于不同的完成阶段。例如：async Rust 有函数和块，但没有迭代器或 drop。Const 还没有访问 trait 的权限。不安全函数不能降级为 `Fn` trait。Try 操作符 `?` 已经存在，但 try 块是不稳定的。生成器完全不稳定；我们只有 `Iterator` trait。

有些效果是语言团队开始称为“携带”的效果。这些是会被解糖成实际类型的效果。例如，当您编写 `async fn` 时，返回类型将解糖为 `impl Future`。

还有一些效果是我们称之为：“非携带”的。这些效果在类型系统中不会解糖为任何类型，而仅仅是作为一种向编译器传递信息的方式。例如 `const` 或 `unsafe`。虽然我们确实检查这些效果的正确使用，但它们最终不会降级为实际的类型。

```
let x = try async { .. }; 
```

```
1\. -> impl Future<Output = Result<T, E>> 2\. -> Result<impl Future<Output = T>, E> 
```

谈到承载的影响时，影响组合变得很重要。比如同时使用 "async" 和 "try"。如果一个函数同时具有这两者，结果类型应该是什么？一个 `Future` 的 `Result`？还是一个包含 `Future` 的 `Result`？

函数上的影响是无序的 *集合*。虽然Rust当前确实要求你按特定顺序声明影响，但影响本身只能以一种方式组合。当我们稳定了 `async/.await` 时，我们决定如果一个异步函数返回一个 `Result`，那么它应该总是返回一个 `impl Future` 的 `Result`。因为影响是 *集合* 而不依赖于顺序，我们可以在语言中定义承载影响应如何组合的方式。

人们仍然可以通过手动编写函数签名来选择退出内置的组合规则。但这很少见，对于绝大多数用途来说，内置的组合规则都是正确的选择。

```
const fn meow() {} // maybe-const const {} // always-const 
```

`const` 影响与其他影响有些不同。`const` 块总是在编译时评估。而 `const` 函数仅在编译时 *可以* 评估。在运行时调用它们也完全没问题。这意味着当我们编写 `const fn` 时，我们已经在编写影响泛化。这个机制是我们逐步能够以向后兼容的方式将 `const` 引入标准库的原因。

`const` 也有些奇怪，因为它不允许访问主机运行时，不能分配内存，也不能访问全局变量。这与像 `async` 这样的影响不同，后者只允许你做更多事情。

| 影响集 | 可访问 | 无法访问 |
| --- | --- | --- |
| std rust | 非终止，展开，非确定性，静态变量，运行时堆，主机API | N/A |
| alloc | 非终止，展开，非确定性，全局变量，运行时堆 | 主机API |
| 核心 | 非终止，展开，非确定性，全局变量 | 运行时堆，主机API |
| const | 非终止，展开 | 非确定性，全局变量，运行时堆，主机API |

这幅图缺少的是，Rust 中所有函数都隐含了一组影响。其中包括一些我们尚无法直接命名的影响。当我们编写 `const` 函数时，它们的影响集不同于我们编写 `no_std` 函数时的影响集，而后者又不同于常规的“std” Rust 函数。

正确理解 `const`、`std` 等的方式是将不同的影响添加到空影响集中。如果我们从零开始，那么所有的影响只是简单的加法。它们只是相加得到不同的结果。

不幸的是，在Rust中，我们尚无法命名空影响集。在影响理论中，这被称为“总体影响”。某些语言，如Koka，支持“总体”影响。实际上，Koka的主要开发人员估计，典型的Koka程序约70%可以是总体的。这引出了一个问题：如果我们能在Rust中表达总体影响，我们能看到类似的数字吗？

## 第三阶段：更多效果

到目前为止，我们只谈到了如何完成现有效果（如`const`和`async`）的工作。但效果泛型的一个好处是，它们不仅允许我们完成正在进行的效果工作。它还将降低引入语言中*新*效果的成本。

这引出了一个问题：如果我们能够添加更多的效果，哪些效果可能是有意义的？显而易见的是，实际上完成添加`try`和生成函数。但除此之外，还有一些有趣的效果我们可以探索。为简洁起见，我将仅讨论这些特性是什么，而不展示代码示例。

+   **no-divergence**：保证函数不能无限循环，从而能够进行静态运行时成本分析。

+   **no-panic**：保证函数永远不会产生恐慌，导致函数展开。

+   **parametricity**：保证函数仅对其参数进行操作。这意味着没有对静态变量的隐式访问，没有全局文件系统，没有线程本地变量。

+   **capability-safety**：保证函数不仅是参数化的，而且不能向下转型抽象类型。例如，如果你得到一个`impl Read`，你不能逆转它以获取一个`File`。

+   **destructor linearity**：保证`Drop`将*始终*被调用，从而成为安全保证。

+   **pattern types**：使函数能够直接操作枚举和数字的变体

+   **must-not-move types**：将是固定和固定项目系统的泛化，使其成为一流语言特性

虽然今天在Rust中添加任何这些特性都没有任何本质上的阻碍，但为了将它们集成到stdlib中而不破坏向后兼容性，我们首先需要效果泛型。

```
effect const = diverge + panic; effect core   = const + statics + non_determinism; effect alloc  = core + heap; effect std    = alloc + host_apis; 
```

这将我们带到设计空间的最后一部分：效果别名。如果我们继续添加效果，最终很容易会陷入到一个我们自己版本的“public static void main”的情况中。

为了缓解这一点，如果我们能够命名特定的效果集，那将是非常好的。在某种程度上，我们已经做到了，其中`const`代表“可能无限循环”和“可能发生紧急情况”。如果我们实际上将“可能无限循环”和“可能发生紧急情况”作为内置效果，那么我们可以将`const`重新定义为这些的别名。

从根本上讲，这并不改变我们到目前为止所讨论的任何内容。只是从语法上讲，这样做会更加愉快。因此，如果我们达到了一个状态，其中我们具有效果泛型，并且我们希望注意到我们的函数前面可能有一种过多的符号表示，那么也许是时候更认真地开始研究这个问题了。

## 结尾

Rust 已经包含了诸如 async、const、try 和 unsafe 等效应类型。因为我们还不能泛化效应类型，通常我们必须在复制代码和不解决用例之间做出选择。一旦开始使用效应，这种选择会让语言感觉非常粗糙。效应泛型为我们提供了一种可以泛化效应的方式，并且我们已经展示它们可以今天就实现，主要是作为 const 泛型的语法糖。

我们目前正在通过 A-Mir-Formality 正式化效应泛型工作。MIR Formality 是 Rust 类型系统的正在进行中的正式模型。由于效应泛型相对直接但对类型系统有深远影响，它是测试正式模型的理想候选项之一。

与此同时，const 工作组也已经开始重构编译器中 const 函数检查的方式。过去，const 检查发生在 MIR 级别的借用检查之前。在新系统中，const 检查将会更早地发生，在 HIR 级别。这不仅会使代码更易维护，如果需要，还可以泛化到更多的效应。

一旦正式建模和编译器重构完成，我们将开始起草效应泛型 trait 定义的 RFC。我们预计这将在 2024 年某个时候发生。

至此，本次讲话结束。非常感谢您始终陪伴我到最后。本次讲话中的所有工作，没有以下人员的支持是不可能的：

+   Oliver Scherer（AWS）

+   Eric Holk（微软）

+   Niko Matsakis（AWS）

+   Daan Leijen（微软）

谢谢！
