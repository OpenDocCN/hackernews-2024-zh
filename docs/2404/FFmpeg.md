<!--yml
category: 未分类
date: 2024-05-27 12:57:30
-->

# FFmpeg

> 来源：[https://ffmpeg.org//index.html#pr7.0](https://ffmpeg.org//index.html#pr7.0)

## A complete, cross-platform solution to record, convert and stream audio and video.

### Converting **video** and **audio** has never been so easy.

```
$ ffmpeg -i input.mp4 output.avi
```

# [](main.rss)**  [](https://twitter.com/FFmpeg)**  [](https://www.facebook.com/ffmpeg)*********News*****

 *****### May 13th, 2024, Sovereign Tech Fund

The FFmpeg community is excited to announce that Germany's [Sovereign Tech Fund](https://www.sovereigntechfund.de/tech/ffmpeg) has become its first governmental sponsor. Their support will help sustain the maintainance of the FFmpeg project, a critical open-source software multimedia component essential to bringing audio and video to billions around the world everyday.

### April 5th, 2024, FFmpeg 7.0 "Dijkstra"

A new major release, [FFmpeg 7.0 "Dijkstra"](download.html#release_7.0), is now available for download. The most noteworthy changes for most users are a [native VVC decoder](#vvcdec) (currently experimental, until more fuzzing is done), [IAMF support](#iamf), or a [multi-threaded `ffmpeg` CLI tool](#cli_threading).

This release is *not* backwards compatible, removing APIs deprecated before 6.0. The biggest change for most library callers will be the removal of the old bitmask-based channel layout API, replaced by the `AVChannelLayout` API allowing such features as custom channel ordering, or Ambisonics. Certain deprecated `ffmpeg` CLI options were also removed, and a C11-compliant compiler is now required to build the code.

As usual, there is also a number of new supported formats and codecs, new filters, APIs, and countless smaller features and bugfixes. Compared to 6.1, the `git` repository contains almost ∼2000 new commits by ∼100 authors, touching >100000 lines in ∼2000 files — thanks to everyone who contributed. See the [Changelog](https://git.videolan.org/?p=ffmpeg.git;a=blob_plain;f=Changelog;hb=n7.0), [APIchanges](https://git.videolan.org/?p=ffmpeg.git;a=blob_plain;f=doc/APIchanges;hb=n7.0), and the git log for more comprehensive lists of changes.

### January 3rd, 2024, native VVC decoder

The `libavcodec` library now contains a native VVC (Versatile Video Coding) decoder, supporting a large subset of the codec's features. Further optimizations and support for more features are coming soon. The code was written by Nuo Mi, Xu Mu, Frank Plowman, Shaun Loo, and Wu Jianhua.

### December 18th, 2023, IAMF support

The `libavformat` library can now read and write [IAMF](https://aomediacodec.github.io/iamf/) (Immersive Audio) files. The `ffmpeg` CLI tool can configure IAMF structure with the new `-stream_group` option. IAMF support was written by James Almer.

### December 12th, 2023, multi-threaded `ffmpeg` CLI tool

Thanks to a major refactoring of the `ffmpeg` command-line tool, all the major components of the transcoding pipeline (demuxers, decoders, filters, encodes, muxers) now run in parallel. This should improve throughput and CPU utilization, decrease latency, and open the way to other exciting new features.

Note that you should *not* expect significant performance improvements in cases where almost all computational time is spent in a single component (typically video encoding).

### November 10th, 2023, FFmpeg 6.1 "Heaviside"

[FFmpeg 6.1 "Heaviside"](download.html#release_6.1), a new major release, is now available! Some of the highlights:

*   libaribcaption decoder
*   Playdate video decoder and demuxer
*   Extend VAAPI support for libva-win32 on Windows
*   afireqsrc audio source filter
*   arls filter
*   ffmpeg CLI new option: -readrate_initial_burst
*   zoneplate video source filter
*   command support in the setpts and asetpts filters
*   Vulkan decode hwaccel, supporting H264, HEVC and AV1
*   color_vulkan filter
*   bwdif_vulkan filter
*   nlmeans_vulkan filter
*   RivaTuner video decoder
*   xfade_vulkan filter
*   vMix video decoder
*   Essential Video Coding parser, muxer and demuxer
*   Essential Video Coding frame merge bsf
*   bwdif_cuda filter
*   Microsoft RLE video encoder
*   Raw AC-4 muxer and demuxer
*   Raw VVC bitstream parser, muxer and demuxer
*   Bitstream filter for editing metadata in VVC streams
*   Bitstream filter for converting VVC from MP4 to Annex B
*   scale_vt filter for videotoolbox
*   transpose_vt filter for videotoolbox
*   support for the P_SKIP hinting to speed up libx264 encoding
*   Support HEVC,VP9,AV1 codec in enhanced flv format
*   apsnr and asisdr audio filters
*   OSQ demuxer and decoder
*   Support HEVC,VP9,AV1 codec fourcclist in enhanced rtmp protocol
*   CRI USM demuxer
*   ffmpeg CLI '-top' option deprecated in favor of the setfield filter
*   VAAPI AV1 encoder
*   ffprobe XML output schema changed to account for multiple variable-fields elements within the same parent element
*   ffprobe -output_format option added as an alias of -of

This release had been overdue for at least half a year, but due to constant activity in the repository, had to be delayed, and we were finally able to branch off the release recently, before some of the large changes scheduled for 7.0 were merged.

Internally, we have had a number of changes too. The FFT, MDCT, DCT and DST implementation used for codecs and filters has been fully replaced with the faster libavutil/tx (full article about it coming soon).
This also led to a reduction in the the size of the compiled binary, which can be noticeable in small builds.
There was a very large reduction in the total amount of allocations being done on each frame throughout video decoders, reducing overhead.
RISC-V optimizations for many parts of our DSP code have been merged, with mainly the large decoders being left.
There was an effort to improve the correctness of timestamps and frame durations of each packet, increasing the accurracy of variable frame rate video.

Next major release will be version 7.0, scheduled to be released in February. We will attempt to better stick to the new release schedule we announced at the start of this year.

We strongly recommend users, distributors, and system integrators to upgrade unless they use current git master.

### May 31st, 2023, Vulkan decoding

A few days ago, Vulkan-powered decoding hardware acceleration code was merged into the codebase. This is the first vendor-generic and platform-generic decode acceleration API, enabling the same code to be used on multiple platforms, with very minimal overhead. This is also the first multi-threaded hardware decoding API, and our code makes full use of this, saturating all available decode engines the hardware exposes.

Those wishing to test the code can read our [documentation page](https://trac.ffmpeg.org/wiki/HWAccelIntro#Vulkan). For those who would like to integrate FFmpeg's Vulkan code to demux, parse, decode, and receive a VkImage to present or manipulate, documentation and examples are available in our source tree. Currently, using the latest available git checkout of our [repository](https://git.videolan.org/?p=ffmpeg.git;a=summary) is required. The functionality will be included in stable branches with the release of version 6.1, due to be released soon.

As this is also the first practical implementation of the specifications, bugs may be present, particularly in drivers, and, although passing verification, the implementation itself. New codecs, and encoding support are also being worked on, by both the Khronos organization for standardizing, and us as implementing it, and giving feedback on improving.

### February 28th, 2023, FFmpeg 6.0 "Von Neumann"

A new major release, [FFmpeg 6.0 "Von Neumann"](download.html#release_6.0), is now available for download. This release has many new encoders and decoders, filters, ffmpeg CLI tool improvements, and also, changes the way releases are done. All major releases will now bump the version of the ABI. We plan to have a new major release each year. Another release-specific change is that deprecated APIs will be removed after 3 releases, upon the next major bump. This means that releases will be done more often and will be more organized.

New decoders featured are Bonk, RKA, Radiance, SC-4, APAC, VQC, WavArc and a few ADPCM formats. QSV and NVenc now support AV1 encoding. The FFmpeg CLI (we usually reffer to it as ffmpeg.c to avoid confusion) has speed-up improvements due to threading, as well as statistics options, and the ability to pass option values for filters from a file. There are quite a few new audio and video filters, such as adrc, showcwt, backgroundkey and ssim360, with a few hardware ones too. Finally, the release features many behind-the-scenes changes, including a new FFT and MDCT implementation used in codecs (expect a blog post about this soon), numerous bugfixes, better ICC profile handling and colorspace signalling improvement, introduction of a number of RISC-V vector and scalar assembly optimized routines, and a few new improved APIs, which can be viewed in the doc/APIchanges file in our tree. A few submitted features, such as the Vulkan improvements and more FFT optimizations will be in the next minor release, 6.1, which we plan to release soon, in line with our new release schedule. Some highlights are:

*   Radiance HDR image support
*   ddagrab (Desktop Duplication) video capture filter
*   ffmpeg -shortest_buf_duration option
*   ffmpeg now requires threading to be built
*   ffmpeg now runs every muxer in a separate thread
*   Add new mode to cropdetect filter to detect crop-area based on motion vectors and edges
*   VAAPI decoding and encoding for 10/12bit 422, 10/12bit 444 HEVC and VP9
*   WBMP (Wireless Application Protocol Bitmap) image format
*   a3dscope filter
*   bonk decoder and demuxer
*   Micronas SC-4 audio decoder
*   LAF demuxer
*   APAC decoder and demuxer
*   Media 100i decoders
*   DTS to PTS reorder bsf
*   ViewQuest VQC decoder
*   backgroundkey filter
*   nvenc AV1 encoding support
*   MediaCodec decoder via NDKMediaCodec
*   MediaCodec encoder
*   oneVPL support for QSV
*   QSV AV1 encoder
*   QSV decoding and encoding for 10/12bit 422, 10/12bit 444 HEVC and VP9
*   showcwt multimedia filter
*   corr video filter
*   adrc audio filter
*   afdelaysrc audio filter
*   WADY DPCM decoder and demuxer
*   CBD2 DPCM decoder
*   ssim360 video filter
*   ffmpeg CLI new options: -stats_enc_pre[_fmt], -stats_enc_post[_fmt], -stats_mux_pre[_fmt]
*   hstack_vaapi, vstack_vaapi and xstack_vaapi filters
*   XMD ADPCM decoder and demuxer
*   media100 to mjpegb bsf
*   ffmpeg CLI new option: -fix_sub_duration_heartbeat
*   WavArc decoder and demuxer
*   CrystalHD decoders deprecated
*   SDNS demuxer
*   RKA decoder and demuxer
*   filtergraph syntax in ffmpeg CLI now supports passing file contents as option values
*   hstack_qsv, vstack_qsv and xstack_qsv filters

We strongly recommend users, distributors, and system integrators to upgrade unless they use current git master.

### July 22nd, 2022, FFmpeg 5.1 "Riemann"

[FFmpeg 5.1 "Riemann"](download.html#release_5.1), a new major release, is now available! Some of the highlights:

*   add ipfs/ipns protocol support
*   dialogue enhance audio filter
*   dropped obsolete XvMC hwaccel
*   pcm-bluray encoder
*   DFPWM audio encoder/decoder and raw muxer/demuxer
*   SITI filter
*   Vizrt Binary Image encoder/decoder
*   avsynctest source filter
*   feedback video filter
*   pixelize video filter
*   colormap video filter
*   colorchart video source filter
*   multiply video filter
*   PGS subtitle frame merge bitstream filter
*   blurdetect filter
*   tiltshelf audio filter
*   QOI image format support
*   ffprobe -o option
*   virtualbass audio filter
*   VDPAU AV1 hwaccel
*   PHM image format support
*   remap_opencl filter
*   added chromakey_cuda filter

We strongly recommend users, distributors, and system integrators to upgrade unless they use current git master.

### January 17th, 2022, FFmpeg 5.0 "Lorentz"

[FFmpeg 5.0 "Lorentz"](download.html#release_5.0), a new major release, is now available! For this long-overdue release, a major effort underwent to remove the old encode/decode APIs and replace them with an N:M-based API, the entire libavresample library was removed, libswscale has a new, easier to use AVframe-based API, the Vulkan code was much improved, many new filters were added, including libplacebo integration, and finally, DoVi support was added, including tonemapping and remuxing. The default AAC encoder settings were also changed to improve quality. Some of the changelog highlights:

*   ADPCM IMA Westwood encoder
*   Westwood AUD muxer
*   ADPCM IMA Acorn Replay decoder
*   Argonaut Games CVG demuxer
*   Argonaut Games CVG muxer
*   Concatf protocol
*   afwtdn audio filter
*   audio and video segment filters
*   Apple Graphics (SMC) encoder
*   hsvkey and hsvhold video filters
*   adecorrelate audio filter
*   atilt audio filter
*   grayworld video filter
*   AV1 Low overhead bitstream format muxer
*   swscale slice threading
*   MSN Siren decoder
*   scharr video filter
*   apsyclip audio filter
*   morpho video filter
*   amr parser
*   (a)latency filters
*   GEM Raster image decoder
*   asdr audio filter
*   speex decoder
*   limitdiff video filter
*   xcorrelate video filter
*   varblur video filter
*   huesaturation video filter
*   colorspectrum source video filter
*   RTP packetizer for uncompressed video (RFC 4175)
*   bitpacked encoder
*   VideoToolbox VP9 hwaccel
*   VideoToolbox ProRes hwaccel
*   support loongarch.
*   aspectralstats audio filter
*   adynamicsmooth audio filter
*   libplacebo filter
*   vflip_vulkan, hflip_vulkan and flip_vulkan filters
*   adynamicequalizer audio filter
*   yadif_videotoolbox filter
*   VideoToolbox ProRes encoder
*   anlmf audio filter

We strongly recommend users, distributors, and system integrators to upgrade unless they use current git master.

### June 19th, 2021, IRC

We have a new IRC home at Libera Chat now! Feel free to join us at #ffmpeg and #ffmpeg-devel. More info at [contact#IRCChannels](https://ffmpeg.org/contact.html#IRCChannels)

### April 8th, 2021, FFmpeg 4.4 "Rao"

[FFmpeg 4.4 "Rao"](download.html#release_4.4), a new major release, is now available! Some of the highlights:

*   AudioToolbox output device
*   MacCaption demuxer
*   PGX decoder
*   chromanr video filter
*   VDPAU accelerated HEVC 10/12bit decoding
*   ADPCM IMA Ubisoft APM encoder
*   Rayman 2 APM muxer
*   AV1 encoding support SVT-AV1
*   Cineform HD encoder
*   ADPCM Argonaut Games encoder
*   Argonaut Games ASF muxer
*   AV1 Low overhead bitstream format demuxer
*   RPZA video encoder
*   ADPCM IMA MOFLEX decoder
*   MobiClip FastAudio decoder
*   MobiClip video decoder
*   MOFLEX demuxer
*   MODS demuxer
*   PhotoCD decoder
*   MCA demuxer
*   AV1 decoder (Hardware acceleration used only)
*   SVS demuxer
*   Argonaut Games BRP demuxer
*   DAT demuxer
*   aax demuxer
*   IPU decoder, parser and demuxer
*   Intel QSV-accelerated AV1 decoding
*   Argonaut Games Video decoder
*   libwavpack encoder removed
*   ACE demuxer
*   AVS3 demuxer
*   AVS3 video decoder via libuavs3d
*   Cintel RAW decoder
*   VDPAU accelerated VP9 10/12bit decoding
*   afreqshift and aphaseshift filters
*   High Voltage Software ADPCM encoder
*   LEGO Racers ALP (.tun & .pcm) muxer
*   AV1 VAAPI decoder
*   adenorm filter
*   ADPCM IMA AMV encoder
*   AMV muxer
*   NVDEC AV1 hwaccel
*   DXVA2/D3D11VA hardware accelerated AV1 decoding
*   speechnorm filter
*   SpeedHQ encoder
*   asupercut filter
*   asubcut filter
*   Microsoft Paint (MSP) version 2 decoder
*   Microsoft Paint (MSP) demuxer
*   AV1 monochrome encoding support via libaom >= 2.0.1
*   asuperpass and asuperstop filter
*   shufflepixels filter
*   tmidequalizer filter
*   estdif filter
*   epx filter
*   Dolby E parser
*   shear filter
*   kirsch filter
*   colortemperature filter
*   colorcontrast filter
*   PFM encoder
*   colorcorrect filter
*   binka demuxer
*   XBM parser
*   xbm_pipe demuxer
*   colorize filter
*   CRI parser
*   aexciter audio filter
*   exposure video filter
*   monochrome video filter
*   setts bitstream filter
*   vif video filter
*   OpenEXR image encoder
*   Simbiosis IMX decoder
*   Simbiosis IMX demuxer
*   Digital Pictures SGA demuxer and decoders
*   TTML subtitle encoder and muxer
*   identity video filter
*   msad video filter
*   gophers protocol
*   RIST protocol via librist

We strongly recommend users, distributors, and system integrators to upgrade unless they use current git master.

### June 15th, 2020, FFmpeg 4.3 "4:3"

[FFmpeg 4.3 "4:3"](download.html#release_4.3), a new major release, is now available! Some of the highlights:

*   v360 filter
*   Intel QSV-accelerated MJPEG decoding
*   Intel QSV-accelerated VP9 decoding
*   Support for TrueHD in mp4
*   Support AMD AMF encoder on Linux (via Vulkan)
*   IMM5 video decoder
*   ZeroMQ protocol
*   support Sipro ACELP.KELVIN decoding
*   streamhash muxer
*   sierpinski video source
*   scroll video filter
*   photosensitivity filter
*   anlms filter
*   arnndn filter
*   bilateral filter
*   maskedmin and maskedmax filters
*   VDPAU VP9 hwaccel
*   median filter
*   QSV-accelerated VP9 encoding
*   AV1 encoding support via librav1e
*   AV1 frame merge bitstream filter
*   AV1 Annex B demuxer
*   axcorrelate filter
*   mvdv decoder
*   mvha decoder
*   MPEG-H 3D Audio support in mp4
*   thistogram filter
*   freezeframes filter
*   Argonaut Games ADPCM decoder
*   Argonaut Games ASF demuxer
*   xfade video filter
*   xfade_opencl filter
*   afirsrc audio filter source
*   pad_opencl filter
*   Simon & Schuster Interactive ADPCM decoder
*   Real War KVAG demuxer
*   CDToons video decoder
*   siren audio decoder
*   Rayman 2 ADPCM decoder
*   Rayman 2 APM demuxer
*   cas video filter
*   High Voltage Software ADPCM decoder
*   LEGO Racers ALP (.tun & .pcm) demuxer
*   AMQP 0-9-1 protocol (RabbitMQ)
*   Vulkan support
*   avgblur_vulkan, overlay_vulkan, scale_vulkan and chromaber_vulkan filters
*   ADPCM IMA MTF decoder
*   FWSE demuxer
*   DERF DPCM decoder
*   DERF demuxer
*   CRI HCA decoder
*   CRI HCA demuxer
*   overlay_cuda filter
*   switch from AvxSynth to AviSynth+ on Linux
*   mv30 decoder
*   Expanded styling support for 3GPP Timed Text Subtitles (movtext)
*   WebP parser
*   tmedian filter
*   maskedthreshold filter
*   Support for muxing pcm and pgs in m2ts
*   Cunning Developments ADPCM decoder
*   asubboost filter
*   Pro Pinball Series Soundbank demuxer
*   pcm_rechunk bitstream filter
*   scdet filter
*   NotchLC decoder
*   gradients source video filter
*   MediaFoundation encoder wrapper
*   untile filter
*   Simon & Schuster Interactive ADPCM encoder
*   PFM decoder
*   dblur video filter
*   Real War KVAG muxer

We strongly recommend users, distributors, and system integrators to upgrade unless they use current git master.

### October 5th, 2019, Bright Lights

FFmpeg has added a realtime bright flash removal filter to libavfilter.

Note that this filter is not FDA approved, nor are we medical professionals. Nor has this filter been tested with anyone who has photosensitive epilepsy. FFmpeg and its photosensitivity filter are not making any medical claims.

That said, this is a new video filter that may help photosensitive people watch tv, play video games or even be used with a VR headset to block out epiletic triggers such as filtered sunlight when they are outside. Or you could use it against those annoying white flashes on your tv screen. The filter fails on some input, such as the [Incredibles 2 Screen Slaver](https://www.youtube.com/watch?v=8L_9hXnUzRk) scene. It is not perfect. If you have other clips that you want this filter to work better on, please report them to us on our [trac](http://trac.ffmpeg.org).

[See for yourself](http://ffmpeg.org/~compn/output20p8.mp4). Example was made with -vf photosensitivity=20:0.8

We are not professionals. Please use this in your medical studies to advance epilepsy research. If you decide to use this in a medical setting, or make a hardware hdmi input output realtime tv filter, or find another use for this, [please let me know](mailto:compn@ffmpeg.org). This filter was a feature request of mine [since 2013](https://trac.ffmpeg.org/ticket/2104).

### August 5th, 2019, FFmpeg 4.2 "Ada"

[FFmpeg 4.2 "Ada"](download.html#release_4.2), a new major release, is now available! Some of the highlights:

*   tpad filter
*   AV1 decoding support through libdav1d
*   dedot filter
*   chromashift and rgbashift filters
*   freezedetect filter
*   truehd_core bitstream filter
*   dhav demuxer
*   PCM-DVD encoder
*   GIF parser
*   vividas demuxer
*   hymt decoder
*   anlmdn filter
*   maskfun filter
*   hcom demuxer and decoder
*   ARBC decoder
*   libaribb24 based ARIB STD-B24 caption support (profiles A and C)
*   Support decoding of HEVC 4:4:4 content in nvdec and cuviddec
*   removed libndi-newtek
*   agm decoder
*   KUX demuxer
*   AV1 frame split bitstream filter
*   lscr decoder
*   lagfun filter
*   asoftclip filter
*   Support decoding of HEVC 4:4:4 content in vdpau
*   colorhold filter
*   xmedian filter
*   asr filter
*   showspatial multimedia filter
*   VP4 video decoder
*   IFV demuxer
*   derain filter
*   deesser filter
*   mov muxer writes tracks with unspecified language instead of English by default
*   added support for using clang to compile CUDA kernels

We strongly recommend users, distributors, and system integrators to upgrade unless they use current git master.

### November 6th, 2018, FFmpeg 4.1 "al-Khwarizmi"

[FFmpeg 4.1 "al-Khwarizmi"](download.html#release_4.1), a new major release, is now available! Some of the highlights:

*   deblock filter
*   tmix filter
*   amplify filter
*   fftdnoiz filter
*   aderivative and aintegral audio filters
*   pal75bars and pal100bars video filter sources
*   mbedTLS based TLS support
*   adeclick and adeclip filters
*   libtensorflow backend for DNN based filters like srcnn
*   VC1 decoder is now bit-exact
*   ATRAC9 decoder
*   lensfun wrapper filter
*   colorconstancy filter
*   AVS2 video decoder via libdavs2
*   IMM4 video decoder
*   Brooktree ProSumer video decoder
*   MatchWare Screen Capture Codec decoder
*   WinCam Motion Video decoder
*   1D LUT filter (lut1d)
*   RemotelyAnywhere Screen Capture decoder
*   cue and acue filters
*   Support for AV1 in MP4 and Matroska/WebM
*   transpose_npp filter
*   AVS2 video encoder via libxavs2
*   amultiply filter
*   Block-Matching 3d (bm3d) denoising filter
*   acrossover filter
*   ilbc decoder
*   audio denoiser as afftdn filter
*   AV1 parser
*   sinc audio filter source
*   chromahold filter
*   setparams filter
*   vibrance filter
*   S12M timecode decoding in h264
*   xstack filter
*   (a)graphmonitor filter
*   yadif_cuda filter

We strongly recommend users, distributors, and system integrators to upgrade unless they use current git master.

### April 20th, 2018, FFmpeg 4.0 "Wu"

[FFmpeg 4.0 "Wu"](download.html#release_4.0), a new major release, is now available! Some of the highlights:

*   Bitstream filters for editing metadata in H.264, HEVC and MPEG-2 streams
*   Experimental MagicYUV encoder
*   TiVo ty/ty+ demuxer
*   Intel QSV-accelerated MJPEG encoding
*   native aptX and aptX HD encoder and decoder
*   NVIDIA NVDEC-accelerated H.264, HEVC, MJPEG, MPEG-1/2/4, VC1, VP8/9 hwaccel decoding
*   Intel QSV-accelerated overlay filter
*   mcompand audio filter
*   acontrast audio filter
*   OpenCL overlay filter
*   video mix filter
*   video normalize filter
*   audio lv2 wrapper filter
*   VAAPI MJPEG and VP8 decoding
*   AMD AMF H.264 and HEVC encoders
*   video fillborders filter
*   video setrange filter
*   support LibreSSL (via libtls)
*   Dropped support for building for Windows XP. The minimum supported Windows version is Windows Vista.
*   deconvolve video filter
*   entropy video filter
*   hilbert audio filter source
*   aiir audio filter
*   Removed the ffserver program
*   Removed the ffmenc and ffmdec muxer and demuxer
*   VideoToolbox HEVC encoder and hwaccel
*   VAAPI-accelerated ProcAmp (color balance), denoise and sharpness filters
*   Add android_camera indev
*   codec2 en/decoding via libcodec2
*   native SBC encoder and decoder
*   drmeter audio filter
*   hapqa_extract bitstream filter
*   filter_units bitstream filter
*   AV1 Support through libaom
*   E-AC-3 dependent frames support
*   bitstream filter for extracting E-AC-3 core
*   Haivision SRT protocol via libsrt
*   vfrdet filter

We strongly recommend users, distributors, and system integrators to upgrade unless they use current git master.

### October 15th, 2017, FFmpeg 3.4 "Cantor"

[FFmpeg 3.4 "Cantor"](download.html#release_3.4), a new major release, is now available! Some of the highlights:

*   deflicker video filter
*   doubleweave video filter
*   lumakey video filter
*   pixscope video filter
*   oscilloscope video filter
*   update cuvid/nvenc headers to Video Codec SDK 8.0.14
*   afir audio filter
*   scale_cuda CUDA based video scale filter
*   librsvg support for svg rasterization
*   crossfeed audio filter
*   spec compliant VP9 muxing support in MP4
*   surround audio filter
*   sofalizer filter switched to libmysofa
*   Gremlin Digital Video demuxer and decoder
*   headphone audio filter
*   superequalizer audio filter
*   roberts video filter
*   additional frame format support for Interplay MVE movies
*   support for decoding through D3D11VA in ffmpeg
*   limiter video filter
*   libvmaf video filter
*   Dolby E decoder and SMPTE 337M demuxer
*   unpremultiply video filter
*   tlut2 video filter
*   floodfill video filter
*   pseudocolor video filter
*   raw G.726 muxer and demuxer, left- and right-justified
*   NewTek NDI input/output device
*   FITS demuxer and decoder
*   FITS muxer and encoder
*   despill video filter
*   haas audio filter
*   SUP/PGS subtitle muxer
*   convolve video filter
*   VP9 tile threading support
*   KMS screen grabber
*   CUDA thumbnail filter
*   V4L2 mem2mem HW assisted codecs
*   Rockchip MPP hardware decoding
*   vmafmotion video filter

We strongly recommend users, distributors, and system integrators to upgrade unless they use current git master.

### April 13th, 2017, FFmpeg 3.3 "Hilbert"

[FFmpeg 3.3 "Hilbert"](download.html#release_3.3), a new major release, is now available! Some of the highlights:

*   Apple Pixlet decoder
*   NewTek SpeedHQ decoder
*   QDMC audio decoder
*   PSD (Photoshop Document) decoder
*   FM Screen Capture decoder
*   ScreenPressor decoder
*   XPM decoder
*   DNxHR decoder fixes for HQX and high resolution videos
*   ClearVideo decoder (partial)
*   16.8 and 24.0 floating point PCM decoder
*   Intel QSV-accelerated VP8 video decoding
*   native Opus encoder
*   DNxHR 444 and HQX encoding
*   Quality improvements for the (M)JPEG encoder
*   VAAPI-accelerated MPEG-2 and VP8 encoding
*   premultiply video filter
*   abitscope multimedia filter
*   readeia608 filter
*   threshold filter
*   midequalizer filter
*   MPEG-7 Video Signature filter
*   add internal ebur128 library, remove external libebur128 dependency
*   Intel QSV video scaling and deinterlacing filters
*   Sample Dump eXchange demuxer
*   MIDI Sample Dump Standard demuxer
*   Scenarist Closed Captions demuxer and muxer
*   Support MOV with multiple sample description tables
*   Pro-MPEG CoP #3-R2 FEC protocol
*   Support for spherical videos
*   CrystalHD decoder moved to new decode API
*   configure now fails if autodetect-libraries are requested but not found

We strongly recommend users, distributors, and system integrators to upgrade unless they use current git master.

### October 30th, 2016, Results: Summer Of Code 2016.

This has been a long time coming but we wanted to give a proper closure to our participation in this run of the program and it takes time. Sometimes it's just to get the final report for each project trimmed down, others, is finalizing whatever was still in progress when the program finished: final patches need to be merged, TODO lists stabilized, future plans agreed; you name it.

Without further ado, here's the silver-lining for each one of the projects we sought to complete during this Summer of Code season:

#### FFv1 (Mentor: Michael Niedermayer)

Stanislav Dolganov designed and implemented experimental support for motion estimation and compensation in the lossless FFV1 codec. The design and implementation is based on the snow video codec, which uses OBMC. Stanislav's work proved that significant compression gains can be achieved with inter frame compression. FFmpeg welcomes Stanislav to continue working beyond this proof of concept and bring its advances into the official FFV1 specification within the IETF.

#### Self test coverage (Mentor: Michael Niedermayer)

Petru Rares Sincraian added several self-tests to FFmpeg and successfully went through the in-some-cases tedious process of fine tuning tests parameters to avoid known and hard to avoid problems, like checksum mismatches due to rounding errors on the myriad of platforms we support. His work has improved the code coverage of our self tests considerably.

#### MPEG-4 ALS encoder implementation (Mentor: Thilo Borgmann)

Umair Khan updated and integrated the ALS encoder to fit in the current FFmpeg codebase. He also implemented a missing feature for the ALS decoder that enables floating-point sample decoding. FFmpeg support for MPEG-4 ALS has been improved significantly by Umair's work. We welcome him to keep maintaining his improvements and hope for great contributions to come.

#### Tee muxer improvements (Mentor: Marton Balint)

Ján Sebechlebský's generic goal was to improve the tee muxer so it tolerated blocking IO and allowed transparent error recovery. During the design phase it turned out that this functionality called for a separate muxer, so Ján spent his summer working on the so-called FIFO muxer, gradually fixing issues all over the codebase. He succeeded in his task, and the FIFO muxer is now part of the main repository, alongside several other improvements he made in the process.

#### TrueHD encoder (Mentor: Rostislav Pehlivanov)

Jai Luthra's objective was to update the out-of-tree and pretty much abandoned MLP (Meridian Lossless Packing) encoder for libavcodec and improve it to enable encoding to the TrueHD format. For the qualification period the encoder was updated such that it was usable and throughout the summer, successfully improved adding support for multi-channel audio and TrueHD encoding. Jai's code has been merged into the main repository now. While a few problems remain with respect to LFE channel and 32 bit sample handling, these are in the process of being fixed such that effort can be finally put in improving the encoder's speed and efficiency.

#### Motion interpolation filter (Mentor: Paul B Mahol)

Davinder Singh investigated existing motion estimation and interpolation approaches from the available literature and previous work by our own: Michael Niedermayer, and implemented filters based on this research. These filters allow motion interpolating frame rate conversion to be applied to a video, for example, to create a slow motion effect or change the frame rate while smoothly interpolating the video along the motion vectors. There's still work to be done to call these filters 'finished', which is rather hard all things considered, but we are looking optimistically at their future.

And that's it. We are happy with the results of the program and immensely thankful for the opportunity of working with such an amazing set of students. We can be a tough crowd but our mentors did an amazing job at hand holding our interns through their journey. Thanks also to Google for this wonderful program and to everyone that made room in their busy lives to help making GSoC2016 a success. See you in 2017!

### September 24th, 2016, SDL1 support dropped.

Support for the SDL1 library has been dropped, due to it no longer being maintained (as of January, 2012) and it being superseded by the SDL2 library. As a result, the SDL1 output device has also been removed and replaced by an SDL2 implementation. Both the ffplay and opengl output devices have been updated to support SDL2.

### August 9th, 2016, FFmpeg 3.1.2 "Laplace"

[FFmpeg 3.1.2](download.html#release_3.1), a new point release from the 3.1 release branch, is now available! It fixes several bugs.

We recommend users, distributors, and system integrators, to upgrade unless they use current git master.

### July 10th, 2016, ffserver program being dropped

After thorough deliberation, we're announcing that we're about to drop the ffserver program from the project starting with the next release. ffserver has been a problematic program to maintain due to its use of internal APIs, which complicated the recent cleanups to the libavformat library, and block further cleanups and improvements which are desired by API users and will be easier to maintain. Furthermore the program has been hard for users to deploy and run due to reliability issues, lack of knowledgable people to help and confusing configuration file syntax. Current users and members of the community are invited to write a replacement program to fill the same niche that ffserver did using the new APIs and to contact us so we may point users to test and contribute to its development.

### July 1st, 2016, FFmpeg 3.1.1 "Laplace"

[FFmpeg 3.1.1](download.html#release_3.1), a new point release from the 3.1 release branch, is now available! It mainly deals with a few ABI issues introduced in the previous release.

We strongly recommend users, distributors, and system integrators, especially those who experienced issues upgrading from 3.0, to upgrade unless they use current git master.

### June 27th, 2016, FFmpeg 3.1 "Laplace"

[FFmpeg 3.1 "Laplace"](download.html#release_3.1), a new major release, is now available! Some of the highlights:

*   DXVA2-accelerated HEVC Main10 decoding
*   fieldhint filter
*   loop video filter and aloop audio filter
*   Bob Weaver deinterlacing filter
*   firequalizer filter
*   datascope filter
*   bench and abench filters
*   ciescope filter
*   protocol blacklisting API
*   MediaCodec H264 decoding
*   VC-2 HQ RTP payload format (draft v1) depacketizer and packetizer
*   VP9 RTP payload format (draft v2) packetizer
*   AudioToolbox audio decoders
*   AudioToolbox audio encoders
*   coreimage filter (GPU based image filtering on OSX)
*   libdcadec removed
*   bitstream filter for extracting DTS core
*   ADPCM IMA DAT4 decoder
*   musx demuxer
*   aix demuxer
*   remap filter
*   hash and framehash muxers
*   colorspace filter
*   hdcd filter
*   readvitc filter
*   VAAPI-accelerated format conversion and scaling
*   libnpp/CUDA-accelerated format conversion and scaling
*   Duck TrueMotion 2.0 Real Time decoder
*   Wideband Single-bit Data (WSD) demuxer
*   VAAPI-accelerated H.264/HEVC/MJPEG encoding
*   DTS Express (LBR) decoder
*   Generic OpenMAX IL encoder with support for Raspberry Pi
*   IFF ANIM demuxer & decoder
*   Direct Stream Transfer (DST) decoder
*   loudnorm filter
*   MTAF demuxer and decoder
*   MagicYUV decoder
*   OpenExr improvements (tile data and B44/B44A support)
*   BitJazz SheerVideo decoder
*   CUDA CUVID H264/HEVC decoder
*   10-bit depth support in native utvideo decoder
*   libutvideo wrapper removed
*   YUY2 Lossless Codec decoder
*   VideoToolbox H.264 encoder

We strongly recommend users, distributors, and system integrators to upgrade unless they use current git master.

### March 16th, 2016, Google Summer of Code

FFmpeg has been accepted as a [Google Summer of Code](https://summerofcode.withgoogle.com/) open source organization. If you wish to participate as a student see our [project ideas page](https://trac.ffmpeg.org/wiki/SponsoringPrograms/GSoC/2016). You can already get in contact with mentors and start working on qualification tasks as well as register at google and submit your project proposal draft. Good luck!

### February 15th, 2016, FFmpeg 3.0 "Einstein"

[FFmpeg 3.0 "Einstein"](download.html#release_3.0), a new major release, is now available! Some of the highlights:

We strongly recommend users, distributors, and system integrators to upgrade unless they use current git master.

### January 30, 2016, Removing support for two external AAC encoders

We have just removed support for VisualOn AAC encoder (libvo-aacenc) and libaacplus in FFmpeg master.

Even before marking our internal AAC encoder as [stable](#aac_encoder_stable), it was known that libvo-aacenc was of an inferior quality compared to our native one for most samples. However, the VisualOn encoder was used extensively by the Android Open Source Project, and we would like to have a tested-and-true stable option in our code base.

When first committed in 2011, libaacplus filled in the gap of encoding High Efficiency AAC formats (HE-AAC and HE-AACv2), which was not supported by any of the encoders in FFmpeg at that time.

The circumstances for both have changed. After the work spearheaded by Rostislav Pehlivanov and Claudio Freire, the now-stable FFmpeg native AAC encoder is ready to compete with much more mature encoders. The Fraunhofer FDK AAC Codec Library for Android was added in 2012 as the fourth supported external AAC encoder, and the one with the best quality and the most features supported, including HE-AAC and HE-AACv2.

Therefore, we have decided that it is time to remove libvo-aacenc and libaacplus. If you are currently using libvo-aacenc, prepare to transition to the native encoder (`aac`) when updating to the next version of FFmpeg. In most cases it is as simple as merely swapping the encoder name. If you are currently using libaacplus, start using FDK AAC (`libfdk_aac`) with an appropriate `profile` option to select the exact AAC profile that fits your needs. In both cases, you will enjoy an audible quality improvement and as well as fewer licensing headaches.

Enjoy!

### January 16, 2016, FFmpeg 2.8.5, 2.7.5, 2.6.7, 2.5.10

We have made several new point releases (**[2.8.5](download.html#release_2.8), [2.7.5](download.html#release_2.7), [2.6.7](download.html#release_2.6), [2.5.10](download.html#release_2.5)**). They fix various bugs, as well as CVE-2016-1897 and CVE-2016-1898. Please see the changelog for each release for more details.

We recommend users, distributors and system integrators to upgrade unless they use current git master.

### December 5th, 2015, The native FFmpeg AAC encoder is now stable!

After seven years the native FFmpeg AAC encoder has had its experimental flag removed and declared as ready for general use. The encoder is transparent at 128kbps for most samples tested with artifacts only appearing in extreme cases. Subjective quality tests put the encoder to be of equal or greater quality than most of the other encoders available to the public.

Licensing has always been an issue with encoding AAC audio as most of the encoders have had a license making FFmpeg unredistributable if compiled with support for them. The fact that there now exists a fully open and truly free AAC encoder integrated directly within the project means a lot to those who wish to use accepted and widespread standards.

The majority of the work done to bring the encoder up to quality was started during this year's GSoC by developer Claudio Freire and Rostislav Pehlivanov. Both continued to work on the encoder with the latter joining as a developer and mainainer, working on other parts of the project as well. Also, thanks to [Kamedo2](http://d.hatena.ne.jp/kamedo2/) who does comparisons and tests, the original authors and all past and current contributors to the encoder. Users are suggested and encouraged to use the encoder and provide feedback or breakage reports through our [bug tracker](https://trac.ffmpeg.org/).

A big thank you note goes to our newest supporters: MediaHub and Telepoint. Both companies have donated a dedicated server with free of charge internet connectivity. Here is a little bit about them in their own words:

*   [Telepoint](http://www.telepoint.bg/en/) is the biggest carrier-neutral data center in Bulgaria. Located in the heart of Sofia on a cross-road of many Bulgarian and International networks, the facility is a fully featured Tier 3 data center that provides flexible customer-oriented colocation solutions (ranging from a server to a private collocation hall) and a high level of security.

*   MediaHub Ltd. is a Bulgarian IPTV platform and services provider which uses FFmpeg heavily since it started operating a year ago. *"Donating to help keep FFmpeg online is our way of giving back to the community"* .

Thanks Telepoint and MediaHub for their support!

### September 29th, 2015, GSoC 2015 results

FFmpeg participated to the latest edition of the [Google Summer of Code](http://www.google-melange.com/gsoc/homepage/google/gsoc2015) Project. FFmpeg got a total of 8 assigned projects, and 7 of them were successful.

We want to thank [Google](https://www.google.com), the participating students, and especially the mentors who joined this effort. We're looking forward to participating in the next GSoC edition!

Below you can find a brief description of the final outcome of each single project.

#### Basic servers for network protocols, mentee: Stephan Holljes, mentor: Nicolas George

Stephan Holljes's project for this session of Google Summer of Code was to implement basic HTTP server features for libavformat, to complement the already present HTTP client and RTMP and RTSP server code.

The first part of the project was to make the HTTP code capable of accepting a single client; it was completed partly during the qualification period and partly during the first week of the summer. Thanks to this work, it is now possible to make a simple HTTP stream using the following commands:

```
    ffmpeg -i /dev/video0 -listen 1 -f matroska \
    -c:v libx264 -preset fast -tune zerolatency http://:8080
    ffplay http://localhost:8080/

```

The next part of the project was to extend the code to be able to accept several clients, simultaneously or consecutively. Since libavformat did not have an API for that kind of task, it was necessary to design one. This part was mostly completed before the midterm and applied shortly afterwards. Since the ffmpeg command-line tool is not ready to serve several clients, the test ground for that new API is an example program serving hard-coded content.

The last and most ambitious part of the project was to update ffserver to make use of the new API. It would prove that the API is usable to implement real HTTP servers, and expose the points where more control was needed. By the end of the summer, a first working patch series was undergoing code review.

#### Browsing content on the server, mentee: Mariusz Szczepańczyk, mentor: Lukasz Marek

Mariusz finished an API prepared by the FFmpeg community and implemented Samba directory listing as qualification task.

During the program he extended the API with the possibility to remove and rename files on remote servers. He completed the implementation of these features for file, Samba, SFTP, and FTP protocols.

At the end of the program, Mariusz provided a sketch of an implementation for HTTP directory listening.

#### Directshow digital video capture, mentee: Mate Sebok, mentor: Roger Pack

Mate was working on directshow input from digital video sources. He got working input from ATSC input sources, with specifiable tuner.

The code has not been committed, but a patch of it was sent to the ffmpeg-devel mailing list for future use.

The mentor plans on cleaning it up and committing it, at least for the ATSC side of things. Mate and the mentor are still working trying to finally figure out how to get DVB working.

#### Implementing full support for 3GPP Timed Text Subtitles, mentee: Niklesh Lalwani, mentor: Philip Langdale

Niklesh's project was to expand our support for 3GPP Timed Text subtitles. This is the native subtitle format for mp4 containers, and is interesting because it's usually the only subtitle format supported by the stock playback applications on iOS and Android devices.

ffmpeg already had basic support for these subtitles which ignored all formatting information - it just provided basic plain-text support.

Niklesh did work to add support on both the encode and decode side for text formatting capabilities, such as font size/colour and effects like bold/italics, highlighting, etc.

The main challenge here is that Timed Text handles formatting in a very different way from most common subtitle formats. It uses a binary encoding (based on mp4 boxes, naturally) and stores information separately from the text itself. This requires additional work to track which parts of the text formatting applies to, and explicitly dealing with overlapping formatting (which other formats support but Timed Text does not) so it requires breaking the overlapping sections into separate non-overlapping ones with different formatting.

Finally, Niklesh had to be careful about not trusting any size information in the subtitles - and that's no joke: the now infamous Android stagefright bug was in code for parsing Timed Text subtitles.

All of Niklesh's work is committed and was released in ffmpeg 2.8.

#### libswscale refactoring, mentee: Pedro Arthur, mentors: Michael Niedermayer, Ramiro Polla

Pedro Arthur has modularized the vertical and horizontal scalers. To do this he designed and implemented a generic filter framework and moved the existing scaler code into it. These changes now allow easily adding removing, splitting or merging processing steps. The implementation was benchmarked and several alternatives were tried to avoid speed loss.

He also added gamma corrected scaling support. An example to use gamma corrected scaling would be:

```
    ffmpeg -i input -vf scale=512:384:gamma=1 output

```

Pedro has done impressive work considering the short time available, and he is a FFmpeg committer now. He continues to contribute to FFmpeg, and has fixed some bugs in libswscale after GSoC has ended.

#### AAC Encoder Improvements, mentee: Rostislav Pehlivanov, mentor: Claudio Freire

Rostislav Pehlivanov has implemented PNS, TNS, I/S coding and main prediction on the native AAC encoder. Of all those extensions, only TNS was left in a less-than-usable state, but the implementation has been pushed (disabled) anyway since it's a good basis for further improvements.

PNS replaces noisy bands with a single scalefactor representing the energy of that band, gaining in coding efficiency considerably, and the quality improvements on low bitrates are impressive for such a simple feature.

TNS still needs some polishing, but has the potential to reduce coding artifacts by applying noise shaping in the temporal domain (something that is a source of annoying, notable distortion on low-entropy bands).

Intensity Stereo coding (I/S) can double coding efficiency by exploiting strong correlation between stereo channels, most effective on pop-style tracks that employ panned mixing. The technique is not as effective on classic X-Y recordings though.

Finally, main prediction improves coding efficiency by exploiting correlation among successive frames. While the gains have not been huge at this point, Rostislav has remained active even after the GSoC, and is polishing both TNS and main prediction, as well as looking for further improvements to make.

In the process, the MIPS port of the encoder was broken a few times, something he's also working to fix.

#### Animated Portable Network Graphics (APNG), mentee: Donny Yang, mentor: Paul B Mahol

Donny Yang implemented basic keyframe only APNG encoder as the qualification task. Later he wrote interframe compression via various blend modes. The current implementation tries all blend modes and picks one which takes the smallest amount of memory.

Special care was taken to make sure that the decoder plays correctly all files found in the wild and that the encoder produces files that can be played in browsers that support APNG.

During his work he was tasked to fix any encountered bug in the decoder due to the fact that it doesn't match APNG specifications. Thanks to this work, a long standing bug in the PNG decoder has been fixed.

For latter work he plans to continue working on the encoder, making it possible to select which blend modes will be used in the encoding process. This could speed up encoding of APNG files.

### September 9th, 2015, FFmpeg 2.8

We published release **[2.8](download.html#release_2.8)** as new major version. It contains all features and bug fixes of the git master branch from September 8th. Please see the **[changelog](https://raw.githubusercontent.com/FFmpeg/FFmpeg/release/2.8/Changelog)** for a list of the most important changes.

We recommend users, distributors and system integrators to upgrade unless they use current git master.

### August 1st, 2015, A message from the FFmpeg project

Dear multimedia community,

The resignation of Michael Niedermayer as leader of FFmpeg yesterday has come by surprise. He has worked tirelessly on the FFmpeg project for many years and we must thank him for the work that he has done. We hope that in the future he will continue to contribute to the project. In the coming weeks, the FFmpeg project will be managed by the active contributors.

The last four years have not been easy for our multimedia community - both contributors and users. We should now look to the future, try to find solutions to these issues, and to have reconciliation between the forks, which have split the community for so long.

Unfortunately, much of the disagreement has taken place in inappropriate venues so far, which has made finding common ground and solutions difficult. We aim to discuss this in our communities online over the coming weeks, and in person at the [VideoLAN Developer Days](https://www.videolan.org/videolan/events/vdd15/) in Paris in September: a neutral venue for the entire open source multimedia community.

The FFmpeg project.

### July 4th, 2015, FFmpeg needs a new host

**UPDATE:** We have received more than 7 offers for hosting and servers, thanks a lot to everyone!

After graciously hosting our projects ([FFmpeg](http://www.ffmpeg.org), [MPlayer](http://www.mplayerhq.hu) and [rtmpdump](http://rtmpdump.mplayerhq.hu)) for 4 years, Arpi (our hoster) has informed us that we have to secure a new host somewhere else immediately.

If you want to host an open source project, please let us know, either on [ffmpeg-devel](http://ffmpeg.org/mailman/listinfo/ffmpeg-devel) mailing list or irc.freenode.net #ffmpeg-devel.

We use about 4TB of storage and at least 4TB of bandwidth / month for various mailing lists, [trac](http://trac.ffmpeg.org), [samples repo](http://samples.ffmpeg.org), svn, etc.

### March 16, 2015, FFmpeg 2.6.1

We have made a new major release (**[2.6](download.html#release_2.6)**) and now one week afterward 2.6.1\. It contains all features and bugfixes of the git master branch from the 6th March. Please see the **[Release Notes](http://git.videolan.org/?p=ffmpeg.git;a=blob;f=RELEASE_NOTES;hb=release/2.6)** for a list of note-worthy changes.

We recommend users, distributors and system integrators to upgrade unless they use current git master.

### March 4, 2015, Google Summer of Code

FFmpeg has been accepted as a [Google Summer of Code](http://www.google-melange.com/gsoc/homepage/google/gsoc2015) Project. If you wish to participate as a student see our [project ideas page](https://trac.ffmpeg.org/wiki/SponsoringPrograms/GSoC/2015). You can already get in contact with mentors and start working on qualification tasks. Registration at Google for students will open March 16th. Good luck!

### March 1, 2015, Chemnitzer Linux-Tage

We happily announce that FFmpeg will be represented at Chemnitzer Linux-Tage (CLT) in Chemnitz, Germany. The event will take place on 21st and 22nd of March.

More information can be found [here](https://chemnitzer.linux-tage.de/2015/en/)

We demonstrate usage of FFmpeg, answer your questions and listen to your problems and wishes. **If you have media files that cannot be processed correctly with FFmpeg, be sure to have a sample with you so we can have a look!**

For the first time in our CLT history, there will be an **FFmpeg workshop**! You can read the details [here](https://chemnitzer.linux-tage.de/2015/de/programm/beitrag/209). The workshop is targeted at FFmpeg beginners. First the basics of multimedia will be covered. Thereafter you will learn how to use that knowledge and the FFmpeg CLI tools to analyse and process media files. The workshop is in German language only and prior registration is necessary. The workshop will be on Saturday starting at 10 o'clock.

We are looking forward to meet you (again)!

### December 5, 2014, FFmpeg 2.5

We have made a new major release (**[2.5](download.html#release_2.5)**) It contains all features and bugfixes of the git master branch from the 4th December. Please see the **[Release Notes](http://git.videolan.org/?p=ffmpeg.git;a=blob;f=RELEASE_NOTES;hb=release/2.5)** for a list of note-worthy changes.

We recommend users, distributors and system integrators to upgrade unless they use current git master.

### October 10, 2014, FFmpeg is in Debian unstable again

We wanted you to know there are [FFmpeg packages in Debian unstable](https://packages.debian.org/search?keywords=ffmpeg&searchon=sourcenames&suite=unstable&section=main) again. **A big thank-you to Andreas Cadhalpun and all the people that made it possible.** It has been anything but simple.

Unfortunately that was already the easy part of this news. The bad news is the packages probably won't migrate to Debian testing to be in the upcoming release codenamed jessie. [Read the argumentation over at Debian.](https://bugs.debian.org/cgi-bin/bugreport.cgi?bug=763148)

**However things will come out in the end, we hope for your continued remarkable support!**

### October 8, 2014, FFmpeg secured a place in OPW!

Thanks to a generous 6K USD donation by Samsung (Open Source Group), FFmpeg will be welcoming at least 1 "Outreach Program for Women" intern to work with our community for an initial period starting December 2014 (through March 2015).

We all know FFmpeg is used by the industry, but even while there are countless products building on our code, it is not at all common for companies to step up and help us out when needed. So a big thank-you to Samsung and the OPW program committee!

If you are thinking on participating in OPW as an intern, please take a look at our [OPW wiki page](https://trac.ffmpeg.org/wiki/SponsoringPrograms/OPW/2014-12) for some initial guidelines. The page is still a work in progress, but there should be enough information there to get you started. If you, on the other hand, are thinking on sponsoring work on FFmpeg through the OPW program, please get in touch with us at opw@ffmpeg.org. With your help, we might be able to secure some extra intern spots for this round!

### September 15, 2014, FFmpeg 2.4

We have made a new major release (**[2.4](download.html#release_2.4)**) It contains all features and bugfixes of the git master branch from the 14th September. Please see the **[Release Notes](http://git.videolan.org/?p=ffmpeg.git;a=blob;f=RELEASE_NOTES;hb=release/2.4)** for a list of note-worthy changes.

We recommend users, distributors and system integrators to upgrade unless they use current git master.

### August 20, 2014, FFmpeg 2.3.3, 2.2.7, 1.2.8

We have made several new point releases (**[2.3.3](download.html#release_2.3), [2.2.7](download.html#release_2.2), [1.2.8](download.html#release_1.2)**). They fix various bugs, as well as CVE-2014-5271 and CVE-2014-5272. Please see the changelog for more details.

We recommend users, distributors and system integrators to upgrade unless they use current git master.

### July 29, 2014, Help us out securing our spot in OPW

Following our previous post regarding our participation on this year's OPW (Outreach Program for Women), we are now reaching out to our users (both individuals and companies) to help us gather the needed money to secure our spot in the program.
We need to put together 6K USD as a minimum but securing more funds would help us towards getting more than one intern.
You can donate by credit card using [Click&Pledge](https://co.clickandpledge.com/advanced/default.aspx?wid=56226) and selecting the "OPW" option. If you would like to donate by money transfer or by check, please get in touch by [e-mail](mailto:opw@ffmpeg.org) and we will get back to you with instructions.
Thanks!

### July 20, 2014, New website

The FFmpeg project is proud to announce a brand new version of the website made by [db0](http://db0.fr). While this was initially motivated by the need for a larger menu, the whole website ended up being redesigned, and most pages got reworked to ease navigation. We hope you'll enjoy browsing it.

### July 17, 2014, FFmpeg 2.3

We have made a new major release (**[2.3](download.html#release_2.3)**) It contains all features and bugfixes of the git master branch from the 16th July. Please see the **[Release Notes](http://git.videolan.org/?p=ffmpeg.git;a=blob;f=RELEASE_NOTES;hb=489d066)** for a list of note-worthy changes.

We recommend users, distributors and system integrators to upgrade unless they use current git master.

### July 3, 2014, FFmpeg and the Outreach Program For Women

FFmpeg has started the process to become an OPW includer organization for the next round of the program, with internships starting December 9\. The [OPW](https://gnome.org/opw/) aims to "Help women (cis and trans) and genderqueer to get involved in free and open source software". Part of the process requires securing funds to support at least one internship (6K USD), so if you were holding on your donation to FFmpeg, this is a great chance for you to come forward, get in touch and help both the project and a great initiative!

We have set up an [email address](mailto:opw@ffmpeg.org) you can use to contact us about donations and general inquires regarding our participation in the program. Hope to hear from you soon!

### June 29, 2014, FFmpeg 2.2.4, 2.1.5, 2.0.5, 1.2.7, 1.1.12, 0.10.14

We have made several new point releases (**[2.2.4](download.html#release_2.2), [2.1.5](download.html#release_2.1), [2.0.5](download.html#release_2.0), [1.2.7](download.html#release_1.2), [1.1.12](download.html#release_1.1), [0.10.14](download.html#release_0.10)**). They fix a [security issue in the LZO implementation](http://blog.securitymouse.com/2014/06/raising-lazarus-20-year-old-bug-that.html), as well as several other bugs. See the git log for details.

We recommend users, distributors and system integrators to upgrade unless they use current git master.

### May 1, 2014, LinuxTag

Once again FFmpeg will be represented at LinuxTag in Berlin, Germany. The event will take place from 8th to 10th of May. Please note that this year's LinuxTag is at a different location closer to the city center.

We will have a shared booth with XBMC and VideoLAN. **If you have media files that cannot be processed correctly with FFmpeg, be sure to have a sample with you so we can have a look!**

More information about LinuxTag can be found [here](http://www.linuxtag.org/2014/)

We are looking forward to see you in Berlin!

### April 18, 2014, OpenSSL Heartbeat bug

Our server hosting the Trac issue tracker was vulnerable to the attack against OpenSSL known as "heartbleed". The OpenSSL software library was updated on 7th of April, shortly after the vulnerability was publicly disclosed. We have changed the private keys (and certificates) for all FFmpeg servers. The details were sent to the mailing lists by Alexander Strasser, who is part of the project server team. Here is a link to the user mailing list [archive](https://lists.ffmpeg.org/pipermail/ffmpeg-user/2014-April/020968.html) .

We encourage you to read up on ["OpenSSL heartbleed"](https://www.schneier.com/blog/archives/2014/04/heartbleed.html). **It is possible that login data for the issue tracker was exposed to people exploiting this security hole. You might want to change your password in the tracker and everywhere else you used that same password.**

### April 11, 2014, FFmpeg 2.2.1

We have made a new point releases (**[2.2.1](download.html#release_2.2)**). It contains bug fixes for Tickets #2893, #3432, #3469, #3486, #3495 and #3540 as well as several other fixes. See the git log for details.

### March 24, 2014, FFmpeg 2.2

We have made a new major release (**[2.2](download.html#release_2.2)**) It contains all features and bugfixes of the git master branch from 1st March. A partial list of new stuff is below:

```
    - HNM version 4 demuxer and video decoder
    - Live HDS muxer
    - setsar/setdar filters now support variables in ratio expressions
    - elbg filter
    - string validation in ffprobe
    - support for decoding through VDPAU in ffmpeg (the -hwaccel option)
    - complete Voxware MetaSound decoder
    - remove mp3_header_compress bitstream filter
    - Windows resource files for shared libraries
    - aeval filter
    - stereoscopic 3d metadata handling
    - WebP encoding via libwebp
    - ATRAC3+ decoder
    - VP8 in Ogg demuxing
    - side & metadata support in NUT
    - framepack filter
    - XYZ12 rawvideo support in NUT
    - Exif metadata support in WebP decoder
    - OpenGL device
    - Use metadata_header_padding to control padding in ID3 tags (currently used in
    MP3, AIFF, and OMA files), FLAC header, and the AVI "junk" block.
    - Mirillis FIC video decoder
    - Support DNx444
    - libx265 encoder
    - dejudder filter
    - Autodetect VDA like all other hardware accelerations

```

We recommend users, distributors and system integrators to upgrade unless they use current git master.

### February 3, 2014, Chemnitzer Linux-Tage

We happily announce that FFmpeg will be represented at `Chemnitzer Linux-Tage' in Chemnitz, Germany. The event will take place on 15th and 16th of March.

More information can be found [here](http://chemnitzer.linux-tage.de/2014/en/info/)

We invite you to visit us at our booth located in the Linux-Live area! There we will demonstrate usage of FFmpeg, answer your questions and listen to your problems and wishes.

**If you have media files that cannot be processed correctly with FFmpeg, be sure to have a sample with you so we can have a look!**

We are looking forward to meet you (again)!

### February 9, 2014, trac.ffmpeg.org / trac.mplayerhq.hu Security Breach

The server on which FFmpeg and MPlayer Trac issue trackers were installed was compromised. The affected server was taken offline and has been replaced and all software reinstalled. FFmpeg Git, releases, FATE, web and mailinglists are on other servers and were not affected. We believe that the original compromise happened to a server, unrelated to FFmpeg and MPlayer, several months ago. That server was used as a source to clone the VM that we recently moved Trac to. It is not known if anyone used the backdoor that was found.

We recommend all users to change their passwords. **Especially users who use a password on Trac that they also use elsewhere, should change that password at least elsewhere.**

### November 12, 2013, FFmpeg RFP in Debian

Since the splitting of Libav the Debian/Ubuntu maintainers have followed the Libav fork. Many people have requested the packaging of ffmpeg in Debian, as it is more feature-complete and in many cases less buggy.

[Rogério Brito](http://cynic.cc/blog/), a Debian developer, has proposed a Request For Package (RFP) in the Debian bug tracking system.

Please let the Debian and Ubuntu developers know that you support packaging of the real FFmpeg! See Debian [ticket #729203](http://bugs.debian.org/cgi-bin/bugreport.cgi?bug=729203) for more details.

### October 28, 2013, FFmpeg 2.1

We have made a new major release (**[2.1](download.html#release_2.1)**) It contains all features and bugfixes of the git master branch from 28th October. A partial list of new stuff is below:

```
    - aecho filter
    - perspective filter ported from libmpcodecs
    - ffprobe -show_programs option
    - compand filter
    - RTMP seek support
    - when transcoding with ffmpeg (i.e. not streamcopying), -ss is now accurate
    even when used as an input option. Previous behavior can be restored with
    the -noaccurate_seek option.
    - ffmpeg -t option can now be used for inputs, to limit the duration of
    data read from an input file
    - incomplete Voxware MetaSound decoder
    - read EXIF metadata from JPEG
    - DVB teletext decoder
    - phase filter ported from libmpcodecs
    - w3fdif filter
    - Opus support in Matroska
    - FFV1 version 1.3 is stable and no longer experimental
    - FFV1: YUVA(444,422,420) 9, 10 and 16 bit support
    - changed DTS stream id in lavf mpeg ps muxer from 0x8a to 0x88, to be
    more consistent with other muxers.
    - adelay filter
    - pullup filter ported from libmpcodecs
    - ffprobe -read_intervals option
    - Lossless and alpha support for WebP decoder
    - Error Resilient AAC syntax (ER AAC LC) decoding
    - Low Delay AAC (ER AAC LD) decoding
    - mux chapters in ASF files
    - SFTP protocol (via libssh)
    - libx264: add ability to encode in YUVJ422P and YUVJ444P
    - Fraps: use BT.709 colorspace by default for yuv, as reference fraps decoder does
    - make decoding alpha optional for prores, ffv1 and vp6 by setting
    the skip_alpha flag.
    - ladspa wrapper filter
    - native VP9 decoder
    - dpx parser
    - max_error_rate parameter in ffmpeg
    - PulseAudio output device
    - ReplayGain scanner
    - Enhanced Low Delay AAC (ER AAC ELD) decoding (no LD SBR support)
    - Linux framebuffer output device
    - HEVC decoder, raw HEVC demuxer, HEVC demuxing in TS, Matroska and MP4
    - mergeplanes filter

```

We recommend users, distributors and system integrators to upgrade unless they use current git master.*****