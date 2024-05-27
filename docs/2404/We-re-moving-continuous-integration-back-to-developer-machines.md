<!--yml
category: 未分类
date: 2024-05-27 13:37:47
-->

# We're moving continuous integration back to developer machines

> 来源：[https://world.hey.com/dhh/we-re-moving-continuous-integration-back-to-developer-machines-3ac6c611](https://world.hey.com/dhh/we-re-moving-continuous-integration-back-to-developer-machines-3ac6c611)

Between running Rubocop style rules, Brakeman security scans, and model-controller-system tests, it takes our remote BuildKite-based continuous integration setup about 5m30s to verify a code change is ready to ship for HEY. My Intel 14900K-based Linux box can do that in less than half the time (and my M3 Max isn't that much slower!). So we're going to drop the remote runners and just bring continuous integration back to developer machines at 37signals.

It's remarkable how big of a leap multi-core developer machines have taken over the last five-to-seven years or so. Running all these checks and validations in a reasonable time on a local machine would have been unthinkable not too long ago. But the 14900K has over 20 cores, the M3 Max has 16, and even a lowly M2 MacBook has 8\. They're all capable of doing a tremendous amount of parallelized work that would have seem fantastical to do locally in the mid 2010s.

HEY is a pretty substantial code base too. About 55,000 lines of Ruby code, which is verified by some 5,000 test cases along with another 300-some system tests. Virtually all of these tests go through the full-stack and hit the database. These are not mocked to the hilt.

To me, the most satisfying part of the improved performance of modern developer CPUs is the possibility to simplify our stacks. Installing, operating, and caring for a remote CI setup is a substantial complication. Either you do it on your own hardware, and deal with that complexity directly, or you pay through the nose for a cloud-based setup. Getting to flush all of it down the simplification drain is an amazing step forward.

As always, the simplified future is not evenly distributed. I can't see the likes of Shopify or GitHub being able to run the full battery of tests against their millions of lines of code locally any time soon. But 99.99% of all web apps are much closer to HEY in breadth than they are to those behemoths. And small teams ought to remove all the moving parts possible. Never aspire to a more complicated stack than what your application calls for.

So we need to keep burning those

[bridges of complexity](https://world.hey.com/dhh/introducing-propshaft-ee60f4f6)

once we get to the other side. I can't wait to set fire to every single one of the remote continuous integration bridges we have here at 37signals. Progress is a bonfire.