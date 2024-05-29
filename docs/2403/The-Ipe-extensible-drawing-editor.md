<!--yml
category: 未分类
date: 2024-05-27 14:37:49
-->

# The Ipe extensible drawing editor

> 来源：[https://ipe.otfried.org/](https://ipe.otfried.org/)

# The Ipe extensible drawing editor

## 

Ipe is a drawing editor for creating figures in PDF format. It supports making small figures for inclusion into LaTeX-documents as well as making multi-page PDF presentations.

Ipe's main features are:

*   Entry of text as LaTeX source code. This makes it easy to enter mathematical expressions, and to reuse the LaTeX-macros of the main document. In the display text is displayed as it will appear in the figure.
*   Produces pure PDF, including the text. Ipe converts the LaTeX-source to PDF when the file is saved.
*   It is easy to align objects with respect to each other (for instance, to place a point on the intersection of two lines, or to draw a circle through three given points) using various snapping modes.
*   Users can provide ipelets (Ipe plug-ins) to add functionality to Ipe. This way, Ipe can be extended for each task at hand.
*   Ipe can be compiled for Unix, Windows, and OSX.
*   Ipe is written in standard C++ and Lua 5.4.

You can find more information about Ipe features in the [manual](https://otfried.github.io/ipe).

The manual is also available as a book in [epub format](https://ipe.otfried.org/ipe-manual.epub) and [pdf format](https://ipe.otfried.org/ipe-manual.pdf).

If you want to develop Ipe scripts or ipelets, you'll need the [Ipe library documentation](https://ipe.otfried.org/ipelib).

#### Download the current Ipe version

The current version of Ipe is Ipe 7.2.29. (Older versions can be found [here](https://github.com/otfried/ipe/releases), and even older versions [here](https://github.com/otfried/old-ipe-releases/releases).)

I'm making four downloads of Ipe available: a binary distribution for Windows, a binary package for Mac OS X, binary packages for several Linux-distributions, and a source package that should compile on any recent Unix system.

You can also install Ipe as a [flatpak](https://flathub.org/apps/org.otfried.Ipe)—this will work on many Linux distributions, also older ones for which we don't build packages anymore.

##### Windows binary package

[ipe-7.2.29-win64.zip](https://github.com/otfried/ipe/releases/download/v7.2.29/ipe-7.2.29-win64.zip)

Unzip this package somewhere on your Windows computer. This package requires at least Windows 7.

##### Mac OS X binary package

Download the following disk image and copy Ipe.app onto your Mac running Mac OS 10.10 or higher.

Security warning: When you try to normally open a newly downloaded version of Ipe, MacOS will show a security warning that says that it cannot verify the developer and that it cannot check for malicious software. This is the [normal behaviour of MacOS](https://support.apple.com/en-au/guide/mac-help/mh40616/mac). Control-Click the Ipe icon, then choose "Open" from the shortcut menu.

For Apple computers with Intel processor: [ipe-7.2.29-mac-intel.dmg](https://github.com/otfried/ipe/releases/download/v7.2.29/ipe-7.2.29-mac-intel.dmg)

For Apple computers with ARM processor ("Apple silicon"): [ipe-7.2.29-mac-arm.dmg](https://github.com/otfried/ipe/releases/download/v7.2.29/ipe-7.2.29-mac-arm.dmg)

For MacOS, [IpePresenter](https://ipepresenter.otfried.org) is provided as a separate app. Copy IpePresenter.app onto your Mac:

For Apple computers with Intel processor: [ipepresenter-7.2.29-mac-intel.dmg](https://github.com/otfried/ipe/releases/download/v7.2.29/ipepresenter-7.2.29-mac-intel.dmg)

For Apple computers with ARM processor ("Apple silicon"): [ipepresenter-7.2.29-mac-arm.dmg](https://github.com/otfried/ipe/releases/download/v7.2.29/ipepresenter-7.2.29-mac-arm.dmg)

##### Linux binary packages

Several Linux distributions (including Debian, Ubuntu, Linux Mint, Fedora, Arch Linux) offer an ipe package, which you can install through the distribution's package manager.

It takes a while for a new Ipe version to make it into a distribution, especially the long-term stable distributions. Thanks to the wonderful [openSuse build service](https://build.opensuse.org), I can provide Ipe installation packages for some recent Linux distributions, currently Debian, Ubuntu, Mint, Fedora, and openSuse. (On archlinux, you can rely on the [official package](https://aur.archlinux.org/packages/ipe/), which is always quite up to date.)

Before you install these packages, remove any old Ipe version you installed through your distribution's package manager!

#### Other Linux distributions and older releases (e.g. Ubuntu 20.04, Debian 11, Fedora 36)

If your Linux distribution is not mentioned below, or if your Linux release is older, you can install Ipe as a [flatpak](https://flathub.org/apps/org.otfried.Ipe). (For instance, since Ipe 7.2.27 Ipe relies on Qt6, we cannot provide packages for Ubuntu 20.04, Debian 11, or Fedora 36 anymore.)

#### Debian, Ubuntu, Mint

For these Debian-based distributions, download a DEB-package according to the following table.

Then install the package by saying:

```
$ sudo gdebi ipe_7.2.29-1_amd64.deb

```

To remove Ipe again, use

```
$ sudo apt-get remove ipe

```

#### Fedora and openSuse

For these distributions you need to download an RPM-package, according to the following table.

##### May the source be with you

[ipe-7.2.29.tar.gz](https://github.com/otfried/ipe/archive/refs/tags/v7.2.29.tar.gz)

This includes sources to build Ipe, as well as the Ipe documentation. See the install.txt file for instructions.

#### Ipe merchandise

Are you an Ipe fan? You can show everybody by wearing the [Ipe T-shirt](https://www.shirtee.com/en/store/ipe), and sponsor Ipe development at the same time.

#### Sponsor Ipe development

You now have the opportunity to become a [member of the community](donate.html) that sponsors Ipe's development.

#### Wiki

After the manual, your second source of useful information, example files, or answers to frequently asked questions is the [Ipe 7 Wiki](https://github.com/otfried/ipe-wiki/wiki). The idea is that Ipe users add useful tips and tricks, or anything related to Ipe here.

#### Mailing lists

There are two mailing lists for Ipe. The first mailing list is used solely to announce new versions of Ipe, and perhaps new ipelets that may be interesting to a broad audience. Traffic on this list is very light, as most messages on this list come from me. You can subscribe to the announcement list [here](https://mailman.science.uu.nl/mailman/listinfo/ipe-announce).

The second list is used to discuss Ipe. You can subscribe [here](https://mailman.science.uu.nl/mailman/listinfo/ipe-discuss). Please don't use it to report bugs—the bug tracker is much better at that.

Both lists are maintained by René van Oostrum. Thank you!

#### IpePresenter

Ipe now comes with a companion program IpePresenter. IpePresenter is a presentation tool to show PDF presentations (made with Ipe or with the beamer Latex package)—see the [IpePresenter homepage](https://ipepresenter.otfried.org) for details.

IpePresenter is included in the binary packages for Ipe on Windows and Linux, and is also available as a separate binary package (without Ipe) for Windows and MacOS.

#### Reporting bugs

Before reporting a bug, please verify that you have the latest Ipe version, and check that the problem is not explained in the [FAQ](https://otfried.github.io/ipe/96_faq.html). Please do not send bug reports directly to me (the first thing I would do with your report is to enter it into the bug tracker).

To report bugs, please use the [Ipe bug tracker](bugzilla.html) (click on New issue).

#### Copyright

The extensible drawing editor Ipe is "free," this means that everyone is free to use it and free to redistribute it on certain conditions. Ipe is not in the public domain; it is copyrighted and there are restrictions on its distribution as follows:

Copyright © 1993–2024 Otfried Cheong

This program is free software; you can redistribute it and/or modify it under the terms of the Gnu General Public License as published by the Free Software Foundation; either version 3 of the License, or (at your option) any later version.

As a special exception, you have permission to link Ipe with the CGAL library and distribute executables, as long as you follow the requirements of the Gnu General Public License in regard to all of the software in the executable aside from CGAL.

This program is distributed in the hope that it will be useful, but without any warranty; without even the implied warranty of merchantability or fitness for a particular purpose. See the [Gnu General Public License](https://www.gnu.org/copyleft/gpl.html) for more details.

#### Other downloads

Several separate programs are available from the [ipe-tools](https://github.com/otfried/ipe-tools) repository on GitHub. These are available in source form only.

##### Svgtoipe

svgtoipe converts SVG figures to Ipe format. It cannot handle everything in SVG, but should work for geometric objects and gradients. This program is actually a Python script.

##### Poweripe

Do you prefer to create your presentations in Ipe? But your bosses/colleagues/clients keep asking for Powerpoint files?

Fear no more! Poweripe is a Python script that translates an Ipe presentation into a Powerpoint presentation.

##### Pdftoipe

pdftoipe converts arbitrary PDF files to Ipe's XML format, or at least it tries to. You'll need the poppler library to compile it.

##### Ipe Python module

ipepython is an extension module for Python 3 that allows you to read and write Ipe documents and to process all the information inside.

##### Matplotlib backend

Matplotlib is a Python module for scientific plotting. With this backend, you can create Ipe figures directly from matplotlib.

##### Figtoipe

figtoipe converts figures made with Xfig to Ipe's XML format. It does not handle all the features of Xfig. figtoipe was improved and is currently maintained by Alexander Bürger. Thank you!

##### ipe5toxml

If you still have figures that were made with Ipe 5, you can use this program to convert them to the format understood by Ipe 6\. You can then run ipe6upgrade to convert them to Ipe 7 format. The source to ipe5toxml is in the public domain.