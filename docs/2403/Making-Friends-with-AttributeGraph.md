<!--yml

category: 未分类

date: 2024-05-27 14:43:14

-->

# 与 AttributeGraph 交朋友

> Source: [https://saagarjha.com/blog/2024/02/27/making-friends-with-attributegraph/](https://saagarjha.com/blog/2024/02/27/making-friends-with-attributegraph/)

如果你使用 [SwiftUI](https://developer.apple.com/xcode/swiftui/) 已经有一段时间了，你可能已经注意到它提供的公共 Swift API 实际上只是故事的一半。通常情况下，除非出现 *非常* 大的问题，否则不显眼的私有框架 AttributeGraph 会从幕后跟踪你应用程序的几乎每个方面，以便在需要更新时做出决策。可以毫不夸张地说，这个 C++ 库实际上是控制一切的核心，SwiftUI 只是在上面加了一层薄薄的外壳，用于绘制一些适合平台的控件，并提供一个稳定的编程接口。与其名字相符，AttributeGraph 为声明式 UI 框架提供了必要的基础：跟踪数据依赖关系的属性图。

精通这些依赖关系是编写高级 SwiftUI 代码的关键。不幸的是，由于它是闭源框架的私有实现细节，因此在线搜索 AttributeGraph 通常只能找到那些在应对崩溃时急需帮助的人的结果。（尽管逆向工程相当令人不快，但是 [一些人](https://www.fivestars.blog/articles/lets-build-state/) [试过](https://medium.com/@kateinoigakukun/inside-swiftui-how-state-implemented-92a51c0cb5f6) [试图](https://kyleye.top/posts/demystify-attributegraph-1/).）Apple [提供了](https://developer.apple.com/videos/play/wwdc2020/10040) [几个](https://developer.apple.com/videos/play/wwdc2021/10022) [视频](https://developer.apple.com/videos/play/wwdc2023/10160/)，介绍了高层设计，但并不意外地回避了提及 AttributeGraph 的存在。其他开发人员确实提到过，但 [只是一笔带过](https://chris.eidhof.nl/presentations/day-in-the-life/)。

这让我们陷入了真正的困境！我们可以整天调用 `Self._printChanges()`，但仍然无法理解到底发生了什么，特别是如果我们的问题与 *丢失* 更新有关而不是更新过多的话。老实说，除非 AttributeGraph 工作不正常，否则了解它内部的运行机制并没有多大用处。我们不会轻易调用那些私有 API，所以探索它们没有太多意义。更重要的是理解 SwiftUI 的作用以及设置依赖关系以支持它的需要。我们可以借鉴生成式 AI 的方法，尝试猜测事物的实现方式。与 AI 不同的是，我们还可以测试我们的理论。我们不会知道我们的猜测是否正确，但我们肯定可以检查以确保我们没有错！

## 引入 `Setting`

当我们探索 SwiftUI 如何传播更改时，拥有一个真实的例子将非常有帮助。请欢迎使用 `Setting`，一个类似于 [`AppStorage`](https://developer.apple.com/documentation/swiftui/appstorage) 的简单的 `UserDefaults` 包装器：

```
@propertyWrapper
struct Setting<T> {
	init(_ key: String, defaultValue: T)
	var wrappedValue: T { get set }
	var projectedValue: Binding<T> { get }
	var isSet: Bool { get }
	func reset()
} 
```

我们还没有实现它，但是看起来很容易看到你可能如何使用它：

```
@Setting("favoriteNumber", defaultValue: 42)
var favoriteNumber

favoriteNumber = 69
print(favoriteNumber) // 69
print($favoriteNumber.isSet) // true
$favoriteNumber.reset()
print($favoriteNumber.isSet) // false
print(favoriteNumber) // 42 
```

我们的 API 更加明确地定义了未设置值应该如何工作，但除此之外几乎与苹果的一致：我们甚至将 `Binding` 暴露为 `projectedValue`，以便 `Setting` 可以直接与 SwiftUI 控件一起使用。下面是初步实现的样子。其中任何内容都不应该特别令人惊讶：

```
@propertyWrapper
struct Setting<T> {
	let key: String
	let defaultValue: T

	init(_ key: String, defaultValue: T) {
		self.key = key
		self.defaultValue = defaultValue
	}

	var wrappedValue: T {
		get {
			UserDefaults.standard.object(forKey: key) as? T ?? defaultValue
		}
		set {
			UserDefaults.standard.setValue(newValue, forKey: key)
		}
	}

	var projectedValue: Binding<T> {
		Binding(
			get: {
				wrappedValue
			},
			set: {
				wrappedValue = $0
			}
		)
	}

	var isSet: Bool {
		UserDefaults.standard.value(forKey: key) != nil
	}

	func reset() {
		UserDefaults.standard.removeObject(forKey: key)
	}
} 
```

这 *几乎* 可以工作。然而，在我们的实现中有一个编译器错误，针对 `projectedValue` —— 它捕获了 `self`，当 `Binding` 更新时，它试图调用 `wrappedValue` 的设置器。但由于 `self` 是不可变的，编译器不会允许我们修改 `wrappedValue`。或者说，它会？

我们这里并没有真正修改 `self`：设置器只是写入用户默认设置。特别是，它根本不触及结构体的任何成员，因此 `self` 不会改变。Swift 有一种特殊的方法来适应这种情况：一个 `nonmutating` 的设置器。这实际上是 `State` 自己使用的方式，让你在它所属的视图不可变的情况下也能对其进行赋值：它将其状态存储在外部，就像我们所做的一样。让我们更新我们的 `wrappedValue` 实现来使用这个。

```
var wrappedValue: T {
	get {
		UserDefaults.standard.object(forKey: key) as? T ?? defaultValue
	}
	nonmutating set {
		UserDefaults.standard.setValue(newValue, forKey: key)
	}
} 
```

有了这个，代码编译通过了，让我们试试：

```
struct ContentView: View {
	@Setting("enabled", defaultValue: false)
	var enabled

	var body: some View {
		Toggle("Item", isOn: $enabled)
	}
} 
```

## 处理更新

如果你将上面的代码复制到你自己的测试项目中，你可能已经注意到它并没有像我们希望的那样工作。你可以随意切换开关，但 UI 并不会更新。怎么回事？我们已经创建了一个 `Binding` 啊！

使用调试器，很容易验证 `Toggle` *确实* 像应该的那样调用 `Binding` 回调。当我们切换开关时，它调用了设置实现，然后到我们的 `wrappedValue` 设置器，它写入到用户默认设置。事实上，如果重新启动应用程序，UI 将读取在用户默认设置中保存的正确状态，因此问题在于 SwiftUI 如何更新视图而不是值的后备存储。由于我们附加了调试器，我们可以看到 getter 被调用了几次，这必须是框架设置视图初始状态的方式。不过，在此之后，UI 不会更新，即使 getter 返回了新值。值得注意的是，即使应该因为新状态而重新评估 `ContentView` 的 body，它也没有被重新评估。

### 检查 `State`

如果我们在示例代码中用`State`替换`Setting`，一切都按预期工作。这完全不足为奇，因为这是我们*应该*做的事情。不知何故，SwiftUI知道何时`State`发生变化并触发视图更新。为了做到这一点，它必须以某种方式监视基础值的存储……等等，这正是属性包装器的作用！当您调用setter时，您有机会运行自己的代码，而`State`必须使用它告诉SwiftUI有关变化的情况。撇开我们之前关于`nonmutating`的对话，`State`可能看起来像这样：

```
struct State<Value> {
	var _value: Value

	var wrappedValue: Value {
		get {
			_value
		}
		set {
			_value = newValue
			SwiftUI._noteChanges(self)
		}
	}
} 
```

尽管这解释了为什么我们的代码不起作用，但对我们来说仍然是一个问题，因为*我们*不能自己调用这个方法。这是SwiftUI的内部事务，只有`State`（和其他内置类型）知道如何做。但是，我们不必这样做。该框架提供了一个允许解决我们问题的工具：组合属性包装器。

### 使用`DynamicProperty`进行组合

由于只有内置类型知道如何更新UI，SwiftUI允许我们通过*组合*扩展这个系统。如果一个属性包装器内部包含一个`State`，那么对其进行变异可以启动刷新。让我们对属性包装器进行一些更改（并跳过保持不变的部分）：

```
@propertyWrapper
struct Setting<T> {
	let key: String
	let defaultValue: T

	@State
	var value: T?

	init(_ key: String, defaultValue: T) {
		self.key = key
		self.defaultValue = defaultValue
		self.value = UserDefaults.standard.object(forKey: key) as? T
	}

	var wrappedValue: T {
		get {
			value ?? defaultValue
		}
		nonmutating set {
			value = newValue
			UserDefaults.standard.setValue(newValue, forKey: key)
		}
	}
} 
```

请注意增加的`value`，我们返回的新`State<T>`变量。在`wrappedValue`的setter中，我们像以前一样写入用户默认设置，但我们还更新`value`，作为`State`，它可以触发视图刷新。或者说，本来可以的，但我们还没有完全完成。SwiftUI不知道我们放在属性包装器中的`State`：为了让它窥视并连接事物，我们需要将`Setting`符合`DynamicProperty`协议。稍后我们会详细探讨为什么需要这样做，但随着这些变化，此代码最终可行。切换开关会更新UI，并将其写入默认值，这个值跨启动保持不变。成功！

### 滥用依赖关系

尽管我们的设计可行，但有点…不太愉快？“真理的源泉”存在于两个地方：在`State`变量`value`和用户默认设置中。`UserDefaults`非常快速，不需要我们代其缓存值。我们以前的设计更清晰，其中默认设置是备份存储，并且我们直接与其通信。如果`State`的setter副作用是我们所需的全部，我们能不能稍微整理一下代码？如果我们这样做呢？

```
@propertyWrapper
struct Setting<T>: DynamicProperty {
	let key: String
	let defaultValue: T

	@State
	var value: T? = nil

	init(_ key: String, defaultValue: T) {
		self.key = key
		self.defaultValue = defaultValue
	}

	var wrappedValue: T {
		get {
			UserDefaults.standard.object(forKey: key) as? T ?? defaultValue
		}
		nonmutating set {
			value = newValue
			UserDefaults.standard.setValue(newValue, forKey: key)
		}
	}
} 
```

尽管我们将东西存储在`value`中以触发更新，但我们不再将其作为真理的源泉。相反，我们只返回用户默认设置中的内容。在上一个版本中，我们非常注意确保它始终与`value`匹配，因此这应该产生相同的值。

如果你尝试这个版本，你会发现它不再工作了！这背后的原因实际上相当微妙。如果你迄今为止一直在跟进并且想快速解决一个谜题，看看你能否在进入下一节之前调试出发生了什么。提示：尝试在`wrappedValue`的getter中打印`value`。

* * *

我给出的提示有点误导。是的，在`wrappedValue`的getter中打印`value`确实告诉你，它的值与用户默认设置中的值匹配（嗯，直到第一次设置它之前，但你可以把那段代码加回去并检查一下，这并不重要）。但更重要的是，**添加打印语句使其再次工作了！**如果你删除打印语句，它就不再工作了；如果你重新添加它，它又会重新工作。

如果你再测试一下，你会发现重要的不是打印语句，而是`value`本身的访问。即使这段代码也可以工作：

```
get {
	_ = value
	return UserDefaults.standard.object(forKey: key) as? T ?? defaultValue
} 
```

这不是优化器的偶然现象。`State.wrappedValue`的getter实际上有些特殊：它返回值，但它还告诉SwiftUI这个值已经“被读取”。这实际上是SwiftUI如何优化视图更新，只更改UI的部分内容。在调用视图的主体时，它必须保留在此过程中读取的所有状态的列表。如果将来状态发生任何更改，它根据访问该状态的视图来决定哪些视图需要更新，并且会跳过重绘与之无关的视图。对我们来说，这意味着我们不能只在setter中设置`value`：我们还必须在getter中访问它，以向SwiftUI指示我们依赖于它。

这一点很重要，需要注意的是，尽管这种机制很聪明，但也有些粗糙。SwiftUI 并不知道我们对这个值做了什么，只知道我们已经访问过它。它不能知道我们可以对它进行任意计算来生成所有的“派生”属性。事实上，我们更新的`State`类型并不一定要与最终的依赖值匹配（你可以想象一个合理的情况，即`wrappedValue`可能是`State`值的`description`）。而且，我们甚至不必对我们访问的值做任何事情。这是访问的副作用和更新的副作用。你可以认为这比我们正确的基于`State`的解决方案要糟糕得多，但它更加灵活。看起来是这样的（再次跳过冗余部分）：

```
@propertyWrapper
struct Setting<T>: DynamicProperty {
	// Dummy state that SwiftUI thinks we depend on
	@State
	var _update = false

	var wrappedValue: T {
		get {
			_ = _update
			return UserDefaults.standard.object(forKey: key) as? T ?? defaultValue
		}
		nonmutating set {
			_update.toggle()
			UserDefaults.standard.setValue(newValue, forKey: key)
		}
	}
} 
```

## `State` 生命周期

我们的`Setting`相当不错，但它缺少一个重要的功能：用户默认设置可以从外部更新，不仅仅是从我们在应用程序中所做的更改。`UserDefaults`是[KVO兼容的](https://developer.apple.com/documentation/foundation/userdefaults#2926902)，以允许我们响应这些更改。因为我们知道如何自己设置依赖关系，更新我们的代码以处理这一点并不难：

```
@propertyWrapper
struct Setting<T>: DynamicProperty {
	class Observer: NSObject {
		let key: String
		let _update: State<Bool>

		init(key: String, _update: State<Bool>) {
			self.key = key
			self._update = _update
			super.init()
			UserDefaults.standard.addObserver(self, forKeyPath: key, context: nil)
		}

		deinit {
			UserDefaults.standard.removeObserver(self, forKeyPath: key)
		}

		override func observeValue(forKeyPath keyPath: String?, of object: Any?, change: [NSKeyValueChangeKey: Any]?, context: UnsafeMutableRawPointer?) {
			_update.wrappedValue.toggle()
		}
	}

	let observer: Observer

	init(_ key: String, defaultValue: T) {
		self.key = key
		self.defaultValue = defaultValue
		self.observer = Observer(key: key, _update: __update)
	}
} 
```

不幸的是，我们不能使用 Swift 的类型安全的 `KeyPath` 回调 API，因为 `key` 是一个字符串，所以我们需要更多涉及 `NSObject` 子类的样板代码。我们可以传递它我们用于管理更新的 `State`，它可以在值更改时通知 SwiftUI。否则，其他所有东西都可以保持不变。

正如你现在可能已经猜到的那样，这并不起作用。至少这次不会破坏应用内的行为：它只是在外部发生更改时不会更新。虽然 KVO 回调确实被调用，但从内部调用 `_update` 却没有任何可见效果。

### 反射自省

尽管对我们来说 `State` 看起来是不透明的，但它必须在内部维护一些私有数据。至少，它必须有某种身份：即使它是一个结构体，它管理的数据的生命周期比任何特定的 `State` 实例更长。这种状态（双关语未打算）对我们不可见，但它掌握了我们所看到的行为的关键。尽管我们试图在我们的代码中更新“相同”的 `State`，但对于一个更改通过而另一个被丢弃，必须有某些不同之处。

幸运的是，Swift 二进制文件通常包含非常丰富的元数据，可以帮助我们暴露这些内部信息。`Mirror` 使用此元数据支持其反射 API，但调试器也知道如何读取它，从而允许我们深入挖掘我们不拥有的类型的内部。让我们尝试在 `wrappedValue` 中打印 `__update`（注意额外的前导下划线来引用属性包装器本身），我们知道它已准备好发布我们的更改：

```
(lldb) po __update ▿ State<Bool>  - _value : false ▿ _location : Optional<AnyLocation<Bool>>
 ▿ some : <StoredLocation<Bool>: 0x600002f88c00> 
```

如预期，`State` 是由几个属性组成的复合体。`_value` 相当不言自明，我们之前猜测它的存在。然而，另一个属性 `_location` 则有些神秘。我们可以使用 LLDB 进一步深入挖掘它（例如，我们可以看到有一个名为 `_wasRead` 的标志，以及与 AttributeGraph 的连接），但在深入之前，让我们在 `Observer.observeValue(forKeyPath:of:change:context:)` 中做同样的转储：

```
(lldb) po _update ▿ State<Bool>  - _value : false
  - _location : nil 
```

啊哈！这次，`_location` 是 `nil`。这个 `_location` 必须（除其他事项外）包含回到 SwiftUI 的连接，用于在通知时使用。因为在这一点上它还没有设置，所以没有人在那里监听我们的更新。

值得考虑一下这种情况是如何发生的。尽管 `Setting` 和 `Observer` 都使用了“相同”的 `State`，但它们实际上并没有得到 `_update` 的一致视图。正如前面提到的，`State` 是结构体，所以当 `Observer` 初始化时，它会得到状态的 *拷贝*，这是在 `Setting` 初始化时。如果我们在该初始化程序中设置断点，我们可以看到此时 `_location` 是 `nil`。`Observer` 的本地副本是在那时创建的，并且从未看到任何更改。另一方面，`Setting` 的 `_update` 在此时也有一个 `nil` 的 `_location`，但在未来的某个时刻，这将变为有效值。是谁在做这件事？何时以及如何做的？

当然，答案是 SwiftUI 为我们做了这些。在计算视图主体之前不久，它会遍历并在所有相关的`State`变量上设置`_location`，以便进行依赖跟踪时准备就绪。通常，属性包装器没有能力从自身外部获取上下文（例如，通过查找谁拥有它）。SwiftUI 可以像我们一样使用反射来发现需要在上面安装`_location`的`State`成员，从而避开此问题。为了发现嵌套类型中的`State`，它需要一点帮助：这就是为什么我们之前必须添加`DynamicProperty`一致性的原因。在这种情况下，它使用反射来查找`DynamicProperty`成员，然后在其中查找`State`。

SwiftUI 在我们的`Setting`属性包装器内部执行相同的操作，当拥有它的视图进行更新时。这意味着我们需要在这时设置`Observer._update`，而不是在初始化器中。幸运的是，框架在恰到好处的时机调用[`DynamicProperty.update()`](https://developer.apple.com/documentation/swiftui/dynamicproperty/update()-29hjo)，让我们能够做到这一点。让我们更新`Setting`以使用它：

```
@propertyWrapper
struct Setting<T>: DynamicProperty {
	let key: String
	let defaultValue: T

	@State
	var _update = false

	class Observer: NSObject {
		let key: String
		var _update: State<Bool>!

		init(key: String) {
			self.key = key
			super.init()
			UserDefaults.standard.addObserver(self, forKeyPath: key, context: nil)
		}

		deinit {
			UserDefaults.standard.removeObserver(self, forKeyPath: key)
		}

		override func observeValue(forKeyPath keyPath: String?, of object: Any?, change: [NSKeyValueChangeKey: Any]?, context: UnsafeMutableRawPointer?) {
			_update.wrappedValue.toggle()
		}
	}

	let observer: Observer

	init(_ key: String, defaultValue: T) {
		self.key = key
		self.defaultValue = defaultValue
		self.observer = Observer(key: key)
	}

	func update() {
		observer._update = __update
	}
} 
```

因此，外部更新（例如使用`defaults write`）也会更新视图。有了这个，我们的`Setting`功能完善且可以在 SwiftUI 中使用。这是我们为参考创建的最终版本：

虽然直接在您的应用中使用这段代码可能不值得（请注意上面的免责声明！），但理解它的工作原理和原因可能有助于调试您自己的 SwiftUI 项目。
