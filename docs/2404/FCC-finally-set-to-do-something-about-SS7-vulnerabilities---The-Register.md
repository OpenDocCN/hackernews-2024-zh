<!--yml
category: 未分类
date: 2024-05-27 12:53:29
-->

# FCC finally set to do something about SS7 vulnerabilities • The Register

> 来源：[https://www.theregister.com/2024/04/02/fcc_ss7_security/](https://www.theregister.com/2024/04/02/fcc_ss7_security/)

The FCC appears to finally be stepping up efforts to secure decades-old flaws in American telephone networks that are allegedly being used by foreign governments and surveillance outfits to remotely spy on and monitor wireless devices.

At issue are the Signaling System Number 7 (SS7) and Diameter protocols, which are used by fixed and mobile network operators to enable interconnection between networks. They are part of the glue that holds today's telecommunications together.

According to the US watchdog and some lawmakers, both protocols include security weaknesses that leave folks vulnerable to unwanted snooping. SS7's problems have been known about for years and years, as far back as at least [2008](https://fahrplan.events.ccc.de/congress/2008/Fahrplan/events/2997.en.html), and we wrote about them [in 2010](https://www.theregister.com/2010/04/22/gsm_info_disclosure_hack/) and [2014](https://www.theregister.com/2014/12/26/ss7_attacks/), for instance. Little has been done to address these exploitable shortcomings.

SS7, which was developed in the mid-1970s, can be potentially [abused](https://www.theregister.com/2022/02/02/whistleblower_nso_group/) to track people's phones' locations; [redirect](https://www.theregister.com/2017/05/03/hackers_fire_up_ss7_flaw/) calls and text messages so that info can be intercepted; and [spy](https://www.theregister.com/2018/06/01/wyden_ss7_stingray_fcc_homeland_security/) on users.

The Diameter protocol was developed in the late-1990s and includes support for network access and IP mobility in local and roaming calls and messages. It does not, however, encrypt originating IP addresses during transport, which makes it easier for miscreants to carry out network spoofing attacks.

"As coverage expands, and more networks and participants are introduced, the opportunity for a bad actor to exploit SS7 and Diameter has increased," according to the FCC [[PDF](https://s3.documentcloud.org/documents/24527269/da-24-308a1.pdf)].

On March 27 the commission asked telecommunications providers to weigh in and detail what they are doing to prevent SS7 and Diameter vulnerabilities from being misused to track consumers' locations.

The FCC has also asked carriers to detail any exploits of the protocols since 2018\. The regulator wants to know the date(s) of the incident(s), what happened, which vulnerabilities were exploited and with which techniques, where the location tracking occurred, and — if known — the attacker's identity.

This time frame is significant because in 2018, the Communications Security, Reliability, and Interoperability Council (CSRIC), a federal advisory committee to the FCC, issued several security best practices to prevent network intrusions and unauthorized location tracking.

Interested parties have until April 26 to [submit comments](https://www.fcc.gov/ecfs/search/search-filings), and then the FCC has a month to respond.

### 'Grave threats posed by carriers' lax security'

The FCC's call for comments comes in response to a request from US Senator Ron Wyden (D-OR) who last month asked that the White House "address the grave threats posed by wireless carriers' lax cybersecurity practices [[PDF](https://s3.documentcloud.org/documents/24527132/wyden-phone-hacking-letter-to-president-biden.pdf)]."

These threats, according to Wyden, are caused by flaws in SS7 and Diameter, and have been abused by "authoritarian governments to conduct surveillance" and obtain people's information.

"America needs to ramp up our defenses against mercenary surveillance companies that help foreign dictators threaten US national security, human rights and journalists working to expose wrongdoing," Wyden said in a statement. "I look forward to working with the FCC to secure America's phone networks through mandatory minimum cybersecurity standards."

This isn't the first time Senator Wyden has demanded the government address vulnerabilities in SS7 — or the first time he's called the protocol flaws a national security issue.

In April 2023, the senator [accused AT&T](https://www.theregister.com/2023/04/12/firstnet_cybersecurity_audit_wyden/) of "concealing vital cybersecurity reporting" about its FirstNet phone network used by first responders and the US military.

In a letter sent to the US government's CISA and NSA, Wyden called for an annual cybersecurity audit of FirstNet because of SS7 misuse.

"These phone network vulnerabilities are being actively exploited to conduct cross-border surveillance," Wyden wrote. ®