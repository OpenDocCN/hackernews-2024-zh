<!--yml
category: 未分类
date: 2024-05-27 15:09:00
-->

# Apple is allowing alternative browser engines in iOS 17.4 — but only in the EU - The Verge

> 来源：[https://www.theverge.com/2024/1/25/24050478/apple-ios-17-4-browser-engines-eu](https://www.theverge.com/2024/1/25/24050478/apple-ios-17-4-browser-engines-eu)

With iOS 17.4, Apple is making [a number of huge changes](/e/23814241) to the way its mobile operating system works in order to comply with new regulations in the EU. One of them is an important product shift: for the first time, Apple is going to allow alternative browser engines to run on iOS — but only for users in the EU.

Since the beginning of the App Store, Apple has allowed lots of browsers but only one browser engine: WebKit. WebKit is the technology that underpins Safari, but it’s far from the only engine on the market. Google’s Chrome is based on an engine called Blink, which is also part of the overall Chromium project that is used by most other browsers on the market. Edge, Brave, Arc, Opera, and many others all use Chromium and Blink. Mozilla’s Firefox runs on its own engine, called Gecko.

On iOS, though, all those browsers have been forced to run on WebKit instead, which means many features and extensions simply don’t work anymore. That changes with iOS 17.4 — anyone building a browser, or building an in-app browser for their app, can use a non-WebKit engine if they wish. Each developer will have to be authorized by Apple to switch engines “after meeting specific criteria and committing to a number of ongoing privacy and security mitigations,” Apple said in a release announcing the change, at which point they’ll get access to features like Passkeys and multiprocessing. Apple’s also adding a new choice screen to Safari so that when you first open the browser, you’ll be able to choose a different default if you want.

Apple is clearly only doing this because it is required to by the EU’s new Digital Markets Act (DMA), which [stipulates](https://ec.europa.eu/commission/presscorner/detail/en/QANDA_20_2349), among other things, that users should be allowed to uninstall preinstalled apps — including web browsers — that “steer them to the products and services of the gatekeeper.” In this case, iOS is the gatekeeper, and WebKit and Safari are Apple’s products and services. (The same section of the DMA also means Microsoft has to [let people disable Bing](/2023/11/16/23963579/microsoft-windows-11-eu-digital-markets-act-feature-changes) web search and uninstall Edge, and it will cause other changes, too.)

Even in its release announcing the new features, Apple makes clear that it’s mad about them: “This change is a result of the DMA’s requirements, and means that EU users will be confronted with a list of default browsers before they have the opportunity to understand the options available to them,” the company says. “The screen also interrupts EU users’ experience the first time they open Safari intending to navigate to a webpage.” Apple’s argument for the App Store has always amounted to: only Apple can provide a good, safe, happy user experience on the iPhone. Regulators don’t see it that way. And Apple’s furious about it.

Again, these changes are only for iPhone users in the EU. Apple says it allows European users to travel without breaking their browser engine but will make sure only accounts belonging to people who live in the EU get these new engines. Elsewhere in the world, you’ll still be getting WebKit Chrome and WebKit everything else. Apple argues (without any particular merit or evidence) that these other engines are a security and performance risk and that only WebKit is truly optimized and safe for iPhone users.

But in the EU, we’re likely to see these revamped browsers in the App Store as soon as iOS 17.4 drops in March: Google, for one, has been working on a non-WebKit version of Chrome for [at least a year](https://9to5google.com/2023/02/06/google-chrome-blink-ios-webkit/). European users are about to get a serious browser war on their iPhones.

***Correction January 26th, 7:45 AM ET:** This story originally referred to Chromium as a browser engine, when in reality Chromium is the overall browser project and Blink is the engine. We regret the error.*