<!--yml
category: 未分类
date: 2024-05-29 12:12:23
-->

# Linux 6.9 Set To Drop The Old NTFS File-System Driver - Phoronix

> 来源：[https://www.phoronix.com/news/Linux-6.9-Dropping-Old-NTFS](https://www.phoronix.com/news/Linux-6.9-Dropping-Old-NTFS)

Merged two years ago with Linux 5.15 with

[the "NTFS3" driver developed by Paragon Software](https://www.phoronix.com/news/NTFS3-For-Linux-5.15)

with working read-write support and other improvements for supporting Microsoft's NTFS file-system driver. This driver was a big improvement over the original NTFS read-only driver found in the mainline kernel and faster than using the NTFS-3G FUSE file-system driver. Now with enough time having passed and the NTFS3 driver working out well, the older NTFS driver is set for removal.

Ahead of the Linux 6.9 merge window opening this weekend or next depending upon how the v6.8 cycle plays out this weekend, Christian Brauner submitted a "

[vfs ntfs](https://lore.kernel.org/lkml/20240308-vfs-ntfs-ede727d2a142@brauner/)

" pull request that nukes the old NTFS driver. He argues for its removal as:

> "This removes the old ntfs driver. The new ntfs3 driver is a full replacement that was merged over two years ago. We've went through various userspace and either they use ntfs3 or they use the fuse version of ntfs and thus build neither ntfs nor ntfs3\. I think that's a clear sign that we should risk removing the legacy ntfs driver.
> ...
> It's unmaintained apart from various odd fixes as well. Worst case we have to reintroduce it if someone really has a valid dependency on it. But it's worth trying to see whether we can remove it."

Removing this legacy NTFS kernel driver lightens the Linux source tree by 29,303 lines.