<!--yml
category: 未分类
date: 2024-05-27 14:30:53
-->

# Driverless User Space File Systems for Windows, macOS, and Linux

> 来源：[https://thelig.ht/user-space-file-systems/](https://thelig.ht/user-space-file-systems/)

*TLDR: The [userspacefs](https://pypi.org/project/userspacefs/) Python package allows you to easily a write user space file system for all three major desktop platforms. The [dbxfs](https://pypi.org/project/dbxfs/) Python package now provides a [ready to use executable](https://www.dropbox.com/scl/fo/5bjakuwingc6vk9kcgcy0/h?rlkey=bgab24afqq8rgb78qo9hwsum8&dl=0) for Windows.*

For years I’ve been fascinated with the idea of user space file systems. My first exposure to the concept must have been sometime in 2002 when I learned about GNU Hurd. There are of course many benefits to being able to run file systems in user space but to the budding system designer within me its strongest appeal was as a natural, logical, and elegant extension of the file system based operating system paradigm as we know it.

GNU Hurd was just the tip of the iceberg. GNU Hurd was still mostly Unix with user space file systems as an added feature, whereas Plan 9 was Unix rearchitected around the concept as a core primitive. The end result was an even less complex system with a unified API across the entire stack. Using the file system as the core system API, we can go a step further and implement network protocols, graphics interfaces, and other hardware drivers as user space servers instead of some ad-hoc combination of custom kernel and user space components. Today we know this as the [narrow waist](http://www.oilshell.org/blog/2022/02/diagrams.html) design principle.

This idea captured the imagination of a generation of programmers and system designers. Naturally, the idea spread and native user space file system functionality eventually surfaced in Linux and the BSDs in the form of FUSE. This has enabled [dozens of critical applications](https://en.wikipedia.org/wiki/Filesystem_in_Userspace#Applications) that simply weren’t practical prior to the introduction of FUSE.

Outside of the open source POSIX systems, there have been efforts to bring a FUSE-like API to Windows and macOS but these have always required the installation of a third-party kernel driver. A dependency on a kernel driver makes deploying a user space file system more complex, often prohibitively so. It turns out that no driver is necessary.

## WebDAV

Sometime in 2011, while casually browsing the Mac OS X man pages, I noticed that it had the ability to mount WebDAV servers as local file systems. I had known this for years already and it didn’t really register as anything interesting to me except that there was a command line interface to do it. After a few days it hit me, “You can implement a user space file system this way!” The idea is simple, just host a WebDAV server on localhost and run a command to mount it. The idea was an interesting hack but I had an idea for an even more interesting hack: write a library that emulated the FUSE ABI but used WebDAV on the backend to transparently mount it on Mac OS X. I said ABI because I wouldn’t just do this at the source level, I would use `DYLD_INSERT_LIBRARIES` to trick pre-compiled programs to use my library instead. Work on davfuse started on July 17th, 2012.

I spent nights and weekends over the next year implementing the idea. When I was finally done, it just worked. I was ecstatic. davfuse was [released on September 11th, 2013](https://news.ycombinator.com/item?id=6369172), unfortunately to little if any fanfare. I didn’t really put much effort into publicizing the library because davfuse was intended to just be a proof of concept. In the time that I started working on davfuse, I learned that Windows also natively supported mounting WebDAV file systems. Suddenly the vast majority of the desktop computing userbase could experience user space file systems, making the technique very powerful. My goal then shifted to producing a killer application for Windows using this technique.

## Safe

I was an avid user of [EncFS](https://vgough.github.io/encfs/) during this time. It was a convenient way for me to store my encrypted my files on my Dropbox. So the idea was simple, create a user-friendly version of EncFS that ran on Windows and Mac OS X. Work started on July 26th 2013 and Safe was [released on April 14th 2014](https://news.ycombinator.com/item?id=7588369).

When you’re building something, you want to [focus on one innovative thing](https://boringtechnology.club/) so you don’t get distracted and can release something on time. In this case, I focused on the wrong thing. For various reasons, even though it worked well enough, EncFS was not appropriate for long term for use on Windows. For delivering this type of product, the one thing I should have focused on instead was designing a trustworthy encryption system that worked seamlessly on both Windows and Mac OS X. In other words, building Safe well wasn’t as simple as wrapping EncFS in a user-friendly way. In fact, EncFS ended up being more of a ball-and-chain than a boon for my productivity because I promised compatibility with EncFS. That limited my ability to improve the encryption. Safe didn’t really have any users and I didn’t want to deal with supporting something that was already deprecated so I just shut it down.

Even though the code of davfuse and Safe have become useless, I learned a lot about software engineering and product development during my time building them.

## SMB and dbxfs

While WebDAV works reasonably well it’s not as efficient as it could be for this use case. While implementing davfuse, a coworker asked me why I decided to use WebDAV instead of SMB. I was aware of SMB but I had assumed it was a complex and esoteric network technology, so I didn’t even consider using it at first. He assured me it was a simple binary protocol and more efficient than WebDAV. I kept this idea in the back of my mind.

Sometime during 2015 I transitioned to using an [ARM Linux desktop](https://en.wikipedia.org/wiki/Novena_(computing_platform)). This forced me to look for a Dropbox alternative. One thing I never liked about the official Dropbox application was that it was based on synchronization. I didn’t really need sync functionality for my desktop because it was always online and my network connection was fast so I didn’t mind downloading files on the fly. I also didn’t think it would be too difficult to build an on-the-fly Dropbox file system. All of these factors led me to try to build one. Except this time I would write it in Python (for development productivity) and I would use SMB instead of WebDAV (this would ultimately be a mistake). Work on dbxfs started on August 22nd, 2015.

I structured dbxfs in two parts. The generic user space file system functionality was contained in a Python package called [userspacefs](https://pypi.org/project/userspacefs/) whereas the Dropbox-specific functionality was contained in a package called [dbxfs](https://pypi.org/project/userspacefs/). dbxfs would be the first consumer of the userspacefs package. I did this just in case others wanted to make use of the user space file system functionality that dbxfs used (I don’t think anyone ever did.)

dbxfs was publicly [released on October 3rd, 2018](https://news.ycombinator.com/item?id=18133450). The reason it took so long was that I was just using it privately and didn’t really want to deal with having to support other people using it. It was always intended to be a tool for hackers, in a [suckless](https://www.suckless.org/) type of way. Like I mentioned, it was a passive side project just meant to support me while I worked on other things. It initially released with support for Linux and macOS, the only platforms I used at the time. Two days later, on October 5th 2018, I added OpenBSD support, which I used sometimes. I initially planned to add Windows support shortly after but I didn’t use Windows and no one ever requested it, so I didn’t do it.

## Windows Support

Fast-forward to the second half of 2023\. I had some down time due to various life changes and I decided to do some cleanup on dbxfs. This time around, I decided to make sure the entire userspacefs package passed under mypy’s strict type checking. This required some refactoring of the SMB server code. To make sure the SMB server code still worked while refactoring, I pointed my Windows VM at it. This revealed an issue about which I had long forgotten: Windows doesn’t allow you to mount SMB servers on arbitrary ports. This means that you can’t run a user space SMB server because you would have to bind to port 445 which is only allowed by administrator users. Another issue was that recent versions of Windows by default had disabled support for SMBv1\. There are ways to get around both of these issues with extra support code and UAC prompts but that would significantly reduce the practicality of the userspacefs approach.

So that killed any passive hopes I had in the back of my mind about easily getting this to work on Windows. This really bugged me. It was like a loose end in the back of my mind became even looser and I just didn’t like that. It also was a let down that SMB didn’t end up being the viable efficient alternative to WebDAV for which I had hoped. In any case, I felt like I had to resolve this story. I knew that WebDAV worked on Windows from when I worked on davfuse and Safe, so I set about to port my davfuse WebDAV server code to Python with the ultimate goal of fully porting dbxfs to Windows. This was late October 2023.

The lion’s share of the work was completed right at the end of 2023. Along the way I cleaned up the userspacefs API so that there was no impedance mismatch with Python’s os module. I.e. you could implement the [userspacefs file system API with Python’s os module with nearly zero glue code](https://thelig.ht/code/userspacefs/file/userspacefs/stdlibfs.py.html). I also fully [documented the file system API as code](https://thelig.ht/code/userspacefs/file/userspacefs/abc.py.html) which helps a lot for typing with mypy. I also added a basic GUI for dbxfs so that Windows users don’t have to use the command line. I’m pleased with how everything turned out.

## Big Unfortunate Caveat

While I was in the middle of adding the Windows support, it turns out that [Microsoft has suddenly decided to deprecate WebDAV support](https://learn.microsoft.com/en-us/windows/whats-new/deprecated-features). To get around this, userspacefs will show a UAC prompt to re-enable the service if it is disabled. There isn’t much to be said other than Windows really should allow mounting SMB on any port. If any Microsoft engineers are reading this, please get in touch.

*Edit 2024-01-05: A friend has informed me that Windows now provides an explicit API for user space file systems called [Projfs](https://learn.microsoft.com/en-us/windows/win32/projfs/projected-file-system). It looks promising.*

*Edit 2024-01-05: [marwis on Hacker News](https://news.ycombinator.com/item?id=38887069) has commented that recent versions of Windows 11 allow [mounting SMB on alternative ports](https://techcommunity.microsoft.com/t5/storage-at-microsoft/smb-alternative-ports-now-supported-in-windows-insiders/ba-p/3974509). This is great news.*

## The Future

Now that [userspacefs](https://pypi.org/project/userspacefs/) works on all major desktop platforms and is fully type checked, I plan to eventually get it to compile with mypyc. This goes hand-in-hand with adding full strict type-checking to dbxfs as well. This code is over 8 years old now and I’m not fond of rewrites. I don’t want to have to rewrite this code in C++ just for efficiency’s sake. Again this is still just a passive side project :)

I think userspacefs is in a good place right now and is broadly useful. Now that it supports Windows, this opens up a massive user base to the world of user space file systems. If you have questions on how to use it feel free to get in touch: [@cejetvole](https://twitter.com/cejetvole).

Rian Hunter
2024-01-05