<!--yml
category: 未分类
date: 2024-05-27 15:00:54
-->

# My Toddler Still Loves Planes, So I Upgraded Her Radar

> 来源：[https://jacobbartlett.substack.com/p/my-toddler-still-loves-planes-so](https://jacobbartlett.substack.com/p/my-toddler-still-loves-planes-so)

This is part indie dev story, part release notes, and part technical documentation as I detail the long-awaited upgrade to [Aviator — Radar on your Phone](https://apps.apple.com/app/aviator-radar-on-your-phone/id6469189335)! 

Late-night plane-spotting using the newly upgraded Aviator 2.0

> *For the HackerNews crowd—please [follow me on X/Twitter](https://twitter.com/jacobs_handle) if you liked this post!*

[My Toddler Loves Planes, So I Built Her A Radar](https://jacobbartlett.substack.com/p/my-toddler-loves-planes-so-i-built) was by far the best thing I’ve written. I managed to hit that sweet spot of wholesome and technical, and my sheer enjoyment of the project shined through in my narrative.

Posting a week after the [OpenAI drama](https://news.ycombinator.com/item?id=38309611), I was **this** close to making the top post of the week, but was just beaten out by the late, great [Charlie Munger](https://news.ycombinator.com/item?id=38451278)

One would think, as an enterprising indie developer, I’d jump straight on top of the hype train, shoot out another release, and begin dreaming up monetisation opportunities. 

But there was a stumbling block! Unfortunately, Aviator had been broken since early December. [OpenSky Network](https://twitter.com/OpenSkyNetwork) had been having an incident which gave all my users an error.

A power outage in the open-source OpenSky Network 

After trying very hard not to think about this for several weeks, a miracle happened. I opened the app again, expecting to torture myself with the persistent 502 error message, when I heard the characteristic beep-boop of planes detected on the radar.

**It was working again!**

Newly energised by this belated Christmas miracle, I dove headfirst into two intense evenings* of overhauling Aviator with the ideas that had been brewing in the back of my brain.

> **After getting my toddler to bed, of course.*

This release includes several shiny new features, and solves several problems my daughter and I experienced while we played with the original.

I’ve overhauled the UI so the controls now live in a separate menu. This utilises [progressive disclosure](https://jacobbartlett.substack.com/p/enums-and-design-systems) to keep the major controls front-and-centre, and the advanced, less frequently accessed tools below the fold.

Yes, picking a colour is absolutely one of the 3 core controls. My daughter likes it more than the planes.

This menu is implemented as a modal with two presentation detents:

It also has a presentation drag indicator so users know they can extend the controls menu, and vitally enables background interaction so we don’t cover the main radar UI in shadow.

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

I’m planning to utilise the new [TipKit](https://developer.apple.com/documentation/tipkit) in 2.1 to ensure this progressive disclosure isn’t lost on our younger, less experienced Apple fans.

This `presentationBackground` wraps up another piece of polish: I used [Paul Hudson’s emboss effect](https://www.hackingwithswift.com/quick-start/swiftui/how-to-add-metal-shaders-to-swiftui-views-using-layer-effects). This is another Metal shader which applies a lovely metallic texture to the menu, further improving on the skeuomorphism of a control panel for a real-life radar.

> If you want to learn more about making your own shaders in SwiftUI, read my tutorial [Metal in SwiftUI: How to Write Shaders](https://jacobbartlett.substack.com/p/metal-in-swiftui-how-to-write-shaders).

While building 1.0, I experienced a persistent problem. The radar was *too damn good*. 

More specifically, it was capturing flight details of aircraft I didn’t have a hope of seeing, either far beyond the horizon or hidden behind terrain. This became a real pain-point in the terraced archipelago of suburban London. 

This wasn’t just a sensitivity issue, however — when I went out to a lovely open space, I could see planes for miles and miles. 

The solution was clear: implement zoom. 

Aviator’s view over London, default (left) and zoomed in (right)

The MapKit SDK has the concept of a `camera` which hovers above the planet, showing your map. You can feed it a `distance` (in metres) and it zooms in and out of the Earth’s surface based on this.

```
// AviatorView.swift

@AppStorage("zoomed") var zoomed: Bool = false

// ...

cameraPosition = .camera(MapCamera(centerCoordinate: coordinate,
                                   distance: (zoomed ? 70 : 100) * 1_000,
                                   heading: angle))
```

I also, in my rush to get a competent release out over the weekend, simply removed one of the radar circles to emphasise the zoomed-in UI.

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

Originally, I had a `Slider` component which handled zoom levels, however even my nifty A17 chip couldn’t quite handle this at a reasonable frame-rate. I’m using a binary zoom toggle for now, but might brainstorm alternative ways to handle this interface.

[The OpenSky Network API](https://openskynetwork.github.io/opensky-api/) gathers a wealth of information from aircraft transponders, which I was not taking advantage of in 1.0\. These helped feed into many of the new features in 2.0.

Foremost of these is the ability to toggle flags — displaying the countries of origin for each plane, as a simple emoji underneath each icon.

Toggling origin flags on and off

This utilised the `origin_country` property from the OpenSky Network API. This gave the plane start point as a straightforward text string. Using a comprehensive list of country names found online, I could convert the string into a 2-character country code, turn this into unicode scalars, and ultimately construct a flag emoji.

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

It was then simple to add text to the MapKit annotation when the user toggled this feature on.

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

Another problem I regularly ran into was a consequence of my quant life in sunny Britain.

While Aviator shows you all the planes it can find, many of these are obscured by cloud cover on drizzly days — leaving you squinting into the gloom in a quixotic quest for flashing lights.

To address this, I added another button to hide these higher aircraft — toggling between sunny-mode and rainy-mode. 

Note the high-altitude planes on the upper-right quadrant disappearing when on cloud mode

This was dead simple to implement using data retrieved from OpenSky Network. I used 2.5km as a benchmark floor for low cloud cover.

```
struct Flight {    

    // ...

    var isLowAltitude: Bool {
        geo_altitude < 2500.0
    }
}
```

This can then be used to filter out high-up flights from the UI:

```
@AppStorage("cloudy") var cloudy: Bool = false 

private var planeMapAnnotations: some MapContent {
    ForEach(flights.filter { cloudy ? $0.isLowAltitude : true }, id: \.icao24) { flight in
        // MapKit annotation ...
    }
}
```

My toddler and I found another teething issue with Aviator when running user testing in the park. We sighted a helicopter flying directly overhead, however Aviator displayed it as a run-of-the-mill airplane.

Fortunately, the OpenSky Network API has one more piece of data to help us out. We’re able to add a query parameter to get a special `category` property, which denotes the type of aircraft to which the transponder belongs. This can include:

*   Light aircraft (under 15,500 lbs)

*   Heavy aircraft (over 300,000 lbs)

*   Rotorcraft (i.e. a helicopter)

*   Space / Trans-atmospheric vehicle

*   Parachutist / Skydiver(!)

With the latest update, Aviator now differentiates between Aircraft size class; and even distinguishes satellites and helicopters as separate icons.

Perhaps I’ll display skydivers separately in 3.0, lest my users freak out at an aircraft rushing towards the ground

I first just needed to make some simple code changes to my `AircraftCategory` enum.

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

Originally, I wanted to use a helicopter emoji however, the 🚁 emoji looked overly rotund when converted into a flat colour, so I used a more abstract cross in a circle SFSymbol. 

I could then take these and adjust the appearance of the flight annotation view which makes up the icons that show up on the radar. It was a little awkward to get the emoji to behave like an image — I used a scaled `Text` view as a mask over a simple `Rectangle` so I could apply the colouring and CRT screen effects.

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

I’ve implemented many improvements using data from the OpenSky Network API. But what about improvements to the use of the API itself?

Possibly my most-requested feature, after an Android version*, is to use one’s own credentials for the API, in the hope of avoiding the same rate-limiting which the unauthenticated version uses.

Therefore, in the full controls menu, I now allow users to enter their own credentials, as well as linking to the registration page.

> **I am still waiting for an enterprising Android engineer to port it themselves—I’ve placed enough detail in these posts! You’re welcome to do so, just make it free for everybody.*

OpenSky Network API login, linking to the registration screen on their website.

This was dead simple to build using the standard SwiftUI `TextField` and `SecureField`. 

```
// ControlsView.swift

@AppStorage("username") var username: String = ""
@AppStorage("password") var password: String = ""

TextField("Username", text: $username)
SecureField("Password", text: $password)
```

> *In my excitement to get 2.0 into production, I was a little sloppy, placing the password in User Defaults. I’ll put it on the Keychain in the next release, but for now avoid re-using any sensitive passwords for this. I’ll also fix the autocapitalisation.*

Finally, I base-64 encoded these to decorate the API request with HTTP basic authentication.

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

As always, it’s uniquely gratifying to create something my daughter wants to play with. I look forward to her developing many more interests — with any luck, soon she’ll get super into platforming games or heavy metal.

If anyone has any feature ideas, or tips for ASO, please drop a comment. And most importantly, download [Aviator — Radar on your Phone](https://apps.apple.com/app/aviator-radar-on-your-phone/id6469189335) 2.0 today (and leave a review, please)!

If you enjoyed this series, please consider sharing it with your friends and colleagues.

[Share](https://jacobbartlett.substack.com/p/my-toddler-still-loves-planes-so?utm_source=substack&utm_medium=email&utm_content=share&action=share)