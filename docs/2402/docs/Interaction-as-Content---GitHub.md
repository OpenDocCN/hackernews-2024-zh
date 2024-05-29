<!--yml
category: 未分类
date: 2024-05-29 13:29:13
-->

# Interaction as Content · GitHub

> 来源：[https://gist.github.com/loreanvictor/bddd8824c744024d338e935bd7e96707](https://gist.github.com/loreanvictor/bddd8824c744024d338e935bd7e96707)

# Can We Get More Decentralised Than The Fediverse?

[](#can-we-get-more-decentralised-than-the-fediverse)

I guess that the [fediverse](https://en.wikipedia.org/wiki/Fediverse) will be as decentralised as email: a bit, but not that much. Most people will be dependent on a few major hubs, some groups might have their own hubs (e.g. company email servers), personal instances will be pretty rare. This is in contrast to personal blogging, where every Bob can easily host their own (and they often do). I mean that's already implied by the name: fediverse is [a federated universe, not a distributed one](https://en.wikipedia.org/wiki/Distributed_social_network#:~:text=Differences%20between%20distributed%20and%20federated%20networks,-See%20also%3A%20peer&text=Both%20kind%20of%20networks%20are,has%20no%20center%20at%20all).

Why does this matter? Well I like not being dependent on one entity, but I would like it much more if I was dependent on no entities at all. In other words, I like to publish my own personal blog and get all the goodies of a social network, without being dependent on other micro-blogging / social content platforms.

So in this writing, I'm going to:

*   ❓ Contemplate on why the fediverse gets federated not distributed *(spoilers: its push vs pull)*
*   🧠 Ideate on how could we get a distributed social system *(spoilers: by extending RSS)*
*   🛠️ Reflect on how would that look in practice *(spoilers: kinda weird, but I think doable?)*

Ok first, what do I mean by saying "the fediverse is federated not distributed" or "its not decentralised enough"? Well I see three levels of decentralisation (relevant here):

*   🏦 Fully central, i.e. one center (e.g. twitter servers)
*   🇪🇺 Federated, i.e. multiple centers (e.g. the fediverse, email servers)
*   🏴‍☠️ Distributed, i.e. no centers (e.g. personal blogging)

Why does fediverse leans towards the second? Because it is a *push-based* model: You need to push your content to whomever is interested, instead of just making it available for interested people to *pull it on their own*. It is the same as email, where you (or your email server) need to deliver each email to all recipients (by talking to each of their email servers). Those email servers also need to recognise and trust you too, which makes the whole network even more *federated*.

> **💡 Example**
> 
> Assume **Bob** wants to post something, **Alice**, **Carol** and **Malorey** would like to read it. In the fediverse (or a push-based system), the following happens:
> 
> ```
> Bob posts, then:
> Bob --[notifies]--> Alice.
> Bob --[notifies]--> Carol.
> Bob --[notifies]--> Malorey. 
> ```
> 
> In a pull-based system, like personal blogging with [RSS](https://en.wikipedia.org/wiki/RSS) feeds, this happens instead:
> 
> ```
> Bob posts, then:
> Alice   --[queries]--> Bob.
> Carol   --[queries]--> Bob.
> Malorey --[queries]--> Bob. 
> ```

👆 In the pull-based system, more work in the end is required (when should **Alice** query **Bob**? Also **Bob** needs to respond to the query, though thats super easy as it is static responses), but the work is better distributed, lowering the maximum amount of work someone has to do (in this case, **Bob**). Which means they need fewer resources to participate, which means more decentralised participation.

Also trust plays a role here: in a push-based system, **Bob** needs to be allowed to notify **Alice**, **Carol** and **Malorey**, which further restricts free-form participation. In a pull-based system though, **Bob** doesn't even know about **Alice**, **Carol** and **Malorey**, meaning anyone can participate more freely.

Ok before getting to a solution for a *pull-based* (and subsequently, more decentralised) social networking solution, I'd like to take a moment to consider all the pros and cons of the two approaches. We can do that without considering particulars of solutions and protocols, since the essential differences are all about the *push vs pull* content distribution model.

### 🏴‍☠️ Pull: More Decentralised

[](#%EF%B8%8F-pull-more-decentralised)

As mentioned above, making content available for interested parties to pull needs waay less resources than pushing your content onto them (either they do the work, or you do it for them). It also requires less trust and gatekeeping, so anyone can easily participate with their own nodes, servers, CDNs, whatever.

### ⚙️ Pull: Granular Access Control

[](#%EF%B8%8F-pull-granular-access-control)

In a push-based protocol, the protocol needs to somewhat have a concept of who can push what to whom, meaning anything built on top of it needs to conform to that design (e.g. [ActivityPub](https://en.wikipedia.org/wiki/ActivityPub) defines concepts of blocking, accepting follow requests, etc.).

A pull-based system doesn't need to think about access control at all. Anyone can do whatever weird form of access control they want on the content they've made available. You can publish some of your activity to some public feed while publishing some others to some more private feed with friends or co-workers access.

Its kind of obvious, if content isn't pushed, it is not circulated as fast (e.g. realtime). This might be ok for some stuff, and not for others (direct messaging kind of loses its meaning in a pull-based system, for example).

### <g-emoji class="g-emoji" alias="arrow_up_down">↕️</g-emoji> Push: Native Model of Two-way Interactions

[](#%EF%B8%8F-push-native-model-of-two-way-interactions)

A push-based system is all about two-way interactions: X pushes something onto Y. A pull-based system breaks that down to individual interactions: X posts something, Y pulls something.

Because push models two-way interactions, it acts much better on content circulation which can be modelled as two-way interaction. For example, if **Alice** comments on **Bob**'s post, in a push-based system that is the same as **Bob** posting something and notifying **Alice**. In a pull-based system though, **Bob** needs to query everyone who he knows and might've said something, to check whether what they've said is a comment on his post or not. Which is orders of magnitude more difficult.

Beyond content delivery that can be modelled as two-way interactions (e.g. comments, quotes, etc), both designs are lacking in the content discovery area in a broader sense, and in both cases you'd need to have third-party aggregators / crawlers / search services for that, similar to what search engines do for the distributed world of web pages.

While kind of independent, such discovery is an essential part of any such social network (a social network without explore, recommendation, tags, communities, etc. is just a messaging service). Any solution for this discovery issue will naturally fill-in the discovery gaps of pull vs push based systems.

In other words, if we were to practically build a pull-based system, we'd need some aggregators / search providers, which would also tell **Bob** who have reacted to their post, though in a push-based system **Bob** wouldn't be dependent on these fellas to get the answer to that question.

Assuming all those trade-offs are worth the benefits of a pull-based system, what would it look like? Well the best place to start is [RSS](https://en.wikipedia.org/wiki/RSS), since it is the defacto standard of syndicating and circulating content in a pull-based design:

*   Its been iterated upon and polished for that specific puporse,
*   It has tons of tools and clients already (RSS readers, etc),
*   A ton of content already in circulation supports RSS (Youtube, Reddit, Medium, most podcasts, most personal blogs and news outlets, etc).

What is missing here? Well social media are generally successful mostly by lowering the barriers of content creation, an important part of which is making it super easy to create content through interacting with some other existing content.

We can bring that into RSS by treating ***any interaction as content***. If you post something, thats an entry in your feed (as before). If you comment on something, thats also an entry in your feed. If you like something, thats another entry in your feed. If you follow someone (which would mean subscribing to some RSS feed), thats also another entry in your feed. To mark that interactive nature of some feed entry, we can simple extend RSS a bit:

```
<item>
    <title>Comment on "Exploring New Technologies"</title>
    <link>http://www.my.blog/posts/456</link>
    <description>This is bullshit man, you've missed a ton of nuance in this analysis.</description>
    <pubDate>Mon, 21 Feb 2024 14:34:56 GMT</pubDate>
    <guid isPermaLink="true">http://www.my.blog/posts/456</guid>
    <social:context type="comment" url="http://www.other.blog/posts/123">
        <item>
            <title>Exploring New Technologies</title>
            <link>http://www.other.blog/posts/123</link>
            <guid isPermaLink="true">http://www.other.blog/posts/123</guid>
            <pubDate>Mon, 21 Feb 2024 12:34:56 GMT</pubDate>
        </item>
    </social:context>
</item>
```

For easier discussion, I'll refer to this schematic extension as **RISS** (think of it as Really Intuitive Social Syndication, or any other acronym of your liking).

Ok that's cool and all, but would it really make sense to build products and platforms around such a protocol, if it existed? Would such products and platforms provide tangible user benefits? I think so, though I'm not sure to what extent.

### ✨ Anything, Anywhere, All at Once

[](#-anything-anywhere-all-at-once)

The most immediate benefit will be that users can get access to a lot of social content all in one place. At a basic level, this is like a nice RSS reader where you get all your news, with added engagement of being able to interact with the content.

At a deeper level though, this means you can ***find*** almost everything in one place. Most of content streams on the internet support RSS (YouTube, Medium, Reddit, podcasts, etc.). Producing RSS feeds is also relatively cheap, so content not supporting it can also [be cheaply bridged](https://gist.github.com/thefranke/63853a6f8c499dc97bc17838f6cedcc2). Top that with a nice search / aggregator, and you've effectively made the borders between various communities disappear for your users (I don't need to follow someone on YouTube to miss their content on Twitch. I can follow them anywhere in one place).

### 👁️‍🗨️ Separation of Speech and Reach

[](#%EF%B8%8F%EF%B8%8F-separation-of-speech-and-reach)

This benefit hinges on adoption so is not immediate, and might not be that great as well. But, with such a model, publishing is completely separated from distribution, meaning no one can bar anyone from publishing and their direct subscribers receiving their content (except the ISPs?). However, anyone can refuse to help distribute anything they don't like, as this is not in anyway hindering publishing of said content, and there is no exclusivity on distribution as well.

In contrast, in a centralised system, publishing and distribution are entangled, and distribution is done exlusively by the central platform operator as well, meaning them choosing "not to promote" is the borderline the same as "not allowing to be published". Even in a federated system, a server might decide they don't want to allow me to push content to my followers on that server anymore, effectively cutting off access.

Now I know people are going to complain regardless, but I do feel this separation is important for regulating such online spaces. Furthermore, I think such neat separation plays a great role in the financials of content generation as well, the same way that the distribution that lead to anyone with their own website accessible through search engines also lead to new, more open monetisation models (that are of course not without their flaws).

It actually might be possible to get more decentralised than the fediverse, via a simple extension on RSS. It might not be worth it since there will be sacrifices, but there will also be gains, so it might. And the end result might be a faster growing decentralised network as it can already incorporate much more popular content and creators, with also much lower barrier to entry and cleaner seperation of concerns and responsibilities.

I' personally pretty busy right now, but when I get time, I think I will start exploring the potential of RISS a bit more.