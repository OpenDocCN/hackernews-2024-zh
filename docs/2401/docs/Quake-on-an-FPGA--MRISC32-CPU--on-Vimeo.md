<!--yml
category: 未分类
date: 2024-05-27 14:51:53
-->

# Quake on an FPGA (MRISC32 CPU) on Vimeo

> 来源：[https://vimeo.com/901506667](https://vimeo.com/901506667)

Booting the MC1 MRISC32 computer and running Quake 1.

The CPU runs at 110MHz, and the resolution is 320x180 (256 colors). The central rasterization routines have been optimized with MRISC32 vector instructions.

Framerates are playable (over 30 FPS on average), but there is still work to do on the memory subsystem in order to raise the performance even more.

Music: River Meditation by Jason Shaw

More info: [mrisc32.bitsnbites.eu](https://mrisc32.bitsnbites.eu)