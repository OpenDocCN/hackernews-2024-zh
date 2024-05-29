<!--yml
category: 未分类
date: 2024-05-27 14:44:35
-->

# Data Egress: What is it and how much does it cost?

> 来源：[https://getdeploying.com/reference/data-egress](https://getdeploying.com/reference/data-egress)

An often overlooked cost of using cloud services is data egress. This is the cost of sending data out of the cloud provider's network to the public internet.

Here's what 1 TB of egress beyond the free allowance would cost you with each provider:

> ***Important:** Prices may vary by region and other factors not listed here. For ease of comparison, I selected the region closest to North Virginia (USA) or Frankfurt (Germany), and I made several [assumptions](/reference/how-we-estimate-costs) to come up with these estimates. Be sure to check the provider's pricing page for the most up-to-date information.*

## Understanding cloud egress

If you're a developer, chances are you've used cloud services for tasks like storing files, running your apps, or hosting websites.

These services are typically charged based on usage, but one of the costs you might not be aware of until you get your cloud bill is data egress.

It's important to understand how it works because data egress fees can quickly add up, *especially* when you plan on moving a lot of data around.

Let's break down what data egress is, how much it costs, and what you can do to keep your data egress costs down.

## What is data egress?

Data egress is the term used to describe data leaving a network, more specifically, **data leaving your cloud provider's network** out to the public internet. This can be data sent from a cloud provider to a user, or data sent from one cloud provider to another.

Cloud providers typically charge for egress based on the amount of data sent out of their network, and it's usually measured in gigabytes (GB) or terabytes (TB) of data transfer per month.

## Data egress vs ingress

From the cloud provider's perspective, there are two types of data transfer:

*   **Ingress:** data entering a network. Typically free.
*   **Egress:** data leaving a network. Typically charged.

In practice, it might look like this:

*   When a user **uploads** a file to a cloud storage service, that's considered *ingress to* the cloud provider.
*   When the user **downloads** the file, that's considered *egress from* the cloud provider.

To download the file, the user's device is requesting data from the cloud provider's network, and the cloud provider is sending the data out to the user (or to an intermediary like a content delivery network). This is where egress fees come into play.

## Why do cloud providers charge for egress?

Cloud providers charge for egress because it costs them money to send data out of their network. They have to pay for the infrastructure and bandwidth required to send data to users.

It's also important to note that not all networks are created equal. Some cloud providers may have better peering agreements with ISPs or more reliable network infrastructure, which can affect the cost of egress too.

However, egress fees may also be used to discourage certain types of usage that may saturate the network, or constantly moving large amounts of data between cloud providers.

## Keeping egress costs down

Most cloud providers do offer a certain amount of free egress each month. For example, as an account-wide allowance (eg. 100 GB / mo), or pooled across the number of servers you have with them (eg. 1 TB / mo per server).

So depending on your usage and the cloud provider you choose, you may be able to avoid egress fees altogether or keep them to a minimum.

Here are some additional factors to consider when trying to keep egress costs down:

*   **Content Delivery Network (CDN)** Cache and serve static assets closer to your users. That way, you can reduce the amount of data transferred from your cloud provider to your users.
*   **Compression:** Compress your data before sending it to reduce the amount of data transferred. Gzip and Brotli are popular compression algorithms.
*   **Data transfer pools:** Consider using a cloud provider that offers a data transfer pool. This allows you to pool together the data transfer allowances of multiple services within the same account.
*   **Monitoring**: Set up usage and billing alerts to notify you when you're approaching your free allowance or a certain threshold.
*   **Private networking:** Your cloud provider may offer free egress for data transferred between services within the same data center or region when using a private network. However, do watch out for NAT gateway charges and other fees that may apply.

## Conclusion

Data egress is the cost of sending data out of a cloud provider's network to the public internet. It's important to understand how it works because data egress fees can quickly add up, especially if you're moving a lot of data out of the cloud.

**Tip:** You can use this website to [compare cloud providers](/compare) and their egress costs to find which one is the best fit for your use case.