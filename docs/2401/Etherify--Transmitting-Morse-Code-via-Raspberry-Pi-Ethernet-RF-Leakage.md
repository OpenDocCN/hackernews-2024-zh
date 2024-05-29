<!--yml
category: 未分类
date: 2024-05-27 14:48:28
-->

# Etherify: Transmitting Morse Code via Raspberry Pi Ethernet RF Leakage

> 来源：[https://www.rtl-sdr.com/etherify-transmitting-morse-code-via-raspberry-pi-ethernet-rf-leakage/](https://www.rtl-sdr.com/etherify-transmitting-morse-code-via-raspberry-pi-ethernet-rf-leakage/)

November 10, 2020

# Etherify: Transmitting Morse Code via Raspberry Pi Ethernet RF Leakage

Over on his blog SQ5BPF has been documenting a TEMPEST experiment where he's been able to [transmit data via RF being leaked from a Raspberry Pi's Ethernet connection](https://lipkowski.com/etherify/). The idea was born when he found that his Raspberry Pi 4 was leaking a strong RF signal at 125 MHz from the Ethernet cable. He went on to find that it was easy to turn a tone on and off simply changing the Ethernet link speed with the "ethtool" command line tool. Once this was known it is a simple matter of creating a bash script to generate some morse code.

Quite amazingly the Ethernet RF leakage is very strong. With the Raspberry Pi 10 meters away, and a steel reinforced concrete wall in between, SQ5BPF was able to receive the generated morse code via an RTL-SDR connected to a PC. Further experiments show that with a Yagi antenna he was able to receive the signal from 100 meters away.

His [post](https://lipkowski.com/etherify/) explains some further experiments with data bursting, and provides links to the scripts he created, so you can try this at home.

Update - SQ5BPF also notes the following:

> The leakage differs a lot with the hardware used. The Raspberry Pi 4 is exceptional and also allows to switch the link speed quickly, so was a nice candidate for a demo, but other hardware works as well.
> 
> The first tests were done on some old laptops I had laying around, and they leak as well. Maybe someday I will publish this, but everyone of them behaves differently.