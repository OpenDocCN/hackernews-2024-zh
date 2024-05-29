<!--yml
category: 未分类
date: 2024-05-27 14:31:09
-->

# Go ahead and block AI web crawlers

> 来源：[https://coryd.dev/posts/2024/go-ahead-and-block-ai-web-crawlers/](https://coryd.dev/posts/2024/go-ahead-and-block-ai-web-crawlers/)

AI companies are crawling the open web to, ostensibly, improve the quality of their models and products. This process is extractive and accrues the benefit to said companies, not the owners of sites both small and large.

AI companies are crawling the open web to, ostensibly, improve the quality of their models and products. This process is extractive and accrues the benefit to said companies, not the owners of sites both small and large.

**[Per The Verge and OpenAI](https://www.theverge.com/2023/8/7/23823046/openai-data-scrape-block-ai)**

> "Web pages crawled with the GPTBot user agent may potentially be used to improve future models …"
> 
> "…allowing GPTBot to access your site can help AI models become more accurate and improve their general capabilities and safety."

All of which assumes that you see some broader benefit to letting a private institution improve their product with negligible benefit to you.

**[Again, via The Verge](https://www.theverge.com/2023/8/21/23840705/new-york-times-openai-web-crawler-ai-gpt)**

> *The New York Times* has blocked OpenAI's web crawler, meaning that OpenAI can't use content from the publication to train its AI models. If you check [the NYT's robots.txt page](https://www.nytimes.com/robots.txt), you can see that the *NYT* disallows GPTBot, the crawler that OpenAI introduced [earlier this month](https://www.theverge.com/2023/8/7/23823046/openai-data-scrape-block-ai).

The publication has gone on to block additional crawlers and you can see the full list by [accessing their robots.txt file](https://www.nytimes.com/robots.txt). There are open resources like [Dark Visitors](https://darkvisitors.com/) cropping up to maintain and provide lists of extractive AI crawlers.

All of this marks a clear and fundamental distinction between search crawlers and AI crawlers. The former extracts value from open content, the latter indexes and directs users *to* content, enhancing discoverability and aggregating data.

It is not incumbent upon news publications, blogs, social media sites or any other platform to cede this data to AI companies for free, nor should they. But, as search giants (and startups — looking at you Browser Company) lean (questionably) into surfacing extractive summaries rather than directing users to original, independent results, it makes more and more sense to block the agents they're using to crawl and surface those answers.

Licensing deals to offer content up to AI companies or existing tech giants are similarly attractive and again, [only of benefit to the company yielding access to data they control but didn't create](https://www.reuters.com/technology/reddit-ai-content-licensing-deal-with-google-sources-say-2024-02-22/).

That there's some sort of broad societal benefit to all of this is, at this point, mere speculation. What this does accomplish though is allow companies to continue to champion chat bots and image generators that remain rife with issues while chasing ever increasing valuations.

I'm not excited when a product I use integrates AI, I'm weary and wary of it. What does it do to make the experience better? I'd love to know. Copilot is a mild improvement over traditional autocomplete. Advice on a subject is best provided by someone that has experience on what they're advising you on rather than a bucket of bits that will spit out believable-sounding text output.

The companies building these tools will argue that more data will improve accuracy and improve the tools overall, but you have no responsibility to concede that point. Blocking these crawlers also assumes that you trust these companies to obey a long-lived, standard tool like `robots.txt` and given their argument that everything they can access is fair game for ingestion, it's fair to question whether they'll have the basic decency to do so.

My `robots.txt` looks like this — if I'm missing anything, [I'd love to know](https://coryd.dev/contact).

```
Sitemap: https://coryd.dev/sitemap.xml

User-agent: *
Disallow:

User-agent: AdsBot-Google
User-agent: Amazonbot
User-agent: anthropic-ai
User-agent: Applebot
User-agent: AwarioRssBot
User-agent: AwarioSmartBot
User-agent: Bytespider
User-agent: CCBot
User-agent: ChatGPT-User
User-agent: ClaudeBot
User-agent: Claude-Web
User-agent: cohere-ai
User-agent: DataForSeoBot
User-agent: Diffbot
User-agent: FacebookBot
User-agent: FriendlyCrawler
User-agent: Google-Extended
User-agent: GoogleOther
User-agent: GPTBot
User-agent: img2dataset
User-agent: ImagesiftBot
User-agent: magpie-crawler
User-agent: Meltwater
User-agent: omgili
User-agent: omgilibot
User-agent: peer39_crawler
User-agent: peer39_crawler/1.0
User-agent: PerplexityBot
User-agent: PiplBot
User-agent: scoop.it
User-agent: Seekr
User-agent: YouBot
Disallow: /
```

**Other great posts on the subject:**

**Update March 27, 2024:** Many thanks to [Jens](https://meiert.com/en/) for pointing out that the `User-agent` rules can be safely combined preceding a `Disallow` statement.

<svg xmlns="http://www.w3.org/2000/svg" data-tablericon-name="brand-github" fill="none" stroke="currentColor" stroke-linecap="round" stroke-linejoin="round" stroke-width="2" viewBox="0 0 24 24"><title>GitHub repository</title>xmlns="http://www.w3.org/2000/svg"width="24"height="24"viewBox="0 0 24 24"fill="none"stroke="currentColor"stroke-width="2"stroke-linecap="round"stroke-linejoin="round"class="icon icon-tabler icons-tabler-outline icon-tabler-brand-github"></svg>Take a look at [the GitHub repository for this project](https://github.com/ai-robots-txt/ai.robots.txt). (And give it a star if you feel like it.)