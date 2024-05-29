<!--yml
category: 未分类
date: 2024-05-27 14:45:15
-->

# Chris's Wiki :: blog/unix/XOffscreenIconMistake

> 来源：[https://utcc.utoronto.ca/~cks/space/blog/unix/XOffscreenIconMistake](https://utcc.utoronto.ca/~cks/space/blog/unix/XOffscreenIconMistake)

One of the somewhat odd things about my old fashioned [X Window System](https://en.wikipedia.org/wiki/X_Window_System) environment is that when I 'iconify' or 'minimize' a window, it ([mostly](/~cks/space/blog/sysadmin/HowIUseFvwmIconMan)) winds up as an actual icon on my root window (what in some environments would be called the desktop), [in contrast to the alternate approach where the minimized window is represented in some sort of taskbar](/~cks/space/blog/unix/XIconificationManyWays). I have strong opinions about where some of these icons should go, and [some tools](/~cks/space/blog/programming/ProgrammerLaziness) to automatically arrange this for various windows, including [the GNU Emacs windows I (now) use for reading email](/~cks/space/blog/sysadmin/EmailToolsAffectMyBehavior).

Recently I started a new MH-E GNU Emacs session from home, did some stuff, iconified it, and GNU Emacs disappeared entirely. There were no windows and no icons. I scratched my head, killed the process, started it up again, and the same thing happened all over again. Only when I was starting to go through the startup process a third time did I realize what was going on and the mistake I'd made. You see, **I'd told [my window manager](https://fvwm.org/) to put the GNU Emacs icons off screen** (to the right) and my window manager had faithfully obliged me. Normally I could have recovered from this by moving [my virtual screen](/~cks/space/blog/sysadmin/MyVirtualScreenUsage) over to the right of where it had previously been, but I'd also told my window manager to position the icons for GNU Emacs relative to the current virtual screen, not the one it had been iconified on.

(In [fvwm](https://fvwm.org/) terms, I'd set GNU Emacs to have a 'sticky' icon, which normally means that it stays on your screen as you [move around between virtual screens](/~cks/space/blog/sysadmin/DualDisplayVsMultiDesktop).)

How I could do this starts with how I was setting the icon position for the GNU Emacs I was reading email. Unlike (some) X programs, GNU Emacs doesn't take icon positions as a command line argument (as far as I know), but it does support [setting icon positions through Lisp](https://www.gnu.org/software/emacs/manual/html_node/elisp/Position-Parameters.html). However, I use GNU Emacs on one of our servers to read my email ([with X forwarding](/~cks/space/blog/tech/LatencyImpactMyXExperience)) from both my work desktop and my home desktop, and they have different display configurations; work has two side by side 4K displays, with the GNU Emacs icons on the right display, and at home I have a single display ([and I make more use of multiple virtual screens](/~cks/space/blog/sysadmin/DualDisplayVsMultiDesktop)). Since the icons are positioned at different spots, I have two Lisp functions to set the icon position ('home-icon-position' and 'work-icon-position', more or less).

So that morning, what I did was I started GNU Emacs from my home machine and ran the 'work-icon-position', which told my window manager (via GNU Emacs) that I wanted the icon to have the left position of '5070' pixels. Since I was using a single display that is only 3840 pixels wide, fvwm dutifully carried out my exact instructions and put the icon 1230 pixels or so off to the right of my actual display.

(And then fvwm kept the icon 1230 pixels off the right side when I switched virtual screens, because that's also what I'd told fvwm to do.)

Icons are (little) windows and X is perfectly happy to let you position windows off screen (in any direction, you can put windows at negative coordinates if you want). As you'd expect, a window that is positioned entirely off screen isn't visible. So the actual mechanics of this icon position setting was no problem, and [fvwm](https://fvwm.org/) isn't the kind of program that second-guesses you when you position an icon off screen. So when I positioned the GNU Emacs icons off screen, fvwm put them off screen and they disappeared.

PS: I could have recovered the iconified Emacs in various ways, for example by locating it in various ways and having it deiconify, or explicitly moving its icon back onto the screen. It was just simpler and faster, in my state that morning, to terminate an Emacs I hadn't done much with and try again.