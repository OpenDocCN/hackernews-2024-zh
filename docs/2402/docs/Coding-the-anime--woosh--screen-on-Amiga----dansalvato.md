<!--yml
category: 未分类
date: 2024-05-29 13:20:23
-->

# Coding the anime "woosh" screen on Amiga - @dansalvato

> 来源：[https://dansalva.to/coding-the-anime-woosh-screen-on-amiga/](https://dansalva.to/coding-the-anime-woosh-screen-on-amiga/)

 </images/20240223/1.mp4#t=0.01>

Your browser does not support the video tag.

The Amiga was a spectacle of graphics and sound when it debuted in 1985\. While it can trivially display colorful images like in the above example, doing so in the context of a game engine presents a lot of unique challenges.

 If you haven't seen the gameplay proof-of-concept video for Magicore Anomala, you can check it out [here](https://www.youtube.com/watch?v=4SB20aFHc08).

## Challenge 1: The RAM requirements

A run-of-the-mill Amiga 500 has 512kb of "Chip RAM" and 512kb of expansion RAM (sometimes called "Slow RAM"), which is the platform target for Magicore. Only Chip RAM can be used by the Amiga chipset to present graphics and sound, which makes it much more valuable—all other kinds of expansion RAM can only be accessed by the CPU.

The fullscreen character graphic (CG) is a 320x240 bitmap with 32 colors, which takes up 48kb of RAM uncompressed. That is a *lot*. Between all the common assets, level data, and screen memory allocations, we don't want to afford that kind of overhead on the RAM.

Thankfully, I recently added support for asset compression, using the ZX0 compression format. Compressed, the CG is about 8kb, which is acceptable.

When the level assets are loaded, the compressed CG is loaded into the expansion RAM. Then, right before it gets displayed, I unpack it into Chip RAM.

The trick is that instead of finding 48kb of free Chip RAM to use, I reuse other parts of screen memory:

*   The room background image (the grassy field, in this example)
*   The screen layer used to render hazardous objects (not shown in the video)
*   The textbox screen area (as seen in the gameplay proof-of-concept video)

All three of these memory regions are contiguous in RAM, and they come out to 48,000 bytes, which is the *exact* size of the CG!

It's also okay to overwrite the room background image, because we can restore it after the CG is done being displayed.

In the video below, I force the CG to always be shown on the screen, so we can watch how the data gets decompressed and overwritten.

 </images/20240223/2.mp4#t=0.01>

Your browser does not support the video tag.

Pretty cool, right? As you can see, the CG takes maybe 500ms to fully decompress. But that "loading time" is hidden into the flow of the cutscene.

## Challenge 2: The "screen split" effect

I was initially thinking about doing a vertical wipe for the screen transition. But for that to look nice, the wipe would have to be a gradient, adjusting the color palette every scanline. That's pretty possible, but the coprocessor (copper) alone struggles to set all 32 colors in a single horizontal blank, and—I'll be honest—I didn't want to deal with "racing the beam".

The screen split effect is easier to pull off, and I think it looks cooler to the common viewer. In fact, the copper was practically purpose-designed for this effect! Check out [this](https://www.youtube.com/watch?v=YlAhRJjOhDg) video, which demonstrates a similar effect built right into Amiga Workbench.

This demonstrates two special features of Amiga working in tandem:

1.  The copper runs in parallel to the CPU with its own instruction list. Those instructions can tell it to change certain hardware registers at specific lines on the screen.
2.  The screen memory can be changed to *anywhere* in Chip RAM by setting screen pointers in the hardware registers. That means you can have multiple bitmap screens and switch between them whenever you want—every frame (e.g. for double buffering), or even multiple times in one frame.

Let's say my "main" screen memory begins at `0x20000`. Normally, I instruct the copper to arm the bitplane DMA registers with this address. Once I enable the bitplanes, the DMA happily marches through this region of memory, drawing its data to the screen while incrementing the address pointer as it goes.

There is an interesting trick. Let's say each horizontal line takes up `0x100` bytes of screen memory. What if I set the screen pointer to `0x20800` instead?

The screen will appear to "scroll up" by 8 lines, because the screen officially starts 8 lines down into memory.

I have the top half of the split scroll up in this fashion. Then, at the split point, the copper is instructed to shut off bitplane DMA (and change the BG color to red).

Now, all the bitplane-related hardware registers are effectively frozen in time. Once we reach the bottom of the split, the copper resets the BG color and resumes bitplane DMA. The display picks up right where it left off, just lower down on the screen!

If you're wondering what this looks like, here is the copperlist used for the effect:

```
vs_TCop:
 ; Wait for top split to end
vs_TCopTop: dc.w  $9007,$fffe
 ; Set BG color
vs_TCopColorTop: dc.w COLOR00,0
 ; Disable sprite and bitplane DMA
 dc.w DMACON,$0120
 ; Wait for bottom split to start
vs_TCopBottom: dc.w  $90e1,$fffe
 ; Restore BG color
vs_TCopColorOld: dc.w COLOR00,0
 ; Enable sprite and bitplane DMA
 dc.w DMACON,$8120
```

That's it! Every frame, I use the CPU to adjust `vs_TCopTop` and `vs_TCopBottom` based on the current width of the split. (Not shown: Adjusting the screen pointer for the top split, as described above.)

## Challenge 3: The "motion lines"

You can't reach full anime without having the lines that go "woosh" in the background.

I use sprites to draw the lines, which is a good choice because they can be drawn and moved fully independently from screen memory. The issue is that Amiga sprites are both very limited and very complicated.

### Sprite colors

Sprites share a color palette with bitplanes, meaning I want to use up as few colors as possible. The sprite is only 3 colors, leaving 28 for the CG (and 1 for the background).

The problem is that different sprites use different colors in the palette. The first two sprites use colors 16-19, the second two sprites use colors 20-23, and so on.

This changes if you combine sprites. By "attaching" two sprites together, they become one sprite with a 16-color palette (colors 16-31). That means for the motion lines, I can use 4 attached sprites, and use only colors 29-31 in the graphic. It's a silly workaround for a silly limitation.

### Reusing a sprite graphic

The first 4 bytes of a sprite graphic are actually "control bits" that tell the Amiga the position and height of the sprite. That is actually a pain—what if we want to draw the same graphic in multiple locations?

My first thought was to manually set the hardware registers for sprite control bits, but I simply could not get the sprite to display on screen when doing this. Amiga sprite DMA works similarly to bitplane DMA; it has a pointer to the sprite data that it walks through in order to display it to the screen. But when manually setting the control bits, I just couldn't get it to do that. I'm sure it can be done, but I decided to find another way.

I instead created 8 fake sprites that are only 4 bytes large—*just* the control bits. I set all the sprite pointers to those fake sprites.

Around line 19, the sprite DMA looks at all the sprite pointers and arms itself with the control bits, preparing to draw the data to the screen at the specified position.

Once it does so, I pull a switcheroo: I change all the sprite pointers to the "motion line" graphic. Now, the DMA is armed to draw all the sprites at different positions, but using the same graphic.

Again, this can be trivially done in the copperlist:

```
 ; Set dummy sprite pointers (to arm control bits)
vs_CopSprP:  COPPTR     SPR0PT
 COPPTR     SPR1PT
 COPPTR     SPR2PT
 COPPTR     SPR3PT
 COPPTR     SPR4PT
 COPPTR     SPR5PT
 COPPTR     SPR6PT
 COPPTR     SPR7PT
 ; Wait for line 19
 dc.w  $14df,$fffe
 ; Now that control bits are armed, set data pointers
vs_CopSprP2: COPPTR     SPR0PT
 COPPTR     SPR1PT
 COPPTR     SPR2PT
 COPPTR     SPR3PT
 COPPTR     SPR4PT
 COPPTR     SPR5PT
 COPPTR     SPR6PT
 COPPTR     SPR7PT
```

### Sprites don't get drawn when bitplanes are off

Before the CG reaches the top of the screen, there is a bunch of empty space between the top of the screen and the start of the CG. If bitplanes are enabled during this time, they will draw junk data to the screen. Here is an example of that:

 </images/20240223/3.mp4#t=0.01>

Your browser does not support the video tag.

Dang, that's actually kind of cool-looking. Missed opportunity?

In that screen region, we want to disable bitplanes so that the DMA doesn't run away with junk data like that.

One problem: If you disable bitplanes, then sprites also don't get drawn! I don't know why it works like this, but it does.

 </images/20240223/4.mp4#t=0.01>

Your browser does not support the video tag.

See how the lines only get drawn within the bounds of the CG?

My solution was to keep just 1 bitplane enabled, and set the screen pointer to empty data. That way, it *is* drawing to the screen, but it's just drawing nothing.

But where do I find a screen full of empty data? Thankfully, I don't have to. There are "bitplane modulo" registers (`BPLMOD1` and `BPLMOD2`) that let you increment the screen pointer by a certain amount after each line. This is useful for interleaved bitplanes, which I won't get into here.

At 1 bit per pixel, a 320-pixel line is 320 bits, or 40 bytes. If I set `BPLMOD1` to -40, then it will go backwards 40 bytes after each line, causing it to draw the same 40 bytes over and over, on each new line.

That means I only need to find 40 bytes of empty data, which is easy to find; my screen has "safety margins" which hold nothing but junk data from objects that are drawn beyond the screen borders. I can just clear out the first 40 bytes of the safety margin, and I'm good to go.

## Conclusion

I originally wasn't sure if I would include CGs like this in the game, because I was worried about the RAM requirements. But now that I have data compression implemented, I proved that the overhead is extremely reasonable, and I can add this extra bit of flair to Magicore.

There were a lot of other small challenges I didn't go over here, like getting the bottom of the 100px motion line to not abruptly disappear after the sprite leaves the top of the screen. But the ones I covered were the most interesting to me, especially how they involve unique quirks of Amiga hardware.

Also, if you like Amiga, you might have noticed that this effect doesn't use the blitter at all! If you want to read a blitter-related post, try [this](/getting-clever-with-the-amiga-blitter) one.

Amiga is so great at displaying colorful graphics that I hope I can impress people with its capabilities today, just as they were in the latter half of the 80s.