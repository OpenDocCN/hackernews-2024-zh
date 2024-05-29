<!--yml
category: 未分类
date: 2024-05-27 14:31:46
-->

# Virtual Reality: still not quite there, again. | by Andrej Karpathy | Medium

> 来源：[https://karpathy.medium.com/virtual-reality-still-not-quite-there-again-5f51f2b43867](https://karpathy.medium.com/virtual-reality-still-not-quite-there-again-5f51f2b43867)

# Virtual Reality: still not quite there, again.

The first time I tried out Virtual Reality was a while ago — somewhere in the late 1990's. I was quite young so my memory is a bit hazy, but I remember a research-lab-like room full of hardware, wires, and in the middle a large chair with a big helmet that came down over your head. I was put through some standard 3 minute demo where you look around, things move around you, they scare you by dropping you down, basics. The display was low-resolution, had strong ghosting artifacts, there was long response lag, and the whole thing was quite a terrible experience. I remember finally putting up the helmet and feeling simultaneously highly nauseated and… extremely excited. This was the future! I imagined the imminent miniaturization, rapid improvements in visual fidelity, VR arcades, holodecks!

1990’s virtual reality. Not sure if this is exactly the one I used, but quite similar.

And then… nothing happened. I never saw that headset again. I rarely ever saw VR mentioned in tech headlines. No VR arcades sprung up across the street that I could visit with my friends. For the young me, VR became an embarrassing misprediction, right next to “obviously we’ll have flying cars/jetpacks soon”. Eventually, dreams of fantastical digital universes slipped from my mind entirely. Over time I realized that some technical advances are too easy to imagine, but too hard to execute, and VR falls into this category. I just had to patiently wait for its time to come.

Fast forward to 2012, you can imagine my excitement when I saw the Oculus Kickstarter campaign. It was a dream come true: someone was actually building a serious consumer-grade VR headset, and Gabe Newell was personally endorsing the campaign! Starstruck, I impulsively reached for the “Back this project” button… but then realized I forgot my Kickstarter password. That, and I was afraid this was vaporware. They promised a consumer version eventually and I had decided I can just wait for that.

Oculus VR Kickstarter, 2012\. Cool visuals and Serious star power.

I don’t want this to be too long of a story. TLDR: I start obsessively checking all updates on Oculus. They get acquired by Facebook. Vive gets introduced. I end up buying **both** Vive and Oculus consumer version, then cancel the latter based on Reddit discussions. So somewhere mid 2016, a Vive gets delivered to my doorstep somewhere around noon and I skip work, intending to play with it the whole day. I was living in a small dorm room at the university so I moved almost everything from my room to the (shared) living room to get enough space to just barely satisfy the minimum size requirement for a room-scale setup.

An admittedly-hard-to-see panorama of my humble room, everything cleaned out & ready for VR.

I think I played with the Vive for about 2 hours that day and you know what? It was… pretty cool. I powered down the computer and went back to work for the remainder of the day.

> “Pretty cool.”

This is the phrase I would come to hear over and over again, as I demoed my Vive to my friends. I tried to AB test many aspects of my presentation: the games that I launched, their order, how I described VR or its possibilities, but nothing changed this reaction too much. My friends would put it on, try out some of the games and then, quite content, hand it back to me. They’d insist it was “cool”, some were even “blown away”, but it was clear they also weren’t too eager to get back. I later discovered that out of ~few hundred friends (most of them science/tech) I only had a single friend who actually bought a VR system like I did (and I almost bought TWO!, nearly squandering all the savings I could afford with my sorry PhD “salary”). It seemed that none of my friends were too excited (beyond the pretty cool first experience) and, somehow, neither was I.

Friends trying out VR. Left: Tilt Brush. Middle: Holopoint. Right: Dodging something.

Today, my Vive is organized in a messy pile of wires in the corner my room. I turn it on from time to time to try out the latest and greatest, but for the most part it collects dust. I did get to explore quite a bit though, so I feel qualified to have some opinions on what things VR do/do not work **today**.

# The features of doing VR wrong.

**Overpriced tech demos**. The first issue I noticed immediately is that VR games are expensive (e.g. up to $59.99), but as a friend of mine described it, many of them are “not too deep”. They are the $0.99 games for your phone, except on your face, and for $29.99\. I think I ended up spending several hundred dollars buying games for a total of maybe 10 hours of game time. There are many other games that are obviously lazily ported over from PC, in many cases resulting in terrible user experiences. Some of the games were so overpriced and under-cooked (with game-breaking bugs) that I ended up spending the time to file Steam complaints to get money back. Luckily, Steam is quite fast with this and promptly reimbursed the games. A VR consumer has to be careful out there.

An example of a way over-priced arcade shooter in space that 90% of people will play for <10 minutes.

**VR design anti-patterns**. It’s also surprising how many developers try to ignore the new form factor and its constraints. For instance, you **cannot** translate or rotate (or worse, accelerate) the camera because it gives people nausea. You’d think this simple fact would be obvious common knowledge, but more than 50% of VR games still think that accelerating you around is okay. For example, the PlayStation VR Shark Encounter features a shark aggressively shaking your cage, except of course you don’t *feel it*. It’s wrong.

**A single bug away from nausea**. I’ve also experienced game bugs that do weird things to your field of view. Games can switch rapidly from a 3D view to a 2D view “glued” to your eyes, or the screen can flicker, or everything will briefly reverse along some axis, or the inputs to your eyes get switched, or the camera will rapidly spiral out, or something weird. These are extremely negative experiences that usually result in tearing your headset from your head and having to sit down for a while, or completely give up for the evening. VR headsets can also lose tracking in weird ways, which can cause the camera to either drift in some random direction or cause a jitter. In short, the cost of camera errors is extremely high.

# The features of doing VR properly

I found that it’s not too difficult to create an experience that a person would describe as “pretty cool”. Even the number of people who have their “mind blown” is by itself irrelevant to the success of the platform. What **is** difficult is making one that a person wants to come back to. Only a few games have achieved this for me so far. They fall into three categories:

**1\. Full body experiences**. These are games like [AudioShield](http://store.steampowered.com/app/412740/) and [Holopoint](http://store.steampowered.com/app/457960/). There is usually some background music and you have to move your body to achieve game goals. I find these games fun and repeatable whenever I’m in a dancing/moving/feeling really cool mood. Similarly, I love games that get you to manipulate things with your controllers (e.g. Job Simulator). If you’ve only used VR with a gamepad, you are missing out.

In AudioShield you have to defend yourself from musical notes that are flying at you. Can’t help not dancing while at it.

**2\. Creative experiences**. These are things like [TiltBrush](http://store.steampowered.com/app/327140/), or any other app that channels your creativity in this new paradigm. TiltBrush today is like the MSPaint of the past, with just the most basic tools and features, but I believe there is a lot of potential for applications like this.

TiltBrush allows you to paint in 3D. Many brushes have dynamic effects. You can link effects to music.

**3\. Social experiences**. These are things like AltspaceVR, Rec Room, or Keep Talking and Nobody Explodes. The genius part about these experiences is that the developers don’t have to do the hard work of importing too much complexity into the game. All they have to do is involve people, and their social interactions deliver the complexity and repeatability. I’ve spent quite a bit of fun time in AltspaceVR: watching projected videos in the simulated living room, dancing with people, etc. I met a real-life friend in Altspace and we went for a walk and threw objects at each other, it was great.

Random people partying together in Rec Room.

In my opinion the experiences that will eventually make VR most compelling will have these features, and ideally all of them combined. They will connect people over multiplayer (3), give them ways of creating and sharing (2), and take advantage of full body presence in inventive ways (1).

# What is VR for?

If you look at the experiences built for VR today, you’ll notice that most of them are (usually single-player) games. For example I looked at 100 “new releases” for VR today and 100% of them are games. This might be because games are easier or faster to build.

> VR is for games as much as Personal Computers are for games, spreadsheets, or searching cooking recipes.

It is notoriously difficult to predict future uses of novel technologies. In 1980’s Personal Computer software involved games and personal finance applications. Amusingly, today all of the action is in a single binary application (the browser), but we can look at some of the 20 most popular websites and see that they cluster around some basic human wants:

```
**Information: “I want to know something”** Google, YouTube, Wikipedia, Stack Overflow
**Social/Communication: “I want to talk to someone”** Google (gmail), Facebook, Twitter, Instagram
**Entertainment: “I want to be amused”** YouTube, Reddit
```

In particular, the most valuable companies here have little to do with games, and all of their products are free to use**.** The PC gaming market is still there and doing fine, estimated to be worth around [$36B in 2016](https://www.superdataresearch.com/market-data/market-brief-year-in-review/), most of it from free-to-play online titles. We can see similar trends reflected in the mobile market. Looking at some of the top used apps, we see:

```
**Information: “I want to know something”**
Google Maps/Search
**Social/Communication: “I want to talk to someone”**
Facebook, Messenger, Snapchat, Instagram, Whatsapp, Gmail, Instagram
**Entertainment: “I want to be amused”**
YouTube, Pandora, Netflix, Spotify, Apple Music
```

Again, we see very little gaming, we see that all of these are free-to-use, and several of these apps play to the unique strengths of the form factor that differentiate it from what already exists, such as Maps (GPS, compass, …), and especially photo sharing (camera).

Long story short, currently the most content made for VR are games, but looking at the trends above it seems quite unlikely to me that games (in any quantity) will be the thing that makes VR go big. It’s also not clear to me that many people “get” this, or agree with it, with the possible exception of Facebook, judging by [Zuck’s latest VR demo](https://www.youtube.com/watch?v=NCpNKLXovtE).

A mixed physical/digital selfie from Zuck’s demo that hurts your brain when you really think about what’s happening here.

# Where does all of this put us?

So what does the future of VR look like? In terms of the well-known and often-referenced hype curve, one might argue that VR is finally in the stage of slowly climbing, after its peak of expectations around the 90s. However, I think a more nuanced view is that some technologies (especially those that are 1) easy to predict and 2) potentially very impactful) can in fact undergo multiple cycles whenever something exciting happens in the space. No one wants to miss the possibly few hundred $B wave that is coming at some point. I think it is likely that we are in such a situation now (right):

Artificial Intelligence (something that I know much more about) also falls into the category of “easy to predict and potentially very impactful”, and has similarly gone through several periods excitement followed by “[AI winters](https://en.wikipedia.org/wiki/AI_winter)”.

**Why not yet?** Despite the excitement that goes back to my childhood, I’ve come to be more pessimistic than optimistic about VR in the short term. There are still too many problems. VR is not like a TV that you can leave running in the background while you chat with a friend or cook a dinner. It’s not like a mobile phone that I can keep around and casually glance at to get some instant gratification. Today, VR is an activity (you have to take a long sequence of non-default actions to plug in), it disconnects you from your immediate surroundings, any interruptions are costly (e.g. I get a phone call, or I need to eat or use the restroom), and also makes you look quite silly. You also won’t avoid some stigma associated with escapism / being a nerd, something that you cannot fix in a few years.

I don’t know, I can’t quite see it.

**When it does happen**. In the medium term (e.g. a decade or two), I could see us make a lot of progress on the hardware and mitigate many of the above problems. For instance, we could shrink VR into face/hand-tracking, high-visual-fidelity, very-low-tracking-error-rate Ray-Ban sunglasses that make you look cool, and you can just slip on in a second to “plug in”. If this does happen, I feel confident making a few predictions about what the killer app for VR would look like. It:

*   Would have the features above (1\. offers functionality that isn’t already “good enough” on an existing technology (especially a mobile phone), in this case e.g. body/face tracking and interactions, 2\. allows users to create and share, and 3\. is social first).
*   It will be **free.** It will not be $59.99\. You’ll pay in other ways of course, either by paying $9.99 for a silly hat, or with your privacy.
*   It will cater to basic human needs we see coming up over and over again across time: “I want to know something”, “I want to talk to someone” and “I want to be amused”. It will not be any specific game.

Or hey, even more amusingly, a killer app could be something B2B, like enabling remote robotic work, where the worker’s commands get recorded and become training data for autonomous robotic systems. This is the core premise of my [short story on AI](https://karpathy.github.io/2015/11/14/ai/), which I can now plug here. woohoo!

**The long term**. And finally in the long term, how likely is it that we’ll have a compelling parallel digital universe (e.g. Ready Player One style) that a good fraction of humanity will plug into for a good portion of their life? On this time scale I’m relatively optimistic. After all, we’ll need some artificial difficulty in form of social fun and games when the AIs are doing all the work. Just kidding. I think. A *great* way to end the post.