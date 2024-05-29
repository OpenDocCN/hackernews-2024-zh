<!--yml
category: 未分类
date: 2024-05-27 15:08:44
-->

# NoML Open Letter

> 来源：[https://noml.info/](https://noml.info/)

# NoML Open Letter

A specification for those who want content searchable on search engines, but not used for machine learning.

Publishers need improved ways to indicate how they want content to be used in search and machine learning. Using robots.txt does not cover all use cases, and so a complementary approach is needed as proposed here. It is one which can be applied to individual webpages as desired, and can be preserved as such in datasets of web content.

[Sign the open letter via Github,](https://github.com/Mojeek/noml-open-letter/issues/new?assignees=PrivacyDingus&labels=&projects=&template=sign.md&title=SIGN%3A+NAME) **or if you'd prefer to**, you can [sign the letter via email.](mailto:josh@mojeek.com?subject=Sign%20the%20open%20letter&body=Please%20provide%20your%20name,%20a%20URL%20if%20you%20would%20like%20your%20name%20to%20be%20hyperlinked%20somewhere,%20and%20an%20affiliation%20(company,%20organisation%20etc.)%20if%20relevant.%20If%20you%20are%20signing%20the%20letter%20as%20a%20company%20or%20organisation,%20consider%20attaching%20a%20logo)

## NoML Proposal

### Four cases

Blocking a user agent in robots.txt (such as the search spider BingBot) for ML/AI products may also block potential visibility in associated search results. So if you don’t want your content to be used in products which leverage machine learning, you may also lose the benefits of being listed in search results. This is an unacceptable situation where content creators and publishers may be required to make a choice between “all or nothing”. We therefore need an approach which addresses four basic use cases as follows:

| **1.** Do use content for search Do use content for machine learning | **2.** Do use content for search **Do not** use content for machine learning |
| **3\. Do not** use content for search Do use content for machine learning | **4\. Do not** use content for search **Do not** use content for machine learning |

Several suggestions for how to address this issue have been put forward. Here we propose something different, simpler and we think fairer. It is a simple adaptation of how meta and X-Robots tags are already used as we explain below in detail.

### Explanation and Examples

Since there are many companies scraping/crawling webpages in order to collect data, and often without identifying themselves, a way which addresses more than just search engine crawler bots and using robots.txt is needed. The proposal here is to add a new ‘noml’ value to the already-existing [meta and X-Robots tag](https://www.semrush.com/blog/robots-meta/). Meta robots tags are used already for search engine crawlers, so companies like Google that crawl for their search engine already follow requests made in this way. For example the `noindex` and `nofollow` values are used to instruct search engines on how to handle a webpage.

Also meta data is stored in Common Crawl whose crawled data is a very common, and often the biggest, source of data used in machine learning and to train AI models. The `noindex` attribute is used to tell search engines not to include a page in their search results, even though they can crawl the page. This tag is used when a webpage is not intended to be indexed by search engines. The `nofollow` attribute is used to tell search engines not to follow the links on a webpage. This attribute is used when a webpage that contains links should not be considered as endorsements or recommendations. Similarly the `noml` could be used to instruct any service (e.g. a search engine or AI builder) that any data from that page should not be used in machine learning.

This can be simply expressed for HTML pages using:

`<meta name="robots" content="noml">`

and for non-HTML using:

`X-Robots-Tag: noml`

Just as for the meta tag, where the name “robots” refers to all user-agent tokens, you can also identify individual user-agents. For example, you can currently request that Google does not include a page in the search results:

`<meta name="googlebot" content="noindex">`

and similarly you could request that Google **does include a page in their search** index, but **does not use the data for machine learning** with:

`<meta name="googlebot" content="noml">`

and request that Microsoft **does not include** a page in their Bing search index and **does not use** the page for training, with:

`<meta name="bingbot" content="noindex, noml">`

you might also request that OpenAI, for example, does not use the page for machine learning with:

`<meta name="gptbot" content="noml">`

however in this case, since OpenAI is not operating a search engine, you could (also) block them from crawling in the [robots.txt](https://en.wikipedia.org/wiki/Robots.txt) file using:

`User-agent: gptbot`
`Disallow: /`

Further examples of are shown below, for HTML and non-HTML:

|  | Include page in search | Follow links | Use in machine learning |
| `<meta name="robots">` 
* * *

`X-Robots-Tag:` | ✓ | ✓ | ✓ |
| `<meta name="robots" content="noindex">` 
* * *

`X-Robots-Tag: noindex` | ✕ | ✓ | ✓ |
| `<meta name="robots" content="nofollow">` 
* * *

`X-Robots-Tag: nofollow` | ✓ | ✕ | ✓ |
| `<meta name="robots" content="noml">` 
* * *

`X-Robots-Tag: noml` | ✓ | ✓ | ✕ |
| `<meta name="robots" content="noindex, nofollow">` 
* * *

`X-Robots-Tag: noindex, nofollow` | ✕ | ✕ | ✓ |
| `<meta name="robots" content="noindex, nofollow, noml">` 
* * *

`X-Robots-Tag: noindex, nofollow, noml` | ✕ | ✕ | ✕ |

### Search Engine API Usage

Crawler based search engines such as Mojeek, Bing and Google, also provide their results to other search engines and services via APIs. We propose that these search engines, as Mojeek would do, include the ‘noml’ directive within their API response. Search API providers should make it part of their terms of API service that results labelled as such are not used for machine learning purposes by API end users.

### Conclusion

This proposed solution achieves the desired goals, by simply adding one additional value to already existing methods, and without necessarily requiring more user-agents to be used.

With this proposal creators and publishers can indicate separately whether they want content to be findable in search engines and/or used for machine learning.

### Share the open letter

## Dear AI companies, Crawlers, Search Engines, ML Projects, Scrapers etc.

We, the undersigned, support the adoption of this proposal, which enables creators, publishers, and all other content contributors on the web to indicate whether their content can be utilized for machine learning training and applications.

### Signatures from organisations and companies

### Signatures from individuals