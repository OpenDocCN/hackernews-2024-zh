<!--yml
category: 未分类
date: 2024-05-27 15:13:55
-->

# GitLab password reset bug leaves more than 5.3K servers up for grabs | SC Media

> 来源：[https://www.scmagazine.com/news/gitlab-password-reset-bug-leaves-more-than-5-3k-servers-up-for-grabs](https://www.scmagazine.com/news/gitlab-password-reset-bug-leaves-more-than-5-3k-servers-up-for-grabs)

[Critical GitLab vulnerability CVE-2023-7028](https://www.scmagazine.com/news/gitlab-vulnerability-risks-account-takeover-via-simple-password-reset) was not patched on more than 5,300 servers as of Tuesday, potentially enabling remote takeover of software developers’ accounts.

The bug, with a maximum CVSS score of 10, [was first disclosed and patched by GitLab](https://about.gitlab.com/releases/2024/01/11/critical-security-release-gitlab-16-7-2-released/) on Jan. 11\. The vulnerability in GitLab’s login system would allow an attacker to have a password reset link sent to their own unverified email address without any user interaction by the victim.

“Account takeover can be achieved by crafting a specially formatted HTTP request that is capable of sending a password reset email to an unverified email address in an unpatched version,” a GitLab spokesperson told SC Media in a Jan. 12 email.

Security updates were released for GitLab versions 16.5.6, 16.6.4 and 16.7.2 and backported to versions 16.1.6, 16.2.9, 16.3.7 and 16.4.5, as well.

One researcher who tested the bug on GitLab Community Edition version 16.6.1 and [shared their results on AttackerKB](https://attackerkb.com/topics/VBDvNxhyjr/cve-2023-7028) said CVE-2023-7028 was, “Very effective and easy to exploit.”

Nearly two weeks after patches became available, [5,379 vulnerable instances of GitLab](https://dashboard.shadowserver.org/statistics/combined/map/?map_type=std&day=2024-01-23&source=http_vulnerable&source=http_vulnerable6&tag=cve-2023-7028%2B&geo=all&data_set=count&scale=log) were detected worldwide by the Shadowserver Foundation. The nonprofit organization, which monitors malicious activity online, [posted the data from Jan. 23 on X](https://twitter.com/Shadowserver/status/1750115947430416434), noting that the United States and Germany had the most vulnerable instances, with 964 and 730 respectively.

[Shadowserver’s dashboard tool](https://dashboard.shadowserver.org/statistics/combined/time-series/?date_range=7&source=http_vulnerable&source=http_vulnerable6&tag=cve-2023-7028%2B&group_by=geo&style=stacked) showed fewer vulnerable instances (4,652) on Jan. 24\. A Shadowserver spokesperson confirmed with SC Media that there was "a drop in detected vulnerable instances, which is a positive development," but that more time would be needed to determine whether this decrease was a trend or a "blip" in Shadowserver's scans.

## Indicators of compromise for GitLab CVE-2023-7028

GitLab customers with self-managed instances of the affected products — GitLab Community Edition and GitLab Enterprise Edition — should review their logs for exploitation of CVE-2023-7028 using the two following methods, as outlined by GitLab:

*   Check gitlab-rails/production_json.log for HTTP requests to the /users/password path with params.value.email consisting of a JSON array with multiple email addresses
*   Check gitlabs-rails/audit_json.log for entries with meta.caller.id of PasswordsController#create and target_Details consisting of a JSON array with multiple email addresses

The company said no exploitation of the bug on GitLab.com or GitLab Dedicated instances has been detected.

GitLab recommended customers also enable two-factor authentication (2FA), which prevents account takeover via CVE-2023-7028, although users of unpatched instances are still vulnerable to being locked out of their accounts if an attacker exploits the flaw to reset their password.