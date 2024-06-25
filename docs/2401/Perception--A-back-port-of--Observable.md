<!--yml

category: 未分类

日期：2024-05-27 14:45:32

-->

# Perception: @Observable 的回溯

> 来源：[`www.pointfree.co/blog/posts/129-perception-a-back-port-of-observable`](https://www.pointfree.co/blog/posts/129-perception-a-back-port-of-observable)

Swift 5.9 为语言带来了强大的观察工具，但不幸的是，它们仅适用于 iOS 17、macOS 14、tvOS 17、watchOS 10 和更新的版本。但根据一些统计数据，不到 50% 的设备使用 iOS 17，所以大多数开发者将无法在几年内使用这些工具。

所以我们已经将观察工具回溯到适用于苹果平台的版本，包括 iOS 13、macOS 10.15、tvOS 13、watchOS 6，并将其发布为[开源库](http://github.com/pointfreeco/swift-perception)。这意味着您可以通过使用我们的库来大大简化您的 SwiftUI 视图*今天*。

加入我们快速浏览我们的新库：[Perception](http://github.com/pointfreeco/swift-perception)。

## 使用 Perception

该库提供了其自己版本的 Swift 5.9 中的观察工具，但它们可以在旧的 Apple 平台上使用。在设计您的模型时，您将使用我们的`@Perceptible`宏，而不是使用`@Observable`宏：

```
+import Perception

-@Observable
+@Perceptible
 class FeatureModel {
   var count = 0
 } 
```

这就是`FeatureModel`类跟踪其属性访问并在这些属性发生变化时广播的全部内容。

这个模型可以在 SwiftUI 视图中使用，但必须执行一个额外步骤以确保视图订阅模型的更改。我们必须将视图包装在一个特殊的视图中，称为`WithPerceptionTracking`：

```
 struct FeatureView: View {
   let model: FeatureModel 

   var body: some View {
+    WithPerceptionTracking {
       Form {
         Text(model.count.description)
         Button("Increment") {
           model.count += 1
         }
       } 
     }
+  }
 } 
```

一方面，我们不得不将我们的视图包装在这个视图中，这有点不太幸运，但另一方面，今天就可以使用这些观察工具是太棒了，而不必等待 iOS 17 的广泛采用。

如果您忘记使用`WithPerceptionTracking`，该库将会帮助您。如果在视图中访问`@Perceptible`类的字段，而 *不* 在`WithPerceptionTracking`内部，则会触发运行时警告：

> 🟣 运行时警告：Perceptible 状态已被访问，但未被跟踪。通过将您的视图包装在‘WithPerceptionTracking’视图中来跟踪状态的变化。

这让您立即知道何时设置不正确，并以一种引人注目但[不显眼的](https://www.pointfree.co/blog/posts/70-unobtrusive-runtime-warnings-for-libraries)方式执行此操作。要调试此问题，请展开 Xcode 的 Issue Navigator（⌘5），并点击显示的堆栈帧，找到您正在访问状态而不在`WithPerceptionTracking`内部的视图中的行。

## Perception 库的工作原理

新的 Observation 框架是 Swift 开源项目的一部分，这意味着所有的[源代码](https://github.com/apple/swift/tree/f7f5070454850ed6bda85a9574b1759115705da4/stdlib/public/Observation)都可以立即获得，包括[@Observable 宏的源代码](https://github.com/apple/swift/tree/f7f5070454850ed6bda85a9574b1759115705da4/lib/Macros/Sources/ObservationMacros)。因此，我们能够将所有这些代码复制到一个新项目中，并在进行了一些小的更改后使其全部编译通过。

我们还对代码进行了一些重大更改，使其表现出我们想要的行为。我们对代码进行的第一个重大更改是将所有“observation”的变体重命名为“perception”（*例如*，`@Observable`变为`@Perceptible`）。我们这样做是为了清楚地表明这些工具是与苹果直接在 Swift 工具链中提供的工具分开的。但我们还[弃用](https://github.com/pointfreeco/swift-perception/blob/92858a542c498742d51c1e736591d91a807d65a7/Sources/Perception/Perceptible.swift#L19-L23)了所有使用重命名的后备工具，这样一旦您将最低部署目标设置为 iOS 17，您就可以轻松过渡到我们的库。

此外，我们希望我们的工具在运行在 iOS 17 设备上时能够推迟使用 Apple 的原生 Observation 框架。这需要在运行时进行一些额外的工作。所有的工作都发生在`PerceptionRegistrar`中，它包装了一个原生的`ObservationRegistrar`（如果可能的话），如果不可能，它就会回退到我们的后端端口，即`_PerceptionRegistrar`：

```
public struct PerceptionRegistrar: Sendable {
  private let _rawValue: AnySendable
  public init() {
    if #available(iOS 17, macOS 14, tvOS 17, watchOS 10, *) {
      #if canImport(Observation)
        self._rawValue = AnySendable(ObservationRegistrar())
      #else
        self._rawValue = AnySendable(_PerceptionRegistrar())
      #endif
    } else {
      self._rawValue = AnySendable(_PerceptionRegistrar())
    }
  }
} 
```

我们提供了获取这种类型的诚实、未包装的`ObservationRegistrar`或`_PerceptionRegistrar`的方法：

```
extension PerceptionRegistrar {
  #if canImport(Observation)
    @available(iOS 17, macOS 14, tvOS 17, watchOS 10, *)
    private var registrar: ObservationRegistrar {
      self._rawValue.base as! ObservationRegistrar
    }
  #endif

  private var perceptionRegistrar: _PerceptionRegistrar {
    self._rawValue.base as! _PerceptionRegistrar
  }
} 
```

请注意，这些属性在技术上是不安全的，因为我们进行了强制类型转换。但我们可以小心地只在基础包装值是我们预期的注册器类型时才调用这些属性。

然后，我们需要在`PerceptionRegistrar`上实现`access`、`willSet`、`didSet`和`withMutation`方法，并以一种可以在运行时动态选择调用我们的后备工具或苹果原生工具的方式来实现。

例如，`access`可以作为一种方法来实现，该方法受限于与`Observable`类型而不是`Perceptible`类型一起使用：

```
extension PerceptionRegistrar {
  public func access<Subject: Perceptible, Member>(
    _ subject: Subject,
    keyPath: KeyPath<Subject, Member>
  ) {
    …
  }
} 
```

然后在这个方法的主体中，我们可以动态地检查 iOS 17 是否可用，如果是，尝试将对象转换为`Observable`协议，以及进行一些花哨的操作来打开外在性并将关键路径转换为正确的类型：

```
extension PerceptionRegistrar {
  @_disfavoredOverload
  public func access<Subject: Perceptible, Member>(
    _ subject: Subject,
    keyPath: KeyPath<Subject, Member>
  ) {
    #if canImport(Observation)
      if #available(iOS 17, macOS 14, tvOS 17, watchOS 10, *) {
        func `open`<T: Observable>(_ subject: T) {
          self.registrar.access(
            subject,
            keyPath: unsafeDowncast(keyPath, to: KeyPath<T, Member>.self)
          )
        }
        if let subject = subject as? any Observable {
          open(subject)
        }
      } else {
        perceptionCheck()
        self.perceptionRegistrar.access(subject, keyPath: keyPath)
      }
    #endif
  }
} 
```

类似的技巧也可以用于`willSet`、`didSet`和`withMutation`方法。

这些小技巧使 iOS 16 及更早版本的设备可以使用我们的感知框架，但 iOS 17 及更新版本的设备将使用 Swift 5.9 中的本地观察工具。

## 从今天开始入门

今天就在你的项目中尝试使用[Perception](http://github.com/pointfreeco/swift-perception)，开始利用 Swift 令人惊叹的观察工具，即使你无法面向最新的苹果平台。
