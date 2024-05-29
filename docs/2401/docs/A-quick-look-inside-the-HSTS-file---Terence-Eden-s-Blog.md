<!--yml
category: 未分类
date: 2024-05-27 14:30:34
-->

# A quick look inside the HSTS file – Terence Eden’s Blog

> 来源：[https://shkspr.mobi/blog/2024/01/a-quick-look-inside-the-hsts-file/](https://shkspr.mobi/blog/2024/01/a-quick-look-inside-the-hsts-file/)

You type in to your browser's address bar `example.com` and it automatically redirects you to the https:// version. How does your browser know that it needed to request the more secure version of a website?

The answer is... A *big* list. The [HTTP Strict Transport Security](https://en.wikipedia.org/wiki/HTTP_Strict_Transport_Security) (HSTS) list is a list of domain names which have told Google that they *always* want their website served over https. If the user tries to manually request the insecure version, the browser won't let them. This means that a user's connection to, for example, their bank cannot be hijacked. A dodgy WiFi network cannot force the user to visit an insecure and fraudulent version of a site.

After about a decade of use, the list is now 14MB in size, with around 130,000 entries in it. You can [view the list online](https://source.chromium.org/chromium/chromium/src/+/main:net/http/transport_security_state_static.json) or [download it](https://chromium.googlesource.com/chromium/src/net/+/refs/heads/main/http).

The format is relatively straightforward:

```
{

 "name": "example.com",

 "policy": "bulk-1-year",

 "mode": "force-https",

 "include_subdomains": true 

},
```

When the list is updated, [Chrome creates a trie with Huffman coding compression](https://source.chromium.org/chromium/chromium/src/+/main:net/tools/transport_security_state_generator/README.md?q=transport_security_state_static.json&ss=chromium%2Fchromium%2Fsrc&start=11) - so it doesn't have to parse that monster file each time.

The most popular (over 1,000 entries) TLDs / Public Suffixes are:

| Rank | TLD | Entries |
| --: | :-: | --: |
| 1 | com | 43,236 |
| 2 | tk | 19,022 |
| 3 | de | 5,216 |
| 4 | org | 4,731 |
| 5 | gov | 4,507 |
| 6 | net | 4,410 |
| 7 | ga | 4,326 |
| 8 | nl | 2,671 |
| 9 | cf | 2,458 |
| 10 | ml | 2,271 |
| 11 | co.uk | 2,139 |
| 12 | fr | 1,714 |
| 13 | ru | 1,516 |
| 14 | eu | 1,283 |
| 15 | com.br | 1,226 |
| 16 | gq | 1,225 |
| 17 | io | 1,215 |
| 18 | com.au | 1,202 |
| 19 | it | 1,103 |
| 20 | cz | 1,004 |

After `.com`, the free `.tk` domain names absolutely dominate. I wonder how many of them are fraudulent?

There are 2,676 `.uk` domain names - only 537 of which aren't on `.co.uk`.

Going a bit further, there are 418 IDNs (which start with `xn--`).

And about 187 have "porn" in the domain.

You can't really extrapolate *much* from this as a data set. Lots of the domains seem to have expired or otherwise no longer work. Reading around [https://hstspreload.org](https://hstspreload.org) it notes that because this list is *hard-coded* into Chrome it can take months before a site is added. Similarly, removal can take a long time as well.

I can't help feeling that there should be a better way to manage all this though.