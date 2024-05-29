<!--yml
category: 未分类
date: 2024-05-28 18:10:03
-->

# Going native: The future of the Bitwarden mobile app : Bitwarden

> 来源：[https://old.reddit.com/r/Bitwarden/comments/1b32bbz/going_native_the_future_of_the_bitwarden_mobile/](https://old.reddit.com/r/Bitwarden/comments/1b32bbz/going_native_the_future_of_the_bitwarden_mobile/)

Hi everyone. I wanted to post a quick update on the plans that are progressing around the Bitwarden mobile app. For those of you that don’t know, our current mobile app is created using a technology called Xamarin, a framework provided by Microsoft that allows you to create a single app that works on both iOS and Android. I chose Xamarin in the early days of Bitwarden because it was a technology that I was proficient at (.NET and C#) and it afforded me the time to maintain a mobile app along with all the other apps I was building for Bitwarden. Xamarin is a real time saver, for sure and it has served us well over the past 8 years, but it comes with some downsides as well:

*   Our Xamarin app doesn’t “feel native”. It’s obvious to anyone using our app that something feels off about it. The design, responsiveness, and overall usability give a negative impression compared to native apps.
*   Our Xamarin app is a bit sluggish and uses a lot more resources on your device than you might expect.
*   Microsoft is making drastic changes to Xamarin’s future and are re-developing it into a new product, now called MAUI. Support for Xamarin is ending. Unfortunately, the transition to MAUI has been a subpar experience for us.
*   Xamarin doesn’t give us access to cutting edge features. When new features come out on iOS and Android we have to wait for Microsoft to support those features in Xamarin before we can use them in our app. This is why we have been slow to adopt passkey in our mobile apps, for example.

Because of some of these things, and because we have matured as an engineering organization here at Bitwarden, Xamarin doesn’t make sense for us to pursue any longer.

Early last year we began planning to retire our Xamarin-based mobile apps and made the decision to transition our mobile apps to fully native apps written in Swift (for iOS) and Kotlin (for Android). Over the past 6 months we have been actively developing these new native apps and at this time they are nearing completion. I wanted to share some sneak peeks of these new apps and rollout plans over the coming months with you all.

# The upgrade to MAUI

In an effort to support passkeys sooner than later, we’ve had a parallel effort going on with adding passkey support in the existing Xamarin-based mobile app. This required us to “upgrade” the Xamarin app to the new MAUI framework. As anticipated, the upgrade has not been smooth, however, we are nearing the completion of that project and plan to release this temporary solution soon. Although this is largely a new app under the hood, overall, the new MAUI shouldn’t look or feel any different than the Xamarin app that we have today.

Demo video: [https://www.youtube.com/watch?v=-rVQOESKbbA](https://www.youtube.com/watch?v=-rVQOESKbbA)

# Native app release

In a few months you will begin to see our completely revamped native mobile apps roll out. These new apps will look and feel different. They are completely new Bitwarden apps. Hopefully you will notice large improvements to the overall experience of using the mobile apps. The designs are different, using all native platform controls, but the layouts still follow similar user flows that we already have.

iOS

Android

# Design iteration

Now that we have new native apps to build upon, following their initial release we also plan to begin introducing other UX improvements and redesign how you interact with certain flows throughout the app. This may include things like redesigning certain screens entirely, optimization of critical user flows, and introducing onboarding walkthroughs for new users. These types of updates are informed by usability research conducted by our product design team and tested with volunteers from the Bitwarden community.

In closing, we understand that our mobile app has lagged behind in recent years. Xamarin served us well, but it’s time to move on. When released, we hope you will all enjoy the new native apps we have been working hard at building. Your feedback is important to help make the experience of using Bitwarden great for everyone.