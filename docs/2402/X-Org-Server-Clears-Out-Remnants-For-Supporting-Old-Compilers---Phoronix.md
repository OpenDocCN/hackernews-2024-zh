<!--yml
category: 未分类
date: 2024-05-29 13:16:07
-->

# X.Org Server Clears Out Remnants For Supporting Old Compilers - Phoronix

> 来源：[https://www.phoronix.com/news/XServer-Clear-Out-Old-Compilers](https://www.phoronix.com/news/XServer-Clear-Out-Old-Compilers)

There are still no signs of a new X.Org Server feature release coming in the near-term with most of the major stakeholders divesting from the xorg-server besides the XWayland portion of the code-base. But for those interested in the past few days there have been some NetBSD/OpenBSD build fixes to the X.Org Server as well as clearing out some remnants of old compiler support.

In particular,

[old Sun compiler support](https://gitlab.freedesktop.org/xorg/xserver/-/commit/6dafe3dbe658b1a2ad927eceb274808a1ac9bc05)

was cleared out with various "__SUNPRO_C" for the Sun Studio compilers. Additionally, old

[USL compiler](https://gitlab.freedesktop.org/xorg/xserver/-/commit/27b55301077f33d7fd67611bdc003e72cd321d0b)

checks have been removed too.

While some may argue against dropping support for old and outdated compilers on the basis of the X.Org Server's importance and history, support for these environments were effectively dead already. Due to the switch from the GNU Autoconf to Meson build system for the X.Org Server, Meson doesn't support these outdated targets and in turn weren't buildable as it was. Thus it's just removing dead code at this stage.

There's also been bits of other spring cleaning in the codebase, such as this

[interesting patch](https://gitlab.freedesktop.org/xorg/xserver/-/commit/a91a862332a6d5d61aa749a8fd8b87985541a602)

by Oracle's Alan Coopersmith. The longtime X.Org developer explained:

> "unifdef SUNSYSV
> 
> I can't tell what this code was originally for - it was added in 1988, 4 years before the release of the SysV R4 release of Solaris 2.0, and I can't find anywhere that defined SUNSYSV."

X.Org Server: the old codebase that's rare to see new feature releases besides

[the ongoing slew of security issues](https://www.phoronix.com/news/X.Org-2024-Six-More-Security)

that in turn has led to new patch releases at least. It will be interesting to see if any new X.Org Server feature release(s) take place in 2024 outside of the XWayland scope given the diminishing developer interest and lots of talk but little action by those anti-Wayland holdouts in the community to actually contribute to X.Org Server development.