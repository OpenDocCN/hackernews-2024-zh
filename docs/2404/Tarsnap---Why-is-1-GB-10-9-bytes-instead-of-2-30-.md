<!--yml
category: 未分类
date: 2024-05-27 13:05:37
-->

# Tarsnap - Why is 1 GB 10^9 bytes instead of 2^30?

> 来源：[https://www.tarsnap.com/GB-why.html](https://www.tarsnap.com/GB-why.html)

Because in 1960, the [Bureau International des Poids et Mesures](https://www.bipm.org) decided that the SI prefix G- meant 10⁹.

#### But it means 2^(30), really!

No it doesn't. Let's look at some examples:

| A Gm is... | 10⁹ meters |
| A GW is... | 10⁹ watts |
| A GA is... | 10⁹ amperes |
| A Gmol is... | 10⁹ moles |

#### But it's different in computing!

Let's look at some more examples:

| A 2.2 GHz CPU operates at... | 2.2 × 10⁹ cycles per second |
| 1 Gbps Ethernet transmits data at... | 10⁹ bits per second |
| The 2.4 GHz band which wireless ethernet operates within lies... | between 2.4 × 10⁹ and 2.5 × 10⁹ Hz |
| A 200 GB hard drive holds... | 200 × 10⁹ bytes of data |

#### But what about RAM?

You're right: If you buy a "1 GB" stick of RAM, it will hold 2^(30) bytes of data.

However, this is a special case: Unlike everything else in the world of computing, RAM is addressed in hardware. When you're designing a piece of silicon, you want to have N address lines and have every combination of zeroes and ones map to a memory location — to do otherwise would make the logic far more complicated. *Nothing else* is addressed this way.

Finally, even for RAM calling 2^(30) bytes "1 GB" isn't really proper; instead, the IEC binary multiplier prefix "Gi-" should be used.

#### This is all a conspiracy by hard drive manufacturers who want to cheat us out of the disk space we're paying for!

We all love good conspiracy theories... but really, this isn't about evil megacorporations trying to cheat you. Hard drive prices are determined almost entirely by competition between manufacturers, so if hard drives were labelled in GiB instead of being labelled in GB, we'd be paying the same number of dollars for the same number of bytes anyway — if this really was a global conspiracy, it would be one of the dumbest conspiracies ever.