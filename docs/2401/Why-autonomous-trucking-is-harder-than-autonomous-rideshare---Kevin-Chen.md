<!--yml
category: 未分类
date: 2024-05-27 14:46:27
-->

# Why autonomous trucking is harder than autonomous rideshare — Kevin Chen

> 来源：[https://kevinchen.co/blog/autonomous-trucking-harder-than-rideshare/](https://kevinchen.co/blog/autonomous-trucking-harder-than-rideshare/)

Recently, The Verge asked, [“where are all the robot trucks?”](https://www.theverge.com/23981006/autonomous-truck-semi-driverless-aurora-kodiak-infrastructure)

It’s a good question.

Trucking was supposed to be the ideal first application of autonomous driving. Freeways contain predictable, highly structured driving scenarios. An autonomous truck would not have to deal with the complexities of intersections and two-way traffic. It could easily drive hundreds of miles without encountering a single pedestrian.

[](/assets/blog/av-truck-difficulty/truck-watercolor-hero-1792w.jpg)

DALL-E 3 prompt: “Generate an artistic, landscape aspect ratio watercolor painting of a truck with a bright red cab, pulling a white trailer. The truck drives uphill on an empty, rural highway during wintertime, lined with evergreen trees and a snow bank on a foggy, cloudy day.”

The trucks could also be commercially viable with only freeway driving capability, or freeways plus a short segment of surface streets needed to reach a [transfer hub](https://web.archive.org/web/20180201192005/https://medium.com/@UberATG/the-future-of-trucking-b3d2ea0d2db9). The AV company would only need to deal with a limited set of businesses as customers, bypassing the messiness of supporting a large pool of consumers inherent to the B2C model.

Autonomous trucks would not be subject to rest requirements. As [*The Verge*](https://www.theverge.com/23981006/autonomous-truck-semi-driverless-aurora-kodiak-infrastructure) notes, “truck operators are allowed to drive a maximum of 11 hours a day and have to take a 30-minute rest after eight consecutive hours behind the wheel. Autonomous trucks would face no such restrictions,” enabling them to provide a service that would be literally unbeatable by a human driver.

* * *

If you had asked me in 2018, when I first started working in the AV industry, I would’ve bet that driverless trucks would be the first vehicle type to achieve a million-mile driverless deployment. Aurora even [pivoted](https://techcrunch.com/2020/07/20/aurora-expands-to-texas-in-bid-to-ramp-up-self-driving-truck-efforts/) their entire company to trucking in 2020, believing it to be easier than city driving.

Yet sitting here in 2024, we know that both Waymo and Cruise have driven millions of miles on city streets — a large portion in the dense urban environment of San Francisco — and there are no driverless truck deployments. What happened?

I think the problem is that *driverless* autonomous trucking is simply harder than driverless rideshare.

The trucking problem appears easier at the outset, and indeed many AV developers quickly reach their initial milestones, giving them false confidence. But the difficulty ramps up sharply when the developer starts working on the last bit of polish. They encounter thorny problems related to the high speeds on freeways and trucks’ size, which must be solved before taking the human out of the driver’s seat.

## What is the driverless bar?

Here’s a simplistic framework:

*   No driver in the vehicle.
*   No guarantee of a timely response from remote operators or backend services.
*   Therefore, all safety-critical decisions must be made by the onboard computer alone.
*   Under these constraints, the system still meets or exceeds human safety level.

This is a really, really high bar. For example, on surface streets, this means the system *on its own* is capable of driving at least 100k miles without property damage and 40M miles without fatality.

The system can still have flaws, but virtually all of those problems must result in a lack of progress, rather than collision or injury. In short, while the system may not know the *right* thing to do in every scenario, it should never do the *wrong* thing.

(There are several high quality safety frameworks for those interested in a rigorous definition. ^(^(It’s beyond the scope of this post.)))

Now, let’s look at each aspect of trucking to see how it exacerbates these challenges.

## Truck-specific challenges

### Stopping distance vs. sensing range

The required sensor capability for an autonomous vehicle is determined by the most challenging scenario that the vehicle needs to handle. A major challenge in trucking is stopping behind a stalled vehicle or large debris in a travel lane. To avoid collision, the autonomous vehicle would need a sensing range greater than or equal to its stopping distance.

We’ll make a simplifying assumption that stopping distance defines the minimum detection range requirements. A driverless-quality perception system needs perfect [recall](https://en.wikipedia.org/wiki/Precision_and_recall) on other vehicles within the vehicle’s worst-case stopping distance.

Passenger vehicles can decelerate up to –8 m/s². Trucks can only achieve around –4 m/s², which increases the stopping distance and puts the sensing range requirement right at the edge of what today’s sensors can deliver.

Here are the sight stopping distances for an empty truck in dry conditions on roads of varying grade:

Sight stopping distances defined as the distance needed to stop assuming a 2.5-second reaction time with no braking, followed by maximum braking. The distance is computed for an empty truck in dry conditions on roads of varying grade. Stopping distance increases in wet weather or when driving downhill with a load (not shown).

Now let’s compare these distances with the capabilities of various sensors:

**Lidar** sensors provide trustworthy 3D data because they take direct measurements based on physical principles. They have a usable range of around 200–250 meters, plenty for city driving but not enough for every truck use case. Lidar detection models may also need to accumulate multiple scans/frames over time to detect faraway objects reliably, especially for smaller items like debris, further decreasing the usable detection range.

Note that some solid-state lidars claim significantly more range than 250 meters. These numbers are collected under ideal conditions; for computing minimum sensing capability, we are interested in the range that can provide perfect recall and really great precision. For example, the lidar may be unable to reach its maximum range over the entire field of view, or may require undesirable trade-offs like a scan pattern that reduces point density and field of view to achieve more range.

**Radar** can see farther than lidar. For example, [this high-end ZF radar](https://www.zf.com/products/en/cars/products_64255.html) claims vehicle detections up to 350 meters away. Radar is great for tracking moving vehicles, but has trouble distinguishing between stationary vehicles and other background objects. Tesla Autopilot has infamously [shown](https://slate.com/technology/2021/08/teslas-allegedly-hitting-emergency-vehicles-why-it-could-be-happening.html) this problem by braking for overpasses and running into stalled vehicles. “Imaging” radars like the ZF device will do better than the radars on production vehicles. They still do not have the azimuth resolution to separate objects beyond 200 meters, where radar input is most needed.

**Cameras** can detect faraway objects as long as there are enough pixels on the object, which leads to the selection of cameras with high resolution and a narrow field of view (telephoto lens). A vehicle will carry multiple narrow cameras for full coverage during turns. However, cameras cannot measure distance or speed directly.

A combined camera + radar system using machine learning probably has the best chance here, especially with recent advances in ML-based early fusion, but would need to perform well enough to serve as the primary detection source beyond 200 meters. Training such a model is closer to an open problem than simply receiving that data from a lidar.

In summary, we don’t appear to have any sensing solutions with the performance needed for trucks to meet the driverless bar.

### Controls

Controlling a passenger vehicle — determining the amount of steering and throttle input to make the vehicle follow a trajectory — is a simpler problem than controlling a truck. For example, passenger vehicles are generally modeled as a single rigid body, while a truck and its trailer can move separately. The planner and controller need to account for this when making sharp turns and, in extreme low-friction conditions, to avoid [jackknifing](https://en.wikipedia.org/wiki/Jackknifing).

These features come in addition to all the usual controls challenges that also apply to passenger vehicles. They can be built but require additional development and validation time.

## Freeway-specific challenges

OK, so trucks are hard, but what about the freeway part? It may now sound appealing to build L4 freeway autonomy for passenger vehicles. However, driving on freeways also brings additional challenges on top of what is needed for city streets.

### Achieving the minimal risk condition on freeways

Autonomous vehicles are supposed to stop when they detect an internal fault or driving situation that they can’t handle. This is called the [minimal risk condition (MRC)](https://cyberlaw.stanford.edu/blog/2022/01/deep-weeds-levels-driving-automation-lurks-ambiguous-minimal-risk-condition). For example, an autonomous passenger vehicle that detects an error in the HD map or a sensor failure might be programmed to execute a pullover or stop in lane depending on the problem severity.

While MRC behaviors are [annoying](https://twitter.com/friscolive415/status/1690281516935589888) for other road users and embarrassing for the AV developer, they do not add undue risk on surface streets given the low speeds and already chaotic nature of city driving. This gives the AV developer more breathing room (within reason) to deploy a system that does not know how to handle every driving scenario perfectly, but knows enough to stay out of trouble.

It’s a different story on the freeway. Stopping in lane becomes much more dangerous with the possibility of a rear-end collision at high speed. All stopping should be planned well in advance, ideally exiting at the next ramp, or at least driving to the closest shoulder with enough room to park.

This greatly increases the scope of edge cases that need to be handled autonomously and at freeway speeds. For example:

**Scene understanding:** If the vehicle encounters an unexpected construction zone, crash site, or other non-nominal driving scenario, it’s not enough to detect and stop. Rerouting, while a viable option on surface streets, usually isn’t an option on freeways because it may be difficult or illegal to make a u-turn by the time the vehicle can see the construction. A freeway under construction is also more likely to be the only path to the destination, especially if the autonomous vehicle in question is not designed to drive on city streets.

Operational solutions are also not enough for a scaled deployment. AV developers often disallow their vehicles from routing through known problem areas gathered from manually driven scouting vehicles or announcements made by authorities. For a scaled deployment, however, it’s not reasonable to know the status of every mile of road at all times.

Therefore, the system needs to find the right path through unstructured scenarios, possibly following instructions from police directing traffic, even if it involves traffic violations such as driving on the wrong side of the road. We know that current state-of-the-art autonomous vehicles still occasionally drive into [wet concrete](https://www.sfgate.com/tech/article/cruise-stuck-wet-concrete-sf-18297946.php) and [trenches](https://twitter.com/duckduckgeese/status/1715334636103168200), which shows it is nontrivial to make a correct decision.

**Mapping:** If the lane lines have been repainted, and the system normally uses an HD map, it needs to ignore the map and build a new one on-the-fly from the perception system’s output. It needs to distinguish between mapping and perception errors.

**Uptime:** Sensor, computer, and software failures need to be virtually eliminated through redundancy and/or engineering elbow grease. The system needs almost perfect uptime. For example, it’s fine to enter a max-braking MRC when losing a sensor or restarting a software module on surface streets, provided those failures are rare. The same maneuver would be dangerous on the freeway, so the failure must be eliminated, or a fallback/redundancy developed.

These problems are not impossible to overcome. Every autonomous passenger vehicle has solved them to some extent, with the remaining edge cases punted to some combination of MRC and remote operators. The difference is that, on freeways, they need to be solved with a very high level of reliability to meet the driverless bar.

### Freeways are boring

The features that make freeways simpler — controlled access, no intersections, one-way traffic — also make [“interesting” events](https://medium.com/cruise/why-testing-self-driving-cars-in-sf-is-challenging-but-necessary-77dbe8345927) more rare. This is a double-edged sword. While the simpler environment reduces the number of software features to be developed, it also increases the iteration time and cost.

During development, “interesting” events are needed to train data-hungry ML models. For validation, each new software version to be qualified for driverless operation needs to encounter a minimum number of “interesting” events before comparisons to a human safety level can have statistical significance. Overall, iteration becomes more expensive when it takes more vehicle-hours to collect each event.

AV developers can only respond by increasing the size of their operations teams or accepting more time between software releases. (Note that simulation is not a perfect solution either. The rarity of events increases vehicle-hours run in simulation, and so far, nobody has shown a substitute for real-world miles in the context of driverless software validation.)

## Is it ever going to happen?

Trucking requires longer range sensing and more complex controls, increasing system complexity and pushing the problem to the bleeding edge of current sensing capabilities. At the same time, driving on freeways brings additional reliability requirements, raising the quality bar on every software component from mapping to scene understanding.

If both the truck form factor and the freeway domain increase the level of difficulty, then driverless trucking might be the hardest application of autonomous driving:

Now that scaled rideshare is mostly working in cities, I expect to see scaled [freeway rideshare](https://waymo.com/blog/2024/01/from-surface-streets-to-freeways-safely.html) next.

Does this mean driverless trucking will never happen? No, I still believe AV developers will overcome these challenges eventually. [Aurora](https://blog.aurora.tech/products/the-aurora-driver-is-feature-complete), [Kodiak](https://kodiak.ai/news/kodiak-unveils-industry-first-semi-truck-designed-for-scaled-driverless-deployment), and [Gatik](https://archive.ph/2023.12.15-162144/https://www.forbes.com/sites/richardbishop1/2023/12/14/krogergatik-partnership-provides--counterpoint-to-gm-cruise-debacle/) have all promised some form of driverless deployment by the end of the year. We probably won’t see anything close to a million-mile deployment in 2024 though. Getting there will require advances in sensing, machine learning, and a lot of hard work.

* * *

*Thanks to Steven W. and others for the discussions and feedback.*

* * *

Thanks for reading! If you’re enjoying my [writing](/blog/), I’d love to send you infrequent notifications for new posts via my [newsletter](https://buttondown.email/kevinchen). You’ll receive the full text of each post, plus occasional bonus content.

You can also follow me on Twitter ([@kevinchen](https://twitter.com/kevinchen)) or [subscribe via RSS](/feed.xml).