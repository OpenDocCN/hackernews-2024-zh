<!--yml

category: 未分类

日期：2024-05-27 14:31:09

-->

# 请继续阻止AI网络爬虫

> 来源：[https://coryd.dev/posts/2024/go-ahead-and-block-ai-web-crawlers/](https://coryd.dev/posts/2024/go-ahead-and-block-ai-web-crawlers/)

AI公司正在爬取公开网络，表面上是为了改进其模型和产品的质量。这一过程是一种抽取行为，并将好处积累给了这些公司，而非站点所有者，无论其大小。

AI公司正在爬取公开网络，表面上是为了改进其模型和产品的质量。这一过程是一种抽取行为，并将好处积累给了这些公司，而非站点所有者，无论其大小。

**[根据 The Verge 和 OpenAI](https://www.theverge.com/2023/8/7/23823046/openai-data-scrape-block-ai)**

> "Web pages crawled with the GPTBot user agent may potentially be used to improve future models …"
> 
> "…允许GPTBot访问您的网站可以帮助AI模型变得更加准确和提高其总体能力和安全性。"

所有这一切都假设你认为让私人机构通过几乎没有给你带来任何好处的方式改进他们的产品有某种更广泛的好处。

**[再次，通过 The Verge](https://www.theverge.com/2023/8/21/23840705/new-york-times-openai-web-crawler-ai-gpt)**

> *纽约时报* 阻止了 OpenAI 的网络爬虫，这意味着 OpenAI 无法使用该出版物的内容来训练其AI模型。如果您查看[纽约时报的robots.txt页面](https://www.nytimes.com/robots.txt)，您会发现 *NYT* 禁止了 GPTBot，这是 OpenAI 本月早些时候引入的爬虫。

该出版物已继续阻止其他爬虫，并通过[访问其robots.txt文件](https://www.nytimes.com/robots.txt)可见完整列表。出现了像[Dark Visitors](https://darkvisitors.com/)这样的开放资源，以维护并提供抽取性AI爬虫列表。

所有这些标志着搜索爬虫和AI爬虫之间的明显和根本区别。前者从开放内容中提取价值，后者索引并引导用户*至*内容，增强了可发现性并聚合了数据。

新闻出版物、博客、社交媒体网站或任何其他平台无需免费将此数据交给AI公司，也不应该。但是，随着搜索巨头（以及初创公司——看看你Browser Company）倾向于（可疑地）提供抽取式摘要而非将用户引导至原始、独立的结果，阻止它们使用用于抓取和展示这些答案的代理程序变得越来越合理。

授权协议以向AI公司或现有科技巨头提供内容同样具有吸引力，再次证明，[只对掌控但未创建数据的公司有利](https://www.reuters.com/technology/reddit-ai-content-licensing-deal-with-google-sources-say-2024-02-22/)。

在这一点上，这些工具有某种广泛的社会效益是纯粹的推测。然而，这确实允许公司继续支持充满问题的聊天机器人和图像生成器，并追逐不断增长的估值。

当我使用的产品集成AI时，我并不感到兴奋，而是感到疲倦和警惕。它如何使体验更好？我很想知道。Copilot相比传统的自动完成功能略有改进。关于一个主题的建议最好由有经验的人提供，而不是由一个只会输出听起来可信的文本的一大堆比特桶来提供。

建造这些工具的公司会辩称更多的数据会提高准确性并改善工具的整体表现，但你无需接受这一点。阻止这些爬虫也意味着你信任这些公司遵守像`robots.txt`这样的长期标准工具，鉴于他们主张他们可以访问的一切都可以用于摄取，质疑他们是否有基本的良知去这样做是公平的。

我的`robots.txt`看起来像这样 — 如果我漏掉了什么，[我很乐意知道](https://coryd.dev/contact)。

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

**该主题的其他优秀文章：**

**2024年3月27日更新：** 感谢[Jens](https://meiert.com/en/)指出`User-agent`规则可以在`Disallow`语句前安全地组合使用。

<svg xmlns="http://www.w3.org/2000/svg" data-tablericon-name="brand-github" fill="none" stroke="currentColor" stroke-linecap="round" stroke-linejoin="round" stroke-width="2" viewBox="0 0 24 24"><title>GitHub仓库</title>xmlns="http://www.w3.org/2000/svg"width="24"height="24"viewBox="0 0 24 24"fill="none"stroke="currentColor"stroke-width="2"stroke-linecap="round"stroke-linejoin="round"class="icon icon-tabler icons-tabler-outline icon-tabler-brand-github"></svg>请查看[此项目的GitHub仓库](https://github.com/ai-robots-txt/ai.robots.txt)。（如果你愿意，请给它点赞。）
