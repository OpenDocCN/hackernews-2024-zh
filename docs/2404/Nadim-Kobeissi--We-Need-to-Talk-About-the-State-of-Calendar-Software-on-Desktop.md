<!--yml
category: 未分类
date: 2024-05-27 13:17:59
-->

# Nadim Kobeissi: We Need to Talk About the State of Calendar Software on Desktop

> 来源：[https://nadim.computer/posts/2024-04-18-calendar.html](https://nadim.computer/posts/2024-04-18-calendar.html)

## We Need to Talk About the State of Calendar Software on Desktop

2024-04-18

Smartphones are fine. There are no problems today with finding good calendar software for any smartphone out there. But when it comes to desktops (or laptops), there are exactly two cases in which using calendars in 2024 isn’t a complete disaster:

1.  You’re using your calendar provider’s web application,
2.  You’re using a Mac.

Any other situation leads you to find yourself drowning in the quicksand of abjectly outdated, unmaintaned, broken calendar software offerings: if you’re on Linux or Windows, and **especially** if you want to link together multiple different calendars from different accounts (Google, Outlook, CalDav, etc.).

This is insane. Calendars are important. We use them every day. How in heck did they end up being so broken on the desktop, *which is where we get work done*, except in two specific case scenarios? No, I’m sorry, Thunderbird’s calendar isn’t a good calendar. It’s dialog boxes from 1998 coming up with that *“donk!”* sound, not being dismissable immediately, and also coming up twice for no reason, or telling you that you modified the event when you didn’t. This isn’t what I’m talking about when I talk about “good calendaring software”.

### How did we end up here?

Easy: it’s an incredibly unfashionable topic *and* a thankless thing to work on. Who wants to write calendar software? Who’s going to pay for calendar software?! Add to that the fact that web apps really are enough for most everyone out there, since they usually only have one calendar to worry about and it’s usually either their work calendar or their personal calendar, both provided either by Google, Outlook or iCloud.

Volunteering to fix calendaring software reminds me of how [Tal Rasha volunteered](https://youtu.be/MKNoy3m5X5Q?si=H7JKYwPSWvU3BIvk&t=95) to chain himself to the Runes of Containment in Lut Gholein so that he could bind Baal’s Soulstone within himself forever. Who the heck wants that as their fate? There are so many better, more well-paying, less onerous and more prestigious things to do with your time. It’s almost as bad as writing a mail client. I don’t even have the courage to write a *blog post* about the need for better mail clients.

But *what if you have multiple calendars from multiple accounts that you need to look at at once,* like most professionals in tech? *What if you just want a goddamn desktop app?*, well, in that case, your only lucky break is to be part of…

### The curious Mac exception

For some reason, macOS has a *thriving* ecosystem of incredibly competitive calendar software for desktops. Aside from Apple’s built-in Calendar app, you have [BusyCal](https://www.busymac.com/busycal/) and [Fantastical](https://flexibits.com/fantastical), to name a couple. They command outrageous pricing and sell lots of copies! BusyCal’s $49.99, while Fantastical is only available by subscription. BusyCal in particular is nuts. Every feature you can think of, every calendaring protocol. It’s world-class calendaring software. A real calendaring Cadillac.

But we do not have multiplatform *or* open source equivalents. What about, you know, *most computers out there*? How do we live in a world where **most workstations don’t have functioning goddamn calendars outside of a web browser?!**

### Something has to be done

My take is that the only realistic, viable solution to break free from this nonsense quagmire where most workstations don’t have good calendars might just be for us to roll up our sleeves and dive into the creation of a new open source calendar project. The most promising approach is almost certainly leveraging [Tauri](https://tauri.app). Why Tauri?

*   Tauri integrates Rust and TypeScript — Rust for performance and security, TypeScript for ease of development and maintenance. There’s a bridge between both.
*   Tauri excels in creating multiplatform applications with remarkably low overhead. This is crucial as one of the core frustrations with existing Electron apps is their bloated nature and sluggish performance.

By building on Tauri, we can target multiple platforms while maintaining a unified codebase, which simplifies updates and bug fixes.

So, here you are, at the tail end of my rant about the dismal state of desktop calendar software. We can continue to grumble into our coffees each morning or we can actually do something about this farce. Seriously, how much longer are we willing to tolerate this nonsense? The blueprint is right there: a sleek, multiplatform calendar app built on Tauri, waiting to be forged from the fires of our collective irritation and genius. If none of this inspires you to action, then maybe continue your Sisyphean struggle with whatever archaic tools you’re suffering through.

Or, if the spirit of benevolence (or frustration) moves you enough, send a grant my way — I’ll take on the herculean task of writing this mythical software myself. I’ll be your Tal Rasha!