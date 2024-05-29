<!--yml
category: 未分类
date: 2024-05-27 15:00:55
-->

# A New Manic Miner Port for an Arduino Uno Game Engine - Hackster.io

> 来源：[https://www.hackster.io/news/a-new-manic-miner-port-for-an-arduino-uno-game-engine-d9ab11e33dfa](https://www.hackster.io/news/a-new-manic-miner-port-for-an-arduino-uno-game-engine-d9ab11e33dfa)

Microcontrollers are not computers — except they *kind of* are. They have processors, memory, and some form of storage. Though there are some limitations and a lot of architectural differences, microcontrollers can act as computers. But because they have a lot less power under the hood than modern computers, they can usually only mimic older computers and video game consoles. That was, however, enough for Scott Porter (AKA Smashcat) to [create their own game engine for the Arduino Uno that can run](https://github.com/Smashcat/UNO_Manic_Miner) *[Manic Miner](https://github.com/Smashcat/UNO_Manic_Miner)* — a video game originally released for the ZX Spectrum.

*Manic Miner* was a big hit and is easily one of the best games to come out of the 1980s for *any* system. Though Matthew Smith wrote the game for the ZX Spectrum, it was soon ported to many other 8-bit computers and then to just about every other system that came later. It is a platfomer with rudimentary graphics, but with many creative and challenging levels. Porter built a video game engine for the Arduino Uno Rev3 and decided that *Manic Miner* would be the perfect title to showcase its capability.

This game engine has capabilities similar to early 8-bit computers, with some restrictions imposed by the low RAM and lack of a dedicated video chip. The latter factor meant that Porter had to make the Arduino generate a composite video signal. That requires very accurate timing, so they recommend using an official Uno Rev3 with a real crystal instead of a generic board that may have a resonator instead.

It is capable of producing monochrome graphics with a resolution of 256×256 at a fixed framerate of 50fps. Up to nine sprites can appear onscreen at a time, each with a 16×16 resolution. One of those sprites can have pixel-perfect collision detection against all of the others. The game engine can display scrolling tile map backgrounds and Porter reports that it has plenty of cycles to spare for the game logic.

A custom PCB shield helps with connections, providing the video output, a speaker driver, and control buttons. There is also an NES controller port for people that would prefer to use that over the built-in buttons.

Porter's ported version of *Manic Miner* looks great. It has all 20 original levels, plus all of the special bosses and mechanics. The high score is even persistent between boots and there is two-channel audio. We did observe a few glitches in the demonstration video, but the game looks entirely playable.