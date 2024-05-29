<!--yml
category: 未分类
date: 2024-05-27 15:00:59
-->

# Apple has been “augmenting reality” for years — Chris Pinola

> 来源：[https://blog.pinola.co/blog/apple-has-been-augmenting-reality-for-years/](https://blog.pinola.co/blog/apple-has-been-augmenting-reality-for-years/)

Well I tried fitting this rant into a tweet ([non-denominational](https://mas.to/@chrnola)) and realized I had quite a bit to say.

Apple’s refusal to acknowledge very real usability challenges with their products has always irked me. Not least of all because their handling of these affairs is so uniquely *them*. They employ a distinct brand of gaslighting and [blaming-the-user](https://www.orlybooks.com/_next/image?url=%2Fbook_covers%2Fblaming-the-user.jpg&w=1920&q=75) that I have come to associate solely with them. In particular, I have been increasingly annoyed about how Apple handles challenges related to multi-device interoperability features.

Until the COVID-19 pandemic hit, I had never owned an iPad. When it became apparent that I and countless others would be spending a lot more time at home for the foreseeable future, I caved and bought my first and only iPad.

I struggled to justify the expense to myself since I viewed the iPad as a toy, not a “serious” computer for “real work”^(. I use a Mac for work, and could occasionally benefit from having an extra screen in a pinch. Therefore, knowing I could use [Sidecar](https://support.apple.com/en-us/102597) (a feature which turns an iPad into a portable second display for a Mac) made me feel a bit better about the purchase. Perfect. Add to cart.)

Well imagine my surprise upon learning that Sidecar only works when both the iPad and Mac are associated with the same Apple ID^(. From the [Sidecar support docs](https://support.apple.com/en-us/102597):)

> Both devices must be signed in with the same Apple ID using two-factor authentication. Sidecar does not support Managed Apple IDs.

This poses a problem for me because I own my iPad, but I **do not** own the Mac that I use for work. As is often the case for software professionals, my employer issued me a Mac to use. To the not-sufficiently-radicalized, this seems like a luxurious perk of being an employee of such a company. And, sure, it isn’t *not* that, but the reality is a bit murkier.

Since my company owns my work Mac, they have ultimate control over it. I don’t, for instance, have (unconditional) administrator control of the device. Since they own the hardware, they are also able to manage the device via MDM tools. This enables them to, for example, install an extra trusted root CA certificate on the device and intercept all my TLS-encrypted web traffic. That plaintext traffic is then able to be inspected, logged, filtered, and attributed to me.

To be clear — I am not complaining *that* companies like my employer go to these lengths. In fact, I think the practices are justifiable given the nightmarish realities of trying to defend an organization from modern cybersecurity threats. I merely want to establish that a company laptop is a very different thing from a company car. Given all this, it blows my mind that so many of my peers are comfortable using their employer-issued devices for conducting personal business.

Imagine voluntarily trusting your employer (and usually most of their IT staff) with access to your personal text messages, DMs, photos, notes, passwords, medical records, social media activity, *details of your job search*, etc? Couldn’t be me.

My disappointing experience with Sidecar on the iPad lead me to realize that my favorite feature of the external display sitting on my desk is that it does not require me to *own* the device on the other end. The incentives for why Apple would be reluctant to support my use case are obvious. I simply have to buy more of their hardware to fix my problems. Except, I can’t in this case. The only way I’m ever going to experience Sidecar at work is if my employer issues me an iPad (which is never going to happen, nor would I want it to).

This brings me to the Apple Vision Pro. No, I don’t have one, but I am ashamed to admit that I am curious about it specifically and augmented reality generally. In fact, I’ve written this entire post wearing a (much cheaper) pair of [XREAL glasses](https://www.xreal.com/us/air), which like every other display in my life, I can connect to *any other* device, regardless of whether I own it, and it works! I keep hearing how awesome using a Vision Pro as a giant external Mac display is, and that has piqued my interest more than the entertainment features of the Vision Pro have.

At the conclusion of my in-store Vision Pro demo last week, the Apple employee who helped me asked if I was interested in purchasing the device. I told him a shortened version of my Sidecar rant and he ultimately agreed with me. “You absolutely shouldn’t!” is what he said when I mentioned I was opposed to the idea of signing in to my employer-issued Mac with my personal Apple ID^(. It wasn’t much, but I felt slightly vindicated having someone from the company at least acknowledge that this problem exists. He confessed to not being familiar enough with the Vision Pro to confirm whether I would encounter the same problem. Well, I looked into it when I got home and, lo and behold, here’s what [the Vision Pro docs](https://support.apple.com/guide/apple-vision-pro/see-your-mac-screen-tan357ede966/1.0/visionos/1.0) say about Mac Virtual Display:)

> Both devices must be signed in with the same Apple ID using two-factor authentication. Mac Virtual Display does not support Managed Apple IDs.

Foiled again! Now, I know they aren’t exactly hurting for revenue, but I bet Apple would sell even more of these things if they were to prioritize eliminating their completely self-imposed “same Apple ID” limitation. I understand that this limitation likely exists for pragmatic technical reasons (e.g. iCloud Keychain is used to facilitate key exchange between the devices) but we all know they could solve this problem if they cared to (pairing codes, etc).

Perhaps I’m simply envious because I’m uncomfortable acknowledging my own limitations. Maybe part of Apple’s success stems from their ability to ruthlessly keep feature bloat under control and build for their 90th percentile user. My people-pleasing tendencies make me pretty bad at wrangling this product complexity sometimes, so perhaps watching Apple so brazenly own the simplicity of their product evokes feelings of jealousy.

Whatever it is, I’m over it.