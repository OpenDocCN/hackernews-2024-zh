<!--yml
category: 未分类
date: 2024-05-27 12:55:35
-->

# A stealth attack came close to compromising the world’s computers

> 来源：[https://www.economist.com/science-and-technology/2024/04/02/a-stealth-attack-came-close-to-compromising-the-worlds-computers](https://www.economist.com/science-and-technology/2024/04/02/a-stealth-attack-came-close-to-compromising-the-worlds-computers)

In 2020 XKCD, a popular online comic strip, published a cartoon depicting a teetering arrangement of blocks with the label: “all modern digital infrastructure”. Perched precariously at the bottom, holding everything up, was a lone, slender brick: “A project some random person in Nebraska has been thanklessly maintaining since 2003.” The illustration quickly became a cult classic among the technically minded, for it highlighted a harsh truth: the software at the [heart of the internet](https://www.economist.com/technology-quarterly/2024-02-03) is maintained not by giant corporations or sprawling bureaucracies but by a handful of earnest volunteers toiling in obscurity. A cyber-security scare in recent days shows how the result can be near-disaster.

On March 29th Andres Freund, an engineer at Microsoft, published a short detective story. In recent weeks he had noticed that SSH—a system to log on securely to another device over the internet—was running about 500 milliseconds more slowly than expected. Closer inspection revealed malicious code embedded deep inside XZ Utils, some software designed to compress data used inside the Linux operating system, which runs on virtually all publicly accessible internet servers. Those servers ultimately undergird the internet, including vital financial and government services. The malicious code would have served as a “master key”, allowing attackers to steal encrypted data or plant other malware.