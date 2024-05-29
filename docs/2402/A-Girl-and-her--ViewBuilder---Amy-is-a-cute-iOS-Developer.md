<!--yml
category: 未分类
date: 2024-05-29 13:21:50
-->

# A Girl and her @ViewBuilder | Amy is a cute iOS Developer

> 来源：[https://cuteios.dev/2024/01/10/viewbuilder.html](https://cuteios.dev/2024/01/10/viewbuilder.html)

***note***: the story here is fictional, but it’s an approach I love to take when solving similar problems. Hope you enjoy it and get a good understanding of how make use of `@ViewBuilder` in SwiftUI apps.

## Once upon a time

One upon a time, a girl was presented with a task. Refactor this code her manager demanded, another team is wanting to reuse this part of the UI. “Happy to” she responded, knowing that there was no getting out of this work. ‘Tis much better to build this well and share it around than have many implementations of the same thing with very small differences between them. None of which quite solving the problem. Loading up a [Riot Grrrl](https://music.apple.com/au/playlist/riot-grrrl-essentials/pl.4ac9566eba234d99971cface91950c1c) playlist in Apple Music, she pulled down the latest version of the codebase and began to work. The code she found wasn’t the best. If it were, she wouldn’t be doing this task now would she? She had seen this kind of code many times before. A few different components wrapped up in what would colloquially be described as a card. There was a [VStack](https://developer.apple.com/documentation/swiftui/vstack) with the different bits and pieces stuck in there along with a fairly standard set of [view modifiers](https://developer.apple.com/documentation/swiftui/viewmodifier).

## Understanding the requirements

There are many approaches the girl could take to solving this problem. One option would be to rename the existing view, expose properties for things like title, image, text and throw it into a shared part of the project. But she knows well and true that will only cause pain down the track. She needs a way to make the content arbitrary while maintaining style and functionality. This will give her the best base component to allow others to build upon.

## Looking at alternatives

In her time working on apps built in SwiftUI the girl knew of a few solutions she could apply to make this all work as she wanted

*   view builders
*   view extensions
*   view modifiers

#### View Modifiers

View modifiers have been around since the start of SwiftUI and have been a long established way of applying a common set of modifications to a view. The girls team has experience with this and she knows there will be little cognitive overhead for them to pick up the preferred way of doing this. So how would this look to her teammates? This is always an important question to ask and so she opened up a [Swift Playground](https://developer.apple.com/swift-playgrounds/) (We all remember those things right? More than just for WWDC scholarship applications) and started roughing out what it would be.

```
public struct CardModifier: ViewModifier {
  func body(content: Content) -> some View {
    content
      .frame(maxWidth: .infinity, alignment: .leading)
      .fixedSize(horizontal: false, vertical: true)
      .padding(8)
      .background(Color.background)
      .mask(RoundedRectangle(cornerRadius: 8, style: .continuous))
  }
} 
```

And then it’s usage in a view

```
Text("Hello World")
  .modifier(CardModifier()) 
```

But is this as good as it could be? The girl sat there in her chair, one leg tucked under her in a posture that confounds her colleagues and sipped on the tea she had recently poured for herself. A miracle that she remembered to take a break and spend time contemplating next steps. She knew that this would solve a problem, but would it encourage the team to use it and have an understanding of what is being accomplished? She gazed at her Swift Playground a bit more, but was not satisfied she could answer that question in the affirmative. One approach down, on to the next one.

#### View Extensions

Knowing that SwiftUI is very idiomatic, the girl started to ponder the question “What if this was an extension on View?”. Thankfully this one could be proven viable or not by a simple addition to her open Swift Playground.

```
extension View {
  func displayAsCard() -> some View {
    modifier(CardModifier())
  }  
} 
```

This then gave her the freedom to apply it to any view type and have it understandable that the view itself was being displayed as a card. But she knew, even this didn’t have what she was looking for. It was a single change to an existing View in the app but it lacked discoverability and had a high cognitive overhead on her colleagues. She didn’t want to make life more difficult for her friends that she interacted with ever day. Well, there was one who was super annoying but what work environment doesn’t have people like that in them. The girl shuddered and as she was still holding her tea for warmth in both hands took a calming sip of tea. Thankfully, she knew of a solution and where she wanted to go with this bit of work.

#### View Builders

Each year the girl spent a good amount of her time travelling the world and speaking with others in the industry. She had to be super careful in this as the environments could very easily be toxic. She was there to learn more about what she loved doing day to day. A repeated theme amongst conversations is how Swift as a language allows for the writing of DSL’s and this was a super powerful technique to use when writing apps with SwiftUI. She had heard of the idea about [result builders](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/advancedoperators/#Result-Builders) and their specific application in SwiftUI being [@ViewBuilder](https://developer.apple.com/documentation/swiftui/viewbuilder). Needing a refresher before continuing, she queued up a few WWDC videos on the topics being [Write a DSL in Swift using result builders](https://developer.apple.com/wwdc21/10253) and [Embrace Swift generics](https://developer.apple.com/wwdc22/110352) that deal with the topic. She couldn’t help but be excited that both presenters in the videos were female. With how much progress had been made in recent years, there was still a long way to go towards gender equality.

The girl now had the approach she wanted to take sorted out. She fetched yet another cup of tea and got to work on the solution.

```
struct Card<Content: View>: View {
  let configuration: CardViewConfiguration
  let content: Content

  init(configuration: CardViewConfiguration = .default, @ViewBuilder content: () -> Content) {
    self.configuration = configuration
    self.content = content()
  }

  var body: some View {
    content
      .frame(maxWidth: .infinity, alignment: .leading)
      .fixedSize(horizontal: false, vertical: true)
      .padding(configuration.padding)
      .background(configuration.backgroundColor)
      .mask(
        RoundedRectangle(
          cornerRadius: configuration.cornerRadius,
          style: .continuous
        )
      )
  }
}

struct CardConfiguration {

  public let backgroundColor: Color
  public let cornerRadius: CGFloat
  public let padding: CGFloat

  public init(backgroundColor: Color, cornerRadius: CGFloat, padding: CGFloat) {
    self.backgroundColor = backgroundColor
    self.cornerRadius = cornerRadius
    self.padding = padding
  }

} 
```

This had the advantage of being usable within the app in a way to simply described what was being created and allowed others to build what they want on top of it.

```
Card {
  Text("Hello World")
} 
```

## Bringing the team on board

Satisfied with how the use of ViewBuilder came out, the next step for the girl was to bring her team onboard with the approach. She could slap together a PR and send it up for others to review, but that wouldn’t achieve much. If others were to start adopting a similar approach in how they wrote views, she would need to give them as much help as possible. This was more than just chatting with other engineers. Now it was time for a checklist. She still had a lot of work to do to ensure adoption by others.

*   Chat with UX designers about upcoming usages.
*   Chat with accessibility experts about how it would look for vision impaired users.
*   Inline documentation that covers example usage.
*   Brown bag (small talk / meeting at work) covering the usage of `@ViewBuilder` in the app and other possibilities for the use of similar DSL’s.
*   Work with engineers as they adopt the new UI component and adopted similar patterns in their work.

The girl leaned back in her chair, cup of tea firmly grasped in both hands. It will go unmentioned how many she had consumed throughout this day. She was satisfied that she had met the requirements of the work and the team is now in a better place.