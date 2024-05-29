<!--yml
category: 未分类
date: 2024-05-27 14:52:45
-->

# Safe and reliable production changes for fast moving teams; and how Rivian recently got this wrong

> 来源：[https://blog.substrate.tools/safe-and-reliable-production-changes-for-fast-moving-teams-and-how-rivian-recently-got-this-wrong/](https://blog.substrate.tools/safe-and-reliable-production-changes-for-fast-moving-teams-and-how-rivian-recently-got-this-wrong/)

Changing production infrastructure can be scary because we don’t want to break things for customers, mistakes happen, and there can be many unknowns.

In this post I’ll be discussing a recent over-the-air (OTA) software update to Rivian vehicles that went badly. It is speculative; I have no insider knowledge of Rivian’s software, systems or practices. However, I do have over 20 years experience building, operating and leading teams who build and operate scalable and reliable online systems at companies such as Square, Segment and Confluent.

I’ve long been a car nerd, too, and modern vehicles, especially EVs are an exciting confluence of cars and technology, with software taking an ever greater role in the function and experience of the vehicles. I think Tesla and Rivian are doing the most to push the state of the art forward in automotive software. I pay particularly close attention to what’s happening with Rivian because that’s what I drive.

I think a very similar methodology can apply to both Rivian’s fleets of cars and our large-scale cloud services for doing safe rollouts of changes to a production environment serving customers.

## **The broken Rivian over-the-air software update**

On Monday, November 13, 2023, Rivian started rolling out version 2023.42 to customer vehicles. [RivianTrackr has a detailed timeline](https://rivian.software/2023-42-timeline/?ref=blog.substrate.tools). Typical monthly releases fix bugs, add features, and don’t cause much drama. This one did not go so well.

Roughly 3% of vehicles received this update which put their infotainment screen into a reboot loop. While the vehicles were still drivable, many of the controls were unavailable for several days while Rivian worked on a fix. Many Rivian owners (and I think Rivian themselves) worried they might have to visit a service center for a fix. Fortunately, Rivian was able to fix it with another over-the-air update after several days.

## **Process and Design Flaws**

Rivian appears to make updates available to customers a few at a time, as visualized on [electraFi’s](https://electrafi.com/firmwareRivian?ref=blog.substrate.tools) Rivian dashboard. These over-the-air updates come approximately monthly, with about two weeks of internal testing before starting a phased rollout to customer vehicles.

Thanks to these practices, Rivian caught this issue when it was released to about 3% of vehicles. But that’s still around 2,000 vehicles. 2,000 customers.

Rivian could borrow some pages from the web operations playbook to improve their process and protect their customers from many such issues in the future..

### **Canary Deploys**

With the botched 2023.42 update [Rivian explained](https://www.reddit.com/r/Rivian/comments/17usikn/202342_ota_update_issue/?ref=blog.substrate.tools) that they pushed the wrong build with the wrong certificates to customer vehicles. This made me immediately think they *probably* don’t have a canary fleet of vehicles that they roll out to first.

It seems like they have a beta fleet that is Rivian owned and Rivian employee owned vehicles. These have the ability to validate their beta certificates and get the new releases before customer vehicles. There must be some process where they enroll a vehicle in the beta program. I’m assuming this is similar to Apple’s beta program where you have to install a set of certificates to enroll and validate beta builds. Production (customer vehicles) can’t validate those certificates. So when a beta build got pushed to them, it put the infotainment in a reboot loop.

If Rivian had a canary fleet of even a single vehicle that had never run a beta build, it would have failed the update, and they could have caught this issue before it hit customers. There are plenty of problems that only happen in certain conditions that canary deploys don’t catch. But I think this particular problem could have been caught immediately with a small canary fleet.

### **Pre-flight checks**

The next problem was that Rivian vehicles downloaded and installed the update without validating it first. They seemed to do a full install and then fail at 90%, which leads me to believe they don’t do the certificate validation until the new OS image is booting. This seems like a very unfortunate design decision. Reliable systems should have a pre-flight check that aborts any change if certain health conditions are not met. In this case Rivian vehicles could have downloaded the new OS image, tried to validate the certificates and then aborted the install when the validation failed. They could then send telemetry back to Rivian so the engineers operating the rollout would have quickly seen their mistake and the customer inconvenience could have been little more than a message about the failed update, and asking them to try again later.

Again, you can’t account for every possible problem in a pre-flight check. But validating the certificate of an OS image is about the most fundamental check you could do, second only to ensuring the downloaded image matches a known checksum.

### **Rollbacks**

Finally, I’m surprised that Rivian doesn’t have any kind of automatic rollback mechanism in place for a failed update. It’s pretty common for things like enterprise routers and switches and various embedded systems to keep the last known good OS image. If the new OS fails to boot, the system will auto-rollback to the last known good image.

This is especially important for devices that are deployed in locations that are hard to get to for repair or devices that require higher uptime than any time to repair. To me it seems like having roughly 75,000 vehicles all over the US, some with customers who may not have a backup vehicle means the time to repair is long, and the uptime requirements are high.

## **No one root cause**

We often like to talk about the ‘root cause’ of a problem, as if there is a single mistake, bug or design flaw that caused the problem to happen. Then we can identify that issue, fix or prevent it, and move on. In Rivian’s [limited communications](https://www.reddit.com/r/Rivian/comments/17usikn/202342_ota_update_issue/?ref=blog.substrate.tools) about the 2023.42 update issues, [Wassym](https://www.reddit.com/user/WassymRivian?ref=blog.substrate.tools), Rivian’s VP of Software Development said: “We made an error with the 2023.42 OTA update - a fat finger where the wrong build with the wrong security certificates was sent out.”

This makes it sound like the root cause was someone typed the wrong thing and the wrong build with the wrong certificate went out to a percentage of customer vehicles.

But in my experience that is rarely how it works in practice. There are usually multiple contributing factors that cause a significant customer facing issue or outage like the one many Rivian owners experienced.

I’ve explained 3 process and design problems that I suspect combined to allow this problem to happen.

1.  No Canary Deploys
2.  No pre-flight checks
3.  No Rollbacks

If Rivian had any of the above, they could have significantly mitigated the impact of this issue.

And I haven’t even tried to speculate about what tooling lets someone fat finger the deploy to push the wrong build and certificate so the tooling itself might be a 4th contributing factor.

## What does this have to do with Cloud systems?

Many of the reliability best practices are the same. With Substrate we’re mostly concerned with AWS accounts, IAM roles and VPCs. A multi-account strategy on AWS allows changes to progress through multiple pre-production environments and even to be served by multiple accounts in production to achieve canary and partial deployments.

💡

If you’re interested in more reliable and secure cloud systems, check out

[Substrate](https://substrate.tools/?ref=blog.substrate.tools)

.

****The Right way to AWS****

Substrate is a CLI tool that helps teams build and operate secure, compliant, isolated AWS infrastructure.

From developers who have been there.

## **Philosophy**

According to “[Accelerate: The Science of Lean Software and DevOps](https://en.wikipedia.org/wiki/Accelerate_(book)?ref=blog.substrate.tools)”, organizations that deploy changes the most frequently also have the least amount of defects and outages in production. This somewhat counterintuitive observation happens for a few reasons:

*   Frequent production changes mean smaller changes. Smaller changes are usually less complex, easier to understand, and easier to fix if something does go wrong.
    *   Conversely, infrequent changes tend to be large, complex, and hard to fix when something goes wrong.
*   Teams that deploy frequently have many opportunities to practice deploying to and changing production. We tend to get quite good at things we practice regularly.
    *   These teams are also more likely to automate or smooth out any sharp edges in their tooling if they are using it very frequently.
*   Frequently deploys mean less rush to get your code ready for a fixed schedule release. With infrequent releases, teams may feel pressure to rush their code or testing so they don’t miss the release train and have to wait for the next one.

The next part of safely changing production is ensuring any changes are well tested before they reach production. High functioning software engineering teams tend to have some sort of CI (Continuous Integration) testing setup. For the rest of this post we’ll assume you have a well functioning CI system or some similar testing setup and won’t be making any recommendations in that area.

We will be focusing on how a change makes its way through various environments into production for all of your customers.

We’re convinced most teams building and operating online services are best served by frequent small changes to production, and a phased rollout approach where changes are tested in a way that minimizes blast radius, including in production.

One of the best ways to achieve this for infrastructure changes is detailed in [Terraform best practices for reliability at any scale](https://www.notion.so/RevenueCat-5bc42bbcd05c4a7cac7e897988dced72?pvs=21&ref=blog.substrate.tools).

## Take Away

Designing reliable systems can be complex, but following some key principles can make your systems much more likely to survive the inevitable bugs and mistakes.

1.  Canary deploys - Deploy your change to a tiny fleet of systems that are exactly the same as the rest of production.
2.  Phased rollouts - After canary, gradually roll your change out to more and more of your server fleet or customers.
3.  Pre-flight checks - At a minimum, your deployment system should verify that the artifact it’s deploying passes certain integrity checks. More complex versions can also verify downstream dependencies.
4.  Rollbacks - Think through how you would roll back from any change, and practice it. When in doubt, roll back! At Square we evangelized this idea so much that it was immortalized internally as go/rollback and now exists publicly at [https://outage.party](https://outage.party/?ref=blog.substrate.tools).