<!--yml
category: 未分类
date: 2024-05-27 13:03:41
-->

# VMS Software prunes OpenVMS hobbyist program • The Register

> 来源：[https://www.theregister.com/2024/04/09/vsi_prunes_hobbyist_prog/](https://www.theregister.com/2024/04/09/vsi_prunes_hobbyist_prog/)

Bad news for those who want to play with OpenVMS in non-production use. Older versions are disappearing, and the terms are getting much more restrictive.

The corporation behind the continued development of OpenVMS, VMS Software, Inc. – or VSI to its friends, if it has any left after this – has announced the latest [Updates to the Community Program](https://vmssoftware.com/about/news/2024-03-25-community-license-update/). The news does not look good: you can't get the Alpha and Itanium versions any more, only a limited x86-64 edition.

OpenVMS is one of the granddaddies of big serious OSes. A direct descendant of the OSes that inspired DOS, CP/M, OS/2, and Windows, as well as the native OS of the hardware on which Unix first went 32-bit, VMS has been around for nearly half a century. For decades, its various owners have offered various flavors of ["hobbyist program"](https://www.openvmshobbyist.com/news.php) under which you could get licenses to install and run it for free, as long as it wasn't in production use.

Since Compaq acquired DEC, then HP acquired Compaq, its prospects looked checkered. HP [officially killed it off](https://www.theregister.com/2013/06/10/openvms_death_notice/) in 2013, then in 2014 [granted it a reprieve](https://www.theregister.com/2014/07/31/openvms_spared/) and sold it off instead. New owner VSI [ported it to x86-64](https://www.theregister.com/2016/10/13/openvms_moves_slowly_towards_x86/), releasing that [new version 9.2](https://www.theregister.com/2022/05/10/openvms_92/) in 2022.

Around this time last year, we covered VSI [adding AMD support and opening a hobbyist program](https://www.theregister.com/2023/04/13/openvms_921_x86_hobby/) of its own. It seems from the latest announcement that it has been disappointed by the reception:

Although HPE [stopped offering](https://www.openvmshobbyist.com/forum/viewthread.php?forum_id=20&thread_id=3524) hobbyist licenses for the original VAX versions of OpenVMS in 2020, VSI continued to maintain OpenVMS 8 (in other words, the Alpha and Itanium editions) while it worked on version 9 for x86-64\. VSI even offered a [Student Edition](https://training.vmssoftware.com/student-license/), which included a [freeware Alpha emulator](http://mail.migrationspecialties.com/FreeAXP.html) and a copy of OpenVMS 8.4 to run inside it.

Those licenses run out in 2025, and they won't be renewed. If you have vintage DEC Alpha or HP Integrity boxes with Itanic chips, you won't be able to get a legal licensed copy of OpenVMS for them, or renew the license of any existing installations – unless you pay, of course.

There will still be a [Community license](https://vmssoftware.com/community/community-license/) edition, but from now on it's x86-64 only. Although OpenVMS 9 mainly targets hypervisors anyway, it does support bare-metal operations on a single model of HPE server, the [ProLiant DL380 Gen9](https://www.hpe.com/psnow/doc/c04346247). If you have one of them to play with – well, tough. Now Community users only get a VM image, supplied as a VMWare `.vmdk` file. It contains a ready-to-go "OpenVMS system disk with OpenVMS, compilers and development tools installed." Its license runs for a year, after which you will get a fresh copy.

This means you won't be able to configure your own system and keep it alive – you'll have to recreate it, from scratch, annually. The only alternative for those with older systems is to [apply](https://share.hsforms.com/131UO5x_iSEGxlYs8pQTYSQdi37l) to be an OpenVMS Ambassador.

The decision *is* understandable. There is no new Alpha or Itanium hardware and never will be. VSI is a business, and not a big one; it needs to make money. While it was maintaining v8, it continued offering hobbyist licenses for it, but now, if OpenVMS has a future, it's v9\. VSI put a lot of effort – and money – into creating the new x86-64 edition. Version 9 is its only chance of survival, so that's what it wants to promote.

But saying that… rare indeed are those passionate about vintage enterprise software that they want to run old big-iron servers in their basements, and keep them legally licensed. Alienating those folks doesn't look good, and surely won't help. *The Reg* FOSS desk suspects that the main result will be to encourage them to run up the skull and crossbones flag and become software pirates. ®