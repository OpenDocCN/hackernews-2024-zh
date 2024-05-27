<!--yml
category: 未分类
date: 2024-05-27 12:49:40
-->

# #10341 ([Intel Caption] Microsoft RTMP Ingest - FFmpeg fails to decode closed-caption to srt/webvtt) – FFmpeg

> 来源：[https://trac.ffmpeg.org/ticket/10341#comment:4](https://trac.ffmpeg.org/ticket/10341#comment:4)

# [Intel Caption] Microsoft RTMP Ingest - FFmpeg fails to decode closed-caption to srt/webvtt

Summary of the bug: All Windows FFmpeg versions after 4.2.3 fail to extract the Closed-caption eia-608 and convert it to srt or webvtt. It's stuck without generating any output.
How to reproduce:

```
% ffmpeg.exe -f lavfi -i movie=flvdecoder_input223.flv[out+subcc] -y  -map 0:1  ./output_p.srt
ffmpeg version N-102809-gde8e6e67e7-20210630 Copyright (c) 2000-2021 the FFmpeg developers
  built with gcc 10-win32 (GCC) 20210408
  configuration: --prefix=/ffbuild/prefix --pkg-config-flags=--static --pkg-config=pkg-config --cross-prefix=x86_64-w64-mingw32- --arch=x86_64 --target-os=mingw32 --enable-version3 --disable-debug --disable-w32threads --enable-pthreads --enable-iconv --enable-libxml2 --enable-zlib --enable-libfreetype --enable-libfribidi --enable-gmp --enable-lzma --enable-fontconfig --enable-libvorbis --enable-opencl --enable-libvmaf --enable-vulkan --enable-amf --enable-libaom --disable-avisynth --enable-libdav1d --disable-libdavs2 --disable-libfdk-aac --enable-ffnvcodec --enable-cuda-llvm --enable-libglslang --enable-libgme --enable-libass --enable-libbluray --enable-libmp3lame --enable-libopus --enable-libtheora --enable-libvpx --enable-libwebp --enable-lv2 --enable-libmfx --enable-libopencore-amrnb --enable-libopencore-amrwb --enable-libopenjpeg --enable-librav1e --disable-librubberband --enable-schannel --enable-sdl2 --enable-libsoxr --enable-libsrt --enable-libsvtav1 --enable-libtwolame --enable-libuavs3d --disable-libvidstab --disable-libx264 --disable-libx265 --disable-libxavs2 --disable-libxvid --enable-libzimg --extra-cflags=-DLIBTWOLAME_STATIC --extra-cxxflags= --extra-ldflags=-pthread --extra-ldexeflags= --extra-libs=-lgomp --extra-version=20210630
  libavutil      57\.  0.100 / 57\.  0.100
  libavcodec     59\.  3.100 / 59\.  3.100
  libavformat    59\.  3.101 / 59\.  3.101
  libavdevice    59\.  0.100 / 59\.  0.100
  libavfilter     8\.  0.103 /  8\.  0.103
  libswscale      6\.  0.100 /  6\.  0.100
  libswresample   4\.  0.100 /  4\.  0.100
[flv @ 0000022bbf6bbb40] Estimating duration from bitrate, this may be inaccurate
Input #0, lavfi, from 'movie=flvdecoder_input223.flv[out+subcc]':
  Duration: N/A, start: 0.000367, bitrate: N/A
  Stream #0:0: Video: rawvideo (I420 / 0x30323449), yuv420p, 1280x720 [SAR 1:1 DAR 16:9], 29.97 fps, 29.97 tbr, 1k tbn
  Stream #0:1: Subtitle: eia_608
Output #0, srt, to './output_p.srt':
  Metadata:
    encoder         : Lavf59.3.101
  Stream #0:0: Subtitle: subrip
    Metadata:
      encoder         : Lavc59.3.100 srt
Stream mapping:
  Stream #0:1 -> #0:0 (eia_608 (cc_dec) -> subrip (srt))
Press [q] to stop, [?] for help
size=       0kB time=00:00:00.00 bitrate=N/A speed=   0x

```

Patches should be submitted to the ffmpeg-devel mailing list and not this bug tracker.