<!--yml
category: 未分类
date: 2024-05-27 15:04:21
-->

# secd chews up CPU cycles and Watch battery life goes in the terlet | Ars OpenForum

> 来源：[https://arstechnica.com/civis/threads/secd-chews-up-cpu-cycles-and-watch-battery-life-goes-in-the-terlet.1485088/](https://arstechnica.com/civis/threads/secd-chews-up-cpu-cycles-and-watch-battery-life-goes-in-the-terlet.1485088/)

tl;dr When secd runs amok on my Macs my Apple Watch burns through battery

Looking for any similar experiences -- maybe we can solve this

I have two M1 Macs running macOS Monterey 12.4 signed in to iCloud and using iCloud Keychain. Periodically I will notice either: a macOS system process called secd is using ~95% of a CPU core on my Mac, and then I'll see that my Apple Watch's battery is consuming battery at an alarming rate; or my Watch battery is inexplicably low for that time of day, then check on one Mac or the other and see secd running wild. (When secd starts gobbling CPU on either Mac, in almost all cases the other Mac will soon show the same issue.)

(I first noticed this secd issue in August or September 2021, prior to Monterey's release; at that time I had the 13" M1 MacBook Pro and a 2018 i7 Mini, running whatever was the up-to-date version of Big Sur. I now have an M1 Mini and the 14" MacBook Pro with the M1 Pro, because apparently the more Apple punishes me with software glitches, the more of their stuff I'll buy.)

44mm Apple Watch 6, cellular, replaced under AppleCare 7 months ago; when the gremlins aren't plaguing me, I take it off every night at bedtime after 16 hrs of wear with between 35-50% battery remaining (depends on that day's workout.)

When secd is running wild on my Macs, my Watch battery will begin the day at 100% and fall below 10% after 6 or 7 hours of wear.

Turning off "always on" on the Watch display (so the screen goes dark automatically after a few seconds) does not appreciably change battery consumption when secd is running amok on my Macs. Cellular is enabled on my Watch, but normally my phone is with me, even on my workouts, so I don't think cellular usage/battery drain is part of my problem. (When secd is quiescent on my Macs, cellular and GPS usage on the Watch doesn't drain my Watch battery before bedtime.)

When secd CPU usage is running at the problematic ~95%, when I open Keychain Access and select the "iCloud" keychain, then Edit > Select All (to highlight every item/entry in the keychain), all items in the keychain will remain selected for perhaps half a second, then fall back to just a single item being selected -- as if there is an underlying event that is incessantly pinging or refreshing the iCloud keychain in the background. (When secd is quiet, I can highlight every entry in the iCloud keychain and all items remain highlighted indefinitely -- in the course of writing up this post I inadvertently left all items in the iCloud keychain highlighted for almost an hour.)

I don't really love the idea of secd running out of control on my Macs, but the real problem is the Watch battery.

The only fix is a PITA: on both Macs, go into ~/Library/Keychains, where there's a GUID-labeled folder (folder name is like 36 characters long, plus another 4 hyphens); delete the folder and restart the computer. (A bunch of hassle follows, signing back into iCloud and re-authenticating iCloud Keychain and email accounts, etc.), but once this secd issue begins to plague me and make the Watch burn battery, the problem continues on the Watch until it gets exorcised. (The Watch will not fall back to normal battery usage on its own.) I have to unpair the Watch from my iPhone and set it back up (I always set it back up from scratch, for fear that restoring from backup will simply restore the battery usage problem.)

Dunno what sets off the secd problem; there's no action or event I can point to. For a few months this winter it seemed to recur on a weekly basis -- overnight on Sundays, if I recall correctly; lately I may get 10 days of peace before something kicks it back off.

Apple Support was no help (several calls over the past 8 months, some many hours in duration); searching the web brings evidence of other people noticing secd chewing up CPU cycles, but I haven't found any fix. In February I: unpaired my Watch; signed out of iCloud on my Mac Mini and wiped the drive & did a fresh install of Monterey, then left it sitting until I: signed out of iCloud on both my iPad Pro and iPad Mini, then wiped them using Settings > Transfer or Reset; disabled iCloud Keychain & signed out of iCloud on my iPhone; then, using my MacBook Pro, working one line at a time (because the secd glitch wouldn't let me Select All), deleted every item from iCloud Keychain (so to the extent that *I* can control things, I had nuked everything iCloud Keychain stashes on the iCloud servers.) I then wiped the MacBook Pro, also, and did a fresh install of Monterey, then painstakingly signed my iPhone back into iCloud and began the long process of restoring all the other devices. My relief from secd lasted perhaps a month. Since then I've just been waiting 10 or so days between relapses, then restoring normalcy by nuking the GUID folder from Keychains and unpairing/re-pairing my Watch.

Help.