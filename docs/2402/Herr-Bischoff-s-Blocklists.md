<!--yml
category: 未分类
date: 2024-05-29 13:20:59
-->

# Herr Bischoff's Blocklists

> 来源：[https://ipbl.herrbischoff.com/](https://ipbl.herrbischoff.com/)

# Herr Bischoff's Blocklists

Feel free to use the following lists without restriction and without guarantees or warranties of any kind. I will do my best to keep it updated but that's about it. They are refreshed at the given intervals, so fetching more often than that just wastes resources. Excessive queries will eventually get you blocked but usually not added to the lists themselves. Just be kind and considerate and everything else will follow.

All lists contain IPv4 as well as IPv6 addresses in CIDR notation, dropping the /32 and /128 suffixes for single addresses.

* * *

## IPBL

It contains IPs that were caught making requests they were not supposed to make. This includes, among other signals:

*   attempting to aggressively probe servers
*   attempting to exploit known vulnerabilities
*   being a malware bootstrapping source
*   making repeated automated login attempts
*   repeatedly ignoring robots.txt directives
*   running an unidentified crawler
*   running brain-dead scanning scripts
*   running port scans
*   sending junk email

```
https://ipbl.herrbischoff.com/list.txt (Updated hourly)
```

* * *

## DROP

All Spamhaus DROP lists consolidated as IPs. It's useful for scripts or firewalls that don't have an ASN to IP range converter built-in and you just don't want to bother with writing your own. It contains both IPv4 and IPv6 addresses, so you will either have to use it as-is or filter it yourself.

See [https://www.spamhaus.org/faq/section/Spamhaus%C2%A0DROP#218](https://www.spamhaus.org/faq/section/Spamhaus%C2%A0DROP#218) for original source information about updates. Excessive queries will eventually get you blocked. Just be kind and considerate and everything else will follow.

```
https://ipbl.herrbischoff.com/drop.txt (Updated daily)
```

* * *

## BADASN

The ASNs these IPs belong to are beyond saving. They have significant issues with abusive traffic and content. Some are actively involved in contact scraping. The operators are either unable to manage it or simply don't care. Likely, there's some incentive involved for them to not care or they are flat-out spam operators themselves. Some may even be bullet-proof hosters for despicable material and actions. Anyway, they need to be blocked by default and I do so for the infrastructure I'm responsible for.

```
AS29119     ServiHosting
AS36352     ColoCrossing
AS43097     JSC
AS48666     ASN allocation status is Reserved (Potentially a Bogon)
AS50360     Tamatiya EOOD
AS51088     A2B IP B.V.
AS58110     IP Volume LTD
AS62904     Eonix Corporation
AS64496     ASN allocation status is Reserved (Potentially a Bogon)
AS202425    IP Volume inc
AS202685    Aggros Operations Ltd.
AS203168    Constant MOULIN
AS204007    Kleissner Investments s.r.o.
AS206092    IPXO LIMITED
AS207959    XSServer GmbH
AS209298    Online Marketing Sources Kft.
AS210558    1337 Services GmbH
AS211632    Internet Solutions & Innovations LTD.
AS212370    PEENQ.NL
AS213371    ABC Consultancy (Squitter Networks)
AS397630    Blazing SEO, LLC
AS397702    1776 Solutions
AS400328    Intelligence Hosting LLC
```

This is an opinionated list. Then again, I have zero tolerance for spam, CSAM, racism, supremacy delusions of any kind, sexism, nazism, and a lot more "isms".

```
https://ipbl.herrbischoff.com/badasn.txt (Updated daily)
```

* * *

## TOR

This is a list of known Tor nodes. It is compiled from several sources, cleaned up and consolidated.

```
https://ipbl.herrbischoff.com/tor.txt (Updated every 6 hours)
```

* * *

## AI

A list of known IP ranges used by companies creating LLMs (large language models) via machine learning. Their scraping is usually done with full disregard of copyright and other established norms, effectively selling you back your own content.

```
https://ipbl.herrbischoff.com/ai.txt (Updated sporadically)
```

* * *

## GEOIP

These have a thousand and one uses but are surprisingly hard to come by in bulk.

*   IPv4 and IPv6 ranges in one file.
*   Compiled from alternative data sources, in an attempt to improve accuracy.
*   Deduplicated and subnets merged for efficiency.
*   Simplified and sorted by country TLD.
*   CIDR notation.

```
https://ipbl.herrbischoff.com/geoip/ (Updated daily)
```

* * *

## Notes

There is no de-listing process for any of the lists. If your IP is on it, adjust your server's behaviour and eventually it will disappear.

The lists are only available via HTTPS. The server supports TLSv1.2+, so make sure your infrastructure is up-to-date. If you observe any issues with it, please let me know:

```
printf "%s@%s.%s\n" marcel herrbischoff com
```

* * *

Operating since 2021.