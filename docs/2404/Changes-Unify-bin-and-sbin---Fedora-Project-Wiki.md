<!--yml
category: 未分类
date: 2024-05-27 13:11:43
-->

# Changes/Unify bin and sbin - Fedora Project Wiki

> 来源：[https://fedoraproject.org/wiki/Changes/Unify_bin_and_sbin](https://fedoraproject.org/wiki/Changes/Unify_bin_and_sbin)

# Unify `/usr/bin` and `/usr/sbin`

## Summary

The `/usr/sbin` directory becomes a symlink to `bin`, which means paths like `/usr/bin/foo` and `/usr/sbin/foo` point to the same place. `/bin` and `/sbin` are already symlinks to `usr/bin` and `usr/sbin`, so effectively `/bin/foo` and `/sbin/foo` also point to the same place. `/usr/sbin` will be removed from the default `$PATH`. The same change is also done to make `/usr/local/sbin` point to `bin`, effectively making `/usr/local/bin/foo` and `/usr/local/sbin/foo` point to the same place. The definition of `%_sbindir` will be changed to `%_bindir`, so packages will start using the new directory after a rebuild without any further action. Maintainers *may* stop using `%_sbindir`, but don't need to.

## Owner

## Current status

## Detailed Description

The split between `/bin` and `/sbin` is not useful, and also unused. The original split was to have "important" binaries statically linked in `/sbin` which could then be used for emergency and rescue operations. Obviously, we don't do static linking anymore. Later, the split was repurposed to isolate "important" binaries that would only be used by the administrator. While this seems attractive in theory, in practice it's very hard to categorize programs like this, and normal users routinely invoke programs from `/sbin`. Most programs that require root privileges for *certain* operations are also used when operating without privileges. And even when privileges are required, often those are acquired dynamically, e.g. using `polkit`. Since many years, the default `$PATH` set for users includes both directories. With the advent of systemd this has become more systematic: systemd sets `$PATH` with both directories for all users and services. So in general, all users and programs would find both sets of binaries.

One additional use of the `/bin`—`/sbin` split is `consolehelper`. In this approach, the user-facing program (`/bin/foo`) is a symlink to `/bin/consolehelper`, which is a suid binary that elevates privileges and calls the "real" `foo` (`/sbin/foo` or `/usr/libexec/foo`). Most uses of consolehelper have been moved to polkit ([https://fedoraproject.org/wiki/Features/UsermodeMigration](https://fedoraproject.org/wiki/Features/UsermodeMigration)), but some users remain ([https://bugzilla.redhat.com/show_bug.cgi?id=502765](https://bugzilla.redhat.com/show_bug.cgi?id=502765)). Use of `/sbin` for the privileged program is incompatible with the proposed merge; those packages will need to be adjusted to move the binary that requires privileges under `/usr/lib` or `/usr/libexec` (see Scope below).

Since generally all user sessions and services have both directories in `$PATH`, this split actually isn't *used* for anything. Its main effect is confusion when people need to use the absolute path and guess the directory wrong. Other distributions put some binaries in the other directory, so the absolute path is often not portable. Also, it is very easy for a user to end up with `/sbin` before `/bin` in `$PATH`, and for an administrator to end up with `/bin` before `/sbin` in `$PATH`, causing confusion. If this feature is dropped, the system became a little bit simpler, which is useful especially for new users, who are not aware of the history of the split.

Many years ago we merged `/bin` and `/usr/bin` ([https://fedoraproject.org/wiki/Features/UsrMove](https://fedoraproject.org/wiki/Features/UsrMove)). In some ways *that* split was similar: it had historical justification that went away more than a decade prior, it was impossible to cleanly categorize programs into the the categories so effectively both parts were needed for boot, and even though it was making the system more complicated for little gain, the split was being carried forward because it was easier to do so than to remove it ([http://www.freedesktop.org/wiki/Software/systemd/TheCaseForTheUsrMerge](http://www.freedesktop.org/wiki/Software/systemd/TheCaseForTheUsrMerge)). *This* split is much less visible, but it's also making the system more complicated for no gain, and removing it is the natural follow-up.

## Feedback

[https://lists.fedoraproject.org/archives/list/devel@lists.fedoraproject.org/message/TRQ3I7BDTHG6N7WQEQ7PMK3P6PVWXQMI/](https://lists.fedoraproject.org/archives/list/devel@lists.fedoraproject.org/message/TRQ3I7BDTHG6N7WQEQ7PMK3P6PVWXQMI/)

"Programs in /usr/bin have their documentation in section 1 of the manual, while programs /usr/sbin are documented in section 8."

[https://lists.fedoraproject.org/archives/list/devel@lists.fedoraproject.org/message/NALSOW35LXN6HZPULMQ46WXNGZO4SYK4/](https://lists.fedoraproject.org/archives/list/devel@lists.fedoraproject.org/message/NALSOW35LXN6HZPULMQ46WXNGZO4SYK4/)

> "The manual sections have historical meaning:
> 
> [https://en.wikipedia.org/wiki/Man_page#Manual_sections](https://en.wikipedia.org/wiki/Man_page#Manual_sections) eg. 1 = general commands 8 = system administration
> 
> So unless the tools themselves are changing their purpose or are in the wrong section now, the manual sections should stay the same."

## Benefit to Fedora

*   Packagers don't have to think whether to install programs in `%_bindir` or `%_sbindir`.
*   Users don't have to think whether programs are in `%_bindir` or `%_sbindir`.
*   Fedora becomes more compatible with other distributions (for example, we have `/sbin/ip` while Debian has `/bin/ip`, and we have `/bin/chmem` and `/bin/isosize`, but Debian has `/sbin/chmem` and `/sbin/isosize`, and we also have `/sbin/{addpart,delpart,lnstat,nstat,partx,ping,rdma,resizepart,ss,udevadm,update-alternatives}`, while Debian has those in under `/bin`, etc.)
*   Fedora becomes more compatible with Arch, which did the merge a few years ago.
*   `execvp` and related functions iterate over fewer directories. This probably doesn't matter for speed, but is a nice simplification when looking at logs or `strace` output.

## Scope

*   Proposal owners:
    *   Adjust `%_sbindir` in `/usr/lib/rpm/macros` (part of `rpm` package) to evaluate to `%_bindir`. Packages will be updated automatically during the mass rebuild.
    *   Add a `%filetrigger` to `filesystem` package to create symlinks to `../bin/foo` for every `foo` that is uninstalled from `/usr/sbin`.
    *   Add a `%posttrans` trigger to `filesystem` package to check that `/usr/sbin` only contains symlinks and do `ln -fs bin /usr/sbin`. (Those scriptlets make it easier to have a smooth transition. At all times, the old paths will still work. After the transition is complete we can drop the scriptlets and provide the `/usr/sbin` symlink in the `filesystem` package.)
    *   Adjust systemd package to build with `-Dsplit-bin=no`.
    *   File a pull request for Packaging Guidelines to stop mentioning `%{_sbindir}`. The macro will remain defined to avoid breakage of packages which use it.

*   Other developers:
    *   Packages which install to hardcoded `$DESTDIR/usr/sbin`, but then use `%{_sbindir}` in `%files`, will need to be adjusted.
    *   Packages which use consolehelper and install the same name under both directories will need to be adjusted to use a different directory. Some of those packages may be retired instead. See list below.

*   Packages using usermode with binaries in both directories:
    *   `anaconda-live`
    *   `beesu`
    *   `chkrootkit`
    *   `hddtemp`
    *   `mate-system-log`
    *   `setuptool`
    *   `subscription-manager`
    *   `system-switch-java`
    *   `xawtv`

*   Packages which package a symlink in `/usr/sbin` that will need to be dropped:
    *   opensmtpd
    *   rpcbind
    *   policycoreutils
    *   systemd-udev

*   Trademark approval: N/A (not needed for this Change)

*   Alignment with Community Initiatives: nope

## Upgrade/compatibility impact

The change should be mostly invisible for users. While the transition is ongoing, both sets of paths should work and users should have both directories in `$PATH`. Once the transition is finished, both sets of paths should work, but users will only have `/usr/bin` in `$PATH`.

## How To Test

## User Experience

## Dependencies

## Contingency Plan

*   If the move of a binary of any specific package causes problems, we can create a compat symlink like `/usr/sbin/foo → ../bin/foo` using a `%postin` scriptlet in that package.
*   If the change is causing problems in general and needs to be reverted, we'd need to undo the changes to macro definitions in `rpm` and rebuild some or all packages.
*   Contingency deadline: in principle can be done at any time, but would require a rebuild of some or all affected packages.
*   Blocks release? no. It is OK if the change is done partially.

## Documentation

TBD.

## Release Notes