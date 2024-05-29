<!--yml
category: 未分类
date: 2024-05-27 15:23:14
-->

# ICANN proposes creating .INTERNAL domain • The Register

> 来源：[https://www.theregister.com/2024/01/29/icann_internal_tld/](https://www.theregister.com/2024/01/29/icann_internal_tld/)

The Internet Corporation for Assigned Names and Numbers (ICANN) has proposed creating a new top-level domain (TLD) and never allowing it to be delegated in the global domain name system (DNS) root.

The proposed TLD is .INTERNAL and, as the name implies, it's intended for internal use only. The idea is that .INTERNAL could take on the same role as the 192.168.x.x IPv4 bloc – available for internal use but never plumbed into DNS or other infrastructure that would enable it to be accessed from the open internet.

ICANN's Security and Stability Advisory Committee (SSAC) [advised](https://itp.cdn.icann.org/en/files/security-and-stability-advisory-committee-ssac-reports/sac-113-en.pdf) the development of such a TLD in 2020\. It noted at the time that "many enterprises and device vendors make ad hoc use of TLDs that are not present in the root zone when they intend the name for private use only. This usage is uncoordinated and can cause harm to Internet users" – in part by forcing DNS servers to handle, and reject, queries for domains only used internally.

DNS, however, can't prevent internal use of *ad hoc* TLDs. So the SSAC recommended creation of a TLD that would be explicitly reserved for internal use.

A consultation process produced 35 candidate strings, each of which was checked to ensure it wasn't already a TLD, and for "potential for confusing similarity, for length, and for its capacity to be memorable and meaningful." Assessments were conducted for all six United Nations languages: Arabic, Chinese, English, French, Russian and Spanish. That process saw many candidates "deemed unsuitable due to their lack of meaningfulness."

For example, .DOMAIN was binned because it was felt not to "convey that its purpose is specifically for private-use applications."

After years of debate, ICANN and other internet governance orgs were left with two viable candidates: .PRIVATE and .INTERNAL.

Last Thursday, ICANN [announced](https://itp.cdn.icann.org/en/files/root-system/identification-tld-private-use-24-01-2024-en.pdf) [PDF] that .INTERNAL was its choice.

.PRIVATE lost out because assessors felt it "may carry the unintended imputation of privacy to a higher degree, and more potential was seen for conflicting meanings across the gamut of assessed languages."

ICANN's board still has to sign off the creation of .INTERNAL. But if you want to get ahead of the pack, there's nothing stopping you. Indeed, some outfits already use *ad hoc* TLDs. Open source Wi-Fi firmware project WRT has used .LAN, and networking vendor D-Link has employed .dlink.

There's nothing stopping you doing likewise.

But as ICANN's [proposal](https://itp.cdn.icann.org/en/files/security-and-stability-advisory-committee-ssac-reports/sac-113-en.pdf) for the idea noted: "Operators who choose to use private namespaces of the kind proposed in this document should understand the potential for that decision to have corresponding costs, and that those costs might well be avoided by choosing instead to use a sub-domain of their own publicly registered domain name."

Which is why *The Register* loves the standards process. ®