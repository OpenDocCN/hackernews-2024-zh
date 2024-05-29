<!--yml
category: 未分类
date: 2024-05-29 12:46:55
-->

# ❌ SSPL is BAD ❌

> 来源：[https://ssplisbad.com/](https://ssplisbad.com/)

# ❌ SSPL is BAD ❌

## TL;DR

SSPL is **terrible** for every user, companies and generally speaking, to the whole community. It is a warning signal of the bad days that are coming for everyone if all of us don't react fast.

*   🚨 SSPL licensed products are **not open-source**
*   🚨 SSPL is **killing** cloud and managed services **competitors**
*   🚨 SSPL will **rise hosting prices**
*   🚨 SSPL is **killing open-source**
*   🚨 SSPL goal is probably not about fighting big firms, but more about **getting money back to investors**

## What is SSPL

[SSPL (Server Side Public License)](https://www.mongodb.com/licensing/server-side-public-license) is a license introduced in 2008 by the company "MongoDB, Inc" to lock down the usage of "MongoDB" which was, at that time, an open-source product. The SSPL is now the main license for products like "Elasticsearch" ([announce](https://www.elastic.co/blog/licensing-change)), "Kibana", and "Graylog" ([announce](https://www.graylog.org/post/graylog-v4-0-licensing-sspl)) which was open-source products too.

The license states in [article 13](https://www.mongodb.com/licensing/server-side-public-license#13-offering-the-program-as-a-service) that if you want to provide the product directly to your customers, you have to make the "service source code" available publicly.

Here is article 13:

> 13.  Offering the Program as a Service.
> 
> If you make the functionality of the Program or a modified version available to third parties as a service, **you must make the Service Source Code available via network download** to everyone at no charge, **under the terms of this License**. Making the functionality of the Program or modified version available to third parties as a service includes, without limitation, **enabling third parties to interact with the functionality of the Program** or modified version remotely through a computer network, offering a service the value of which entirely or primarily derives from the value of the Program or modified version, or offering a service that accomplishes for users the primary purpose of the Program or modified version.
> 
> “Service Source Code” means the Corresponding Source for the Program or the modified version, and the **Corresponding Source for all programs that you use to make the Program or modified version available** as a service, including, without limitation, management software, user interfaces, application program interfaces, automation software, monitoring software, backup software, storage software and hosting software, all such that a user could run an instance of the service using the Service Source Code you make available.

The "service source code" means, the product itself, the website, the API, the back office, backups systems, monitoring systems, and everything else **with no limitation**. But that's not all... You have to publish the source code of "**all programs that you use to make the Program or modified version**".

"All programs". Do you see how global and vague it is?

Did you use Figma to design your logo? **You have to publish Figma source code?**

Did you use a Windows/macOS based computer to work on your product? So **you have to publish Windows/macOS source code?**

Did you use a Dell computer to work on your product? **You have to publish the BIOS source code of your Dell computer?**

Nobody is capable of publishing the source code of all programs that are used to conceive a product. And to finish to give any change to everyone to give back to the community, anything you will publish has to be done using the SSPL license.

In fact, the license is so vague that **we don't even know if it possible to use external APIs** for which we don't have the source code.

This license forbids everyone to provide the product directly to its customer. It is the case for cloud providers and managed services providers but not only.

Everyone that proposes MongoDB, Elasticsearch, or Graylog products directly to its customers is not allowed to do it.

You have an API that exposes a route to a MongoDB change stream? **BEEP! You are not allowed!**

You have an API that exposes a route to an Elasticsearch document? **BEEP! You are not allowed either to do this!**

You want to give your customers access to your Graylog dashboards or let them send logs to it? **BEEP again! You can't!**

To resume, if you want to give customers access to a product licensed with SSPL, you have to:

*   Share the sources of everything: website, API, backup and monitoring systems, hosting systems, storage systems, etc...
*   Share the sources of everything you worked with to create make this available: code IDE, graphical editors, operating systems, etc...
*   Add SSPL license to every resources shared above.

SSPL is a shortcut to say "no one can provide our product to someone without our agreement". And by agreement, it is of course "without paying fees", "without respecting our rules" and "without do what we want you to do". Totally opposed to freedom and open-source spirit.

## Why SSPL is so bad to everyone

Presented as a nice solution to avoid some evil cloud companies to make profits without giving back to the community, this license is in fact a danger for freedom and a competitors killer. It simply lock down products that were open-source to commercial ones, hidden behind a false "free" spirit. Should we now call them "[freemium](https://en.wikipedia.org/wiki/Freemium)" software? Probably.

SSPL is a danger to freedom because [it is not an open-source license](https://opensource.org/node/1099). MongoDB, Elasticsearch, Kibana, and Graylog are simply [no more open-source products](https://www.elastic.co/pricing/faq/licensing#does-this-mean-that-elasticsearch-and-kibana-are-no-longer-open-source). The companies behind those products can now fully decide who can propose those products to customers and negociate financial agreements (that will not be publicly revealed), leading to the death of real competitors.

These companies can now offer their own cloud hosting solutions without having any competitor. And without publishing their source code of course... They can have a deal with other cloud providers of course... with fees they will define, killing the competition and innovation.

It means that for us, customers, we will not have anymore the ability to choose our cloud provider. Mechanically, the prices of hosting will raise and the only choice we will have to fight back will be to dedicate a team to manage the hosting part on our own infrastructure... which is obviously what we all tried to avoid during years with cloud hosting solutions!

#### The other big problem is about contributions.

Open-source people refuse to contribute to projects licensed with SSPL. It means that a lot of independent contributors are stopping to improve those products or developing plugins and modules. It means too that those products are now almost 100% developed by the company behind the product. We saw in the past the problems that this [kind of governance raises with Node.js](http://anandmanisankar.com/posts/nodejs-iojs-why-the-fork/).

Also, cloud companies will not contribute back anymore to those products. Who will ever contribute to a project that is changing its behavior to run against you? Having cloud companies contributing to open-source projects is a huge chance for all of us to get high performances, reliability, and security improvements. Losing those contributions is terrible for the community!

## Who is using SSPL

*   "MongoDB" product is licensed under SSPL since 2018. "MongoDB, Inc" is the company behind. This company:

*   "Elasticsearch" product is licensed under SSPL since 2021. "Elastic NV" is the company behind. This company:

*   "Graylog" product is licensed under SSPL since 2020. "GrayLog, Inc." is the company behind. This company:

## Some questions to ask to ourselves

*   How a company can ask others to "give back to the community" while they are not using open-source anymore?
*   How a company can ask others to publish their source code while it doesn't publish its own cloud services source code?
*   How a company can declare open-source is part of its DNA while it uses a non-open source license?
*   How many of the 3000+ employees share their work and source code with the community? (spoiler alert: way behind 3000)
*   Are those companies tech companies that want to share with the community or sales companies that want to increase investors' profits?

## Wrong arguments about SSPL

#### "I'm not concerned, I'm not a cloud provider"

We are actually all concerned about this.

First, because we used an open-source software that is now **no more open-source** (or at least its updates, which means we will have to stop using these products in few weeks).

Second, because the companies behind these products will now be the only ones to provide cloud services. Losing competitors on a market is almost always synonymous of **price increase** and **lack of innovation**.

Third, because we are losing a big part of third contributors to these projects, meaning we are **losing the talent** of many people around the world.

#### "It's a good thing, these companies need money to survive"

Any company needs money to survive, of course. And money in open-source is not bad at all.

This is why MongoDB has its "Enterprise Server" edition. Or Elasticsearch has its "Elastic Enterprise Search" product. Or Graylog has its "Enterprise" version. These companies got profits before SSPL, with their open-source editions and commercial editions.

They provided premium support too, for big companies that need high level skilled people.

#### "Competitors are still here as SSPL can be negotiated"

Some competitors (mainly big firms) will negotiate a license allowing them to provide cloud solutions. But these licenses will include different fees and conditions between actors. Meaning that the companies using SSPL will be able to fix cloud competitors' prices and conditions. When a company controls the entire market, there are, in fact, no more competitors. There is just a false perception of competition which is way worst than no competition at all.

#### "SSPL is about fighting big cloud actors and it is great for smaller businesses"

SSPL is probably not about fighting big cloud actors but rather charge them. The smallest actors will probably never get access to license negotiations meaning that the smallest competitors will simply go bankrupt and vanish.

## Advice to open-source projects

Even if, at first, SSPL seems to be a great solution, please, don't make that mistake.

There is problems with some cloud providers, we can't ignore it and we have to fight it, all of us. But fighting it by moving your product out of open-source world is probably not the good way to do it.

You, for a significant part, have created this product. Nobody in the world knows it better than you do. Providing high valued support is probably good leverage. Adding some certifications and products designed for big firms is the insurance of being the leader on your market.

*   ✅ If you need to be profitable, do enterprise licenses and premium support: "if you need support, we have the best team on the planet for that".
*   ✅ Use copyleft licenses to ensure companies will give back to the community: "if you modify our code, share it with the community".

If, however, you choose SSPL licensing you will probably:

*   lost a lot of independent contributors and big firms employees contributors
*   betray your contributors who worked to improve your open-source software
*   leave the open-source world to do a freemium software
*   associate your product to a very bad buzz
*   lose the benefits of a large number of diverse players who help you to promote your product around the world, increasing the need for premium support

## Disclaimer

MongoDB, Elasticsearch, Kibana, Graylog, Node.js, Figma, Windows, macOS are trademarks and property of their respective owners. All product names used associated to open-sourced projects are for identification purposes of their open-sourced products only.

## Share and discuss

SSPL and its implications have to be known. Share this page to provoke debate and help the community.