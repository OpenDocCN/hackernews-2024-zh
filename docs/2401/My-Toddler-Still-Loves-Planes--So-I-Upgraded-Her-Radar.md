<!--yml

类别：未分类

日期：2024-05-27 15:00:54

-->

# 我的幼儿仍然热爱飞机，所以我升级了她的雷达。

> 来源：[https://jacobbartlett.substack.com/p/my-toddler-still-loves-planes-so](https://jacobbartlett.substack.com/p/my-toddler-still-loves-planes-so)

这部分是独立开发者的故事，部分发布说明，部分技术文档，我在其中详细描述了 [Aviator——您手机上的雷达](https://apps.apple.com/app/aviator-radar-on-your-phone/id6469189335) 的期待已久的升级！

使用新升级的 Aviator 2.0 进行深夜观飞机活动

> *对于 HackerNews 社区的朋友们——如果你喜欢这篇文章，请[关注我在 X/Twitter 上](https://twitter.com/jacobs_handle)！*

[我的幼儿热爱飞机，所以我给她建了一个雷达](https://jacobbartlett.substack.com/p/my-toddler-still-loves-planes-so)到目前为止是我写过的最棒的东西。我成功地抓住了那种温馨和技术的甜蜜点，而且我在叙述中展现出对项目的纯粹享受。

在[OpenAI 的风波](https://news.ycombinator.com/item?id=38309611)之后一周发布，我差一点就成为了本周最热门的帖子，但最终被已故的[查理·芒格](https://news.ycombinator.com/item?id=38451278)赶超了。

作为一个有远见的独立开发者，人们会认为我会直接跳上热潮，发布另一个版本，并开始梦想着货币化机会。

但是有一个绊脚石！不幸的是，自 12 月初以来，Aviator 就一直出故障。 [OpenSky Network](https://twitter.com/OpenSkyNetwork) 发生了一起事件，导致我的所有用户都出现了错误。

开源 OpenSky 网络发生了一次停电。

几周来，我努力不去想这件事情，奇迹发生了。我再次打开应用，期待着折磨自己的持久的 502 错误消息时，我听到了雷达上检测到的飞机的特有的滴滴声。

**它又能工作了！**

在这个迟来的圣诞奇迹的激励下，我投入了两个疯狂的晚上，全身心地对 Aviator 进行改版，用我脑后一直在酝酿的想法。

> **当然，是在让我的幼儿上床睡觉之后。**

此版本包含了一些闪亮的新功能，并解决了我和女儿在玩原版时遇到的一些问题。

我已经彻底改版了 UI，所以现在控制器放在一个单独的菜单中。这利用了 [渐进式披露](https://jacobbartlett.substack.com/p/enums-and-design-systems) 来保持主要控件始终置于中心位置，而高级的、不经常访问的工具则在折叠下面。

是的，选择颜色绝对是 3 个核心控制之一。我的女儿比喜欢飞机更喜欢它。

此菜单实现为一个带有两个呈现档位的模态框：

它还具有演示拖动指示器，以便用户知道他们可以展开控件菜单，并至关重要地启用了后台交互，以便我们不会在主雷达 UI 中遮挡视线。

```
// ControlsView.swift

var body: some View {
    // ... 
    .presentationDetents(availableDetents, selection: $selectedDetent)
    .presentationDragIndicator(.visible)
    .presentationCornerRadius(24)
    .presentationBackground {
        LinearGradient(colors: [.gray, .spaceGrey],
                           startPoint: .top,
                           endPoint: .bottom)
                .padding(-2)
                .embossEffect()
        }
    .presentationBackgroundInteraction(.enabled)
```

我计划在2.1中利用新的[TipKit](https://developer.apple.com/documentation/tipkit)来确保这种渐进式披露不会让我们年轻、经验不足的苹果粉丝们错过。

这个`presentationBackground`包装了另一个抛光效果：我使用了[Paul Hudson的浮雕效果](https://www.hackingwithswift.com/quick-start/swiftui/how-to-add-metal-shaders-to-swiftui-views-using-layer-effects)。这是另一个Metal着色器，它在菜单上应用了一种可爱的金属质感，进一步改善了作为真实雷达控制面板的拟态。

> 如果您想了解更多关于在SwiftUI中制作自己的着色器的信息，请阅读我的教程[SwiftUI中的金属：如何编写着色器](https://jacobbartlett.substack.com/p/metal-in-swiftui-how-to-write-shaders)。

在构建1.0版本时，我遇到了一个持久的问题。雷达效果*太好了*。

更具体地说，它捕获了我不可能看到的飞机的飞行详情，无论是远在地平线之外还是隐藏在地形后面。这在伦敦郊区的梯田群岛中成为一个真正的痛点。

然而，这不仅仅是一个敏感问题 —— 当我去一个可爱的开放空间时，我可以看到数英里外的飞机。

解决方案很明确：实现缩放。

飞行员在伦敦上空的视图，默认（左）和放大（右）

MapKit SDK具有悬浮在地球上方显示地图的`camera`概念。您可以向其提供一个`distance`（以米为单位），它会根据此距离放大和缩小地球表面。

```
// AviatorView.swift

@AppStorage("zoomed") var zoomed: Bool = false

// ...

cameraPosition = .camera(MapCamera(centerCoordinate: coordinate,
                                   distance: (zoomed ? 70 : 100) * 1_000,
                                   heading: angle))
```

在周末匆忙发布一个称职的版本时，我也简单地去掉了其中一个雷达圈以突出放大的用户界面。

```
// RadarView.swift 

private var radarCircles: some View {
    GeometryReader { geometry in
        let diameter = min(geometry.size.width, geometry.size.height)
        let middleDiameter = diameter * (2.0 / 3.0)
        let innerDiameter = diameter * (1.0 / 3.0)

        ForEach(zoomed ? [diameter, middleDiameter]
                       : [diameter, middleDiameter, innerDiameter], id: \.self) {

        // drawing the radar circles ... 
}
```

最初，我有一个处理缩放级别的`Slider`组件，但即使是我漂亮的A17芯片也无法以合理的帧速率处理这个问题。我现在正在使用二进制缩放切换，但可能会考虑替代方法来处理这个界面。

[OpenSky Network API](https://openskynetwork.github.io/opensky-api/)从飞机应答器中收集了大量信息，我在1.0版本中没有利用这些信息。这些信息有助于推动2.0版本的许多新功能。

其中最重要的是切换标志的能力 —— 在每个图标下方显示每架飞机的原产国的简单表情符号。

切换原点标志开和关

这里利用了来自 OpenSky Network API 的`origin_country`属性。这将飞机的起点给出为一个简单的文本字符串。使用在线找到的全面国家名称列表，我可以将字符串转换为2字符国家代码，将其转换为 Unicode 标量，最终构造成一个国旗表情符号。

```
// Country.swift

extension Flight {

    var flag: String {
        guard let countryCode = Country(name: origin_country)?.abbreviation else { return "" }
        return countryCode.uppercased().unicodeScalars.reduce(into: "") {
            if let scalar = UnicodeScalar(UInt32(127397) + $1.value) {
                $0.unicodeScalars.append(scalar)
            }
        }
    }
}
```

当用户打开此功能时，向MapKit注释添加文本就变得很简单了。

```
@AppStorage("showFlags") var showFlags: Bool = false

private var planeMapAnnotations: some MapContent {
    ForEach(flights.filter, id: \.icao24) { flight in
        Annotation(showFlags ? flight.flag : "", coordinate: flight.coordinate) {
            // custom annotation view ... 
        }
    }
}
```

我经常遇到的另一个问题是我在阳光明媚的英国的量化生活的结果。

虽然 Aviator 展示了它能找到的所有飞机，但在阴雨天气时，许多飞机被云层遮挡，让你眯着眼睛在阴暗中寻找闪烁的灯光，这显得有些狂妄。

为了解决这个问题，我添加了另一个按钮来隐藏这些更高的飞机——在晴天模式和雨天模式之间切换。

注意，在云模式下，右上象限的高空飞机消失了。

使用从 OpenSky Network 检索的数据实现这个功能非常简单。我将 2.5 公里作为低云层遮挡的基准。

```
struct Flight {    

    // ...

    var isLowAltitude: Bool {
        geo_altitude < 2500.0
    }
}
```

然后可以使用这些内容从 UI 中过滤掉高空飞行：

```
@AppStorage("cloudy") var cloudy: Bool = false 

private var planeMapAnnotations: some MapContent {
    ForEach(flights.filter { cloudy ? $0.isLowAltitude : true }, id: \.icao24) { flight in
        // MapKit annotation ...
    }
}
```

我和我的幼儿在公园里进行用户测试时，发现了 Aviator 的另一个问题。我们看到一架直升机直接飞过头顶，但 Aviator 却将其显示为一架普通的飞机。

幸运的是，OpenSky Network API 还有一个数据可以帮助我们。我们可以添加一个查询参数来获取一个特殊的 `category` 属性，该属性表示应答机属于的飞机类型。这可以包括：

+   小型飞机（重量小于 15,500 磅）

+   重型飞机（超过 300,000 磅）

+   旋翼飞机（即直升机）

+   太空 / 跨大气层飞行器

+   伞兵 / 跳伞者（！）

在最新的更新中，Aviator 现在区分飞机大小类别；甚至将卫星和直升机区别为不同的图标。

也许在 3.0 中我会将跳伞者单独显示出来，以免我的用户因飞机冲向地面而感到恐慌

我首先只需要对我的 `AircraftCategory` 枚举进行一些简单的代码更改。

```
// Flight.swift

enum AircraftCategory: Int {

    // ...   

    var image: Image {
        switch self {
        case .rotorcraft: return Image(systemName: "xmark.circle.fill")
        default: return Image(systemName: "airplane")
        }
    }

    var emoji: String? {
        switch self {
        case .spaceTransatmospheric: return "🛰️"
        default: return nil
        }
    }

    var scaling: Double {
        switch self {
        case .small: return 0.6
        case .large: return 1
        case .heavy: return 1.5
        // ... 
        default: return 1.0
        }
    }
}
```

最初，我想使用直升机表情符号，但是，当转换为平面颜色时，🚁 表情符号看起来过于圆胖，所以我使用了更抽象的圆中十字 SFSymbol。

我随后可以使用这些内容调整飞行注释视图的外观，该视图组成了雷达上显示的图标。让表情符号表现得像图片一样有点尴尬，我使用了一个缩放的`Text`视图作为简单`Rectangle`上的蒙版，以便我可以应用上色和 CRT 屏幕效果。

```
// FlightAnnotationView.swift

var body: some View {
    aircraft
        .scaleEffect(flight.category.scaling)
        // other effects ...
}

@ViewBuilder
private var aircraft: some View {
    if let emoji {
        GeometryReader { geo in
            Rectangle()
                .mask(
                    Text(emoji)
                        .font(.system(size: min(geo.size.width, geo.size.height)))
                        .frame(width: geo.size.width, height: geo.size.height, alignment: .center)
                )
        }

    } else {
        image
    }
}
```

我已经使用来自 OpenSky Network API 的数据实施了许多改进。但是对 API 本身的使用有何改进呢？

在 Android 版本之后，可能是我最多请求的功能之一*，就是使用自己的凭据访问 API，希望避免未经身份验证版本使用的相同速率限制。

因此，在完整的控制菜单中，我现在允许用户输入他们自己的凭据，并链接到注册页面。

> **我仍在等待一个有进取心的 Android 工程师自行移植它——我在这些帖子中放了足够的细节！欢迎你这样做，只要对所有人免费。*

OpenSky Network API 登录，链接到他们网站上的注册页面。

使用标准的 SwiftUI `TextField` 和 `SecureField` 构建这个功能非常简单。

```
// ControlsView.swift

@AppStorage("username") var username: String = ""
@AppStorage("password") var password: String = ""

TextField("Username", text: $username)
SecureField("Password", text: $password)
```

> *在激动地将 2.0 推向生产时，我有些粗心，在用户默认设置中放置了密码。我将在下一个版本中将其放入钥匙串中，但现在请避免重复使用任何敏感密码。我也会修复自动大写。*

最后，我将这些进行了 base-64 编码，以装饰 API 请求并进行 HTTP 基本身份验证。

```
// FlightAPI.swift

func fetchLocalFlightData(coordinate: CLLocationCoordinate2D) async throws -> [Flight] {

    // flight data url from coordinate ... 

    var request = URLRequest(url: url)

    if let base64LoginString = getBase64LoginString() {
        request.setValue("Basic \(base64LoginString)", forHTTPHeaderField: "Authorization")
    }

    let data = try await session.data(for: request).0
    let flightData = try decoder.decode(FlightData.self, from: data)
    return flightData.states
}

func getBase64LoginString() -> String? {
    guard let username = UserDefaults.standard.string(forKey: "username"),
          let password = UserDefaults.standard.string(forKey: "password") else { return nil } 
    let loginString = String(format: "%@:%@", username, password)
    guard let loginData = loginString.data(using: String.Encoding.utf8) else { return nil }
    return loginData.base64EncodedString()
}
```

一如既往，能够创造出我女儿想玩的东西是非常令人满足的。我期待着她能培养出更多的兴趣 - 也许很快她会对平台游戏或重金属音乐感兴趣。

如果有人有任何功能想法或 ASO 的建议，请留下评论。最重要的是，今天立即下载[飞行员 - 你手机上的雷达](https://apps.apple.com/app/aviator-radar-on-your-phone/id6469189335)2.0版（并留下评论，请）！

如果你喜欢这个系列，请考虑与你的朋友和同事分享。

[分享](https://jacobbartlett.substack.com/p/my-toddler-still-loves-planes-so?utm_source=substack&utm_medium=email&utm_content=share&action=share)
