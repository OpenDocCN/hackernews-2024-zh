<!--yml

category: 未分类

日期：2024-05-29 12:42:25

-->

# 告诉你何时获取`012345`的2FA应用程序

> 来源：[https://jacobbartlett.substack.com/p/building-a-2fa-app-that-detects-patterns](https://jacobbartlett.substack.com/p/building-a-2fa-app-that-detects-patterns)
> 
> 如果你喜欢应用程序但讨厌阅读，赶快下载**[Check ’em: The Based 2FA App](https://apps.apple.com/app/check-em-the-based-2fa-app/id6477842236)**！

就像所有在2010年代初成年的恢复的edgelords一样，我有点怀念像4chan这样的图像板的鼎盛时期。在纳粹毁掉一切之前，它们是早期互联网野西部的最后堡垒。

一个经典的梗是[GET](https://knowyourmeme.com/memes/get)，你会因为正确预测随机生成的帖子ID包含一系列有趣数字而感到无比自豪。

如今，现在[所有平常人都已经长大并找到工作了](https://www.wired.co.uk/article/moot-joins-google)，我们接近昔日魔法的最近体验就是多因素认证代码了。

如果你知道，*你就知道*。

必须重新认证你的银行、电子邮件或云服务的繁琐工作。当你得到像`787000`或`123450`这样的*好*数字时，会带来一丝小小的喜悦。

灵感来了。

这些多因素认证（MFA）代码使用一个常见的算法，每30秒刷新一次。我们只接触到6位认证代码中可能的一小部分。

就像[我所有的独立项目](https://jacobbartlett.substack.com/p/my-toddler-loves-planes-so-i-built)，我有一个清晰的愿景，围绕这个愿景我可以建设：

> 如果你的2FA应用告诉你每次出现酷炫数字时，会怎样？

我知道我该做什么。

我不需要太多的动作来弄清楚这是否有效。

如果概念——在出现酷炫的2FA数字时收到通知——可行，那么我可以通过几个关键功能将其打造成一个真正的应用程序：

+   使用摄像头捕捉两步验证（2FA）的秘钥。

+   存储多个2FA代码

+   实现更多数值模式。

+   允许用户选择他们想要了解的模式。

我知道我找对了方向：我向90%的人解释时他们觉得我是个白痴。而剩下的10%只看到了纯粹的独创性。

对于一些人来说是白痴，对于其他人来说是天才。

[TOTP](https://www.protectimus.com/blog/totp-algorithm-explained/)，或者*基于时间的一次性密码*，是一个惊人简单的概念。它是一种使用两个输入的认证过程：

+   一个秘密密钥，存储在认证服务和您自己的设备上。

+   当前时间，或者更具体地说，自Unix时间以来经过的30秒间隔的数量。

一个算法通过这两个输入确定性地哈希以生成你熟悉和喜爱的6位代码。这个哈希算法相当标准，在苹果的CryptoKit中找到。感谢我们在[苹果论坛](https://forums.developer.apple.com/forums/thread/120918)的朋友们，这里是完整的TOTP算法：

```
// CodeGenerator.swift

private let secret = Data(base64Encoded: "AAAAAAAAAAAAAAAAAAAAAAAAAAA")!

func otpCode(date: Date = Date()) -> String {
    let digits = 6
    let period = TimeInterval(30)
    let counter = UInt64(date.timeIntervalSince1970 / period)
    let counterBytes = (0..<8).reversed().map { UInt8(counter >> (8 * $0) & 0xff) }
    let hash = HMAC<Insecure.SHA1>.authenticationCode(for: counterBytes, using: SymmetricKey(data: secret))
    let offset = Int(hash.suffix(1)[0] & 0x0f)
    let hash32 = hash
        .dropFirst(offset)
        .prefix(4)
        .reduce(0, { ($0 << 8) | UInt32($1) })
    let hash31 = hash32 & 0x7FFF_FFFF
    let pad = String(repeating: "0", count: digits)
    return String((pad + String(hash31)).suffix(digits))
}
```

为了确保这样做正确；我在我的[Google](https://myaccount.google.com/)帐户上设置了 2FA，并在我的应用程序中显示了秘密，使用这个算法。

巧合的是，我得到了一个非常好的代码来确认我的 2FA 设置。

并且，就像魔术一样（在一些恼人的 base32 到 base64 转换之后），Google 接受了我的 2FA！

现在我们已经有了我们的 2FA 的基本架构，我们可以实现最后一块拼图的证明：生成通知。

我们的关键限制在于我们的移动设备。

我们实际上不能永远保持像 2FA 生成这样的后台进程运行，而且*当然*不能将用户秘密存储在后端推送服务器上。

因此，为了让这个概念起作用，我们必须稍微狡猾：预先计算未来的 2FA 代码，并安排在它们在现实生活中出现的时间发送。

此外，我们在 iOS 上一次只能安排 64 次推送，因此我们应该：

1.  保存几个通知，要求用户重新进入应用。

1.  鼓励用户通过点击通知打开应用，切换重新计算 2FA 代码。

现在我们知道我们的 POC将如何工作，让我们开始构建吧。

让我们为我们的低调 2FA 代码增添些色彩吧。

我们计划预先计算许多代码，然后实现某种正则表达式来检测每个代码是否是一个值得检查的 GET。

我超简单的 SwiftUI 视图可以方便地显示这些代码，使用基于 `UICollectionView` 的 `List` 来确保良好的性能（简单的 `ScrollView` 中的 `VStack` 在处理超过 10,000 项之前就会开始出现问题！）。

```
// ContentView.swift

struct ContentView: View {

    var body: some View {
        List {
            ForEach(makeOTPs(), id: \.self) {
                Text($0)
                    .fontDesign(.monospaced)
                    .font(.title)
                    .kerning(4)
            }
            .frame(maxWidth: .infinity)
        }
    }

    func makeOTPs() -> [String] {
        (0..<10_000).map {
            otpCode(increment: $0)
        }
    }
}
```

目前看起来很不错。

初始的 2FA 代码列表。

现在，我们可以添加一个简单的基于正则表达式的评估器来检查行程——也就是包含三个匹配数字序列的 TOTP，例如 `120333`。

```
extension String {
    func checkThoseTrips() -> Bool {
        (try? /(\d)\1\1/.firstMatch(in: self)) != nil
    }
}
```

我们为我们的 `Text` 视图添加了一个 `fontWeight` 修饰符，以便在滚动时轻松检测这些 GET。

```
Text($0)
    .fontWeight($0.checkThoseTrips() ? .heavy : .light)
```

哦，如此神奇！*检查这些行程*！

我们甚至可以对我们的正则表达式进行基本修改，以检测到受尊敬的*四张*——我会把这留给读者来练习。

我们粗心的 `ForEach` 实现导致了一个警告：

```
ForEach<Array<String>, String, Text>: the ID 312678 occurs multiple 
times within the collection, this will give undefined results!
```

我们确实得到了几十次这样的警告！

在这里将代码字符串用作视图标识是一个坏主意。

自从我们生成了 10,000 个 OTP，几乎可以肯定有几个是匹配的——这类似于[生日问题](https://en.wikipedia.org/wiki/Birthday_problem)，即可能匹配的对数远远超过一百万。

让我们开始计算一些有趣的代码吧。

这里的关键在于预先计算，预见未来：TOTP 是秘密和日期输入的确定性哈希。因此，我们可以提前确定未来的长序列日期，以确定您在何时看到哪个 OTP 代码。

让我们调整我们的 OTP 生成来返回代码和日期：

```
// TOTP.swift

struct OTP {
    let date: Date
    let code: String
}

func otpCode(date: Date = Date(), increment: Int = 0) -> OTP {
    let digits = 6
    let period = TimeInterval(30)
    let adjustedDate = date.addingTimeInterval(period * Double(increment))
    let counter = UInt64(adjustedDate.timeIntervalSince1970 / period)
    let counterBytes = (0..<8).reversed().map { UInt8(counter >> (8 * $0) & 0xff) }
    let hash = HMAC<Insecure.SHA1>.authenticationCode(for: counterBytes, using: SymmetricKey(data: secret))
    let offset = Int(hash.suffix(1)[0] & 0x0f)
    let hash32 = hash
        .dropFirst(offset)
        .prefix(4)
        .reduce(0, { ($0 << 8) | UInt32($1) })
    let hash31 = hash32 & 0x7FFF_FFFF
    let pad = String(repeating: "0", count: digits)
    let code = String((pad + String(hash31)).suffix(digits))
    return OTP(date: adjustedDate, code: code)
}
```

为了测试这一点，让我们生成大量这些代码，并搜索 GET 的全套：*五张*。

```
func interestingCodes() -> [OTP] {
    (0..<1_000_000)
        .map { otpCode(increment: $0) }
        .filter { $0.code.checkThoseQuints() }
}
```

在我的 M1 运行散列函数进行了一些数字计算——大约 30 秒左右——我们得到了一些可以认真检查的 GETs。

这真是……太美了。检查它们吧。

看到伟大数字很有趣，但是如果您不能真正将这些GET应用于现实生活中的实际认证，这款应用的概念就不比一个随机数生成机器更好。

现在我们知道有趣数字何时到达，我们想要排队一个推送通知，以便我们即时捕捉到这个数字：

```
// NotificationScheduler.swift

private func createNotification(for otp: OTP) {
    let center = UNUserNotificationCenter.current()
    let content = UNMutableNotificationContent()
    content.title = "Quads GET!!"
    content.body = otp.code
    content.sound = UNNotificationSound.default
    let components = Calendar.current.dateComponents([.year, .month, .day, .hour, .minute, .second], from: otp.date)
    let trigger = UNCalendarNotificationTrigger(dateMatching: components, repeats: false)
    let request = UNNotificationRequest(identifier: UUID().uuidString, content: content, trigger: trigger)
    center.add(request) { (error) in
        // ... 
    }
}
```

这些是在生成我们视图中使用的“有趣代码”后立即安排的。不久后，我一次收到了2条精彩的推送通知！

我依然每次获得一个订阅者时都会告诉我的妻子。

当我确认这个通知确实对应于现实中出现的数字时，这变得更加令人兴奋！

在2FA应用程序本身查找四联

这款应用现在已经超越了随机数生成器：这段代码确实可以用来登录我的谷歌账户。

要确定不同类型的有趣数字，我们需要引入“有趣性”的概念。这可能包括但不限于一些潜在的数字序列类型：

这些有趣的数字类型可以枚举为……嗯，作为一个枚举案例，可选地为我们生成的每个OTP创建。

```
// Interestingness.swift

enum Interestingness {

    case sexts
    case quints
    case quads

    init?(code: String) {
        if code.checkThoseSexts() {
            self = .sexts
        // ...

    var title: String {
        switch self {
        case .sexts: return "Sextuples GET!!!"
        // ...

    func body(code: String) -> String {
        switch self {
        case .sexts: return "Check those sexts: \(code)"
        // ...
```

我们使用的每个`checkThose`方法都包装了不同的正则表达式，并按照我们最关心的顺序运行它们——例如，六联的稀有性比四联高100倍。

经过一次早该进行的重构后，我们创建了我们的概念验证。让我们回顾一下：

+   应用程序允许我输入一个（硬编码的）2FA秘钥。

+   应用程序本地生成一个6位数的2FA验证码，每30秒一次。

+   应用程序会安排推送通知，当生成四联、五联和六联时显示。

我打算休息几天玩一下这款应用。我怀疑我手头的这个想法可能会成为一个很酷的应用的基础。

我已经使用这款应用，这是我想法的原型，使用了几天了。而且我*非常喜欢*它。我迫不及待地等待第一次获得六联。

现在是时候在这个概念周围增加一些具体内容，构建一个完整的2FA应用了。就像我之前提出的，这实际上只需要四个主要的新功能：

+   扫描2FA二维码并将其安全地存储在钥匙串中。

+   在UI中显示和管理多个2FA账户。

+   让用户设置他们关心的数字。

+   实现更多种类的*有趣性*。

最后，一个非功能性需求：我需要优化非常慢的代码生成工作，也许可以使用批处理或本地持久化。

我没有打算在设计上做任何花哨的事情——标准的苹果`List`视图组件会让我走得更远，符合[人机界面指南](https://developer.apple.com/design/human-interface-guidelines)。

让我们保持用户体验简洁明了：我知道主要功能主要体现在推送通知中；而且这功能相当完美。这意味着将QR扫描器和设置隐藏在显示模态流程的工具栏按钮后面。

我的MVP的基本列表UI

几个开源库将为我节省大量处理重复任务的时间。[CodeScanner](https://github.com/twostraws/CodeScanner/) 提供简单的 SwiftUI QR 码扫描功能，而 [KeychainAccess](https://github.com/kishikawakatsumi/KeychainAccess) 则轻松存储这些双因素认证账户的密钥。

这个扫描库使用摄像头访问将 QR 码转换为易于解析的 URL，如下所示：

```
otpauth://totp/Google%3Atest%40gmail.com?secret=bv7exx7sltbcqffec1qyxscueydwsu5h&issuer=Google
```

现在，我们可以轻松地将我们的账户导入到应用中！

看看它们：现在带有 QR 扫描器！

使用 SwiftUI 的 `@AppStorage`，以及一些 `List` 和一些 `Toggle`，我们可以轻松地构建用户设置屏幕。

我在 `onDisappear` 中使用闭包来告诉父视图重新开始数值计算并重新安排通知。这是我批处理所有内容的最简单方法，而不是每次切换变化时运行昂贵的计算。

```
// CodeView.swift

var body: some View {  
    // ... 
    .sheet(isPresented: $showSettings) {
        SettingsView(onDisappear: {
            viewModel.recomputeNotifications()
        })
    }
}
```

瞧，我是一个独立开发者，我可以在构建过程中做到这一点！

我决定下载几款其他的双因素认证应用程序，看看有没有可以借鉴的点子。坦率地说，我预料到市场上会有相当拥挤和竞争激烈的应用程序，但有些应用真的非常糟糕。

双因素认证应用领域：大多数情况下非常糟糕

真的，超过 50% 的应用在您使用之前会弹出极具攻击性的付费墙……明明有完全免费的好选择。

没人再为了好玩而开发应用了吗？

尽管有这么多的付费墙，我确实设法记下了一些可以借鉴的好点子。

这对于拥有多个账户的任何人来说当然是相当关键的。更多账户也意味着更多机会来获取稀有的 GET 请求！

更新我的钥匙链代码，现在我们可以扫描多个 QR 码，持久化我们的账户数据（包括密钥），并且它们完美地用于登录我的各种账户！

我还实现了正确的内置 `List` 功能，因此我们可以滑动删除不再需要的验证码。

在进行竞争对手分析时，我发现 Google Authenticator 保存了我多年前添加的所有双因素认证码，这些码是我之前的 iPhone 上添加的！

那时我意识到自己在数据层面犯了两个错误。

首先，将我们的钥匙链同步到 iCloud 意味着账户会出现在您的其他所有苹果设备上。使用 Keychain Access 库非常简单：

```
// KeychainManager.swift 

self.keychain = Keychain().synchronizable(true)
```

其次，在我匆忙地选择 SwiftData 作为持久化层时，我患了“闪烁物体综合症”，只使用钥匙链保存密钥，通过新框架持久化其余的账户元数据。

这意味着我无法将我的账户信息同步到任何其他设备上——只有密钥本身是无用的！

因此，我意识到必须将整个 `Account` 放在钥匙链上。

我的新方法将 QR 码的完整 URL 保存在钥匙链中。现在，`Account` 对象本身是暂时的；每次应用程序加载时都从 URL 重新计算。

这意味着`Account`可以出现在您登录的任何iDevice上！这种短暂的方法巧妙地一举两得。现在我们在加载时需要获取键链中的帐户：

```
// AccountManager.swift 

func fetchAccounts() throws -> [Account] {
    try KeychainManager.shared.fetchAll()
        .compactMap { createAccount(from: $0) }
}

private func createAccount(from urlString: String) -> Account? {
    guard let url = URL(string: urlString),
          let account = SecretURLParser.shared.account2FA(from: url) else {
        return nil
    }
    return account
}
```

> 我的代码有点混乱，但最终的应用程序总共约为1,500行代码 - 当我想撰写关于DI的文章时，我会使用正确的DI框架重建它。如果你是一名初级工程师，请不要在家里尝试这个！

我做了很多通用编码工作，以改进UI并优雅地重构代码，但在我的开发过程中也有一些宝贵的东西是相当有趣的。

这非常有意思，但最好的开源应用程序也做到了同样，所以我感觉至少要和它一样好。

幸运的是，有一个鲜为人知的Google API，它可以在网上爬取网站的FavIcons，并允许您以多种分辨率下载它们。

我如何解决网站？我发现通过简单地使用QR码上的`issuer`属性和尝试`.com`得到了非常好的结果。

```
struct FavIcon {

    let url: URL

    init(issuer: String) {
        let domain = "\(issuer).com"
        let url = URL(string: "https://www.google.com/s2/favicons?sz=128&domain=\(domain)")!
        self.url = url
    }
}
```

这里我使用了[CachedAsyncImage](https://github.com/lorenzofiamingo/swiftui-cached-async-image)库，在图标的加载速度上获得了极快的性能。

每个2FA帐户的图像

我还添加了一个金属着色器来处理背景移除，并使图标更加突出。

这是SwiftUI视图扩展：

```
//  View+ColorEffect.swift

import SwiftUI

extension View {

    func eraseBackground(backgroundColor: Color = Color(uiColor: UIColor.secondarySystemBackground)) -> some View {
        modifier(EraseBackgroundShader(backgroundColor: backgroundColor))
    }
}

struct EraseBackgroundShader: ViewModifier {

    let backgroundColor: Color

    func body(content: Content) -> some View {
        content
            .colorEffect(ShaderLibrary.eraseBackground(
                .color(backgroundColor)
            ))
    }
}
```

当然还有MSL着色器代码：

```
#include <metal_stdlib>
#include <SwiftUI/SwiftUI_Metal.h>
using namespace metal;

[[ stitchable ]]
half4 eraseBackground(
    float2 position,
    half4 color,
    half4 backgroundColor
) {

    if (color.r >= 0.95 && color.g >= 0.95 && color.b >= 0.95) {
        return backgroundColor;
    }

    return color;
}
```

这是它们的外观。它们不错，但也不是惊人的。

金属着色器用于去除图标上的白色背景

我开始过度工程化。我们先搁置一下，看看以后的感觉如何。

现在它作为一个基本的2fa应用程序运行得相当不错。

谁会想到，要超过大多数人，我只需不设置极具侵略性的付费墙（每周$4.99？真的？！）

在一些样板软件开发工作后，包括时间安排、基本UI和数据存储，现在它真的运行得很好 - 坚持使用基本的SwiftUI组件是确保一切“正常运行”的绝佳方式。

> *并有助于使一切变得可访问！

我还实现了一些我通过竞争对手研究发现的不错的QoL功能，例如点击即复制。

我利用了像`@ScaledMetric`和`ViewThatFits`这样的可访问性工具，确保应用程序在视觉需求不同的情况下表现出色。我甚至通过紧密遵循苹果的基本SwiftUI组件和颜色，免费获得了浅色模式。

```
// AccountView.swift

@ScaledMetric(relativeTo: .largeTitle) private var iconSize: CGFloat = 36

private var icon: some View {
    CachedAsyncImage(url: FavIcon(issuer: account.issuer).url, content: {
        $0
            .resizable()
            .aspectRatio(contentMode: .fit)

    }, placeholder: {
        Text(String(account.issuer.first?.uppercased() ?? account.name.first?.uppercased() ?? ""))
            .font(.largeTitle)
            .monospaced()
    })
    .frame(width: iconSize, height: iconSize, alignment: .center)
}

private var code: some View {
    ViewThatFits {
        HStack(alignment: .center, spacing: 16) {
            codeText
        }
        VStack(alignment: .leading, spacing: 4) {
            codeText
        }
    }
}
```

在最大的可访问字体大小下检查它们

为了改进真正的核心价值主张，我实现了更多有趣性的选项：

+   六、五和四位数像`000000`

+   计数序列像`012345`

+   十万像`300000`

+   单位像`000001`和十像`000010`

+   数学常数像pi（`314159`）

+   如普朗克常数，`661034`（6.6x10⁻³⁴）

+   回文像`012210`

+   重复的二和三，如`121212`和`123123`

有些是有趣的leet-code谜题实现，有些是恼人的正则表达式，而有些则非常直接。

```
func checkThatCounting() -> Bool {
    let characters = Array(self)
    for i in 1..<characters.count {
        if let prevDigit = Int(String(characters[i - 1])),
           let currentDigit = Int(String(characters[i])),
           currentDigit != prevDigit + 1 {
            return false
        }
    }
    return true
}

func checkThatPalindrome() -> Bool {
    self == String(self.reversed())
}

func checkThoseRepeatedThrees() -> Bool {
    self.prefix(3) == self.suffix(3)
}

func checkThoseHunderedThousands() -> Bool {
    suffix(5) == "00000"
}
```

现在我已经更新了设置界面，让你可以按照稀有度（常见、罕见和超稀有）或者类型（例如重复、常数、序列或者圆数）进行排序。

在“设置”菜单中切换。

我如何计算每个稀有等级的概率？

对于像`012345`这样的完美计数序列，只有6个可能的序列（最多到`567890`），在一百万可能的数字组合中。

30秒乘以100万组合，除以6个可能的序列，意味着每个账户平均只能期望每500万秒出现一次完美的计数序列——也就是说，**平均每58天**一次。

这确实非常罕见。

然而，像`123321`这样的回文数字，有1000个可能的3序列数字组成。这意味着平均每0.34天你可能会看到它们！更为常见。

中间，像重复的二（例如`141414`）有100个可能的数字（从`00`到`99`），所以它们平均每3.5天发生一次。因此，相当罕见，但不是*超*稀有。

有些序列，比如四重，数起来比较困难，因此生成数千万个OTP并计算每种有趣性的出现次数，以了解它们的相对频率，是更简单的方法。

应用程序可以快速处理64个有趣的2FA代码，但只有在我启用所有常见的`Interestingness`时才能。当我只想要超稀有的GET时，处理时间很长。

我需要调用分块处理——在处理数百万个潜在OTP时，一旦发现有效的有趣代码，立即返回并安排通知。

我的老朋友`Combine`框架给了我们一个简洁的解决方案！

```
// CodeGenerator.swift 

var codeSubject = PassthroughSubject<OTP, Never>()

func generateCodes(accounts: [Account]) { 
    // ...
    codeSubject.send(otp)
}
```

我还使用了一些`Task`，以便我们可以在处理过程中取消和重新启动计算，例如用户在处理过程中更改了其设置。分离任务可以确保我们的加密和字符串分析操作不会占用UI线程。

```
// CodeViewModel.swift

private var otpComputationTask: Task<Void, Never>?
private var notificationSchedulingTask: Task<Void, Never>?

func recomputeNotifications() {
    handleNotificationScheduling()
    handleOTPComputation()
}

private func handleNotificationScheduling() {
    notificationSchedulingTask?.cancel()
    notificationSchedulingTask = Task.detached(priority: .high) {
        guard await NotificationScheduler.shared.isAuthorized() else { return }
        NotificationScheduler.shared.cancelNotifications()
        for await (code, count) in CodeGenerator.shared.codeSubject.values {
            try? await NotificationScheduler.shared.scheduleNotification(for: code)
        }
    }
}

private func handleOTPComputation() {
    let accounts = accounts
    otpComputationTask?.cancel()
    otpComputationTask = Task.detached(priority: .high) {
        guard await NotificationScheduler.shared.isAuthorized() else { return }
        CodeGenerator.shared.generateCodes(accounts: accounts)
    }
}
```

现在调度工作相当顺利，按顺序输出，而不是一次性大块输出！

```
Scheduled repeatedTwos: 292929 @ 2024-02-25 23:33:30 +0000
Scheduled repeatedTwos: 878787 @ 2024-02-26 06:03:30 +0000
Scheduled quints: 666660 @ 2024-02-26 10:54:00 +0000
Scheduled quints: 255555 @ 2024-02-26 21:11:00 +0000
Scheduled repeatedTwos: 606060 @ 2024-02-26 23:27:00 +0000
Scheduled sexts: 666666 @ 2024-04-16 23:22:00 +0000
Scheduled boltzmannConstant: 141023 @ 2024-04-19 02:05:00 +0000
Scheduled counting: 012345 @ 2024-04-20 04:51:30 +0000
Scheduled planksConstant: 661034 @ 2024-04-20 05:38:00 +0000
```

这就是那个让我无法忍受的东西。我非常希望能在应用图标上使用真正的“check 'em”表情。它简直完美无缺。

然而，我的好朋友指出，我们在狮门电影公司的朋友可能会感到有点诉讼倾向。

*但我非得要它！*

或许终究还是有希望的：

不像你，我对美国的版权制度有信心。

部分填写完成的表格，用于请求使用电影中的静态画面。

现在我们开始等待游戏。

寂静无声。

我已经对美国的版权制度失去了所有信心。该死的，鲍勃·艾格，公平使用到底发生了什么？！

这是我从DALL-E 3获得的最佳结果。它手指数目不对，手的一侧也不对，但经过数小时的尝试，我终于接受了这个结果。

DALL-E真的不喜欢画手背。我试过了。

这个概念得到了验证。应用运行良好！现在是时候进行一些润色和个人特性的工作，然后我们向世界展示Check 'em的乐趣了。

我创建了一个TODO列表——新功能和bug修复——在发布第一个版本之前我可以在我的V1中实现它们。

```
// High priority -
// TODO: - Add ordering as a query item to the stored URL in the keychain
// TODO: - Haptic buzz on refresh
// TODO: - Only request push notifications when they have entered the Settings Screen
// TODO: - Add settings link to enable notifications 
// TODO: - Bug - Ignore scanned duplicates in the view model accounts - don't append scans to accounts if it's already there
// TODO: - Cancel processing tasks when opening Settings view
// TODO: - Push notification deep links to an app review prompt, when the GET is still present
// TODO: - Ultra-rare GETs not being sent?? Can't make them happen locally in simulator, but quints are fine - they appear to be queued
// TODO: - Bug - Progress view doesn't appear on the second load
// TODO: - Bug - Ignore scanned duplicates in the view model accounts - don't append scans to accounts if it's already there
// TODO: - Bug - There's a bug where the percentage fluctuates up and down when there are 2 concurrent calculations
// TODO: - Add TipKit to QR and Settings

// Low priority -
// TODO: - Use @SceneStorage for state restoration; so we aren't waiting ages for the keychain operations
// TODO: - Look back/forwards one code 
// TODO: - Create a "collection" screen using deep links - collecting the seen GETs as stored items (with a dictionary on the keychain)
```

自然地，由于我眼前没有产品经理，我立即开始处理优先级最低的任务：通过深链接来构建收藏——我不想让我的稀有GET浪费掉！

这件事实际上有助于我们解决了最初的概念验证中遇到的一个问题：我们需要通过让用户与通知交互来激励他们重新进入应用程序。

创建一个收集任务有点棘手，因为涉及到一些移动的部分：

1.  允许用户点击通知并深入链接到应用程序。

1.  安全地存储他们点击的代码的有趣程度。

1.  渲染这些成一个集合屏幕。

给通知添加一个深链接相当简单。

```
// Notifications.swift

// ... 
content.userInfo = ["deepLink": "checkem://\(otp.code)"]
```

但有点让人恼火的是，我不得不创建一个`AppDelegate`来处理通知——SwiftUI目前还不能很好地处理这些。

```
// AppDelegate.swift

func userNotificationCenter(_ center: UNUserNotificationCenter,
                            didReceive response: UNNotificationResponse,
                            withCompletionHandler completionHandler: @escaping () -> Void) {

    let userInfo = response.notification.request.content.userInfo

    if let deepLinkString = userInfo["deepLink"] as? String,
       let deepLinkURL = URL(string: deepLinkString) {
        guard let code = deepLinkURL.code else { return }
        try? CollectionManager.shared.save(code: code)
    }

    completionHandler()
}
```

最后，我懒惰地添加了一个长长的、逗号分隔的存储代码列表到钥匙串中。

```
// KeychainManager.swift   

func storeCollectionItem(code: String) throws {
    var collection = try keychain.get(Constants.collectionKey) ?? ""
    if !collection.isEmpty {
        collection.append(",")
    }
    collection.append(code)
    try keychain.set(collection, key: Constants.collectionKey)
}
```

这更多是出于快速发布的愿望，而不是深思熟虑的工程决策，如果我的超级用户接近每个钥匙串项目4kB的软限制，我将会后悔（硬限制更接近16MB，所以我应该没问题！）。

这项工作迅速取得了回报，因为收藏屏幕很快就开始填满我的稀有GET！

包含所有稀有GET的收藏菜单

我最初隐藏了收藏品，直到用户点击了通知，但后来我意识到通过显示菜单选项来诱使用户收集所有内容更具吸引力。

iOS 17的`sensoryFeedback` API为我们提供了一些极其微妙的触觉反馈，实际上如此微妙，以至于我不喜欢它们。所以我从Carbn中删除了触觉引擎并在这里重复使用。

我只是在现有的刷新代码中添加了一个真正可怕的副作用：

```
// CodeView.swift

.onReceive(timer) { _ in
    let didChange = viewModel.refresh()
    if didChange {
        HapticEngine.shared.play(haptic: .refresh)
    }
}
```

孩子们，不要在家里尝试这个！

有一个bug，`CachedAsyncImage`库急切加载不存在的FavIcons，导致一个模糊的地球… 但我想我会发布这个版本。

它的工作效果基本上是90%，我宁愿发货，而不是替换我心爱的第三方SwiftUI库之一。

steam.com找不到FavIcon

在发货之前，我对其他一些bug更加注意了——特别是这个bug很糟糕，因为有人可能会扫描两次QR码，得到相同账户的怪异重复。

```
// TODO: - Bug - Ignore scanned duplicates in the view model accounts - don't append scans to accounts if it's already there
```

与更换节省时间的库相去甚远，这个bug只需要一行代码来修复。

```
// CodeViewModel.swift

func create(account: Account, url: URL) throws {
    guard !accounts.contains(where: { $0.name == account.name }) else { return }
    // ... 
}
```

由于Keychain是基于名称对2FA账户进行索引，所以这个修复非常合理。

我发现另一个代码未排队的问题。

```
// TODO: - Ultra-rare GETs not being sent?? Can't make them happen locally in simulator, but quints are fine - they appear to be queued
```

结果表明，我误解了`@AppStorage`的实际行为——默认值只应用于UI，而不是实际存储在用户默认中。

```
// SettingsView.swift

@AppStorage("sexts") private var sexts: Bool = true
```

编写一个函数，在第一次加载应用程序时填充UserDefaults解决了这个问题。

```
// CheckEmApp.swift 

@main
struct CheckEmApp: App {

    init() {
        initializeDefaultsIfRequired()
    }

    // ...

    func initializeDefaultsIfRequired() {
        guard UserDefaults.standard.object(forKey: "sexts") == nil else { return }
        CodeGenerator.shared.initializeDefaults()
    }
```

通过新iOS 17 TipKit做了一点小改进，为用户提供了关于首次加载应用程序时应该做什么的一点想法。

第一次启动时显示的提示

这实施起来出乎意料地简单，借助新API。

```
// CodeView.swift

@ViewBuilder
private var tips: some View {
    TipView(QRTip()).tipImageSize(CGSize(width: tipImageSize, height: tipImageSize))
    TipView(SettingsTip()).tipImageSize(CGSize(width: tipImageSize, height: tipImageSize))
    TipView(CollectionTip()).tipImageSize(CGSize(width: tipImageSize, height: tipImageSize))
}
```

我想我们准备好要发布了。

通过[AppScreens](https://appscreens.com/)设置我们的商店列表，决定性的第二张截图展示了Check 'em的真正力量（展示了我的猫）。

真的吗？

对不起法国，我没有精力在晚上11点填写表格 🤷‍♂️

听着，我并不是世界上最自由主义的人，但我不想为了将目标市场增加1%而跳过几个额外的障碍。请做得更好！

*(对我所有的法国读者说抱歉)*

很快，我们在App Store Connect上设置好并准备好按按钮！

谢谢你一路上与我同行！

这是一个相当有趣的项目：不仅让我能够激发我那爱发现模式的极客大脑；我还处理了一些巧妙的处理、线程和优化问题！

我接下来的步骤是[全力提升性能](https://jacobbartlett.substack.com/p/high-performance-swift-apps)，以便在`v1.1`版本中，它加载并处理一次性密码比正常更快！

如果你喜欢这个应用程序，请提出你想看到的数字建议！最后，如果有人想看到Android版本，我很乐意分享我的源代码让你跑一下。
