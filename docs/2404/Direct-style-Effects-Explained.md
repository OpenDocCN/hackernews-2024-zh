<!--yml

类别：未分类

日期：2024-05-27 13:35:51

-->

# 直接风格效果解释

> 来源：[https://www.inner-product.com/posts/direct-style-effects/](https://www.inner-product.com/posts/direct-style-effects/)

直接风格效果，也称为代数效果和效果处理程序，是编程语言中的下一个大事件。它们已经在[Unison](https://www.unison-lang.org/docs/fundamentals/abilities/)和[OCaml](https://github.com/ocaml-multicore/ocaml-effects-tutorial)中可用，即将进入[Scala](https://www.youtube.com/watch?v=0Fm0y4K4YO8)，我在其他[与行业密切相关的背景](https://without.boats/blog/coroutines-and-effects/)中也看到了关于它们的讨论。

目标非常简单：允许我们以自然的方式编写代码，而无需单子，但仍能享受单子带来的推理和组合的好处。同时，我看到关于直接风格效果的[一些](https://twitter.com/debasishg/status/1780636969841914279) [混乱](https://twitter.com/channingwalton/status/1780517826505166989)。在本文中，我希望通过解释直接风格效果的何时、为何以及如何，以Scala 3实现为例，解决这种混乱。

这里有很多事情要讲述。首先，我们将讨论我们试图解决的问题以及我们所操作的约束条件。然后，我们将看看在Scala 3中的简单实现，并描述语言特性——上下文函数，它使其能够实现。接下来，我们将看到此实现的一些不足之处，并了解如何通过两种语言特性来解决这些问题，一种是众所周知的（限定延续），另一种正在开发中的（类型系统创新）。最后，我将提供有关此主题的更多信息的一些指引。

## 我们关心什么

当我们为一种编程风格辩论而不是其他选择时，我们对编程进行价值判断是有帮助的。明确表达这些价值是很有帮助的。正如我在[其他地方](https://noelwelsh.com/posts/what-and-why-fp/)所写的，我认为函数式编程的核心价值是**推理**和**组合**。副作用阻止我们实现这两者，但每个有用的程序必须以某种方式与世界互动。（如果您不确定副作用是什么，请参阅[Creative Scala的这一章节](https://www.creativescala.org/creative-scala/substitution/index.html)进行详细了解。）因此，用符合这些核心原则的东西替换副作用被视为函数式编程中的重要问题。解决此问题的解决方案称为**效果系统**。

请注意：在本文中，我使用术语*副作用*表示不受控制的效果，仅使用*效果*表示以更可控的方式控制的效果。

单子是现代函数式编程中最常见的效果系统，但这并不意味着它们是唯一的方法。Haskell的早期版本使用流。[Clean](https://wiki.clean.cs.ru.nl/Language_features)语言使用独特类型，与Rust的借用检查器中看到的仿射类型非常相关。当前大多数研究工作集中在所谓的**代数效果**和**效果处理程序**的方法上。正是这种方法我们将要探索，尽管我们需要先了解一些背景。

现在我们知道为什么效果系统很有趣，让我们看看效果系统中一些设计选择。

## 效果系统的设计空间

推理和组合对于任何效果系统都是不可协商的标准。但是还有其他一些希望具备的标准。在这里，我们将看看代码编写的风格，描述与动作之间的分离，以及效果系统如何帮助我们推理和组合有副作用的代码的一些微妙之处。

### 直接和单子风格

我们必须编写的代码风格决定了使用效果系统的可用性。如果一个效果系统要求程序员付出过多努力，那么它在实践中是无法使用的，无论它具有什么其他属性。在这里，我们将看看**直接风格**，这是我们想要编写的代码，以及**单子风格**，这是单子效果系统迫使我们编写的代码。

直接风格的代码就是通常编写的代码。您调用函数，它们返回结果，然后您将这些结果用于进一步的计算。这是我们在直接风格中编写的代码示例。

```
val a: A = ???
val b = doSomething(a)
val c = doSomething2(b)
val d = doSomething3(c)
```

我们对直接风格不必多言，只要说写作风格具有可取性即可。

正如大多数Scala程序员将经历的那样，如果我们要使用单子，我们必须以不同的风格编写代码。在单子风格中，上述代码最终看起来像是这样：

```
doSomething(a)
  .flatMap(b => doSomething(b))
  .flatMap(c => doSomething(c))
```

这被认为是足够恼人的，以至于支持单子的语言通常为它们提供特殊的语法。在Scala中，我们可以这样写：

```
for {
  b <- doSomething(a)
  c <- doSomething(b)
  d <- doSomething(c)
} yield d
```

这并不算太糟糕。许多开发人员已经编写过这样的代码。然而，这仍然是一种不同的编码风格，需要学习，因此是一个入门障碍。这也是一个整体程序的转换。一旦我们的代码的一部分开始使用单子，通常情况下所有代码都必须转换为单子风格。因此，理想情况下，另一种效果系统应该允许我们继续以直接风格编写代码。

### 描述与动作

任何效果系统必须在描述应该发生的效果与实际执行这些效果之间有所区分。这是组合的要求。考虑任何编程语言中可能最简单的效果：向控制台打印输出。在Scala中，我们可以通过`println`来作为副作用实现这一点：

```
println("OMG, it's an effect")
```

想象一下我们想要将在控制台上打印文本的效果与改变文本颜色的效果组合起来。使用`println`的副作用我们做不到这一点。一旦调用了`println`，输出就已经打印出来了；没有机会改变颜色。

让我清楚地表明目标是*组合*。我们当然可以使用两个恰好按正确顺序发生的副作用来获得所需颜色的输出。

```
println("\u001b[91m") 
println("OMG, it's an effect")
```

然而，这与组合这两种效果的效果不同。例如，上面的示例不会重置前景色，因此所有后续输出都将是亮红色。这是副作用的经典问题：它们具有“远距离作用”，这意味着程序的一部分可以改变程序的另一部分的含义。这反过来意味着我们不能局部推理，也不能以组合的方式构建程序。

我们真正想要的是像下面这样编写代码

```
Effect.println("OMG, it's an effect").foregroundBrightRed
```

这将仅限于给定文本的前景色。只有当我们在描述效果时与实际运行效果有所分离时才能做到这一点，正如我们上面所做的。

### 通过效果推理和组合

效果系统应该帮助我们推断代码的行为。例如，考虑以下方法签名：

```
def cthulhuFhtagn(): Unit
```

当我们调用这个方法时会发生什么？返回`Unit`表明它具有某种副作用，但这种副作用是什么？它可能会打印到控制台，引发异常，或者唤醒一个古老的神来摧毁地球。我们无法确定。

使用`IO`单子是类似的。如果我们看到的是方法签名

```
def cthulhuFhtagn(): IO[Unit]
```

我们再次不知道会发生什么效果，但我们确实有一些方法来操作这些效果。例如，我们可以尝试通过编写

```
IO.canceled *> cthulhuFhtagn()
```

或者使用`handleError`从错误中恢复。

重要的是我们可以以可组合的方式操作这些效果。例如，我们可以将`IO`传递给另一个方法，该方法选择如何操作它。

```
def cancelOrRecover(effect: IO[Unit]): IO[Unit] =

  IO.realTimeInstant
    .map(time => starsAreRight(time))
    .ifM(
      true = effect.handleError(...),
      false = IO.cancel *> effect 
    )

cancelOrRecover(cthulhuFhtagn())
```

我们不能在使用副作用的第一种情况下做到这一点。

在我们深入效果系统之前，还有另一个问题我想快速处理一下，那就是效果的组合。对`IO`的一个批评是它将所有效果都归为一种类型。我们可能希望更加精确地表达，比如*这个*方法需要记录日志和访问数据库，而*那个*方法从键盘读取并打印到屏幕上。单子变换器是实现这一点的一种方法，但它们很难使用。一个更常见的替代方法是标签终极。这个方法的签名

```
def cthulhuFhtagn[F[_]: WakeGreatOldOne](): F[Unit]
```

表示这个方法需要一个`WakeGreatOldOne`效果，我们可以用它来决定是否调用该方法。标签终极同样不方便，但不至于使其在Scala世界中变得相对常见。

## Scala 3中的直接式效果系统

现在让我们在 Scala 3 中实现一个直接式效果系统。这需要一些在 Scala 3 中新引入的机制。由于这对许多读者可能是不熟悉的，我们将从一个例子开始，解释它背后的编程技术，然后解释它所体现的概念。

我们的示例是一个简单的向控制台打印效果系统。实现如下。你可以将其保存到一个文件中（比如，称为`Print.scala`），然后用`scala-cli`运行它，命令为`scala-cli Print.scala`。

```
 type Console = Console.type

type Print[A] = Console ?=> A
extension [A](print: Print[A]) {

  def prefix(first: Print[Unit]): Print[A] =
    Print {
      first
      print
    }

  def red: Print[A] =
    Print {
      Print.print(Console.RED)
      val result = print
      Print.print(Console.RESET)
      result
    }
}
object Print {
  def print(msg: Any)(using c: Console): Unit =
    c.print(msg)

  def println(msg: Any)(using c: Console): Unit =
    c.println(msg)

  def run[A](print: Print[A]): A = {
    given c: Console = Console
    print
  }

  inline def apply[A](inline body: Console ?=> A): Print[A] =
    body
}

@main def go(): Unit = {

  val message: Print[Unit] =
    Print.println("Hello from direct-style land!")

  val red: Print[Unit] =
    Print.println("Amazing!").prefix(Print.print("> ").red)

  Print.run(message)
  Print.run(red)
}
```

`Print[A]` 是一个描述：一个程序，当运行时可能会向控制台打印，并计算一个类型为 `A` 的值。它被实现为一个[上下文函数](https://docs.scala-lang.org/scala3/reference/contextual/context-functions.html)。你可以将上下文函数视为带有 `given`（隐式）参数的普通函数。在我们的情况下，`Print[A]` 是一个带有 `Console` 给定参数的上下文函数。（`Console` 是 Scala 标准库中的一种类型。）

上下文函数类型有一个特殊规则，使得它们的构造更加简单：如果表达式的类型是上下文函数类型，则普通表达式将被转换为产生上下文函数的表达式。让我们通过实际操作来理解这一点。在上面的例子中，我们有以下这行：

```
val message: Print[Unit] =
  Print.println("Hello from direct-style land!")
```

`Print.println` 是一个类型为`Unit`的表达式，而不是上下文函数类型。但是 `Print[Unit]` 是一个上下文函数类型。这个类型标注会导致 `Print.println` 被转换为上下文函数类型。你可以通过移除类型标注来自行验证：

```
val message =
  Print.println("Hello from direct-style land!")
```

这将无法编译。

我们还使用了与 `Print.apply` 相同的技巧，它是一个通用的构造函数。你可以用任何表达式调用 `apply`，它将被转换为上下文函数。 （据我所知，使用 `inline` 并非必须，但我从学习的所有例子中都这样做了，因此我也这样做。我假设这是一种优化。）

运行 `Print[A]` 使用了另一种特殊机制：如果在上下文函数的作用域中存在正确类型的给定值，则该给定值将自动应用于函数。这也是直接式组合的实现方式的一部分，下面显示了一个示例。调用 `Print.print` 是在一个 `Console` 可用的上下文中，因此将在周围的上下文函数运行一次。

```
def red: Print[A] =
  Print {
    Print.print(Console.RED)
    val result = print
    Print.print(Console.RESET)
    result
  }
```

这就是直接式效果系统在 Scala 中如何工作的机制：它归结为上下文函数。

注意我们在这些示例中的内容：我们以自然的直接式编写代码，但我们仍然有一个信息丰富的类型 `Print[A]`，它帮助我们推理效果，并且我们可以组合类型为 `Print[A]` 的值。

我稍后将处理不同效果的组合等更多内容。不过首先，我想描述我们所做的背后的概念。

注意，在直接样式效果中，我们将效果分为两部分：定义我们需要的效果的上下文函数，以及这些效果的实际实现。在文献中，这些分别称为代数效果和效果处理程序，这与`IO`有一个重要的区别，后者同一类型既指示了效果的需求又提供了这些效果的实现。

还要注意，我们使用上下文函数的参数类型来指示我们需要的效果，而不是像单子效果中那样使用结果类型。这种差异避免了单子中的“有色函数”问题。我们可以将参数视为指定上下文函数所需环境或上下文的要求，因此得名。

现在让我们来看看效果的组合，以及修改控制流的效果。

### 直接样式效果的组合

直接样式效果以直接方式组合：我们只需将额外参数添加到我们的上下文函数中。这里有一个简单的例子，定义了另一个效果，`Sample`，用于生成随机值，然后构建了一个需要同时使用`Print`和`Sample`的程序。

首先，我们使用与之前相同的模式定义效果。

```
import scala.util.Random

type Sample[A] = Random ?=> A
object Sample {

  def run[A](sample: Sample[A])(using random: Random = scala.util.Random): A = {
    given r: Random = random
    sample
  }

  def int(using r: Random): Int =
    r.nextInt()

  def double(using r: Random): Double =
    r.nextDouble()

  inline def apply[A](inline body: Random ?=> A): Sample[A] =
    body
}
```

现在我们可以同时使用`Print`和`Sample`。

```
val printSample: (Console, Random) ?=> Unit =
  Print {
    val i = Sample { Sample.int }
    Print.println(i)
  }

Print.run(Sample.run(printSample))
```

### 改变控制流的效果

到目前为止，我们所看到的效果控制流非常简单。实际上，它们根本不改变控制流。许多有趣的效果，如错误处理和并发，需要操作程序的控制流。我们在模型中如何处理这些？

为了适应这一点，我们需要进行轻微扩展：当用户程序调用效果处理程序方法时，不仅传递该方法的参数，而且还传递了一个**延续**，该延续可以在效果完成时恢复执行。什么是延续？它表示“程序的其余部分”：一个可以调用以从调用效果处理程序的点继续执行的值。合作线程、纤程、生成器和协程都是使用延续形式的抽象的示例。

延续可以作为程序的一种转换来实现，但是为了性能，我们理想情况下希望有运行时支持。这就是为什么[Scala Native正在获取延续](https://github.com/scala-native/scala-native/blob/main/nativelib/src/main/scala-3/scala/scalanative/runtime/Continuations.scala)。在JVM上，[Project Loom](https://cr.openjdk.org/~rpressler/loom/Loom-Proposal.html)正在添加它们。

Scala 3尚未暴露出一个延续API，但它确实在`scala.util.boundary`中有非局部退出，可以表达一些有趣的事情。这里有一个例子，实现了异常处理的样式。

```
 import scala.util.boundary
import scala.util.boundary.{break, Label}

final class Error[-A](using label: Label[A]) {
  def raise(error: A): Nothing =
    break(error)
}

type Raise[A] = Error[A] ?=> A
object Raise {
  inline def apply[A](inline body: Error[A] ?=> A): Raise[A] =
    body

  def raise[A](error: A)(using e: Error[A]): Nothing =
    e.raise(error)

  def run[A](raise: Raise[A]): A = {
    boundary[A] {
      given error: Error[A] = new Error[A]
      raise
    }
  }
}

@main def go(): Unit = {
  val program: Raise[String] =
    Raise {

      List(1, 2, 3, 4)
        .foreach(x => if x == 3 then Raise.raise("Found 3"))
      "No 3 found"
    }

  val result = Raise.run(program)
  println(result)
}
```

请注意，我们仍然保留了描述和动作之间的分离。`program`直到我们调用`Raise.run`时才运行，控制流在运行的点退出，而不是在定义的点退出。

使用直接风格效果，我们可以编写必须以单子风格使用`traverse`或其他组合器的程序。以下是一个从`List[Int]`生成`Option[List[Int]]`的示例。

```
val traverse: Raise[Option[List[Int]]] =
  Raise {
    Some(
      List(1, 2, 3, 4).map(x => if x == 3 then Raise.raise(None) else x)
    )
  }
println(Raise.run(traverse))
```

这相当于使用Cats编写的以下程序：

```
val traverseCats: Option[List[Int]] =
  List(1, 2, 3, 4).traverse(x => if x == 3 then None else Some(x))
```

您可能会想知道单子如何实现影响控制流而不需要运行时支持。答案是单子要求用户显式指定控制流。这正是`flatMap`所做的：它表达了按顺序发生的事情，并通过将这些信息作为一系列`flatMap`链给单子，单子可以按照特定单子实现有意义的顺序进行评估。实际上，单子等价于[界定延续](https://dl.acm.org/doi/10.1145/174675.178047)。因此，直接风格效果和单子效果可以看作是编写相同内容的两种不同语法。

## 捕获、类型和效果

到目前为止，我们所看到的内容表明，效果的实现和使用是直接的，大部分情况下都是如此。但是，至少有一个我们需要注意的问题：捕获效果。

在下面的代码中，我们在闭包中捕获了一个`Error[String]`，然后试图在其有效范围之外调用该`Error`的`raise`方法。这会导致运行时异常。

```
val capture: Raise[() => String] =
  Raise { error ?=> () =>
    if 3 < 2 then "Nothing to see here"
    else Raise.raise(() => "Hahahahaha!")(using error)
  }

val closure = Raise.run(capture)
println(closure())
```

这是否是直接风格效果基础的严重缺陷？不是！到目前为止，我们只看到了Scala 3中当前效果系统的一部分。[捕获检查](https://dotty.epfl.ch/docs/reference/experimental/cc)，它仍处于实验阶段，消除了这类错误。[捕获类型](https://dl.acm.org/doi/10.1145/3618003)论文包含所有技术细节。

事实上，捕获检查比我们到目前为止看到的例子更进一步。它跟踪程序的动态作用域中的能力使用。它可以用来实现不同效果的组合。我们之前看过一个例子，在那里我们构造了一个具有类型`(Console, Random) ?=> Unit`的上下文函数。您可能会对这种类型有些怀疑，因为它没有单独使用我们用于这两种效果的类型别名（`Print`和`Sample`）。通过捕获检查，类型系统为我们解决了这些类型。它还可以用于资源检查，例如确保所有打开的文件都已关闭或实现基于区域的内存管理而没有泄漏。

## 结论和进一步阅读

我们看到，直接风格效果使我们能够以自然的直接风格编写代码，同时保留有助于推理和允许效果组合的有用类型。实现是以下内容的结合：

1.  上下文函数；

1.  继续；和

1.  类型系统改进

它们一起使我们能够以直接风格在效果处理程序上操作效果。

总的来说，我对直接风格效果感到非常激动，特别是在Scala中的直接风格效果。我认为它们比单子效果更符合人体工程学，从而使更广泛的程序员能够轻松使用。我还很高兴能够访问延续，以及更多语言中的尾调用。对于某些问题，如[虚拟机分派](https://noelwelsh.com/posts/understanding-vm-dispatch/)，尾调用确实非常有用。

我也很激动看到Scala继续演进。Scala一直是创新的语言，这些变化只不过是这一传统的延续（意在双关）。我也很期待看到对Scala Native的更多投资。我认为只有在Scala Native中，开发人员才能灵活实现完整的效果系统所需的运行时支持，并且通过提供诸如基于区域的内存管理之类的功能，真正最大化其优势。我还认为Scala Native对于Scala在像无服务器这样的使用案例中的工业采用至关重要，所以我认为对Scala Native的更多投资对社区来说是一个*重大*胜利。

如果你想进一步了解直接风格效果，这里有一些建议，其中既有易于理解的介绍，也有学术论文：

最后，如果你觉得这很有趣，我认为你会喜欢我的书，[函数式编程策略](https://scalawithcats.com/)。它涵盖了这篇文章中的许多概念，如延续传递风格和解释器，以及更多内容。
