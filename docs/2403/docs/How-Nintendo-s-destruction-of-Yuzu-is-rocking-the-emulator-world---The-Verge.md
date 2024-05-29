<!--yml
category: 未分类
date: 2024-05-27 14:56:27
-->

# How Nintendo’s destruction of Yuzu is rocking the emulator world - The Verge

> 来源：[https://www.theverge.com/24098640/nintendo-emulator-yuzu-lawsuit-switch-aftermath](https://www.theverge.com/24098640/nintendo-emulator-yuzu-lawsuit-switch-aftermath)

[When Nintendo sued the developers of Yuzu out of existence on March 4th](/2024/3/4/24090357/nintendo-yuzu-emulator-lawsuit-settlement), it wasn’t just an attack on the leading way to play Nintendo Switch games without a Switch. It was a warning to anyone building a video game emulator.

Seven developers have now stepped away from projects, are shutting them down, or have left the emulation scene entirely. Of those that remain, many are circling the wagons, getting quieter and more careful, trying not to paint targets on their backs. Five developers declined to talk to *The Verge*, telling me they didn’t want to draw attention. One even tried to delete answers to my questions after we’d begun, suddenly scared of attracting press.

In addition to Yuzu, the emulation scene lost these in a single week:

*   The Citra emulator for Nintendo 3DS is gone
*   The Pizza Boy emulators for Nintendo Game Boy Advance and Game Boy Color are gone
*   The Drastic emulator for Nintendo DS is free for now and will be removed
*   The lead developer of Yuzu and Citra has stepped away from emulation
*   The lead developer of Strato, a Switch emulator, has stepped away from emulation
*   Dynarmic, used to speed up various emulators including Yuzu, has abruptly ended development
*   One contributor on Ryujinx, a Switch emulator, has stepped away from the project
*   AetherSX2, a PS2 emulator, is finally gone (mostly unrelated; [development was suspended a year ago](https://www.reddit.com/r/EmulationOnAndroid/comments/1193rxk/aethersx2_is_dead_and_we_are_the_reason/))

Not everyone is so afraid. Four other emulator teams tell me they’re optimistic Nintendo won’t challenge them, that they’re on strong legal footing, and that Yuzu may have been an unusually incriminating case. One decade-long veteran tells me everyone’s just a *bit* more worried.

But when I point out that Nintendo didn’t have to prove a thing in court, they all admit they don’t have money for lawyers. They say they’d probably be forced to roll over, like Yuzu, if the Japanese gaming giant came knocking. “I would do what I’d have to do,” the most confident of the four tells me. “I would want to fight it... but at the same time, I know we exist because we don’t antagonize Nintendo.”

[There’s a new meme](https://www.reddit.com/r/yuzu/comments/1bbaq4s/the_real_current_situation/) where Yuzu is the mythical Hydra: cut off one head, and two more take its place. It’s partly true in how multiple forks of Yuzu (and 3DS emulator Citra) sprung up shortly after their predecessors died: Suyu, Sudachi, Lemonade, and Lime are a few of the public names. But they’re not giving Nintendo the middle finger: they’re treating Nintendo’s lawsuit like a guidebook about how **not** to piss off the company.

In its legal complaint, Nintendo claimed Yuzu was “facilitating piracy at a colossal scale,” giving users “detailed instructions” on how to “get it running with unlawful copies of Nintendo Switch games,” among other things. Okay, no more guides, say the Switch emulator developers who spoke to me.

They also say they’re stripping out some parts of Yuzu that made it easier to play pirated games. [As *Ars Technica* reported](https://arstechnica.com/gaming/2024/03/heres-how-the-makers-of-the-suyu-switch-emulator-plan-to-avoid-getting-sued/), a forked version called Suyu will require you to bring the firmware, title.keys, and prod.keys from your Switch before you can decrypt and play Nintendo games. Only one of those was technically required before. (Never mind that most people don’t have [an easily hackable first-gen Switch](/2018/4/24/17276214/nintendo-switch-hack-jailbreak-security-homebrew-apps-games) and would likely download these things off the net.)

The developer of another fork tells me he plans to do something similar, making users “fend for yourself” by making sure the code doesn’t auto-generate any keys.

*A Suyu moderator sums it up for us.*

Most developers I spoke to are also trying to make it clear they aren’t profiting at Nintendo’s expense. One who initially locked early access builds behind a donation page has stopped doing that, making them publicly available on GitHub instead. The leader of another project tells me nothing will ever be paywalled, and for now, there’s “strictly no donation,” either. When I ask about the Dolphin Emulator, which [faced a minor challenge from Nintendo](/2023/6/1/23745772/valve-nintendo-dolphin-emulator-steam-emails) last year, I'm told it [publicly exposes its tiny nonprofit budget](https://opencollective.com/dolphin-emu) for anyone to scrutinize.

But I don’t know that these steps are enough to prevent Nintendo from throwing around its weight again, particularly when it comes to emulating the Nintendo Switch, its primary moneymaker. One thing to know about this whole situation: the stakes are higher than they’ve ever been. Emulators have typically lagged behind new console generations since it can take a lot of horsepower to digitally replicate the functionality of a console and time to learn its secrets. But that hasn’t been true for Switch.

Not only did hackers find [an unprecedented vulnerability](https://medium.com/@SoyLatteChen/inside-fus%C3%A9e-gel%C3%A9e-the-unpatchable-entrypoint-for-nintendo-switch-hacking-26f42026ada0) in the original Switch less than a year after release, the emulator scene also managed to develop software that plays Switch games *better than the Switch itself* within its own lifespan. They muscled in on Nintendo’s turf by competently running on Switch-style portable devices like the Steam Deck as well as some phones. YouTubers published tutorials on emulating Switch games, some of which Nintendo [reportedly threatened with copyright claims](https://www.eurogamer.net/steam-deck-nintendo-emulation-videos-are-disappearing-from-youtube); Valve even ([briefly](https://kotaku.com/yuzu-nintendo-switch-emulator-steam-deck-dock-valve-1849632836)) showed the Yuzu emulator in an official Steam Deck video.

But the speed of technology may also make it easier for Nintendo to challenge emulators, several developers admit. While Nintendo didn’t begin properly introducing encryption until the Wii, the Switch apparently has five different layers of encryption, and Nintendo’s complaint against Yuzu primarily alleged that the emulator was encouraging users to circumvent them.

That’s an argument that hasn’t yet been tested by the courts, and there’s at least one major reason to think Nintendo might not win if it tried. Section 1201 of the Digital Millennium Copyright Act *does* allow people to narrowly circumvent copy protection for “interoperability of an independently created computer program with other programs,” and the Dolphin Emulator team [has publicly argued](https://dolphin-emu.org/blog/2023/07/20/what-happened-to-dolphin-on-steam/) that would even extend to sharing decryption keys.

But many earlier emulators would never have faced that kind of challenge, and again, no developer has the money to fuck around and find out. “We’re lucky that the systems we’re using are simpler, they don’t have the advanced encryption techniques,” says one developer who focuses on early Nintendo consoles.

Another suggests that the safest emulators are for consoles so old or emulated at so high a level, they don’t even require users to bring a copyrighted BIOS. Sony PSP emulator PPSSPP, for example, [can help defend itself by saying](https://www.ppsspp.org/docs/faq/#:~:text=Do%20I%20need%20a%20BIOS%20file%20to%20run%20PPSSPP%2C%20like%20I%20would%20with%20PSX/PS1%20and%20PS2%20emulators%3F) it emulates everything, including a BIOS and operating system, using its own code.

But the longer I talk to them, the clearer it becomes that the emulator faithful have been slightly shaken by the collapse of Yuzu. None seem completely convinced that Yuzu was doing it wrong.

First rule of emulator club: you do not talk about piracy

“The Yuzu Discord was really careful about not mentioning piracy — they even scanned the logs from bug reports to check whether the people are using self-made copies of games,” one fork contributor tells me.

“They had a metaphorical gun to their heads,” says another, calling bullshit [on Yuzu’s admission](/2024/3/4/24090357/nintendo-yuzu-emulator-lawsuit-settlement#:~:text=also%20says%20Yuzu%20is%20%E2%80%9Cprimarily%20designed%20to%20circumvent%20and%20play%20Nintendo%20Switch%20games.%E2%80%9D) that it was “primarily designed to circumvent and play Nintendo Switch games” and thus broke the law.

None of the developers I spoke to have a tremendous amount to lose. It’s not their livelihoods at stake, and they don’t have extra mouths to feed. But they’re still relying on the good graces of companies like Nintendo.

You can reach me at sean@theverge.com if you have inside info about threats to emulation — or what actually happened to Yuzu.

The big companies do sometimes profit from letting emulation run its course. You can download classic games for the Nintendo Switch and PlayStation 5 *today* because of emulators. Nintendo is said to have hired from the scene for [its own in-house emulation team](https://www.nerd.nintendo.com/), and Sony definitely did, hiring at least one developer of the PlayStation 2 emulator PCSX2 [to bring PS2 games to PS4](https://www.timeextension.com/features/meet-the-company-bringing-classic-games-to-switch-ps5-and-xbox-by-mistake).

Today, his company, Implicit Conversions, works with Sony to put PS1 and PSP games on PS5, with downloadable Sony games [like *Twisted Metal* and *Syphon Filter*](https://www.implicitconversions.com/#:~:text=identification%20purposes%20only.-,Shipped%20Titles,-About%20Us)quietly powered by emulation.It’s one reason to be optimistic: maybe Nintendo will see at least some benefit in leaving the rest of the scene alone.