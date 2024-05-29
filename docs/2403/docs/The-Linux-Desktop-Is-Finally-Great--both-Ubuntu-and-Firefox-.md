<!--yml
category: 未分类
date: 2024-05-27 15:01:07
-->

# The Linux Desktop Is Finally Great (both Ubuntu and Firefox)

> 来源：[https://punkx.org/jackdoe/linux-desktop.html](https://punkx.org/jackdoe/linux-desktop.html)

# TLDR: Both Ubuntu and Firefox are absolutely great, just try them out.

*Ubuntu 23.10, Firefox 118.0.1*
*Sat Mar 16 01:25:58 PM CET 2024*
*by [jackdoe](https://punkx.org/jackdoe)*

I have been using Chrome only pretty much since it came out; it just had smoother scrolling and after that 'translate', and it also used less CPU than Firefox.

I have been using Linux since Slack 3.6, FreeBSD for a while between 2.2.8 and 8.0 (including months of NetBSD/OpenBSD/DragonFly/OpenSolaris), but after that I have mainly been using macOS on my laptop. And I had Windows on my PC for the last 3-4 years to play World of Warcraft (which sometimes works OK with wine, but some patches break, and I got tired of fixing it every other update).

I never managed to make my Windows PC feel like home; I just always treated my Windows as a "hacked" public computer, and due to no full disk encryption, I never put anything sensitive on it, and therefore never worked on it. A few days ago my SSD died, and I installed Ubuntu 23.10 on the new SSD, with full disk encryption and beautifully set up Emacs.

### Things that work perfectly (in no particular order):

*   The installation itself
*   Multiple monitors, one rotated
*   Emacs, this was always the case, but just stating there is no regression
*   Firefox - now has translate
*   Showing when an app is using the microphone (even arecord(1) and of course Discord)
*   Discord, Slack
*   Notifications
*   The Espressif toolchain
*   The RISC-V toolchain (I use it while working on a new boardgame after [PROJEKT:OVERFLOW](https://punkx.org/overflow), and it was super annoying on Windows)
*   Pulseview (works even better than macOS, and much better than on Windows)
*   usbpcap + Wireshark
*   Screen recording
*   llama.cpp with Alpaca
*   Ubuntu's default terminal
*   Inkscape (cli and gui)
*   When you copy a file from Gnome Files and paste in the terminal, it actuall pastes its absolute path

Firefox surprised me the most; it is so smooth, and now it has translations, which was a huge blocker for me because I am in The Netherlands, and I don't speak Dutch, so I have to constantly translate things because most sites just infer the language from the IP address rather than the Accept-Language headers.

### Things that weren't great:

*   Lutris - which is winehacks + wine has some issues installing the latest Battlenet.exe, so I had to google how to fix it, but after that, WoW plays with 200-300 fps without issues on a 3080, has no input lag, and all my weakauras and addons just work, the text-to-speech is crap though (I use it to tell me when some important cooldowns are ready).
*   Text to Speech - sounds like a broken robot
*   ~~Installing fonts still requires *cp z.ttf /usr/share/fonts/truetype && sudo fc-cache -fv*~~ [[popey@lobsters](https://lobste.rs/s/qurpee/linux_desktop_is_finally_great_both#c_jqzcl4) corrected me that there is actually install button if you double click on the font, I just didn't see it]

I haven't changed a single setting or copy-pasted a single line from superuser.com. In the past, I have mainly been using EXWM, i3, and awesome, but now I just use the Ubuntu default window manager; I don't even know its name. I also did not have to recompile the kernel.

I don't even know if it's using wayland or not, and I don't care :)

I feel over the years Apple has been treating its users more and more like children, e.g., when you compile a program and the first time you run it, it starts slowly because "something" has to decide if it's "allowed" or should be quarentined, or you can't *gdb -p* into a process. Every time it does that it just pisses me off so much. Since M1 came out, I can say only good things about the hardware, my mac often has 100-200 days uptime, battery life of 10+ hours.. I just cant stand the patronizing attitude.

Is it too much to ask? For my computer to execute the code I wrote and compiled?

I thought that the Linux desktop is leaps behind macOS, and I was proven wrong. Now I can't wait to get back to my Linux PC and just open my emacs. It feels like home.

PS: I told this story to my friend, and he said Fedora is even better.

* * *

*echo 'int main(void) { return 0; }' > aa.c && gcc -o aa aa.c && time ./aa && time ./aa*

**macOS**:
./aa 0.00s user 0.00s system 1% cpu 0.281 total
./aa 0.00s user 0.00s system 54% cpu 0.001 total

**Linux**:

real 0m0.001s
user 0m0.001s
sys  0m0.000s

real 0m0.001s
user 0m0.001s
sys  0m0.000s

At this point its just making fun of me.