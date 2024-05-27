<!--yml
category: 未分类
date: 2024-05-27 13:26:09
-->

# [Michał Sapka's website] BSDs may not be a system for you

> 来源：[https://michal.sapka.me/bsd/why-not-bsd/](https://michal.sapka.me/bsd/why-not-bsd/)

Changing GNU/Linux distribution can be done on a whim, as underneath all of that you’ve got the same basic operating systems. With BSDs it’s not the same. One should try to understand the downsides, as not to waste the next 20 years exploring an OS that simply is not a good fit.

## Hardware

All BSD are much less popular than GNU/Linux^(, and with this comes the most pressing downsides. The support from hardware vendors is, at the very least, problematic. You will have problems with recently released components; most likely your WiFi card will not work. Your graphic card may have drivers, but they may work much slower. One needs to deeply examine hardware at hand and make sure it is supported by the chosen OS. As an example, my laptop has an NVIDIA card hardwired to HDMI-out, so if I want to use an external monitor, I need to use this card. While this works on FreeBSD, there is no support on OpenBSD. Another example would be the Intel wireless NIC which I had to physically replace ^(to not get constant Kernel Panics. It goes without saying that power management is also a problem, but it’s a problem on GNU/Linux as well.))

The lackluster support from vendors is handled by volunteers who try to reverse-engineer the hardware. And here comes another problem - it’s **very** hard to back port anything from GNU/Linux kernel. There is a huge difference between what GPL and BSD license allows. And even if that wasn’t the problem, the GNU/Linux driver land is full of closed blobs. You could have thought that if something is supported there, it has a beautiful open-sourced drivers. Nothing further from the truth. Ever wondered why GNU’s GUIX doesn’t support Nvidia? That’s because the drivers are provided as blobs, and therefore closed-source. As a result, the work needed for BSD support is difficult and slow^(, ^.)

## Software availability

But let’s assume this is not a problem in your case. You have been blessed by the Gods of Hardware Support. You’ve installed the system: your GPU is calculating pixels, your air is full of bits and waves. Next step? Software! And hello to another problem: software support. Most popular software is not compiled against BSD operating systems. VS Code? Steam? Microsoft Office? Firefox? Those programs may be essential for your work or evening. *BSDs on desktop* crowd is not significantly large enough for companies to want to deal with us. We are not completely lost here, as BSD is POSIX-compliant, so it’s possible to compile everything that’s running on GNU/Linux. It requires changes and a bit of luck ^(but any open-source application can be run. Unfortunately, since we are not using GNU/Linux, all hacks that make software run fast there may now work here, or even create huge problems. Firefox on FreeBSD is a huge pile of patches layered over each other. I have no idea how much work is required to support it, but it’s somewhere between “big” and “have mercy”. About that closed-source ones? No Office for you.)

## Help

Let’s once again allow our imagination run wild and assume all software you use is there, but you have a problem. You try to Kagi ^(it and nothing. Nada. GNU/Linux has plethora of sites, blogs, and vlogs. Any problem you may encounter, someone else already solved and documented. In BSD you are expected to read the manual. But wait, you may ask, RTFM? That’s rude! It is, however, true. Since all BSDs have best in class documentation, it is assumed that you will look for help there first. This also means that trivial problems may not ever find themselves a subject of any blog post. Unfortunately, non-trivial ones are also often not documented. The community is friendly and will help you, but you need to do the homework. Since the community is small, it may take some time/luck, but someone will help you help yourself. When GUN/Linux may be used by someone who has zero knowledge about the inner workings of the OS, BSD will not be so kind.)

Notice how I, like a gentleman, always mention GNU when mentioning Linux? Well, BSD teaches you why you should. Since BSD and Linux use different userland software, they are not compatible. And while yes, basic usages of such programs like ls, cat, awk is the same, GNU likes to add a lot of custom extensions. You can assume that **only POSIX** requirements are met. As a result you will find answers for your question on the interwebs which will not work for you, as they are written for GNU-flavored tooling. Unfortunately, POSIX is a weapon for a more civilized age. Folks these days assume *a lot* and BSDs don’t even come with ZSH out of the box. Ever used the basic *Shell*? Too bad, as *Shell* is what you should assume in all your scripts.

## Pace of change

Next: do you like to call yourself an *early adopter*? Being in the *bleeding edge* is what gets you going? BSD are evolving slowly by design. If something works, let’s leave it alone. That’s the mantra. GNU/Linux is changing rapidly - Pipewire, Wayland, SystemD. Even good old *ifconfig* is being deprecated. At the same time BSDs still use technology from decades ago^(. There was never a need to replace them, so no one did it.)

## It’s Linux, right?

And lastly, prepare for a lot of raised brows. Younger folks may have never even heard of BSD. Rocking their MacBooks they don’t know (nor care) about its FreeBSD roots. Very few people I have contact in the meatsphere have ever seen a BSD system, and it’s not that easy to explain that it’s not Linux.

You can think of the problems as something one may have had trying to run RedHat on a computer with a winmodem back in 1999. It’s not an OS that gets out of the way allowing you to get stuff done. You need to *enjoy* making it work for you. Otherwise, all you will find is annoyance and a swift OS change.