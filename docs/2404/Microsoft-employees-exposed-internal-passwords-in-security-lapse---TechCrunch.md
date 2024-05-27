<!--yml
category: 未分类
date: 2024-05-27 13:05:52
-->

# Microsoft employees exposed internal passwords in security lapse | TechCrunch

> 来源：[https://techcrunch.com/2024/04/09/microsoft-employees-exposed-internal-passwords-security-lapse/](https://techcrunch.com/2024/04/09/microsoft-employees-exposed-internal-passwords-security-lapse/)

Microsoft has resolved a security lapse that exposed internal company files and credentials to the open internet.

Security researchers Can Yoleri, Murat Özfidan and Egemen Koçhisarlı with SOCRadar, a cybersecurity company that helps organizations find security weaknesses, discovered an open and public storage server hosted on Microsoft’s Azure cloud service that was storing internal information relating to Microsoft’s Bing search engine.

The Azure storage server housed code, scripts and configuration files containing passwords, keys and credentials used by the Microsoft employees for accessing other internal databases and systems.

But the storage server itself was not protected with a password and could be accessed by anyone on the internet.

Yoleri told TechCrunch that the exposed data could potentially help malicious actors identify or access other places where Microsoft stores its internal files. Identifying those storage locations “could result in more significant data leaks and possibly compromise the services in use,” Yoleri said.

The researchers notified Microsoft of the security lapse on February 6, and Microsoft secured the spilling files on March 5.

When reached by email, a spokesperson for Microsoft did not provide comment by the time of publication. In a statement shared after publication on Wednesday, Microsoft’s Jeff Jones told TechCrunch: “Though the credentials should not have been exposed, they were temporary, accessible only from internal networks, and disabled after testing. We thank our partners for responsibly reporting this issue.”

Jones did not say for how long the cloud server was exposed to the internet, or if anyone other than SOCRadar discovered the exposed data inside.

This is the latest security gaffe at Microsoft as the company tries to rebuild trust with its customers after a series of cloud security incidents in recent years. In a similar security lapse last year, researchers found that [Microsoft employees were exposing their own corporate network logins](https://www.vice.com/en/article/m7gb43/microsoft-employees-exposed-login-credentials-azure-github) in code published to GitHub.

Microsoft also came under fire last year after the company admitted it did not know [how China-backed hackers stole an internal email signing key](https://techcrunch.com/2023/09/08/microsoft-hacker-china-government-storm-0558/) that allowed the hackers broad access to Microsoft-hosted inboxes of senior U.S. government officials. An independent board of cyber experts tasked with investigating the email breach wrote in their report, published last week, that the hackers succeeded because of a “cascade of security failures at Microsoft.”

In March, Microsoft said that [it continues to counter an ongoing cyberattack](https://techcrunch.com/2024/03/08/microsoft-ongoing-cyberattack-russia-apt-29/) that allowed Russian state-backed hackers to steal portions of the company’s source code and internal emails from Microsoft corporate executives.

*Updated with comment from Microsoft.*

> [Microsoft reveals how hackers stole its email signing key… kind of](https://techcrunch.com/2023/09/08/microsoft-hacker-china-government-storm-0558/)