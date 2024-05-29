<!--yml
category: 未分类
date: 2024-05-29 13:21:37
-->

# Emerge Tools | Twitter to X Deep Dive

> 来源：[https://www.emergetools.com/deep-dives/twitter-vs-x](https://www.emergetools.com/deep-dives/twitter-vs-x)

*February 22, 2024*

As iOS apps go, Twitter was as mature as they come. Since [acquiring Tweetie in 2010](https://techcrunch.com/2010/04/09/twitter-acquires-tweetie/), Twitter has been the dopamine fix of choice for presidents and entrepreneurs alike. Everything changed in 2022 when Elon acquired Twitter, cut headcount by 75%, and rebranded it to X.

It’s very rare for a product to abruptly switch from "mature" mode back to "move fast and break things" mode, so today we’re looking at what's actually changed in the iOS app from the final version of Twitter (v9.54) to X (v10.25).

## Twitter vs X

Over the last ~6 months since Twitter became X, the iOS app has increased by 13.3 MB.

X-Ray diff visualization of how the app changed from Twitter to X

Twitter and X are mostly the same app, comprised of a few large dynamic frameworks, plugins for custom share and notification extensions, and localization resources. The iOS app has escaped Musk's predictions of a total rewrite of the whole crazy stack, but there are still a few notable differences:

## Assets

From an app build perspective, the single greatest change from Twitter to X is that the app now ships five variations of the "X" logo, each around 1-3 MB. The top-level asset catalog increased from 938 kB to 11.2 MB.

Mars X, Stars X, Earth X... must be pretty important to be taking up so much *space*.

These are alternative app icons—iOS can dynamically switch icons, but this requires 1024x1024 image assets. These assets can be heavy, especially when using a high-definition photograph of Earth.

Hey look, there’s even a "Cyber" icon!

Where the last version of Twitter contained a very small amount of potential size savings from image compression, these X images are heavier and could benefit from light compression.

While 10 MB of savings may seem like a small amount, X has a user base of millions worldwide. At this scale, even 1 MB can have a [significant environmental impact](/blog/posts/CostOfAByte).

## Dynamic Frameworks

The architecture hasn’t changed much from Twitter to X. It primarily relies on a small number of very large dynamic frameworks that have remained constant from Twitter to X. 77% of X is dylibs — 177 MB out of 231 MB.

*   `TwitterSPMMigration.framework`
*   `T1Twitter.framework`
*   `TwitterAppSPMMigration.framework`
*   `PeriscopeSDK.framework`
*   `WebRTC.framework`

Other than Twitter-native frameworks, WebRTC is a real-time internet communication framework (can also be used for video communication), and Periscope a live-streaming app that was discontinued and folded into Twitter . This small number of dylibs is ostensibly in line with Apple’s guidance since [dyld](/glossary/dyld) is optimized to link a few dynamic frameworks quickly.

The vast majority of the app code lives within these frameworks, which brings us to the largest framework in both Twitter and X.

## SPM Migration

The naming of these frameworks is revealing.

X-Ray of TwitterSPMMigration

"TwitterSPMMigration" suggests that Twitter is moving to Swift Package Manager from Cocoapods or Carthage. Yet this framework has been in the app since at least May 2022 (the first time Emerge ever analyzed the Twitter app). Back then, "TwitterSPMMigration" was ~30 MB compared to the ~80 MB it is now. The migration could still be ongoing or paused completely. Maybe Twitter/X is using it as an Umbrella framework. Your guess is as good as ours 🤷. From Twitter to X, we have seen an increase in the binary size (+3.6 MB) of "TwitterSPMMigration," but seemingly nothing significant.

## Resource Duplication

There are 3 extensions in X and Twitter:

*   `ShareExtension.appex`
*   `TwitterNotificationContentExtenison.appex`
*   `TwitterNotificationServiceExtension.appex`

All 3 of these import `TwitterAppearance.bundle`, which likely contains shared UI components. `TwitterAppeance` is also in the main Twitter target. In all, this bundle is shipped 4x in the app. Sharing this one bundle in a dylib would cut Twitter's size by 8.4 MB.

This is another context in which dynamic frameworks are the optimal solution. Dynamic frameworks are [designed](/blog/posts/static-vs-dynamic-frameworks-ios-discussion-chat-gpt) to share code and resources between two binaries—the framework lives in one place and the dynamic linker loads it into the app or extension when it’s launched.

The TwitterAppearance duplication existed before the acquisition. For the most part, the new X app hasn't added significant duplication. One exception is the "X" logos we pointed out before. In addition to the logos in the main asset catalog, the T1Twitter framework is shipping each logo twice in its asset catalog, albeit these assets are smaller at ~200 KB.

Duplication of logo assets in T1Twitter framework

## What about Grok?

In December, X announced Grok, its version of an AI chatbot. The current version of X has several classes related to the Grok functionality, which you can see below.

## Wrapping up

The Twitter app bundle hasn’t undergone any radical changes since becoming X. Whether due to an unfinished migration or a strategic tilt towards shipping fast, the architectural idiosyncrasies have remained in the app.

A closing thought: you *probably* don’t need that many logos in the app.

## Analysis Links