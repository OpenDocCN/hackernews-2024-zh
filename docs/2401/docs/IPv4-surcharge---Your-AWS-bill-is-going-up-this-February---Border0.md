<!--yml
category: 未分类
date: 2024-05-27 15:23:01
-->

# IPv4 surcharge - Your AWS bill is going up this February | Border0

> 来源：[https://www.border0.com/blogs/ipv4-surcharge---your-aws-bill-is-going-up-this-february](https://www.border0.com/blogs/ipv4-surcharge---your-aws-bill-is-going-up-this-february)

As of tomorrow, your AWS bill will go up! Effective February 1, 2024, there will be a charge of $0.005 per IP per hour for all public IPv4 addresses, whether attached to a service or not. That's a total of $43.80 per year, a pretty hefty number! The reason for this is outlined in the [AWS announcement](https://aws.amazon.com/blogs/aws/new-aws-public-ipv4-address-charge-public-ip-insights/):
‍

> As you may know, IPv4 addresses are an increasingly scarce resource and the cost to acquire a single public IPv4 address has risen more than 300% over the past 5 years. This change reflects our own costs and is also intended to encourage you to be a bit more frugal with your use of public IPv4 addresses.

‍

In this blog, I'll cover how you can save money on your AWS bill by eliminating unnecessary public IPv4 addresses using Border0\. But before we go there, let's look at how many IPv4 addresses Amazon has, how much that's worth, and how much AWS will make with this new charge to your monthly bill.

‍

## How many IPv4 Addresses does AWS have?

‍

Operating the Amazon infrastructure and keeping up with the incredible growth of AWS requires a massive amount of IP addresses. And so it comes as no surprise that over the years, Amazon has spent a lot of money acquiring an enormous number of IPv4  addresses. All so we can continue to spin up our ec2 instances, load balancers, and NAT gateways without worrying about IPv4 addresses.

‍

To determine exactly how many IPv4 addresses Amazon has, we can look at various publicly available data sets. The data I used is the [AWS IP json](https://docs.aws.amazon.com/vpc/latest/userguide/aws-ip-ranges.html) and the various whois (ARIN, RIPE, etc) data entries.

‍

*Crunching all that data, we can determine that Amazon has at least 131,932,752 IPv4 addresses.
Let’s round that up, and say* ***132 Million IPv4 addresses****! That's the equivalent of almost eight /8's 😮* Curious about the data, and what IPv4 addresses were included? See [this link for the raw data.](https://gist.github.com/atoonk/0ee3f5bebcea874f6032215f16c3c30a)

‍

## How much is the Amazon IPv4 estate worth?

‍

IPv4 addresses are like digital real estate. These 32-bit integers have real monetary value and can be bought and sold. In fact, the price of IPv4 addresses has increased significantly over the last decade, and would have made an excellent investment if you got in early!

[https://auctions.ipv4.global/prior-sales](https://auctions.ipv4.global/prior-sales)

‍

So the next logical question is, how much is the Amazon IPv4 estate worth? Based on data from ipv4.global, the average price for an IPv4 address is currently ~35 dollars. With that data in hand, we can do our back-of-the-napkin math:
‍

‍

*So the approximate value of Amazon's IPv4 estate today is about:
‍****$4.6 Billion dollars!*** Not too shabby!

‍

## How much money will AWS make with the new IPv4 charge?

‍

Speaking of dollars, let’s take a look if we can make an educated guess about how much AWS will make from the new IPv4 charge. For that, we need the price per IP and the number of IPv4 addresses in use by AWS customers.

‍

We know the first variable, $0.005 per IP per hour, or $43.80 per year per IPv4 address. The second variable, the number of IPv4 addresses in use by AWS customers, is harder to determine. However, we can make some educated guesses for fun!

‍

As mentioned, the significant variable here is how many IP addresses are used at any given time by AWS customers. Let's explore a few scenarios, starting with a very conservative estimate, say 10% of the IPv4 addresses published in the [AWS JSON](https://docs.aws.amazon.com/vpc/latest/userguide/aws-ip-ranges.html) (79M IPv4 addresses) are used for a year. That's 7.9 Million IPv4 addresses x $43.80, almost $346 Million a year. At 25% usage, that's nearly $865 Million a year. And at 30% usage, that's a billion dollars!

‍

That gives us a pretty good indicator of the scale we’re talking about. Another approach is to try and measure it. How many IP addresses are alive within the AWS network right now? AWS conveniently publishes all addresses, so we could send an ICMP echo request (a ping) to all of them and measure how many send back an echo reply.

‍

That sounded like a fun project! So I wrote a quick [program](https://github.com/atoonk/ping-aws-ips) that downloads the JSON with all the AWS IP addresses and filters out the categories "AMAZON," "EC2," and "GLOBAL ACCELERATOR." We're going to assume these are all the customer-used (charged) IP addresses. I.e., we're not going to ping services like Route53 Health Checks or Cloudfront as those won’t show up on your bill as an IPv4 charge.

‍

The program sends a single ICMP packet to all IP addresses and collects all the replies. With this, we have some actual measurement data, and we observe that we received a reply from roughly 6 Million IPv4 addresses.
6 Million addresses x $43.80 is $ 263 Million annually!

‍

That’s another good data point. However, remember that many ec2 instances and other services will have strict security groups and, by default, won't respond to a ping packet. So, it's fair to say that six million active IPs is the absolute minimum. The actual number of active IPv4 addresses could easily be double that given the various default security groups blocking ICMP.

‍

Given this data, I believe it's fair to say that ***AWS will likely make anywhere between $400 Million and $1 Billion dollars a year with this new IPv4 charge!*** That's a nice bump for AWS, especially given that this was provided for free until today.

‍

## Lowering your AWS bill with Border0

Your AWS services, such as ec2 instances, may have public IPv4 addresses for a variety of reasons. One common reason is to have management access to your servers. For example, using SSH or RDP. Or to access the app running on your machine, like a database or HTTP application.

‍

Some of your applications should likely only be accessible to authorized users and, ideally, not connected directly to the Internet. For example, [recent Border0 research](https://www.border0.com/blogs/help-my-postgres-database-was-compromised) showed that botnets are actively compromising publicly accessible Mysql and Postgres servers! You don't want these unprotected on the Internet for everyone to poke at.

‍

Where possible, we recommend running your AWS infrastructure in a private subnet with only a NAT gateway for outbound Internet connectivity. This way, they're shielded from the Internet, significantly reducing the risk of getting compromised. As a bonus, you save yourself the AWS IPv4 charge! (note: the charge is only for public IP addresses).

[https://www.youtube.com/embed/NHR3Do4WhVU](https://www.youtube.com/embed/NHR3Do4WhVU)

VIDEO

With the deployment of a Border0 connector in your private network, you and the rest of your team can still access all services using just your existing Single Sign-on credentials without needing a VPN.

Deploying Border0 is [easier than you may think](https://youtu.be/e0KeS5x-GZ0)! Curious to give it a try? Check out our [terraform example](https://www.border0.com/blogs/border0-terraform-provider) or this [blog on Border0 for AWS](https://www.border0.com/blogs/elevating-security-and-simplifying-aws-access-with-border0).
‍

Border0 offers [a generous free tier](https://portal.border0.com/register), and getting started is easy!
With Border0, access is easier and more secure; your engineers and security team will love it. And, since you're saving on public IPs, your FinOps folks will be happy, too!

‍