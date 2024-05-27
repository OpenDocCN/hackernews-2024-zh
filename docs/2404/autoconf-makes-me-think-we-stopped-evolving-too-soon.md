<!--yml
category: 未分类
date: 2024-05-27 12:55:30
-->

# autoconf makes me think we stopped evolving too soon

> 来源：[https://rachelbythebay.com/w/2024/04/02/autoconf/](https://rachelbythebay.com/w/2024/04/02/autoconf/)

## autoconf makes me think we stopped evolving too soon

I've gotten a few bits of feedback asking for my thoughts and/or reactions to the whole "xz backdoor" thing that happened over the past couple of days. Most of my thoughts on the matter apply to autoconf and friends, and they aren't great.

I don't have to cross paths with those tools too often these days, but there was a point quite a while back when I was constantly building things from source, and a ./configure --with-this --with-that was a given. It was a small joy when the thing let me reuse the old configure invocation so I didn't have to dig up the specifics again.

I got that the whole reason for autoconf's derpy little "recipes" is that you want to know if the system you're on supports X, or can do Y, or exactly what flavor of Z it has, so you can #ifdef around it or whatever. It's not quite as relevant today, but sure, there was once a time when a great many Unix systems existed and they all had their own ways of handling stuff, and no two were the same.

So, okay, fine, at some point it made sense to run programs to empirically determine what was supported on a given system. What I don't understand is why we kept running those stupid little shell snippets and little bits of C code over and over. It's like, okay, we established that this particular system does <library function foobar> with two args, not three. So why the hell are we constantly testing for it over and over?

Why didn't we end up with a situation where it was just a standard thing that had a small number of possible values, and it would just be set for you somewhere? Whoever was responsible for building your system (OS company, distribution packagers, whatever) could leave something in /etc that says "X = flavor 1, Y = flavor 2" and so on down the line.

And, okay, fine, I get that there would have been all kinds of "real OS companies" that wouldn't have wanted to stoop to the level of the dirty free software hippies. Whatever. Those same hippies could have run the tests ONCE per platform/OS combo, put the results into /etc themselves, and then been done with it.

Then instead of testing all of that shit every time we built something from source, we'd just drag in the pre-existing results and go from there. It's not like the results were going to change on us. They were a reflection of the way the kernel, C libraries, APIs and userspace happened to work. Short of that changing, the results wouldn't change either.

But no, we never got to that point, so it's still normal to ship a .tar.gz with an absolute crap-ton of dumb little macro files that run all kinds of inscrutable tests that give you the same answers that they did the last time they ran on your machine or any other machine like yours, and WILL give the same answers going forward.

That means it's totally normal to ship all kinds of really crazy looking stuff, and so when someone noticed that and decided to use that as their mechanism for extracting some badness from a so-called "test file" that was actually laden with their binary code, is it so surprising that it happened? To me, it seems inevitable.

Incidentally, I want to see what happens if people start taking tarballs from various projects and diff them against the source code repos for those same projects. Any file that "appears" in the tarball that's allegedly due to auto[re]conf being run on the project had better match something from the actual trees of autoconf, automake, ranlib, gettext, or whatever else goofy meta-build stuff is being used these days.

```
$ find . -type f | sort | xargs sha1sum
7d963e5f46cd63da3c1216627eeb5a4e74a85cac  ./ax_pthread.m4
c86c8f8a69c07fbec8dd650c6604bf0c9876261f  ./build-to-host.m4
0262f06c4bba101697d4a8cc59ed5b39fbda4928  ./getopt.m4
e1a73a44c8c042581412de4d2e40113407bf4692  ./gettext.m4
090a271a0726eab8d4141ca9eb80d08e86f6c27e  ./host-cpu-c-abi.m4
961411a817303a23b45e0afe5c61f13d4066edea  ./iconv.m4
46e66c1ed3ea982b8d8b8f088781306d14a4aa9d  ./intlmacosx.m4
ad7a6ffb9fa122d0c466d62d590d83bc9f0a6bea  ./lib-ld.m4
7048b7073e98e66e9f82bb588f5d1531f98cd75b  ./lib-link.m4
980c029c581365327072e68ae63831d8c5447f58  ./lib-prefix.m4
d2445b23aaedc3c788eec6037ed5d12bd0619571  ./libtool.m4
421180f15285f3375d6e716bff269af9b8df5c21  ./lt~obsolete.m4
f98bd869d78cc476feee98f91ed334b315032c38  ./ltoptions.m4
530ed09615ee6c7127c0c415e9a0356202dc443e  ./ltsugar.m4
230553a18689fd6b04c39619ae33a7fc23615792  ./ltversion.m4
240f5024dc8158794250cda829c1e80810282200  ./nls.m4
f40e88d124865c81f29f4bcf780512718ef2fcbf  ./po.m4
f157f4f39b64393516e0d5fa7df8671dfbe8c8f2  ./posix-shell.m4
4965f463ea6a379098d14a4d7494301ef454eb21  ./progtest.m4
15610e17ef412131fcff827cf627cf71b5abdb7e  ./tuklib_common.m4
166d134feee1d259c15c0f921708e7f7555f9535  ./tuklib_cpucores.m4
e706675f6049401f29fb322fab61dfae137a2a35  ./tuklib_integer.m4
41f3f1e1543f40f5647336b0feb9d42a451a11ea  ./tuklib_mbstr.m4
b34137205bc9e03f3d5c78ae65ac73e99407196b  ./tuklib_physmem.m4
f1088f0b47e1ec7d6197d21a9557447c8eb47eb9  ./tuklib_progname.m4
86644b5a38de20fb43cc616874daada6e5d6b5bb  ./visibility.m4
$ 

```

... there's no build-to-host.m4 with that sha1sum out there, *except* for the bad one in the xz release. That part was caught... but what about every other auto* blob in every other project out there? Who or what is checking those?

And finally, yes, I'm definitely biased. My own personal build system has a little file that gets installed on a machine based on how the libs and whatnot work on it. That means all of the Macs of a particular version of the OS get the same file. All of the Debian boxes running the same version get the same file, and so on down the line.

I don't keep asking the same questions every time I go to build stuff. That's just madness.