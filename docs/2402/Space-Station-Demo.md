<!--yml
category: 未分类
date: 2024-05-27 14:35:53
-->

# Space Station Demo

> 来源：[https://www.lexaloffle.com/bbs/?pid=94322#p](https://www.lexaloffle.com/bbs/?pid=94322#p)

Copy and paste the snippet below into your HTML.

Note: This cartridge's settings do not allow embedded playback. A [Play at lexaloffle] link will be included instead.

Here's a quick demo of a 6 degree of freedom 3D engine that I have been working on.
You can just explore a small space station in a Descent-like style.

**Controls**
Z: Thrust
Arrow Keys: Turn
X+Arrows: Roll

**Notes**
The map is stored in a **long** string and is generated from a Blender OBJ file. The station is as complex as I can make it before running out of compressed code space. To make something larger, I think I would have to switch to more of a 3D tile system with room building blocks that snap together.

The engine uses a simplified portal system paired with convex sectors to determine room visibility and draw order.

**Thanks**
Thanks to Fred72 for the polyfill code. ([https://www.lexaloffle.com/bbs/?tid=3393](https://www.lexaloffle.com/bbs/?tid=3393))

Updates: fixed some code that broke with version 2.5g