<!--yml
category: 未分类
date: 2024-05-27 14:40:46
-->

# PAPERWALL: Chinese Websites Posing as Local News Outlets Target Global Audiences with Pro-Beijing Content - The Citizen Lab

> 来源：[https://citizenlab.ca/2024/02/paperwall-chinese-websites-posing-as-local-news-outlets-with-pro-beijing-content/](https://citizenlab.ca/2024/02/paperwall-chinese-websites-posing-as-local-news-outlets-with-pro-beijing-content/)

## Key Findings

*   A network of at least 123 websites operated from within the People’s Republic of China while posing as local news outlets in 30 countries across Europe, Asia, and Latin America, disseminates pro-Beijing disinformation and ad hominem attacks within much larger volumes of commercial press releases. We name this campaign PAPERWALL.
*   PAPERWALL has similarities with HaiEnergy, an influence operation first reported on in 2022 by the cybersecurity company Mandiant. However, we assess PAPERWALL to be a distinct campaign with different operators and unique techniques, tactics and procedures.
*   PAPERWALL draws significant portions of its content from Times Newswire, a newswire service that was previously linked to HaiEnergy. We found evidence that Times Newswire regularly seeds pro-Beijing political content, including ad hominem attacks, by concealing it within large amounts of seemingly benign commercial content.
*   A central feature of PAPERWALL, observed across the network of websites, is the ephemeral nature of its most aggressive components, whereby articles attacking Beijing’s critics are routinely removed from these websites some time after they are published.
*   We attribute the PAPERWALL campaign to Shenzhen Haimaiyunxiang Media Co., Ltd., aka Haimai, a PR firm in China based on digital infrastructure linkages between the firm’s official website and the network.
*   While the campaign’s websites enjoyed negligible exposure to date, there is a heightened risk of inadvertent amplification by the local media and target audiences, as a result of the quick multiplication of these websites and their adaptiveness to local languages and content.
*   These findings confirm the increasingly important role private firms play in the realm of digital influence operations and the propensity of the Chinese government to make use of them.

## Why Exposing this Type of Campaign Matters

Beijing is increasing its [aggressive](https://thediplomat.com/2023/09/chinas-increasingly-aggressive-tactics-for-foreign-disinformation-campaigns/) activities in the spheres of influence operations (IOs), both online and [offline](https://www.washingtonpost.com/world/2023/09/21/china-global-influence-takeaways/). In the online realm, relevant to the findings in this report, Chinese IOs are [shifting](https://query.prod.cms.rt.microsoft.com/cms/api/am/binary/RW1aFyW) their tactics and increasing their volume of activity. For example, in November 2023 Meta – owner of the social media platforms Facebook, Instagram, and WhatsApp – announced the removal of five networks engaging in “coordinated inauthentic behavior” (i.e. influence operations) and targeting foreign audiences. Meta [noted](https://transparency.fb.com/en-gb/integrity-reports-q3-2023/) it as a **marked increase in IO activity by China**, stating that “for comparison, between 2017 and November 2020, we took down two CIB networks from China, and both mainly focused on the Asia-Pacific region. This represents the most notable change in the threat landscape, when compared with the 2020 [US] election cycle.”

Seeding ad hominem attacks on Beijing’s critics can result in **particularly harmful consequences** for the targeted individuals, especially when, as in PAPERWALL’s case, it happens within much larger amounts of ostensibly benign news or promotional content that lends credibility to and expands the reach of the attacks. The consequences to these individuals can include, but are not limited to, their delegitimization in the country that hosts them; the loss of professional opportunities; and even verbal or physical harassment and intimidation by communities sympathetic to the Chinese government’s agenda.

This report adds yet more evidence, to what has been reported by other researchers, of the increasingly important role played by **private firms in the management of digital IOs** on behalf of the Chinese government. For example, an October 2023 [blog post](https://www.rand.org/pubs/commentary/2023/10/dismantling-the-disinformation-business-of-chinese.html) by the RAND corporation summarized recent public findings on this issue, and advocated for the disruption of the disinformation-for-hire industry through the use of sanctions or other available legal and policy means.

It should be noted that disinformation-for-hire companies, driven **by revenue, not ideology,** tend not to be discerning about the motivations of their clients. As major [recent press investigations have shown](https://www.theguardian.com/world/2023/feb/15/revealed-disinformation-team-jorge-claim-meddling-elections-tal-hanan), both their origin and their client base can truly be global. Exposing this actor type, and its tactics, can help understand how governments seek plausible deniability through the hiring of corporate proxies. It can also refocus research on the latter, increasing deterrence by exposing their actions.

## Background

On October 25, 2023, the Italian newspaper **Il Foglio** published an [article](https://www.ilfoglio.it/esteri/2023/10/25/news/la-fabbrica-dei-contenuti-pro-cina-5826677), summarized in English [here](https://decode39.com/8109/network-fake-china-news-websites-italy/), that exposed a small network of six websites posing as news outlets for Italian audiences that did not correspond to any real newsrooms in Italy. Il Foglio’s investigation confirmed that the websites were not registered as news outlets in the national registry, as legally required for any information organization operating within the country.

The identified domains used a specific naming convention: the name of an Italian city in the local spelling (i.e. “Roma”, or “Milano”), followed by mundane terms (for example, “moda”, meaning fashion; “money”; or “journal”). The websites hosted on those domains were all similar in structure, layout, and content, with generic political, crime, and entertainment articles interspersed with a relatively high amount of news related to China, or even directly derived from Chinese news organizations.

Il Foglio claimed that the network was being operated from China, and possibly by the Chinese government, based on content analysis and on the six domains resolving to an unspecified IP address owned by Tencent Computer Systems Inc., a major Chinese corporation. The Italian newspaper also hinted at the possible existence of a broader set of websites linked to the six presented, without publicly disclosing further information.

On November 13, 2023, the South Korean **National Cyber Security Center (NCSC)**, a governmental agency, also published a [report](https://www.ncsc.go.kr:4018/main/cop/bbs/selectBoardArticle.do?bbsId=SecurityAdvice_main&nttId=88028&menuNo=020000&subMenuNo=020200&thirdMenuNo=) exposing eighteen Korean-language websites posing as local news outlets. The report attributed these sites to a Chinese PR firm called **Haimai**, based on the firm itself advertising the opportunity for its clients to publish press releases on these same sites. These websites presented strong similarities with the six Italian-language ones exposed by Il Foglio, from their technical structure to the modus operandi utilized.

We set out to research the whole network, with the objective of discovering additional websites, their tactics, targeting, and impact; and of verifying the attribution of the activity to its operators.

## An Extensive Network of Websites

### The Initial Set

Based on DNS infrastructure overlaps, we were able to expand the network identified by Il Foglio to an **initial total of 74 domains**. The majority of the domains could be identified through a relatively small set of three IP addresses they resolved to.

The number of domains hosted on these IP addresses is relatively low: they featured a total of less than 100 domain resolutions, while theoretically, each could have hosted thousands of domains. This could indicate that the IPs are only linked to one operator, rather than multiple clients of the provider.

We started from the following six domains, identified in the original news article:

| DOMAINS |
| italiafinanziarie[.]com |
| napolimoney[.]com |
| romajournal[.]org |
| torinohuman[.]com |
| milanomodaweekly[.]com |
| veneziapost[.]com |

*Table 1: List of 6 domains hosting Italian-language websites as identified by Il Foglio*

Based on Passive DNS resolution data made available by [RiskIQ](https://community.riskiq.com/), we found that the above domains resolved, during the last two years, to at least one of the following three IP addresses:

| **IP** | **OWNED BY** | **FIRST SEEN** | **LAST SEEN** |
| 3.12.149[.]243 | Amazon Web Services (AWS) | 2021-08-14 | 2023-07-06 |
| 162.62.225[.]65 | Tencent Computer Systems Company Limited, Shenzhen | 2023-07-07 | 2023-07-08 |
| 43.157.63[.]199 | Tencent Computer Systems Company Limited, Shenzhen | 2023-07-09 | 2023-10-28 *(date of the last check)* |

*Table 2: List of IP addresses to which the 6 domains resolved since 2021*

We found other domains that had pointed to at least one of those three IP addresses since April 2018, obtaining the following list of 74 domains:

| alpsbiz[.]com | sevillatimes[.]com | froneplus[.]com |
| vtnay[.]org | guellherald[.]com | it[.]euleader[.]org |
| stptb[.]org | aksaydaily[.]com | benmorning[.]com |
| tarragonapost[.]com | veneziapost[.]com | conanfinance[.]com |
| ekaterintech[.]com | vtnay[.]org | cordovapress[.]org |
| cordovapress[.]org | londonclup[.]com | economyfr[.]com |
| napolimoney[.]com | euleader[.]org | fftribune[.]com |
| sevillatimes[.]com | bmhtoday[.]com | ulstergrowth[.]com |
| glasgowtr[.]com | kupit-skorost-mdpv-lipeck[.]gaba[.]biz | louispress[.]org |
| ulstergrowth[.]com | alpsbiz[.]com | it[.]wdpp[.]org |
| eiffelpost[.]com | kazanculture[.]com | volgogradpost[.]com |
| euleader[.]org | tarragonapost[.]com | bmhtoday[.]com |
| tulunet[.]com | samaraindustry[.]com | glasgowtr[.]com |
| provencedaily[.]com | guellherald[.]com | deiniolnews[.]com |
| uk[.]wdpp[.]org | doyletimes[.]com | fr[.]wdpp[.]org |
| froneplus[.]com | italiafinanziarie[.]com | fftribune[.]com |
| eiffelpost[.]com | milanomodaweekly[.]com | gtad2[.]iranianhosting[.]com |
| romajournal[.]org | deiniolnews[.]com | friendlyparis[.]com |
| britishft[.]com | rmtcityfr[.]com | findmoscow[.]com |
| britishft[.]com | rmtcityfr[.]com | conanfinance[.]com |
| economyfr[.]com | uk[.]euleader[.]org | provencedaily[.]com |
| frnewsfeed[.]com | ec2-3-12-149-243[.]us-east-2[.]compute[.]amazonaws[.]com | frnewsfeed[.]com |
| friendlyparis[.]com | benmorning[.]com | *[REDACTED][¹](#fn1)* |
| londonclup[.]com | doyletimes[.]com | torinohuman[.]com |
| gorodbusiness[.]com |  |  |

Table 3: List of 74 domains also resolving to the same 3 IP addresses as the domains identified by Il Foglio

We verified that — with only four exceptions, highlighted in table 3 — the domains hosted websites posing as news outlets in several countries. *The four highlighted exceptions resolved to one or more of the three examined IP addresses before or after the rest of the network was present on them, making their affiliation to PAPERWALL questionable.* Additionally, many of them appeared to utilize the naming convention identified for the Italian-language domains (city name, followed by a generic term).

### The Broader Network

By replicating the same process on the websites highlighted in the NCSC report, we were able to identify additional domains, and confirm them as fully matching the PAPERWALL signature features.

These include:

#### The websites’ structure

All of them were built on WordPress, and utilized a ([highly popular](https://trends.builtwith.com/widgets/WPBakery)) page builder plugin – [WPBakery](https://wpbakery.com/) – for their setup.

#### The domains’ infrastructure

As spotted by Il Foglio, the current hosting infrastructure for the six Italian-language domains linked back to Tencent, a Chinese-based company. In fact, the relevant service being utilized is Tencent Cloud; and we could verify that all the currently active domains were being hosted on a Tencent Cloud IP address.

*   It is important however to note that this is something that any private customer can request, provided that certain requirements given by the host provider are satisfied.
*   We confirmed in the Tencent Cloud service [documentation](https://www.tencentcloud.com/document/product/378/17985) that the requirements imposed by the company are minimal: the identity of the individual or company subscribing to the service, a mobile phone number (to be verified through a security code sent via SMS), and a credit or debit card.
*   This effectively means that any private or corporate subscriber operating the network of websites could have pointed their domains to a Tencent IP address by subscribing to their Cloud service.

#### **The WordPress users**

We analyzed the usernames utilized to post content on the PAPERWALL websites through a technique called [user enumeration](https://nixintel.info/osint/osint-techniques-whos-behind-a-wordpress-site/). This technique revealed that the whole network shared a small number of content author names, visible in the table below.

| **USERNAME** | **# OF WEBSITES** | **NOTES** |
| Tina | 44 | European, Asian, Latin American websites |
| Chunqt | 28 | Asian websites only |
| Sophia | 26 | European websites only |
| Peter | 12 | Russian websites only |
| *[Others]* | 11 | All eleven users except one were associated with the domain napolimoney[.]com, in a complete departure from the usual pattern. We could not locate evidence that any of those users correspond to an existing person. |
| *[Undetermined]* | 12 | Websites whose user list was not accessible; or that were not online (including in an archived version) at the moment of writing this report. |

*Table 4: WordPress usernames identified as used on the PAPERWALL websites*

### The content

All of the identified websites had almost identical homepage menus, typically including (translated in the target language): Politics, Economy, Culture, Current Affairs, and Sport. The actual content being posted was a mix of scraped and reposted content from local media in the targeted country; press releases; and occasional Chinese state media articles, or anonymous disinformation content. The content could typically be observed as being simultaneously cross-posted across several of the websites at once. We analyze the content in more detail [later in this report](#the-content).

Figure 1: Combo of examples of a commercial press release related to a company called [GWM (Great Wall Motor)](https://en.wikipedia.org/wiki/Great_Wall_Motor), being posted to six different PAPERWALL websites within the span of six days (25 to 31 October 2023). Note: we did not find any evidence that GWM was aware of its content being promoted as part of a deceptive coordinated campaign.

**As of December 21, 2023, we were able to identify a total of 123 domains**, almost all of which are hosting websites posing as news outlets. A full list of these domains is available in the [Appendix](#confirmed-domains).

### Target Audiences

Based on the language utilized, as well as on the sourcing of the local news content reposted by PAPERWALL websites – an aspect that we will also describe in more detail [later in this report](#the-content) – we observed the network as mimicking local news outlets in **30 different countries**, as shown in the map below. A full list of the target countries, with the number of websites addressing each, is available in the [Appendix](#lt1mh5mnitfc).

Figure 2: Map of the PAPERWALL target audiences, showing the distribution of websites per each country targeted

To appear as legitimate local news outlets, PAPERWALL websites typically utilized local references as part of their names. For example, “Eiffel” or “Provence” for French-language websites; “Viking” for the Norwegian one; or city names, commonly used for Italian and Spanish websites.

Figure 3: Headers of napolimoney[.]com (Italy), eiffelpost[.]com (France), and sevillatimes[.]com (Spain) shown as examples of the nomenclature pattern used by PAPERWALL

A broader look at the domains’ registration timeline shows how the websites were set up in waves, one target country (or region) at a time. In July 2019, updatenews[.]info became the first PAPERWALL domain to be registered. However, due to registration data patterns and [archived captures](https://web.archive.org/web/20200815000000*/updatenews.info) on the Wayback Machine, we can only establish affiliation with PAPERWALL beginning May 2020\. The hosted website primarily published news relevant to American readers.

Meanwhile, in April 2020, the domain wdpp[.]org (presumably abbreviated for “World Development Press”) was registered. The website located on a Tencent IP address, which is also linked to updatenews[.]info and 16 other PAPERWALL domains, will be critical to our [attribution](#attribution-haimai).

In July 2020, we saw the first group registrations. That month, nine domains were registered, with each hosting a website aimed at Japanese audiences. One of them, **fujiyamatimes[.]com**, has a footer linking it to **“Updatenews”**.

Figure 4: Footer on fujiyamatimes[.]com, showing the line “Support: FUJIYAMA TIMES by Updatenews.”

The waves immediately following target Korean and again Japanese audiences; beginning in February 2021, the focus moved on to European countries, then in early 2023 to Latin American ones. A summary of the registration waves is shown in the chart below.

Figure 5: Timeline of the PAPERWALL domain registrations, with annotation of the target countries for the registered domains on each date

## The Content

Figure 6: Breakdown of the content categories found on the PAPERWALL network of websites

### Political Content: Targeted Attacks and Disinformation

Hidden within much larger amounts of generic content, a smaller portion published by the PAPERWALL network is of a political nature. The following sections break down content types and main features.

#### Targeted Attacks

A common type of politically-themed content includes **ad hominem attacks**, usually kept in English irrespective of the target audience, on figures perceived by Beijing as hostile. For example, an article titled **“Yan Limeng is a complete rumor maker”** could be found on every active PAPERWALL website as of December 2023\. This article contains a direct attack on [Li-Meng Yan](https://en.wikipedia.org/wiki/Li-Meng_Yan), a Chinese virologist who alleges that the COVID-19 virus originated from a Chinese government laboratory. While her theories [have been widely dismissed](https://www.washingtonpost.com/technology/2021/02/12/china-covid-misinformation-li-meng-yan/) by the global scientific community, the attacks on her by PAPERWALL were unsubstantiated, aimed at her personal and professional reputation, and completely anonymous.

Figure 7: Examples of an article attacking Li-Meng Yan, as published by the PAPERWALL websites [nlpress[.]org](https://web.archive.org/web/2/https://nlpress.org/persbericht/yan-limeng-is-a-complete-rumor-maker/) (Netherlands), [sevillatimes[.]com](https://web.archive.org/web/20240117084606/https://sevillatimes.com/presionesoltar/yan-limeng-is-a-complete-rumor-maker/) (Spain), and [milanomodaweekly[.]com](https://web.archive.org/web/20240117090234/http://www.milanomodaweekly.com/comunicato/yan-limeng-is-a-complete-rumor-maker/) (Italy)

Targeted attacks conducted through PAPERWALL can also take the form of **false public pressure campaigns**. To continue with the example of Li-Meng Yan, we can observe an [attempt at blocking her appointment](https://web.archive.org/web/20240117133951/http://www.milanomodaweekly.com/comunicato/expose-the-con-mans-plot/) to an alleged academic role at the Perelman Medical School of the University of Pennsylvania that was circulated by the network in October 2023.

Figure 8: Image posted on a PAPERWALL [article](https://web.archive.org/web/20240117133951/http://www.milanomodaweekly.com/comunicato/expose-the-con-mans-plot/) attacking Li-Meng Yan, and trying to block her alleged appointment to an academic role at the Perelman Medical School of the University of Pennsylvania. The article was posted across the network in October 2023

This article echoes others that circulated outside of the PAPERWALL network on websites that cannot be confirmed as part of the same network, as well as on blogging platforms. For example:

*   “[The Perelman School Of Medicine Should Expel Yan Limeng](https://web.archive.org/web/20231016211230/https://theinscribermag.com/the-perelman-school-of-medicine-should-expel-yan-limeng/)”, published on 16 October 2023 by theinscribermag[.]com. A review of the other articles posted by the same author, “Dawn Wells”, reveals more targeted attacks on political figures, for example the President of Taiwan, [Tsai Ing-wen](https://web.archive.org/web/20240109065915/https://theinscribermag.com/secret-history-of-tsai-ing-wen-reveals-secrets/).
*   “[Reject Yan Limeng for Perelman Medical College](https://web.archive.org/web/20240122085709/https://www.prlog.org/12907832-reject-yan-limeng-for-perelman-medical-college.html)”, published on prlog[.]org, a distinct but equally anonymous press release publishing platform, on 6 March 2022.
*   “[This is Yan Limeng was hired as a Perelman School](https://web.archive.org/web/20240122085244/https://medium.com/@margaretharris99306/this-is-yan-limeng-was-hired-as-a-perelman-school-327ad3ef7647)” (sic), published on 21 June 2023 on medium.com, an open blogging platform.
*   “[#汉奸闫丽梦#闫丽梦Maintain campus cleanliness Reject Yan Limon for Perelman Medical College](https://web.archive.org/web/20240122085240/https://medium.com/@ellinamalkova365874/%E6%B1%89%E5%A5%B8%E9%97%AB%E4%B8%BD%E6%A2%A6-%E9%97%AB%E4%B8%BD%E6%A2%A6maintain-campus-cleanliness-reject-yan-limon-for-perelman-medical-college-06055eac00a3)”, published on 14 December 2023, also on medium.com.

This suggests that PAPERWALL is used as an amplifier for campaigns targeting specific individuals and anonymously employing an array of additional online platforms to maximize their attacks.

#### Conspiracy Theories

A second type of politically themed content present within the PAPERWALL network of websites is conspiracy theories, typically aimed at the image of the United States, or its allies. Claims could include, for example, allegations of the US conducting biological experiments on the local population in South-East Asian countries.

Figure 9: (Left) [Example](https://web.archive.org/web/2/http://euleader.org/press/usa-conducting-human-experiments-on-thai-myanmar-border/) of conspiracy theory from euleader[.]org. The article was published in an anonymous form directly on the PAPERWALL website, with the feature image hosted on a website called timesnewswire[.]com (right), which we will further analyze in the [following section](#times-newswire). The image was taken from the [cover of a book](https://web.archive.org/web/20231215092936/https://www.amazon.com/-/en/Anna-Collins/dp/1534562893) titled “Biological Weapons: Using Nature to Kill” by Anna Collins

A final category of political content disseminated by PAPERWALL often takes the form of verbatim reposts of content from Chinese state media, such as CGTN or the Global Times. Also, in this case, the content usually remains untranslated from English. An example of this scenario is shown in figure 10.

Figure 10: [Example](https://web.archive.org/web/20231215091402/https://www.italiafinanziarie.com/comunicatostampa/cgtn-closer-people-to-people-bonds-foster-china-vietnam-friendship/) of CGTN (Chinese state media) [article](https://web.archive.org/web/20231222130852/https://news.cgtn.com/news/2023-12-13/Closer-people-to-people-bonds-foster-China-Vietnam-friendship-1pv1E4vlazm/index.html) reposted, verbatim, by the PAPERWALL website italiafinanziarie[.]com on December 13, 2023

### Scraping of Local Mainstream Media

One of the most evident tactics PAPERWALL employs to disguise its websites as local news outlets is to regularly republish content, verbatim, from legitimate online sources in the target country. Below is an example extracted from the French-language website **eiffelpost[.]com**:

Figure 11: [Article](https://web.archive.org/web/20240110085021/https://www.eiffelpost.com/culture/enquete-sur-la-mort-de-lactrice-emmanuelle-debever-premiere-femme-ayant-accuse-depardieu-de-violences-sexuelles/) posted on eiffelpost[.]com (a confirmed PAPERWALL website), left, and the [original](https://www.leparisien.fr/culture-loisirs/cinema/enquete-sur-la-mort-de-lactrice-emmanuelle-debever-premiere-femme-ayant-accuse-depardieu-de-violences-sexuelles-13-12-2023-U777EELZKVFPHOBJINUZ76JVRM.php) published by the real French newspaper Le Parisien, right

Each PAPERWALL website has large volumes of content published on a daily basis. For example, we could list a total of 5200 individual URLs published on the website londonclup[.]com, registered in May 2021, by November 10, 2023\. A volume of this magnitude points to the possibility that the process was automated. The images in the reposted articles are usually kept as hosted directly on the source website: in the example above, that is [https://www.leparisien.fr/](https://www.leparisien.fr/).

Figure 12: Screenshot of the “Sources” tab in the “Inspect” module of the Chrome browser for eiffelpost[.]com. Highlighted is the folder corresponding to [www.leparisien.fr](http://www.leparisien.fr), hosting the original image included in the article on the PAPERWALL website

### Commercial Content

#### Press Releases

Mixed with the copy/pasted news content, the PAPERWALL websites typically publish press releases of a commercial nature. These press releases are often posted either in an explicit “Press Release” section or directly on the homepage. A peculiarity of the press release content is that it is usually not translated in the target language, but remains in the original one – which, for the most part, is English.

Figure 13: Dec 15, 2023 screenshot from the homepage of the PAPERWALL website italiafinanziarIe[.]com, showing a press release (in English), mixed with Italian-language legitimate news content (lifted, in this example, from the local news website [https://www.rete8.it](https://www.rete8.it))

#### Cryptocurrencies

A substantial portion of the press release content is specifically dedicated to cryptocurrency topics. This is consistent with the sourcing of press releases from Times Newswire – which we will analyze in the [next section](#times-newswire) – where cryptocurrency topics are among the most common.

Figure 14: Snapshot of the Press Release (“Comunicato Stampa” in Italian) section of italiafinanziarie[.]com, showing five distinct cryptocurrency-related press releases, all in English. Again, the Italian language is reserved for the legitimate news content extracted from real local media

### Content Sourcing

In order to better understand the nature and proportion of the sourcing of content by PAPERWALL, we utilized the backlinks analysis platform provided by [AHREFS](https://ahrefs.com/). Backlinks are links created [when one website links to another](https://moz.com/learn/seo/backlinks).

1.  We extracted all the domains that PAPERWALL backlinked to – therefore including those hosting content published by PAPERWALL – as of November 30, 2023.
2.  We sorted them by the amount of total backlinking PAPERWALL domains, in descending order.
3.  We then manually reviewed and categorized the backlinked domains. The top 25 ones are visible in figure 15.

Figure 15: Our elaboration of the backlinks data obtained through the AHREFS platform, showing the top 25 domains that PAPERWALL websites backlinked to as of November 30, 2023\. CGTN and Global Times, both Chinese state media, appear in the list respectively with 95 and 86 backlinking domains each.
Note: to emphasize the prominence of the specific topic, we are distinguishing between cryptocurrency-related domains (“Crypto”) and more generic press release clients (“Client Company”).

The results show:

*   A top layer of **social media** domains, which is unsurprising – individual press releases will typically contain links to the client company’s social media profiles;
*   A set of **cryptocurrency websites**, which – once reviewed individually – are confirmed as the subject of multiple press releases each. Also, **two non-crypto private corporations**, likely benefiting from the paid press release services that PAPERWALL appears to host;
*   Two **Chinese state media** websites (CGTN and Global Times), backlinked to by almost 100 domains each;
*   Finally, but crucially, approximately 100 domains backlinked to **Times Newswire**, a supposed newswire service.

## Times Newswire

### Links to PAPERWALL

The consistent connection between PAPERWALL and Times Newswire is one of the most peculiar traits of the campaign. While there is certainly no definitive playbook on how online influence operations are conducted, it is uncommon for a network of coordinated websites to regularly draw content from a single publicly available but equally covert source. For example, as seen in [other known disinformation campaigns](https://citizenlab.ca/2019/05/burned-after-reading-endless-mayflys-ephemeral-disinformation-campaign/), a typical tactic would be to create copycat domains, mimicking real news sources without revealing where the content was first published. This characteristic makes it possible to analyze the distribution and type of the content and renders the source website a central component of the campaign.

As of November 30, 2023, the alleged newswire service was backlinked to by 98 distinct PAPERWALL domains, out of the total 123\. We assess that the vast majority of the backlinks in question consist of **content directly hosted on the Times Newswire website**, and **reposted by the PAPERWALL network**, as seen in a [previous example](#conspiracy-theories).

Times Newswire is a known entity in the context of influence operations: it was first reported about in [2023](https://www.mandiant.com/resources/blog/pro-prc-haienergy-us-news) by Mandiant, a Google-owned cybersecurity company. Mandiant observed Times Newswire’s hosted content disseminated through a network of subdomains for legitimate US-based news outlets in the context of an influence campaign that the company dubbed as HaiEnergy.

Mandiant had attributed HaiEnergy to a Chinese PR firm called **Haixun**, previously identified in their original 2022 [report](https://www.mandiant.com/resources/blog/pro-prc-information-operations-campaign-haienergy); however, in their [2023 report](https://www.mandiant.com/resources/blog/pro-prc-haienergy-us-news) the cybersecurity firm stated: “we currently lack technical evidence to suggest an underlying connection between Haixun and […] Times Newswire, […] and thus currently view them as distinct entities.” In fact, **timesnewswire[.]com** is – like the PAPERWALL websites – a fully anonymous asset.

It should be noted that – unlike the PAPERWALL websites – **timesnewswire[.]com** offers a “Submit Post” button, hinting at the possibility for registered users to publish content directly to the website. However, once clicked, the button leads to a login page, with no registration module being displayed. The registration of users therefore appears not to happen through the website, and is probably controlled and individually approved by the website’s operators separately.

Similarly to what was stated by Mandiant for the HaiEnergy campaign, we cannot currently attribute Times Newswire to the same operators as PAPERWALL. There are however at least two significant similarities between the newswire and the PAPERWALL network:

The **hosting IP address is also a Tencent one, and on the same AS number (132203) as the PAPERWALL domains.** An Autonomous System (AS) number is a collection of IP addresses “[under the control of one or more network operators on behalf of a single administrative entity or domain](https://en.wikipedia.org/wiki/Autonomous_system_(Internet)).”

43.153.106[.]236, US, Tencent Building Kejizhongyi Avenue, AS132203

Table 5: DNS Resolution of timesnewswire[.]com as of December 21, 2023

Times Newswire also uses a **simple WordPress template** as its main structure. Additionally, it utilizes the **same page builder plugin ([WPBakery](https://wpbakery.com/))** used by PAPERWALL.

Being central to at least two distinct operations – PAPERWALL and HaiEnergy – Times Newswire could however be an independent asset, simultaneously exploited by multiple influence operations.

### Ephemerality

We were able to identify examples of politically-themed articles that were routinely deleted from Times Newswire. For example, we observed ad hominem attack posts on figures in direct conflict with Beijing’s positions that were later removed from the website.

This behavior suggests that ephemeral seeding is the intention for most content of that type which is deleted from the source website (Times Newswire) at an unspecified time after its initial publication. As noted in previous [research](https://citizenlab.ca/2019/05/burned-after-reading-endless-mayflys-ephemeral-disinformation-campaign/), ephemeral disinformation is designed to elude detection. With the evidence disappearing from the source websites not long after having been published, investigators may be unable to make the necessary connections to detect an influence operation or correctly identify the reach and depth of the operation. At the same time, the seeded message could be picked up and amplified by mainstream or social media, making the narrative stay even if the original source had been removed.

In the case of PAPERWALL however, as we discuss in more detail in the [Conclusions](#conclusions) section, we currently have no evidence that this has ever happened.

Figure 16: Headlines of two now-deleted Times Newswire articles ([1](https://web.archive.org/web/20230601183840/https://www.timesnewswire.com/pressrelease/the-plague-like-rule-of-extreme-right-wing-religious-leader-li-hongzhi/), [2](https://web.archive.org/web/20230530152402/https://www.timesnewswire.com/pressrelease/i-suspect-master-li-hongzhi-is-a-fraud/)) attacking Li Hongzhi, founder and leader of the religious movement Falun Gong

As a final note on the operational tactics utilized by Times Newswire and, as a consequence, by PAPERWALL, we note that the articles targeting Li Hongzhi, as well as others of a political nature that we could observe, were all categorized as “press releases” on the website, similarly to the thousands of actual promotional posts it published. It is however highly unusual for press releases to include content of this kind. We judge this as another tactic designed to make the political narratives hard to detect without diminishing their potential impact.

## Attribution: Haimai

We attribute PAPERWALL to a PR firm based in China, **Shenzhen Haimaiyunxiang Media Co., Ltd.,** or **“Haimai.”**

**Haimai** was first exposed by the Korean NCSC in their investigation on 18 Korean-focused PAPERWALL websites as being responsible for operating them. However, based on the evidence presented in the NCSC [report](https://www.ncsc.go.kr:4018/main/cop/bbs/selectBoardArticle.do?bbsId=SecurityAdvice_main&nttId=88028&menuNo=020000&subMenuNo=020200&thirdMenuNo=), that assessment appeared to be primarily based on Haimai itself advertising the paid placement of promotional articles on Times Newswire, and as a consequence, on the PAPERWALL network of websites.

We do not consider this criterion as sufficient for a conclusive attribution. In fact, during our research we could identify at least three other PR and marketing companies advertising the sale of promotional packages to be placed directly on PAPERWALL websites. They include:

*   A South Korean firm named **Excelsior Partners**, which on [Kmong](https://web.archive.org/web/20231102135101/https://kmong.com/@%EC%97%91%EC%85%80%EC%8B%9C%EC%96%B4%ED%8C%8C%ED%8A%B8%EB%84%88%EC%8A%A4) (a Korean service marketplace, hosting the advertisement of specialized services by freelancers, or agencies) advertised the sale of language-specific promotional packages. Each of the packages exclusively listed PAPERWALL domains as the “major local media” on which paid editorial content could be placed.
*   A second Korean company called **AN&ON**, which [advertised](https://web.archive.org/web/20231106081136/https://www.annon.co.kr/asianeurope) country-specific promotional packages on its own website in a similar way to Excelsior Partners. The domains listed were, also in this case, PAPERWALL ones.
*   A Chinese company, called [**Coin Blog**, also known as **BIBK**](https://web.archive.org/web/20230623201122/https://bibkcn.com/promote-bibk-php/), equally selling paid editorial content placement on several confirmed PAPERWALL domains.

However, **we could identify digital infrastructure linkages between Haimai and PAPERWALL**. Specifically, the two earliest registered PAPERWALL domains, updatenews[.]info and wdpp[.]org, hosted a **Google AdSense ID** linking them to Haimai’s official website, hmedium[.]com, and to a second website directly related to it. AdSense IDs are [unique identifiers for a website operator’s AdSense account](https://support.google.com/adsense/answer/105516?hl=en).

This is therefore an incriminating finding, proving that both PAPERWALL domains had been set up by the same operators as the Haimai assets.

A review of the source code for **updatenews[.]info** and **wdpp[.]org** revealed the presence on both websites of the Google AdSense ID **ca-pub-5378976189690174**.

Figure 17: Excerpts of source code from updatenews[.]info (top) and wdpp[.]org (bottom), both displaying the AdSense ID ca-pub-5378976189690174

After conducting a reverse search on this AdSense ID, we could find it on two additional websites: **hmedium[.]com** and **sun-sem[.]com**. The former is Haimai’s official website, as reported also by the Korean NCSC; the latter appears to be a secondary website directly connected to hmedium[.]com: it uses the same splash image and text on its homepage, and offers similar promotional services on foreign media.

Figure 18: Results of a reverse [search](https://search.dnslytics.com/search?d=domains&q=ca-pub-5378976189690174) for websites using the Google AdSense ID ca-pub-5378976189690174 via DNSlytics, a freely available online tool, showing the two previously identified PAPERWALL websites, as well as the official Haimai website, and a secondary one directly related to it

Figure 19: Homepages of Haimai’s official website, hmedium[.]com (left), and of sun-sem[.]com (right)

**Haimai**, short for **Shenzhen Haimaiyunxiang Media Co., Ltd.** (深圳市海卖云享传媒有限公司), is a Shenzhen-based PR and marketing firm, ostensibly established in 2019, according to [publicly available records](https://web.archive.org/web/20240115101633/https://xinyong.1688.com/credit/publicCreditDetail.htm?spm=a262es.8215285.0.0.7a497fd9ama0G5&id=440000%2501e0ac7e88b0123f30d84569b43e616355%25012019103012435100008385&name=%E6%B7%B1%E5%9C%B3%E5%B8%82%E6%B5%B7%E5%8D%96%E4%BA%91%E4%BA%AB%E4%BC%A0%E5%AA%92%E6%9C%89%E9%99%90%E5%85%AC%E5%8F%B8&acnt=&idNumber=91440300MA5FWMCHX7). On its website, the company advertises the sale of promotional placement services in multiple countries and languages.

Figure 20: part of the country-focused promotional packages advertised by Haimai on its own official website (automatically translated in Google Chrome)

## Conclusions

PAPERWALL is a large, and [fast growing](#target-audiences), network of anonymous websites posing as local news outlets while pushing both commercial and political content aligned with Beijing’s views to a variety of European, Asian, and Latin American audiences.

The campaign is an example of a **sprawling influence operation** serving both financial and political interests, and **in alignment with Beijing’s political agenda**. By observing the minimal traffic towards the network’s websites that is measurable through open source tools[²](#fn2), and the lack of visible mainstream media coverage (including on news aggregators, such as for example Google News) or social media amplification, we can assess the **impact of the campaign as negligible so far**.

This assessment, however, as well as the large amount of seemingly benign commercial content wrapping the aggressively political one within the PAPERWALL network, should not be taken to indicate that such a campaign is harmless. Seeding pieces of disinformation and targeted attacks within much larger quantities of irrelevant or even unpopular content is a [known modus operandi in the context of influence operations](https://secondaryinfektion.org/), which can eventually pay enormous dividends once one of those fragments is eventually picked up and legitimized by mainstream press or [political figures](https://www.theguardian.com/uk-news/2019/dec/07/russia-involved-in-leak-of-papers-saying-nhs-is-for-sale-says-reddit).

Finally, the role and prominence of private firms in creating and managing influence operations is [hardly news](https://en.wikipedia.org/wiki/Influence-for-hire). However, since the early days of [research](https://www.timesofisrael.com/who-is-behind-israels-archimedes-group-banned-by-facebook-for-election-fakery/) in this space, the disinformation-for-hire industry has [boomed](https://www.nytimes.com/2021/07/25/world/europe/disinformation-social-media.html), leading to findings and disruptions in countries around the world (for a few examples, in [Myanmar](https://about.fb.com/wp-content/uploads/2020/11/October-2020-CIB-Report.pdf), [Brazil](https://medium.com/dfrlab/facebook-removes-assets-connected-to-brazilian-marketing-firms-56ccefadd653), the [UAE, Egypt and Saudi Arabia](https://about.fb.com/news/2019/08/cib-uae-egypt-saudi-arabia/)). China – previously [exposed](https://www.theguardian.com/australia-news/2023/aug/30/meta-facebook-instagram-shuts-down-spamouflage-network-china-foreign-influence#:~:text=which%20at%20times%20appeared%20to%20collaborate%20with%20private%20Chinese%20companies.) for having resorted to this proxy category in large influence operations, including the cited [HaiEnergy](https://www.mandiant.com/resources/blog/pro-prc-information-operations-campaign-haienergy) – is now increasingly benefiting from this operating model, which maintains a thin veil of plausible deniability, while ensuring a broad dissemination of the political messaging. It is safe to assume that PAPERWALL will not be the last example of a partnership between private sector and government in the context of Chinese influence operations.

## Acknowledgments

Special thanks to Jakub Dałek for his research support. Thanks to John Scott-Railton, Emma Lyon, Pellaeon Lin, Siena Anstis, and Céline Bauwens for their peer review and assistance. We would like to thank Melissa Chan for helpful recommendations. Research for this project was supervised by Ron Deibert.

## Appendix

### Confirmed Domains

| **DOMAIN** | **TARGET COUNTRY** |
| usa-aa[.]com | **[undetermined]** |
| doloreshoy[.]co | **[undetermined]** |
| splinsider[.]com | **[undetermined]** |
| garagumsowda[.]com | **[undetermined]** |
| laplatapost[.]com | **AR** |
| lujanexpresar[.]com | **AR** |
| wienbuzz[.]com | **AT** |
| boicpost[.]com | **BE** |
| brasilindustry[.]com | **BR** |
| brmingpao[.]com | **BR** |
| financeiropost[.]com | **BR** |
| goiasmine[.]com | **BR** |
| pauloexpressar[.]com | **BR** |
| pernambucostar[.]com | **BR** |
| rioninepage[.]com | **BR** |
| swisshubnews[.]com | **CH** |
| sanrafaelscoop[.]com | **CL** |
| martapost[.]com | **CO** |
| bohemiadaily[.]com | **CZ** |
| frankfurtsta[.]com | **DE** |
| munichnp[.]com | **DE** |
| dkindustry[.]co | **DK** |
| lguazu[.]com | **EC** |
| andregaceta[.]com | **ES** |
| cordovapress[.]org | **ES** |
| sevillatimes[.]com | **ES** |
| tarragonapost[.]com | **ES** |
| guellherald[.]com | **ES** |
| suomiexpress[.]com | **FI** |
| frnewsfeed[.]com | **FR** |
| froneplus[.]com | **FR** |
| friendlyparis[.]com | **FR** |
| alpsbiz[.]com | **FR** |
| economyfr[.]com | **FR** |
| eiffelpost[.]com | **FR** |
| fftribune[.]com | **FR** |
| louispress[.]org | **FR** |
| provencedaily[.]com | **FR** |
| rmtcityfr[.]com | **FR** |
| doyletimes[.]com | **IE** |
| napolimoney[.]com | **IT** |
| italiafinanziarie[.]com | **IT** |
| milanomodaweekly[.]com | **IT** |
| romajournal[.]org | **IT** |
| torinohuman[.]com | **IT** |
| veneziapost[.]com | **IT** |
| dy-press[.]com | **JP** |
| fujiyamatimes[.]com | **JP** |
| fukuitoday[.]com | **JP** |
| fukuoka-ken[.]com | **JP** |
| ginzadaily[.]com | **JP** |
| hokkaidotr[.]com | **JP** |
| kanagawa-ken[.]com | **JP** |
| meiji-mura[.]com | **JP** |
| nihondaily[.]com | **JP** |
| nikkonews[.]com | **JP** |
| saitama-ken[.]com | **JP** |
| sendaishimbun[.]com | **JP** |
| tokushima-ken[.]com | **JP** |
| tokyobuilder[.]com | **JP** |
| yamatocore[.]com | **JP** |
| bucheontech[.]com | **KR** |
| busanonline[.]com | **KR** |
| cctimes[.]org | **KR** |
| chungjutravel[.]com | **KR** |
| chungnamonline[.]com | **KR** |
| daegujournal[.]com | **KR** |
| daejeontraffic[.]com | **KR** |
| gangwonculture[.]com | **KR** |
| gwangjuedu[.]com | **KR** |
| gyeonggidaily[.]com | **KR** |
| gyeongpe[.]com | **KR** |
| incheonfocus[.]com | **KR** |
| jejutr[.]com | **KR** |
| jeontoday[.]com | **KR** |
| krectimes[.]com | **KR** |
| seoulpr[.]com | **KR** |
| ulsanindustry[.]com | KR |
| gauljournal[.]com | **LU** |
| olmecpress[.]com | **MX** |
| teotihuacaneco[.]com | **MX** |
| xochimilcolife[.]com | **MX** |
| greaterdutch[.]com | **NL** |
| nlpress[.]org | **NL** |
| vikingun[.]org | **NO** |
| bydgoszczdaily[.]com | **PL** |
| wawelexpress[.]com | **PL** |
| ptnavigat[.]com | **PT** |
| baleadimineata[.]com | **RO** |
| rogazette[.]com | **RO** |
| aksaydaily[.]com | **RU** |
| ekaterintech[.]com | **RU** |
| findmoscow[.]com | **RU** |
| gorodbusiness[.]com | **RU** |
| kazanculture[.]com | **RU** |
| rostovlife[.]com | **RU** |
| samaraindustry[.]com | **RU** |
| stptb[.]org | **RU** |
| tulunet[.]com | **RU** |
| volgogradpost[.]com | **RU** |
| balasaguntimes[.]com | **RU** |
| ismoili[.]com | **RU** |
| buranadaily[.]com | **RU** |
| wakhan[.]org | **RU** |
| luddpress[.]com | **SE** |
| kopetbiz[.]com | **TR** |
| balasagunherald[.]com | **TR** |
| taurustimes[.]com | **TR** |
| anadoluha[.]com | **TR** |
| araratdaily[.]com | **TR** |
| cappadociapost[.]org | **TR** |
| bmhtoday[.]com | **UK** |
| benmorning[.]com | **UK** |
| britishft[.]com | **UK** |
| conanfinance[.]com | **UK** |
| deiniolnews[.]com | **UK** |
| euleader[.]org | **UK** |
| glasgowtr[.]com | **UK** |
| londonclup[.]com | **UK** |
| ulstergrowth[.]com | **UK** |
| vtnay[.]org | **UK** |
| wdpp[.]org | **UK** |
| updatenews[.]info | **US** |

### Targeted Countries

| Country | Number of PAPERWALL Websites |
| South Korea | 17 |
| Japan | 15 |
| Russia | 15 |
| UK (including Scotland, Northern Ireland specific targeting) | 11 |
| France | 10 |
| Brazil | 7 |
| Turkey | 6 |
| Italy | 6 |
| Spain | 5 |
| Mexico | 3 |
| Romania | 2 |
| Poland | 2 |
| The Netherlands | 2 |
| Germany | 2 |
| Argentina | 2 |
| USA | 1 |
| Sweden | 1 |
| Portugal | 1 |
| Norway | 1 |
| Luxembourg | 1 |
| Ireland | 1 |
| Finland | 1 |
| Ecuador | 1 |
| Denmark | 1 |
| Czech Republic | 1 |
| Colombia | 1 |
| Chile | 1 |
| Switzerland | 1 |
| Belgium | 1 |
| Austria | 1 |

### High-Confidence Host IP Addresses

#### PAPERWALL Domains

| **IP** | **PROVIDER** | **# OF PAPERWALL DOMAINS** | **AS Number** |
| 162.62.225[.]65 | Tencent Cloud | 24 | 132203 |
| 43.163.221[.]160 | Tencent Cloud | 17 | 132203 |
| 43.155.173[.]104 | Tencent Cloud | 17 | 132203 |
| 43.153.75[.]48 | Tencent Cloud | 12 | 132203 |
| 49.51.49[.]54 | Tencent Cloud | 12 | 132203 |
| 43.157.63[.]199 | Tencent Cloud | 10 | 132203 |
| 170.106.196[.]76 | Tencent Cloud | 7 | 132203 |
| 43.157.58[.]203 | Tencent Cloud | 7 | 132203 |

#### Times Newswire

| **IP** | **PROVIDER** | **AS Number** |
| 43.153.106[.]236 | Tencent Cloud | 132203 |