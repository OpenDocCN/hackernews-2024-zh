<!--yml

category: 未分类

date: 2024-05-27 14:41:02

-->

# Swift.org - Swift 5.10发布

> 来源：[https://www.swift.org/blog/swift-5.10-released/](https://www.swift.org/blog/swift-5.10-released/)

# Swift 5.10发布

2024年3月5日

Swift的设计默认安全，可以在编译时防止整类编程错误。在基于C的语言中，如在变量初始化之前使用或使用后释放的情况，这些未定义行为在Swift中都被定义清楚。

并发代码中越来越重要的未定义行为来源之一是一个线程在写入内存的同时，另一个线程无意中访问同一内存。这种不安全性被称为*数据竞争*，而数据竞争使得并发程序异常难以正确编写。Swift通过演员（actors）和任务（tasks）提供的*数据隔离*来解决这个问题，保证对共享可变状态的互斥访问。数据隔离的执行自2020年发布[Swift并发路线图](https://forums.swift.org/t/swift-concurrency-roadmap/41611)以来一直在积极开发中。

**Swift 5.10在并发语言模型中实现了完全的数据隔离**。这一重要的里程碑经过多个版本的积极开发数年才实现。并发模型在Swift 5.5中引入了`async`/`await`、演员和结构化并发。Swift 5.7引入了`Sendable`作为可以在任意并发上下文中共享其值而不引入数据竞争风险的线程安全类型的基本概念。现在，在Swift 5.10中，当启用完整并发检查选项时，编译时会在语言的所有领域强制执行完全的数据隔离。

Swift 5.10中的完全数据隔离为下一个主要版本Swift 6铺平了道路。Swift 6.0编译器将提供一个新的可选Swift 6语言模式，默认情况下强制执行完全的数据隔离，我们将开始过渡以消除Swift编写的所有软件中的数据竞争。

在某些情况下，Swift 5.10将在代码能够通过额外的编译器分析被证明安全时生成数据竞争警告。Swift 6版本的主要开发重点是通过减少常见代码模式中被证明安全的假阳性并改善严格并发检查的可用性，以提升语言发展。

继续阅读了解Swift 5.10中的完全数据隔离、用于演员隔离检查的新不安全选择，以及Swift 6之前剩余的并发演进。

Swift 5.10 在语言的所有角落完善了数据竞争安全语义，并修复了`Sendable`和演员隔离检查中的许多错误，以加强完整并发检查的保证。使用编译器标志`-strict-concurrency=complete`构建代码时，Swift 5.10 将在编译时诊断出潜在的数据竞争，除非使用显式的不安全退出，例如`nonisolated(unsafe)`或`@unchecked Sendable`。

例如，在 Swift 5.9 中，由于在演员之外评估了`@MainActor`隔离的初始化器，以下代码在运行时失败了隔离断言，但在`-strict-concurrency=complete`下未被诊断：

```
@MainActor
class MyModel {
  private init() {
    MainActor.assertIsolated()
  }

  static let shared = MyModel()
}

func useShared() async {
  let model = MyModel.shared
}

await useShared() 
```

上述代码允许数据竞争。`MyModel.shared`是一个`@MainActor`隔离的静态变量，在首次访问时评估了一个`@MainActor`隔离的初始值。`MyModel.shared`从`nonisolated`上下文中同步访问，因此初始值在主演员之外计算。在Swift 5.10中，使用`-strict-concurrency=complete`编译代码会产生一个警告，要求异步执行访问：

```
 warning: expression is 'async' but is not marked with 'await'
    let model = MyModel.shared
                ^~~~~~~~~~~~~~
                await 
```

解决数据竞争的可能修复方法包括 1) 使用`await`异步访问`MyModel.shared`，2) 使`MyModel.init`和`MyModel.shared`都是`nonisolated`并将需要主演员的代码移到单独的隔离方法中，或者 3) 将`useShared()`隔离到`@MainActor`。

