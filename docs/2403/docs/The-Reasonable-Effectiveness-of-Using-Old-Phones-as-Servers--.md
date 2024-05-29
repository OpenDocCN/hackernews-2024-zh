<!--yml
category: 未分类
date: 2024-05-29 12:48:11
-->

# The Reasonable Effectiveness of Using Old Phones as Servers -

> 来源：[https://jarbus.net/blog/phone-server/](https://jarbus.net/blog/phone-server/)

I installed [linux](https://postmarketos.org) on a OnePlus6T. Setup took less than an hour, technical issues included.

# Why[](#why)

*   $90 for 8GB RAM, 128GB storage, 2.8GHz 8-core CPU
*   [Linux on the OnePlus 6T](https://wiki.postmarketos.org/wiki/OnePlus_6_(oneplus-enchilada)) is [well documented](blog/take-the-road-most-documented/)
*   Low power consumption
*   *Tiny* footprint
*   No additional cables
*   Built-in battery backup
*   WiFi, bluetooth, speaker, screen, etc
*   Negligible environmental impact sans shipping
*   I wanted to play with Linux on ARM

I don’t need a dedicated cable to power or connect it. Running on wifi, it only needs to be plugged into my phone’s charger. Whenever I need to charge my phone, I unplug the server and plug in my phone. It takes up so little space that I *literally leave it on my night stand*.

# Unexpected Issues[](#unexpected-issues)

*   I couldn’t get the Phosh image to boot, so I used the Plasma Mobile image instead.
*   The phone would suspend after a few minutes of inactivity. This caused a lot of issues over SSH and took a while to figure out, since servers usually don’t suspend.

# Docker[](#docker)

After resolving the suspend issues, docker appears to work fine. Surprisingly, many images support ARM!

# Tailscale[](#tailscale)

I installed [tailscale](https://tailscale.com) to access the server remotely without exposing it to the internet. Tailscale is phenomenal, my grandma could set it up in less than five minutes if she needed to.

# Viability as a regular phone[](#viability-as-a-regular-phone)

The OnePlus 6T is so much faster than a [PinePhone](https://pine64.org/devices/pinephone/), the Linux phone I used previously. But Linux still isn’t ready for daily-driving:

*   I need to manually connect to wifi for each reboot
*   The app ecosystem is still a work in progress, missing polished apps for navigation, cameras, streaming, etc.
*   Traditional Linux apps like Firefox don’t scale properly

Hopefully these issues are resolved in the future, but for now, the 6T is an excellent server.

I’ve been toying with a few containers, but still haven’t decided on specific services to run yet. I’m considering:

I’ll post updates as things progress.