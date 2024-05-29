<!--yml
category: 未分类
date: 2024-05-27 14:29:59
-->

# SMS Censorship — JMP Blog

> 来源：[https://blog.jmp.chat/b/sms-censorship](https://blog.jmp.chat/b/sms-censorship)

﻿Since almost the very beginning of JMP there have been occasional SMS and MMS delivery failures with an error message like “Rejected for SPAM”. By itself this is not too surprising, since every communications system has a SPAM problem and every SPAM blocking technique has some false positives. Over the past few years, however, the incidence of this error has gone up and up. But whenever we investigate, we find no SPAM being sent, just regular humans having regular conversations. So what is happening here? Are the SPAM filters getting worse?

In a word: yes.

It seems that in an effort to self-regulate and reduce certain kinds of “undesirable content” most carriers have resorted to wholesale keyword blocking of words not commonly found in SPAM, but referring to items and concepts the carriers find undesirable. For example, at least one major USA carrier blocks every SMS message containing the word “morphine”. How any hospital staff or family with hospitalized members are meant to know they must avoid this word is anyone’s guess, hopefully after being told their messages are “SPAM” they can guess to say “they upped Mom’s M dose” instead?

# What We Are Doing

To preserve our reputation with these carriers we have begun to build an internal list of the keywords being blocked by different major carriers, and blocking all messages with those keywords ourselves rather than attempt to deliver them. While this seems like a suboptimal solution, the messages would never have been delivered anyways and this reduces the amount of “SPAM” that the carriers see coming from us. We have also insituted a cooldown such that if your account triggers a “SPAM” error from a major carrier, further messages are blocked for a short time to avoid repeated attempts to send the same message.

So what are the kinds of “undesirable content” the carriers are attempting to avoid here?

*   Obviously please do not use JMP for anything illegal. This has never been allowed and we continue to not tolerate this in any way.
*   Additionally, please avoid sexually explicit or graphically violent discussions, or discussions about drugs illegal in any part of the USA.

This is not really our policy so much as it is that of the carriers we must work with in order to continue delivering your messages to friends and family.

# What You Can Do

Every JMP account comes with, as an option, a Snikket instance of your very own. As always, we **highly recommend** inviting friends and family you have many discussions with (especially discussions about sex, firearms, or drugs) to your Snikket instance and continuing all conversations there in private instead of broadcasting them over the phone network. Sending an invite link to your Snikket instance is easy, and anyone who uses the link will get an account on your instance, with yourself and others as a contact, set up automatically, so it is a great way to speak more securely with family and friend groups. Snikket will also enable higher quality media sharing, video calls, and many other benefits for your regular contacts.

Of course we know you will continue to need SMS and MMS for many of your contacts now and in the future, and JMP is dedicated to continuing to provide best-in-class service for person to person communication in this way as well.