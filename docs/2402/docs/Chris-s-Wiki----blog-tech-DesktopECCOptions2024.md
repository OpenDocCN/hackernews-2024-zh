<!--yml
category: 未分类
date: 2024-05-27 14:57:05
-->

# Chris's Wiki :: blog/tech/DesktopECCOptions2024

> 来源：[https://utcc.utoronto.ca/~cks/space/blog/tech/DesktopECCOptions2024](https://utcc.utoronto.ca/~cks/space/blog/tech/DesktopECCOptions2024)

A traditional irritation with building (or specifying) desktop computers is [the issue of ECC RAM](/~cks/space/blog/tech/UseECCIrritation), which for a long time was either not supported at all or [was being used by Intel for market segmentation](/~cks/space/blog/tech/IntelCPUSegmentationIrritation). First generation AMD Ryzens sort of supported ECC RAM with the right motherboard, but [there are many meanings of 'supporting' ECC RAM](/~cks/space/blog/tech/ECCRAMSupportLevels) and questions lingered about how meaningful the support was ([recent information suggests the support was real](/~cks/space/blog/linux/AMDWithECCKernelMessages)). Here in early 2024 the situation is somewhat better and I'm going to summarize what I know so far.

The traditional option to getting ECC RAM support (along with a bunch of other things) was to buy a 'workstation' motherboard that was built to support Intel Xeon processors. These were available from a modest number of vendors, such as SuperMicro, and were generally not inexpensive (and then you had to buy the Xeon). If you wanted a pre-built solution, vendors like Dell would sell you desktop Xeon-based workstation systems with ECC RAM. You can still do this today.

Update: I forgot AMD Threadripper and Epyc based systems, which you can get motherboards for and build desktop systems around. I think these are generally fairly expensive motherboards, though.

Back in 2022, Intel introduced their [W680 desktop chipset](https://en.wikipedia.org/wiki/LGA_1700#Alder_Lake_chipsets_(600_series)). One of the features of this chipset is that it officially supported ECC RAM with 12th generation and later (so far) Intel CPUs (or at least apparently the non-F versions), along with official support for memory overclocking (and CPU overclocking), which enables faster 'XMP' memory profiles than the stock ones (should your ECC RAM actually support this). There are a modest number of W680 based motherboards available from (some of) the usual x86 PC desktop motherboard makers (and SuperMicro), but they are definitely priced at the high end of things. Intel has not yet announced [a 'Raptor Lake' chipset version of this](https://en.wikipedia.org/wiki/LGA_1700#Raptor_Lake_chipsets_(700_series)), which would presumably be called the 'W780'. At this date I suspect there will be no such chipset.

(The Intel W680 chipset was brought to my attention [by Brendan Shanks on the Fediverse](https://mastodon.social/@bshanks/111897549472732911).)

As mentioned, AMD support for ECC on early generation Ryzens was a bit lackluster, although it was sort of there. With the current [Socket AM5](https://en.wikipedia.org/wiki/Socket_AM5) and [Zen 4](https://en.wikipedia.org/wiki/Zen_4), a lot of mentions of ECC seem to have (initially) been omitted from documentation, as discussed in Rain's [ECC RAM on AMD Ryzen 7000 desktop CPUs](https://sunshowers.io/posts/am5-ryzen-7000-ecc-ram/), and [Ryzen 8000G series APUs don't support ECC at all](https://www.tomshardware.com/pc-components/cpus/amd-confirms-ryzen-8000g-apus-dont-support-ecc-ram-despite-initial-claims). However, at least some AM5 motherboards do support ECC with recent enough firmware (provided that you have recent BIOS updates and enable ECC support in the BIOS, per Rain). These days, it appears that a number of current AM5 motherboards list ECC memory as supported (although [what supported means is a question](/~cks/space/blog/tech/ECCRAMSupportLevels)) and it will probably work, especially if you find people who already have reported success. It seems that even some relatively inexpensive AM5 motherboards may support ECC.

(Some un-vetted resources are [here](https://old.reddit.com/r/truenas/comments/10lqofy/ecc_support_for_am5_motherboards/) and [here](https://forum.level1techs.com/t/am5-consumer-motherboards-with-full-reporting-and-correcting-ecc/200543).)

If you can navigate the challenges of finding a good motherboard, it looks like an AM5, Ryzen 7000 system will support ECC at a lower cost than an Intel W680 based system (or an Intel Xeon one). If you don't want to try to thread those rapids and can stand Intel CPUs, a W680 based system will presumably work, and a Xeon based system would be even easier to purchase as a fully built desktop with ECC.

(Whether ECC makes a meaningful difference that's worth paying for is a bit of an open question.)