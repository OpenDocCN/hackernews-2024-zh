<!--yml
category: 未分类
date: 2024-05-29 13:25:41
-->

# How I turned my open-source project into a business

> 来源：[https://docs.emailengine.app/how-i-turned-my-open-source-project-into/](https://docs.emailengine.app/how-i-turned-my-open-source-project-into/)

When I started writing and publishing open-source software about 15 years ago, I was pretty radical about it. I only used permissive licenses like MIT or BSD, as all I cared about was reach. Using a copyleft license with strings attached seemed to hinder that reach. Getting another A-category company to use my open-source libraries like [Nodemailer](https://nodemailer.com/?ref=docs.emailengine.app) was a badge of honor. I even went so far that when a founder of a major transactional email service sent me an email regarding Nodemailer and offered to make a donation to promote my efforts, I rejected it. I did not want to seem affected by one of the dominant providers because this would not be fair to other providers.

In hindsight, what a fool I was.

In any case, it changed years later when a startup using Nodemailer was acquired for half a billion dollars. I was financially not in a good place back then, and when I saw the news, I started to wonder – what did I get out of this? Sending email notifications was a huge part of that service, and they probably sent millions of email notifications a day using Nodemailer. At the very least, I saved them tons of developer hours by providing a free and solid library for sending these emails. I searched my mailbox for emails related to that company and found a single complaint about a feature. No pull requests, no donations, no nothing. And there was nowhere to complain either as I had knowingly given my software for the world to use with no requirements to compensate anything. My empty wallet was not happy about the turn of events.

So, when I started what eventually became [EmailEngine](https://emailengine.app/?ref=docs.emailengine.app), I tried to cover my back as much as possible. I released the software under the copyleft AGPL license. I also set up an automated CLA process so that no one was able to get their PR merged without signing a CLA first. Many people hate CLAs, and several persons opened a PR first but closed it once they realized that there was a CLA requirement. Well, to be honest, I didn't really care. For example, 98.1% of the code for Nodemailer was written by myself, and only 1.9% was from other contributors, so not getting PRs merged was not a major issue. For EmailEngine, after a year and a half of being published as open source, the same numbers were 99.8% vs 0.2%.

> I use [CLA assistant](https://cla-assistant.io/?ref=docs.emailengine.app) for managing CLAs in my projects

Obviously, I wanted to make some money from my new project, and my business plan was simple. I published the project (it was called IMAP API at that time) as an AGPL-licensed application. I also offered an MIT version, but to get that, you had to subscribe. The subscription fee was 250€ per year. My assumption was that companies - the main target for the software - do not like copyleft licenses and would convert to the permissive license once they see how useful the app is.

Well, it turns out my business plan was bonkers. I only gained a few paying subscribers, and it seemed to me those people weren't even using IMAP API. They just wanted to support my effort. It turned out that smaller companies did not care about the license at all, and larger companies were not using it. After a year and a half and 750€ in total revenue, I decided to jump ship — enough of providing free stuff.

I re-designed the UI of the app to look more professional and implemented a license key system. From that moment if you wanted to use EmailEngine (the new name for IMAP API), you needed a license key that was only available for paying subscribers. I also changed the license from AGPL to a commercial license. The source code is still published publicly on [GitHub](https://github.com/postalsys/emailengine?ref=docs.emailengine.app). It is no longer *open-source* by definition but *source-available*. This change of license was only possible due to requiring outside committers to sign a CLA from the start.

> I still publish MIT-licensed projects, but only for smaller tools, not larger projects. The goal of these tools is to promote my main effort. For example, I extracted the IMAP client functions from EmailEngine and published it under an MIT license as a generic IMAP client library for Node.js. This module  ([ImapFlow](https://imapflow.com/?ref=docs.emailengine.app)) is gaining steam in adoption as it is by far better than any pre-existing alternative. The documentation page sends about 100 visitors per month to EmailEngine's homepage, which is not much, but hey, it is free traffic, and sometimes these visitors do convert, making the effort fruitful.

At first, there wasn't even a trial option. If you did not provide a valid license key 15 minutes after the application started, the app just stopped working.

I kept the price the same, 250€ per year, and during the first month, I sold 1750€ worth of subscriptions. That's like twice the amount I made in the previous year and a half, and it sealed the fate of the project. There was no going back.

Next, I started to increase the pricing; 250€ became 495€, then 695€ and 795€, and finally 895€. To my surprise, it did not mean getting fewer customers. I guess any sub-$1k amount for businesses is peanuts, so the only thing these price increases changed was improving the revenue.

The current MRR for EmailEngine is 6100€ and grows steadily, which in Estonia, where I live, allows me to pay myself a decent salary so that I can work on my project full-time. The only regret I have is that I did not start selling my software earlier and only published free, open-source software. Yes, I have some sponsors in GitHub, but it has never been a substantial amount, ranging from $50-$750 per month, depending on how many sponsors I happen to have. Selling to business customers is definitely more reliable and predictable than depending on the goodwill of random people.

* * *

*Edit: changed "LGPL-licensed" with "AGPL-licensed" as the original OSS license was AGPL, not LGPL.*