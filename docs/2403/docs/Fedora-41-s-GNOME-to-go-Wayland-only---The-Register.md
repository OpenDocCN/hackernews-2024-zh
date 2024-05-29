<!--yml
category: 未分类
date: 2024-05-27 14:55:01
-->

# Fedora 41's GNOME to go Wayland-only • The Register

> 来源：[https://www.theregister.com/2024/03/13/fedora_41_drops_x_gnome/](https://www.theregister.com/2024/03/13/fedora_41_drops_x_gnome/)

The Fedora development team is discussing dropping the GNOME on X11 session in Fedora 41, meaning that the flagship edition will be Wayland-only.

The move comes from [ongoing online discussions](https://pagure.io/fedora-workstation/issue/414#comment-899128) and is planned for Fedora 41, which is the release after next, expected in about seven months' time. It is still under discussion: nothing is set in stone yet, but it's looking likely.

This change has been coming for a while – we [reported on the proposal](https://www.theregister.com/2023/10/13/gnome_proposes_dropping_x11/) in October last year. It's also in line with the stated direction of Fedora's parent and primary sponsor Red Hat, which announced last year that [RHEL 10 would be Wayland-only](https://www.theregister.com/2023/11/29/rhel_10_dropping_x11/).

Note, this doesn't apply to the imminent Fedora 40, expected next month. What could be interpreted as a step on this path *will* happen in Fedora 40, though. The KDE spin of Fedora 40 will include the [newly released KDE Plasma 6](https://www.theregister.com/2024/02/29/kde_plasma_60_released/), and the team has [already stated](https://fedoraproject.org/wiki/Changes/KDE_Plasma_6) that KDE-flavored Fedora will be Wayland-only by default:

So this proposed change just means that the defaults for the GNOME spin will catch up with those of the KDE spin.

The GNOME edition of Fedora has defaulted to Wayland since version 25 in 2016, and the KDE edition since version 34 in 2021 – but the X11 server was still installed, and it was possible to use it by simply choosing "GNOME on X.org" or "KDE Plasma on X.org" on the login screen.

The change doesn't mean it will be impossible to use X.org any more. All it means is that from Fedora 40, KDE users who want X.org will need to manually install it, and from Fedora 41 the same applies to GNOME users.

There are reasons to still want X11 even now: for instance, if you use some accessibility tools, or you need certain graphics driver features that only work with X.org. A fairly common example would be if you run Fedora under VirtualBox and want the additional facilities provided by the Guest Additions' graphics drivers, such as dynamic guest display resizing.

For those with recent Nvidia cards, though, it shouldn't be a problem. Recent versions of Nvidia's proprietary drivers now work fine with Wayland. Fedora contributor Neal Gompa told the *Reg*:

Fedora 40 is expected to use the [newly-released kernel 6.8](https://www.theregister.com/2024/03/11/linux_6_8_arrives/). That means people with older Nvidia cards which need [one of the "legacy" drivers](https://www.nvidia.com/en-us/drivers/unix/legacy-gpu/) – that is, version 470 or older – are possibly out of luck anyway. In our testing, we found that the version 390 legacy drivers refuse to compile and install on kernel 6.5 or newer.

Even so, it's something of a watershed moment. Wayland has been coming for years, but 2024 may prove to be the year that it takes over. Arguably, GNOME and KDE are the two most widely used, mainstream desktop environments in the FOSS world, and Fedora is both one of highest-profile distros as well as one of the most cutting-edge stable-release-cycle ones. Although Fedora has [multiple spins](https://fedoraproject.org/spins/) with other desktops, most of which aren't fully Wayland-enabled yet, this marks a pivot point for Wayland adoption. ®