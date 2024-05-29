<!--yml
category: 未分类
date: 2024-05-29 12:46:01
-->

# Arch Linux - News: The xz package has been backdoored

> 来源：[https://archlinux.org/news/the-xz-package-has-been-backdoored/](https://archlinux.org/news/the-xz-package-has-been-backdoored/)

**Update:** To our knowledge the malicious code which was distributed via the release tarball never made it into the Arch Linux provided binaries, as the build script was configured to only inject the bad code in Debian/Fedora based package build environments. The news item below can therefore mostly be ignored.

We are closely monitoring the situation and will update the package and news as neccesary.

* * *

TL;DR: Upgrade your systems and container images **now**!

As many of you may have already read ([one](https://www.openwall.com/lists/oss-security/2024/03/29/4)), the upstream release tarballs for `xz` in version `5.6.0` and `5.6.1` contain malicious code which adds a backdoor.

This vulnerability is tracked in the Arch Linux security tracker ([two](https://security.archlinux.org/ASA-202403-1)).

The `xz` packages prior to version `5.6.1-2` (specifically `5.6.0-1` and `5.6.1-1`) contain this backdoor.

The following release artifacts contain the compromised `xz`:

*   installation medium `2024.03.01`
*   virtual machine images `20240301.218094` and `20240315.221711`
*   container images created between and including *2024-02-24* and *2024-03-28*

The affected release artifacts have been removed from our mirrors.

We strongly advise against using affected release artifacts and instead downloading what is currently available as latest version!

## Upgrading the system

It is strongly advised to do a full system upgrade right away if your system currently has `xz` version `5.6.0-1` or `5.6.1-1` installed:

`pacman -Syu`

## Upgrading container images

To figure out if you are using an affected container image, use either

`podman image history archlinux/archlinux`

or

`docker image history archlinux/archlinux`

depending on whether you use `podman` or `docker`.

Any Arch Linux container image older than `2024-03-29` and younger than `2024-02-24` is affected.

Run either

`podman image pull archlinux/archlinux`

or

`docker image pull archlinux/archlinux`

to upgrade affected container images to the most recent version.

Afterwards make sure to rebuild any container images based on the affected versions and also inspect any running containers!

## Regarding sshd authentication bypass/code execution

From the upstream report ([one](https://www.openwall.com/lists/oss-security/2024/03/29/4)):

> openssh does not directly use liblzma. However debian and several other distributions patch openssh to support systemd notification, and libsystemd does depend on lzma.

Arch does not directly link openssh to liblzma, and thus this attack vector is not possible. You can confirm this by issuing the following command:

`ldd "$(command -v sshd)"`

However, out of an abundance of caution, we advise users to remove the malicious code from their system by upgrading either way. This is because other yet-to-be discovered methods to exploit the backdoor could exist.