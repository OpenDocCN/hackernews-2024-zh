<!--yml
category: 未分类
date: 2024-05-27 13:40:21
-->

# "run0" as a sudo replacement [LWN.net]

> 来源：[https://lwn.net/Articles/971745/](https://lwn.net/Articles/971745/)

# "run0" as a sudo replacement

[Posted April 30, 2024 by corbet]

[This Mastodon stream](https://mastodon.social/@pid_eins/112353324518585654)

from Lennart Poettering describes a

`sudo`

replacement — called

`run0`

— that will be part of the upcoming systemd 256 release. It takes a rather different approach to the execution of privileged commands, avoiding the use of setuid (which he calls "SUID") permissions entirely.

> So, in my ideal world, we'd have an OS entirely without SUID. Let's throw out the concept of SUID on the dump of UNIX' bad ideas. An execution context for privileged code that is half under the control of unprivileged code and that needs careful manual clean-up is just not how security engineering should be done in 2024 anymore.

* * *

(

[Log in](https://lwn.net/Login/?target=/Articles/971745/)

to post comments)