<!--yml
category: 未分类
date: 2024-05-27 15:04:52
-->

# Oh Sh*t, My App is Successful and I Didn’t Think About Accessibility

> 来源：[https://jacobbartlett.substack.com/p/oh-sht-my-app-is-successful-and-i](https://jacobbartlett.substack.com/p/oh-sht-my-app-is-successful-and-i)

When you have a genius app idea, a brand-new job, or a fresh injection of venture capital, the temptation to tunnel-vision on shipping new features can be overwhelming.

If you play your cards right, this focus pays off, rocketing your app to the top of the download charts. *Not good enough.*You won’t stop until *everybody* is using your app.

Upon achieving this absurd milestone, you realise a hard truth: 1 in 6 users think your app is broken.

Then it hits you.

> Oh shit. 
> 
> My app is successful and I didn’t think about accessibility.

16% of people have some kind of accessibility (*a11y* if you’re one of the cool kids) requirement. But when you’re flying by the seat of your pants, it’s easy to let a11y fall by the wayside — particularly when surrounded by deadlines, with limited buy-in from leadership.

The more successful you become, the worse you’re perceived for failing to acknowledge the a11y needs of your users. This is, unfortunately, hard to prioritise without somebody influential championing a11y.

Today, I’m going to help. I’ll demonstrate how you can get your app up to speed with accessibility, and **fast**:

*   Audit a11y in your SwiftUI app.

*   Get the app looking good at all text scales.

*   Make the app useable for screen-readers.

*   Convince stakeholders to prioritise a11y.

We’re going to look at a [dead simple, cat-themed, companion app](https://github.com/jacobsapps/oh-shit-a11y) I created especially for this article. An app which looks fine on the surface but, as soon as you start evaluating it from an a11y perspective, is *utterly broken*.

If you want to fully understand the techniques, **code along too!** Start from the `Before/` folder as we implement all the techniques detailed in this article (look at `After/` for the final, improved, app).

To get a proper feel for a11y, you should always test accessibility on a real device, as this is how all your users interact with your app.

With any work on a11y, you should first optimise Control Center. *Seriously*, this is a massive timesaver. 

Go to iPhone Settings → Control Center. Add the Text Size control and Accessibility Shortcuts. Text Size allows you to swipe down from anywhere in any app, and immediately select a text scaling to apply — you can even apply sizings on a per-app basis.

Modify Control Centre to include text sizing

Accessibility shortcuts allow you to apply various a11y features on-the-fly, such as AssistiveTouch, VoiceOver, Colour Filters, and Reducing Motion. This is, however, probably easier to access by **triple-pressing the lock button**, which presents the same menu as an action sheet. 

There are 12 possible text sizes in iOS, from `extraSmall` to `accessibilityExtraExtraExtraLarge` (aka `AX5`), which is approximately 310%* larger than the default (`large`). 

> *this is the scaling of `body` font, however different system text sizes scale by different amounts, for example a `largeTitle` is already quite big, so will not scale up as much as a `caption`, which is tiny. 
> 
> This will be important later on!

To make sure text sizes work, I like to run two audits: 

1.  Test the app on the largest non-a11y font, `extraExtraExtraLarge` (aka `xxxl`), ideally with the *Bold Text*accessibility setting enabled too. This is the biggest relatively common font, and also what my mum uses. The app should look *good* at this size.

2.  Test the app on the largest a11y font, `AX5`. While, if you’re not used to it, it might look comically large, there are people for whom this is *the only way* they can interact with your app. While it’s tough to make the UI look perfect at this size, it’s blindingly obvious when a company hasn’t even checked if the app works at accessibility text scalings.

Let’s run through our app. 

Auditing my login screen with large (default), xxxLarge (and bold), and AX5 text scalings

Immediately, we find that our painstakingly crafted login UI falls flat on its face when we apply text scaling. We were sloppy, and we didn’t anticipate larger text sizes — so never thought to make the content scrollable.

What’s more? Somebody with colourblindness would not be able to read that login button, since the contrast is miniscule.

Let’s also check out look at the main list screen of our app.

Auditing my list screen with large (default), xxxLarge (and bold), and AX5 text scalings

This one is less ‘broken’, in that you can see everything, but it’s clear to anybody using an accessibility-sized font that you didn’t bother testing the app at this size — just look at the text labels warring for space, or that tiny ❤ icon next to a massive caption.

Some of our users are unaffected by text scaling — since they may require a screenreader to interact with your app.

iOS VoiceOver does an incredible job handling the heavy lifting here, so a little bit of thoughtfulness from your team goes a long way.

Screenreader audits, like text size audits, are also best done on a real device (press the lock button 3 times, remember!). There is also a developer tool built into Xcode which helps with VoiceOver— the Accessibility Inspector.

Finding the Accessibility Inspector developer tool in Xcode

Walking through the app, it’s less offensively broken compared to the text scaling, but large areas of the UI like our images are entirely invisible to our users because we didn’t bother to add accessibility labels.

> I’m intentionally using hostile language here, because this is how your app is going to be perceived by users you left behind. Hostile.

Screen-reading the login screen tries it’s best, giving the name of the image resource

It’s doubly clear when somebody hasn’t been thoughtful thought about VoiceOver when every single UI element is iterated through by the screen-reader, including unlabelled icons.

Lazy use of a screenreader ends up listing every UI element individually

The situation gets worse with a big list. 

If you don’t give SwiftUI information about your content, you can end up with a situation where it takes 100s of swipes to move to the logout button at the bottom of your scrollable content. And *don’t get me started* on paging.

With many items, it takes a very long time to get to the bottom of the content.

It’s clear we’ve got a fair few issues to solve before we can call our app accessible. Fortunately, SwiftUI is packed to the gills in tools that allow you to get up to speed on a11y extremely fast.

Now we know where the problems are, we can start to address them one by one. And quickly. 

This is the most offensive issue — my app onboarding is completely broken, since the content doesn’t expand into a scroll view on larger text sizes.

In our audit, the button for our login screen was pushed offscreen as font size increased. The rest of the text content strained against each other, labels overlapping and truncating as they had no room to expand.

The natural solution is to make the content scrollable. But we don’t want to haphazardly allow every single screen to scroll when the content fits fine. We aren’t in the business of building Ionic apps **shudders with repressed memories**.

This issue can be addressed with a custom view modifier, what I call the `a11yScrollView()`. The goal of this view modifier is to wrap content in a scroll view, but only if it *needs* to — content only becomes scrollable if it doesn’t fit. I’ve added it to a small library, [A11yUtils](https://github.com/jacobsapps/A11yUtils/blob/main/Sources/A11yUtils/A11yScrollView.swift), so you can use it in your own code.

Right now, this only achieves our dream behaviour in iOS 16.4 and above — applying `scrollBounceBehavior` to the scroll view to prevent it from scrolling if the content fits already.

xIn the future, I’d like to make it work with `ViewThatFits*` for iOS 16 and `GeometryReader` for iOS 15.

> **[Feel free to submit a pull request!](https://github.com/jacobsapps/A11yUtils/tree/main)**

Since nobody is around to demand which OS version I target, I can apply this modifier to the `VStack` in our onboarding view.

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

Now, the content scrolls nicely — looking good across `xxxl`, `AX3`, and `AX5`.

Our content now scrolls on all text scalings. For good measure, I fixed the contrast on the Log In button

> *In my experience, the `ViewThatFits` approach works nicely, but has problems with text entry — the software keyboard actually changes the size of the view, causing the content to resize and the layout engine to redraw the content. If a `@FocusState` toggles the keyboard on appearance, this leads to an infinite redrawing and resizing loop!

While we’re working on the onboarding screen, there’s another simple improvement we can make:

```
VStack(spacing: 20) {
    // ...
    OnboardingReasonsText()
    Spacer()
    LoginButtonView()
}
```

When text size is large, limiting screen real estate, `Spacer()` can cause issues. The SwiftUI layout engine will diligently attempt to create space, even if there is no room for it, which can lead to your text getting truncated.

In this scenario, even with the spacer compressed to a very narrow height, there will be 20 points above and below, as defined in our `VStack`.

We can swap out the `Spacer()` for a `frame()` modifier:

```
VStack(spacing: 20) {
    OnboardingReasonsText()
    LoginButtonView()
        .frame(maxHeight: .infinity, alignment: .bottom)
}
```

Frames behave in a more reliable manner: SwiftUI knows to give as much space as possible above this button, however it is clear that the space itself isn’t a critical part of the UI, so no unnecessary spacing is applied if there isn’t room.

This frame modifier has a clearer *semantic* meaning — that is, it makes our intent clear to SwiftUI. This is a critical concept to understand when it comes to a11y, so we’ll cover it more later on.

Thanks to dynamic type, iOS will scale the font in our app automatically based on the user’s chosen font size. This is also pretty straightforward to implement with a [custom font](https://developer.apple.com/documentation/uikit/text_display_and_fonts/adding_a_custom_font_to_your_app).

This means when we use semantic font sizes in SwiftUI, text will automatically scale.

```
Text(cat.quote)
    .font(.body) // this works fine
```

Even better, when sticking to SFSymbols, we can get dynamic type on our icons!

```
Image(systemName: "heart.circle.fill")
    .font(.body) // this also works fine with SFSymbols
```

Unfortunately, most iOS apps tend to have an Android counterpart, which means your designer will be creating custom icons rather than allowing you to use SFSymbols everywhere. Most apps will also include custom images and media.

Therefore, for our `Before/` example, I’ve treated the icon like any regular image we’d add to SwiftUI, using a hardcoded size.

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

The result? At the largest font scaling, the content looks very mismatched.

Images and icons with a fixed size look strange on larger text scalings

SwiftUI has a nifty tool that gives us the power of dynamic type in our custom images and views: the `@ScaledMetric`property wrapper. 

```
@ScaledMetric(relativeTo: .largeTitle) private var imageSize: CGFloat = 72
@ScaledMetric(relativeTo: .body) private var iconSize: CGFloat = 24
```

We can use `@ScaledMetric` on its own, which uses the default `body` font scaling. It’s even better to use `relativeTo` so SwiftUI knows how to scale it.

`largeTitle` scales a little bit, since it’s already quite large. A `caption` style font scales up far more. In this instance, the icon has `body` scaling so it matches the accompanying quote text.

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

Now, our images look a lot more sensible alongside the text.

Image scaling using @ScaledMetric 

The images scale better, however our UI is still clearly broken, with the text labels vying for space. After exhaustive negotiations with the layout engine, they end up full of line breaks.

**What if we could align content based on a user’s text scaling?**

Fortunately, thanks to my library [A11yUtils](https://github.com/jacobsapps/A11yUtils/blob/main/Sources/A11yUtils/A11yHStack.swift), we can utilise an `A11yHStack`. 

Due to the blistering pace of new SwiftUI tool releases (and Apple’s crappy back-porting), this also has a couple of different implementations based on your minimum supported OS: 

*   On `iOS 16`, it uses `ViewThatFits` to check whether the `HStack` can contain all the content, and if not, uses a `VStack`to arrange the content instead. 

*   On `iOS 15`, it applies a more blunt approach, checking `@Environment(\.sizeCategory)`, and switching to a `VStack` if an accessibility text size category (such as `AX1`, `AX2`, or `AX5`) is used. 

Here’s the simple approach:

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

The `@Environment(\.sizeCategory)` is a useful tool to know about, in the cases where it’s important to know whether an a11y text size category is being used (with `isAccessibilityCategory`). Sometimes, I use this property to remove unnecessary UI elements such as images which simply don’t fit at massive content sizes.

The more modern implementation works similarly but utilises `ViewThatFits`, so content is only re-distributed if it doesn’t fit:

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

If we’re thoughtful, we can apply `A11yHStack` in a few places:

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

Now our content looks much more sensible on very large text sizes.

Utilising A11yHStack for our content in the List cells.

> Another small warning: make sure you test on all supported iOS versions, as I’ve found the `ViewThatFits`approach to give behaviour I wasn’t expecting. When getting started, you may have an easier time re-implementing the `@Environment(\.sizeCategory)` approach.

VoiceOver is a reasonably straightforward piece of the puzzle. 

Making your app play well with a screenreader means:

*   Making sure visual content is adequately described.

*   Making navigation work intuitively.

*   Ensuring our view makes *semantic* sense.

Let’s start with visual descriptions. This is the simplest fix in our repertoire: ensuring our graphical content can be described by VoiceOver. The `accessibilityLabel` modifier is our bread and butter.

```
Image("catKingdom")
    .accessibilityLabel(Text("My three cats: Cody, Rosie, and Luna"))
```

This tells the screenreader how to explain what it’s currently focused on.

Applying an accessibilityLabel to the onboarding image

Next, we can apply the same approach on our main view’s cells. With a small modification to our `Cat` data model, we supply an image description alongside each cat’s photo.

```
struct Cat {
    // ...
    let imageDescription: String
}

Image(cat.image)
    .accessibilityLabel(Text(cat.imageDescription))
```

This is as easy as it sounds, and tends to *just work* on Apple platforms.

Applying voiceover descriptions to all our images via the data model

When it comes to navigation through our app, we have the opportunity to be more thoughtful. By default, VoiceOver will iterate through every leaf node in the SwiftUI view tree and read out all our `Image`s, `Button`s, and `Text`.

Sometimes, this causes lots of unnecessary navigation for the user. In our app, we force users to tap multiple times to navigate past the icon and quote for each cat. Reading out the icon’s name (such as `cat.fill`) adds nothing to the experience. 

We can combine these together as a single a11y element using the `accessibilityElement(children:)` modifier, so the screenreader can treat them as one item when navigating through.

```
HStack {
    Image(systemName: cat.icon)
    Text(cat.quote)
}
.accessibilityElement(children: .combine)
```

Now we can hear the screenreader move through the elements as one item:

Combining VoiceOver UI elements together using accessibilityElement(children:)

In some situations, you might have a really complicated UI, such as a [stylised timer element which graphically ticks down](https://jacobbartlett.substack.com/i/141732200/image-loading-bug). VoiceOver is smart, but it isn’t going infer anything useful from a fancy collection of unexplained views.

This is where the `accessibilityRepresentation` modifier comes in really useful. It allows you to replace a view’s VoiceOver UI with a fully customised a11y representation.

This was particularly useful for my latest indie project, [Check ’em](https://jacobbartlett.substack.com/p/building-a-2fa-app-that-detects-patterns), where a user’s 2FA codes were displayed with a countdown: 

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

This modifier allowed me to introduce a fully custom VoiceOver interface, which was far more useful than iterating through each element of the cell in sequence. It also prevented the screenreader reading 676,252 as *six hundred and seventy-six thousand, two hundred and fifty-two.*

accessibilityRepresentation modifier used in Check ’em, my indie project

The main view of this app uses a common, but sloppy, approach to laying out our content: wrapping our cells inside a `ForEach`, wrapped in a `LazyVStack`, wrapped in a `ScrollView`. 

```
ScrollView {
    LazyVStack(spacing: 24) {
        ForEach(cats, id: \.name) {
            CatView(cat: $0)
        }
    }
}
```

The optimal approach is to **utilise native SwiftUI components wherever possible**. In this instance, a `List` is most appropriate.

Apple makes this a smidgen harder than it needs to be — inexplicably, to use our own custom UI rather than the iOS-Settings-styled `List`, we need to add 3 separate modifiers.

```
List(cats) {
    CatView(cat: $0)
        .listRowBackground(Color.clear)
        .listRowSeparator(.hidden)
}
.listStyle(PlainListStyle())
```

With that boilerplate out of the way, the modifiers now allow us to freely use our own UI in a SwiftUI `List`.

Default List (left) and our custom List (right) that used .listStyle and .listRow modifiers

One question.

**Why go to all this extra trouble to use a native component? **

Firstly, the non-a11y reason: SwiftUI `List` is implemented using a `UICollectionView`. This utilises cell reuse for high performance scrolling, even if you have many items. Conversely, `LazyVStack` keeps all previously-rendered cells in memory, meaning your performance nosedives if you’re scrolling through many items.

> This information comes courtesy of [Thomas Ricouard](https://medium.com/u/b067c9115d59) and friends; read [SwiftUI: The difference between List and LazyVStack](https://dimillian.medium.com/swiftui-the-difference-between-list-and-lazyvstack-3d5eeaccb156) to learn more!
> 
> If you’d like to see more content on app performance, check out my recent deep-dive, [High Performance Swift Apps](https://jacobbartlett.substack.com/p/high-performance-swift-apps).

The a11y-related reason to use native components are simple, yet myriad: 

> **Apple has, collectively, put more thought into making a11y work more than you ever will.**

`List` has a *semantic meaning*: it tells the screenreader that it’s a container that houses a collection of similar content. 

This is why, when we have many list items, the naïve implementation was so broken: instead of allowing us to skip past the `List` container with VoiceOver, we had to swipe through every item in order to reach the *Log Out* button.

With `List`, SwiftUI can inform the screenreader that its content is a single Container, making it trivial for the screenreader to navigate past.

When using VoiceOver, you can skip past the List by setting the navigation increment to “containers” and swiping forwards through the content

There are also many built-in interaction modes with a `List`, such as swipe-to-delete, drag-and-drop, and keyboard-based navigation. These will *just work* when using the native components. Have you any clue how you’d get a screenreader to work with your janky custom implementation?

> Refer to the above passage the next time you need to push back on your designer’s latest stroke of genius.

`List` isn’t even the most complicated component. How would you implement a11y for a custom `Grid`, or a custom distribution of views that wasn’t using the `Layout` protocol?

Now that you understand how to speedrun a11y in your SwiftUI app, before getting to work, the final piece of the puzzle is getting buy-in from the rest of your business. This is where you can flex your soft skill of *wielding organisational influence*.

One big, blunt tool you can wield is the hammer of legislation. Various countries around the world such as the [UK](https://www.legislation.gov.uk/uksi/2018/852/contents/made), the [USA](https://www.section508.gov/manage/laws-and-policies/), and the [EU](https://www.levelaccess.com/compliance-overview/european-accessibility-act-eaa/), have implemented legislation mandating that digital services meet minimum standards of accessibility.

Furthermore, there are serious business benefits to a11y. 16% of the world population currently experience significant disability, meaning you could be missing out on users, positive reviews, and revenue by catering to a wider range of needs. Depending on your target demographics, this may be more or less critical. 

On a smaller scale, I have found that banging the drum on addressing accessibility as product debt is far less effective than simply *demonstrating how your app is broken*. Taking the need for a11y out of the abstract and displaying the buggy sign-up flow is far more persuasive to any engineering organisation.

Nothing is more helpful than having an influential champion in leadership who is bought-in to making the product accessible. Of course, all organisations are different, so your mileage may vary when applying this advice. 

The most important thing, now that you’ve speedrun a11y and your product is totally accessible? 

**Don’t drop the ball!**

Much like with technical debt, now that you’re in a good place, you can bake a11y into your standard development workflow. It’s quick and easy now you know how!

We’ve covered a lot today. 

First we went through the process of auditing your app for common accessibility mistakes that show up when using large text scalings or VoiceOver. 

We looked into fixing the problems that arise when using text scalings, applying scrolling with our `a11yScrollView`, content scaling with `@ScaledMetric`, and alignment with `A11yHStack`.

To make our app work well on a screenreader, we implemented `accessibilityLabel`s, combined `accessibilityElement`s, and even fully-customised `accessibilityRepresentation`s.

We discussed the reasons to stick to native SwiftUI components over custom views, such as semantics, interaction modes, and performance.

Finally, we went over how you might apply soft skills to get buy-in from your organisation to take accessibility seriously.

If you are serious about a11y in your SwiftUI apps, I implore you to work through the [companion app](https://github.com/jacobsapps/oh-shit-a11y) and implement these techniques yourself, it’s really the best way to internalise the tools. If you have a side project you’ve been working on, that’s also a great place to start.

I am gladly accepting contributions (and issues) to my [A11yUtils](https://github.com/jacobsapps/A11yUtils/blob/main/Sources/A11yUtils/A11yHStack.swift) library — I’d love to work with the community to transform this into a comprehensive suite of a11y tools to supplement the APIs already in SwiftUI.

* * *