<!--yml
category: 未分类
date: 2024-05-27 13:34:37
-->

# 2024-04-26 microsoft at work

> 来源：[https://computer.rip/2024-04-26-microsoft-at-work.html](https://computer.rip/2024-04-26-microsoft-at-work.html)

# >>> 2024-04-26 microsoft at work ([PDF](https://computer.rip/2024-04-26-microsoft-at-work.pdf))

I haven't written anything for a bit. I'm not apologizing, because y'all don't pay me enough to apologize, but I do feel a little bad. Part of it is just that I've been busy, with work and travel and events. Part of it is that I've embarked on a couple of writing projects only to have them just Not Work Out. It happens sometimes: I'll notice something interesting, spend an evening or two digging into it, but find that I just can't make a story out of it. There isn't enough information; it's not really that interesting; the original source turned out to just be wrong. Well, this one is a bit of all three. Join me, if you will, on a journey to nowhere in particular.

One of the things I am interested in is embedded real-time operating systems. Another thing I am interested in is Unified Communications. Yet another is failed Microsoft research projects. So if you've ever heard of Microsoft At Work, you probably won't be surprised that it has repeatedly caught my eye. Most likely, you haven't heard of it. Few have; even the normal sources of information on these kinds of things appear to be inaccurate or at least confused about the details.

Microsoft went to work in the summer of 1993, or at least that's when they announced Microsoft At Work. This kind of terrible product naming was rampant in the mid-'90s, perhaps more from Microsoft than usual. MAW, as I and a few others call it, was marketed with a healthy dose of software sales obfuscation. What was it, exactly? an Architecture, Microsoft said. It would enable all kinds of new applications. With MAW, one would be able to seamlessly access the wealth of information on their personal computers. Some reporters called it an Environment. Try this for a lede: "Microsoft Corp. unveils integrated computer program."

The announcement included a demo that got a lot more to the point: a fax machine that ran Windows.

Even this was strangely obfuscated: enough newspaper reports described it as a "fax like product" that I think this verbiage was sincerely used in the announcement. Today, we would refer to MAW as an effort towards "smart" office machines, but in 1993 we hadn't quite learned that vocabulary yet. Microsoft must have been worried that it would be dismissed as "just a fax machine." It couldn't be that, it had to be something more. It had to be a "fax like product," built with "Windows architecture."

I am being a bit dismissive for effect. MAW was more ambitious than just installing Windows on a grape. The effort included a unified communications protocol for the control of office machines, including printers, for which a whole Microsoft stack was envisioned. This built on top of the Windows Printing System, a difficult-to-search-for project that apparently predated MAW by a short time, enough so that Windows Printing System products were actually on the market when MAW was announced---MAW products were, we will learn, very much not.

Windows Printing System modules were sold for at least the HP LaserJet II and III. If you did not experience them, these printers placed their actual rasterization logic onto a modular card that could be swapped out, usually to switch between PCL or PostScript "personalities." The PostScript module was offered mostly for MacOS compatibility, Apple having selected PostScript as a common printer control language. The Windows Printing System module took this operating system specialization a step further, using Windows' simple GDI graphics protocol to draw output to the printer.

I am actually a little unclear on whether or not the Windows Printing System lead directly to the cheap "WinPrinters" that are also associated with the idea of GDI-based printing. "WinPrinters," so-called by analogy to WinModems, are entirely dependent on the host computer to perform rasterization. While extremely irritating from the perspective of software support, this was an important cost-savings measure in consumer printers. Executing a capable printer control language was rather demanding; the Apple LaserWriter famously had a faster processor than the Macintosh computers it was a peripheral to. Printers with independent rasterization, particularly the more complex PostScript, came at a substantial price premium to those that required the host to perform rasterization.

While some details of reporting on the Windows Printing System make me worry that it was in fact rasterizing on device (like the curiously specific limit of "up to 79" TrueType fonts), I'm fairly sure it was indeed a precursor to the later inexpensive designs. Rather than a cost-savings measure, though, Microsoft seems to have marketed it as a premium feature. Because of the Windows Printing System's higher level of integration with the operating system, it brought numerous new features, many of which we take for granted today. TrueType font support at all, for example, a cutting-edge feature in '93\. Duplex control from the print dialog rather than the printer's own display, and for that matter, the ability to see printer status messages (like "PC LOAD LETTER") on the computer you just printed from.

And at the end of the day, offloading rasterization from the printer had an advantage: the Windows Printing System was faster than PCL or PostScript.

Even if it did become the dominant printing method years later, the Windows Printing System of the MAW era doesn't seem to have fared very well. Because it took the position of an add-on cartridge (like a font cartridge), it would have been an added-cost option for printer buyers---an added cost of $132.99, according to a period advertisement. The dearth of available documentation or even post-launch advertising for the Windows Printing System cartridge suggests disappointing sales numbers.

The fortunes of Windows Printing Technology would turn a year later, though, as Lexmark introduced their WinWriter series: "With the Microsoft Windows Printing System Built In!" Speaking of the Lexmark WinWriter series, this whole printing thing is kind of a tangent. What about MAW? The Windows Printing System, it seems, was not really a part of MAW. It was just generally related and available when MAW was announced, so it was rolled into the press conference. It is a bit ironic that the Lexmark WinWriter, truly the Printer for Windows, was *not* a MAW device despite shipping well after MAW was announced.

So, back to the course: MAW was not just Windows on a fax machine, not just the Windows Printing System, but an integrated system of Windows on a fax machine, the Windows Printing System, a generalized network protocol, and apparently a page description language. This was all, as you can see, rather document-focused. MAW would allow Windows users to easily, seamlessly interact with these common office machines, sending and receiving documents like it was 1999.

And later, it would do more: Microsoft was clear from the beginning that MAW had a higher vision, one that is remarkably similar to the later concept of Unified Communications. Microsoft envisioned Windows on a *phone,* bringing desk phones into the same architecture, or environment, or whatever. Remember the phone part, it comes back.

In practice, MAW would do nothing. It was a complete and total failure. It took two years for the first MAW office machine to reach the market, a Ricoh fax machine. Fortunately, a [television commercial](https://vimeo.com/111038922) has been preserved, giving us a small window into the Windows on a Fax Machine experience. "Microsoft's At Work Still Loafing on the Job," is how the Washington Post put it in 1995.

They call it "the first real step toward the paperless digital office," a nod towards the promise of Microsoft's document-messaging vision, before noting that virtually no products had shipped, everything was behind schedule, and Microsoft had reorganized the At Work team out of existence. Microsoft At Work was seldom spoken of again. Few products ever launched, those that did sold poorly (the Windows licensing fee imposed on them being one of several factors contributing to noncompetitive price tags), and by the time Windows gained proper USB support few would remember it had ever happened.

In other words, a classic Microsoft story.

But I'm not here to chronicle Microsoft's foibles, there are other writers for that. I'm here to chronicle their weird operating system projects. And that's what got me reading into MAW: the promise of not just one, but *two* weird operating system projects.

Regard that promise with suspicion.

Wikipedia tells us that MAW included "Microsoft At Work Operating System, a small RTOS to be embedded in devices." That's very interesting. I love a small RTOS to be embedded in devices! Tell us more.

Researching this MAW embedded operating system turns out to be a challenge. You see, it is not the better known of the operating systems produced by the MAW initiative. That would be WinPad, curiously not mentioned at all in the MAW Wikipedia article, but instead in the Windows CE article, as a precursor to CE. Windows CE gets a lot more affection than MAW, and so we know quite a bit more about WinPad. It was an early attempt at an operating system for a touchscreen mobile device, one that, in classic Microsoft fashion, competed internally with *another* project to build an operating system for a touchscreen mobile device (called Pegasus) and died out along with the rest of MAW.

It was based on 16-bit Windows 3.1, using a stripped-down UI layer that resembled Windows 95\. Probably not coincidentally, there seems to have been an effort to port WinPad onto Windows 95, and fortunately developer releases of WinPad have been preserved. With some effort, you can get them running on top of appropriate Windows versions in an emulator.

WinPad was envisioned as a core part of MAW, the key enabler of that paperless office. With MAW and WinPad, you could synchronize documents, emails, and faxes, everything you could ever want in 1995, onto your handheld device and then carry it with you. WinPad also didn't work. Evidently the performance was lousy and it required entirely unrealistic battery capacities. Not a surprising outcome when one ports a mid-'90s desktop operating system to a tablet. How charming! But not exactly my target. What about this RTOS?

If you dig into these things for too long, you start to question your life, or at least reality. References to this MAW embedded operating system are so sparse that I quickly started to wonder if it existed at all, or if it was simply confused with WinPad. This MAW OS would run directly on the office machines. Is it possible that it was, in fact, WinPad that ran on a fax machine? Or at least that whatever ran on the fax machine was a direct precursor to WinPad, an earlier new UI layer on top of 16-bit Windows?

The nagging thing that kept me on the hunt for this MAW embedded OS was, oddly enough, the Sega Saturn. A series of newspaper archives, many gathered by [Mega Drive Shock](https://mdshock.com/2020/06/22/sega-and-microsoft-announce-partnership-for-saturn-os/), tell an interesting story. Microsoft, it seemed, had been contracted to provide the operating system for the Sega Saturn. Well, this seems to have been a misconception, although clearly a period one. As the news cycle carried on, the scope of this Microsoft-Sega partnership (at first denied by Microsoft!) was reduced to Microsoft providing some sort of firmware related to the Saturn's CD drive.

There is, though, a tantalizing detail. The *Electronics Times* reported that "Microsoft looks set to port its Microsoft At Work operating system to Hitachi's new SH series of microprocessors." The article explicitly linked the porting to the Saturn effort, but also mentioned that the MAW operating system was being ported to Motorola 68000.

Do you know what never ran on the Hitachi SH or Super-H architecture? 16-bit Windows.

Do you know what did? Windows CE.

Is it possible? Do you think? Is Windows CE a derivative of Windows for Fax Machines?

I'm pretty sure the answer is no. A reader pointed me at John Murray's 1998 book *Inside Windows CE,* which provides a brief and presumably authoritative history of the platform. It specifically discusses Windows CE as a follow-on project to the failed WinPad, which it describes as 16-bit Windows 3.1, and goes on to say it "was designed for office equipment such as copiers and fax machines."

It is, of course, possible that the book is incorrect. But given the dearth of references to this MAW embedded RTOS, I think this is the more likely scenario:

MAW devices like the Ricoh IFS77 ran 16-bit Windows 3.1 with a new GUI intended to appear more modern while reducing resource requirements. Some reporters at the time noted that Microsoft was cagey about the supported architectures, I suspect they were waiting on ports to be completed. The fax machine was probably x86, though, as there's little evidence MAW actually ran on anything else.

This operating system was extended for the WinPad project, and efforts were made to port it to architectures more common in the embedded devices of the time like SH and 68000\. Microsoft may have reached some level of completion on that project and sold it to Sega for the Saturn's complicated storage controller, but it's also possible that the connection between the Saturn and MAW is mistaken and the software Microsoft delivered to Sega was a simple, from-scratch effort. The strange arc of media reporting on the Microsoft-Sega relationship offers the tantalizing possibility that Microsoft was intended to deliver a complete OS for the Saturn but had to pare it back as a result of problems with porting WinPad, but it seems more likely it just results from an overeager electronics industry press and the Sega NDA that a Microsoft spokesperson admitted to being subject to.

MAW failed to win the market, and WinPad failed to win a BillG review. The project was canceled. From the ashes of WinPad and the similarly failed Pegasus, some of the same people started work on a brand new project, Pulsar, which would become Windows CE.

MAW didn't survive the '90s.

Well, some things are like that. I still got 240 lines out of it.

Update: Alert reader abrasive (James Wah) writes in that they had previously dumped the CD-ROM firmware from the Saturn and performed some reverse engineering. Several things suggest that it was not developed by Microsoft, including a Hitachi copyright notice. It seems likely, then, that the supposed Microsoft-Sega partnership never produced anything or was never real in the first place.