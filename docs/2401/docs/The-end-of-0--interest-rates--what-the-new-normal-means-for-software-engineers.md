<!--yml
category: 未分类
date: 2024-05-27 15:20:42
-->

# The end of 0% interest rates: what the new normal means for software engineers

> 来源：[https://newsletter.pragmaticengineer.com/p/zirp-software-engineers](https://newsletter.pragmaticengineer.com/p/zirp-software-engineers)

*👋 Hi, this is Gergely with a subscriber-only issue of the Pragmatic Engineer Newsletter. In every issue, I cover challenges at Big Tech and startups through the lens of engineering managers and senior engineers. [Subscribe](https://newsletter.pragmaticengineer.com/about) to get issues like this in your inbox, every week.*

This article is part 2 in a 4-part series on the end of 0% interest rates. Other issues cover:

*   **[What it means for startups and the tech industry](https://newsletter.pragmaticengineer.com/p/zirp)** (Part 1.) More pressure to achieve profitability, less venture capital funding, Big Tech gets bigger, and bootstrapping more common.

*   **[What it means for software engineers](https://newsletter.pragmaticengineer.com/p/zirp-software-engineers)** (Part 2, this one.) A tougher job market, harder to negotiate offers, slower career growth, and higher performance expectations.

*   **[What it means for engineering managers](https://newsletter.pragmaticengineer.com/p/zirp-engineering-managers)** (Part 3.)Fewer EMs, more responsibilities for the rest, the rise of the tech lead role, and an opportunity to build more cohesive teams due to lower attrition.

*   **[What it means for software engineering practices](https://newsletter.pragmaticengineer.com/p/zirp-engineering-practices)** (Part 4.) Monoliths over microservices, more shifting left of responsiblities and pragmatic, simpler architecture.

* * *

We’re going through a major change in the tech industry, caused by the demise of a lengthy period of zero interest rate policy (ZIRP) by central banks. Recently, we covered [what the end of ZIRP means for tech startups and the wider tech sector](https://newsletter.pragmaticengineer.com/p/zirp): more pressure to achieve profitability, less venture capital funding, Big Tech getting bigger, and it’s harder for startups to access funding beyond the seed stage, while bootstrapping becomes more common.

Today, we cover what the new, post-ZIRP era means if you are a software engineer. I’ve talked with engineers and engineering managers for their analysis, and gathered relevant and fresh data to bring answers.

We go into:

1.  **A decade of ZIRP; a recap.** It’s not just higher interest rates that are a problem; the ZIRP era saw the smartphone and cloud revolutions, which created more efficient ways to build startups (cloud,) and offered a cheap distribution channel (smartphones).

2.  **Job market realities.** It’s tougher than it’s been in a decade, with fewer jobs, and more qualified candidates. This is great if you’re hiring, but harder for applicants.

3.  **Getting a (new) job as a software engineer.** What worked in an “employee’s market,” doesn’t function nearly as well. Suggestions on how to adapt to an employer’s market.

4.  **Compensation changes.** It’s logical for compensation to stagnate, or go down. But things are a bit more nuanced, considering the trimodal nature of software engineering packages. It’s harder to move “up” a tier now.

5.  **Negotiating offers.** The advice to negotiate hard doesn’t universally apply in today’s market. You should still negotiate, but tactfully.

6.  **Career growth and promotions.** Promotions will most certainly slow down across the industry and within companies growing slower than before.

7.  **Performance expectations and job security.** Performance expectations are going up. It’s helpful to assume less job security, and to optimize for career security.

The last few years in tech were remarkable for innovations like the smartphone and cloud revolution, but also because these innovations took place during the lengthiest period of rock bottom rates and easy access to capital in modern history:

Interest rates were at or below 1% in the US, between 2008 to 2022.

Over the last 30 years, there’s been a couple of major technology revolutions: the internet and the PC revolution (started in the 1990s, accelerated in the 2000s.) It is notable, however, how both the smartphone and cloud computing revolutions of the 2010s coincided with the start of this long ZIRP period:

The smartphone and cloud revolutions began when interest rates fell to their lowest in decades

Low interest rates meant more capital flowed into tech, searching for financial returns of a size no longer available elsewhere due to 0% rates, mostly as venture capital into startups. Simply put, it was easier than ever to raise funding for a tech startup. At the same time, cloud computing and smartphones went mainstream and made it easier to build companies, fast.

**Cloud computing brought more efficiency and ease of scalability.** To begin a software startup before AWS launched in 2006, a founder needed to purchase and set up servers, then operate them. This required thousands of dollars, and weeks to launch a website. But with a cloud provider, all that was needed was a credit card, and a founder could have a website up and running in minutes, for only a few dollars. Scaling up also became easier and almost always cheaper with the cloud.

was at Amazon when AWS was created, and he remembers the paradigm shift in [The past and future of modern backend practices](https://newsletter.pragmaticengineer.com/p/the-past-and-future-of-backend-practices):

> "This transition [to the cloud] enabled early AWS customers like Netflix, Lyft, and Airbnb, to access the same level of computing power as established tech giants, even while these startups were in their infancy. Instead of purchase orders (POs,) months-long lead times, and a large IT department or co-location provider, you only needed a credit card and could start instantly!“

Mainstream cloud computing means it’s almost trivial – and very cheap – to launch websites, services, data processing pipelines, backends for apps, and anything that is data storage, data processing, and compute. The cloud’s been a major efficiency boost for startups and tech companies. It also makes responding to demand spikes only a matter of buying more cloud resources, or investing in tools like content delivery networks (CDNs.)

**Smartphones created a new, initially cheap, distribution channel.** The launch of Apple and Google’s app stores meant smartphone users quickly got hooked on installing apps on their devices. Companies launching mobile apps early on experienced organic growth purely by having an app; some became billion-dollar businesses thanks to launching a mobile app fast, including Spotify (2009,) Instagram (2010,) Airbnb (2010,) Square (2010,) Waze (2010,) Venmo (2010,) Uber (2011,) and Tinder (2012.)

By 2015, app stores were saturated with apps, and a native iOS or Android app was no longer an easy way to attract users. Also, many users got [app fatigue](https://techcrunch.com/2016/02/03/app-fatigue/), meaning startups had to spend on advertising to lure new customers. The average cost of getting a potential customer to install a native app has steadily risen, hitting [around $5](https://www.businessofapps.com/marketplace/user-acquisition/research/user-acquisition-costs/) in the US in 2023.

**What’s next for efficiency and distribution?** For tech innovation to continue like it did in the 2010s, wide-ranging efficiency improvements, and a cheap distribution channel are necessary. Currently, AI and large language models (LLMs) are what hopes are pinned on to deliver a major efficiency jump. If AI can successfully be integrated into software engineering workflows, we could see it again become faster and cheaper to build software. 

*It’s still unclear how big an improvement AI will deliver, but AI helpers are boosting efficiency, as we covered in an article about [the productivity impact of AI coding tools](https://newsletter.pragmaticengineer.com/p/ai-coding-tools).*

On the other hand, there doesn’t seem to be a new, cheap distribution channel akin to the early internet, or a new gadget that could spur growth without the need for advertising, like the smartphone did. This means that if you build a tech product customers are willing to pay for, you still need to pay platforms like Google for search and app store ads, Meta for social media ads, Apple for app store ads, and Amazon, to reach potential customers. These ads are expensive, and so growth hacking – getting traffic via free or cheap methods – has become more popular with startups.

**The end of the ZIRP period and cheap distribution, can already be felt by those working in tech.** It’s harder to get funding, and harder to reach new customers. Most businesses already feel the pressure, and some have responded, but many are yet to. Here’s what this will mean for tech workers, and how to prepare.

Since the hottest-ever job market for software engineers in 2021-2022, the number of developer jobs has steadily decreased, and software engineering layoffs have increased. In March 2023, we covered the trend of [software engineering jobs decreasing, globally](https://newsletter.pragmaticengineer.com/p/the-scoop-42). Here is how job openings for developers have changed on the jobs aggregator, Indeed, and on Hacker News, where startups associated with startup incubator Y Combinator frequently hire from:

*The number of monthly jobs ads changing on Hacker News, and the change of the weighted “Indeed Jobs Posting Index changes,” over time. Sources: [Hacker News](https://news.ycombinator.com/)*, *[Hiring Lab](https://www.hiringlab.org/2022/12/15/introducing-the-indeed-job-postings-index/)*

Layoffs peaked across tech companies at the end of 2022, early 2023\. The site Layoffs.fyi tracks all reported job losses across all industries:

*Publicly reported layoffs across companies, starting from 2020\. Image source: [Layoffs.fyi.](https://layoffs.fyi/)*

Not all layoffs included software engineers, but tech was often hit hard by cuts. For example, tech workers were overrepresented in redundancies at Lyft (~50% of tech staff cut when the tech staff employed ~40% of all employees) when we [analyzed those layoffs](https://newsletter.pragmaticengineer.com/p/the-scoop-50-why-are-many-snap-employees).

**The job market seems even tougher for engineering managers.** I’ve talked to around a dozen managers actively job hunting in the US and Europe, and anecdotally, all say the market is difficult. *We previously covered this in [Senior engineering leadership roles are hard to get](https://newsletter.pragmaticengineer.com/i/139786187/senior-engineering-leadership-roles-are-hard-to-get).*

More layoffs mean more experienced software engineers actively looking for positions, and fewer vacancies mean even more competition. What does all that mean for jobseekers?

[Kanat Bekt](https://twitter.com/kanateven) is the CTO of [SupplyPike](https://www.supplypike.com/): a Series B software startup building supply chain software (to help retailers with cash flow and stay compliant when using a variety of suppliers). The company raised a $25M Series B in the summer of 2022 and is based in Rogers, Arkansas (Southern US). Software engineers work a few days per week in the office in a hybrid setting.

Kanat shared with me the number of applications that came in the last 2 months, for software engineering positions the company posted:

The number of applications SupplyPike received in 2 months, for positions advertised at  SupplyPike in Rogers, Arkansas

How the dynamics of the applications change is what is really interesting. From Kanat:

> “**Internship applications have doubled.** In the past years, we never got more than 400-500 inbound applications for this exact same position. I have to add that many of the applications are, unfortunately, worthless: so many of these are GPT-assisted and spam submissions. This was also the case in the previous years.
> 
> **Software engineer applications tripled, and there are more Big Tech candidates.** In 2022 and 2023, we saw around 150-200 applications for our software engineer role at new grad and the mid-level. What is new this year is how we are getting a lot of masters students apply from all over the US. Lots of applicants have Big Tech experience at the likes of Facebook, Amazon, Google, and so on.
> 
> **Senior engineers are no longer ‘shopping around’ that much.** The number of senior engineer applications remained flat from the last few years. The big difference is how, in past years, there were lots of applicants who weren’t really serious about interviewing.
> 
> We now see a lot more senior people who have unfortunately been laid off, and are coming into the interviews with the goal of taking the offer, should they get it.
> 
> **Compensation asks are now returning to ‘normal’ levels.** When the larger layoff waves in tech started early 2023, we saw a large influx of candidates. However, back then, compensation expectations from these people were out of reach for what we had to offer – and what we considered competitive. The big change is that most applicants are asking for more ‘normal’ packages that are in-line what we have consistently been offering.”

One thing that is a bit misleading in just looking at the number of applications is how, in the case of SupplyPike – even though they do not advertise it – there is more than one headcount behind their advertised positions. Kanat shared that they optimistically expect to hire maybe 4-5 interns, 2-3 software engineers and 1-2 senior engineers in the coming months. This type of split is pretty common across companies, so let’s adjust it to the higher number (so: e.g. to 5 interns planned to hire), to see how many inbound applicants there *could* be for one position hired:

A more realistic view on about how many inbound applications came in per open headcount over 2 months to  SupplyPike for hybrid positions advertised

**Better-known companies and full-remote ones will see more competition than the chart above.** SupplyPike is a good data point to look at, because the company is not as well-known as Big Tech; and it requires candidates to relocate to take the position. At the same time, within the Arkansas region, it is one of the more attractive employer, given it is a well-funded scaleup, and a tech-first company.

Keep in mind how competition will be much more fierce both at better-known “brand” companies, as well as companies that hire full-remote, across a whole country or a region. And even when applying to attractive, but lesser-known companies: expect that your competition could well be former Big Tech software engineers, like it is the case with SupplyPike.

When there were not enough qualified applicants for senior roles in a country, companies sponsored visas and paid for engineers to relocate from abroad. However, it’s an employer’s market now, so candidates that need no visa sponsorship or relocation have immediate preference. The only exceptions are highly specialized jobs which few people can do, or ones where the pay is too low for local candidates.

A hiring manager told me they expect plenty of companies will still offer visa sponsorship, but that more won’t cover relocation costs, and will leave it up to a candidate to decide if they’re willing to pay to relocate.

If you want to move to another country, seek companies that find your skillset and experience hard to hire for, locally, and aim to apply to them. If you can’t find such businesses, you might need to build up more expertise in a niche field, or become better known in the segment you work in.

When money wasn’t an issue, exceeding allocated headcount to close a standout candidate was common enough. At Uber, I observed hiring managers occasionally making the case for hiring, even when they’d run out of headcount. The justification was that stand out candidates were rare, and the headcount could be deducted from the next half year’s budget. This led to some “headcount trading” between teams, or for budget changes to be approved at director-level.

Well, this is far rarer, these days. Here’s principal engineer, Vic Vijayakumar, [sharing](https://x.com/VicVijayakumar/status/1750246851423420813?s=20) that a friend who did well in interviews didn’t get an offer because no room could be made for him:

Source: Vic Vijayakumar [on X](https://twitter.com/VicVijayakumar/status/1750246851423420813?s=20)

To highlight a positive of this rejection, Vic’s friend raised funding for his startup, and Vic is an angel investor in it!

We previously covered what a normal [attrition rate is for software engineers](https://newsletter.pragmaticengineer.com/p/attrition), and how it’s been around 10-15% at most companies, historically:

This attrition refers to voluntary, regretted attrition. “Voluntary” attrition is when an employee leaves of their own will (getting laid off isn’t voluntary,) while “regretted” means a departing employee was a good performer who’ll be missed. *Company-wide or performance-related layoffs are outside of this metric.*

**Voluntary, regretted attrition seems to be trending down in 2024, as per an update from [Dominic Jacquesson](https://www.linkedin.com/in/dominicjacquesson/).** He’s the VP Insight and VP Talent at Index Ventures, to whom I turned for insights when covering [attrition for software engineers](https://newsletter.pragmaticengineer.com/p/attrition) back in 2022\. I checked in again with Dominic, who ran a poll in Founders Keepers CPO Forum, a community of European startup HR leaders. Here is what 47 founders shared:

Attrition seems to be declining across European startups. Data source: Dominic Jacquesson (Index Ventures) and Founders Keepers CPO Forum

Dominic added context:

> “I suspect that if this poll were run with US startups, the results would be even more clearly skewed towards reduced attrition. In the US, attrition had climbed so high in 2020-2021 in a crazily competitive talent environment, that it’s got further to fall now, compared to Europe.”

In 2024, I’d expect voluntary attrition to drop significantly sector-wide, and that most places see closer to 5-10% voluntary, regretted attrition.

University graduates – and especially bootcamp grads – face what is probably the toughest job market since the Dotcom Bust, back in 2001\. For this group, the market has been very difficult since the pandemic began and companies hired fewer grads due to the rise of remote working, which made it harder to mentor less experienced developers. As a result, many companies simply stopped hiring interns or new grads, even during the [hiring boom of 2021-2022](https://newsletter.pragmaticengineer.com/p/perfect-storm-causing-a-hot-tech-hiring-market).

Now that many companies have returned to the office – in full or at least partially – those companies can onboard new recruits much easier. Most new grad positions I’ve found at tech companies are in-the-office at least a few times per week. For example, this is how [Microsoft is hiring interns](https://jobs.careers.microsoft.com/global/en/search?exp=Students%20and%20graduates&et=Internship&l=en_us&pg=1&pgSz=20&o=Relevance&flt=true), how [Meta is hiring interns and new grads](https://www.metacareers.com/jobs?teams%5B0%5D=Internship%20-%20Engineering%2C%20Tech%20%26%20Design&teams%5B1%5D=University%20Grad%20-%20Engineering%2C%20Tech%20%26%20Design), and how Booking.com [is hiring graduate software engineers](https://careers.booking.com/early-careers/). The common denominator: all are in-office roles.

Two years ago, I [offered advice for early-career software engineers](https://blog.pragmaticengineer.com/advice-for-junior-software-engineers/) in a challenging job market. It still applies:

*   Apply to less competitive companies, including 'unsexy' ones

*   Apply to local companies, not just remote ones

*   Apply to consultancies/developer agencies

*   Know that almost nobody will sponsor relocation with visas for entry-level positions

Extra advice: in-person roles in less popular locations where fewer people are willing to relocate to, will get fewer applicants. As someone starting their career, perhaps consider applying for less competitive positions and relocating if needed.

There are pockets of high demand for talent. The obvious one is, of course, **AI-related software engineering**. AI is the single hottest industry right now, with venture capital flowing in. AI companies are hiring researchers, ML engineers, and software engineers to build their systems. These workplaces seem to be the only ones in hypergrowth mode, with no obvious signs of slowing down. *We previously covered how the Applied software engineering team at OpenAI grew from a few engineers to about 150 in only 3 years, in [Inside OpenAI: how does ChatGPT ship so fast?](https://newsletter.pragmaticengineer.com/p/inside-openai-how-does-chatgpt-ship)*

**‘Martech’ engineers** seem to be in high demand, according to [Vrashabh Irde](https://www.linkedin.com/in/%F0%9F%91%8B%F0%9F%8F%BC-vrashabh-irde-3bb66b1b/) (Vrash,) director of engineering at Carwow. Martech stands for marketing technology, and includes areas like search engine optimization (SEO,) email marketing, marketing systems, digital events systems, and similar. Vrash told me:

> “The demand for Martech engineers is increasing. Many companies are realizing the need for dedicated engineering teams to handle marketing tasks such as analytics, SEO, optimization, performance, and efficiency. As a result, martech engineers are turning to contracting and demanding higher rates.”

Why could there be an upswing in demand? [Juan Mendoza](https://www.linkedin.com/in/juan-mendoza-2b3a63a4/) writes [The Martech Weekly](https://www.themartechweekly.com/) (TMW) newsletter. In March 2023, he published a deep dive on how marketing companies use data warehouses, and extract marketing insights internally. For this, data engineers are needed who understand marketing use cases. [From TMW](https://www.themartechweekly.com/a-home-for-marketing-data/) (emphasis mine):

> “The reality is that as marketing becomes increasingly mature in its digital sophistication, so do investments in data tools and platforms. **But it’s not the marketers that need to skill up, it’s the data engineers that enable the services to work for marketers.”**

I’d add they’ll also want to hire software engineers with experience in making marketing and ads teams more efficient.

There will be more such smaller pockets across regions and industries where demand has increased. It’s likely that this will be for senior roles, and for positions that help generate revenue: just like Martech engineers help sales and marketing teams get more customers and close more deals.

So, how do you approach job hunting in a more challenging job market?