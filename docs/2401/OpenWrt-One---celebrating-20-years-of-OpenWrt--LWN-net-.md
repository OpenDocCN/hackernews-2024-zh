<!--yml

分类：未分类

日期：2024-05-27 14:40:17

-->

# OpenWrt One - 庆祝 OpenWrt 20 周年 [LWN.net]

> 来源：[`lwn.net/ml/openwrt-devel/a8aaa495-da0b-4ddc-8c4f-3e1192d8b012@phrozen.org/`](https://lwn.net/ml/openwrt-devel/a8aaa495-da0b-4ddc-8c4f-3e1192d8b012@phrozen.org/)

**线程信息**



[搜索 openwrt-devel 存档

]

| **发件人**： |   | 约翰·克里斯宾 <john-AT-phrozen.org> |
| --- | --- | --- |
| **发送至**： |   | OpenWrt 开发列表 <openwrt-devel-AT-lists.openwrt.org> |
| **主题**： |   | OpenWrt One - 庆祝 OpenWrt 20 周年 |
| **日期**： |   | 2024 年 1 月 9 日，星期二 11:49:56 +0100 |
| **消息 ID**： |   | <a8aaa495-da0b-4ddc-8c4f-3e1192d8b012@phrozen.org> |
| **抄送**: |   | compliance-AT-sfconservancy.org |

```
tl;dr

In 2024 the OpenWrt project turns 20 years! Let's celebrate this 
anniversary by launching our own first and fully upstream supported 
hardware design.

If the community likes the idea outlined below in greater details, we 
would like to start a vote.

---

The idea

It is not new. We first spoke about this during the OpenWrt Summits in 
2017 and also 2018\. It became clear start of December 2023 while 
tinkering with Banana Pi style devices that they are already pretty 
close to what we wanted to achieve in ’17/‘18\. Banana PIs have grown in 
popularity within the community. They boot using a self compiled Trusted 
Firmware-A (TF-A)and upstream U-Boot (thx MTK/Daniel) and some of the 
boards are already fully supported by the upstream Linux kernel. The 
only nonopen sourcecomponents are the 2.5 GbE PHYandWi-Fi firmware 
blobsrunning on separate cores that areindependent of the main SoC 
running Linuxand the DRAM calibration routines which are executed early 
during boot.

I contacted three project members (pepe2k, dangole, nbd) on December 6th 
to outline the overall idea. We went over several design proposals, At 
the beginning we focused on the most powerful (and expensive) 
configurations possible but finally ended up with something rather 
simple and above all,feasible. We would like to propose the following as 
our "first" community driven HW platform called "OpenWrt One/AP-24.XY".

Together with pepe2k (thx a lot) I discussed this for many hours and we 
worked out the following project proposal. Instead of going insane with 
specifications, we decided to include some nice features we believe all 
OpenWrt supported platforms should have (e.g. being almost 
unbrickablewith multiple recovery options, hassle-free system console 
access, on-board RTC with battery backup etc.).

This is our first design, so let's KiSS!

Hardwarespecifications:

* SOC: MediaTek MT7981B
* Wi-Fi: MediaTek MT7976C (2x2 2.4 GHz + 3x3/2x2 + zero-wait DFS 5Ghz)
* DRAM: 1 GiB DDR4
* Flash: 128 MiB SPI NAND+ 4 MiB SPI NOR
* Ethernet: 2x RJ45 (2.5 GbE + 1 GbE)
* USB (host): USB 2.0 (Type-A port)
* USB (device, console): Holtek HT42B534-2 UART to USB (USB-C port)
* Storage: M.2 2042 for NVMe SSD (PCIe gen 2 x1)
* Buttons: 2x (reset + user)
* Mechanical switch: 1x for boot selection (recovery, regular)
* LEDs: 2x (PWM driven), 2x ETH Led (GPIO driven)
* External hardware watchdog: EM Microelectronic EM6324 (GPIO driven)
* RTC: NXP PCF8563TS (I2C) with battery backup holder(CR1220)
* Power: USB-PD-12V on USB-C port (optional802.3at/afPoE via RT5040 module)
* Expansion slots: mikroBUS
* Certification: FCC/EC/RoHS compliance
* Case: PCB size is compatible to BPi-R4 and the case design can be re-used
* JTAG for main SOC: 10-pin 1.27 mm pitch (ARM JTAG/SWD)
* Antenna connectors: 3x MMCX for easy usage, assembly and durability
* Schematics: these will be publicly available (license TBD)
* GPL compliance: 3b. "Accompany it with a written offer ... to give any 
third party ... a complete machine-readable copy of the corresponding 
source code"
* Price: aiming for below 100$

How will the device be distributed?

OpenWrt itself cannot handle this for a ton of reasons. This is why we 
spoke with the SFC early. The idea is that BPi will distribute the 
device using the already established channels and for every device sold 
a donation will be made to ourSFC earmarked fund for OpenWrt. This money 
can then be used to cover hosting expenses or maybe an OpenWrt summit.

SFC is committed to working with us in various ways on this project — 
including making sure OpenWrt'strademark is properly respected, that 
this router isabeautiful example of excellent GPL/LGPL compliance, 
andthatthis becomes a great promotional opportunity for our project and 
FOSS generally!

FAQ

* Why are there are 2 different flash chips?
- the idea is to make the device (almost!) unbrickable and very easy to 
recover
- NAND will hold the main loader (U-Boot) and the Linux image and will 
be the default boot device
- NOR will be write-protected by default (with WP jumper available on 
the board) and will hold a recovery bootloader (and other essential 
data, like Wi-Fi calibration)
- a dedicated boot select switch will allow changing between NOR and NAND

* What will the M.2 slot be used for?
- we will use M.2 with M-key for NVMe storage. There is a 
work-in-progress patch to make PCIe work inside the U-Boot bootloader. 
This will allow booting other Linux distributions such as Debian and 
Alpine directly from NVMe

* Why is there no USB 3.x host port on the device?
- the USB 3.x and PCIe buses are shared in the selected SoC silicon, 
hence only a single High-Speed USB port is available

* What is the purpose of the console USB-C port?
- Holtek UART to USB bridge with CDC-ACM support on USB-C makes the 
device ultra easy to communicate with. No extra hardware or drivers will 
be required. Android for example has CDC-ACM support enabled by default

* What MAC OUI will the device have?
- we plan to register an OUI block for OpenWrt which can also be used 
for other vendor extensions such as Wi-Fi beacon IEs

* What is the purpose of the mikroBUS connector?
- mikroBUS was chosen as we wanted to make the hardware extendable. 
There are dedicated pins for UART, SPI, I2C buses and RST/INT signals. 
The standard uses regular 2.54 mm pitch connectors (you can use 
available mikroBUS modules or just connect to it something else, with 
2.54 mm jumper cables)

  * Why have the RTC on board instead of a mikroBUS module?
  - we believe there are many things a Wi-Fi (or networking in general) 
device should have on-board by default. Always having a correct time on 
the device is crucial in many applications, like VPN, DNSSEC, …

Timeline of events leading up to this e-mail

Forgive us for the lack of public communication during the initial 
phase(which as you can see was short and quick). We wanted to ensure 
that this project is feasible before disclosing it to the community. It 
would be a real shame if we announced something that we later found out 
to not be feasible thus failing expectations raised within the community.

03.12 - initial idea
06.12 - ping pepe2k, dangole, nbd
07.12 - ping MediaTek and ask if this sounds doable
08.12 - ping jow, Hauke
08.12 - request for call with SFC, we want them involved as soon as possible
09.12 - MediaTek replies and says they can help
09.12 - ping apacar, ynezz, dwmm2, lynxis, rsalvaterra
12.12 - MediaTek spoke with Banana Pi, they also like the idea
18.12 - call with SFC (Hauke joined, we found no prior slot to talk)
20.12 - started writing the U-Boot PCIe driver, made recovery from USB 
and android fastboot recovery work.
... and then the end of year celebrations started and not much happened 
for 2 weeks.
03.01-08.01 - write this text

Thanks,
Signed-off-by: Alexander Couzens <lynxis@fe80.eu>
Signed-off-by: Bradley M. Kuhn <bkuhn@sfconservancy.org>
Signed-off-by: Daniel Golle <daniel@makrotopia.org>
Signed-off-by: David Bauer <mail@david-bauer.net>
Signed-off-by: Denver Gingerich <denver@sfconservancy.org>
Signed-off-by: Felix Fietkau <nbd@nbd.name>
Signed-off-by: Hauke Mehrtens <hauke@hauke-m.de>
Signed-off-by: John Crispin <john@phrozen.org>
Signed-off-by: Jo-Philipp Wich <jo@mein.io>
Signed-off-by: Paul Spooren <mail@aparcar.org>
Signed-off-by: Petr Štetiar <ynezz@true.cz>
Signed-off-by: Piotr Dymacz <pepe2k@gmail.com>
Signed-off-by: Steven Liu <steven.liu@mediatek.com>

_______________________________________________
openwrt-devel mailing list
openwrt-devel@lists.openwrt.org
https://lists.openwrt.org/mailman/listinfo/openwrt-devel

```
