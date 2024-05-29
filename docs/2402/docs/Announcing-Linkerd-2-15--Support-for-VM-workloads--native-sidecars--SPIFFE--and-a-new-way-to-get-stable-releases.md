<!--yml
category: 未分类
date: 2024-05-27 15:02:53
-->

# Announcing Linkerd 2.15: Support for VM workloads, native sidecars, SPIFFE, and a new way to get stable releases

> 来源：[https://buoyant.io/blog/announcing-linkerd-2-15-vm-workloads-spiffe-identities](https://buoyant.io/blog/announcing-linkerd-2-15-vm-workloads-spiffe-identities)

*Note: this is* [*an expanded version of the post on linkerd.io*](https://linkerd.io/2024/02/21/announcing-linkerd-2.15/) *with additional context and clarification, and an FAQ at the bottom.*

Note 2: Please see [our followup post with some clarifications](https://buoyant.io/blog/clarifications-on-linkerd-2-15-stable-announcement/).

Today we're happy to announce the release of Linkerd 2.15, which adds support for **mesh expansion**, **native sidecars**, and **SPIFFE identities**, and introduces **a new model for stable releases**.

The new "mesh expansion" feature in Linkerd 2.15 allows users to bring applications running on VMs, physical machines, and other non-Kubernetes locations into the mesh, delivering Linkerd's uniform layer of secure, reliable, and observable connectivity across both Kubernetes and non-Kubernetes workload alike. The 2.15 release also introduces support for SPIFFE, a standard for workload identity which allows Linkerd to provide a consistent layer of uniform layer of cryptographic identity and authentication to off-cluster workloads, and for native sidecar containers, a new Kubernetes feature that eases some of the long-standing annoyances of the sidecar model in Kubernetes, especially with Job workloads.

Finally, this release introduces some important changes in the way that we're publishing Linkerd: as of 2.15, we will no longer be producing open source stable releases. If you're running Linkerd in production today, please see the section *A new model for stable releases* below.

As usual, the 2.15 release includes a massive list of bugfixes and improvements. Read on for details!

## Mesh expansion

As [we promised last November](https://linkerd.io/2023/11/07/linkerd-mesh-expansion/), Linkerd 2.15 introduces *mesh expansion*: the ability to deploy Linkerd's ultralight Rust microproxies anywhere outside of Kubernetes and connect them to a Linkerd control plane running on a Kubernetes cluster. This allows Linkerd to handle non-Kubernetes workloads, upleveling all TCP communication to and from these workloads secure, reliable, and observable. Non-Kubernetes applications get the full set of Linkerd features, including mutual TLS, retries, timeouts, circuit breaking, latency-aware load balancing, dynamic per-request routing, zero trust authorization policies, and much more.

Mesh expansion is an important part of achieving our goal of making Linkerd the *universal networking layer for cloud native organizations.* While we love Kubernetes, we recognize that even the most sophisticated organizations often still have significant investments in applications that don't run outside of it. With Linkerd 2.15, regardless of whether your workloads are running on resource-constrained ARM64 edge devices, legacy "big iron" VMs, or physical machines in your server closet, Linkerd's uniform layer of security, reliability, and observability is at your disposal.

This move was made significantly easier by Linkerd's core design of ultralight microproxies written in the Rust programming language. The use of Rust, which has the "[ability to prevent memory-related bugs, manage concurrency, and generate small, efficient binaries](https://github.blog/2023-08-30-why-rust-is-the-most-admired-language-among-developers/#:~:text=Rust's%20minimal%20runtime%20and%20control,%2Dtime%2C%20and%20efficiency%20needs.)" allows Linkerd not just to avoid the memory vulnerabilities that are endemic to languages like C and C++, but to provide minimal resource footprint and—most importantly—a minimal operational burden to the user. Linkerd's Rust microproxies are key to its simplicity-first approach, and our ability to deliver small, static binaries which can be compiled for a wide variety of architectures and platforms was key to unlocking Linkerd's new mesh expansion capabilities. 

## SPIFFE support

One major challenge in mesh expansion is how to generate *workload identities* for non-Kubernetes workloads*.* To solve this, we introduced support for [SPIFFE](https://spiffe.io/), a CNCF graduated project that addresses exactly that concern.

Workload identity is central to Linkerd's approach to communication security. Rather than relying on easily-spoofed IP addresses to identify clients and servers, Linkerd *doesn't trust the network*: it secures communication with mutual TLS, which not only encrypts the communication and prevents tampering, but also but cryptographically authenticates the identities of client and server based on unique workload identities. Mutual TLS is a stricter variation of the same well-established protocol (TLS) that powers the majority of the Internet today.

Prior to Linkerd 2.15, Linkerd could simply use the workload's Kubernetes ServiceAccount to automatically generate a workload identity. Using this pre-existing identity was central to our "zero config zero trust" approach that makes dropping mTLS into an existing Kubernetes application trivial, and we continue to support it in Linkerd 2.15.

For workloads running outside of Kubernetes, however, there are no ServiceAccounts to rely on: only applications running on machines. To solve this, we turned to SPIFFE, a standard hosted by the CNCF, and its reference implementation, SPIRE. These two projects solve the problem of generating secure workload identity for arbitrary processes on arbitrary machines. Linkerd 2.15 generates SPIFFE ids for non-Kubernetes workloads using SPIRE, and these ids can be used alongside Linkerd's existing ServiceAccount-based ids  as the basis for Linkerd's zero-trust authorization policies.

With Linkerd 2.15 you can now encrypt all traffic to your VM workloads by default, and add zero-trust controls over all access right down to the level of individual HTTP routes and gRPC methods for specific clients.

### Native sidecar support

Linkerd 2.15 adds support for native sidecar containers, a new Kubernetes feature that was introduced in 1.28 and is enabled by default in Kubernetes 1.29\. Deploying Linkerd with native sidecars [fixes some of the long-standing annoyances of using sidecar containers in Kubernetes](https://buoyant.io/blog/kubernetes-1-28-revenge-of-the-sidecars), especially around support for Jobs and race conditions around container startup.

## A new model for stable releases

In Linkerd 2.15 we're making some significant changes to the way that Linkerd is delivered.

While Linkerd is and will always be open source, we are shifting our stable release efforts entirely to delivering [Buoyant Enterprise for Linkerd (BEL), our production-grade Linkerd distribution](/enterprise-linkerd/). BEL is Linkerd plus a set of additional tools, features, and testing designed for sustained, large-scale production use. *BEL is the distribution of Linkerd that we run ourselves* and this is what we recommend for serious use.

Linkerd 2.15.0 and all future stable releases will be delivered as BEL releases—complete with extra tools, features, and testing. We're really excited about this change because BEL is amazing. To support it, *we're making BEL free for non-production use, and always free for production use at organizations with fewer than 50 employees.*

Moving to BEL from an existing Linkerd 2.14 installation is as easy as updating your Helm repo. This is a big, change and we want to make it simple and predictable for everyone with plenty of breathing room. We ourselves are Linkerd operators, devops engineers, and platform owners who operate production systems with Linkerd, and we'd never want anything to get in the way of your core job of delivering the secure, highly-available platform your own businesses rely on. To make it easier on your, *BEL use is completely unrestricted for 90 days, until May 21st, 2024*, and *organizations that sign up in the next 30 days are also eligible for a substantial discount on pricing.* We're also happy to accommodate non-profits, high-volume use cases, and other organizations with unique needs. Please see the FAQ below for more details.

This change is absolutely necessary for us to fund the incredible set of features we're busy adding to Linkerd. We're tackling significant new work like adding ingress traffic, egress control, IPv6, Windows, eBPF, ambient approaches, and more to Linkerd, and we're determined to to deliver them with the same ultralight, ultrafast, "just works" approach that makes Linkerd so uniquely today in a space that's notorious for complexity. Linkerd 2.15 is just a  step in a long journey of compounding value.

But to accomplish these lofty goals, *we need the companies that are building their businesses on top of Linkerd to do their part to fund the project*. This change ensures that they can do that, while giving the thriving ecosystem of startups, developers, and students the ability to use Linkerd without restriction.

We'll continue publishing edge releases to the GitHub repo as usual. These have always served as a valuable mechanism for early testing and vetting by the community, and we hope to see more of this under this new framework.

And to be clear: Linkerd continues to be, and always will be, open source. This change is about the release artifacts, not about code, governance, community, or anything else. We've been open source users, contributors, and advocates since long before we created Linkerd, and our commitment to a healthy, inclusive, collaborative, and ever-growing open source Linkerd remains as strong as ever.

## What's next for Linkerd?

You might've noticed that the pace of iteration for Linkerd has increased substantially. Over the past 6 months alone we've shipped 12 stable releases, 16 edge releases, and merged hundreds and hundreds of branches, while keeping our quality bar as high as ever—if not higher. Momentum compounds, and we're excited to tackle the next set of challenges.

First, there are a couple important features that didn't quite make it into 2.15.0 that we're going to address as part of subsequent point releases. This starts with **extending our support for mesh expansion to include private networks**, so that customers who don't have a shared flat network spanning Kubernetes and non-Kubernetes applications can make full use of the new mesh expansion capabilities.

We're also going to **bring our Gateway API and non-Gateway API interfaces up to parity** in an upcoming 2.15 point release. Today we have a non-unified set of APIs across our Gateway API-based configs and our earlier pre-Gateway API configs, and that causes unnecessary friction for users. While we believe that the Gateway API is the future, we recognize it's not the present for many users. We're committed to supporting our users who are using ServiceProfiles and other pre-Gateway API configuration mechanisms, and bringing these two interfaces into parity is vital to achieving that goal.

**Support for IPv6** is another short-term priority for Linkerd and we expect to have some news regarding that in the near future. We've seen increasing demands for this from our customers and the required changes are reasonably well-scoped.

Beyond those smaller features, we're very excited to work on two big ones in the short term: **handling ingress traffic** and **adding control over egress traffic**. These are very common requests from customers who want to extend Linkerd's comprehensive yet *simple* layer of traffic control, security, and visibility to handle traffic in and out of the cluster, and we have some very exciting ideas for how to deliver this.

Finally, we continue to investigate other ways of delivering Linkerd, including "ambient" and other approaches. While Linkerd's unique Rust microproxy approach means it doesn't suffer from the same downsides as Envoy-based service meshes, we're not religious about our decisions, and continually evaluate the tradeoffs at hand in the interest of operational simplicity for our customers.

## Getting started with Linkerd 2.15

You can try Linkerd 2.15.0 via CLI or Helm install. Check out the [getting started guide](https://docs.buoyant.io/buoyant-enterprise-linkerd/latest/overview/) and happy meshing!

## FAQ

### So how do I get Linkerd now?

You have three options. First, stable releases are now available in the BEL Linkerd distribution. These are installable via CLI and Helm charts and look, feel, and act just like the stable releases of the past. To get started, follow the [instructions here](https://docs.buoyant.io/buoyant-enterprise-linkerd/latest/overview/). Note that companies with 50 or more employees must pay to use these stable releases in production after May 21st, 2024, but non-production use is unrestricted—so feel free to test away.

We also produce edge releases, typically on a weekly basis. These are available in the open source repo. See the [Linkerd edge release instructions here](https://linkerd.io/releases/). See the notes below on what an edge release is.

Finally, the source code to Linkerd is available on GitHub, and this is where development happens. This code is licensed under the Apache v2 license and is freely available to anyone in the world.

### What's the difference between edge releases and stable releases?

All Linkerd development happens "on main": all changes, whether in support of upcoming new features, refactors, bug fixes, or something else, land on the *main* branch where they are merged together.

Edge releases contain the latest code in from the main branch at the point in time when they were cut. This means they have the latest features and fixes, but it also means they don't have stability guarantees. Upgrading between edge releases may involve breaking changes, and may involve partial features that are later modified or backed out.

Stable releases are designed to introduce minimal change to an existing system and come with documented stability guarantees. In stable releases, we take the specific bug fix changes (or, occasionally, feature additions) and "back port" these changes against the code in the previous stable version. This minimizes the overall delta between releases. We also do any additional work required to ensure that upgrades and rollbacks between stable releases are seamless and contain no breaking changes.

In short: edge releases optimize for being on the bleeding edge; stable releases optimize for stability and minimal change.

### What is Buoyant Enterprise for Linkerd?

BEL is our production-ready distribution of Linkerd, from the same team that builds Linkerd itself. It is a distribution of Linkerd plus a set of additional tools, features, and testing designed for sustained, production use. 

Despite its name, BEL can be used by anyone, enterprise or not, to build secure, reliable, and observable Kubernetes platforms.

BEL does feature a set of additional tools and features, including a dynamic zone-aware load balancer which can dramatically cut cloud spend without the reliability sacrifices of Topology-Aware Hints; a Kubernetes operator that automates Linkerd installs and upgrades; tools for managing Linkerd's comprehensive L7 authorization policies; and much more. See the [BEL docs for more](https://docs.buoyant.io/buoyant-enterprise-linkerd/latest/overview/).

### Why are you making this change? 

This is necessary for us to continue delivering this incredible project that so many thriving businesses are built on.

The Linkerd maintainers are some of the world's best engineers who tackle some of computing's most difficult problems. We're a small but mighty team of software engineers, network programmers, and distributed systems experts. We love our jobs, but this is also some of the hardest work in the cloud native ecosystem. And the quality bar for Linkerd is astronomical! Linkerd is used to power 911 call centers, to handle sensitive medical and financial data, and to protect Kubernetes clusters around the world.

We have SO MUCH exciting and impactful work ahead of us—as we said above, Linkerd 2.15 is just the next step in a long journey of compounding value.  We also have an obligation to make sure every Linkerd adopter is getting reliable, secure software that they can deploy with complete confidence, and we take that responsibility seriously.

But for us to do that work and maintain that quality, there's only one sustainable path: we need the many, many companies around the world that are building their businesses on top of Linkerd to do their part to fund the project. Continuing to deliver this incredible project for free while some of the world's largest businesses use it without supporting it is not a long-term viable path for Linkerd.

### What about future Linkerd 2.14 point releases?

Linkerd 2.14.10 was the last stable release we shipped to open source. Upcoming Linkerd 2.14 point releases, including 2.14.11 and beyond, will only be available as BEL releases.

### Who has to pay for BEL?

Anyone can download and deploy BEL in non-production situations. Companies with fewer than 50 employees do not need to pay to use BEL in production.

After May 21st, 2024, companies with 50 or more employees must pay to use BEL in production. See [our pricing page](https://buoyant.io/pricing/) for details.

### How much does BEL cost?

See [our pricing page](https://buoyant.io/pricing/) for details.

It's important to us to accommodate non-profits, high-volume use cases, and other organizations with unique needs. If you're in one of these categories, please [contact us to start a conversation](https://buoyant.io/contact).

### How do we pay for BEL?

BEL is purchasable on an annual basis. [Contact us](https://buoyant.io/contact) to get the process started. 

### I love Linkerd but I'm just an engineer. I don't know how to ask my company to pay for something. What should I do?

Don't worry, we handle this all the time. We'll help you get sorted out. [Contact us](https://buoyant.io/contact) to get the process started. We're friendly, we promise!

### I hate this. Who can I yell at?

Hi there, this is William Morgan, Buoyant's CEO. You can email me at william@buoyant.io or you can come find me in person at Kubecon EU in Paris next month. I'm the one who made this decision. (I'm also, perhaps not coincidentally, the person who pays the Linkerd maintainers.) Come talk to me and I'm happy to explain exactly why I did it and all the alternatives and options we considered.

‍

‍