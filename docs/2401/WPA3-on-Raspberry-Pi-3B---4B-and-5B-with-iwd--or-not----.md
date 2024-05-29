<!--yml
category: 未分类
date: 2024-05-27 15:04:22
-->

# WPA3 on Raspberry Pi 3B+, 4B and 5B with iwd (or not...)

> 来源：[https://rachelbythebay.com/w/2024/01/24/wpa3/](https://rachelbythebay.com/w/2024/01/24/wpa3/)

## WPA3 on Raspberry Pi 3B+, 4B and 5B with iwd (or not...)

Okay, it's been several months since I [last](/w/2023/11/07/wpa3/) wrote about WPA3 on Raspberry Pi hardware. Now, I have some good news: it now mostly works, assuming you're willing to do a little tinkering. You no longer have to wrangle custom firmwares and binary blobs into place. That's been done for you.

One important thing here: I'm only talking about Raspbian/Raspberry Pi OS here, and then only bookworm (12). If you're running something else, none of this may apply. For all I know, it might have been working all along if your distribution figured it out sooner.

So then, if you have a 3B+, 4B or 5B on bookworm, get ready to rock.

Every so often I look at the list of package updates coming down the pipe through apt for my systems, and usually groan at the usual round of CVE patches. But, this time I saw something rather different. It's a change to "firmware-brcm80211", and what's this? Something specific to the CYW43455 that the 3B+ and later use? Oh, that's interesting, right?

```
  * brcm80211: cypress: Use a generic CYW43455 firmware
    - Version: 7.45.234 (4ca95bb CY) CRC: 212e223d Date: Thu 2021-04-15 03:06:00 PDT Ucode Ver: 1043.2161 FWID 01-996384e2

```

I wondered if this would stop the madness, and so applied it and rebooted. "iw phy" showed the good news - at the very bottom, it now supports both SAE_OFFLOAD and SAE_OFFLOAD_AP. This means it can actually do SAE... but you're going to have to say goodbye to wpa_supplicant in favor of iwd.

If you switched over to NetworkManager for consistency with the rest of the bookworm changes, this is not a big deal. If you're running your network some other way, you get to figure this out yourself.

The steps work out like this: apt update, then apt upgrade so you get the firmware-brcm80211 from January 17th (or later, once this post gets old, I guess). Then apt install iwd.

Assuming you've been running your Pi in a crappy non-WPA3 network, go into nmtui or equivalent and disable that network. In a minute or two, you're not going to need it any more.

Then disable wpa_supplicant and drop a bit of config into /etc/NetworkManager/conf.d/iwd.conf:

```
[device]
wifi.backend=iwd

```

Reboot ... or do equivalent wrangling of drivers and binary crap to get it to unload and reload the fresh stuff. Then run "iw phy" again. If it says SAE_OFFLOAD and SAE_OFFLOAD_AP, you're ready to proceed. If not, well, something's wrong, and you should turn on the crappy old non-WPA3 network again.

Assuming it worked, then go into nmtui or whatever and tell it to activate the actual WPA3-only network. Paste in your PSK and let it go, and a few seconds later you should be in business.

That's it. That's all it takes.

Now then, what about the 3B, you might ask. It's on a different chip that doesn't even do 5 GHz, so changing the 43455 firmware wouldn't help any. It seems to be a 43430 instead, and I have no idea if there's any chance of getting a similar firmware change for it. Obviously, if this changes, I'll post something about it, too.

I can't comment on the other models, like the Zeroes and the other weird little boards they have. I only have access to these relatively normal models, and that's what I was able to work on.

Go forth and have better networks!

* * *

January 24, 2024: This post has an [update](/w/2024/01/24/fail/).