您可以在[Swift 5.10 发布说明](https://github.com/apple/swift/blob/release/5.10/CHANGELOG.md)中找到有关完整数据隔离编程模型的更多详细信息。

例如，`@unchecked Sendable`符合要求对于表明代码在不能被编译器自动证明安全免于数据竞争的情况下是重要的。这些工具在同步通过操作系统特定原语实现或在与 C/C++/Objective-C 实现的线程安全类型一起工作时，编译器无法推理的情况下是必要的。然而，`@unchecked Sendable`的符合要求很难正确使用，因为它将整个类型从数据竞争安全检查中排除。在许多情况下，类型中只有一个特定的属性需要排除，而其余的实现符合静态并发安全。

Swift 5.10 引入了一个新的`nonisolated(unsafe)`关键字，用于取消对存储属性和变量的演员隔离检查。`nonisolated(unsafe)`可以用于任何形式的存储，包括存储属性、局部变量和全局/静态变量。

例如，全局和静态变量可以从代码的任何位置访问，因此它们要么是不可变的且`Sendable`，要么是隔离到全局演员中：

```
import Dispatch

struct MyData {
  static let cacheQueue = DispatchQueue(...)
  // All access to 'globalCache' is guarded by 'cacheQueue'
  static var globalCache: [MyData] = []
} 
```

使用`-strict-concurrency=complete`构建上述代码时，编译器会发出警告：

```
warning: static property 'globalCache' is not concurrency-safe because it is non-isolated global shared mutable state
  static var globalCache: [MyData] = []
             ^
note: isolate 'globalCache' to a global actor, or convert it to a 'let' constant and conform it to 'Sendable' 
```

所有对`globalCache`的使用都受到`cacheQueue.async { ... }`的保护，因此该代码在实践中不会发生数据竞争。在这种情况下，可以将`nonisolated(unsafe)`应用于静态变量以消除并发警告：

```
import Dispatch

struct MyData {
  static let cacheQueue = DispatchQueue(...)
  // All access to 'globalCache' is guarded by 'cacheQueue'
  nonisolated(unsafe) static var globalCache: [MyData] = []
} 
```

`nonisolated(unsafe)` 还消除了只用于在没有并发访问可能性时跨隔离边界传递特定实例的非`Sendable`值的`@unchecked Sendable`包装类型的需要：

```
// 'MutableData' is not 'Sendable'
class MutableData { ... }

func processData(_: MutableData) async { ... }

@MainActor func send() async {
  nonisolated(unsafe) let data = MutableData()
  await processData(data)
} 
```

请注意，如果没有正确实现同步机制来实现数据隔离，来自排他性强制执行或诸如线程检测器之类的工具的动态分析仍可能识别失败。

**Swift 的下一个版本将是 Swift 6。** Swift 5.10 中的完整并发模型过于严格，并且有几个 Swift Evolution 提案正在积极开发中，以改进通过消除假阳性数据竞争错误来改进完整数据隔离的可用性。这项工作包括[在编译器确定不存在并发访问可能性时，解除跨隔离边界传递非`Sendable`值的限制](https://github.com/apple/swift-evolution/blob/main/proposals/0414-region-based-isolation.md)，[函数和关键路径的更有效`Sendable`推断](https://github.com/apple/swift-evolution/blob/main/proposals/0418-inferring-sendable-for-methods.md)，等等。您可以在[Swift.org/swift-evolution](https://www.swift.org/swift-evolution/#?version=6.0)找到将完善 Swift 6 的一系列提案。

你可以通过在项目中[尝试完整的并发检查](/documentation/concurrency/)来帮助塑造向 Swift 6 语言模式的过渡，并提供您的体验反馈。

如果您发现任何编译器仍未在编译时诊断数据竞争的完整并发检查错误，请[报告问题](https://github.com/apple/swift/issues/new/choose)。

您还可以提供反馈，帮助改进并发文档、编译器错误消息以及即将推出的 Swift 6 迁移指南。如果您遇到编译器诊断您不理解的数据竞争警告，或者不确定如何解决给定的数据竞争警告，请在[Swift 论坛上的讨论线程](https://forums.swift.org/tags/c/swift-users/15/concurrency)上使用`concurrency`标签开启讨论。

官方提供的 Swift 5.10 二进制文件可以从[Swift.org](https://swift.org/download/)下载，适用于 macOS、Windows 和 Linux。

以下语言提案通过[Swift Evolution](https://github.com/apple/swift-evolution)流程得到接受，并且已经在[Swift 5.10](https://www.swift.org/swift-evolution/#?version=5.10)中实现：
