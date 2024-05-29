<!--yml
category: 未分类
date: 2024-05-27 14:47:12
-->

# The Ugly Business of Monetizing Browser Extensions

> 来源：[https://mattfrisbie.substack.com/p/the-ugly-business-of-monetizing-browser](https://mattfrisbie.substack.com/p/the-ugly-business-of-monetizing-browser)

Always remember: If you aren't paying for the product, *you are the product.*

Earlier this year, I was excited by the launch of ChatGPT, and as a weekend side project I launched [ChatGPT Assistant](https://chrome.google.com/webstore/detail/chatgpt-assistant-use-ai/kldepdcdedfibmjnggmolhffdddbphjg), a browser extension to easily send prompts to ChatGPT. I never planned to monetize; my goal was merely to build free software that people would find useful.

At present, there’s about 26,000 people with ChatGPT Assistant installed. For a side project, I’m pretty happy with this. It’s a modest number compared to other Chrome extensions with hundreds of thousands or millions of users, so I didn’t think it would draw much attention.

I was not expecting what would come next. Over the ensuing weeks, people started to come out of the woodwork with monetization opportunities - some of which were quite gross.

Emails started to come in asking to advertise inside the extension.

> *I'm a fan of ChatGPT Assistant and I really like how convenient and useful it is.*
> 
> *Have you considered offering promotional spots to those interested in promoting their products on your extension? I'm interested in promoting my own extension on ChatGPT Assistant and would love to discuss this possibility with you.*
> 
> *Let me know if you're open to this.*

Not surprising, and in my estimation, not particularly nefarious. The open web runs on advertising, and it’s certainly possible to run ads ethically.

However, there is one wrinkle with extension pages: they are not subject to adblockers. If an extension developer were to insert ads or tracking inside either the popup or options page, any adblocking software you have installed would have no way of intercepting those requests.

A company called [Datos](https://datos.live/) reached out to me

> *My name is ___, I am ____ at Datos - a global data analytics provider.
> 
> On behalf of the company, I would like to express our interest in the data partnership with your company. *
> 
> *We are looking for partners who can provide user behavior analytics to anticipate customer needs by understanding where they are on the web journey, what information or assistance they need, and what problems they might encounter along the way.
> 
> Would you have interest in scheduling an exploratory discussion? Please share your thoughts and we'll plan accordingly.*

I was intrigued! I set up a call with them to learn more.

The company pays various sources to collect anonymized clickstream data, which it then sells it to anyone who wants it: business intelligence analysts, hedge funds, that sort of thing. They wanted data sources with high data quality and at least 100,000 monthly active users.

At the end of the call, they said they wanted a trial dataset to assess how much they would pay me. Here’s what sort of stuff they’re collecting:

The final and most common form of monetization was offers to just buy the extension outright. I got a bunch of these:

One person tried to entice me by sending screenshots of their escrow transactions:

Extension developers will be quite familiar with this. [This person collected all their inbound in one place on GitHub](https://github.com/extesy/hoverzoom/discussions/670).

I decided to haggle with these people to see how high they would be willing to go. Based on my handful of data points [# users, max_offer], it seems that these individuals are willing to pay about $0.20 per user.

Now I surely don’t know what the new owners of these extensions are doing with them, but I don’t think their intentions are noble. The vector for abuse is easy to understand: buy a well-used extension, compromise its users, and squeeze the juice however you like. Maybe you steal their credit card numbers, may be you sell their traffic, maybe you use them in a botnet. Again: it’s possible that these acquiring entities do not intend harm, but it sure doesn’t look that way.

To transfer ownership of a Chrome extension, all you have to do is [fill out this form](https://support.google.com/chrome_webstore/contact/one_stop_support). That’s it. The users *will*  *not be notified of the new ownership*, and their installed extension will continue to silently update with whatever new updates the new owners decide to push out.

For example, [ChatGPT for Google](https://chrome.google.com/webstore/detail/chatgpt-for-google/jgjaeacdkonaoafenlfkkkmbaopkbilf) is by far the most popular ChatGPT extension, with over 2 million installs. It started out innocently, as an open source project on GitHub. I myself was a happy user. However, if you check out the [GitHub repository](https://github.com/wong2/chatgpt-google-extension), you’ll notice that it was recently acquired:

All of this happened silently. Who bought it? What are they doing with it? We’ll never know, but I’m certainly not keeping it installed.

Note: **I did not take any of these people up on their offers** and have not monetized ChatGPT Assistant

* * *

*Matt Frisbie is the author of [Building Browser Extensions](https://www.buildingbrowserextensions.com/)*

*You can reach me at [mattfriz.com](https://www.mattfriz.com/)*