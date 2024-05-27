<!--yml
category: 未分类
date: 2024-05-27 12:58:22
-->

# Release v4.6.0: The end · pivpn/pivpn · GitHub

> 来源：[https://github.com/pivpn/pivpn/releases/tag/v4.6.0](https://github.com/pivpn/pivpn/releases/tag/v4.6.0)

Hi everyone,

It's time to say goodbye.

This is the final official release of PiVPN.

I inherited this project from [@0-kaladin](https://github.com/0-kaladin) and [@redfast00](https://github.com/redfast00), who moved on with their lives. I maintained it as my own with the great help of [@orazioedoardo](https://github.com/orazioedoardo), to whom I'm immensely grateful. He held the boat and kept it floating while I could not be present, and he too gave a lot of himself to this project!

But now it's time for me too to move on!

I've been giving less and less attention to PiVPN, and the desire to keep up with it is no longer what it once was. When PiVPN was created, it filled a big void and had a clear mission and purpose, which I feel has been fulfilled! We went from OpenVPN being something hard to set up and complicated to manage, to WireGuard being able to run on any toaster and easy to set up. There are so many tools out there that do the job much better than PiVPN does, and I genuinely believe PiVPN's mission in life was accomplished and is no longer relevant. Just as everything in nature has a start, there's also an end, and this is how PiVPN ends its journey.

PiVPN has been home for so many of you, starting with Linux, bash, open-source, and everyone was always very welcomed, just like it was for me [7 years ago](https://github.com/pivpn/pivpn/commit/fbec57d1fda70341394bfd2bc90e1dab0af2c125). I cannot express how grateful I am to all the [84 Contributors](https://github.com/pivpn/pivpn/graphs/contributors) for this amazing project.

It has been a wild ride, and I've learned so much from PiVPN and from every single one of you!

THANK YOU!!

PiVPN repositories will be archived and set to read-only, and will no longer be maintained. Unless [@0-kaladin](https://github.com/0-kaladin) rises back again and decides otherwise.

The PiVPN Website and its documentation are hosted on GitHub, therefore it will remain accessible under the pivpn.io domain for as long as [@0-kaladin](https://github.com/0-kaladin) keeps paying the bills, just the same way I will keep hosting the redirection for the installation for as long as possible. I will still make a few commits to update the documentation about the project's state, but that will be it.

I will maintain ownership of the repository, but I won't pass it down to anyone else. First, because I feel it's not up to me to decide who to pass the project down to, and second, because there is no one else to pass the project to.

"But I want and can maintain it, can I take it over?" Let me put it plain and simple: No! I don't know you, I don't trust you! Fork it and carry on!

About this release, here's what it brings:

### New Features

*   Add possibility to use Pi-hole in unattended install ([#1825](https://github.com/pivpn/pivpn/pull/1825))

### Bugfixes and Refactors

*   Updates to subnet generation and client creation
    *   refactor(core): allow any subnet and netmask
    *   fix(scripts): prevent adding more clients than the subnet allows
    *   fix(scripts): correctly remove leading zeros from ipv6 quartets
    *   refactor(core): new probabilistic subnet generation with fallback to other RFC1918 subnets

**Full Changelog**: [`v4.5.0...v4.6.0`](https://github.com/pivpn/pivpn/compare/v4.5.0...v4.6.0)

Once again, Thank you all so much for everything! See you around!
4s3ti