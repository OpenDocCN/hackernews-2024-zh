<!--yml
category: 未分类
date: 2024-05-27 14:31:43
-->

# Introduction - OpenIPC

> 来源：[https://openipc.org/](https://openipc.org/)

### OpenIPC is an alternative open firmware for your IP camera.

OpenIPC is an open-source operating system targeting IP cameras with ARM and MIPS processors from several manufacturers in order to replace that closed, opaque, insecure, often abandoned and unsupported firmware pre-installed by a vendor.

OpenIPC Firmware comes as binary pre-compiled files for easy installation by end-user. Also, we provide full access to the source files for further development and improvement by any capable programmer willing to contribute to the project. OpenIPC source code is released under one of the most simple open source license agreements, [MIT License](https://opensource.org/licenses/MIT), giving users express permission to reuse code for any purpose, even as part of a proprietary software. We only ask you politely to contribute your improvements back to us. We would be grateful for any feedback and suggestions.

[Installation Instructions](https://github.com/OpenIPC/wiki/blob/master/en/installation.md) [Precompiled binary files](/supported-hardware) [Source code on GitHub](https://github.com/OpenIPC)

OpenIPC Firmware uses [Buildroot](https://buildroot.org/) to build its Linux distro, and utilizes either [Majestic](https://github.com/OpenIPC/majestic), [Mini](https://github.com/OpenIPC/mini) or [Venc](https://github.com/OpenIPC/silicon_research) streamer.

Majestic code while is not open, provides unprecedented performance and capabilities for a wide range of hardware. The author of Majestic streamer is looking into possibilities to open-source the codebase after he secures enough funds to support further open development. You can [help](/support-open-source) to make it happen sooner.

### Why OpenIPC Firmware?

First of all, OpenIPC Firmware brings you freedom. With OpenIPC on your camera you are the master of your streams. You have full access to the system and can control what, where, and how you want your camera to behave. There are no backdoors, no botnets, no crypto-mining malware, no keyloggers, no password sniffers, nothing you could possibly expect in a closed binary system with no access to its source code.

As for the Firmware capabilities, we strive to provide universal support for a wide range of cameras, thus we are focused on basic functionality first, allowing end-users to upgrade their cameras' firmware and stream video and audio (where supported) without too much hassle.

We have some interesting bits though. Our Firmware supports external IPEYE cloud storage, streaming video to Youtube and Telegram, using SOCKS5 proxy, setting up a Virtual Tunnel, and more.

There are also several projects focused on specialized equipment, such as a camera to be mounted on a drone, for mounting on construction helmets, for use in surveying tools, for medical research of the organs of vision, underwater fishing, etc.