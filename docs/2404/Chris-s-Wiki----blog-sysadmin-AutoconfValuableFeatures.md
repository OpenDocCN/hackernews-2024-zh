<!--yml
category: 未分类
date: 2024-05-27 13:36:55
-->

# Chris's Wiki :: blog/sysadmin/AutoconfValuableFeatures

> 来源：[https://utcc.utoronto.ca/~cks/space/blog/sysadmin/AutoconfValuableFeatures](https://utcc.utoronto.ca/~cks/space/blog/sysadmin/AutoconfValuableFeatures)

In the wake of the [XZ Utils backdoor](https://en.wikipedia.org/wiki/XZ_Utils_backdoor), which involved [GNU Autoconf](https://www.gnu.org/software/autoconf/manual/), it has been popular to suggest that Autoconf should go away. Some of the people suggesting this have also been proposing that the replacement for Autoconf and the '`configure`' scripts it generates be something simpler. As a system administrator who interacts with configure scripts (and autoconf) and who deals with building projects such as [OpenZFS](https://openzfs.org/wiki/Main_Page), it is my view that people proposing simpler replacements may not be seeing the features that people like me find valuable in practice.

(For this I'm setting aside [the (wasteful) cost of replacing Autoconf](/~cks/space/blog/programming/AutoconfNotReplaceable).)

Projects such as [OpenZFS](https://openzfs.org/wiki/Main_Page) and others rely on their configuration system to detect various aspects of the system they're being built on that can't simply be assumed. For OpenZFS, this includes various aspects of the (internal) kernel 'API'; for other projects, such as [conserver](https://en.wikipedia.org/wiki/Conserver), this covers things like whether or not the system has IPMI libraries available. As a system administrator building these projects, I want them to automatically detect all of this rather than forcing me to do it by hand to set build options (or demanding that I install all of the libraries and so on that they might possibly want to use).

As a system administrator, one large thing that I find valuable about configure is that [it doesn't require me to change anything shipped with the software in order to configure the software](/~cks/space/blog/programming/ConfigureNoSourceCodeChanges). I can configure the software using a command line, which means that I can use various means to save and recall that command line, ranging from 'how to build this here' documentation to automated scripts.

Normal configure scripts also let me and other people set the install location for the software. This is a relatively critical feature for programs that may be installed as a Linux distribution package, as a *BSD third party package, by the local system administrator, or by an individual user putting them somewhere in their own home directory, since all four of these typically need different install locations. If a replacement configure system does not accept at least a '--prefix' argument or the equivalent, it becomes much less useful in practice.

Many GNU configure scripts also let the person configuring the software set various options for what features it will include, how it will behave by default, and so on. How much these are used varies significantly between programs (and between people building the program), but some of the time they're critical for selecting defaults and enabling (or disabling) features that not everyone wants. A replacement configure system that doesn't support build options like these is less useful for anyone who wants to build such software with non-standard options, and it may force software to drop build options entirely.

(There are some people who would say that software should not have build options any more than it should have runtime configuration settings, but this is not exactly a popular position.)

This is my list, so other people may well value other features that are supported by Autoconf and configure (for example, the ability to set C compiler flags, or that it's well supported for building RPMs).