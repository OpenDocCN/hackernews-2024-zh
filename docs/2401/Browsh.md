<!--yml
category: 未分类
date: 2024-05-27 14:52:23
-->

# Browsh

> 来源：[https://www.brow.sh/](https://www.brow.sh/)

**Browsh is a fully-modern text-based browser.** It renders anything that a modern browser can; HTML5, CSS3, JS, video and even WebGL. Its main purpose is to be run on a remote server and accessed via SSH/Mosh or the in-browser HTML service in order to significantly reduce bandwidth and thus both increase browsing speeds and decrease bandwidth costs.

## Download (v1.8.0)

Browsh is available as a single static binary on all major platforms. The only dependency is a recent 57+ version of Firefox.

[Latest version](/downloads)

|

[Releases archive](https://github.com/browsh-org/browsh/releases) A Docker image is also available:
`docker run -it browsh/browsh`

## Live SSH Demo

**Temporarily offline**

Just point your SSH client to

*brow.sh*

, eg;

`ssh brow.sh`

. No auth needed. The service is for demonstration only, sessions last 5 minutes and are logged.

Note that SSH is actually a very inefficient protocol, for best results install Browsh on your own server along with [Mosh](https://mosh.org/).

## In-browser Services

**Temporarily offline**

*   [html.brow.sh](https://html.brow.sh) Uses very basic graphics and HTML anchor tags. Although this service may appear similar to the terminal client it does not yet have feature parity.
*   [text.brow.sh](https://text.brow.sh) Uses nothing but pure text, better for usage with `curl`, for instance.

[https://www.youtube.com/embed/zqAoBD62gvo](https://www.youtube.com/embed/zqAoBD62gvo)

VIDEO

## Donate

Browsh is currently maintained and funded by

[one person](http://tombh.co.uk)

. If you'd like to see Browsh continue to help those with slow and/or expensive Internet, please consider

[donating](/donate)

.