<!--yml
category: 未分类
date: 2024-05-29 12:37:54
-->

# 10 years of web scraping: a perspective about selling web data

> 来源：[https://substack.thewebscraping.club/p/selling-web-scraped-data](https://substack.thewebscraping.club/p/selling-web-scraped-data)

During the past few days, I’ve had several occasions to think about my journey in the web scraping industry, which started ten years ago and is still far from the end. It’s been a ride so please allow me to share with you my personal perspective about these ten years in web scraping, with an outlook on the future too.

Me and

, the other co-founders of [Databoutique.com](https://www.databoutique.com/), were colleagues in a Big 4 consultancy firm in Italy, working on the so-called business intelligence.

We knew each other more or less in 2008: it was a time when having a data warehouse for a bank (and not one of the most critical ones but for the marketing offices) implied a 6 figures bill for buying hardware and software licenses on top, from Oracle to ETLs and data viz. On top of that, you need to add the cost of the consultants who were designing and building everything.

After some years of spinning the hamster’s wheel, we started thinking that besides internal data of the companies, there was a huge flow of data available on the web that no one was capturing (at least, no one we were aware of).

We started then our scraping project of real estate websites and at it was so damn simple compared to today’s challenges.

Just as a reminder: AWS EC2 was just launched at the end of 2008 and it took some time for us to understand what was this Cloud Computing.

Before that, our scraping infrastructure was a Hetzner server where all the scrapers ran and a database for storing parsed data. That’s it.

No proxies, and no rotating IPs, it just worked so.

The same unsophistication applied to software. The initial release of Scrapy is dated 2008 but it took us some years to discover it, so every spider we wrote was a C++ program with cURL to make the requests and some other instructions for the parsing of the data.

Despite the simplicity of the architecture, this lasted 2-3 years before switching to a more modern one, but it was enough to cover the Western world real estate market.

The tech stack, as well as the target websites, changed during these years, following the technical evolution of the web and, in reflection, of web scraping.

* * *

I’ll talk about the challenges of web scraping as a guest [in the next NetNut webinar this Thursday](https://netnut.io/lp/website-unblocker-webinar/?utm_source=pier), where there will be also the launch of their new Website Unblocker, a tool that should be able to bypass most of the common anti-bot systems around.

**[Click here to register for free, see you there!](https://netnut.io/lp/website-unblocker-webinar/?utm_source=pier)**

* * *

Despite the naivety of the solution, I think we could scrape simultaneously 50+ websites, creating a huge database of listings from all around the world.

Number of sales of this database? 0

While the technology stack changed over time, there’s one thing that didn’t in these years: **selling web data is f*ing hard.**

During our adventure with Re Analytics, our data factory company, Andrea and I explored different ways to monetize web data and we can categorize them into two buckets: selling alternative data to the finance world and selling web data to the retail industry.

While we have some sort of success in both situations, we could also experiment with the pain behind it.

Alternative data is data gathered from nontraditional sources. For investors, that generally means looking beyond company filings and what brokers say to get an edge in the market. They can be credit card transactions, receipts collections and, of course, web scraping.

None of the two of us worked for a hedge fund, so we had to learn during the years what it takes to be a great data vendor, and I could summarize it in this formula:

\(Accuracy + Data History + Exclusivity\)

It means that you need to set up a process to acquire high-quality data (and eventually learn how to fill the gaps in it in the best way possible), for a long enough period to be able to backtest any model using your data. Once you have it, just sell it to a few customers, otherwise, the *Alpha* they get is drastically reduced.

Let me develop this idea a bit more by simplifying some concepts: the financial markets are forward-looking. Investors are willing to know now what’s happening in the next days, weeks, or months and for this, they’re looking for some data that help them explain how a certain listed company or market is going.

Example: given a certain trend in credit card transactions and comparing it with previous years, they can understand if the consumer goods industry will suffer or not the next Christmas season.

Another example, this time coming from web data, is the following: by monitoring years of inventory levels for a certain website, you can understand how single brands or industries are performing in terms of sales before they officially communicate it with their company filings.

* * *

* * *

But unlike credit cards or other proprietary data feeds, no one if not the website is responsible for the data. This means that you can websites could change and, of course, they won’t notice you in advance. So you need to be as fast as possible in fixing broken extractors, delivering the highest level of ***accuracy*** possible since any gap or wrong data could reduce the effectiveness of the financial model built on top of the time series.

And all this work is needed to be done for at least one or two years, since analysts need to backtest their model using your data, to understand if it’s somehow correlated with stock prices. Yes, it could happen even the worst case possible, that you collect data for years only to find out that there’s not any meaningful correlation with the stock market. You only know it after getting enough ***data history*** to start experimenting.

Last but not least, let’s say you’ve found something meaningful for investors and you were good (and lucky) enough to be able to scrape data for some years. It’s time to monetize it: in any other business, once you create a product, you want to sell it to as many customers as possible. Unluckily, in alternative data, you can’t unless yours becomes a must-have dataset. The reason is quite simple: it’s like sharing a secret with a friend. This secret is more valuable when fewer people know about it, while if everyone knows it, it becomes public and no longer interesting to know. The fewer people will buy your dataset, the more advantage your customers have, since they know more information than their competitors. While ***exclusivity*** is key for your customers, it’s not that great for your business.

In fact, let’s complete the original equation:

\(Accuracy + Data History + Exclusivity = (Few Customers * Good Contracts) / High Capex = Non Scalable Business\)

While for us it provides a good revenue stream, we cannot say that alternative data should be the one and only core of our activity.

You have a few customers paying a good amount of money, but the costs of creating such types of datasets are so high that it’s difficult to create a scalable and profitable business only from them.

As a confirmation of this theory, successful businesses in this space are companies that sit over a mountain of proprietary data, like payment processors, online travel agencies, and so on, that found a way to monetize this data they get in a reliable way with almost no effort.

On the contrary, I don’t have in mind any company that creates alternative data from web scraping and surpassed ten/twenty USD million in revenues for several years. But this is my personal idea and I’d be happy to be proved wrong.

Another major revenue stream that companies involved in web data extraction have is selling it to retail companies: brands, online players, and so on.

But selling raw data most of the time is not enough: several companies are not ready to ingest data feeds from day 0 and in many cases the insights from data are too distant from the raw level, requiring algorithms, models, and all the expertise of laser-focused companies.

In this case, the equation for selling web data is the following:

\(Data Collection + Proprietary Expertise + Customization\)

This is true also for us at Re Analytics: we’re making the effort of extracting web data, adding the expertise we’ve built in these years to create a sort of Saas to sell to our customers in the fashion industry.

The issue is that every customer wants different websites and has different needs where to put the focus: someone wants more insights on their kids’ collection, someone else on discounts, and almost every new customer wants new websites we actually don’t cover.
Instead of selling a product, we’re selling a custom-made service, but let me ask you a question: how many of you went to Zara or H&M, and how many instead went to a professional tailor for a suite?

I think I know the answer and we all know how big are today Zara or H&M while artisans could make a good living from their jobs but for sure could not compete in terms of revenues with them.

The reason is quite simple: a tailor-made suit costs a lot, since there’s so much work and expertise inside of it, that it could be also the best suit in the world but few people could buy it in comparison to a 5$ t-shirt from Zara.

Even in this case, we could complete the equation as follows:

\(Data Collection + Proprietary Expertise + Customization = (Few customers * Good Contracts) / High Running Costs = Non Scalable Business \)

On the contrary, I still believe that web scraping has great potential, mostly because today we’re very far from mass adoption for the reasons we’ve said before.

No company doesn’t try, at least once, to make some web scraping internally, but given the actual challenges and the need for maintenance of the data feed, almost every internal project fails, while current solutions on the market are too expensive for most of them.

For this combination of factors, there’s a large audience of unserved potential customers that makes it even more difficult to understand the total web scraping market potential.

One thing, instead, is sure: there’s no more space for naivety like there was 10 years ago, today web scraping requires professional figures, who know how to overcome the different challenges of these days.

Companies cannot think to assign internally to generic IT people these tasks: they need to create scraping teams or, if they don’t have the space, ask for help from professionals.

While this is true for the end customers, this lesson we’ve learned is also applicable also to actual web data-relying companies like Re-Analytics.

If you’re building an AI agent, a SAAS, or whatever, you should be laser-focused on it and not on data collection. This reality check and growing awareness about our business led us to pivot Re Analytics last year to Databoutique.com, a marketplace for web scraped data.

At Re Analytics we’ve been forced to become experts in web scraping during these ten years and, despite that, we understood we could make it by counting on our only forces. We had to say no to smaller projects involving new websites to be acquired because the effort would have been too much and the reward too little in comparison. We have received a no countless times from potential customers because we need to acquire new websites and data would not be ready in time. We were spending so much energy and time to keep up and running both the front-end and the data acquisition pipelines that every improvement on both sides was hard to make.

By decoupling the data production from the value proposition to customers, every company can be more focused on its core business. Suppose your strength stays in extracting value from data for your customers, in the form of AI agents, dashboards, SAAs, or whatever. In that case, you don’t have to worry anymore about the data acquisition. As an example, on Databoutique you can browse the catalog and have a look if there’s the website you need: if so, you can directly buy the dataset for a few bucks and eventually ask for a refresh at the frequency you desire. If it’s not available, just ask it and someone will do it for you. But if you wish to keep your scraping operation in-house, you can simply integrate your offers with other websites while, on the other hand, acting as a seller on the marketplace, to add a revenue stream for your company.

Or if you’re just starting your company right now and you need web data, you can easily validate your ideas by directly buying data instead of making scrapers, at a fraction of the cost that it would take to learn to scrape or simply ask on Upwork.

This is because, on the seller side, web scraping is no longer an artisanal work but it’s productized, to serve more customers. This industrialization enables them to split the extraction costs among more buyers, keeping prices low enough to be more convenient for buying rather than doing it internally.

As a data seller, the selling equation looks like:

\(\begin{cases} LowSellingPrice * Many Customers > HigherPrice * Single Buyer \\ LowSellingPrice * Many Customers = MoreMoneyForYou+More Money For Tools = More Reliable Data Feeds \end{cases} \)

The theory is that for sellers, selling more datasets at a fair market price is more convenient than freelancing one-to-one, and this extra revenue allows them to invest more in tools for their work and create more reliable data feeds.

For buyers, instead, the equation will change in this way:

\(Affordable Datasets = (MoreDatasetBought * More Customers) / Fewer Data Acquisition Costs \)

The role of Databoutique is to bring more data buyers onto the platform and make them happy, so they keep buying. In this way more and more sellers make money and more datasets are available, making Databoutique more attractive for buyers, in a sort of virtuous circle that makes the web scraping industry more efficient and finally remunerative.

We just started a few months ago and the feedback we’re receiving is great. If you agree with the idea that a more efficient web scraping industry is possible, I’m asking you a favor: please share this article with a friend or a colleague who needs web data and talk to him about Databoutique.

[Share](https://substack.thewebscraping.club/p/selling-web-scraped-data?utm_source=substack&utm_medium=email&utm_content=share&action=share)

This will help us and all the web scraping professionals already involved in the project to grow and new players to join.

Thanks!

* * *