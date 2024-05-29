<!--yml
category: 未分类
date: 2024-05-29 13:19:00
-->

# NVIDIA open source driver to use NVK + Zink for OpenGL on newer GPUs | GamingOnLinux

> 来源：[https://www.gamingonlinux.com/2024/02/nvidia-open-source-driver-to-use-nvk-zink-for-opengl-on-newer-gpus/](https://www.gamingonlinux.com/2024/02/nvidia-open-source-driver-to-use-nvk-zink-for-opengl-on-newer-gpus/)

A [recent merge](https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/27628) request on the Mesa Git repository added the initial support for allowing drivers to chose Zink as the translation layer for handling OpenGL. This basically does an OpenGL-on-Vulkan implementation making it easier for newer cards to support OpenGL through a generic implementation, reducing duplicate code and making it easier to keep OpenGL around since while not yet deprecated it is being less used on modern games.

The merge allows owners of the NVIDIA GeForce RTX 20xx series GPUs and newer to opt to use Zink instead of relying on the default NVC0 Gallium3D implementation to handle OpenGL requests. Those who want to give it a try, it should be as simple as setting the `NOUVEAU_USE_ZINK=1` environment variable after updating Mesa 24.1 when your distribution receives the update that is currently only on the main branch of Mesa repository.

Since the merge from branch `airlied/zink-driver-choice` to `main` was merged [2 days ago](https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/27628#note_null), most distros might not be distributing those patches yet. For those who use source-based distros or any source-base package building system like AUR, the package `mesa-git` is [available](https://aur.archlinux.org/packages/mesa-git) as a means of tracking the main branch and testing these bleeding-edge features.

While NVK as a technology is still in-development with plenty left to do, there has been lots of improvements being implemented on the last year. Related articles:

Additionally, developer Mike Blumenkrantz recently put up another [blog post](https://www.supergoodcode.com/woof/) about NVK work and thanks to some recent changes mentioned that "As a result, all GL games now work on NVK. No hyperbole. They just work".

Article taken from [GamingOnLinux.com.](https://www.gamingonlinux.com/)