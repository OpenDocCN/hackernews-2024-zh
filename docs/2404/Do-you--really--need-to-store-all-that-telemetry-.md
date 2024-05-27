<!--yml
category: 未分类
date: 2024-05-27 13:13:11
-->

# Do you *really* need to store all that telemetry?

> 来源：[https://mattklein123.dev/2024/04/10/do-you-need-to-store-that-telemetry/](https://mattklein123.dev/2024/04/10/do-you-need-to-store-that-telemetry/)

In my [last post](https://mattklein123.dev/2024/04/03/observability-cost-crisis/) I talked about why modern observability has become so expensive. At the end of the post I posit a question: *What if by default we never send any telemetry at all?*

In this post I’m going to talk specifically about the perceived need of storing all possible telemetry data. Is this a real need? Or is this an idea that has been drilled into us for so long that we think there’s no other possible way of doing it? If we can break away from the disconcerting idea that all possible telemetry is not stored by default, what new paradigms and cost models for observing and debugging systems become possible?

## Reasons for emitting telemetry

Before diving into the question of whether we really *need* to store telemetry, it’s first important to break down the reasons why we *do* store telemetry. Telemetry emission can be generally placed into three discrete categories:

1.  **Telemetry used for debugging, monitoring, and general observability**: When people mention telemetry in the observability context, this is typically the category that they are thinking about. Emitting logs, metrics, and traces to be used for alarming on anomalous conditions as well as debugging hard to understand issues. The end goal is to prevent downtime, continuously improve performance, and make customers happy.
2.  **Telemetry required for compliance and auditing**: Security and compliance requirements may force us to store specific pieces of information for our applications (audit trails, network access logs, etc.). This class of telemetry generally never has to be queried live so can be sent to relatively cheap cold storage and queried later if absolutely necessary.
3.  **Telemetry required for business intelligence**: Established businesses have core metrics that they track for decision making. Telemetry (whether in the form of traditional metrics, analytics events, or something else) is emitted by applications and tracked in business intelligence dashboards.

For the purpose of the remainder of this post I will only consider the first category. Telemetry required for compliance, auditing, or business intelligence is very important though out of scope due to its required storage and is thus exempt from the thought exercise: can we disable everything by default?

## Are there truly unique problems?

When presented with the idea of disabling all telemetry by default, the most common skeptical response is: “What if I need to debug a problem that just happened?” This is a perfectly reasonable response and I admit that the idea of disabling all telemetry by default is likely to be deeply disconcerting to some.

Let’s start by breaking down how telemetry helps with observability of systems:

1.  **Metrics** provide a cost effective summarization of application events and can be used for understanding the overall shape of system behavior.
2.  **Logs/events** provide a more detailed trail of exact application behavior at higher relative cost to metrics. Logs are great for digging into the details of what led up to a specific event.
3.  **Traces** provide a parent/child grouping of events and timings either within a single process or across multiple processes/distributed nodes. Traces are great for understanding exact relationships between system components.

In my personal experience as an engineer working on very large scale systems for nearly 25 years, I understand the fear of disabling telemetry by default, but I also believe that there is *no problem that happens once and never happens again*. This is the critical point. If problems are not truly unique and are likely to happen again, can’t we *take steps to catch the issue the next time around with targeted telemetry specific to that problem*? Said another way, once a problem is known, can we instruct the system to collect additional information and provide it to us the next time the problem occurs?

One might then naturally ask, “if I disable all telemetry by default how would I know about problems in the first place?” While the terms APM, monitoring, observability, etc. have become muddled due to vendor marketing, it is important to differentiate the act of “monitoring” a system from the act of debugging. The ideas presented in this post do not aim to change the methods of “monitoring” for problems: we are notified of problems via customer complaints, alert conditions on metrics or event counts (that are sent all the time so that we can run alert queries), in newer systems possibly anomaly detection, and so on. A different way of thinking about “off by default” is that all signals become opt-in vs. opt-out. I.e., the data needed to run an alert query would be opted-in by default. This post is talking about what happens *after* we are notified of a problem.

I admit that the scale of the systems that I have worked on have strongly shaped my views on this topic. Between working on high performance computing in AWS EC2, to building Twitter’s edge proxy, to creating [Envoy](https://www.envoyproxy.io/) at Lyft, the order of magnitude of requests per second across the entire system and on each node has always meant that capturing detailed logging by default was never practical purely from an overhead perspective. (By default, Envoy logs at Info level and at that level emits nearly no logs after startup, a pattern I have come to use on all systems that I build, for better or worse.)

Instead, I have relied on metrics for large scale debugging and monitoring of these systems. While this has been a successful approach overall (Envoy after all is run by thousands of organizations and is considered a highly reliable piece of software), it’s not without issues. Taking Envoy as specific example (even though the same idea applies to other systems I have worked on):

1.  Envoy emits so many metrics that the cost of the metrics themselves becomes prohibitive for many organizations, to the point that many operators [statically disable a portion of Envoy metrics by default](https://www.envoyproxy.io/docs/envoy/latest/api-v3/config/metrics/v3/stats.proto#envoy-v3-api-msg-config-metrics-v3-statsmatcher). (As an aside this disabling mechanism has exposed [many bugs](https://github.com/envoyproxy/envoy/issues/8771) over the years because the metrics themselves have been erroneously used for control logic – but that is an interesting story for a different day!)
2.  When problems do crop up that require more intensive debugging, the lack of explicit logging at all times can lead to a prolonged debugging process.

On the topic of the prolonged debugging process, what does that look like using traditional telemetry production techniques?

1.  Attempt a local reproduction. If this is easily possible, using debug and trace logs to root cause the issue is usually relatively easy.
2.  If that fails (or in parallel), start with code inspection to try to manually intuit what might have happened.
3.  If local reproduction and manual intuition fails, we are then left with the painful process of manual code changes, possibly across many deployments, to add telemetry breadcrumbs to aid in catching the issue *the next time it happens* and root causing it.

Thankfully (3) above doesn’t happen often, but it does happen, necessitating painfully involved debugging experiences.

Building these types of systems over many years and debugging them has led me to think that there must be a more efficient way. And if there is a more efficient way for debugging and observing systems like Envoy, can those ideas be applied to systems at large in such a way to both improve the debugging and observation experience but *also reduce cost along the way*?

## How does adding a control plane help?

For 30 years how telemetry is produced has not changed: we define all of the data points that we need ahead of time and ship them out of the origin process, typically at large expense. If we [apply the control plane / data plane split](https://konghq.com/learning-center/cloud-connectivity/control-plane-vs-data-plane) to observability telemetry production we can fundamentally change the status quo for the first time in three decades via getting real-time access to the data we need to debug without having to store and pay for all of it:

1.  Enable/disable metrics, logs, and events at the source.
2.  Filter telemetry at the source.
3.  Live stream telemetry at the source to a web portal or CLI without storing it anywhere along the way!

Beyond simply enabling/disabling/filtering at the source, we can add *intelligence* to our telemetry production points and do so much more. We can have the control plane send instructions on a specific sequence of events to match on (effectively a finite state machine sent from control plane to data plane), followed by a set of actions to take when that sequence of events occurs. Actions could be to dump extra information, take profiles, take screenshots on mobile apps, and so on.

Continuing with the Envoy example in the previous section, instead of having to painfully make code changes and deploy them over a long period of time, if all of the built-in metrics, debug, and trace logs could be controlled dynamically with very specific match conditions and actions, think about how much shorter the debugging cycle would be. If we assume that problems are never unique and will happen again, we can start doing iterative debugging in real-time against a fleet of running instances, and likely vastly reduce the time required to root cause an issue.

Another way of thinking about it is that *this model makes debug information available with immediacy relative to the criticality of the issue as defined by the number of occurrences*. Meaning, during an incident the wait time for recurrence will be close to zero, while the wait time might be longer for very uncommon things that are likely to be considered less critical.

This approach is powerful across multiple dimensions:

1.  By targeting specific conditions, operators can avoid wading through volumes of information that are not relevant to their observation of the system, significantly reducing cognitive load.
2.  Operators can decide in real-time what is the best observation method. Would they like to see actual log lines? Would they like to see synthetic metric aggregates of log lines? Would they like to see a subset of explicit metrics?
3.  Not surprisingly, because the data needed for investigation is enabled and nothing more, the value of the transported and stored data approaches 100%, making the ROI of the more limited dataset a great value proposition.

As long as operators are able to accept the idea that problems *will* occur again, with the right system they can catch them the next time around!

## What about local storage?

Along with real-time control, we can also add *local storage* of telemetry data in an [efficient circular buffer](https://blog.bitdrift.io/post/bitdrift-ring-buffer). Typically, local storage is cheap and underutilized, allowing for “free” storage of a finite amount of historical data, that wraps automatically. Local storage provides the ability to “time travel” when a particular event is hit. How many times have we logged an error only to realize that it’s impossible to figure out the real source of the error without the preceding debug/trace logs?

A possibly obvious tradeoff of this system is that the lookback period is bounded, with the number of seconds of data available a function of the buffer size and the data rate. I still think that the benefits of the circular buffer in terms of cost efficiency outweigh the downsides of limited historical retention.

When coupled with the matching and actions described in the previous section, dumping the local buffer becomes a specific action to take, along with many other possible actions.

I will note that this is not a new idea: the Apollo spacecraft guidance computer had a system to store recently executed instructions in a circular buffer and dump them when a problem occurred to ease debugging. Similar debugging tools have been implemented for many years in embedded devices and other systems. Circular buffers have also been used in modern observability systems as part of [centralized trace aggregation systems](https://docs.honeycomb.io/manage-data-volume/sample/honeycomb-refinery/monitor/#collector-metrics). The key difference is moving local storage all the way to the edge where it is easier to scale and coupling it with a control plane, which unlocks the ability to deploy dynamic queries across many targets that can result in history being dumped to ease debugging.

Imagining the combination of local storage and real-time control being used to debug Envoy is what started me down this entire path in the first place!

## Putting it all together

The paradigms used to emit telemetry have remained unchanged for many years. To this end, engineers are very used to sending data, and expecting it to be there in case they might need it.

Adding a control plane, local storage, and not sending any data by default is a *drastic* change to how engineers think about observability. Some engineers find these ideas deeply disconcerting, and fear that the data might be needed so it should be sent no matter what.

By starting from a default of no exported telemetry, engineers have the ability to “change the game” in multiple ways:

1.  Because the major cost of telemetry production is what happens to data after it leaves the process, not the production within a process, we can free developers from thinking about cost at all. Emit as many metrics, logs, events, traces, etc. as desired. It’s effectively free!
2.  Real-time control via the control plane allows telemetry to be enabled and disabled on-demand, whether to temporarily debug an issue, permanently generate (synthetic) metrics to populate dashboards and alerts, etc. Ad-hoc investigations can lead to dynamic production of telemetry, solely for the purpose of solving the issue at hand, before being disabled again.
3.  Critical telemetry needed for auditing and/or monitoring *can* be sent by default by request of the operator. Furthermore, the definition of critical telemetry can be changed without the need for any code changes or deployments.

It’s true that changing how engineers and operators think about telemetry production and consumption is a tall order, but in my opinion the benefits of this change far outweigh the challenges of relearning fundamental techniques: namely, vastly decreased cost and vastly *increased* fidelity for helping to understand and fix product issues that are actively problematic.

I founded [bitdrift](https://bitdrift.io/) last year with the singular purpose of moving away from 30 year old telemetry production paradigms and tackling this problem head on. Is it indeed possible to not send any telemetry by default and still successfully observe large scale production systems?

Since we [launched Capture](https://blog.bitdrift.io/post/honey-i-shrunk-the-telemetry) publicly a few months ago it’s been super interesting to start talking to potential customers; the responses we have received vary widely, ranging from those that immediately buy into the idea of off by default to those that are inherently skeptical, asking the same (completely reasonable!) question posed above: “But what if I *really* need to look at some of that data after the fact?”

I continue to be extremely excited by the idea of what we can build if we start from the position of accepting that bugs *will* happen again, not sending any telemetry by default, and proceeding from there.

So, say it with me: “No, I don’t *really* need to store all of that telemetry!” Or, [“Just say no to the logging industrial complex!”](https://twitter.com/mattklein123/status/1775632985192349754)