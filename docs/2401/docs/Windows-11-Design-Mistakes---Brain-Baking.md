<!--yml
category: 未分类
date: 2024-05-27 15:15:22
-->

# Windows 11 Design Mistakes | Brain Baking

> 来源：[https://brainbaking.com/post/2024/01/windows-11-design-mistakes/](https://brainbaking.com/post/2024/01/windows-11-design-mistakes/)

My mother-in-law bought a new laptop that came pre-installed with Windows 11\. I thought that was a good idea since we were looking for an easy-to-use operating system. I was wrong. This post is a reminder to myself that, next time someone needs to be introduced to the world of digital bureaucracy, I should instead install a Linux distribution.

I wonder when Microsoft started missing the mark? Probably after Windows XP? Layer upon layer upon layer of unnecessary crap eventually became the achievement called Windows 11, where the AI-assisted Bing, the ridiculous Microsoft Store, the security “enhancements”, and the unwanted integration of Xbox entertainment had me curse for almost two hours straight until I finally managed to successfully install a local government extension to get an eID card reader working.

As soon as you boot up Windows and hit the start button, I noticed lots of moving components that weren’t there last time I frequently used windows. Okay fine, that was 17 years ago, but still. Why should users have to put up with trending news articles right inside that start menu? Or with Xbox account activation questions? Or with Bing? Or with what kind of weather it is today? The flashing things on the screen, the more confused my mother-in-law is, and rightly so. So I tried to hide, disable, uninstall, revert, delete, bin, trash, kill, and burn everything I deemed unnecessary.

But I couldn’t: Windows simply wouldn’t let me. Either I couldn’t immediately find where to configure the thing—made worse by a Dutch installation—or it simply was part of the “core Windows” experience and impossible to alter.

And then I had to execute a `.EXE` install file to get an USB-powered card reader working.

But I couldn’t: Windows simply wouldn’t let me. Since when did double-clicking on an executable become impossible? Instead, the Microsoft Store opened, and a friendly message told me I shouldn’t be doing anything possibly malicious: the installer didn’t come from the verified Store. Recent macOS versions also make installing new software a progressively worse experience, but at least there I can simply press “allow”, or worst case chance the privacy & security setting to allow applications downloaded from other sources than the App Store.

It turned out that the Windows installation has something called ‘S-Mode’ enabled, where doing naughty stuff is *verboten*. This includes pressing `WIN+R` and typing in `regedit.exe`, by the way, which I also found out the hard way in another attempt to tell Windows to shut up by fiddling with the hopefully correct `HKEY_LOCAL_MACHINE` keys.

Then I tried disabling S-Mode. And you guessed it: I couldn’t. By this point the amount of curses per minute started increasing exponentially. The easiest way to disable it is through the Microsoft Store by… downloading a special piece of software that removes it? But that requires a Microsoft account which we didn’t have. Alternative options were holding down `SHIFT` while selecting ‘Reboot’ in the start menu to access a repair mode where you can boot into UEFI/BIOS mode to disable Secured Boot, but that would also permanently disable hard-disk encryption. And it didn’t work. Another option is to use the repair mode to run a command-line that *does* boot `regedit.exe` and follow steps found in questionable articles online. And it didn’t work.

By this point, I was ready to throw in the towel and simply made a stupid Microsoft account to access that downgrade software. Finally, progress! With S-Mode gone, I was able to get that card reader to work. But wait, why did the Edge browser start page suddenly change? Why are we constantly logged into that new account? After logging out and removing the association with the account, Windows also decided to throw away profile information already configured in Edge such as shortcut links on the start page that my mother-in-law used to navigate the web. Another stream of curses followed.

Then I tried uninstalling Edge but I couldn’t (I think there’s a pattern here somewhere). In true Internet Explorer fashion, the browser is hard-wired with Windows. Edge’s distracting flashing buttons and annoying AI-assisted search bar got on my nerves pretty quickly. Luckily, replacing it with another Chromium-based browser isn’t too much of a culture shock for occasional web surfers. Firefox is not an option as some Belgian online banking systems are poorly optimized for that browser and switching browsers simply is too much to ask.

Disabling automatic updates—or at least the frequent and quite annoying messages—resulted in another *wouldn’t let me*. The best I could do is disable it for 5 weeks. I am well aware of the advantages (and disadvantages) of automatic security patches, but for inexperienced computer users, these “Restart your computer!” messages are reasons to panic, not to rest assured.

* * *

Modern iterations of out-of-the-box operating systems come with a surprising amount of cruft, and that’s just sad. Microsoft isn’t the only one guilty of this: Apple’s Sonoma macOS comes with so many useless features that only take up precious solid state hard disk space. I don’t care about widgets and Siri, I want my OS to do what it has to do: help operate the system. I don’t want my OS to spy on me and to send network packets to `microsoft.com` or `apple.com` without me doing anything. I don’t want an obligatory convoluted app store. I don’t want a start menu/bar full of ads. I don’t want to create an account and to be “logged in”.

And I certainly don’t want your AI-assisted help, thank you very much.

Next time, I will start by formatting and installing Ubuntu. This little Windows adventure made me question whether or not I should switch from macOS back to Linux, [just like Seb did](https://seblog.nl/2024/01/05/1/een-nieuw-begin).

<svg class="icon icon-text"><title>tags icon</title></svg> [`design`](https://brainbaking.com/tags/design "Tag: design")