<!--yml
category: 未分类
date: 2024-05-27 12:57:30
-->

# A first look at Europe’s alternative iPhone app stores - The Verge

> 来源：[https://www.theverge.com/24100979/altstore-europe-app-marketplace-price-games](https://www.theverge.com/24100979/altstore-europe-app-marketplace-price-games)

Almost a month after Apple’s [begrudging capitulation](/2024/3/5/24090161/ios-17-4-update-released-dma-eu-third-party-app-store-nfc-payments) to the [Digital Markets Act](/24040543/eu-dma-digital-markets-act-big-tech-antitrust) (DMA), only one third-party iOS app store is currently live in Europe. It’s the B2B-focused [Mobivention](https://app-marketplace.eu/?lang=en) marketplace that allows companies to distribute their own apps internally. While that’s fine and all, things won’t stay this way for long — and it’s what’s coming soon that’ll really pique the interest of *Verge* readers.

Both [the Epic Games Store](/2024/3/8/24094543/epic-games-ios-developer-license-apple-dma) and [MacPaw’s Setapp](/2024/2/29/24086792/setapp-subscription-only-ios-app-store) have been announced, but it’s [AltStore](https://altstore.io/) that’s likely to hit EU users’ phones first. This new app marketplace from developer Riley Testut is a version of AltStore, [an App Store alternative that launched in 2019](/2019/9/25/20884363/altstore-riley-testut-delta-nintendo-emulator-ios-app-store-alternative-jailbreak) that doesn’t require users to jailbreak their devices. The primary drive for its creation was Delta, a Nintendo emulator that Testut and his business partner Shane Gill are now bringing to the iPhone through their European app marketplace. 

Currently, the new version of AltStore is deep in Apple’s approval process and will be ready to go live once it gets the thumbs up from the company. Thankfully, we’ve already had a chance to preview the marketplace and spend some time kicking its tires.

*The new AltStore is launching with both Delta game emulator and Clip clipboard apps from a single developer.*

One reason we’ve not seen more app stores launch at this point in time is partially down to Apple making it too costly. For example, its [Core Technology Fee](/2024/1/26/24051823/apple-third-party-app-stores-50-cent-fee) (CTF) requires developers to pay Apple 50 euro cents for [every annual app install *over 1 million*](https://www.apple.com/newsroom/2024/01/apple-announces-changes-to-ios-safari-and-the-app-store-in-the-european-union/)*,* butdevelopers of third-party app stores must pay the CTF [for *every* first annual install](https://developer.apple.com/support/core-technology-fee/) of their app marketplace. In other words, every download of AltStore and Mobivention costs their developers 50 euro cents — a fee that could quickly become unsustainable. The current AltStore has been downloaded over a million times, for example.

There’s no best practice guide on managing this, but Mobivention has passed the CTF fees onto its customers through [membership packages](https://app-marketplace.eu/?lang=en). At the time of writing, AltStore hasn’t announced how it plans to handle this.

Such fees aren’t financially devastating for users, but they could be enough of a blocker to stop the slightly curious from exploring alternative app stores — especially if people aren’t really sure what they’ll find there. No one likes paying for services they may not use, after all.

### Installing an app marketplace

Another potential roadblock to widespread third-party marketplace adoption is just how fiddly it is, with each store taking around a dozen screen interactions to install. 

It goes like this: you begin by clicking a browser-based link to load the alternative store. From there, you receive a pop-up informing you that your installation settings don’t allow marketplaces from that developer. Then, you head into Settings, enable the marketplace, return to your browser, click the download link again, and receive another prompt asking you to confirm the install. Finally, you can open the store and browse the available apps.

*Apple wants to make it very, very clear that installing a third-party marketplace is going to be a hassle.*

It’s not a tricky procedure to follow, but there are enough steps and scary language to make it irritating and act as a deterrent — especially when Apple’s App Store only requires a single click to get going. It’s hard to view this as anything other than the company’s attempt to sap people’s energy and dissuade them from carrying on, especially given Apple’s historical prowess at designing user experiences. 

Thankfully, installing third-party apps themselves is easier. On both Mobivention and AltStore, it’s effectively the same process as the App Store: you click on a button that says “install” and… it installs. On first inspection, at least.

While this method works for AltStore’s bundled apps — Delta and Clip — using software from other providers requires a slightly different approach. AltStore allows you to add “sources,” which are URLs developers share that contain JSON files holding app metadata. Once these sources are added, the apps they point to can be downloaded from AltStore. It’s a little *Inception*-esque: stores within a store.

Clearly, this decentralized approach differs from Apple’s all-inclusive App Store and could further deter the general public. It’s a little complicated for most people. Saying that, I’d bet a lot of enthusiasts are rubbing their hands together with glee about this unrestrained approach to app distribution. 

These sources won’t be available at release, but Testut says this is a “priority post-launch,” and there will soon be a curated list of recommended source partners to download apps from.

As I didn’t try out a source in the course of my testing, this left me to focus on the two apps available at launch: Delta and Clip. And this is where things get particularly exciting, because Delta, especially, is terrific.

### Are the apps worth all the pain?

Delta is primarily a Nintendo emulator that focuses on the NES, SNES, N64, and pre-Switch handhelds. I wasn’t expecting to be impressed by the free app, but it genuinely blew me away. Playing classic games on my iPhone is something I didn’t even know I missed. 

*Delta supports horizontal gameplay, of course.*

Actually using Delta was a breeze. You can upload ROMs via iCloud Drive or from your phone’s Download folder, and the performance while playing various titles was excellent. I will say that the controls were awkward on the touchscreen, but connecting an external controller made things much easier — even if I had a few issues accessing Delta’s menu afterward. 

All in all, though, as someone who grew up with these games, finally playing them on an iPhone feels nothing short of magical.

Clip was another app I enjoyed using. This clipboard manager requires a minimum Patreon pledge of $1 a month (plus taxes) to download. You can cancel this monthly pledge at any time and still continue to use Clip, but it won’t receive any updates.

*When you copy something, you immediately receive a notification (top of image) and can swipe down to save it to your clipboard.*

Regarding the app itself, the version of Clip I tried differs from similar software offered on Apple’s App Store in that it constantly runs in the background. Normally, clipboard managers on iOS have to use a variety of workarounds to achieve comparable functionality. For example, [Paste](https://pasteapp.io/) requires you to open the app each time you want to add something you’ve copied to the clipboard.

This is where Clip thrives, by comparison. When you copy something, you immediately receive a notification and can swipe down to save it to your clipboard. This means you have the option to add it if it’s something useful — like an address — or dismiss the notification if it’s something you don’t want logged, like a password. I found saving your copied items like this into a centralized location to be incredibly useful, as it makes sharing and reusing these snippets painless.

Clip works well, and it’s a tool I can see myself using, but it does raise some red flags. There’s a reason that Apple doesn’t allow fully functioning clipboard managers on the App Store after all. Security-wise, there’s a potential danger in allowing an app to snoop on everything you’re copying and pasting — especially if a bad actor manages to access your data store.

When I put this concern to Testut, he tells me Clip uses “standard iOS security (e.g. sandboxing)” and that everything is stored in an SQLite database, something that can’t be accessed by other apps, “unless your device is jailbroken.”

### Caveat emptor

Nevertheless, it’s these types of apps that have raised concern around using third-party marketplaces — [especially by companies like Apple](https://developer.apple.com/support/dma-and-apps-in-the-eu/). It contends that the DMA is throttling its ability to “detect, prevent, and take action against malicious apps on iOS and to support users impacted by issues with apps downloaded outside of the App Store.”

There’s some truth to that, but it’s not quite so binary. Apple still has to do a baseline review and [notarize all apps on third-party app stores](https://developer.apple.com/support/dma-and-apps-in-the-eu/) in order to “ensure [they] are free of known malware, viruses, or other security threats, function as promised, and don’t expose users to egregious fraud.” Under the DMA, Apple is also allowed to take “[necessary and proportionate](https://techcrunch.com/2024/03/07/europes-dma-rules-for-big-tech-explained/)” steps to protect users and mitigate any security issues. 

For example, after I had tested Clip, Testut had to tweak the app’s background monitoring feature in order for Apple to notarize it. The first version I tried used the user’s location to remain active, but was rejected by Apple. Testut then updated Clip with a Map feature — so there’s a reason for the app to remain active in the background — to receive approval.

This back and forth clearly shows that third-party marketplaces aren’t quite the Wild West some have feared.

This isn’t to say there aren’t dangers involved with operating outside of Apple’s walled garden though. Clip might protect your data, but what about the next app you decide to try? The sparsely populated app privacy sections on AltStore don’t help alleviate this concern, [especially compared to the App Store](/2020/12/14/22174017/apple-app-store-new-privacy-labels-ios-apps-public). Being less secure doesn’t automatically mean you’ll have your identity or data stolen, but some additional transparency related to data collection, permissions, and privacy would certainly be welcome.

Likely, the biggest hurdle for the general public to adopt third-party marketplaces will be leaving the comforting embrace of the App Store. People have been downloading apps from Apple since 2008\. Whether it’s security, user privacy, app updates, fraud protection, or refunds, you feel confident that Apple has it under control on the App Store.

Third-party app stores introduce an element of doubt. What happens if you’re [out of the EU for over a month](/2024/3/7/24093437/apple-iphone-third-party-app-store-dma-eu) and apps you depend on stop getting updates? Or you want a refund on a defective piece of software? Or an app scams you?

In the case of AltStore, Testut says that since all marketplace payments are done via Patreon pledges, Patreon will deal with any disputes as it does on the existing AltStore. Other app marketplaces will take different approaches. With Apple, you always know where you stand.

While AltStore and Mobivention aren’t well known enough to inspire confidence in the same way Apple does, other big hitters might. Both the aforementioned [Epic Games Store](/2024/3/8/24094543/epic-games-ios-developer-license-apple-dma) and [Setapp](/2024/2/29/24086792/setapp-subscription-only-ios-app-store) marketplaces are on the horizon, and their higher profiles could convince people of their ability to mitigate harm and moderate disputes. Normalizing app downloads outside the App Store will also get a boost after the spring when [Apple enables web distribution](/2024/3/12/24098334/apple-ios-web-distribution-eu-app-store-changes) for large developers.

Of course, for the public to get used to alternative marketplaces, consumer-focused ones need to launch first. While AltStore may be close to going live, the approval process has been slow and drawn out causing the launch to miss its March target.

Fundamentally, in their current state, third-party iOS app stores like AltStore will only be attractive to power users, groups of enthusiasts who are desperate to solve niche issues or have particular interests in something they can’t get on the App Store, like a fully functioning clipboard manager or game emulator.

And Apple? It’s probably pretty happy with this. The fewer things that mess with [its big old moneymaker](/2021/5/1/22414402/epic-expert-apple-app-store-fortnite-court-profit), the better — even if its approach to DMA compliance makes the company [low-hanging fruit](/2024/3/25/24111232/european-commission-digital-markets-act-investigation) for hungry EU regulators.