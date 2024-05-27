<!--yml
category: 未分类
date: 2024-05-27 13:17:22
-->

# OpenJS: “XZ Utils Cyberattack Likely Not an Isolated Incident” - Socket

> 来源：[https://socket.dev/blog/openjs-xz-utils-cyberattack-likely-not-an-isolated-incident](https://socket.dev/blog/openjs-xz-utils-cyberattack-likely-not-an-isolated-incident)

OpenJS is warning open source project maintainers to be vigilant against social engineering takeover attempts after receiving one targeting the organization.

“The recent attempted [XZ Utils backdoor](https://openssf.org/blog/2024/03/30/xz-backdoor-cve-2024-3094/) ([CVE-2024-3094](https://www.cve.org/CVERecord?id=CVE-2024-3094)) may not be an isolated incident as evidenced by a similar credible takeover attempt intercepted by the OpenJS Foundation, home to JavaScript projects used by billions of websites worldwide,” OpenJS Foundation Executive Director Robin Bender said in a joint [statement](https://openjsf.org/blog/openssf-openjs-alert-social-engineering-takeovers) with the [OpenSSF](https://openssf.org/blog/2024/04/15/open-source-security-openssf-and-openjs-foundations-issue-alert-for-social-engineering-takeovers-of-open-source-projects/).

The credible takeover attempt involved a series of emails requesting OpenJS take action to update one of its popular JavaScript projects to “address any critical vulnerabilities,” from various GitHub-associated email addresses where the author requested to be designated as a new maintainer. OpenJS noted the similarities between this approach and the one used by “Jia Tan” to add the XZ/liblzma backdoor.

The foundation published an extensive list of tips for recognizing these types of threats and suspicious patterns, and steps to secure open source projects and critical infrastructure.

## Open Source Security Needs a Systemic Overhaul[#](#Open-Source-Security-Needs-a-Systemic-Overhaul)

The security industry is still rocked by the potential devastation posed by the [xz-utils backdoor incident](https://socket.dev/blog/how-to-use-socket-to-find-out-if-you-were-affected-by-the-backdoored-xz-package), an attempt that was foiled by sheer luck after its unlikely discovery. This isn’t a strategy for securing open source software at scale. The ecosystem is living on borrowed time until the next incident, which may already be in progress.

Experts who are thinking about the big picture want to shift away from these sporadic reactive measures towards a more realistic and proactive security strategy. The community is slowly moving to think of OSS in terms of infrastructure, which requires a sustained and systemic change that integrates security as a foundational component. This is a difficult problem to solve.

In a [discussion](https://fosstodon.org/@camdoncady@infosec.exchange/112278085619243206) on Mastodon, US Airfare engineer Camdon Cady likened the current challenges facing existing physical infrastructure to the security issues within the Linux ecosystem:

> In the US, we recently had a ship hit a bridge, causing the bridge to collapse. The technical fix (collision protection around the bridge pylons) has been known for decades, and is widely implemented on new bridges. Bumpers have not been backported to all existing bridges.

> I'm wondering if perhaps the Linux ecosystem is actually a bunch of aging infrastructure. If so, that fact might be obscured by seemingly healthy contributions to the Linux kernel. There's a ton of other "basic infrastructure" that gets shipped alongside the kernel, though: coreutils, binutils, gcc, make, autotools, cmake, zlib, xz, etc etc etc.

Information security expert Kurt Seifried [noted](https://fosstodon.org/@kurtseifried@infosec.exchange/112278322554923641) that although the OSS community knew the Linux kernel and all the utilities surrounding it would be widely used, "we didn’t think everybody’s car and lightbulbs would be running it.” OSS has become the invisible infrastructure powering every modern device and system, from household appliances to critical national infrastructure.

A recent [report](https://www.atlanticcouncil.org/in-depth-research-reports/report/open-source-software-as-infrastructure/) drafted in collaboration with the Open Source Policy Network and a network of OSS developers and maintainers, contends that OSS, like physical infrastructure, should be strengthened to be sustainable with widespread use, to prevent it from collapsing under catastrophic security risks. The report does not reflect a belief that OSS is inherently insecure, but highlights its parallel with infrastructure that necessitates a government role for the long-term health of the ecosystem.

> When policy focuses only on terrible, potential outcomes, its ideas tend to reflect that bias toward fear, but this need not be the framing for OSS. Open source enables and solves much more than it imperils. Its security is as much a guarantor of continued value to users large and small, from individuals to national intelligence agencies, as it is a bulwark against malicious intent.

> While OSS has come back to attention as an issue of national policy in the European Union (EU), and indeed become one for the first time in the United States in some ways as a product of fear and calamity, opportunities run much deeper. Infrastructure of such scale and magnitude is supported, reinforced, and amplified—not fixed in a brief whirlwind of activity—much like the consistent provisions of clean water, roads and bridges, and healthy capital markets.

It’s not surprising that the OpenJS foundation and the JavaScript ecosystem, which is used by over 95% of all websites, would be the next target of threat actors looking to be designated as maintainers on important projects. The unfortunate reality is that these kinds of malicious attempts get published to public registries at an astonishing rate. Our team at Socket [catches 100+ software supply chain attacks](https://twitter.com/feross/status/1780032494315991404) in npm, PyPI, and Go *every week*.

Software engineer Andrew Nesbitt has been [playing around with the concept of a "blast radius"](https://fosstodon.org/@andrewnez@mastodon.social/112271839907501807) for open source security advisories on [Ecosyste.ms](https://advisories.ecosyste.ms/advisories?order=desc&sort=blast_radius). He uses the CVSS score of a security advisory multiplied by the number of repositories that depend upon that package to determine the “blast radius.” Looking at it this way, Nesbitt concludes that a moderate vulnerability in a very popular library has a potentially larger impact than a critical CVE in an unpopular one.

This experiment on [Ecosyste.ms](http://Ecosyste.ms) highlights the deeply interconnected relationship between open source packages and their dependencies, and how quickly security incidents can propagate throughout the ecosystem, affecting numerous projects and applications even from a single point of failure.

The approach of treating OSS merely as a free resource, instead of a critical protected asset, is proving inadequate in the face of increasingly sophisticated cyber threats across the rapid expansion of OSS. These analogies to traditional physical infrastructure are the seeds of a mandate for systemic overhaul.

OpenJS is raising alarms about the threats that came across their desk, but the broader challenge lies in identifying and mitigating the supply chain attacks that remain undetected. Securing the development process from the ground up is where it starts. Developers should take this warning to heart and rigorously audit their existing packages to ensure they don’t bring in potentially malicious dependencies or carelessly trust packages or maintainers that have historically been safe.