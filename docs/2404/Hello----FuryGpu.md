<!--yml
category: 未分类
date: 2024-05-27 12:47:58
-->

# Hello! — FuryGpu

> 来源：[https://www.furygpu.com/blog/hello](https://www.furygpu.com/blog/hello)

After almost four years of developing a custom full-stack GPU in my spare time, I figured it was about time to start putting together some materials to highlight the project, its technical details, and its future. This has been my first (and only!) FPGA design, and I hope to use this space to share some of what I have learned while building this *extremely* complicated piece of hardware.

Throughout the last fourteen years of my career in the games industry (and for nearly a decade before in my spare time!) I’ve been focused on the software side of rendering - the techniques necessary to utilize the GPU hardware to render 3D graphics at real-time framerates. While I was extremely familiar with everything involved in this process on the host side, the actual details of how the hardware that performed these actions was built was not something I had ever had access to. After putting together Ben Eater’s [8-bit breadboard compute](https://eater.net/8bit)r and then picking up an [Arty Z7](https://digilent.com/shop/arty-z7-zynq-7000-soc-development-board/) development board on a whim, I realized that building a GPU from scratch, while a massive and daunting project, was something I certainly could teach myself to do. I’d spend a few months making a spinning cube or something, and be done with it.

Nearly four years later and the project has turned from a neat little tech demo idea into a fully-fledged, real-world, plug-it-into-your-computer GPU. I taught myself SystemVerilog, figured out how FPGAs work, and spent countless hours refactoring, redesigning, and streamlining the design until I could get Quake to render at semi-real-time framerates. During this time, Xilinx released their [Kria SoMs](https://www.amd.com/en/products/system-on-modules/kria.html) - insanely cheap Zynq UltraScale+ FPGAs with a ton of DSP units and a (comparatively) massive amount of LUTs and FFs, and of particular interest, a hardened PCIe core. The next step was clear - this GPU needed to become a *real* GPU. One I could plug into my computer and use to play real games.

Designing the schematic for and laying out a PCIe graphics card, even with much of the FPGA circuitry built into the SOM, was a herculean effort. However, I’d already spent several years on the project, so why not? After several months of work laying out the board in KiCad and fixing all the issues I’d find as soon as JLCPCB delivered each revision, I finally had a board with a powerful FPGA, DisplayPort and HDMI output, and a 4-lane PCIe connector that I could plug into my test rig and write drivers for.

Of all the parts of this project, writing Windows drivers for it have been *the most* painful. Eventually, I got everything working. I wrote a custom graphics API to communicate with the GPU, wrote Windows kernel drivers for the display and audio, and now have a fully-functional piece of graphics hardware that can render Quake at a solid 60 frames per second.

While I’m writing all of this after the fact, I do hope that some of what I eventually detail here will prove useful (or at least, interesting) to the next person that decides to embark on such a ridiculous journey.

- Dylan