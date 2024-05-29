<!--yml
category: 未分类
date: 2024-05-29 12:41:52
-->

# HMRC's Linux version of Basic PAYE Tools broke after update • The Register

> 来源：[https://www.theregister.com/2024/03/26/hmrc_linux_paye_tools/](https://www.theregister.com/2024/03/26/hmrc_linux_paye_tools/)

Updated Did you know that His Majesty's Revenue and Customs (HMRC) offers free Linux tools? Sadly, though, they recently stopped working.

*Reg* reader Pete Donnell alerted us to the fact that the UK tax authority offers a Linux version of its Basic PAYE Tools, which [the download page](https://www.gov.uk/basic-paye-tools) describes as "free payroll software from HM Revenue and Customs (HMRC) for businesses with fewer than 10 employees."

Sounds good. We were impressed to see that, alongside its Windows and Mac versions, there's also a version of the software that supports "Ubuntu Linux version 20.04 LTS and 23.10." There's a detailed [User Guide](https://www.gov.uk/government/publications/basic-paye-tools-user-guide/basic-paye-tools-user-guide) too, and more than one service status page, ranging from [rather dated](https://www.gov.uk/government/publications/basic-paye-tools-service-availability-and-issues) to [extremely dated](https://customs.hmrc.gov.uk/channelsPortalWebApp/channelsPortalWebApp.portal?_nfpb=true&_pageLabel=pageNoNavigation_ShowContent&id=HMCE_PROD1_031142&propertyType=document).

Of course, we wouldn't be writing about this if there weren't a snag. This story has been brewing for some time, but Pete told us that on February 5:

Despite the service status pages not showing any problems, it's been broken ever since. He tells us that he's had the software set to automatically update for about 15 years, and it's never failed before. He's tried it on both Ubuntu 20.04 and a clean install of Xubuntu 23.10, among other things. Mac users [are also reporting problems](https://discussions.apple.com/thread/255234259).

Worryingly, Donnell notes that the software is seemingly written in Python 2\. That version has been unsupported and dead for [just over four years](https://pythonclock.org/).

We contacted HMRC and put them in touch with Donnell. He subsequently spoke with them, and provided them with some troubleshooting information. A month later, he let us know that nothing had changed yet.

It seemed like a welcome surprise to find that a government department not merely acknowledges the existence of Linux, but supports it and offers Linux-specific downloads … but every silver lining has a cloud.

If anyone else is using the PAYE tools and has similar problems or, indeed, if you have the software working or can help Donnell troubleshoot what looks to be a Python 2 and Django app, do let us know. It's a free download and he tells us that it fails before any form of employer's account is needed. ®

### Update to add

Mr Donnell has got back in touch to let us know that the combined powers of *The Reg* readership helped him to isolate the issue and resolve it. He tells us: