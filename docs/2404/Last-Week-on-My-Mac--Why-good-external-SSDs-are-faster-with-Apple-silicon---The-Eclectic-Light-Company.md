<!--yml
category: 未分类
date: 2024-05-27 13:13:42
-->

# Last Week on My Mac: Why good external SSDs are faster with Apple silicon – The Eclectic Light Company

> 来源：[https://eclecticlight.co/2024/04/14/last-week-on-my-mac-why-good-external-ssds-are-faster-with-apple-silicon/](https://eclecticlight.co/2024/04/14/last-week-on-my-mac-why-good-external-ssds-are-faster-with-apple-silicon/)

Two two-letter words can cover a multitude of performance sins. *Up to* almost invariably sets a target that can never be achieved, and one spec’s *up to* may be very different from another. I’m thinking here of Thunderbolt and USB4, both of which claim *up to* 40 Gb/s, but fail to achieve it in different ways, and with different types of Mac. To understand this, we need to read the fine print with care.

#### Different Macs

Look at the fine print in Apple’s specifications for its original [Mac mini M1](https://support.apple.com/111894). It has two “Thunderbolt / USB 4 ports” with support for:

*   Thunderbolt 3 (up to 40 Gb/s)
*   USB 4 (up to 40 Gb/s)
*   USB 3.1 Gen 2 (up to 10 Gb/s).

Compare that with one of the last Intel Macs, such as a [MacBook Pro (16-inch, 2019)](https://support.apple.com/111932), with its support for:

*   Thunderbolt (up to 40 Gb/s)
*   USB 3.1 Gen 2 (up to 10 Gb/s)

with no mention of USB4, but still claiming the same *up to* 40 Gb/s.

#### Different *up tos*

According to Intel, [Thunderbolt 3](https://www.thunderbolttechnology.net/sites/default/files/18-241_Thunder7000Controller_Brief_FIN_HI.pdf) and 4 offer up to 4 lanes of PCI Express 3.0 totalling 32.4 Gb/s for general-purpose data transfer, and 4 lanes of DisplayPort 1.4 HBR3 for video. So the fastest we can expect from an Intel Mac’s Thunderbolt ports is 32.4 Gb/s when transferring data with an external SSD. We know from long and sometimes bitter experience that means read and write speeds of up to 3 GB/s, often significantly less than that. So much for Thunderbolt’s *up to* 40 Gb/s.

In contrast, [USB4](https://en.wikipedia.org/wiki/USB4) can support what is technically known as USB4 Gen 3×2, marketed as USB 40Gbps, with a nominal transfer rate of 40 Gb/s or up to 4.8 GB/s. Because of protocol overheads, in practice that means a suitably fast NVMe SSD should clock read and write speeds of well over 3 GB/s, possibly even towards 4 GB/s. That’s USB4’s *up to* 40 Gb/s.

So the fastest you’ll see with your Intel Mac’s Thunderbolt ports is up to 3 GB/s, while a suitable USB4 enclosure running from any Apple silicon Mac should comfortably exceed that if it really does support USB 40Gbps.

#### Down to 10 Gb/s

Ready-made external SSDs and enclosures rely on different chips to deliver Thunderbolt and USB4 support, although few vendors go to the trouble of making this explicit. More expensive certified Thunderbolt models should use Intel chips, but some of those can’t manage up to 40 Gb/s, only up to 20 Gb/s. More recent hardware may opt instead for two chips, one to support Thunderbolt, the other for a consolation prize to those whose computers are stuck with some variant of USB 3.

The latest may instead target USB4 at 40 Gb/s and fall back to USB 3.1 Gen 2 when that isn’t available. Those therefore run at better than 3 GB/s with the USB4 support in Apple silicon Macs, but only 1 GB/s of USB 3 when connected to a port on an Intel Mac. The better vendors will make that clear when describing their products, but when you buy a brand with an ad hoc name that you’ve not seen before, you may not realise this until you connect the SSD to your Mac.

#### Hubs?

But what happens if you connect a USB 40Gbps enclosure via a “Thunderbolt 4” hub? That will depend on which modes the hub can operate in, both to the enclosure and back to the Mac, although no hub that I’ve seen makes that clear.

In practice, with my Satechi Thunderbolt 4 Slim Hub I can get 3.0 GB/s write and 3.2 GB/s read speeds with a USB4 enclosure, confirming that it’s still exceeding the speeds of a directly connected Thunderbolt 3 SSD without USB4 support. Whether that’s also true of other recent “Thunderbolt 4” hubs, such as the CalDigit Element and Plugable Thunderbolt 4, I don’t know.

#### Over and out

After several days testing the latest Express 1M2 enclosure from OWC, I have changed my recommendations for the best external SSDs. Previously I had chosen the relatively reliable Thunderbolt 3 *up to* 3 GB/s, even though few drives ever seemed capable of achieving that *up to.* If you’re still needing good performance with an Intel Mac, that makes sense.

But if you need best performance with an Apple silicon Mac, you’re far better off with a high-quality USB 40Gbps enclosure such as OWC’s Express 1M2, which should reliably return over 3 GB/s even through a compatible hub. I much prefer the word *over* to *up to.*