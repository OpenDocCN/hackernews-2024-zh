<!--yml
category: 未分类
date: 2024-05-27 15:20:35
-->

# Wine on Wayland: A year in review (and a look ahead)

> 来源：[https://www.collabora.com/news-and-blog/news-and-events/wine-on-wayland-a-year-in-review-and-a-look-ahead.html](https://www.collabora.com/news-and-blog/news-and-events/wine-on-wayland-a-year-in-review-and-a-look-ahead.html)

Alexandros Frantzis
January 30, 2024

2023 was a great year for the Wayland driver for Wine. Our goal was to move forward from the experimental phase and make the driver a proper upstream component. A year later, after [several merge requests](https://gitlab.winehq.org/wine/wine/-/merge_requests?scope=all&state=merged&search=winewayland), many people are now already able to use the [latest Wine release](https://gitlab.winehq.org/wine/wine/-/releases/wine-9.0) to enjoy some of their favorite Windows applications in a completely X11-free environment!

Here is what we have in upstream so far:

*   Basic window management (fullscreen, maximization, resize, etc)
*   Software rendering (i.e., GDI)
*   Mouse support, including mouselook
*   Keyboard support, including keymap handling
*   Vulkan, including Direct3D through WineD3D/Vulkan or DXVK
*   Basic support for HiDPI

Our work is not yet done, however. We will continue our upstreaming efforts in 2024, focusing on:

*   Emulation of display mode changes through compositor scaling
*   OpenGL support
*   Improved positioning of transient windows (popups, menus, etc)
*   Even more window management (e.g., minimization)
*   Clipboard and drag-and-drop
*   General robustness improvements, bug fixes, code improvements

Some other features that would be great to have eventually:

*   Support for system DPI auto-detection and, ideally, per-monitor DPI handling in Wine core
*   Integration with the upcoming Wayland color-management (and hdr) protocol
*   Cross-process rendering

In every past driver update I included a video showcasing the progress we have made. This year however there are already several videos made by people using the Wayland driver (which is very exciting to see), so I'll let those videos speak for themselves!

Enjoy!