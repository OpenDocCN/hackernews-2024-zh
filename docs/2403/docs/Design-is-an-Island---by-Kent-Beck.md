<!--yml
category: 未分类
date: 2024-05-29 12:31:27
-->

# Design is an Island - by Kent Beck

> 来源：[https://tidyfirst.substack.com/p/design-is-an-island](https://tidyfirst.substack.com/p/design-is-an-island)

> First published April 2009\. This was a period when I was working consistently on the material that would become, a decade and a half later, Tidy First?.

If design is responsive motion in a space of all possible designs, then some designs are acceptable and some are not. Designing, then, is like walking an island. As long as you don’t get your feet wet, the design is okay. (This is a visual way of describing design as an optimization problem, but I’ll stick with the metaphor because it turns out to be surprisingly apt.)

The “water line” in design isn’t fixed and rigid. Just as tides change the sea level, tides affect what is an acceptable design. The holiday season is high tide for many retailers. Designs that are acceptable at other times of the year break down in the 10% of the year when you do 50% of your business. Changes that are acceptable in February or March introduce unacceptable risk in October and November.

If you are moving from one design to another in safe steps, it may be acceptable to get your feet wet temporarily as long as you don’t drown when the next tide comes in. Changes introduced during less stressed times such as overnight fit this picture. On the other hand, if you don’t have less stressed times, then your options for moving from one design to another are reduced. These constraints on design change become constraints on the design.

Climbing higher on an island requires effort, just as improving designs require effort. If you are above the waterline and in no danger, further investment in design may be money better spent elsewhere. I cringed a bit as I wrote this, as I like leaving designs as clean as possible. Even then I only climb the peak I’m on, refining the design for the current set of requirements. I don’t seek out the highest peak on the island, or in the world. This mountain may be good enough for a year, or even forever.

If the sea level is high enough there may only be a single island, a single family of designs that makes sense. More often, though, design is an archipelago, with contrasting islands shouldering into the open air. For example, SOA and REST may both work for a distributed application. Rising waters in the form of higher performance, reliability, or a changing programmer community might drown one or the other option. Many of the gentlemanly discussions about SOA and REST seem to rely on hypotheticals, “Yes, you idiot, but if the water level rises 25 meters your option will clearly be under water.” It would be more productive to characterize the aspects contributing to sea level and discuss ways of safely moving back and forth.

Some have proposed the metaphor (or more than metaphor) of attractors for design. Acceptable designs for a particular set of constraints seem to cluster around a few “styles”. I don’t have the background or time to expand on this now, but it may be fertile ground for future exploration: why do designs cluster to form islands?

Returning to our watery tropical paradise (as long as you are visualizing islands you may as well visualize pleasant ones), another common occurence in software development is an earthquake which jumbles and restructures the landscape. The one that struck me yesterday was testability. I had a just fine, dry feet design until I went to write a unit test. At that, the whole landscape changed. It wasn’t just that the water level rose a little higher. By introducing the need to unit test, whole islands sank like Atlantis into the sea. Other designs that had seemed impossibly remote rose up as acceptable alternatives. As you can see, this process is accompanied by a fair amount of smoke and heat.

Decreasing downtime is another example of tectonic change in design. Reducing downtime by an order of magnitude requires not just a better design, but a different design. Increasing reliability changes the shape of the seabed.

Before we return to the mundane world from the sun and surf I’ll stretch the metaphor one more bit. Changing in safe steps (a topic I see I need to address soon) sometimes requires moving underwater. When the current island starts to sink, moving to another island is necessary. This may require special preparations to move about underwater for a time. For example, if safe steps require reducing the efficiency of servers, you might bring in extra servers to handle the load while taking the steps.

Moving to another island (escaping a local maxima) often requires making a design worse in various ways–increasing duplication, reducing efficiency, making code harder to understand–before it can be made better again. If you can leap from island to island, so much the better, but prudence dictates safe steps in most circumstances.

Given the prevalence of tectonic activity in software development, some underwater travel is inevitable. Better to prepare for it and limit the amount of time spent breathing from a tank than spend exponential effort erecting a tower on a platform on a (soon to be former) mountain. Being prepared for both overland and underwater travel is the best preparation for software design success.