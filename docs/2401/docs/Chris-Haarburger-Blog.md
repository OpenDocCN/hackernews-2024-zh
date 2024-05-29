<!--yml
category: 未分类
date: 2024-05-27 15:01:40
-->

# Chris Haarburger Blog

> 来源：[https://chris-haarburger.com/posts/security_startups.html](https://chris-haarburger.com/posts/security_startups.html)

Jan 20, 2024

## Ten Simple IT Security Rules for Early-Stage Startups

In the earliest days of a startup, there are almost no rules and processes. Nonetheless, as soon as the first few employees join, the risk of data breaches, phishing, or getting [pwned](https://haveibeenpwned.com) is increasing.

It is essential to introduce formal processes and rules as slowly as possible to keep these early days' speed as long as possible. Therefore, I created a minimal set of rules that are easy to digest for everybody but can significantly reduce the most relevant IT security risks. From my experience, the more elaborate these kinds of rules are formalized, the less likely that they are read or even adhered to. Therefore, I did my best to boil it down to 10 simple rules for all team members:

1.  Generate and save all your passwords in our password manager, for which we have a company license. Save all company-related sensitive information (passwords, credit card data, bank accounts, etc.) in that account. Don’t save such information anywhere else.
2.  Enable 2-factor authentication based on one-time-password (OTP) for every service that offers 2FA. Sync OTPs using the password manager. Never reveal OTPs on the phone, not even to colleagues or superiors (phishing alert!). The password manager will show you automatically for which services you can enable 2FA and have yet to do so.
3.  Always use unique (i.e., separate passwords for separate services) automatically-generated passwords that your password manager suggests—minimum requirements: 10 characters using letters, numbers and special symbols. On mobile, use a strong passcode to unlock your phone. Avoid being watched when typing passwords.
4.  If there is any sign of a breach on your accounts, change your password to the respective affected service and notify your superior. Generally, don’t trust e-mails and never open attachments from unknown senders.
5.  Encrypt your hard drives using FileVault (macOS) or Bitlocker (Windows) and never leave your machine without locking it.
6.  Watch out for phishing e-mails. If in doubt, ask the sender on another channel if the e-mail is actually from that sender, especially if the e-mail is about transferring money.
7.  Only use our company Google Account (Gmail, GDrive) to store files, not your laptop hard drive. Only data from your Google-account is backed up regularly (dumped to as NAS every night).
8.  If you need to share passwords/credentials, always use the “share” feature in the password manager. Don’t send passwords via text/e-mail/WhatsApp/
9.  Perform all regular software updates for apps and operating systems on your computer and smartphone.
10.  SAAS tools: Only use software that the company provides. When sharing data with 3rd parties, don't share publicly. Instead, always restrict sharing to specific individuals.

Thanks a lot to my friend Flo for feedback and refinement.