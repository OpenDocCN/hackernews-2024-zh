<!--yml
category: 未分类
date: 2024-05-27 15:22:42
-->

# Volt Typhoon's botnet infected 'hundreds' of older routers • The Register

> 来源：[https://www.theregister.com/2024/01/31/volt_typhoon_botnet/](https://www.theregister.com/2024/01/31/volt_typhoon_botnet/)

China's Volt Typhoon spies infected "hundreds" of outdated Cisco and Netgear equipment with malware so that the devices could be instructed to break into US critical infrastructure facilities, the Justice Department has said.

On Tuesday [news broke](https://www.theregister.com/2024/01/30/fbi_china_volt/) that the Feds had blocked the malicious bot network that was set up on end-of-life, US-based small office/home office routers. Now more details have come out about how an FBI team infiltrated the operation and harvested crucial data before remotely wiping the KV Botnet, according to four warrants ([5018](https://lnks.gd/l/eyJhbGciOiJIUzI1NiJ9.eyJidWxsZXRpbl9saW5rX2lkIjoxMDIsInVyaSI6ImJwMjpjbGljayIsInVybCI6Imh0dHBzOi8vd3d3Lmp1c3RpY2UuZ292L29wYS9tZWRpYS8xMzM2NDIxL2RsP2lubGluZT0mdXRtX21lZGl1bT1lbWFpbCZ1dG1fc291cmNlPWdvdmRlbGl2ZXJ5IiwiYnVsbGV0aW5faWQiOiIyMDI0MDEzMS44OTQzMDk5MSJ9.4LRHVZyUDYfMnRWC7s_-GUsAOhl4SewCG7GXGj4tAX8/s/3078670442/br/236285728656-l), [5530](https://lnks.gd/l/eyJhbGciOiJIUzI1NiJ9.eyJidWxsZXRpbl9saW5rX2lkIjoxMDMsInVyaSI6ImJwMjpjbGljayIsInVybCI6Imh0dHBzOi8vd3d3Lmp1c3RpY2UuZ292L29wYS9tZWRpYS8xMzM2NDE2L2RsP2lubGluZT0mdXRtX21lZGl1bT1lbWFpbCZ1dG1fc291cmNlPWdvdmRlbGl2ZXJ5IiwiYnVsbGV0aW5faWQiOiIyMDI0MDEzMS44OTQzMDk5MSJ9.8vmYc17z664gSNfykf8ncM2yRyRspFYF6SCIHiTZhb8/s/3078670442/br/236285728656-l), [5451](https://lnks.gd/l/eyJhbGciOiJIUzI1NiJ9.eyJidWxsZXRpbl9saW5rX2lkIjoxMDQsInVyaSI6ImJwMjpjbGljayIsInVybCI6Imh0dHBzOi8vd3d3Lmp1c3RpY2UuZ292L29wYS9tZWRpYS8xMzM2NDA2L2RsP2lubGluZT0mdXRtX21lZGl1bT1lbWFpbCZ1dG1fc291cmNlPWdvdmRlbGl2ZXJ5IiwiYnVsbGV0aW5faWQiOiIyMDI0MDEzMS44OTQzMDk5MSJ9.d-xmdG8esN2ZFRwqLXHNvwIo6ZpVS-2HQ5js3wwfJo8/s/3078670442/br/236285728656-l) and [5432](https://lnks.gd/l/eyJhbGciOiJIUzI1NiJ9.eyJidWxsZXRpbl9saW5rX2lkIjoxMDUsInVyaSI6ImJwMjpjbGljayIsInVybCI6Imh0dHBzOi8vd3d3Lmp1c3RpY2UuZ292L29wYS9tZWRpYS8xMzM2NDExL2RsP2lubGluZT0mdXRtX21lZGl1bT1lbWFpbCZ1dG1fc291cmNlPWdvdmRlbGl2ZXJ5IiwiYnVsbGV0aW5faWQiOiIyMDI0MDEzMS44OTQzMDk5MSJ9.xD5SrU9r17bUh7t5QaI4TWGGTTTlIfn6D8a3QecwFJY/s/3078670442/br/236285728656-l)) filed by the FBI in a southern Texas court last month and released today.

"China's hackers are targeting American civilian critical infrastructure, pre-positioning to cause real-world harm to American citizens and communities in the event of conflict," FBI Director Christopher Wray said in a [statement](https://www.justice.gov/usao-sdtx/pr/us-government-disrupts-botnet-peoples-republic-china-used-conceal-hacking-critical). "Volt Typhoon malware enabled China to hide as they targeted our communications, energy, transportation, and water sectors."

The Feds claim the Middle Kingdom's cyber-spies downloaded a virtual private network module to the vulnerable routers and set up an encrypted communication channel to remotely control the botnet, and potentially order the devices to carry out attacks as well as hide their activities. Specifically: Volt Typhoon used the US-based routers and IP addresses to [target](https://www.theregister.com/2024/01/31/critical_infrastructure_hacking/) US critical infrastructure, we're told.

The warrants allowed law enforcement to remotely install software on the routers to search for, and then seize or copy, information about the illicit activity before wiping the malware from the compromised devices.

To do this — and to limit the Feds' search to routers infected with the remote-control botnet malware — the FBI sent specific KV Botnet commands to compromised routers to collect "non-content information about those nodes," according to the warrants.

This includes the IP address, port numbers used by infected routers to communicate with other nodes, as well as IP addresses and ports used by each node's parent, and data on the command-and-control nodes.

"A router that is not infected by the KV Botnet malware would not receive or respond to this command," court documents claim. 

The Feds, along with foreign agency partners in Five Eyes nations, [first warned](https://www.theregister.com/2023/05/25/china_volt_typhoon_attacks/) about this threat in May 2023.

Also today, the US government's cybersecurity agency and the FBI [issued an alert](https://www.cisa.gov/news-events/alerts/2024/01/31/cisa-and-fbi-release-secure-design-alert-urging-manufacturers-eliminate-defects-soho-routers) urging manufacturers to eliminate defects in SOHO router web management interfaces. This, according to the Feds, includes automating update capabilities, locating the web management interface on LAN-side ports only, and requiring an explicit manual override to remove security settings. ®