<!--yml
category: 未分类
date: 2024-05-27 14:48:08
-->

# DOS, OS/2 construction on Macbook Apple Silicon

> 来源：[https://retrocoding.net/building-for-dos-os2-and-dos-on-a-macbook-apple-silicon](https://retrocoding.net/building-for-dos-os2-and-dos-on-a-macbook-apple-silicon)

I've previously written about [OpenWatcom as a gateway to the ancient world](https://retrocoding.net/openwatcom-gateway-to-ancient-world-of-x86). However, at that time, I focused on installing it using Docker, as OpenWatcom 1.9 only works on Windows and Linux. I'm not a full-time Windows or Linux user, so when I discovered that OpenWatcom's 2.0 series can be compiled and run on macOS, **and** it can run on ARM macOS, I was thrilled.

It's not mentioned on their website, and only in less prominent places like GitHub Issues. However, after testing it out, I believe it's worth sharing with others who share my hobby: writing old Windows (and DOS) applications but still wants to use their newest, shiniest MacBook to do so.

## Preparation

There are some preparations you need to do before building OpenWatcom source code on an Apple Silicon.

**First, the compiler and platform SDK**. To build anything on macOS, you need their official compiler and IDE. Xcode. Recently, they tried to slim everything down, so it'll be 2 GB-ish download from [developer.apple.com](http://developer.apple.com).

**Second, homebrew**. This is optional if you know what you're doing. However, to smoothly compile OpenWatcom, you'd need DOSBOX command line and I feel the easiest way to obtain DosBox is using homebrew. Instal it from [brew.sh](https://brew.sh).

## Getting Dependencies

To get the dependencies, first we install DosBox. In Terminal, use this command to install dosbox.

```
brew install dosbox 
```

That's the only dependencies you need.

## Compilation

Compilation is actually straightforward. You'd only to obtain the source code for "latest" OpenWatcom v2.0 series. I'm using the February 2024 archive. Go to the release page and dowload the source (.tar.gz). Extract it

```
tar xvzf open-watcom-v2-2024-02-02-Build.tar.gz 
```

Before we do everything, now open the file `setenv.h` and search for line with `export OWTOOLS` and change it to:

```
export OWTOOLS=CLANG 
```

This will change the compilation to clang, the compiler built into Xcode and then search for the line with `export OWDOSBOX`, uncomment and change it to.

```
export OWDOSBOX=dosbox 
```

From this point forward you're ready. The last step is just run this

```
./build.sh 
```

And, wait. This will compile the OpenWatcom compiler using your `clang` compiler on your ARM64 platform. If everything is finished, you can put the compiled product by using this command.

```
./build.sh rel 
```

This will copy the resulting compilers, supporting headers for all supported platforms, as well as the libraries into `rel` directory.

I'm usually using watcom on a directory that I put on paths. For me it's on `/opt/watcom`. So if you haven't made it yet, you can run this to create a directory for your watcom installation and copy everything into that place.

```
mkdir -p /opt/watcom
cd rel && copy -a . /opt/watcom 
```

To use the binary, you need to add the toolchain to `PATH`. The toolchain for Apple Silicon mac is in `armo64` subdirectory. You need to also setup the `WATCOM` environment variable. That will point to our installation directory.

```
export WATCOM=/opt/watcom
export PATH=$PATH:$WATCOM/armo64
export EDDAT=$WATCOM/eddat 
```

For each platform (16-bit Windows, 32-bit Windows, DOS, OS2) you need to set different environment variables for `INCLUDE` and `LIBS`. Here's for 32-bit Windows platform.

```
export INCLUDE=$WATCOM/h:$WATCOM/h/nt:$WATCOM/h/nt/ddk:$WATCOM/h/nt/sdk:$WATCOM/h/nt/sdk/ntddk:$WATCOM/h/nt/sdk/wdm:$WATCOM/h/nt/sdk/wdm/dd
export LIBS=$WATCOM/lib386:$WATCOM/lib386/nt:$WATCOM/lib386/nt/ddk:$WATCOM/lib386/nt/sdk:$WATCOM/lib386/nt/sdk/ntddk:$WATCOM/lib386/nt/sdk/wdm:$WATCOM/lib386/nt/sdk/wdm/ddk 
```

Please refer to the OpenWatcom/Watcom manual on where the headers and the libs are. You can basically guess from `/h` directory and `/libxxx` directory.

After that you can run `wcl386` to see whether it works or not.

```
❯ wcl386 -h
Open Watcom C/C++ x86 32-bit Compile and Link Utility
Version 2.0 beta Feb 10 2024 00:00:59
Copyright (c) 2002-2024 The Open Watcom Contributors. All Rights Reserved.
Portions Copyright (c) 1988-2002 Sybase, Inc. All Rights Reserved.
Source code is available under the Sybase Open Watcom Public License.
See https://github.com/open-watcom/open-watcom-v2 
```

This means that your watcom compilers are ready to be used to target old platforms.