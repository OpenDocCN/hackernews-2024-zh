<!--yml
category: 未分类
date: 2024-05-28 18:17:20
-->

# Fedora Workstation 41 To No Longer Install GNOME X.Org Session By Default - Phoronix

> 来源：[https://www.phoronix.com/news/Fedora-41-No-GNOME-Xorg-Install](https://www.phoronix.com/news/Fedora-41-No-GNOME-Xorg-Install)

Fedora Workstation has long defaulted to using GNOME's Wayland session by default, but it has continued to install the GNOME X.Org session for fallback purposes or those opting to use it instead. But for the Fedora Workstation 41 release later in the year, there is a newly-approved plan to no longer have that GNOME X.Org session installed by default.

Recently there was a Fedora Workstation ticket opened to no longer install the GNOME X.Org session by default. This is just about whether the X.Org session is pre-installed but would continue to live in the repository for those wanting to explicitly install it.

The Fedora Workstation working group

[decided](https://pagure.io/fedora-workstation/issue/414#comment-899128)

to go ahead with this change for the Fedora 41 cycle, not the upcoming Fedora 40 release:

> "Fedora Workstation WG discussed this today and we agreed we should do this for Fedora 41, since it is really too late already for F40 and it should really be handled as a System Wide Change anyway."

So pending any obstacles by FESCo, which is unlikely. Fedora Workstation 41 will not be installing the GNOME X.Org session by default. Long live Wayland.