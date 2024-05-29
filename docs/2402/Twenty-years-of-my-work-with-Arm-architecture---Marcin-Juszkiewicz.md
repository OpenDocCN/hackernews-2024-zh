<!--yml
category: 未分类
date: 2024-05-27 14:50:05
-->

# Twenty years of my work with Arm architecture – Marcin Juszkiewicz

> 来源：[https://marcin.juszkiewicz.com.pl/2024/02/12/twenty-years-of-my-work-with-arm-architecture/](https://marcin.juszkiewicz.com.pl/2024/02/12/twenty-years-of-my-work-with-arm-architecture/)

Twenty years ago I bought Sharp Zaurus SL-5500 Linux PDA. With StrongARM SA1110 inside. And I had no idea how it will end…

### Hobby

At first it was a hobby. Finding what I can do with it (compared to previous devices running PalmOS) was fun. SharpROM, OpenZaurus 3.2, OpenZaurus 3.3-pre1 etc. New apps, taskbar plugins…

Then I wanted to build something. Found an OpenZaurus toolchain, tried it and went for help. And got “forget this toolchain, we have a new toy: OpenEmbedded”.

And I sunk in…

Took me some time to get familiar with that (lot of n00b questions asked). Then many attempts to get a bootable image. Turned out that my image was one of first properly booting on a SL-5500 device. So I got write access to OpenEmbedded with “now you can merge all of it” message.

#### OpenZaurus

I spent three years working on OpenZaurus distribution. In my free time. Started as a user, went through being developer and [ended as a release manager](/2006/03/18/openzaurus-354-released/). Through those years I moved from my SL-5500 to the c760 (donated by user). Had several other Zaurus models on my desk in meantime.

Those were interesting years. I learnt a lot about structure of Linux filesystem, how packaging works, all those differences between build-time and install-time dependencies, sorting out upgrades etc. And how to cooperate with other developers and users. Also how to avoid those who deserve to be ignored.

### Embedded Linux developer

At one moment working with Arm architecture stopped being hobby and became daily work.

Matthew Allum from OpenedHand proposed me a full time contract so [I left my daily job](/2006/12/15/job-change/) (as a PHP programmer) and started working on Poky Linux and its version of OpenEmbedded.

New devices, new interesting challenges, new co-workers and customers. Proper development boards (those big ones with lot of interesting connectors for even more expensive equipment), prototypes of misc devices…

After OpenedHand there were other companies interested in my services. Bug Labs with their Java based system on top of Poky Linux, Vernier with school palmtops and others. Still embedded Linux.

### Invited speaker

My first conference was FOSDEM 2007\. And then were several other ones but I was an attendee rather than speaker.

2009 changed it — Andrea Gallo from ST-Ericsson invited me to give a talk at [their workshop](/2009/10/19/st-ericsson-community-workshop-2009/) in Grenoble. My presentation was about supporting their developer board NHK-15 in Poky Linux and OpenEmbedded.

Next day was [ELCE where I gave a talk](/2009/10/27/elc-e-2009/) called “Hacking with OpenEmbedded”. It was also conference where I met Leif Lindholm and Jon Masters for the first time.

### Linaro

2010 came, I had over three years of paid embedded Linux work behind me. [Canonical asked and we signed papers.](/2010/04/06/another-job-change/) Then another papers and I became one of the first 20 people working at Linaro. To work on improving Arm support in Linux.

It was time when working with Arm meant embedded work, just using main distributions instead of embedded ones. All those device specific kernels, images etc.

I did cross compilation toolchains for Debian/Ubuntu, AArch64 bring up using OpenEmbedded and several other tasks.

### Red Hat

In 2013 I ended my contract with Canonical, [left Linaro](/2013/05/31/my-time-at-linaro-is-over/) and [joined Red Hat](/2013/08/01/new-job-senior-software-engineer-red-hat/). And met the other side of Arm world.

I was not doing developer boards or SBC systems. The project was to get Red Hat Enterprise Linux distribution running on 64-bit Arm servers. Before they even existed…

It was a fun time. Native builds in very slow system emulators took hours/days. Prototype servers were faster but also very unstable.

Anyway, we delivered. [RHEL 7.2 had AArch64 support.](/2015/11/20/red-hat-enterprise-linux-server-for-arm-7-2-development-preview-released/) Countless packages got fixed both in RHEL and Fedora.

### Linaro again

In 2016 [I joined Linaro again](/2016/04/08/back-linaro-org/) (this time as Red Hat engineer). To work on Arm servers. For data centers and clouds.

Virtualization, OpenStack, Big Data, emulation of Arm servers etc.

### Acknowledgements

There were many people who helped me on this journey.

First of all Anna Wagner-Juszkiewicz — without her patience and forcing me to create my own company I would not go that far.

Then OpenEmbedded team members:

*   Philip Balister
*   Phil Blundell
*   Florian Boor
*   Holger Freyther
*   Koen Kooi
*   Christopher Larson
*   Mickey Lauer
*   Richard Purdie
*   Holger Schurig
*   and countless contributors

Some work changes related people:

*   Cliff Brake (my first freelance OE jobs)
*   Tim Bird (CELF was my first customer)
*   Matthew Allum (full time contract so I left web development)
*   Christian Reis (Canonical/Linaro job)
*   Jon Masters (“come, work at Red Hat with us”)

Linaro folks:

*   Fathi Boudra
*   Andrea Gallo
*   Gema Gomez
*   Lee Jones
*   Leif Lindholm
*   Sahaj Sarup
*   Ebba Simpson
*   Riku Voipio
*   Linus Walleij
*   Wookey

Of course I am unable to list everyone. During those twenty years I met many people, some became friends, some became adversaries. There are faces/names I remember (and more of those I should remember). And there are people who recognize me when I do not recognize them (sorry folks).

### Hall of fame shelf

There is a plan to frame several devices from my past:

*   Sharp Zaurus SL-5500 (from which all that started)
*   Atmel AT91SAM9263EK (first developer board I owned)
*   Applied Micro Mustang (first AArch64 server)

And who knows, maybe some other hardware but first would need to get it. Some already got recycled, some had to be destroyed.

### Plans for next years?

I plan to continue working on Arm architecture. It brings me fun still.

And who knows, maybe one day some standards compliant AArch64 based system will replace my x86-64 desktop. All those UEFI, ACPI, SBSA, SBBR, SystemReady buzzwords in working piece of hardware. Just without spending thousands of €€€ for it.