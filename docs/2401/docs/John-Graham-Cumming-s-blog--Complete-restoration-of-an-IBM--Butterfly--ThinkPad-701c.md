<!--yml
category: 未分类
date: 2024-05-27 15:09:40
-->

# John Graham-Cumming's blog: Complete restoration of an IBM "Butterfly" ThinkPad 701c

> 来源：[https://blog.jgc.org/2023/12/restoration-of-ibm-thinkpad-701c.html](https://blog.jgc.org/2023/12/restoration-of-ibm-thinkpad-701c.html)

Just over a year ago there was a [discussion](https://news.ycombinator.com/item?id=33983088) on Hacker News about the IBM ThinkPad 701c (the one with the lovely folding "butterfly" keyboard). In particular, it was a discussion about the incredible web site [701c.org](http://701c.org) which gives very, very detailed instructions on how to dismantle, restore and preserve those lovely machines.

In the discussion a chap [said](https://news.ycombinator.com/item?id=34002977): "*I have two in non-working condition, pretty beat up. If I can make someone really happy and they want to restore them, feel free to send me an email. I wanted to do it myself for the last 10 years it so but realistically know I won’t. I’m based in NL.*" And I figured, he's not too far from me, maybe I could get them and create one restored machine from them.

A year later my fully restored (and memory upgraded!) IBM ThinkPad 701c is finished. Here it is:

It was a long journey to get here. I've [written](https://blog.jgc.org/2023/03/repairing-tiny-ribbon-cable-inside-28.html) before about the painstaking operation to repair a damaged ribbon cable for the pointer. 

This blog post is a deeper dive into going from these two machines (see below for representative pictures of the mess they were in) to the picture above. Everything I know about restoring this machine came from the 701c.org website and a couple of emails with its creator. Without him this would not have happened. Thank you!

As you can see everything is a mess. The batteries have leaked and damaged the cases, screws are missing, parts are warped, covers are missing, the rubberized paint has become a sticky mess. But at least one of the keyboards seems to be OK (but see above about the ribbon cable!).

One of the machines would actually power on with nothing on the display. The other was totally dead. Time to dismantle the two and see what's salvageable.

## Dismantling 

The 701c.org site and the IBM manual and video it links to have detailed instructions on how to take these machines apart. As I took them apart case pieces that had been soaked in battery goo just broke away in my hands. So taking them apart was a slow and delicate operation. Here are some internal pictures during that process.

One of the machines was an IBM ThinkPad 701cs (which has a less capable screen than the 701c). In addition, the screen frame just cracked apart as I was removing it. But it did have a working modem:

Both machines had extensive damage from the batteries having leaked:

The state of the batteries once I had managed to remove them from the cases:

This battery goo had damaged one bottom case to the point that it was crumbling in my hands. Luckily, the other case had survived:

Between the two machines I had enough pieces to make a single machine. The major case parts were there (they "just" needed stripping and repainting) and I had a complete set of screws as well. The screws matter because some of them are T1 torx screws (not something you want to mislay). 

Before trying to see if I could get the machine to boot into anything I first needed to clean up the battery mess.

## Cleaning up battery leakage

As you can see above the batteries had leaked inside the laptops and made a mess. Via a very slow process of using 90% isopropyl alcohol (IPA) and a sharp tool and a toothbrush all that material was removed and with a multimeter I was able to verify that all the connections were good and there were no shorts.

In the end the terminals ended up looking something like this:

You can see how the metal has been damaged but luckily the connectors are still intact. I also washed all the circuit boards with 90% IPA and replaced the old Kapton tape. Since I was going to end up building a frankenmachine I stuck coloured stickers on parts. Throughout the photos you'll see little red and green stickers indicating which original machine the parts came from.

## Why no screen?

One question that got immediately answered on dismantling the machines was "Why does one machine power on but have a blank screen?". Answer: because someone has completely removed the screen connector. I suspect one machine might have been used as a source of parts at some point.

Happily the other machine had a cable and so I was able to take multiple parts from different computers and create something that starts.

## Finding a combination of parts that work

On first boot the frankenmachine complained about three things: memory, date and time and CMOS.

All of those things are actually symptoms of the same thing: the CMOS battery is dead and has lost settings and the machine has lost its way. But it's a simple fix! Hitting ESC allowed me to run diagnostics on the machine:

It had a lot to complain about but was bootable and even managed to get into Windows 95 (for a while). It would subsequently crash because there was a problem with the machine not knowing how much RAM it had (8MB!).

## Fixing the CMOS battery

To stabilize the machine it was essential to replace the CMOS battery. It's pretty simple fix (see the 701c.org page for

[details](https://www.701c.org/projects/fixes#cmos)

).

Old battery before removal:

New battery installed:

(It was also around this time that I did the delicate ribbon cable repair for the keyboard) With the new CMOS battery in place the machine was no longer complaining about CMOS troubles.

And a run through the configuration utility and the machine was 100% stable and usable. Which brought me on to the most painful part of the operation: stripping the paint off the case. At was at this point that I began to question ever having started this project.

## A total pain: stripping and repainting

Once again the 701c.org site will really take you

[through the process](https://www.701c.org/projects/painting-decals)

, but it's a lot of work. I couldn't get the same stripper and paint as I'm in Europe so I ended up using Tamiya TS-6 Matt Black Spray Matt and Tamiya TS-80 Flat Clear Spray Matt for the case. I stripped it using BRUTUS graffiti and paint remover.

I highly recommend having a pack of different grades of sandpaper and plastic razor blades.

Rubberized paint bubbling from the graffiti remover.

Scraping off rubberized and dissolved paint using a plastic razor blade.

Time for sanding after scraping and graffiti remover seems to have done its job.

Masking before spraying.

Mid-spraying. I put on three coats of the black paint.

Left to dry for at least 48 hours.

Once painted and dried the case needed the decals re-applied. But along the way there were little touchups like re-chroming and recolouring the IBM logo for the lid.

And repairing the cover for the memory slot by gluing in place a little piece of plastic to replace the broken latch. And so on and so on.

## Decals

I bought a

[pack of decals](https://www.701c.org/shop/p/701ccs-dry-transfer-icon-set)

from 701c.org and there make a huge difference. You just have to apply them very carefully. If you're old enough to have used Letraset you'll be fine here. But work slowly and carefully.

## Final reassembly

On both machines the metal that connects the keyboard to the bottom case had broken off. Once the battery leaks and swells it applies pressure to this joint and destroys it. Here I glued it back into place with epoxy and carefully melted what little plastic dots I could to hold it into place.

The screen can be reassembled as a single unit.

You'll notice here a PCMCIA Ethernet adapter in place. I bought from eBay a complete set of accessories: docking station, cables, floppy disk drive and this PCMCIA card.

There are a whole load of small but important things that have to be done in re-assembly: glue the speaker back in place, reinsert the battery lock and its tiny spring, insert the spring and cover for the power button, don't break anything. But you can follow the 701c.org

[disassembly](https://www.701c.org/projects/teardown-guide)

instructions in reverse.

A final boot test before final reassembly. Just complaining about the lack of keyboard.

Once fully reassembled I had a fully working Windows 95 machine. 

But who can resist an upgrade?

## A memory upgrade

The machine came with 8MB of RAM but is upgradable to 40MB. Happily, it's possible to buy the needed 32MB SODIMM on eBay. Just a simple matter of slipping it into the slot on the motherboard that's accessible from the back.

The SODIMM in place.

Yes, 40MB!

## What's next?

Well, I haven't replaced the

[sleep battery](https://www.701c.org/projects/fixes#sleep)

(apparently, there's a company in Hungary that makes a replacement but they won't ship outsider Hungary). And I can live without the sleep memory mode and I actually disabled hibernate in the BIOS.

And I haven't replaced the

[main battery](https://www.701c.org/projects/battery)

. I have all the parts but need access to a large 3D printer to print the case. One day.

And the machine doesn't have access to the Internet with this version of Windows 95\. It has a helpful file called Internet.Txt that

[tells you what the Internet is](https://blog.jgc.org/2023/11/why-is-internet-so-popular-according-to.html)

! But it does have a modem and PCMCIA Ethernet so should be doable.

## But, I'm ready. 

So with this machine I'm ready.

So ready for 1995.

(If you really like retro computing I have a whole web site just for that:

[Two Stop Bits](https://twostopbits.com/news)

).