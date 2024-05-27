<!--yml
category: 未分类
date: 2024-05-27 12:55:22
-->

# OWASP discloses breach due to a Wiki web server misconfig • The Register

> 来源：[https://www.theregister.com/2024/04/02/owasp_discloses_data_breach/](https://www.theregister.com/2024/04/02/owasp_discloses_data_breach/)

A misconfigured MediaWiki web server allowed digital snoops to access members' resumes containing their personal details at the Open Web Application Security Project (OWASP) Foundation.

According to the nonprofit, which works to improve web app security, it became aware of the misconfig and subsequent data breach in late February after receiving "a few" report requests.

"If you were an OWASP member from 2006 to around 2014 and provided your resume as part of joining OWASP, we advise assuming your resume was part of this breach," OWASP said in a Good Friday [notification](https://owasp.org/blog/2024/03/29/OWASP-data-breach-notification) posted on its website.

"We recognize the significance of this breach, especially considering the OWASP Foundation's emphasis on cybersecurity," it added.

The resumes contained names, email addresses, phone numbers, physical addresses, "and other personally identifiable information," presumably people's places of employment, we're told. 

While the good news is that these resumes are at least a decade old in most cases, that's still a lot of individuals' details — OWASP boasts "tens of thousands of members" across more than 250 chapters worldwide.

Presumably, at least some of these people still have the same identifiers that they did back in 2014, which can now be used for identity fraud and other nefarious purposes if they end up in the wrong hands or in a dark web database dump.

If you have the same email address or phone number, for example, OWASP urges caution when answering or receiving unsolicited emails and calls. These could lead to phishing attempts and financial crimes.

According to the open source community, it no longer collects resumes as part of its membership application process and now uses two-factor authentication to protect member data.

To make sure this doesn't happen again, OWASP said it disabled directory browsing and checked the web server for additional configuration and security issues, and removed all of the resumes from the site.

Additionally, the foundation purged the CloudFlare caches, and requested that the accessed data be removed from the web archive.

While OWASP is attempting to notify affected individuals via email, the age of the resumes, between 10 and 18 years, makes it more difficult.

"Regardless, we will contact the email addresses discovered during our investigations," it pledged. ®