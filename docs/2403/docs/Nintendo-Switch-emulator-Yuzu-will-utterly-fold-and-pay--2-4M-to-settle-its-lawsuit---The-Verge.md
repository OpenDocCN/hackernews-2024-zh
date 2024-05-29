<!--yml
category: 未分类
date: 2024-05-27 14:35:20
-->

# Nintendo Switch emulator Yuzu will utterly fold and pay $2.4M to settle its lawsuit - The Verge

> 来源：[https://www.theverge.com/2024/3/4/24090357/nintendo-yuzu-emulator-lawsuit-settlement](https://www.theverge.com/2024/3/4/24090357/nintendo-yuzu-emulator-lawsuit-settlement)

Just over a week ago, [Nintendo sued the developers of the leading Nintendo Switch emulator, Yuzu](/2024/2/27/24085075/nintendo-switch-emulator-yuzu-lawsuit), for “facilitating piracy at a colossal scale.” Now, it appears that Yuzu will give up without a fight — and give Nintendo everything it wanted. And it affects the Nintendo 3DS emulator Citra, too.

According to a joint filing, Tropic Haze has not only agreed to pay $2,400,000 to Nintendo but also says Yuzu is “primarily designed to circumvent and play Nintendo Switch games.” The company agrees to be permanently enjoined from working on Yuzu, hosting Yuzu, distributing Yuzu’s code or features, hosting websites and social media that promote Yuzu, or doing anything else that circumvents Nintendo’s copyright protection.

“Piracy was never our intention”

Oh, and it will surrender the [yuzu-emu.org](http://yuzu-emu.org) domain name to Nintendo, agree to delete not only its copies of Yuzu but also “all circumvention tools used for developing or using Yuzu—such as TegraRcmGUI, Hekate, Atmosphère, Lockpick_RCM, NDDumpTool, nxDumpFuse, and TegraExplorer,” and hand over any “physical circumvention devices” and “modified Nintendo hardware” to Nintendo. It also agrees to not delete any other “evidence” that infringes Nintendo’s IP rights.

Yuzu and Citra developer Bunnei confirmed in the Yuzu discord that both the Nintendo Switch and Nintendo 3DS emulators are affected. “We write today to inform you that yuzu and yuzu’s support of Citra are being discontinued, effective immediately,” it begins:

> Hello yuz-ers and Citra fans: We write today to inform you that yuzu and yuzu’s support of Citra are being discontinued, effective immediately.
> 
> yuzu and its team have always been against piracy. We started the projects in good faith, out of passion for Nintendo and its consoles and games, and were not intending to cause harm. But we see now that because our projects can circumvent Nintendo’s technological protection measures and allow users to play games outside of authorized hardware, they have led to extensive piracy. In particular, we have been deeply disappointed when users have used our software to leak game content prior to its release and ruin the experience for legitimate purchasers and fans.
> 
> We have come to the decision that we cannot continue to allow this to occur. Piracy was never our intention, and we believe that piracy of video games and on video game consoles should end. Effective today, we will be pulling our code repositories offline, discontinuing our Patreon accounts and Discord servers, and, soon, shutting down our websites. We hope our actions will be a small step toward ending piracy of all creators’ works.
> 
> Thank you for your years of support and for understanding our decision.

You can read through the entirety of the proposed final judgment and permanent injunction at the bottom of this story; Judge John J. McConnell [signed them, with zero changes (pdf)](https://storage.courtlistener.com/recap/gov.uscourts.rid.56980/gov.uscourts.rid.56980.11.0.pdf), on Wednesday March 6th. But on Monday, the source code [for Yuzu](https://github.com/yuzu-emu/yuzu) and [Citra](https://github.com/citra-emu/citra) had both already been pulled down from GitHub, and their websites were replaced with a copy of Bunnei’s message.

It’s not yet clear if this is the literal end of Yuzu or Citra, since copies of both emulators and their source code are in the wild. Some online supporters specifically mentioned backing up the Yuzu code after Nintendo sued two weeks ago.

I’m also not clear on whether this result could impact other emulators beyond Yuzu and Citra. If Yuzu had fought this lawsuit in court, [one of the biggest questions](/2024/2/27/24085075/nintendo-switch-emulator-yuzu-lawsuit#:~:text=bring%2Dyour%2Down%2DBIOS%20approach) would have been whether Yuzu is actually circumventing Nintendo’s protections since the emulator itself does not contain Nintendo’s keys. (Yuzu is a “bring-your-own-BIOS” emulator.) But now, Nintendo and Tropic Haze are asking a judge to specifically find that Yuzu circumvents its copyright protections by using those keys, even if it doesn’t come with them.

They are asking a federal judge to say yes to this, specifically:

> Developing or distributing software, including Yuzu, that in its ordinary course functions only when cryptographic keys are integrated without authorization, violates the Digital Millennium Copyright Act’s prohibition on trafficking in devices that circumvent effective technological measures, because the software is primarily designed for the purpose of circumventing technological measures.

But although Nintendo is making an example out of Yuzu, one that might create a chilling effect, it shouldn't create a legal precedent. "Good news is that settlements are not legal determinations even with court sign off so they are not legally precedential," Richard Hoeg, a business attorney who hosts the Virtual Legality podcast, tells The Verge.

Hoeg suspects Yuzu settled because Nintendo at least gave them a cap on their liability. "It’s a lot of money, but it’s a known amount, and I suspect the advice they were getting was that their exposure was high and they had a good chance to lose after paying lawyers for a long time."

***Update, 2:30PM ET:** Added Bunnei’s confirmation in the Yuzu discord, and their full memo.*

***Update, 2:49PM ET:** Added that Yuzu’s GitHub source code has been pulled down.*

***Update, 3:47PM ET:** Added that Citra’s GitHub source code has been pulled down.*

***Update, 6:35PM ET:** Added comment from Richard Hoeg.*

***Update, March 6th, 4:15PM ET:** Added that the settlement has been approved by a judge, with no changes.*