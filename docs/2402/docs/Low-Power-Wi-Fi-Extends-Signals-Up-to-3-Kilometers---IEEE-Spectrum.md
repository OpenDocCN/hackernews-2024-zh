<!--yml
category: 未分类
date: 2024-05-27 14:33:33
-->

# Low-Power Wi-Fi Extends Signals Up to 3 Kilometers - IEEE Spectrum

> 来源：[https://spectrum.ieee.org/wi-fi-halow](https://spectrum.ieee.org/wi-fi-halow)

Most people have probably experienced the frustration of weak Wi-Fi signals. Even getting a network to cover every corner of a fairly modest house can be a challenge. That’s not a problem for Wi-Fi technology developed by the Australian startup [Morse Micro](https://www.morsemicro.com/), which just demonstrated a Wi-Fi signal with a 3-kilometer range.

**Morse Micro has developed a [system-on-chip](https://spectrum.ieee.org/tag/system-on-chip) (SoC) design that uses a wireless protocol called [Wi-Fi HaLow](https://www.wi-fi.org/discover-wi-fi/wi-fi-certified-halow), based on the [IEEE 802.11ah standard](https://ieeexplore.ieee.org/document/7920364). The protocol significantly boosts range by using lower-frequency radio signals that propagate further than conventional Wi-Fi frequencies. It is also low power, and is geared toward providing connectivity for [Internet of Things](https://spectrum.ieee.org/iot-for-arson-forensics) (IoT) applications.**

**To demonstrate the technology’s potential, Morse Micro recently [conducted a test](https://www.youtube.com/watch?v=2xlUijXucoM) on the seafront in San Francisco’s Ocean Beach neighborhood. They showed that two tablets connected over a HaLow network could communicate at distances of up to 3 km while maintaining speeds around 1 megabit per second—enough to support a slightly grainy video call.**

**Wi-Fi HaLow Shatters Limits with a 3-Kilometer Range Morse/Micro/[youtube](https://www.youtube.com/watch?v=2xlUijXucoM)**

**“It is pretty unprecedented range,” says [Prakash Guda](https://www.linkedin.com/in/prakashguda/), vice president of marketing and product management at Morse Micro. “And it’s not just the ability to send pings but actual megabits of data.”**

**The HaLow protocol works in much the same way as conventional Wi-Fi, says Guda, apart from the fact that it operates in the 900-megahertz frequency band rather than the 2.4-gigahertz band. Lower-frequency signals are able to propagate further and are better at penetrating solid objects, which is the secret to HaLow’s much longer ranges.**

**“It is pretty unprecedented range. And it’s not just the ability to send pings but actual megabits of data.” **—Prakash Guda, Morse Micro****

**The protocol also allows channel bandwidths as narrow as 1 MHz, compared with the 20-MHz channels that are standard in Wi-Fi. Guda says this makes it possible to have many more dedicated channels that don’t interfere with one another, which is useful for IoT applications where you want to connect several devices to the same network.** 

**The trade-off is lower throughput, says Guda, but as their test shows, HaLow can still support data-intensive applications like video. The technology is also much lower power than conventional Wi-Fi, as it has in-built features that allow devices to remain dormant for long periods and wake only when they need to transmit data. This makes it feasible to power HaLow devices using batteries. Guda says Morse Micro is working with partners building battery-powered IoT devices that will need to operate for years, and in those cases, HaLow won’t run the battery down too soon.**

**But IoT connectivity is a crowded space, says [Ermanno Pietrosemoli](https://www.ictp.it/member/ermanno-pietrosemoli), a scientific consultant at the International Center for Theoretical Physics (ICTP) in Trieste, Italy, who specializes in wireless networks. “[HaLow] is a viable competitor, but there are many players in this field,” he says.** 

**The technology has a considerable range advantage over IoT-specific alternatives like [ZigBee](https://spectrum.ieee.org/busy-as-a-zigbee) or [Bluetooth](https://spectrum.ieee.org/apptricity-beams-bluetooth-signals-over-20-miles). And while the [LoRa standard](https://spectrum.ieee.org/loras-bid-to-rule-the-internet-of-things) is capable of distances of tens of kilometers, its throughput is too low to enable video or even real-time voice, says Pietrosemoli. HaLow’s high data rates could be useful for more data-heavy applications, like wirelessly connecting lots of security cameras, he adds.**

**Despite the HaLow standard being ratified in 2017, the technology has very low adoption, says Pietrosemoli, which might give developers pause due to concerns around whether the technology will continue to be supported. In contrast, competing technologies like [SigFox](https://spectrum.ieee.org/stopping-a-wildfire-with-a-lowcost-sensor-network), which operates on principles similar to those of HaLow, and [Narrowband IoT](https://spectrum.ieee.org/italy-launches-a-new-wireless-network-for-the-internet-of-things) (NB-IoT)—a low-power cellular technology—are already widely deployed.**

**[Bill Ray](https://www.gartner.com/analyst/61747), a VP analyst at Gartner, is even more circumspect. While Morse Micro has built some impressive technology, he says HaLow has failed to gain any significant traction in the industry. The fact that Qualcomm, which was an early booster of the technology, has lost interest is indicative, he adds.**

**“We are not optimistic about the standard, even if it can be stretched to 3 km,” says Ray. “Bluetooth offers greater interoperability, while LoRa offers greater range, putting HaLow in a middle position with limited application.”**

**Guda puts a different spin on it. By combining high data rates and long ranges, he says, the technology can serve the whole gamut of IoT applications. “It’s the best of both worlds,” he says.**

**He acknowledges the lack of adoption does raise concerns about interoperability and overreliance on a single vendor, but he says things are changing. California-based [Newracom](https://newracom.com/) now also produces a HaLow SoC, and in 2021 the Wi-Fi Alliance [launched a certification program](https://www.wi-fi.org/news-events/newsroom/wi-fi-certified-halow-delivers-long-range-low-power-wi-fi) for the standard that Guda says will improve interoperability.**

**“If you’re trying to make some equipment based on a new technology, you want to make sure that there’s interoperability and there’s more than one supplier,” he says. “That’s the critical mass that we have now.”**

***This article appears in the April 2024 print issue as “Wi-Fi HaLow Extends Signals to 3 Kilometers.”***

**From Your Site Articles**

**Related Articles Around the Web**