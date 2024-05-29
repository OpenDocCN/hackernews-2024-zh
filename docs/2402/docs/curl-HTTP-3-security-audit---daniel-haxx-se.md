<!--yml
category: 未分类
date: 2024-05-29 13:18:59
-->

# curl HTTP/3 security audit | daniel.haxx.se

> 来源：[https://daniel.haxx.se/blog/2024/02/23/curl-http-3-security-audit/](https://daniel.haxx.se/blog/2024/02/23/curl-http-3-security-audit/)

An external security audit focused especially on curl’s HTTP/3 components and associated source code was recently concluded by [Trail of Bits](https://www.trailofbits.com/). In particular on the HTTP/3 related curl code that uses and interfaces the ngtcp2 and nghttp3 libraries, as that is so far the only HTTP/3 backend in curl that is not labeled as experimental. The audit was sponsored by [the Sovereign Tech Fund](https://www.sovereigntechfund.de/) via [OSTIF](https://ostif.org/).

The audit **revealed no major discoveries or security problems** but led to improved fuzzing and a few additional areas are noted as suitable to improve going forward. Maybe in particular in the fuzzing department. (If you’re looking for somewhere to contribute to curl, there’s your answer!)

The audit revealed that we had accidentally drastically shrunk the fuzzing coverage a while back without even noticing – which we of course immediately rectified. When fixed, we fortunately did not get an explosion in issues (phew!), which thus confirmed that we had not messed up in any particular way while the fuzzing ability had been limited. But still: several man weeks of professional code inspection and no serious flaws were detected. I am thrilled over this fact.

Because of curl’s use of third party libraries for doing QUIC and HTTP/3, the report advises that there should be follow-up audits of the involved libraries. Fair proposal, but that is of course something that is beyond what we as a project can do.

Trail of Bits is professional and a pleasure to work with. Now having done it twice, I have nothing but good things to say about the team we have worked with.

From curl’s side, I would like to also highlight and thank Stefan Eissing and Dan Fandrich for participating in the process.

**The full report is available on the curl website, [here](https://curl.se/docs/audits.html).**

## The third

This is (quite fittingly since it is for HTTP/3) the third external security audit performed on curl source code, even if this was more limited in scope than the previous ones done in 2016 and 2022\. Quite becomingly, the amount of detected important issues have decreased for every new audit. We love scrutiny and we take security seriously. I think this shows in the audit reports.

## Related

[OSTIF’s blog about the audit](https://ostif.org/curl-audit-complete/).

## Image

The top image is a mashup of the official curl logo and the official IETF HTTP/3 logo. Done by me.