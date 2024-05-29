<!--yml
category: 未分类
date: 2024-05-27 14:55:27
-->

# sPACE Attack: Spoofing eID’s Password Authenticated Connection Establishment | by CtrlAlt | Medium

> 来源：[https://ctrlalt.medium.com/space-attack-spoofing-eids-password-authenticated-connection-establishment-11561e5657b1](https://ctrlalt.medium.com/space-attack-spoofing-eids-password-authenticated-connection-establishment-11561e5657b1)

## Proof of Fraudulent Bank Account Opening

To demonstrate the practical exploitability of the vulnerability, a bank account has been successfully opened in the name of a victim at a major German bank. As a result, the attacker has access to the newly established bank account owned by the victim. It is important to note that the victim used a smartphone updated to the latest version with all patches installed and followed all security recommendations provided by the BSI. No physical access or remote code execution was required.

> Note: The author implemented precautions to ensure that no other users were impacted by the exploit. However, it is important to understand that this exploit had the potential to affect every user with the app installed if the author had chosen to do so. Furthermore, the victim agreed in advance to the attack, to avoid legal issues.

The victim is a German citizen who owns an iPhone and a German ID card equipped with activated eID functionality. The victim intends to use the eID to access the official portal of the Arbeitsagentur (refer to [https://www.arbeitsagentur.de/](https://www.arbeitsagentur.de/)). To do so, the victim visited the official Apple app store and downloaded the official AusweisApp.

Victim downloading the legitimate AusweisApp from the App Store.

The victim opened [https://www.arbeitsagentur.de/](https://www.arbeitsagentur.de/) on the smartphone and initiated the login process for the Arbeitsagentur.

> Note: The service the victim attempts to access is not important. Any service that initiates the link to the AusweisApp would have triggered the attack. As demonstrated in the research paper, phishing attacks are also effective.

At a certain point of the login process, there is a redirection (“deeplink”) to the AusweisApp, where the eID authentication is intended to take place. As show in the research paper, both Apple’s and Google’s deeplink behavior, combined with the approach the eID uses for deeplinks, can be utilized by an attacker to redirect to another app.

As a result of this deeplink vulnerability in the eID, an app containing a modified eID kernel, uploaded to the official app store by the attacker, was triggered. Importantly, this does not require any permissions, no remote code execution or physical access.

The next screen shows the triggering of the “AusweisApp”. It is important to note that this is not the legitimate AusweisApp downloaded just a minute ago, but an app owned by the author, including a modified eID kernel.

Victim being redirect to the malicious app.

The victim is now within the malicious app, performing an eID identification. **This eID process takes a bit longer than normal.** In the background, the modified eID kernel executed the attack, redirecting the APDUs and the PIN to the attacker.

Victim using the eID. In the background, the PIN / APDUs are redirected and the bank account is opened.

Following the attack, the attacker allows the victim to complete the intended eID login for the Arbeitsagentur. The victim successfully accesses the Arbeitsagentur and remains unaware to the just executed attack.

On the attacker’s side, the attacker initiated the system to listen for incoming connections from victims. Upon the victim’s initiation of the modified eID kernel within the malicious app, a connection is established to the attacker’s system. The attacker system responds to this connection by initiating the bank account opening process at the bank.

> Note: The name of the affected bank will not be disclosed, as the specific bank is not important. Any relying party using the eID system is vulnerable to this attack.

The attacker’s system waiting for the connection from the victim.

The attacker extracted data from the eID using the “Selbstauskunft” and utilized this information to complete and submit the account opening form at the bank.

The attacker’s system proceeded with the identification process for the bank account opening. This action once again activated the AusweisApp on the attacker’s system. The eID of the victim was accessed for a second time, enabling the identification for the bank account opening. Finally, the process finished, granting the attacker access to the newly created bank account associated with the victim.

**The attacker now has access to a bank account which belongs to the victim.**