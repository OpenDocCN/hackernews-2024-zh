<!--yml
category: 未分类
date: 2024-05-29 12:50:30
-->

# Sunshine

> 来源：[https://toaster.llc/blog/sunshine/index.html](https://toaster.llc/blog/sunshine/index.html)

 <center-column>Zero to Photon:

Sunshine

2024 . 3 . 21

In the process of making a camera, inevitably you're going to be taking a lot of pictures. While capturing imagery with [Photon](/photon), there were two sunlight-related phenomena that I found surprising and interesting.

* * *

## Transparency

The first phenomenon occurred in Kentucky while I was visiting my father.

I was streaming images from a prototype while working in the living room of my childhood home. I picked up the prototype and the back side of the PCB caught a ray of sunlight shining through the windows. Immediately the imagery transformed — it showed not only the living room, but the living room with wiring traces superimposed:

I'm pretty sure I didn't accidentally implement a wire trace renderer in the imaging pipeline...

At first I assumed these were the metal traces of Photon's PCB:

**Left**

: PCB layout; the outer rectangle is the bounds of the image sensor IC. The inner rectangle is the pixel array region of the image sensor.

**Right**: the actual PCB under a microscope.

But a closer inspection shows that the traces on the PCB don't match the mystery traces:

Not a match!

**Left**: PCB layout; zoomed and cropped to the bounds of the image sensor pixel array.

**Right**: mystery traces; captured while covering the lens so that the only light entering the image sensor is from its back side.

If the mystery traces aren't PCB traces, what are they?

When I returned to San Jose, I was able to look at the back side of a spare image sensor under a microscope:

Back side of a spare image sensor

Enhance:

Bingo!

**Left**: back side of image sensor; desaturated and cropped to the bounds of the image sensor's pixel array.

**Right**: same mystery traces.

It's a match! The mystery traces are the BGA circuitry on the back side of the image sensor.

The phenomenon appears to be particularly sensitive to light in the UV spectrum. For example, the mere ambient UV from having an open window in my living room triggers it for a device without a backplate:

 </uv-traces-living-room.06918f22.mp4> 

The traces appear when opening a window, even with the device out of direct sunlight.

Note the distinct purple hue of the traces, suggesting the UV spectrum.

Compared to the visible spectrum of light, I'm guessing UV can more easily penetrate the FR-4 of the PCB due to its higher energy.

Fortunately Photon has an aluminum enclosure which prevents this phenomenon from occurring in the field; if Photon had a plastic enclosure, light shielding would be necessary.

* * *

## Cosmic Glitch

The second phenomenon happened in San Jose while streaming images from a Photon prototype. As soon as I walked out the door to test the imaging outside, the streaming images halted.

Probably just a bug in my USB stack, I figured. I went back inside and got images streaming again, but the instant I stepped outside it happened again. As crazy as it seemed at the time, the consistency was undeniable: something about being outdoors was glitching the prototype.

A few more experiments confirmed it: the problem didn't occur when the prototype was outdoors but in the shade, and wrapping the board in electrical tape also prevented it. (This was before I had metal enclosures to test with.)

To narrow the problem down to specific components, I made an aluminum-foil mask and taped it to the side of my kitchen table, and configured my computer to beep when the glitch occurred. Now I could move the board around to direct the tiny sun beam:

 </glitches.0675a26c.mov> 

The sunbeam easily identified the two chips that are disrupted by sunlight: a 16 MHz clock, and a 3.3V LDO.

Enable audio to hear when the glitches occur.

This strategy made it very clear which components are affected, and they all have one thing in common: [chip-scale packaging!](https://en.wikipedia.org/wiki/Chip-scale_package) These types of chips aren't encased in black plastic, which allows light to enter the silicon die and disrupt normal operation.

Photon is apparently an excellent sunlight detector!</center-column>