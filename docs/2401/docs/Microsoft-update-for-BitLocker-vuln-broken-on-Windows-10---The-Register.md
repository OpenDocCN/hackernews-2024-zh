<!--yml
category: 未分类
date: 2024-05-27 14:48:34
-->

# Microsoft update for BitLocker vuln broken on Windows 10 • The Register

> 来源：[https://www.theregister.com/2024/01/12/microsoft_update_for_bitlocker_vuln/](https://www.theregister.com/2024/01/12/microsoft_update_for_bitlocker_vuln/)

Updated Microsoft sent yet another problematic patch into the wild this week in the form of KB5034441\. However, rather than deal with a BitLocker vulnerability, the patch is failing to install for some users.

The [patch](https://support.microsoft.com/en-gb/topic/kb5034441-windows-recovery-environment-update-for-windows-10-version-21h2-and-22h2-january-9-2024-62c04204-aaa5-4fee-a02a-2fdea17075a8), released on January 9, was meant to address a vulnerability that allowed attackers to bypass BitLocker encryption by using the Windows Recovery Environment (WinRE). It was released for Windows 10 21H2 and 22H2 but appears to have been subject to Microsoft's legendary quality control.

When installing the update, some users are finding themselves faced with an `0x80070643` error, a generic failure message. Unfortunately, according to Microsoft, "because of an error in the error code handling routine," this might not be the correct error. The actual error could be one related to the recovery partition not being large enough: "Windows Recovery Environment servicing failed. (CBS_E_INSUFFICIENT_DISK_SPACE)."

There's a good chance that a PC with a standard Windows 10 installation might not have a recovery partition large enough to handle the update, something Microsoft tacitly acknowledged in [KB5028997](https://support.microsoft.com/en-gb/topic/kb5028997-instructions-to-manually-resize-your-partition-to-install-the-winre-update-400faa27-9343-461c-ada9-24c8229763bf) with instructions for how to resize the partition.

These instructions are not for the faint of heart. The first link requires the user to open a command prompt window with administrative privileges. It's downhill from there as users are directed to fire off commands to disable the WinRE before deleting and recreating recovery partitions. It's a risky process, with plenty of potential for the unwary to make mistakes.

Microsoft has asked users encountering issues with disk space – which might be concealed by the erroneous install failure error `0x80070643` – to have a go at the procedure to manually resize the partition to deal with the problem.

A glance at [social media](https://www.reddit.com/r/Windows10/comments/192l9kj/comment/kh32u6f/) shows that the problem is widespread, and users are reluctant to apply Microsoft's workaround. Understandably, it has been called "too technical and scary" by some, while another noted: "This is an issue that Microsoft needs to correct themselves."

Another said: "We shouldn't be the ones correcting the mistakes that Microsoft made. Just hold off on that update now, and Microsoft will push a fix in the future."

*The Register* contacted Microsoft to find out if the software giant intends to update its patch or push out something that does not require hacking away at the file system from the command line after a failure.

In the meantime, we'd have to concur with the user who said: "Microsoft needs to do a better job checking their updates before pushing them."

Indeed. Showing an unhelpful error message and then requiring a user to delve into the world of the command line to fix things. What is this? Linux? ®

### Updated to add on January 16:

A Microsoft spokesperson told us that the company had updated its public documentation on the matter. While the guidance is unchanged, Microsoft said: "We are working on a resolution and will provide an update in an upcoming release."

So that's alright then."