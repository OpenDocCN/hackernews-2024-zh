<!--yml

category: 未分类

date: 2024-05-27 15:04:52

-->

# 哎呀，我的应用很成功，但我没有考虑到可访问性。

> 来源：[https://jacobbartlett.substack.com/p/oh-sht-my-app-is-successful-and-i](https://jacobbartlett.substack.com/p/oh-sht-my-app-is-successful-and-i)

当你有一个天才的应用创意、一份全新的工作或者一笔新的风险投资时，只专注于发布新功能的诱惑可能会无法抵挡。

如果你做得聪明，这种专注将带来回报，将你的应用推向下载榜的顶端。*这还不够好*。直到*每个人*都在使用你的应用之前，你都不会停止。

达到这个荒谬的里程碑后，你意识到一个残酷的事实：6 个用户中有 1 个认为你的应用有问题。

然后，你突然领悟到。

> 哎呀。
> 
> 我的应用很成功，但我没有考虑到可访问性。

16% 的人有某种形式的可访问性要求（如果你是潮流的一员就叫做 a11y）。但当你处于繁忙的工作期限中，并且领导层的支持有限时，很容易让 a11y 被忽视。

你越成功，你就越被认为是因为未能承认用户对 a11y 的需求而被批评。不幸的是，没有有影响力的人支持 a11y，这很难成为优先事项。

今天，我来帮忙。我将演示如何快速提高应用的无障碍性能：

+   审核你的 SwiftUI 应用的 a11y。

+   使应用在所有文字缩放下看起来都很好。

+   使应用能够被屏幕阅读器使用。

+   说服利益相关者优先考虑 a11y。

我们将看看我专门为这篇文章创建的一个[超级简单的、以猫为主题的伴侣应用](https://github.com/jacobsapps/oh-shit-a11y)。这款应用在表面上看起来很好，但一旦你从 a11y 的角度进行评估，就会*彻底失败*。

如果你想完全理解这些技术，**也可以跟着编码！** 从`Before/`文件夹开始，因为我们会在这篇文章中详细介绍所有的技术（查看`After/`获取最终改进的应用）。

要真正了解 a11y，你应该在真实设备上测试可访问性，因为这是所有用户与你的应用交互的方式。

在任何 a11y 的工作中，你应该首先优化控制中心。*真的*，这能大大节省时间。

进入 iPhone 设置 → 控制中心。添加文字大小控制和无障碍快捷方式。文字大小允许你从任何应用的任何地方向下滑动，并立即选择要应用的文本缩放 —— 你甚至可以按应用程序的基础应用大小来应用大小。

修改控制中心以包含文字大小调整

无障碍快捷方式允许你即时应用各种 a11y 功能，如辅助触控、VoiceOver、色彩过滤和减少动态效果。然而，**三次按锁定按钮**可能更容易访问，它显示与操作表相同的菜单。

iOS 有12种可能的文本大小，从`extraSmall`到`accessibilityExtraExtraExtraLarge`（也称为`AX5`），后者大约比默认的`large`字体大310%。

> *这是`body`字体的缩放，但是不同系统的文本大小按不同比例缩放，例如`largeTitle`已经相当大，因此不会像`caption`那样缩放得那么多。
> 
> 这一点以后会变得很重要！

为了确保文本大小适用，我喜欢进行两项审计：

1.  在最大非无障碍字体`extraExtraExtraLarge`（也称为`xxxl`）上测试应用，最好还启用*Bold Text*辅助设置。这是相对常见的最大字体，并且也是我妈妈使用的字体。应用在这个尺寸下应该看起来*很好*。

1.  在最大的a11y字体`AX5`上测试应用。如果你不习惯，可能会觉得看起来太大，但对于一些用户来说，这是他们唯一与你的应用交互的方式。虽然在这个尺寸上完美展现UI很困难，但如果一个公司连检查应用是否适用于无障碍文本缩放都懒得做，那就显而易见了。

让我们来运行一下我们的应用。

用大（默认）、xxxLarge（和粗体）和AX5文本缩放审计我的登录界面。

立刻发现，我们精心设计的登录界面在应用文本缩放时失败了。我们做得太草率了，没有预料到更大的文本尺寸——所以从未考虑过使内容可滚动。

更重要的是？有色盲的人可能无法阅读登录按钮，因为对比度微乎其微。

让我们也来看看我们应用的主列表界面。

用大（默认）、xxxLarge（和粗体）和AX5文本缩放审计我的列表界面。

这个相对来说不那么‘坏’，因为你可以看到所有内容，但是对于使用较大字体的人来说，很明显你没有在这个尺寸上进行测试——只需看看文本标签争夺空间，或者那个微小的❤图标与巨大的标题。

我们的一些用户不受文本缩放影响——因为他们可能需要屏幕阅读器来与你的应用交互。

iOS VoiceOver在这里处理繁重工作做得非常好，因此你的团队稍加思考就会大有帮助。

屏幕阅读器审计，如文本大小审计，最好在真实设备上完成（按锁定按钮3次，记住！）。Xcode中还有一个开发者工具内置的无障碍检查器，可以帮助使用VoiceOver。

在Xcode中找到无障碍检查器开发者工具。

浏览应用时，相比于文本缩放，它的问题没有那么严重，但是我们的一些UI大面积区域，比如我们的图像，对我们的用户来说完全看不见，因为我们没有添加无障碍标签。

> 我在这里故意使用了敌对的语言，因为这就是用户会如何看待你的应用的原因。敌对。

屏幕阅读登录界面尽其所能，显示图像资源的名称。

当某人没有考虑到 VoiceOver 时，每个 UI 元素都由屏幕阅读器逐个迭代，包括未标记的图标，这一点变得格外清晰。

屏幕阅读器的懒惰使用会导致每个 UI 元素被单独列出。

当有一个大列表时，情况变得更糟。

如果你不给 SwiftUI 提供关于你的内容的信息，可能会出现这样一种情况：要移动到可滚动内容底部的注销按钮，需要滑动数百次。而且*别让我开始*分页问题。

对于许多项目，要到内容底部需要很长时间。

显然，在我们能称我们的应用程序是可访问的之前，我们需要解决相当多的问题。幸运的是，SwiftUI 中提供了丰富的工具，可以让您迅速掌握 a11y 相关知识。

现在我们知道问题出在哪里，我们可以开始逐个解决它们。而且速度很快。

这是最令人反感的问题 —— 我的应用程序入门完全出问题了，因为内容在更大的文本尺寸上没有展开成滚动视图。

在我们的审核中，登录界面的按钮在字体大小增加时被推到屏幕外。其余文本内容相互紧贴，标签重叠并因无法扩展而截断。

自然的解决方案是使内容可滚动。但是我们不想让每个屏幕的内容都在内容适合的情况下随意滚动。我们不从事构建 Ionic 应用的业务 **由于被压抑的回忆而颤抖**。

可以通过自定义视图修饰符解决这个问题，我称之为`a11yScrollView()`。这个视图修饰符的目标是在内容需要时将内容包裹在滚动视图中 —— 只有当内容不适合时才会变得可滚动。我已经将其添加到一个小型库[A11yUtils](https://github.com/jacobsapps/A11yUtils/blob/main/Sources/A11yUtils/A11yScrollView.swift)，这样您可以在自己的代码中使用它。

目前，这仅在 iOS 16.4 及以上版本中实现了我们梦想中的行为 —— 将`scrollBounceBehavior`应用于滚动视图，以防止如果内容已适合则无法滚动。

在未来，我希望使其与 iOS 16 的`ViewThatFits*`和 iOS 15 的`GeometryReader`配合使用。

> **[随时提交拉取请求！](https://github.com/jacobsapps/A11yUtils/tree/main)**

既然没人在身边要求我针对哪个操作系统版本进行目标，我可以将此修饰符应用于我们入门视图中的`VStack`。

```
// OnboardingView.swift

import A11yUtils
import SwiftUI

struct OnboardingView: View {

    @Binding var isLoggedIn: Bool

    var body: some View {
        NavigationStack {
            VStack(spacing: 20) { /* ... */ }
              .padding(.horizontal)
              .a11yScrollView()
              .navigationTitle("Create account")
        }
    }
}
```

现在，内容可以很好地滚动 —— 在`xxxl`，`AX3`和`AX5`上看起来不错。

我们的内容现在在所有文本缩放上都可以滚动。为了保险起见，我修复了登录按钮的对比度。

> *根据我的经验，`ViewThatFits`方法效果不错，但在文本输入方面存在问题 —— 软键盘实际上会更改视图的大小，导致内容重新调整大小和布局引擎重绘内容。如果`@FocusState`在出现时切换键盘，这将导致无限重绘和调整大小的循环！

当我们正在处理入门屏幕时，还有另一个简单的改进可以做：

```
VStack(spacing: 20) {
    // ...
    OnboardingReasonsText()
    Spacer()
    LoginButtonView()
}
```

当文本大小较大时，限制屏幕实际空间时，`Spacer()` 可能会导致问题。SwiftUI 布局引擎将尽力创建空间，即使没有空间，也会导致文本被截断。

在这种情况下，即使将 spacer 压缩到非常窄的高度，`VStack` 中定义的上下都会有 20 个点。

我们可以将 `Spacer()` 替换为 `frame()` 修饰符：

```
VStack(spacing: 20) {
    OnboardingReasonsText()
    LoginButtonView()
        .frame(maxHeight: .infinity, alignment: .bottom)
}
```

框架的行为更可靠：SwiftUI 知道在此按钮上方尽可能提供尽可能多的空间，但很明显，空间本身并不是 UI 的关键部分，因此如果没有空间，就不会应用不必要的间距。

这个 `frame` 修饰符有一个更明确的*语义*意义 — 即，它让我们在 SwiftUI 中的意图变得清晰。这在讨论 a11y 时是一个关键概念，我们稍后会详细介绍。

由于动态类型，iOS 将根据用户选择的字体大小自动调整我们应用中的字体。这在使用[自定义字体](https://developer.apple.com/documentation/uikit/text_display_and_fonts/adding_a_custom_font_to_your_app)时也相当直观易懂。

这意味着当我们在 SwiftUI 中使用语义字体大小时，文本将自动缩放。

```
Text(cat.quote)
    .font(.body) // this works fine
```

更好的是，当坚持使用 SFSymbols 时，我们可以在我们的图标上获得动态类型！

```
Image(systemName: "heart.circle.fill")
    .font(.body) // this also works fine with SFSymbols
```

不幸的是，大多数 iOS 应用程序往往有对应的 Android 版本，这意味着您的设计师将创建自定义图标，而不是让您在各处使用 SFSymbols。大多数应用程序还将包含自定义图像和媒体。

因此，在我们的 `Before/` 示例中，我把这个图标看作是我们在 SwiftUI 中添加的任何常规图像，使用了硬编码大小。

```
Image(cat.image)
    .resizable()
    .aspectRatio(contentMode: .fit)
    .frame(width: 72, height: 72) // hardcoded size - it won't scale!
    .clipShape(Circle())

Image(systemName: cat.icon)
    .resizable()
    .aspectRatio(contentMode: .fit)
    .foregroundStyle(.secondary)
    .frame(width: 24, height: 24) // hardcoded size - it won't scale!
```

结果是？在最大的字体缩放下，内容看起来非常不匹配。

图像和固定大小的图标在较大的文本缩放下看起来很奇怪。

SwiftUI 提供了一个巧妙的工具，使我们可以在自定义图像和视图中使用动态类型：`@ScaledMetric` 属性包装器。

```
@ScaledMetric(relativeTo: .largeTitle) private var imageSize: CGFloat = 72
@ScaledMetric(relativeTo: .body) private var iconSize: CGFloat = 24
```

我们可以单独使用 `@ScaledMetric`，它使用默认的 `body` 字体缩放。最好使用 `relativeTo`，这样 SwiftUI 就知道如何缩放它。

`largeTitle` 字体尺寸略有缩放，因为它已经相当大了。`caption` 样式的字体尺寸会大得多。在这个例子中，图标的 `body` 缩放使其与伴随的引用文本匹配。

```
Image(cat.image)
    .resizable()
    .aspectRatio(contentMode: .fit)
    .frame(width: imageSize, height: imageSize) // dynamically scales
    .clipShape(Circle())

Image(systemName: cat.icon)
    .resizable()
    .aspectRatio(contentMode: .fit)
    .foregroundStyle(.secondary)
    .frame(width: iconSize, height: iconSize) // dynamically scales
```

现在，我们的图像在文本旁边看起来更合理。

使用 @ScaledMetric 进行图像缩放

图像缩放得更好，但我们的 UI 仍然明显有问题，文本标签争夺空间。在与布局引擎进行详尽的协商后，它们最终充满了换行符。

**如果我们能够根据用户的文本缩放来对齐内容会怎样？**

幸运的是，多亏了我的库 [A11yUtils](https://github.com/jacobsapps/A11yUtils/blob/main/Sources/A11yUtils/A11yHStack.swift)，我们可以利用 `A11yHStack`。

由于SwiftUI工具的快速发布速度（以及苹果糟糕的向后兼容性），基于您的最低支持的操作系统，这也有一些不同的实现：

+   在`iOS 16`上，它使用`ViewThatFits`来检查`HStack`是否能容纳所有内容，如果不能，则使用`VStack`来重新排列内容。

+   在`iOS 15`上，它采用了更直接的方法，检查`@Environment(\.sizeCategory)`，并在使用辅助功能文本大小类别（如`AX1`、`AX2`或`AX5`）时切换到`VStack`。

这是一个简单的方法：

```
@Environment(\.sizeCategory) private var sizeCategory

var body: some View {
    if sizeCategory.isAccessibilityCategory {
        VStack(alignment: .leading, spacing: spacing) {
            content()
        }
        .frame(maxWidth: .infinity, alignment: .leading)
    } else {
        HStack(alignment: alignment, spacing: spacing) {
            content()
        }
    }
}
```

`@Environment(\.sizeCategory)`是一个有用的工具，在需要知道是否使用辅助功能文本大小类别（使用`isAccessibilityCategory`）的情况下特别重要。有时，我使用这个属性来移除在大文本尺寸下根本不适合的不必要的UI元素，比如图片。

更现代的实现方式类似，但利用了`ViewThatFits`，因此只有当内容不适合时才会重新分配：

```
var body: some View {
    ViewThatFits {
        HStack(alignment: alignment, spacing: spacing) {
            content()
        }
        VStack(alignment: .leading, spacing: spacing) {
            content()
        }.frame(maxWidth: .infinity, alignment: .leading)
    }
}
```

如果我们深思熟虑，我们可以在几个地方应用`A11yHStack`：

```
import A11yUtils 

A11yHStack {
    Image(cat.image)
    VStack {
        A11yHStack {
            Text(cat.name)
            Text("\(cat.age) years old")
        }
        HStack {
            Image(systemName: cat.icon)
            Text(cat.quote)
        }
    }
}
```

现在我们的内容在非常大的文本大小上看起来更合理了。

在列表单元中使用A11yHStack来利用我们的内容。

> 另一个小的警告：确保在所有支持的iOS版本上进行测试，因为我发现`ViewThatFits`方法会产生我意料之外的行为。在开始时，重新实现`@Environment(\.sizeCategory)`方法可能会更容易些。

VoiceOver是解决难题的一个相当简单的部分。

确保您的应用与屏幕阅读器良好兼容意味着：

+   确保视觉内容得到充分描述。

+   使导航工作更直观。

+   确保我们的视图具有*语义*意义。

让我们从视觉描述开始。这是我们工具箱中最简单的修复措施：确保我们的图形内容可以被VoiceOver描述。`accessibilityLabel`修饰符是我们的基本工具。

```
Image("catKingdom")
    .accessibilityLabel(Text("My three cats: Cody, Rosie, and Luna"))
```

这告诉屏幕阅读器如何解释它当前聚焦的内容。

为引导图像应用accessibilityLabel

接下来，我们可以在主视图的单元上应用相同的方法。通过对我们的`Cat`数据模型进行小的修改，我们为每只猫的照片提供图像描述。

```
struct Cat {
    // ...
    let imageDescription: String
}

Image(cat.image)
    .accessibilityLabel(Text(cat.imageDescription))
```

这一点听起来很容易，并且在苹果平台上*只是起作用*。

通过数据模型为所有图像应用VoiceOver描述

当涉及到通过我们的应用进行导航时，我们有机会更加周到。默认情况下，VoiceOver将遍历SwiftUI视图树中的每个叶子节点，并朗读所有我们的`Image`、`Button`和`Text`。

有时，这会导致用户进行许多不必要的导航。在我们的应用中，我们强制用户点击多次才能导航到每只猫的图标和引用。朗读图标的名称（如`cat.fill`）对体验毫无帮助。

我们可以将它们结合在一起作为一个单一的无障碍元素使用 `accessibilityElement(children:)` 修饰符，这样屏幕阅读器在导航时可以将它们视为一个项目。

```
HStack {
    Image(systemName: cat.icon)
    Text(cat.quote)
}
.accessibilityElement(children: .combine)
```

现在我们可以听到屏幕阅读器将元素作为一个项目移动：

使用 `accessibilityElement(children:)` 结合 VoiceOver UI 元素

在某些情况下，您可能会有一个非常复杂的 UI，例如一个[图形化倒计时的风格化计时器元素](https://jacobbartlett.substack.com/i/141732200/image-loading-bug)。VoiceOver 很聪明，但它无法从一堆未解释的视图中推断出任何有用的信息。

这就是 `accessibilityRepresentation` 修饰符非常有用的地方。它允许您用完全定制的无障碍表示替换视图的 VoiceOver UI。

这对我最新的独立项目 [Check ’em](https://jacobbartlett.substack.com/p/building-a-2fa-app-that-detects-patterns) 特别有用，用户的 2FA 代码显示为倒计时：

```
// AccountView.swift

struct AccountView: View {

    var body: some View {
        Section(account.name) {
            // Image, 2FA Code Text, and Countdown UI ...
        }
        .accessibilityRepresentation {
            VStack {
                Text(account.name)
                if let code = account.code {
                    ForEach(Array(code.enumerated()), id: \.0) {
                        Text(String($0.1))
                    }
                }
                if let countdown = account.countdown {
                    Text("Expires in \(countdown)")
                }
            }.accessibilityElement(children: .combine)
        }
    }
```

此修饰符使我能够引入一个完全定制的 VoiceOver 界面，比按顺序迭代单元格的每个元素要有用得多。它还防止屏幕阅读器将 676,252 读作 *六十七万六千二百五十二*。

在 Check ’em 中使用的 accessibilityRepresentation 修饰符，我的独立项目

此应用的主视图采用了常见但松散的布局方法：将我们的单元格包裹在 `ForEach` 中，再包裹在 `LazyVStack` 中，最后包裹在 `ScrollView` 中。

```
ScrollView {
    LazyVStack(spacing: 24) {
        ForEach(cats, id: \.name) {
            CatView(cat: $0)
        }
    }
}
```

在这种情况下，最佳方法是**尽可能使用原生 SwiftUI 组件**。在这种情况下，使用 `List` 是最合适的。

苹果使得这比必要的要困难一些 —— 令人费解的是，要使用我们自己的自定义 UI 而不是 iOS 设置风格的 `List`，我们需要添加 3 个单独的修饰符。

```
List(cats) {
    CatView(cat: $0)
        .listRowBackground(Color.clear)
        .listRowSeparator(.hidden)
}
.listStyle(PlainListStyle())
```

有了这些样板文件，修饰符现在允许我们在 SwiftUI 的 `List` 中自由使用我们自己的 UI。

默认列表（左）和我们自定义列表（右）使用了 .listStyle 和 .listRow 修饰符

一个问题。

**为什么要费这么大劲去使用本地组件？**

首先，非无障碍原因：SwiftUI 的 `List` 是使用 `UICollectionView` 实现的。这利用了单元格复用来实现高性能滚动，即使有很多项也不例外。相反，`LazyVStack` 将所有先前渲染的单元格保留在内存中，这意味着如果您滚动许多项，性能将急剧下降。

> 这些信息来自 [Thomas Ricouard](https://medium.com/u/b067c9115d59) 和他的朋友们；阅读 [SwiftUI: List 与 LazyVStack 的区别](https://dimillian.medium.com/swiftui-the-difference-between-list-and-lazyvstack-3d5eeaccb156) 了解更多！
> 
> 如果您想看更多关于应用性能的内容，请查看我的最新深度分析，[高性能 Swift 应用](https://jacobbartlett.substack.com/p/high-performance-swift-apps)。

使用本地组件的与无障碍相关的原因是简单而又多样的：

> **苹果公司已经集体投入更多的思考来使无障碍性工作，远远超过你能想象的。**

`List`具有*语义意义*：它告诉屏幕阅读器它是一个容器，包含一组相似的内容。

这就是为什么当我们有很多列表项时，天真的实现如此糟糕：不是让我们通过VoiceOver跳过`List`容器，而是让我们必须逐个滑动每个项目，才能到达*登出*按钮。

通过`List`，SwiftUI可以告知屏幕阅读器其内容是单个容器，使屏幕阅读器能够轻松地跳过。

当使用VoiceOver时，您可以通过将导航增量设置为“容器”并向前划过内容，跳过`List`。

与使用本机组件时*仅仅工作*的多种内置交互模式相比，`List`还有许多，如滑动删除、拖放和基于键盘的导航。当您使用原生组件时，这些功能会*自动生效*。你有没有想过如何让屏幕阅读器与您笨拙的自定义实现一起工作？

> 在您需要回应设计师最新创意时，请参考上述段落。

`List`甚至不是最复杂的组件。您如何为自定义的`Grid`或不使用`Layout`协议的视图分布实现无障碍性？

现在您知道如何在您的SwiftUI应用程序中快速实现无障碍性之前，开始工作之前，拼图的最后一块是让整个业务都认同这一点。这是您可以展示您软技能*行使组织影响力*的地方。

一个大而粗糙的工具，您可以运用的是立法的锤子。世界各国如[英国](https://www.legislation.gov.uk/uksi/2018/852/contents/made)，[美国](https://www.section508.gov/manage/laws-and-policies/)和[欧盟](https://www.levelaccess.com/compliance-overview/european-accessibility-act-eaa/)都已实施了法律，要求数字服务符合最低无障碍标准。

此外，无障碍性对业务有重大的好处。全球16%的人口目前有明显的残疾，这意味着如果考虑到更广泛的需求，您可能会错过用户、正面评价和收入。根据您的目标人群，这可能更或者不那么关键。

在较小的范围内，我发现在处理作为产品债务的无障碍性时，简单地*展示应用程序如何存在问题*，比单纯地敲击警钟要有效得多。将对无障碍性的需求从抽象中剥离出来，并展示有错误的注册流程，对任何工程组织来说都更具说服力。

没有比领导层中的有影响力的支持者更有帮助的了，他们致力于使产品易于访问。当然，每个组织都不同，因此在应用此建议时，您的效果可能会有所不同。

现在，您已经快速完成了无障碍操作，您的产品完全可以访问？

**不要掉以轻心！**

就像技术债务一样，现在您处于一个良好状态，您可以将无障碍集成到您的标准开发工作流程中。现在你知道怎么做，这很快很容易！

今天我们涵盖了很多内容。

首先，我们经历了审核您的应用程序的过程，以查找在使用大文本缩放或VoiceOver时出现的常见无障碍错误。

我们研究了在使用文本缩放时出现的问题，使用我们的`a11yScrollView`进行滚动，使用`@ScaledMetric`进行内容缩放，以及使用`A11yHStack`进行对齐。

为了使我们的应用在屏幕阅读器上运行良好，我们实现了`accessibilityLabel`s，结合了`accessibilityElement`s，甚至完全定制了`accessibilityRepresentation`s。

我们讨论了坚持使用原生SwiftUI组件而不是自定义视图的原因，例如语义、交互模式和性能。

最后，我们讨论了如何运用软技能来说服您的组织认真对待无障碍问题。

如果您在SwiftUI应用程序中真的关心a11y，我恳请您通过[伴侣应用程序](https://github.com/jacobsapps/oh-shit-a11y)逐步实施这些技术，这真的是内化工具的最佳方式。如果您有一个正在进行的副项目，那也是一个很好的开始。

我很高兴接受对我的[A11yUtils](https://github.com/jacobsapps/A11yUtils/blob/main/Sources/A11yUtils/A11yHStack.swift)库的贡献（和问题） — 我很愿意与社区合作，将其转变为一个全面的无障碍工具套件，以补充已经在SwiftUI中的API。

* * *
