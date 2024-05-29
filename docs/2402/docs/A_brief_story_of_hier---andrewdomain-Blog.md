<!--yml
category: 未分类
date: 2024-05-27 14:41:32
-->

# A_brief_story_of_hier - andrewdomain Blog

> 来源：[https://andrewdomain.com/posts/a_brief_story_of_hier/](https://andrewdomain.com/posts/a_brief_story_of_hier/)

Recently Fedora opened [this curious Change proposal](https://fedoraproject.org/wiki/Changes/Unify_bin_and_sbin) about the unification of the **/bin** and **/sbin** paths.

The news was welcomed by the community with short oneline sentences, that didn’t help me understand the connotations of that proposal so I thought about using the occasion to do some little research to make light on this matter…

What I found out is that Fedora was not the first distro to implement this choice, but was one of the first ones to start this 10+ years old discussion about reforming the filesystem hierarchy under Linux, which now is reaching a more advanced state.

For several reasons and necessities, the structure of the filesystems that host the binaries and libraries had changed on all the major GNU/Linux distributions, after long and painful internal work, to provide a much cleaner structure and easier administration, with the possible side effect of leaving behind the needs of some users, that will probably migrate towards other systems.

> NOTE for the reader:
> When I’m talking about “the directories”, or “those directories”, or.. you got it. I’m probably referring to **bin**, **sbin**, and **lib**, no matter if placed under **/** or under **/usr**.

## compliance

I think I should start by making it very clear that a Linux-based system is an animal of its own kind. Linux distributions are not conventional UNIX(SUS-compliant) or POSIX or whatever kind of systems. **They’re just Linux systems**.

Linux systems pick only the best/most fitted aspects of each specification and incorporate it, while not being 100% compliant with the whole.. maybe sometimes compliance is optionally added etc… But as a general rule, each Linux distro could very easily be very different from the others.

The whole ecosystem of Linux distros is kept together thanks to **some conventions**, that can be more or less technically enforced, which become standards. So that in the meantime there is plenty of room for each one to express their own idea on how the system should work.

### man 7 hier

One example of an almost-standardized aspect of Linux distros that kinda survived over the years, is the **FHS (Filesystem Hierarchy Standard)** that is maintained by the Linux Foundation.

The FHS describes the purpose of each path in your filesystem, that your distro’s maintainers have probably followed, and that won’t probably hurt the responsible sysadmin to read.

**man 7 hier** is your local, system-specific (major version, …) reference for the version of the FHS specification that your system supports. Check out the **STANDARDS** section of the manpage to see exactly which version your system is supporting (on this day on Gentoo it’s the 3.0 from March 2015), and while you’re at it, check also the **BUGS** section to remind yourself that yes.. it is just a convention that is not really technically enforced.
Because theoretically, you could put your binaries in /etc, mount /home as a tmpfs, or cause whatever degree of mayhem that you wish.

We’re still (mainly) under GNU/Linux after all, where historically there has been an effort to guarantee to the user the maximum possible degree of freedom.

## /usr

For the rest of this article, we will see some vicissitudes around the bin, sbin, and lib* directories over the years.

To do so, I will start from this chunky first table with descriptions that I’ve copy/pasted from the (**rather traditional**) hier manpage itself, with the purpose of showing how those definitions change later on..

| PATH | descr |
| --- | --- |
| /bin | This directory contains executable programs which are needed in single user mode and to bring the system up or repair it. |
| /sbin | Like /bin, this directory holds commands needed to boot the system, but which are usually not executed by normal users. |
| /lib* | This directories should hold those shared libraries that are necessary to boot the system and to run the commands in the root filesystem. (EDIT: In this description we’re gonna also include the architecture-specific variants of /lib (/lib32, /lib64, …) that serve the same purpose) |
| /usr | This directory is usually mounted from a separate partition. It should hold only shareable, read-only data, so that it can be mounted by various machines running Linux. |
| /usr/bin | This is the primary directory for executable programs. Most programs executed by normal users which are not needed for booting or for repairing the system and which are not installed locally should be placed in this directory. |
| /usr/sbin | This directory contains program binaries for system administration which are not essential for the boot process, for mounting /usr, or for system repair. |
| /usr/lib* | Object libraries, including dynamic libraries, plus some executables which usually are not invoked directly. More complicated programs may have whole subdirectories there. (EDIT: same as before for the architecture-dependant.) |

The natural question here I think is: “Why do we even have this distinction between those directories”. At first glance, it seems like unnecessary overcomplication, after all, we’re talking about the same entities.
And also: “Why is this filesystem **usually mounted from a separate partition**??”.

Let’s start by understanding better the purpose of the /usr filesystem, and equally importantly, where it even comes from.

### split-usr

There is a very interesting mail that people close to where the changes are made keep quoting as a reference to this phenomenon, which provides historical context.
I’m talking about [this mail](http://lists.busybox.net/pipermail/busybox/2010-December/074114.html) from the busybox ml.

The argument is pretty clear: there were technical reasons that made this distinction necessary in the first place.

> ```
> When the operating system grew too big to fit on the first RK05 disk pack (their 
> root filesystem) they let it leak into the second one, which is where all the 
> user home directories lived (which is why the mount was called /usr).  They 
> replicated all the OS directories under there (/bin, /sbin, /lib, /tmp...) 
> ```

The same reasons that made us split the /usr (or split the /, depending on where you see it from), and adopt /bin and /sbin, with their relative dependencies located under /lib*, with the sole purpose of ensuring the mount of the partition where /usr was, at boot time, i.e. **to bring the system from single-user mode, to multi-user mode** (also check this [sysvinit to systemd translation](https://fedoraproject.org/wiki/SysVinit_to_Systemd_Cheatsheet#Runlevels/targets)) .

> ```
> Of course they made rules about "when the system first boots, it has to come up 
> enough to be able to mount the second disk on /usr, so don't put things like 
> the mount command /usr/bin or we'll have a chicken and egg problem bringing 
> the system up."
> ...
> The /bin vs /usr/bin split (and all the others) is an artifact of this, a 
> 1970's implementation detail 
> ```

You see that here is referred to as **the /bin vs /usr/bin split**, which we may call by the name of **split-usr**, but it would not be entirely correct.

The term **split-usr** as used today, has a different meaning than the one used in this article and it probably has to do with the necessity of providing more distinction in the possible structure of a system in regard to its use of /usr (check [glossary](#glossary)). It has to be said that this mail also comes from more than 10 years ago, while the pains of the upgrade, reorganizations, implementations,… are much more recent.

==**Why not physically place everything under /, instead of /usr**==
Other distributions that detached from a standard filesystem specification like [GoboLinux](https://gobolinux.org/at_a_glance.html) and others, do not have a /usr partition that contains the “os” and seem to not miss it.
By following the /usr pattern, as most Linuxes have historically done, we’ve taken the same road as other UNIXes had, like SunOS that in 4.0.3 started adopting a readonly /usr filesystem, and Oracle Solaris, that from version 11 started symlinking for compatibility purposes, the relative directories under / to their counterparts in /usr, where the actual system is.

The fact that we have “the operating system”, i.e. the static part of it composed of all the binaries and libraries, that shouldn’t vary between hosts with the same version, under a certain filesystem different from root, opens the doors to interesting scenarios like:

*   a simple readonly mount of /usr for security reasons.
*   the easier backup of /usr before upgrades.
*   the share of /usr between hosts (e.g. using a network share).
*   centralized update of this static part, by sharing it.

Following those logics, We could host some tiny Linux boxes that require very few resources, by having a rw / with all the (this is a recurring expression) **site specific** files and configurations and sharing a common /usr between all of them.

==**do we need a split-usr configuration to accomplish this?**==
Good question, from the article above the answer is pretty clear. The message of it is that we clearly maintained unnecessary complexity for decades, only because “if it works, don’t touch it!”. For most scenarios, this mechanism is completely unnecessary and for many people working on Linux, this possibility is also unknown. From my personal experience, you can easily see huge virtualization-based infrastructures, where Linux boxes have a really simple filesystem separation: /var/log separated is a good practice that not everybody follows, but not go as far as mounting /usr ro.

This is often not enough to make a change of that magnitude, the condition of “if it works, don’t touch it!” must fail, and it kinda of does.

The article that goes by the title [Booting without /usr is broken](https://www.freedesktop.org/wiki/Software/systemd/separate-usr-is-broken/) does a good job of explaining one of the reasons why the **split-usr mechanism** is mostly defective on modern distros.

> ```
> Quite a number of programs these days hook themselves 
> into the early boot process at various stages. 
> A popular way to do this is for example via udev rules. 
> The binaries called from these rules are sometimes located on /usr/bin, 
> or link against libraries in /usr/lib, or use data files from /usr/share. 
> If these rules fail udev will proceed with the next one, 
> however later on applications will then not properly detect
> these udev devices or features of these devices. 
> ```

Another very important reason that is recurrently mentioned in the mlists, and briefly outlined in this other article on freedesktop.org, which goes by the title of [The case for the /usr merge](https://www.freedesktop.org/wiki/Software/systemd/TheCaseForTheUsrMerge/), is that software nowadays is too complex.

If software A is mainly built with feature A in mind and depends on library A, based on the consensus of the community, gets packaged with support for feature X, with the relative dependency on library X, then it is the maintainer’s burden (if aware of the distinction) to engage with the rest of the community to find a way to put library X, that was previously situated on /usr/lib, into /lib, as a vital dependency of software A, that could be used to bring up /usr during the early boot stages (Check dependencies for the software on your machine with `ldd $(which <BINARY_NAME>)` to have a clearer idea about this).
Moreover, the developer of the upstream software (which is not related to your distro’s organization), is not aware of the Linux-specific /usr-split mechanism, and for that matter, the maintainer’s job (the one who is aware of the /usr-split) gets more complicated; it has also been said on various mailing lists, that testing such mechanism (/usr-split) is rather complicated.

In conclusion of this topic, those mentioned above have proven to be fairly common issues in all distros, and with all init systems ([ref](https://www.mail-archive.com/debian-devel@lists.debian.org/msg337190.html)), we could say that the **split-usr mechanism** has not been well maintained throughout the decades.

It could be the case to look for a better solution.

The dilemma is: either we choose to repackage most distros in the classic early-days-UNIX way to repair split-usr, or we accept the current state of things:

*   binaries are too complex
*   boot-time service startup dependencies issues
*   upstream probably doesn’t care about our concerns

and from now on we unanimously treat distributions in the same way most users have been treating them before, effectively making the whole system much easier to maintain and administer.

### merged-usr

The conversation starting from [this reply](https://lwn.net/Articles/477541/) for the **“The case for the /usr merge”** article on lwn.net, shows some very interesting points about why the initrd can be used as a more reliable, easier to maintain, and generally modern solution, as opposed to the split-usr mechanism to bring the system up.

At this point, one could also think that if we decide to use the initrd for this purpose, maintaining the **split-usr mechanism** itself is a duplication of effort.

So the proposed direction is clear: to move **everything**, i.e. “the directories” that leaked out of /usr (see note at the top), inside /usr again, effectively having **“the whole os”** on that filesystem. The difference here, with the previous split setup, is that **if** /usr is not on the same partition as /, the mount has to be taken care of by the initrd, instead of by the directories in /.

As freedesktop.org’s articles and some other posts pointed out, the adoption of an operating system **entirely** placed under /usr, arguably favors the easier maintenance of those use case shown above, like the shareable /usr. The only difference is that the mount process has to happen inside the initramfs and be transparent for the system during the boot phase, to avoid falling into those chicken-and-egg and dependency issues.

> **FROM** : mounting the operating system during boot using the tools found in /bin and /sbin, i.e. relying on the split-usr mechanism (and how well it has been maintained).
> 
> **TO** : mounting the operating system before the actual boot, relying on the initrd and the content it has been packaged with.
> 
> When I say boot, I mean the boot phase as seen by the init (..whatever init), which is different than the expression “the boot process”. The boot process is complex, and from the point at which the bootloader loads the kernel and the initrd, after some kernel work, the initrd takes control (there is a /sbin/init or /usr/sbin/init or whatever, inside the initrd that gets executed) and tries to accomplish its tasks. Only after the initrd decides so, control is given to the actual init daemon and to the final system, and **it is this phase** that has issues to work with the split-usr mechanism (check [glossary](#glossary) again) for the issues discussed above.

So systems can still have /usr on a separate partition, the only difference is that **in that case**, the initrd is **the only supported way of bringing it up** (check [this](https://forums.gentoo.org/viewtopic-p-8591784.html?sid=6b728f7132cbd104e809c348de60b9c1#8591784) explanation).

The initrd solution is however not perfect and 100% victimless. There have been complaints about those use cases that have to do with resource-contstrained environments. I’m talking about the embedded world ([ref](https://lwn.net/Articles/670306/)): where not having to maintain an initrd is a huge benefit.

> ```
> I have a system that boots in three seconds, which is fairly long
> already. Adding an initrd would extend that to five seconds and require
> a twenty minute rebuild of the initrd on upgrade.
> 
> Using an initrd would also add new failure modes where the initrd build
> fails because of memory constraints (e.g. because an application process
> is stuck in D state and did not terminate when we entered the "system
> update" runlevel). 
> ```

The possible first wave of users leaving the community.

Also, [this here](https://lwn.net/Articles/670324/) is a very well-written collection of concerns about the change to merge-usr in Debian, being it a non-subscription-based supported distribution of GNU/Linux.

> ```
> I, for example, am afraid of having to merge /usr in existing systems
> during upgrades, causing repartitions to be necessary. I am afraid of
> partition layout suddenly not fitting any more during an upgrade,
> causing downtimes and customers considering to take the opportunity to
> migrate to a really supported enterprise distribution.
> 
> And, I really don't want to have to adapt, test and verify scripts and
> backup schemes to changed partition layout. This will be necessary for
> new systems, and it is really a horror vision to have to do this for
> existing systems during upgrades. 
> ```

Other possible losses for the community.

That I think is a valid point for those scenarios where we’re updating/upgrading our operating systems in-place, instead of migrating the services deployed on them.
During the years, many poor-quality scripts could have been built by colleagues that maybe are not even part of the company anymore, not to talk about the upgrade process itself, where a lot of things could go wrong, depending on the degree of care that has been taken of that os, by the different people that have worked on it.
That is a scenario where a handful of administrators can easily not be able to face an important number of systems in such a situation.

…

The /usr-merge has already happened in probably all the major distros, with the relative distro-maintenance-pain and sysadmin-upgrade-pain. From this point on, so as not to leave things unfinished, it would be much easier to introduce new changes in order to apply further simplifications.

## bin & sbin

During the same years of the first discussions about the /usr-merge, there were also [discussions](https://lists.fedoraproject.org/pipermail/devel/2011-October/158845.html) about the purpose of the bin and sbin directories and their current state.

In the ml post linked above, a couple of very interesting and enlightening points have been made, some similar to the ones for the /usr-merge:

> ```
> a) the split between sbin and bin requires psychic powers from
>    upstream developers:
> 
>    The simple fact is that moving binaries between these dirs is really
>    hard, and thus developers in theory would need to know at the time
>    they first introduce a binary whether it *ever* might ever make sense to
>    invoke it as unprivileged user (because in that case the binary
>    belongs in /bin, not /sbin). History shows that more often than
>    not developers have placed their stuff in the wrong directory, see
>    /sbin/arp, /sbin/ifconfig, /usr/sbin/httpd and then there is no smart
>    way to fix things anymore since changing paths means breaking all hardcoded
>    scripts. And hardcoding paths in scripts is actually something we
>    encourage due to perfomance reasons. The net effect is that many
>    upstream developers unconditionally place their stuff in bin, and
>    never consider sbin at all which undermines the purpose of sbin
>    entirely (i.e. in systemd we do not stick a single binary in sbin,
>    since we cannot be sure for any of its tools that it will never
>    ever be useful for non-root users. and systemd is as low-level,
>    system-specific it can get).
> ...
> f) splitting things up complicates stuff. If you want to keep things
>    separate you really need a good reason for that. We should always
>    focus on simplifiying things. And merging things into /usr does just
>    that: it drastically simplifies the complexities we have collected
>    over 30+ years of Unix heritage.
> 
> g) given that some distros place certain binaries in other places than
>    others, merging the dirs has the big benefit that the four paths are
>    equivalent and scripts from other distros work on ours, regardless
>    where the other distros place their stuff 
> ```

Those above are the recurring “binaries getting too complex”, or “we’ve been using it like this as a majority, for a lot of time” as before. But for this specific matter, there is also more.

The “bin vs. sbin separation” is another mechanism that has probably not been maintained well throughout the decades. In modern distributions, you can easily find `PATH=/bin:/sbin:/usr/sbin:/usr/local/bin:/opt/bin:/everything` for all the users, which is not exactly a good practice as per the intended bin/sbin distinction..

As claimed by Fedora’s [Changes/Unify bin and sbin](https://fedoraproject.org/wiki/Changes/Unify_bin_and_sbin), there are many mechanisms for a normal user to gain privileges without them even knowing, and then its PATH already contains all the sbin variants for reasons like systemd’s behavior of setting a PATH for all users and services that has both bins and sbins. This specific mechanism of systemd is quite intricate and rather than trying to find its functioning in systemd itself, it could be better looked at in an application-specific way. The `man 5 environment.d` on your system (or also [here](https://www.freedesktop.org/software/systemd/man/latest/environment.d.html)) in the **APPLICABILITY** section, outlines the inner workings of this mechanism:

> ```
>  Environment variables exported by the user service manager are passed 
>        to any services started by that service manager. 
>        In particular, this may include services which run user shells. 
>        For example in the GNOME environment, the graphical terminal
>        emulator runs as the gnome-terminal-server.service user unit,
>        which in turn runs the user shell,  so that shell
>        will inherit environment variables exported by the user manager.
>        For other instances of the shell, not 
>        launched by the user service manager, the environment they
>        inherit is defined by the program that starts them. 
>        ...
>        Specifically, for ssh logins, the sshd(8) service builds
>        an environment that is a combination of variables forwarded from
>        the remote system and defined by sshd...
>        A graphical display session will have an analogous mechanism 
>        to define the environment. 
>        Note that some managers query the systemd user instance
>        for the exported environment and inject this configuration
>        into programs they start, 
>        using systemctl show-environment or the underlying D-Bus call. 
> ```

In my case for example, I do have a PATH variable defined inside /etc/environment.d/10-gentoo-env.conf (that has been put there by gentoo’s env-update) which looks similar to `echo PATH=$PATH`. But then I also have it in /etc/profile.env, and also distros like fedora and derivates (rocky Linux, …) seem to have this logic entirely contained in /etc/profile.

```
# Path manipulation
if [ "$EUID" = "0" ]; then
    pathmunge /usr/sbin
    pathmunge /usr/local/sbin
else
    pathmunge /usr/local/sbin after
    pathmunge /usr/sbin after
fi 
```

==**why having PATH=everything is bad?**==
Knowing the bin and sbin distinction, this can be seen as a bad practice. The community of [#gentoo-chat](https://www.gentoo.org/get-involved/irc-channels/) has explained it like this to me (not the exact words):

```
 You don't want the normal user to have /sbin in its PATH.  
 You maybe want only the root user to have /sbin in PATH.  

 Nowadays is also very easy and quick to gain root privileges with sudo,
 so at least normal users should be forced to write /sbin/something in full,
 so that they also remember to think about what they're typing..
 Because it would make sense for everything that is under /sbin, 
 to be dangerous to the system if misused.

 So...  
 normal users having no /sbin in PATH, and
 discouraging interactive root sessions with (e.g.) `sudo -i`. 
```

So ideally there should be a neat distinction between all the binaries that only root can execute, and those that are executable by all the users, but can we even do that given nowadays software?

> The categories of executables these days go beyond the split of “thing a user can run” and “thing the admin can run”. [ref](https://pagure.io/fesco/issue/3135#comment-890527)

Personally, I’d like to think more that [this proposal](https://discussion.fedoraproject.org/t/f40-change-proposal-unify-usr-bin-and-usr-sbin-system-wide/99853/16) is the way to go; that is a more romantic way of seeing Linux. But then [the reply to that](https://discussion.fedoraproject.org/t/f40-change-proposal-unify-usr-bin-and-usr-sbin-system-wide/99853/18), is an amazingly well-painted portrait, of the world we’re living in. I’ll probably take this post as a future reference when I’ll have to explain “things that are happening in Linux”.

> ```
> In the old times there was “root” and lusers, 
> the privilege separation was completely binary. 
> But some time in the 2.2 era, Linux implemented capabilities. 
> This means that you can be root, completely unprivileged, 
> or in one of the 1099511627774 intermediate states.
> ...
> The interesting corollary is that all programs that
> are supposed to “run as root” must always treat 
> every individual operation as something 
> that can fail (because even if the process
> formally has privileges, MAC or something else 
> may prevent the operation) and must report such failure 
> in sufficient detail. It is not OK to say “we failed” or
> “not enough privileges, must be root”, because that is not
> enough to figure out what exactly failed. 
> In fact, the process may be root and still fail. 
> ```

The second paragraph is also a reference that one should come back to when their software is not behaving the way they thought it would.

I could argue no more about the unification.

### my personal view about this

For how small those changes seem from the outside, they proved to be big, and their reasons and implications are numerous.

With the initrd solution to the split-usr issue, an attempt has been made to modernize a workflow that is generally less and less seen in the wild, to not lose the creative part of the community, while maintaining the bigger part happy.

Replicating and maintaining artistic configurations such as shared ro /usr via NFS over an entire Linux infrastructure, requires inventiveness, effort in testing and maintenance, biting your tongue when the manager buys from a bad vendor,… and generally, time, which is an extremely pricy asset. One thing that I couldn’t stop thinking is that by “playing” with your os that way, you are treating your Linux as something that barely exists anymore in the majority of cases.

Nowadays, especially in the enterprise community, you are much more used to virtual machines than baremetal, and more and more used to containers than virtual machines, and generally more used to treating a Linux system as something **atomic**, and as **disposable** as possible.

What happened is that today we don’t really have technical constraints like disk size; on the contrary, today GBs of disk are quite cheap, compared to the cost in time related to the maintenance of an infrastructure as complex as the ones described above.

You usually don’t want to maintain, because that is where the effort comes (and the man-months go).
What you **usually** want to do instead, is maintain only the least possible, vital, information on a git server, and deploy your stuff from there using some kind of automation.
**When it breaks, you usually toss it, you don’t try to repair it**.
To do that, it is required to deal with many different technologies, a task that is much easier to accomplish if you are part of a team with a lot of people, each with its own specialization. In that scenario, at the end of the day, you will find yourself with a lot more machines in your infrastructure, that have a cost in resources, but are a lot easier to administer.

That is a very DevOps way of seeing it I think, which apparently is not exactly aligned to the UNIX way.

## end

In the end, if you’ve been thrown into a world where everything works differently than the way you’re used to seeing them work, and you’re expertise is abandoning you, I think it can be advantageous to be aware of freedesktop’s [efforts](https://www.freedesktop.org/wiki/Specifications/) to maintain compatibility between systems.

Speaking of filesystem hierarchy specifically, I think it can help to give an occasional look when needed, to [this manpage](https://www.freedesktop.org/software/systemd/man/file-hierarchy.html), that you may find in your distro as **man 7 file-hierarchy**.

This is the systemd-specific vision of the filesystem hierarchy under Linux, which is the defacto standard on systemd-based Linux distros and effectively changes the first table at the top of this article like so:

| PATH | descr |
| --- | --- |
| /usr | Vendor-supplied operating system resources. Usually read-only, but this is not required. Possibly shared between multiple hosts. This directory should not be modified by the administrator, except when installing or removing vendor-supplied packages. |
| /usr/bin | Binaries and executables for user commands that shall appear in the $PATH search path. It is recommended not to place binaries in this directory that are not useful for invocation from a shell (such as daemon binaries); these should be placed in a subdirectory of /usr/lib/ instead. |
| /usr/lib | Static, private vendor data that is compatible with all architectures (though not necessarily architecture-independent). Note that this includes internal executables or other binaries that are not regularly invoked from a shell. Such binaries may be for any architecture supported by the system. Do not place public libraries in this directory, use $libdir (see below), instead. |
| /usr/lib/arch-id | Location for placing dynamic libraries into, also called $libdir. Legacy locations of $libdir are /usr/lib/, /usr/lib64/. This directory should not be used for package-specific data, unless this data is architecture-dependent, too. |
| /usr/sbin | A compatibility symlink pointing to /usr/bin/, ensuring that scripts and binaries referencing these legacy paths correctly find their binaries. |
| /bin | A compatibility symlink pointing to /usr/bin/, ensuring that scripts and binaries referencing these legacy paths correctly find their binaries. |
| /sbin | A compatibility symlink pointing to /usr/bin/, ensuring that scripts and binaries referencing these legacy paths correctly find their binaries. |
| /lib | A compatibility symlink pointing to /usr/lib/, ensuring that scripts and binaries referencing these legacy paths correctly find their binaries. |
| /lib64 | On some architecture ABIs, this compatibility symlink points to $libdir, ensuring that binaries referencing this legacy path correctly find their dynamic loader. This symlink only exists on architectures whose ABI places the dynamic loader in this path. |

# Glossary

**unmerged-usr** (sometimes also referred to as **split-usr**, see ref 3 below)

A system with bin, sbin, lib* directories, that exist under the / and under the /usr filesystem in parallel, and with distinct content.

> The intended use for this is the use of the bin,sbin,lib* directories under /, to mount the rest of the filesystem, including /usr where most of the binaries/services are (a.k.a. most of the os).
> i.e. / can be used in single-user mode, to bring the system to multi-user mode.

**merged-usr**

The bin, sbin, and lib* directories are physically located under the /usr filesystem. They have correspondents under /, which are symlinks provided for compatibility.

**split-usr**

A system whose /usr directory is still unpopulated or needs to be mounted during the boot phase. This could also mean a merged-usr system, whose /usr partition was not mounted before init kicks in and the processes start.

> The /usr filesystem does not have to be on the same partition as /,
> but **it will have** to be mounted during early boot by the initramfs,
> so that when the actual init starts, /usr is ready. ([check](https://lwn.net/Articles/670139/))

REFERENCES:
[1] [https://lists.freedesktop.org/archives/systemd-devel/2022-April/047673.html](https://lists.freedesktop.org/archives/systemd-devel/2022-April/047673.html)
[2] [https://lists.freedesktop.org/archives/systemd-devel/2023-February/048831.html](https://lists.freedesktop.org/archives/systemd-devel/2023-February/048831.html)
[3] [http://lists.busybox.net/pipermail/busybox/2010-December/074114.html](http://lists.busybox.net/pipermail/busybox/2010-December/074114.html)

# FAQ - Can I boot a systemd-based Linux distribution without an initrd?

This one is actually a real thing that I heard :) (not in question form tho).

As stated by different articles and posts, the changes about the /usr filesystem are orthogonal to systemd.

> Look for harald in [this post](https://lists.fedoraproject.org/pipermail/devel/2011-October/158594.html).
> [tl;dr](https://lists.fedoraproject.org/pipermail/devel/2011-October/158818.html)

The answer: There is no need to have an initramfs if you don’t use a separate /usr, i.e. if you don’t need to mount /usr at boot time.

# References

Maybe there are things that I wasn’t able to catch, or I didn’t immediately see the implications. Also, interesting points have been made in those resources, that are unrelated to the main topic of this article.
Anyway, here is a list of very interesting material to dig into.

**Blogs,forums,mls**