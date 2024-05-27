<!--yml
category: 未分类
date: 2024-05-27 13:35:39
-->

# Michael Tsai - Blog - Janky Apple ID Security

> 来源：[https://mjtsai.com/blog/2024/04/26/janky-apple-id-security/](https://mjtsai.com/blog/2024/04/26/janky-apple-id-security/)

I had [another](https://mjtsai.com/blog/2023/10/23/secondary-apple-id-mess-and-inadvertent-password-reset/) instance of my Apple ID mysteriously being locked. First, my iPhone wanted me to enter the password again, which I thought was the “normal” thing it has done every few months, almost since I got it. But after doing so it said that my account was locked.

Unlocking the account would require a 1-hour Security Delay, it said, because I had Stolen Device Protection enabled, and I was not at one of my familiar locations. *I was at home.* But I went to **Settings ‣ Privacy & Security ‣ Location Services ‣ System Services ‣ Significant Locations** to check, and for some reason the *only* location in the list was the grocery store that I go to once every two weeks. It didn’t figure out the location of the home/office where the phone spends nearly all its time and which is identified as Home in Apple Maps, Contacts, and Find My.

So I went to my Mac, where there was no delay to unlock the account. However, unlocking didn’t work. It had me enter the password, texted a code to my phone, and then wanted me to enter the password again, but the sheet was broken. I typed the password and clicked **Sign In**, and the button stayed grayed out, showed a spinner, and then stopped, but it neither accepted the password nor showed an error. It just got stuck with **Sign In** disabled. Isn’t the new System Settings great?

(Several of the other Apple ID–related sheets have odd layouts and non-standard behavior. If I were not already familiar with this being the unfortunate status quo, I might worry whether they were fake UI trying to phish me.)

(The iPhone version of System Settings also got stuck in a weird state, where the **Apple ID Suggestions** screen was showing a spinner and a **Continue** button that didn’t work. And the whole app was inset with a black border around it. I had to force-quit it. And then it got stuck again the same way.)

The only thing to do was to click **Cancel** to get out of the sheet. Both of my devices kept popping up alerts about signing in to my Apple ID, and I still didn’t want to wait an hour, so I quit System Settings and relaunched it. I followed the exact same procedure as before to unlock my account, but this time it let me do so using my Mac’s password instead of sending a code to the iPhone. And this time the final sheet asking for my Apple ID password worked.

The good news is that the phone automatically unlocked and made the Apple ID services available again. I didn’t have to enter the new password there.

The bad news is that I had to choose *another* new password for this account. And everything about this process made me feel less secure. If Stolen Device Protection doesn’t work properly, is it going to cause me real trouble sometime? Maybe I should just turn it off. Is there any way I can run my devices *without* them relying on my Apple ID? Alas, I don’t think so.

(I have another Apple ID that I use on my test Macs, and for some reason it needs to be unlocked *every* time I use it to sign in to a new installation. I’ve never been asked to reset its password, though.)

Previously:

Update (2024-04-26): [Dave Wood](https://mastodon.social/@davewoodx/112340263148377801):

> WTF #Apple. I’m minding my own business, and get an alert on my watch & phone. “Sign in with your AppleID”. Ok, why? I enter my password anyway. Then: Locked out. WTF? Then worse. I can’t unlock my account for an hour because I’m not at a familiar location. I’m home. Where I rarely leave. If my home isn’t familiar, where the hell is?

[Vini Barauna](https://mastodon.social/@vinibarauna/112340769414966434):

> Same exact thing happened to my wife’s account earlier today.

[Adam Chandler](https://mastodon.social/@AdamChandler@masto.adamchandler.me/112340816977571006):

> Both of my apple IDs just got locked and hour ago. Passwords were over 2 years old so okay, that’s probably for the best but I changed the first one while taking off from Atlanta and then when I landed in charlotte, my other one also wanted to be changed. Did it on iPad since the lock was active on my iPhone. I have 2 Mac’s at home that will need to be updated to the new passcode when I get home. I thought it’s just because I was out of the country and Apple flagged both.

[nickf](https://hachyderm.io/@nickf/112340724862583216):

> Not 20 minutes after reading your article the same thing happened to me, including having to set a new password. Weird!
> 
> Although I was at home and Stolen Device Protection did recognise that.

[Simon Harris](https://social.harukizaemon.com/@haruki_zaemon/112340697514989285):

> This happened to me less than 10 minutes ago

[nutbunnies](https://mastodon.social/@nutbunnies/112340677857034645):

> I also had this happen to me tonight. Probably a silent forced password reset for an intrusion or something

[Jonathan Wight](https://mastodon.social/@schwa/112340642293655124):

> Xcodes is causing serious problems with my AppleID (apple keeps locking it for “security reasons”).

[Mike Cohen](https://sfba.social/@mikec415/112340656547478733):

> The same thing happened to me and I wasn’t using Xcode. A few people got password reset requests this afternoon

[Marc](https://mjtsai.com/blog/2024/04/26/janky-apple-id-security/#comment-4078216):

> Same things here, and it also wiped out my application specific passwords which caused problems with several apps.

[Nic Lake](https://mastodon.social/@niclake/112340929785009766):

> Same boat. Watch, then iPhone, Mac, and Apple TV all did this. I spoke to a chat agent about it, and they wouldn’t tell me what happened, only that “sometimes random security improvements are added to your account”.

[leo](https://twitter.com/leozera/status/1784050458493059392):

> Happened to me this afternoon

[Thomas Vander Wal](https://mastodon.social/@vanderwal/112341025760658192):

> I got this on an old iPad used for listening to podcasts in the kitchen about 8pm, then all devices were locked. Only after many attempts I got my MBP connected and the iCloud pw reset. Then I could start getting all other devices unlocked with the new password.
> 
> It felt more like a hack than something Apple intended.

[Tom Bridge](https://theinternet.social/@tbridge/112340463598872085):

> Anyone else have their Apple ID locked tonight randomly? I had to re-login on all my devices after a password change and a reset of all my app-specific passwords...

[Chance Miller](https://9to5mac.com/2024/04/26/signed-out-of-apple-id-account-problem-password/):

> Apple’s System Status webpage doesn’t indicate that any of its services are having issues this evening. Still, it’s clear based on social media reports that something wonky is going on behind the scenes at Apple.

Update (2024-04-27): See also:

I had to generate a new app-specific password and add it to Fantastical before it could sync.

Although my iPhone didn’t ask for the new Apple ID password, iMessage silently failed to work. It never asked me to log in again; it just stopped receiving new messages. I toggled it off and then on again, and then it started working for new messages, but the ones sent in the interim never synced down from iCloud.

My secondary Mac did ask me to enter the new Apple ID password. It also silently stopped receiving new iMessages until I launched the Messages app, at which point it did prompt me to log in. It also never synced up the messages received while it was logged out.

[Giuseppe Carlino](https://mastodon.social/@beps/112341887668096123):

> same here with the significant locations messed up.

[Carlo Zottmann](https://mastodon.social/@czottmann@norden.social/112342375139000057):

> My iPhone’s “Significant Locations” aren’t that. Apparently I live in the woods 2km from my actual home, and the fact that I can’t get more details about the other 100s of location records it saved isn’t building confidence

[Brent](https://mastodon.online/@brentharrell/112344134088048230):

> Happened to me last night also. Had to create new password and enter new one on every device. The watch was the worst because the iPhone keyboard doesn’t allow password manager fill and had to get another device view and key on iPhone. Didn’t work after 3 attempts so I canceled out. Went back in to Settings on watch and I was logged in. Overall, took at least 1 hour to complete for all devices. And the initial unlock/reset took at least 3 attempts. Not a warm, fuzzy experience.

[John Gruber](https://daringfireball.net/linked/2024/04/27/apple-id-lockout):

> I just checked on my own iPhone, and the only two “Significant Locations” listed in Settings → Privacy & Security → Location Services → System Services → Significant Locations are “Work” and my favorite (and truly oft-visited) grocery store. But the “Work” location is centered three entire city blocks (~0.2 miles) from my home, which leaves my home just outside the radius that counts as that location. Luckily I wasn’t hit by this account lockout, but this also reassures me that I’m right to not yet have enabled Stolen Device Protection.

Update (2024-04-28): [Nick Heer](https://pxlnv.com/linklog/apple-id-lockout/):

> It is unclear to me if it is affecting *only* accounts associated in some way with a developer Apple ID. Neither of my Apple IDs — both of which are connected to developer tools — were affected by this problem.
> 
> This problem is about eighteen hours old. It would be useful if Apple said [literally anything useful](https://mastodon.social/@niclake/112340915562253907) to acknowledge the issue.

I do not use my regular Apple ID with the developers tools, and my developer Apple ID did not need to be unlocked.

[Pierre Igot](https://toot.community/@betalogue/112343197838679358):

> When your iCloud/Apple ID starts acting up in weird ways, throwing you in a Kafkaesque loop with a “locked” account and a password reset process that ends in a useless “try again later” error message, while System Status remains solidly green for all Apple services, don’t bother calling Apple about it. Even they don’t know what’s going on. Wait until the next morning, and try again, and find that somehow this time the password reset actually works.

[Francisco Tolmasky](https://mastodon.social/@tolmasky/112343254055660691):

> I checked my “Significant Locations” and all it has is a water park we went to for the first time in my life last weekend. Not my home that I literally spend 90% of my time in and is marked as My Home in Apple Maps.

[Joe Cieplinski](https://mastodon.social/@jcieplinski/112343351216165768):

> Okay. Being forced to change passwords for no reason on about a thousand devices is bad enough. Now it won’t even accept my new password when trying to generate the dozens of app-specific passwords I need.

[Ryan Jones](https://twitter.com/rjonesy/status/1784372613541757407):

> I got hit by the Apple ID bug last night. And the poor copy and layouts also had me considering my entire machine had be hacked. It was a mess.

[Ryan Jones](https://twitter.com/rjonesy/status/1784648861220254034):

> Oh christ, the Apple ID reset borked my Apple Wallet.
> 
> I need to verify (?) my cards again, of which there is no button or method. And how does one even verify Apple Cash card?
> 
> […]
> 
> Oh great, Family Sharing was turned off and errors out.
> 
> Name and Photo Sharing too. Just gone. (Even after reboot.)
> 
> Aaaaand iMessage it out of sync between devices.

Update (2024-04-29): I continue to see new reports from people encountering this, as well as reports that Apple Support continues to tell customers that there is no widespread issue. It’s disappointing that new people were still encountering the problem at least two days later and that Apple has yet to post anything on its [System Status page](https://www.apple.com/support/systemstatus/) or provide any information at all.

I decided to disable Stolen Device Protection on my iPhone, which was at home, and iOS said there would be a one-hour security delay because I was not at a familiar location. 🤦‍♂️ It said I would get a notification when the delay ended. Several hours later, the notification never came, and Stolen Device Protection is still enabled. 🤦‍♂️ I am now more determined [than ever](https://mjtsai.com/blog/2024/04/26/janky-apple-id-security/#comment-4078231) to turn it off because I do not trust that the delay works properly. I went back to the grocery store, but now that is no longer listed as a Significant Location. The only location it now shows is a gym that I rarely go to and which I last visited less recently than the grocery store. 🤦‍♂️ However, it did let me turn off Stolen Device Protection when I got home, so maybe the delay works and it’s only the notification that’s broken.

[Dave Wood](https://mastodon.social/@davewoodx/112346518495293507):

> I checked what my iPhone considers my significant locations. It’s disabled! So I have no significant locations. How does the system let me enable Stolen Device Protection without it turning on significant locations?

[Adam Chandler](https://masto.adamchandler.me/@AdamChandler/112349249791850647):

> and my AppleID is locked again. So many horror stories with iCloud locks that this is the most careful I am resetting a password ever.

[David Owens II](https://twitter.com/owensd/status/1784673004963905815):

> Password not working for my Apple ID, ok.
> 
> Try to reset, but since that’s not the “iCloud” account synced to my device but the store account, none of my “signed in devices” get notifications.
> 
> So now I have to wait three more days until I get a text to my number to reset it…

[Kirk McElhearn](https://www.intego.com/mac-security-blog/apple-id-password-reset-what-we-know/):

> Significant Locations shows 55 records on my iPhone, but it only shows one recent location. There’s no way to tell the iPhone which locations you want to consider significant, such as your home or work location, so if you have Stolen Device Protection on, you’re at the whim of Apple’s location services.

I’m not sure what’s going on here, as I’ve seen screenshots from others showing [multiple locations](https://mastodon.social/@davemark/112348959797503247). My iPhone shows only one.

> This event points out one of the risks of depending on an Apple ID. As more people depend on iCloud, getting locked out of your Apple ID can have devastating consequences. You cannot use iCloud email, IMessage, or FaceTime without this account. You cannot access personal or even work documents if you store them on iCloud. And you cannot use third-party apps that depend on iCloud, such as a calendar or contacts app.

Since an e-mail address can be necessary to access accounts (for verification or if the password needs to be reset), I think it’s a bad idea to to use an iCloud address as the login for any important accounts. This also makes me think twice about using Apple Passwords as my authenticator (actual passwords are in PasswordWallet). Hopefully, I would still be able to use the authenticator if my account were locked because the information would be locally cached. But we all know that iCloud tends to discard cached data for seemingly no reason.

> Given the scope of this issue, Apple should explain what happened. Many users were worried that someone had accessed their accounts and rushed to reset their passwords, thinking that their data could be stolen. It’s unclear how many users were affected, but users in many countries had this password reset, and some people even reported this problem occurring as late as Sunday. At the time of this writing, on Monday, April 29, Apple has said nothing.

[Pierre Igot](https://toot.community/@betalogue/112354548833053336):

> As usual, Apple screwed up, and as usual, instead of owning up to it, they are just pretending to themselves that it never happened.
> 
> In other words, Apple are being their usual arrogant selves, at the expense of their users.

Update (2024-05-01): [Pierre Igot](https://toot.community/@betalogue/112361888534658218):

> BTW, unsurprisingly, search for “significant” in Settings in #iOS returns… ∅. “Significant Locations” is actually under Privacy & Security › Location Services › System Services.
> 
> […]
> 
> Whatever they might write, a search for it (“significant” or “familiar”) in System Settings in #iOS still returns zilch.

See also: [Adam Engst](https://tidbits.com/2024/04/27/widespread-reports-of-apple-id-accounts-being-inexplicably-locked/).

Update (2024-05-03): [Warner Crocker](https://warnercrocker.com/2024/05/03/time-for-apple-to-come-clean-about-icloud-part-2/):

> Apple (hell all companies because every company is online and subject to hacks) owe users open communication at the very least. Equally as important, Apple owes its own tech support personnel open and better communication on these problems.
> 
> […]
> 
> I won’t go into a blow by blow account with my **iCloud Migraine** issues. You can find those specifics in blog posts [here](https://warnercrocker.com/2023/10/05/time-for-apple-to-come-clean-about-icloud/), [here](https://warnercrocker.com/2023/05/09/apple-icloud-migraines/), [here](https://warnercrocker.com/2023/07/25/a-possible-answer-to-those-apple-migraines/), and [here](https://warnercrocker.com/2023/05/22/apple-icloud-migraines-continue/). That said, having to re-log into Messages after this event leads me to continue to believe that Apple has deeply rooted issues with iCloud. I’ve been fighting these issues (and Apple) for well over a year.

Update (2024-05-07): [Pierre Igot](https://toot.community/@betalogue/112388728937106140):

> Latest chapter in the fallout from Great Apple ID Password Reset of April 2024: Yesterday, I tried to send a message from my mac.com email address, which is my Apple ID, using Apple’s servers, in MailMate. Because Apple BARELY supports (very begrudgingly) third-party mail clients, you need to define not one, but TWO app-specific passwords for MailMate, one for receiving mail and one for sending mail.
> 
> […]
> 
> The site… asks me to log in again. (I just did!) Fine. THEN it asks me to… confirm my Apple ID password. I then enter my NEW password (the one I reset last week), and… it tells me it’s the wrong password! I try again and again and… same thing.
> 
> So I log out altogether on the Apple ID web page and start from scratch, this time logging in with my Apple ID and the (same) new password (instead of the passkey). It works (wait, didn’t you just say the password was wrong?), but… now Apple says my account has been locked again!

Update (2024-05-09): [Andrew](https://twitter.com/andrewe/status/1788244436733899130) [Escobar](https://twitter.com/andrewe/status/1788230987387658486):

> Apple ID is either broken or being updated ahead of WWDC.
> 
> All my app-specific passwords were wiped when my account was locked on April 24[…] and I still can’t set new ones.

> I’m concerned Apple hasn’t even acknowledged the Apple ID indecent on Friday, April 26.

[Apple ID](https://mjtsai.com/blog/tag/apple-id/) [Apple Password Manager](https://mjtsai.com/blog/tag/apple-password-manager/) [iMessage](https://mjtsai.com/blog/tag/imessage/) [iOS](https://mjtsai.com/blog/tag/ios/) [iOS 17](https://mjtsai.com/blog/tag/ios-17/) [Mac](https://mjtsai.com/blog/tag/mac/) [macOS 14 Sonoma](https://mjtsai.com/blog/tag/macos-14-sonoma/) [Messages in iCloud](https://mjtsai.com/blog/tag/messages-in-icloud/) [Passwords](https://mjtsai.com/blog/tag/passwords/) [Security](https://mjtsai.com/blog/tag/security/) [System Preferences](https://mjtsai.com/blog/tag/system-preferences/) [Top Posts](https://mjtsai.com/blog/tag/top-posts/)