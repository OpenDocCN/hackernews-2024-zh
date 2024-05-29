<!--yml
category: 未分类
date: 2024-05-27 14:34:10
-->

# The world's smallest PNG

> 来源：[https://evanhahn.com/worlds-smallest-png/](https://evanhahn.com/worlds-smallest-png/)

<main class="content-page">

# The world's smallest PNG

by [Evan Hahn](/)

, updated

Jan 21, 2024

(originally posted

Jan 4, 2024

)

The smallest PNG file is 67 bytes. It’s a single black pixel. Here’s what it looks like, zoomed in 200×:

![A single black pixel.](img/d5beb8f4f56c3c6b2f3420d6d28cd29f.png)

Wow, what a beauty.

This file has four sections:

1.  The PNG signature, the same for every PNG: 8 bytes
2.  The image’s metadata, which includes its dimensions: 25 bytes
3.  The image’s pixel data: 22 bytes
4.  An “end of image” marker: 12 bytes

The rest of this post describes this file in more detail and tries to explain how PNGs work along the way.

There’s a big twist at the end, if that excites you. But I hope you’re just excited to learn about PNGs.

## Part 1: the PNG signature

Every single PNG, including this one, starts with the same 8 bytes. Encoded in hex, those bytes are:

```
89 50 4E 47 0D 0A 1A 0A 
```

This is called the [**PNG signature**](https://www.w3.org/TR/2022/WD-png-3-20221025/#5PNG-file-signature). Try doing a hex dump on any PNG and you’ll see that it starts with these bytes.

PNG decoders use the signature to ensure that they’re reading a PNG image. Typically, they reject the file if it doesn’t start with the signature. Data can get corrupted in various ways (ever had a file with the wrong extension?) and this helps address that.

Fun fact: if you decode these bytes as ASCII, you’ll see the letters “PNG” in there:

```
.PNG.... 
```

So that’s the first 8 bytes. One part done! Here’s our “checklist”:

1.  ~~PNG signature~~
2.  Image metadata chunk
3.  Pixel data chunk
4.  “End of image” chunk

What about the rest?

The next part of the PNG is the image metadata, which is one of several **chunks**. What’s a chunk?

### Quick intro to chunks

Other than the PNG signature at the start, PNGs are made up of chunks.

Chunks have two logical pieces: a **type** and some **data bytes**. Types are things like “image header” or “text metadata”. The data depends on the type—the text metadata chunk is encoded differently from the image header chunk.

These logical pieces are encoded with [four fields](https://www.w3.org/TR/2022/WD-png-3-20221025/#5Chunk-layout). These fields are always in the same order for every chunk. They are:

1.  **Length**: the number of bytes in the chunk’s data field (field #3 below). Encoded as a 4-byte integer.
2.  **Chunk type**: the type of chunk this is. There are lots of different chunk types. Encoded as a 4-byte ASCII string, such as “IHDR” for “image header” or “tEXt” for “text metadata”.
3.  **Data**: the data for the chunk. See the “length” field for how many bytes there will be. Varies based on the chunk type. For example, the IHDR chunk encodes the image’s dimensions. May be empty, but usually isn’t.
4.  **Checksum**: a checksum for the rest of the chunk, to make sure no data was corrupted. 4 bytes.

As you can see, each chunk is a minimum of 12 bytes long (4 for the length, 4 for the type, and 4 for the checksum).

Note that the “length” field is the size of the “data” field, *not* the entire chunk. If you want to know the whole size of the chunk, just add 12—4 bytes for the length, 4 bytes for the type, and 4 bytes for the checksum.

You have some wiggle room but chunks have a specific order. For example, the image metadata chunk has to appear before the pixel data chunk. Once you reach the “image is done” chunk, the PNG is done.

Our tiny PNG will have just three of these chunks.

The first chunk of every PNG, including ours, is of type **IHDR**, short for [“image header”](https://www.w3.org/TR/2022/WD-png-3-20221025/#11IHDR).

Each chunk starts with the **length** of the data in that chunk.

The IHDR chunk always has 13 bytes of associated data, as we’ll see in a moment. 13 is `0D` in hex, which gets encoded like this:

```
00 00 00 0D 
```

The **chunk type** is next. This is another four bytes. “IHDR” is encoded as:

```
49 48 44 52 
```

This is just ASCII encoding. Chunk types are made up of ASCII letters. [The capitalization of each letter is significant.](https://www.w3.org/TR/2022/WD-png-3-20221025/#5Chunk-naming-conventions) For example, the first letter is capitalized which means it’s a required chunk.

Next, the chunk’s **data**. IHDR’s data happens to be 13 total bytes, arranged as follows:

*   The first eight bytes encode the image’s width and height. Because this is a 1×1 image, that’s encoded as `00 00 00 01 00 00 00 01`.

*   The next two bytes are the [bit depth](https://www.w3.org/TR/2022/WD-png-3-20221025/#3bitDepth) and [color type](https://www.w3.org/TR/2022/WD-png-3-20221025/#6Colour-values).

    These values are probably the most confusing part of this PNG.

    There are five possible color types. Our image is black-and-white so we use the “greyscale” color type (encoded as `00`). If our image had color, we might use the “truecolor” type (encoded with `02`). There are three other color types which we don’t need today, but you can [read more about them in the PNG specification](https://www.w3.org/TR/2022/WD-png-3-20221025/#4Concepts.PNGImage).

    Once you’ve picked a color type, you need to pick a bit depth. The bit depth depends on the color type, but usually means the number of bits per color channel in an image. For example, hex colors like `#FE9802` have a bit depth of eight—eight bits for red, eight bits for green, and eight bits for blue. Our all-black image doesn’t need all that&mldr;we only need *one* bit! The pixel is either completely black (`0`) or completely white (`1`)—in our case, it’s completely black.

    If we picked a more “expressive” color type and bit depth, we could make the same 1×1 image visually, but the file could be bigger because there could be more bits per pixel that we don’t actually need. For example, if we used the “truecolor” type and 16 bits per channel, each pixel would require 48 bits instead of just one—not necessary to encode “completely black”.

    With bit depth of 1 and a color type of 0, we encode these two values with `00 01`.

*   The next byte is the [compression method](https://www.w3.org/TR/2022/WD-png-3-20221025/#10CompressionCM0). All PNGs set this to `00` for now. This is here just in case they want to add another compression method later. As far as I know, nobody has.

*   Same story for the [filter method](https://www.w3.org/TR/2022/WD-png-3-20221025/#3filter). It’s always `00`.

*   The last part of the chunk’s data is the [interlace method](https://www.w3.org/TR/2022/WD-png-3-20221025/#8InterlaceMethods). PNGs support progressive decoding which allows images to be partly rendered as they download. We aren’t going to use that feature so we set this to `00`.

Finally, every chunk ends with a four-byte **checksum**. It uses [a common checksum function](https://www.w3.org/TR/2022/WD-png-3-20221025/#5CRC-algorithm) called CRC32, and uses the rest of the chunk as an input. Computing that checksum gives us the following bytes:

```
37 6E F9 24 
```

All together, here’s the whole IHDR chunk:

| Bytes | What? |
| --- | --- |
| `00 00 00 0D` | data length of 13 bytes |
| `49 48 44 52` | “IHDR” as ASCII |
| `00 00 00 01` | width |
| `00 00 00 01` | height |
| `01` | bit depth |
| `00` | color type |
| `00` | compression method |
| `00` | filter method |
| `00` | interlace method |
| `37 6E F9 24` | checksum |

So that’s our first chunk! Let’s take another look at our checklist:

1.  ~~PNG signature~~
2.  ~~Image metadata chunk~~
3.  Pixel data chunk
4.  “End of image” chunk

Two more chunks to go—pixel data is next.

## Part 3: pixel data chunk

Our next chunk is **IDAT**, short for [“image data”](https://www.w3.org/TR/2022/WD-png-3-20221025/#11IDAT). This is where the actual pixels are encoded&mldr;or just one pixel, in our case.

Remember that each chunk has four parts: the data’s *length*, the *chunk type*, the *data*, and a *checksum*.

This chunk will have 10 bytes of data. We’ll talk about *what* that data is shortly, but I promise it’s 10 bytes. Let’s encode that length:

```
00 00 00 0A 
```

Next, let’s encode “IDAT” for the chunk type:

```
49 44 41 54 
```

Again, this is just ASCII, and I’m showing the hex-encoded values.

Now for the interesting part: the image data.

### First step: uncompressed pixels

Image data is encoded in a series of [“scanlines”](https://www.w3.org/TR/2022/WD-png-3-20221025/#3scanline), and then compressed.

A scanline represents a horizontal line of pixels. For example, a 123×456 image has 456 scanlines. In our case, we have just one scanline, because our image is just one pixel tall.

Scanlines start with something called a [filter type](https://www.w3.org/TR/2022/WD-png-3-20221025/#9Filter-types) which can improve compression, depending on your image. Our image is so small that this is irrelevant, so we use filter type 0, or “None”.

After the filter type, each pixel is encoded with one or more bits, depending on the bit depth. In our case, we just need one bit per pixel—recall that we have a bit depth of 1; all black or all white.

If your pixel data doesn’t line up with a byte boundary—in other words, if it’s not a multiple of 8 bits—you pad the end of your scanline with zeroes. That’s true in our case, so we add seven padding bits to fill out a byte.

Putting that together (a zero byte to start the scanline, the single zero bit, and seven zero padding bits), our single scanline is:

```
00 00 
```

Now it’s time to “compress” the data.

### Second step: “compression”

Next, we compress the scanline data&mldr;well, not quite.

More accurately, we run it through a compression algorithm. Most of the time, compression algorithms produce smaller outputs—that’s the whole point! But sometimes, “compressing” tiny inputs actually produces *bigger* outputs because of some small overhead. Unfortunately for us, that’s what happens here. But the PNG file format makes us do it.

PNG image data is encoded in [the zlib format](https://www.rfc-editor.org/rfc/rfc1950) using the [DEFLATE compression algorithm](https://zlib.net/feldspar.html). DEFLATE is also used with [gzip](https://en.wikipedia.org/wiki/Gzip) and [ZIP](https://en.wikipedia.org/wiki/ZIP_(file_format)), two very popular compression formats.

I won’t go in depth on DEFLATE here (in part because I am not an expert^(), but here’s what our chunk’s data contains:)

1.  The zlib header: 2 bytes
2.  One compressed DEFLATE block that encodes two literal zeroes^(: 4 bytes)
3.  The zlib checksum (this is separate from the PNG chunk checksum!): 4 bytes

For more on how DEFLATE works, check out [“An Explanation of the DEFLATE Algorithm”](https://zlib.net/feldspar.html). I also recommend [infgen](https://github.com/madler/infgen/), a useful tool for inspecting DEFLATE streams.

All together, here are the ten data bytes:

```
78 01 63 60 00 00 00 02 00 01 
```

Again, unfortunate that we had to run our two-byte scanline through an algorithm that made it *five times bigger*, but PNG makes us do it!

With that, we can compute the PNG’s checksum field and finish off the chunk.

| Bytes | What? |
| --- | --- |
| `00 00 00 0A` | data length of 10 bytes |
| `49 44 41 54` | “IDAT” as ASCII |
| `78 01` | zlib header |
| `63 60 00 00` | “compressed” DEFLATE block |
| `00 02 00 01` | zlib checksum |
| `73 75 01 18` | chunk checksum |

Just one more chunk to go! Taking a final look at our checklist before everything is crossed off:

1.  ~~PNG signature~~
2.  ~~Image metadata chunk~~
3.  ~~Pixel data chunk~~
4.  “End of image” chunk

Let’s finish this up.

## Part 4: the end

Poetically, PNGs end like they begin: with a small number of constant bytes.

**IEND** is the final chunk, short for [“image trailer”](https://www.w3.org/TR/2022/WD-png-3-20221025/#11IEND).

The zero length is encoded with 4 zeroes:

```
00 00 00 00 
```

“IEND” is then encoded:

```
49 45 4E 44 
```

There’s no data in IEND chunks, so we just move onto the checksum. Because everything else in the chunk is constant, this checksum is always the same:

```
AE 42 60 82 
```

Here’s the whole trailer chunk:

| Bytes | What? |
| --- | --- |
| `00 00 00 00` | data length of 0 |
| `49 45 4E 44` | “IEND” as ASCII |
| `AE 42 60 82` | checksum |

And our PNG is done!

## Admiring our work

Here it is one more time, scaled up 200×:

![A single black pixel.](img/d5beb8f4f56c3c6b2f3420d6d28cd29f.png)

Beautiful. It starts with the classic PNG signature, follows up with a bit of metadata, “compresses” the pixel data, and signs off with an empty chunk.

And that’s the world’s smallest PNG!

&mldr;or is it?

## The twist: there are lots of champions

Throughout this post, I’ve said that this is the world’s smallest PNG. But that’s not quite true: it’s *tied* for first. There are several “world’s smallest PNGs”!

*As long as we encode all pixel data in a single byte, we can tie for the world’s smallest PNG.*

For example, you could encode this 8×1 black image, which is also 67 bytes:

![A black rectangle, 8 pixels wide and 1 pixel tall.](img/b95743e9e2abd57ea0ef739103e429ff.png)

This works because we use all eight bits are used to encode pixel data.

With our 1×1 image, recall that seven bits were effectively “wasted” on padding. Here’s basically what happened:

| Bits | What? |
| --- | --- |
| `0` | a black pixel |
| `0000000` | padding |

An 8×1 image can encode eight black pixels like so:

| Bits | What? |
| --- | --- |
| `00000000` | eight black pixels |

Instead of adding more pixels, you could also add more color resolution. Many grey colors can be encoded in a single byte, letting us tie for first. For example, this 1×1 grey pixel is also 67 bytes:

![A single grey pixel.](img/ce106dba64972531222763ae1a52ed9a.png)

Again, this “uses up” the whole byte we have available, unlike our 1×1 image.

For more on this, my former coworker Jordan Rose published [“The Biggest Smallest PNG”](https://belkadan.com/blog/2024/01/The-Biggest-Smallest-PNG/) in response to this post. It shows the biggest 67-byte PNG: a 1×2064 black line.

## Summary

PNGs start with a “signature”. The rest of the file is made up of chunks. Each chunk has a length, type, data, and checksum. Some chunks are always required, like the image header (IHDR) chunk.

The smallest PNGs use the minimum number of chunks and the smallest possible data.

Our PNGs are made up of four parts:

1.  The constant PNG signature (8 bytes)
2.  The IHDR chunk, containing metadata (25 bytes)
3.  The IDAT chunk, image pixel data (22 bytes)
4.  The IEND chunk, an image trailer (12 bytes)

If you’re interested in learning more about PNGs interactively, I built [PNG Chunk Explorer](https://evanhahn.gitlab.io/png-explorer/), which lets you analyze PNGs. Try uploading your own images to see what they’re made of! (It doesn’t work well on mobile, apologies.)

I also built [Single Color Image](https://singlecolorimage.com/), which generates monochromatic PNGs of arbitrary sizes. For example, you could generate a 12×34 purple rectangle. The images should be small but I haven’t yet implemented the most sophisticated compression, so you might need to run its results through a PNG compressor to achieve the smallest sizes.

Finally, I also wrote about the [*largest* possible PNG](/largest-possible-png/). There’s no theoretical file size limit, but there is a maximum number of pixels, and many decoders impose various limits.

I hope this long post has given you a good understanding of the PNG file format. If you read this far and have anything to say, [let me know](/contact/)!

</main>