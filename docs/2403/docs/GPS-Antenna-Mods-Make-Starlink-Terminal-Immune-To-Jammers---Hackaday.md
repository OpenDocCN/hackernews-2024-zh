<!--yml
category: 未分类
date: 2024-05-27 14:40:07
-->

# GPS Antenna Mods Make Starlink Terminal Immune To Jammers | Hackaday

> 来源：[https://hackaday.com/2024/03/06/gps-antenna-mods-make-starlink-terminal-immune-to-jammers/](https://hackaday.com/2024/03/06/gps-antenna-mods-make-starlink-terminal-immune-to-jammers/)

The Starlink receivers need positioning and precise timing information to function, and currently the best way to get that information is to use a global navigation satellite system (GNSS) such as GPS. Unfortunately, the antenna used for this secondary satellite connection leaves something to be desired. Of course, when it comes to solving Starlink problems, there’s no one best than [Oleg Kutkov], whose duty is to fix and improve upon Starlink terminals used in Ukraine — and when the specific problem is GPS bands getting jammed by the invading military, [you better believe that a fix is due](https://olegkutkov.me/2023/11/07/connecting-external-gps-antenna-to-the-starlink-terminal/).

[Oleg] sets the scene, walking us through the evolution of GPS circuitry on the Starlink terminals. Then he shows us the simplest mods you can do, like soldering an improved passive antenna in place of the chip antenna currently being used. Then, he takes it up a notch, and shows us how you could attach an active antenna by using a bias tee module, a mod that would surely work wonders on more than just this device! Then, he brings out the test result tables — and the differences are impressive, in that the Starlink terminals with active antenna mods were able to get GPS signal in areas with active jamming going on, while the unmodified ones could not.

The post is exceptionally accessible, and a must read for anyone wondering about GPS antenna reception problems in customer-accessible devices. This is not the only Starlink hardware mod we’ve seen [Oleg] make, we’ve just covered his Starlink Ethernet port restoration journey that meticulously [fixes Ethernet connectivity oversights](https://hackaday.com/2024/02/28/restoring-starlinks-missing-ethernet-ports/) in the newer models, and the blog also has an article about [powering Starlink terminals without the need for PoE,](https://olegkutkov.me/2023/06/09/how-to-power-starlink-terminal-from-12v-without-poe-injector-and-dc-dc-converters/) so, do check it out if you’re looking for more!