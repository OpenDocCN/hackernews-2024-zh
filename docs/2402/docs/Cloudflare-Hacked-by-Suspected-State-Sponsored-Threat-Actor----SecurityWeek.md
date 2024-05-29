<!--yml
category: 未分类
date: 2024-05-27 14:35:21
-->

# Cloudflare Hacked by Suspected State-Sponsored Threat Actor  - SecurityWeek

> 来源：[https://www.securityweek.com/cloudflare-hacked-by-suspected-state-sponsored-attacker/](https://www.securityweek.com/cloudflare-hacked-by-suspected-state-sponsored-attacker/)

**Web security company Cloudflare on Thursday revealed that a threat actor used stolen credentials to gain access to some of its internal systems.**

The incident was discovered on November 23, nine days after the threat actor, believed to be state-sponsored, used credentials compromised [in the October 2023 Okta hack](https://www.securityweek.com/okta-broadens-scope-of-data-breach-all-customer-support-users-affected/) to access Cloudflare’s internal wiki and bug database.

The stolen login information, an access token and three service account credentials, were not rotated following the Okta incident, allowing the attackers to probe and perform reconnaissance of Cloudflare systems starting November 14, [the security firm explains](https://blog.cloudflare.com/thanksgiving-2023-security-incident).

According to Cloudflare, the attackers managed to access an AWS environment, as well as Atlassian Jira and Confluence, but network segmentation prevented them from accessing its Okta instance and the Cloudflare dashboard.

With access to the Atlassian suite, the threat actor started looking for information on the Cloudflare network, searching the wiki for “things like remote access, secret, client-secret, openconnect, cloudflared, and token”. In total, 36 Jira tickets and 202 wiki pages were accessed.

On November 16, the attackers created an Atlassian account to gain persistent access to the environment, and on November 20 returned to verify that they still had access.

On November 22, the threat actor installed the Sliver Adversary Emulation Framework, gaining persistent access to the Atlassian server, which was then used to move laterally. They attempted to access a non-production console server at a São Paulo, Brazil, data center that is not yet operational.

The attackers viewed 120 code repositories and downloaded 76 of them to the Atlassian server, but did not exfiltrate them.

Advertisement. Scroll to continue reading.

“The 76 source code repositories were almost all related to how backups work, how the global network is configured and managed, how identity works at Cloudflare, remote access, and our use of Terraform and Kubernetes. A small number of the repositories contained encrypted secrets which were rotated immediately even though they were strongly encrypted themselves,” Cloudflare notes.

The attackers used a Smartsheet service account to access Cloudflare’s Atlassian suite, and the account was terminated on November 23, within 35 minutes after the unauthorized access was identified. The user account created by the attacker was found and deactivated 48 minutes later.

Cloudflare says it also put in place firewall rules to block the attackers’ known IP addresses and that the Sliver Adversary Emulation Framework was removed on November 24.

“Throughout this timeline, the threat actor tried to access a myriad of other systems at Cloudflare but failed because of our access controls, firewall rules, and use of hard security keys enforced using our own Zero Trust tools,” the company says.

According to Cloudflare, it found no evidence that the threat actor accessed its global network, customer database, configuration information, data centers, SSL keys, workers deployed by customers, or any other information except from the “data in the Atlassian suite and the server on which our Atlassian runs”.

On November 24, Cloudflare also tasked numerous technical employees with improving security and validating that the threat actor could no longer access the company’s systems.

“Additionally, we continued to investigate every system, account and log to make sure the threat actor did not have persistent access and that we fully understood what systems they had touched and which they had attempted to access,” the company notes.

According to Cloudflare, more than 5,000 individual production credentials were rotated following the incident, close to 5,000 systems were triaged, test and staging systems were physically segmented, and every machine within the Cloudflare global network was reimaged and rebooted.

The equipment at the São Paulo data center, although not accessed, was sent back to the manufacturers for inspection and replaced although no evidence of compromise was discovered.

The goal of the attack, Cloudflare says, was to obtain information on the company’s infrastructure, likely to gain a deeper foothold. CrowdStrike performed a separate investigation into the incident, but discovered no evidence of additional compromise.

“We are confident that between our investigation and CrowdStrike’s, we fully understand the threat actor’s actions and that they were limited to the systems on which we saw their activity,” Cloudflare notes.

**Related:** [GitHub Rotates Credentials in Response to Vulnerability](https://www.securityweek.com/github-rotates-credentials-in-response-to-vulnerability/)

**Related:** [Mandiant Details How Its X Account Was Hacked](https://www.securityweek.com/mandiant-details-crypto-theft-campaign-that-hacked-its-x-account-via-brute-force-attack/)

**Related:** [Okta Hack Blamed on Employee Using Personal Google Account on Company Laptop](https://www.securityweek.com/okta-hack-blamed-on-employee-using-personal-google-account-on-company-laptop/)