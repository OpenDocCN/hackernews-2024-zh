<!--yml
category: 未分类
date: 2024-05-27 12:54:23
-->

# Improved infrastructure pricing – Vercel

> 来源：[https://vercel.com/blog/improved-infrastructure-pricing](https://vercel.com/blog/improved-infrastructure-pricing)

Based on your feedback, we're updating how we measure and charge for usage of our infrastructure products.

*   We're **reducing pricing** on Vercel fundamentals like **bandwidth and functions**
*   For the majority of customers, monthly bills will **remain the same or decrease**
*   We're introducing **new primitives** for easier optimization
*   You now **pay exactly for what you use** in granular increments
*   Our **Hobby tier remains free**

[These changes](https://vercel.com/pricing/coming-soon) will show up on your first bill between June 25 and July 24, 2024\. Emails with next steps for your account will be sent to account owners.

Instead of two large, combined metrics (bandwidth and functions), you will now have granular pricing which allows you to optimize each metric individually.

*¹ Price for most traffic, cost will* [*vary by region*](https://vercel.com/docs/pricing#regional-pricing)

Our updated pricing is more aligned with industry standards and a better representation of the work Vercel is doing for you, rather than bundling together in bandwidth and function metrics.

In addition, over the past year we've added features to the platform to help automatically prevent runaway spend, including [hard spend limits](https://vercel.com/changelog/improved-hard-caps-for-spend-management), [recursion protection](https://vercel.com/changelog/automatic-recursion-protection-for-vercel-serverless-functions), [improved function defaults](https://vercel.com/changelog/serverless-functions-can-now-run-up-to-5-minutes), [Attack Challenge Mode](https://vercel.com/changelog/prevent-malicious-traffic-with-attack-challenge-mode-for-vercel-firewall), and more.

For our Enterprise customers, existing contracts are unaffected. Please reach out to your account team if you have any questions or want to discuss these changes.

Visualize how our more granular infrastructure metrics are accrued.

Visualize how our more granular infrastructure metrics are accrued.

For the majority of Pro customers, monthly bills will **stay the same or go down**.

Emails with next steps for your account will be sent to account owners. **Hobby will remain free** and will have free included usage in proportion to previously included bandwidth and functions.

If a Hobby customer exceeds these new metrics within the next 6 months, their applications will be unaffected to give time to adjust.

This update to our infrastructure pricing does not change any other part of our pricing.

These changes will go into effect during your billing cycle between May and June 2024.

Before that date, the Vercel dashboard will display updated usage and billing information to allow you to understand these changes. Further, updated documentation will be live explaining each priced metric and how to optimize for each.

With our current pricing, developers on the Pro plan have some included usage in the first $20/mo plan, and then can pay for additional usage on demand after the included amount has been reached. For example, bandwidth. The first 1TB is included at $20/mo, and then after it is $40/100GB.

This is not ideal for two reasons. First, there are multiple services included in the “bandwidth” price, including Fast Data Transfer, Fast Origin Transfer, and Edge Requests. With our improved infrastructure pricing, these metrics are now separate to allow you to optimize each individually. We still have included usage in the base $20/mo. Your first 1TB of Fast Data Transfer, 100GB of Fast Origin Transfer, and 10 million Edge Requests are included. When you exceed these values, you can accrue usage of each metric separately.

Second, usage was previously packaged into 100GB increments. If you used 1GB of on-demand usage after your plan's included amount, you would've been charged $40\. With our improved infrastructure pricing, you're only charged $0.15 rather than $15.

Many customers will see price decreases for basics like bandwidth and functions due to this change.

Our more granular infrastructure pricing splits bandwidth and functions into metrics like Fast Data Transfer, Fast Origin Transfer, Edge Requests, and more.

You can learn more about each metric in [our documentation](https://vercel.com/docs/pricing). Further, to explore all prices, you can view a preview of [our updated pricing page](https://vercel.com/pricing/coming-soon).

No—to help support the open-source ecosystem, we provide free Vercel Pro accounts to a variety of frontend frameworks, JavaScript libraries, non-profits, and philanthropic efforts.

These sponsored plans are **not being changed**.

We are committed to providing a free tier for Vercel users. As a reminder, free plans do not require the addition of a payment method and come with included infrastructure usage to build your first few projects.

While making our infrastructure metrics more granular, we have kept a equivalent of the previous free tier included usage. For example, 100GB of Bandwidth is now 100GB of Fast Data Transfer, 10GB of Fast Origin Transfer, and 1 million Edge Requests.

If you exceed the included free tier usage, your projects are paused. You can then upgrade to a paid plan to continue growing with Vercel.

Caching responses can reduce the usage of Vercel Functions.

If the function response is [cached](https://vercel.com/docs/edge-network/caching), it will not run and incur a Function invocation or any GB/hrs of duration. Any request made to Vercel will incur an Edge Request. Requests to static assets incur the same Fast Data Transfer usage if the response is cached or uncached.

Further, Vercel automatically adds etags to all responses to prevent end users from downloading the same asset many times. Our Edge Network automatically [compresses](https://vercel.com/docs/edge-network/compression) responses to reduce your Fast Data Transfer.

We provide customers with tools to observe, control, and alert on their infrastructure spend with [Spend Management](https://vercel.com/blog/introducing-spend-management-realtime-usage-alerts-sms-notifications).

You can define a spend amount (e.g. $40) and receive email, web, and SMS notifications as you reach that amount. When reaching 100%, you can optionally automatically pause all projects with a hard limit.

Over the past year, we've added features to the platform based on your feedback to help automatically prevent runaway spend, including [recursion protection](https://vercel.com/changelog/automatic-recursion-protection-for-vercel-serverless-functions), [improved function defaults](https://vercel.com/changelog/serverless-functions-can-now-run-up-to-5-minutes), [hard spend limits](https://vercel.com/changelog/improved-hard-caps-for-spend-management), [Attack Challenge Mode](https://vercel.com/changelog/prevent-malicious-traffic-with-attack-challenge-mode-for-vercel-firewall), and more.

We've introduced [regional pricing](https://vercel.com/docs/pricing#regional-pricing) for Fast Data Transfer, Fast Origin Transfer, Edge Requests, and other metrics. These resources are priced differently depending on where your users are visiting from, due to differences in the underlying infrastructure.