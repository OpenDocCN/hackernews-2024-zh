<!--yml
category: 未分类
date: 2024-05-27 13:04:37
-->

# TSAC: Very Low Bitrate Audio Compression

> 来源：[https://bellard.org/tsac/](https://bellard.org/tsac/)

# TSAC: Very Low Bitrate Audio Compression

TSAC is an audio compression utility reaching very low bitrates such as 5.5 kb/s for mono or 7.5 kb/s for stereo at 44.1 kHz with a good perceptual quality. Hence TSAC compresses a 3.5 minute stereo song to a file of 192 KiB.

An Nvidia GPU is necessary for fast operation. CPU only is also supported but slower.

## Compression Results

The audio extracts are from [here](https://listening-test.coresv.net/results.htm).

Waiting (Pops):

| original 

<Waiting.wav>

 | stereo 7.26 kb/s 

<Waiting_stereo.wav>

 | mono 5.61 kb/s 

<Waiting_mono.wav>

 | stereo 2.99 kb/s 

<Waiting_stereo_q5.wav>

 |

Greatest_Love_of_All_2min57 (Pops):

| original 

<Greatest_Love_of_All_2min57.wav>

 | stereo 6.79 kb/s 

<Greatest_Love_stereo.wav>

 | mono 5.02 kb/s 

<Greatest_Love_stereo.wav>

 | stereo 2.84 kb/s 

<Greatest_Love_stereo_q5.wav>

 |

9-Have-big-expensive-car.441 (Pops):

| original 

<9-Have-big-expensive-car.441.wav>

 | stereo 7.81 kb/s 

<Have-big-expensive-car_stereo.wav>

 | mono 5.91 kb/s 

<Have-big-expensive-car_mono.wav>

 | stereo 3.25 kb/s 

<Have-big-expensive-car_stereo_q5.wav>

 |

21-classic.441 (Classic):

| original 

<21-classic.441.wav>

 | stereo 6.21 kb/s 

<classic_stereo.wav>

 | mono 4.71 kb/s 

<classic_mono.wav>

 | stereo 2.57 kb/s 

<classic_stereo_q5.wav>

 |

4-Sound-English-male.441 (Voice):

| original 

<4-Sound-English-male.441.wav>

 | mono 6.79 kb/s 

<4-Sound-English-male.441_mono.wav>

 | mono 3.74 kb/s 

<4-Sound-English-male.441_mono_q5.wav>

 | mono 2.18 kb/s 

<4-Sound-English-male.441_mono_q3.wav>

 |

## Download

## Technical information

*   `tsac` is based on a modified version of the [Descript Audio Codec](https://github.com/descriptinc/descript-audio-codec) extended for stereo and a Transformer model to further increase the compression ratio. Both models are quantized to 8 bits per parameter.
*   The Transformer model is evaluated in a deterministic and reproducible way. Hence the result does not depend on the exact GPU or CPU model nor on the number of configured threads. This key point ensures that a compressed file can be decompressed using a different hardware or software configuration.

* * *

Fabrice Bellard - [https://bellard.org/](https://bellard.org)