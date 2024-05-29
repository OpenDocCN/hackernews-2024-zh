<!--yml
category: 未分类
date: 2024-05-27 14:32:02
-->

# An app can be a home-cooked meal

> 来源：[https://www.robinsloan.com/notes/home-cooked-app/](https://www.robinsloan.com/notes/home-cooked-app/)

<main> <stamp-thwack>Robin Sloan
 the lab
 February 2020</stamp-thwack>

# An app can be a home-cooked meal

Have you heard about this new app called BoopSnoop?

It launched in the first week of January 2020, and almost immediately, it was down­loaded by four people in three different time zones. In the years since, it has remained steady at four daily active users, with zero churn: a resounding success, exceeding every one of its creator’s expectations.

🙂

I made a messaging app for, and with, my family. It is ruth­lessly simple; we love it; no one else will ever use it. I wanted to share a few notes about how and why I made it, both to (a) offer a nudge to anyone else consid­ering a similar project, and (b) suggest something a little larger about software.

 </media/boopsnoop-smile-faststart.mp4#t=0.001>

 Your browser can’t play my video clip. Rats! 

Tap or click to unmute.

## Barely there

My story begins with another app, now defunct, called Tapstack.

Opening the app, you saw a live feed from your phone’s camera. Below, a grid of faces, some of them representing individuals, others repre­senting groups. My grid had four cells: my mom, my dad, my sister, and a group collecting all three. Just like Snapchat or Instagram, you tapped to capture a photo, pressed to record a video. As soon as you lifted your finger, your message zipped away, with no editing, no reviewing. A “stack” of messages awaited you in the corner, and, after you tapped through them, they were discarded.

It was all so simple that it was barely there. Tapstack more closely approx­i­mated a clear pane of glass than any app I’ve ever used.

For several years, Tapstack was the main channel for my family’s communication. The app didn’t lend itself to practical corre­spon­dence or logis­tical coordination; its strength was ambient presence. I met one of Tapstack’s designers once, and they told me it seemed espe­cially popular with far-flung families: a diaspora app. Because there was no threading and no history, messages didn’t carry the burden of an expected reply. Really, they were just a carrier wave for another sentiment, and that sentiment was always the same: I’m thinking of you.

A selfie with coffee, a picture of an ice-covered pond, a video of my nephews acting silly: I’m thinking of you, I’m thinking of you, I’m thinking of you.

It never seemed to me that Tapstack attracted a huge number of users. I don’t know if the company ever made a cent. There was no adver­tising in the app, and they never asked their users to pay.

Why didn’t they ask us to pay?

In 2019, I felt a rising dread as the months ticked by and the app didn’t receive a single update. Sure enough, in the fall, Tapstack announced that it was shutting down. It offered its users a way to export their data. It went gracefully.

It was, I have to say, a really great app.

## Here comes a new challenger

My family all agreed we were going to need a replacement, and while my first instinct was to set up a group on Instagram or WhatsApp, the prospect of having our warm channel surrounded — <wbr>encroached upon — <wbr>by all that other garbage made me feel even sadder than the prospect of losing Tapstack.

So, instead of settling for a corporate messaging app … 

I built one just for us.

I’ll show you the screen capture again, but the point is that there’s not much to show. The app is a “magic window” that captures photos and videos and shuttles them around. Messages wait in a queue and, once viewed — <wbr>always full-screen, with no distractions, no prods to comment or share — <wbr>they disappear. That is literally it. The app has basically no interface. There’s a camera button and a badge in the corner, calm green, that indicates how many messages are waiting.

 </media/boopsnoop-smile-faststart.mp4#t=0.001>

 Your browser can’t play my video clip. Rats! 

Tap or click to unmute.

Here are a few mildly technical observations. Feel free to skip ahead if this part doesn’t interest you:

*   Tapstack was simple to start with, and I made it even simpler. Unlike Tapstack, my app doesn’t need a login system. It doesn’t need an interface to create and manage contacts. It already knows exactly who’s using it. (This makes me think about [an old blog post](https://web.archive.org/web/20050120085129/http://www.shirky.com/writings/situated_software.html) by Clay Shirky: “Situated software, by contrast, doesn’t need to be personalized — <wbr>it is personal from its inception.”)

*   The core of the app is a camera view with the now-familiar tap/press for photo/video affordance. This is an [off-the-rack open source component](https://github.com/Awalz/SwiftyCam?utm_source=Robin_Sloan_sent_me); what a gift. I don’t think this project would have been possible without it.

*   Besides the app itself, not much is required: an AWS S3 bucket to hold the photos and videos, a couple of AWS Lambda functions to shuffle things around when new messages are uploaded. The back end is actually fairly elegant — <wbr>which is, uh, not usually my style — <wbr>but, again, that’s only because it’s so simple. There’s barely anything there.

*   I distribute the app to my family using TestFlight, and in Test­Flight it shall remain forever: a cozy, eternal beta.

In a better world, I would have built this in a day, using some kind of modern, flexible HyperCard for iOS.

In our actual world, I built it in about a week, and roughly half of that time was spent wrestling with different flavors of code-signing and identity provi­sioning and I don’t even know what. I burned some incense and threw some stones and the gods of Xcode allowed me to pass.

Our actual world isn’t totally broken. I do not take for granted, not for one millisecond, the open source compo­nents and sample code that made this project possible. In the 21st century, as long as you’re operating within the bounds of the state of the art, program­ming can feel delight­fully Lego-like. All you have to do is rake your fingers through the bin.

I know I ought to pay it forward and publish the code for my app. Even if it doesn’t work for anyone else as-is, it might provide a helpful guide — <wbr>one I would have been grateful to have. But the code is marbled with application-specific values, well-salted with authen­ti­ca­tion keys. This app is Entirely Itself — <wbr>not a framework, not a template — <wbr>and that’s insep­a­rable from the spirit in which it was made. Which brings me to:

## Cooking at home

For a long time, I have struggled to artic­u­late what kind of programmer I am. I’ve been writing code for most of my life; I can make many inter­esting and useful things happen on computers. At the same time, I would not last a day as a profes­sional software engineer. Leave me in charge of a critical database and you will return to a smoldering crater.

Building this app, I figured it out:

I am the program­ming equiv­a­lent of a home cook.

The exhor­ta­tion “learn to code” has its foun­da­tions in market value. “Learn to code” is suggested as a way up, a way out. “Learn to code” offers economic leverage, profes­sional transformation. “Learn to code” goes on your resume.

But let’s substi­tute a different phrase: “learn to cook”. People don’t only learn to cook so they can become chefs. Some do! But many more people learn to cook so they can eat better, or more affordably. Because they want to carry on a tradition. Sometimes they learn because they’re bored! Or even because they enjoy spending time with the person who’s teaching them.

The list of reasons to “learn to cook” overflows, and only a handful have anything to do with the marketplace. Cooking reaches beyond buying and selling to touch nearly all of human experience. It connects to domes­ticity and curiosity; to history and culture; to care and love.

Well, it’s the 21st century now, and I suspect that many of the people you love are waiting inside the pocket computer you are never long without, so I will gently suggest that perhaps coding might connect the same way.

When you liberate program­ming from the require­ment to be profes­sional and scalable, it becomes a different activity altogether, just as cooking at home is really nothing like cooking in a commer­cial kitchen. I can report to you: not only is this different activity rewarding in almost exactly the same way that cooking for someone you love is rewarding, there’s another feeling, too, specific to this realm. I have struggled to find words for this, but/and I think it might be the crux of the whole thing:

This messaging app I built for, and with, my family, it won’t change unless we want it to change. There will be no sudden redesign, no flood of ads, no pivot to chase a userbase inscrutable to us. It might go away at some point, but that will be our decision. What *is* this feeling? Independence? Security? Sovereignty?

Is it simply … the feeling of being home?

*Update, February 2022:* Two years later, my family still uses BoopSnoop every day. I have added one (1) feature, at my mother’s request.

*Update, February 2023:* Yep, still using it every day!

*Update, February 2024:* Still booping. Still snooping. Test­Flight build 26.

February 2020, Oakland

</main>