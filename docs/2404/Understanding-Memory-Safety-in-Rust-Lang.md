<!--yml

类别：未分类

日期：2024-05-27 13:04:30

-->

# 在Rust Lang中理解内存安全

> 来源：[https://coderoasis.com/rust-lang-memory-safety/](https://coderoasis.com/rust-lang-memory-safety/)

过去十年对于Rust编程语言而言是不可思议的，它已成为写快速、机器本地化软件和Web应用程序的首选。语言的设计目标是为了强有力地保证内存安全。我们在这里尝试更好地理解这一点。

类似于`C`的语言运行速度快，属于低级别语言。它们通常缺乏确保程序在使用和终止时内存分配和释放正确的功能。白宫国家网络安全主任指出，如果不遵循适当的编程标准，可能会导致[严重的漏洞和安全问题，这可能导致一些重大的现实世界后果](https://www.tomshardware.com/software/security-software/white-house-urges-developers-to-avoid-c-and-c-use-memory-safe-programming-languages?ref=coderoasis.com)。像Go和Rust这样把内存安全放在首位的语言正越来越受到关注。

Rust语言如何保证内存安全？让我们一起来了解一下！

* * *

> *您喜欢正在阅读的内容吗？我们建议您继续学习，阅读下一篇文章 – *[**用JavaScript从头开始实现RSA**](https://coderoasis.com/implementing-rsa-from-scratch-in-javascript/)* –*

[用JavaScript从头开始实现RSA

请注意，我必须强调这里所介绍的代码和技术仅供教育目的，不应在没有仔细考虑和专家指导的情况下用于真实的应用程序中。同时，理解RSA加密原理并探索各种](https://coderoasis.com/implementing-rsa-from-scratch-in-javascript/)

* * *

## 内存安全是本地的

关于Rust编程语言最重要的理解是，它们不需要额外的库或分析工具来检查问题。尽管可以选择性地进行双重检查，但内存安全特性默认内置在编程语言中。这些是强制性的，是在代码运行之前强制执行的标准。

在Rust中，默认情况下，不安全的代码被视为*编译器错误*而不是*运行时错误*。这对于在Rust中编写代码具有重大优势，因为无效的代码永远不会编译通过。这意味着它永远不会进入生产环境。而在像`C`或`C++`这样的语言中，内存错误往往只有在运行时才会被发现。

请记住，这并不意味着Rust编程语言或者使用它编写的任何软件应用程序完全是无懈可击的。仍可能存在一些罕见的问题，比如[竞争条件](https://doc.rust-lang.org/nomicon/races.html?ref=coderoasis.com)，开发者仍需确保其代码中不存在这些问题。然而，同时，Rust确实排除了许多常见的漏洞和安全问题。

编程语言如`C#`、`Python`和`Java`要求开发者手动进行内存管理。而使用Rust，开发者更能集中精力编写代码并更高效地完成工作。这种便利性通常会以一定的成本为代价——通常是速度上的损失或更大的运行时需求。根据我在Rust上的使用经验，生成的二进制文件通常更为紧凑，且默认情况下以本机机器级速度运行，同时始终保持内存安全。

## 默认为不可变

在研究Rust编程语言的最初阶段，我了解到所有变量默认为不可变。这意味着它们无法重新分配或修改。如果您想要更改或修改一个不可变的变量或函数，则必须明确声明它为可变以便进行更改。

对于很多开发者来说，这可能看起来微不足道或不重要，但它确实使开发者高度关注在他们开发的软件应用程序中哪些值需要是可变的，包括需要它们何时变化。Rust应用程序的结果使开发者更容易确定何时、何地以及可以改变什么。

默认为*不可变*的概念非常重要，与常数的概念有所不同。我们以不可变变量为例。该变量可以在运行时首先计算，然后存储为不可变。这意味着它可以首先计算，然后存储，并且不会更改。现在，我们再来看一个常数。常数必须在编译时计算——这也是程序运行之前。有许多不同类型的值——例如用户输入的一个示例——无法以这种方式作为常数存储。

`C++`编程语言假设这与Rust编程语言相反——其中一切都高度可变。默认情况下，您必须使用`const`关键字将某些内容声明为不可变。理论上，您可以诚实地采用`C++`的编码风格，将`const`作为默认选项。这只会覆盖您为自己的软件应用程序编写的代码。而使用Rust，默认情况下编写的所有软件应用程序都是不可变的。

## 内存安全的代价

还有一些与内存安全相关的成本也需要我们讨论。我脑海中首先想到的是需要学习如何使用语言本身。

我要承认，对我来说，切换到一种新的编程语言从来都不是一件容易的事情。到目前为止，Rust 对我来说有一个巨大的学习曲线。我需要一些时间来掌握内存管理模型，以便能够为我想要学习制作的应用程序生成可用的代码。语言的学习曲线已经成为在线许多不同论坛、社区和网站上从初学者到语言专家的大讨论。

像`C`和`C++`这样的老旧语言，它们甚至拥有更大、更有经验的开发者社区，为它们辩护优于 Rust。它们还有 Rust 在某些方面缺少的东西，例如代码示例、库和完整的应用程序示例。对我来说，有时很难理解为什么有些人会选择使用任何`C`语言而不是 Rust。它们拥有更多的开发工具和围绕它们存在的资源。

然而，在 Rust 存在的这十年中，它已经获得了许多开发工具、良好的文档和一个令人惊叹的社区用于学习和讨论。它们使得快速上手和开发优秀软件应用变得非常容易。第三方库的惊人集合 - 也称为 Crates - 非常广泛，每天都在不断增长。加入 Rust 编程语言的开发者可能需要一些重新培训和学习新工具，但你几乎永远不会缺少所需的资源或给定项目的库。

## 缓冲区溢出示例

```
#include 
#include 
int main (void){
    printf ("Hello, world!\n");
    printf ("Let's overflow the buffer!\r\n");
    uint32_t array[5] = {0, 0, 0, 0, 0,};
    for(int index = 0; index < 6; index++) {
        printf("Index %d: %d\r\n", index, array[index]);
    }
    return 0;
} 
```

代码示例

当我们编译此代码时，它无错误和警告地编译。然而，当我们运行代码时，查看以下输出：

```
Hello, world!
Let's overflow the buffer!

Index 0: 0
Index 1: 0
Index 2: 0
Index 3: 0
Index 4: 0
Index 5: 1
```

编译后的代码输出

## 什么是所有权、借用和引用？

默认情况下，Rust 编程语言中的每个值都有一个*所有者*。这意味着任何值 - 无论何时或代码中的任何点 - 都只能在某一时刻具有完全的读/写控制权。一个值的所有权可以被转移，或者更好地*借用*自原始所有者。然而，在值被借用的同时，这种行为严格由编译器跟踪。如果任何代码试图违反编程语言中定义的所有权规则，则编译器拒绝编译该代码。

当我们看看`C`编程语言如何处理这个问题时，我们发现对象没有所有权。这意味着我们想要的任何东西都可以随时被任何其他东西访问。如何修改、修改或使用任何东西的所有责任都取决于程序员如何开发软件应用程序。

在其他编程语言如 `Python`、`C#` 和 `Java` 中，所有权规则根本不存在。这也是因为它们根本不需要存在。这通常意味着它们处理对象访问 – 还包括内存安全 – 是在运行时处理的。与之前一样，这是有代价的 – 它可以是大的或小的，具体取决于软件应用程序 – 这是速度、大小甚至运行时存在性的问题。

## 所有权代码示例

在下面的例子中，当我们将 `s1` 变量赋值给 `s2` 时，`s1` 不再有效。这样可以防止双重释放错误，因为 `s2` 超出作用域。这样只允许内存释放一次。如果我们在将变量所有权交给 `s2` 后尝试使用 `s1`，则会收到编译时错误。

```
fn main() {
    let s1 = String::from("hello");
    let s2 = s1;

    // println!("{}, world!", s1); // This line would cause a compile-time error
    println!("{}, world!", s2);
}
```

变量所有权

下一个例子展示了在将变量传递给函数时如何通过变量传递所有权。如果 `type` 实现了 `Copy` 特性，比如 `整数`、`布尔值` 和其他类型，那么值也会被复制。然后调用的原始变量也可以继续使用。否则，当所有权被移动时，原始变量就不能再使用了。

```
fn main() {
    let s = String::from("hello"); // s becomes the owner of the memory holding "hello"

    takes_ownership(s); // s's value moves into the function...
                        // ... and so is no longer valid here

    // println!("{}", s); // This line would cause a compile error because the ownership of s's value has been transferred

    let x = 5;                  
    makes_copy(x); // x would move into the function,
                   // but i32 is Copy, so it's okay to still
                   // use x afterward

    println!("{}", x); // Works fine because i32 is Copy and doesn't move ownership
}

fn takes_ownership(some_string: String) { // some_string comes into scope
    println!("{}", some_string);
} // Here, some_string goes out of scope and `drop` is called. The backing memory is freed.

fn makes_copy(some_integer: i32) { // some_integer comes into scope
    println!("{}", some_integer);
} // Here, some_integer goes out of scope. Nothing special happens. 
```

函数所有权

## 一些要点

+   所有权是 Rust 编程语言的一个非常强大的特性，它确保了内存安全和效率。

+   该语言强制每个内存片段只有一个所有者。Rust 消除了诸如双重释放、内存泄漏和悬垂指针等常见错误。

+   在 Rust 中编写应用程序的关键是从现在开始正确理解和使用所有权。

## 借用代码示例

Rust 编程语言允许您引用值而无需拥有它。这称为借用。借用有两种类型 – 它们是不可变的和可变的。

在下面的例子中，`s` 被用作对 `s1` 的引用，这意味着它不拥有 `String`。由于 `s` 变量不拥有所有权，调用 `calculate_length` 后 `s1` 变量仍然有效。

```
fn main() {
    let s1 = String::from("hello");
    let len = calculate_length(&s1);

    println!("The length of '{}' is {}.", s1, len);
}

fn calculate_length(s: &String) -> usize {
    s.len()
} 
```

下一个例子是可变借用。这允许您更改被借用的东西。但是有一个很大的限制。在特定范围内同一时间只能有一个可变引用数据。这是为了防止在编译时发生数据竞争。

```
fn main() {
    let mut s = String::from("hello");

    change(&mut s);

    println!("{}", s);
}

fn change(some_string: &mut String) {
    some_string.push_str(", world");
} 
```

## 生命周期是什么？

生命周期是 Rust 编程语言中对值的引用，它们没有所有者 – 但它们确实有生命周期。这意味着给定引用被允许有效的范围。在大多数情况下，编写 Rust 代码时，生命周期被隐式地留下，因为编译器可以轻松地追踪它们。然而，同时，生命周期也可以用于更复杂的用例，显式地使用。

还需记住，如果试图在给定的生命周期之外访问或修改某些内容 – 更好的是如果它已经超出作用域 – 这将导致编译器错误。这个目标是防止整类bug最终进入生产环境或实际软件应用程序。

`悬空指针（Dangling pointers）`可能在你试图访问一个已被释放或超出作用域的变量或函数时出现。这种情况在`C`和`C++`编程语言中比在Rust中更为常见。`C`编程语言在编译时没有官方或适当的执行对象生命周期的强制规定。`C++`编程语言引入了*智能指针*的概念来帮助预防这种情况 – 它们并不是默认的本地实现或强制执行 – 因此你需要选择使用它们。

许多人把编程语言的安全性留给开发者。我的意思是，当涉及到`C`或`C++`编程语言中的内存安全实践时，这通常是开发者采用的编码风格或机构要求 – 并不是编程语言默认要求你做的事情或内置于语言中。

在`C#`、`Java`或甚至`Python`等编程语言中，内存管理是语言运行时的核心责任。这会以相当大的成本 – 在我个人看来 – 即需要一个相当大的运行时。这可能大大降低软件应用程序的执行速度。而Rust编程语言通过在代码运行之前强制执行`生命周期（lifetimes）`规则来减少执行速度。

## 生命周期代码示例

生命周期保证引用只在需要时有效。编译器使用生命周期来确保它们不会超出所引用的任何数据的生命周期。

在下面的示例中，使用`longest`函数，我们声明了一个生命周期`a`来指定函数的返回值。这将允许它的生命周期与两个输入生命周期中的最短者相同。

```
fn main() {
    let string1 = String::from("abcd");
    let string2 = "xyz";

    let result = longest(string1.as_str(), string2);
    println!("The longest string is {}", result);
}

fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
} 
```

## 一些收获

+   生命周期确保引用不会超出它们所引用的数据的生存期。

+   它们是`类型（Type）`系统的一部分，用于在编译时执行检查，而没有运行时成本。

+   函数中的生命周期定义了不同引用之间生命周期的关系。

+   在Rust中编写应用程序的关键是从现在开始正确理解和使用生命周期。

## 结论

Rust已经改变了关于向现有编程语言添加功能或修改缺乏内存安全特性的对话。它们正在足够改变其他语言以采纳这些特性。

这些可能是其他语言中实现起来可能更难的雄心勃勃的想法。例如，如果将这些功能添加到 `C` 或 `C++` 中，我们将不得不讨论向后兼容性问题。Rust 默认提供的内存安全行为更难实现，因为它将强制现有的遗留代码与新的行为方式进行严格分离，开发人员在切换时必须考虑这一点。

Rust 编程语言在编程世界中占据一席之地的主要原因，以及它相对于其他语言提供的最显著特征是内存安全和编译时行为的保证。使用这些特性——Rust 希望你这样做的内存安全方式——将会在开发者最初需投入更多工作，但这种折衷方案在软件应用程序中后续总是会得到回报。

* * *

> 你是否喜欢阅读 CoderOasis 技术博客的内容？我们推荐阅读我们的 [***Python 从零开始实现 RSA***](https://coderoasis.com/implementing-rsa-from-scratch-in-python/) 系列。

[Python 从零开始实现 RSA

我需要强调的是，这里所展示的代码和技术仅供教育目的，不应在现实应用中使用，除非经过仔细考虑和专家指导。同时，理解 RSA 加密原理并探索各种技术](https://coderoasis.com/implementing-rsa-from-scratch-in-python/)

* * *

> 你知道我们有一个 [**社区论坛**](https://forums.coderoasis.com/?ref=coderoasis.com) 和 [**Discord 服务器**](https://discord.gg/sYNeQAqQZC?ref=coderoasis.com) 吗？我们邀请每个人加入我们！想要与社区的其他成员讨论这篇文章吗？想要加入一个轻松愉快的地方，讨论编程、网络安全、Web 开发和 Linux 等主题吗？今天就考虑加入我们吧！

[加入 CoderOasis.com 的 Discord 服务器！

CoderOasis 提供关于编程、安全、Web 开发、Linux、系统管理等技术新闻文章。 | 112 位会员](https://discord.gg/sYNeQAqQZC?ref=coderoasis.com) [CoderOasis 论坛

CoderOasis 社区论坛，这里是我们的会员们可以一起讨论技术并分享资源的地方。](https://forums.coderoasis.com/?ref=coderoasis.com)
