<!--yml
category: 未分类
date: 2024-05-27 14:29:10
-->

# "No way to prevent this" say users of only language where this regularly happens - Xe Iaso

> 来源：[https://xeiaso.net/shitposts/no-way-to-prevent-this/CVE-2023-6246/](https://xeiaso.net/shitposts/no-way-to-prevent-this/CVE-2023-6246/)

# "No way to prevent this" say users of only language where this regularly happens

Published on 01/30/2024, 227 words, 1 minutes to read

<picture>![An image of A forlorn business man resting his head on a brown wall next to a window.](img/f937e7581f941877303189255f6ea353.png)</picture>A forlorn business man resting his head on a brown wall next to a window. - Photo by Andrea Piacquadio, source: Pexels

In the hours following the release of [CVE-2023-6246](https://www.qualys.com/2024/01/30/cve-2023-6246/syslog.txt) for the project [GNU glibc](https://sourceware.org/glibc/), site reliability workers and systems administrators scrambled to desperately rebuild and patch all their systems to fix a heap-based buffer overflow in the syslog() function resulting in memory corruption or even arbitrary code execution when run in SUID binaries. This is due to the affected components being written in C, the only programming language where these vulnerabilities regularly happen. "This was a terrible tragedy, but sometimes these things just happen and there's nothing anyone can do to stop them," said programmer Willodean Santorella, echoing statements expressed by hundreds of thousands of programmers who use the only language where 90% of the world's memory safety vulnerabilities have occurred in the last 50 years, and whose projects are 20 times more likely to have security vulnerabilities. "It's a shame, but what can we do? There really isn't anything we can do to prevent memory safety vulnerabilities from happening if the programmer doesn't want to write their code in a robust manner." At press time, users of the only programming language in the world where these vulnerabilities regularly happen once or twice per quarter for the last eight years were referring to themselves and their situation as "helpless."

* * *

Facts and circumstances may have changed since publication. Please contact me before jumping to conclusions if something seems wrong or unclear.

Tags: