<!--yml
category: 未分类
date: 2024-05-27 13:04:32
-->

# Chris's Wiki :: blog/programming/ConfigureNoSourceCodeChanges

> 来源：[https://utcc.utoronto.ca/~cks/space/blog/programming/ConfigureNoSourceCodeChanges](https://utcc.utoronto.ca/~cks/space/blog/programming/ConfigureNoSourceCodeChanges)

Often, programs have build time configuration settings for features they include, paths they use, and so on. Some of the time, people suggest that the way to handle these is not through systems like 'configure' scripts (whether produced by [Autoconf](https://www.gnu.org/software/autoconf/manual/) or some other means) but instead by having people edit their settings into things such as your Makefiles or header files ('source code' in a broad sense). As someone who has spent a bunch of time and effort building other people's software over the years, my strong opinion is that you should not do this.

The core problem of this approach is not that you require people to know the syntax of Makefiles or config.h or whatever in order to configure your software, although that's a problem too. The core problem is **you're having people modify files that you will also change**, for example when you release a new version of your software that has new options that you want people to be able to change or configure. When that happens, you're making every person who upgrades your software deal with merging their settings into your changes. And merging changes is hard and prone to error, especially if people haven't kept good records of what they changed (which they often won't if your configuration instructions are 'edit these files').

One of the painful lessons about maintaining systems that we've learned over the years is that you really don't want to have two people changing the same file, including the software provider and you. This is the core insight behind extremely valuable modern runtime configuration features such as 'drop-in files' (where you add or change things by putting your own files into some directory, instead of everything trying to update a common file). When you tell people to configure your program by editing a header file or a Makefile or indeed any file that you provide, you're shoving them back into this painful past. Every new release, every update they pull from your VCS, it's all going to be a source of pain for them.

A system where people maintain (or can maintain) their build time configurations entirely outside of anything you ship is far easier for people to manage. It doesn't matter exactly how this is implemented and there are mny options for relatively simple systems; you certainly don't need GNU Autoconf or even CMake.

The corollary to this is that if you absolutely insist on having people configure your software by editing files you ship, those files should be immutable by you. You should ship them in some empty state and promise never to change that, so that people building your software can copy their old versions from their old build of your software into your new release (or never get a merge conflict when they pull from your version control system repository). If your build system can't handle even this restriction, then you need to rethink it.