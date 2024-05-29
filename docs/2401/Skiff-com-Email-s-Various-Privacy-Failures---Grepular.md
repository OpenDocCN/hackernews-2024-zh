<!--yml
category: 未分类
date: 2024-05-27 14:46:07
-->

# Skiff.com Email's Various Privacy Failures - Grepular

> 来源：[https://www.grepular.com/Skiff_Emails_Various_Privacy_Failures](https://www.grepular.com/Skiff_Emails_Various_Privacy_Failures)

I came across [Skiff.com](https://skiff.com/) on a [Hacker News submission](https://news.ycombinator.com/item?id=37229270). They claim to provide a “Privacy-first end-to-end encrypted email” service. As the author of the [Email Privacy Tester](https://www.emailprivacytester.com) I like to test email providers and clients when I first come across them.

### A Bad Default

One of the first things I noticed after signing up for a free account with Skiff, is that they have the standard “Block Remote Content” option which pretty much all email providers and clients have, but that the option is disabled by default. This default didn’t strike me as matching their “Privacy-first” claim. If privacy is important enough to be the first thing you mention on your website, then blocking remote content should be enabled by default.

I [mentioned this](https://news.ycombinator.com/item?id=37238112) on the Hacker News thread, and the CEO of Skiff, Andrew Milich, responded. He had two responses: “privacy focused mail providers offer this as an option but do not enable it by default”. I’d be surprised if that’s the case for privacy focused mail providers, but I haven’t done a survey. But also, “Mainstream mail providers do not even have it as an option”. That’s just flat out false, and I’m not sure why he thinks it. Loading remote content (or not) is a standard option available from nearly all email clients and providers and has been for decades. Also… Who cares what other mail providers do? If “Privacy-first” is your main selling point, do things that are “Privacy-first”, don’t just copy your competitors.

### A Leak in the Webmail, OSX and Windows clients

Anyway, I proceeded to test their webmail client through the Email Privacy Tester to see if it had any bugs. What I discovered was that if you have “Block Remote Content” enabled, they display placeholder images where the remote content would be, as is normal, and the way most other clients work. However, what is different in Skiff’s case, is that they still fetch the remote content in the background. To their credit, they do proxy the requests so that your IP address isn’t exposed, just like GMail does, but email open information is still exposed. This completely defeats the main purpose of blocking remote content and is not how any other mail providers that I’ve tested over the years work.

I took this back to the same Hacker News thread. The CEO of Skiff responded that what I was saying was “[patently false disinfo](https://news.ycombinator.com/item?id=37243228)“. Unsure of why he refused to even entertain the idea of the bug, I pointed him to a tool where he could verify this himself (Email Privacy Tester). I received the bizarre response that he tested it and found it worked the same as [Tutanota](https://tutanota.com/). I tested Tutanota’s Webmail and no it does not. Tutanota does not have this bug, and unlike Skiff, puts “Privacy-first” and [defaults to blocking remote content](https://tutanota.com/security).

At this point, I figured he doesn’t understand the problem, which is fine. What is not fine is the issue being brushed off as “disinformation”, rather than being forwarded to somebody on his team who might be able to understand, diagnose and fix the problem. I created a screencast and [uploaded it to Youtube](https://www.youtube.com/watch?v=P30Qi2MSbUQ), showing the bug in action.

I also tested the OSX and Windows clients and they behave in the same manner.

### IP Address Leak, on iOS

After testing their iOS client on my iPhone, I discovered that it is even worse than the other clients. First of all, the “Block Remote Content” setting was not synced across so I had to apply it a second time. But the main problem, although they attempt to block remote content from loading on the iOS client when you enable that setting, they fail to do so when that remote content is loaded via one of the following html tags:

```
<p style="content:url('http://TRACKING_URL/')"></p>

<p style="background-image:url('http://TRACKING_URL/');"></p> 
```

And in their failure to prevent that loading, they also fail to proxy it, meaning that not only is the fact that you have read that email exposed, but your real IP address is exposed to the sender too. Given Skiff’s lack of interest in my previous report, I didn’t bother reporting this bug to them directly. I haven’t tested their Android client. If you’re using it, I suggest testing it yourself.

### Notifying their users?

Assuming they eventually fix these bugs (who knows?), it will be interesting to see if they email their users to alert them to their potential exposure. Given the attitude of their CEO so far, I’m guessing they will just brush these issues under the rug and not inform their users. Perhaps they will mention it on a webpage somewhere which the majority of their users wont see, so they can claim their users have been informed. Prove me wrong Skiff.

### A Private Way to allow Remote Content

One of the things the CEO said to me in the thread regarding blocking remote content was “There is no foolproof way to load any remote content without possibly exposing email open information”. I [corrected this](https://news.ycombinator.com/item?id=37240292), and didn’t get a response. It is totally possible to load remote content at the same time as hiding email open information. You would do this by fetching and storing the remote content at the point of delivery, for all email. I don’t know of any providers that do this, but it is certainly a solution to the problem and one that I hope providers start to use one day. Certainly providers who claim to be “Privacy-first”. If all providers started doing this, it would drastically reduce the ability of senders to track email; they’d have to rely on users clicking links containing unique ids.

### Security Audits

According to Skiff’s [Transparency page](https://skiff.com/transparency) they’ve undergone and are undergoing four security audits:

*   Cure53 Aug 2023 (Upcoming)
*   Trail of Bits July 2022
*   Trail of Bits Feb 2022
*   Tom Ritter (Security Consultant) Jan 2021

They don’t provide any further information, so we really have no idea what was actually tested as part of these audits, nor what the results were. I can only assume they haven’t had their webmail client or desktop apps audited to ensure that they’re working as intended; even basic testing of the “Block Remote Content” function would have shown a problem. Security Auditors, take note: If you’re testing something related to email, Email Privacy Tester exists, and it’s GPL-3, so you can even [host your own version](https://gitlab.com/mikecardwell/ept3).

### MTA-STS

I also tested if they support [MTA-STS](https://www.ncsc.gov.uk/collection/email-security-and-anti-spoofing/using-mta-sts-to-protect-the-privacy-of-your-emails). Here, they’ve done half a job. They support it for inbound mail, but not outbound. I confirmed this by disabling the advertising of STARTTLS on my MTA, to their hosts only, and testing if they still delivered mail to my address (which has MTA-STS set up on it). Here is a log of them delivering mail to me over a TLS secured connection before I made the change, followed by a log of them delivering mail to me over a non-TLS connection after I made it:

```
2023-08-25 18:06:51 1qZbCq-00000C-0T <= mike.cardwell@skiff.com H=outbound-smtp.skiff.com [35.166.143.94]:47782 I=[172.19.0.3]:25 P=esmtps X=TLS1.2:ECDHE_RSA_AES_256_GCM_SHA384:256 CV=no SNI="mail.grepular.com" S=2612 id=c4588db9-741e-4c6d-a1d4-490432f7cdc6@mail.skiff.com

2023-08-26 15:37:49 1qZvM8-00000A-TS <= mike.cardwell@skiff.com H=outbound-smtp.skiff.com [35.166.143.94]:36226 I=[172.22.0.4]:25 P=esmtp S=2568 id=f1f681b6-0161-45a0-834c-1ab8f8b86361@mail.skiff.com 
```

Come on, finish the job Skiff.

### DMARC and Spoofing

I tested if they support DMARC by sending an email to my Skiff address from an IP outside my SPF allow-list, and without a DKIM signature. According to my DMARC record, they should have rejected the email, but instead of rejecting, they treated it as if I specified “quarantine” and placed it in the Spam folder. I can’t see from the outside if failing DMARC will always lead to a message being filtered into Spam, or if they simply treat it as just another Spam indicator. Hopefully, an email which fails DMARC will *always* go into the Spam folder at least. The main problem is, this email was spoofed, and there is nothing in the UI to indicate that was the case, to the reader.

### Summary

The existence of bugs is to be expected, but the attitude of Skiffs CEO towards them being reported is not. All he needed to say to me was, “Thanks for the report. It’s not supposed to work that way. I’ll get somebody to look into it”. Therefore, I would recommend you not trust this company with your data. You never know what legitimate privacy/security bug reports they have dismissed in the past, or will dismiss in the future. I will update this blog post if I become aware of them fixing any of these issues.

### Update 2023-Aug-29

They have fixed the IP address leak on iOS. No sign of users being informed of their exposure. Other issues remain.