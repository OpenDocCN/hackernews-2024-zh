<!--yml
category: 未分类
date: 2024-05-28 18:15:08
-->

# Lisa Su Says The "Team Is On It" After Tweet About Open-Source AMD GPU Firmware - Phoronix

> 来源：[https://www.phoronix.com/news/Lisa-Su-Tweet-OSS-Firmware](https://www.phoronix.com/news/Lisa-Su-Tweet-OSS-Firmware)

George Hotz with Tiny Corp that is working on Tinygrad and TinyBox for interesting developments in the open-source AI space has previously

[called out AMD over ROCm issues](https://www.phoronix.com/news/Lisa-Su-ROCm-Commitment)

. Yesterday yielded new tweets by "the tiny corp" over AI training runs crashing with MES errors and then called for AMD open-sourcing the firmware to which AMD CEO Lisa Su has responded.

Tiny Corp

[tweeted](https://twitter.com/__tinygrad__/status/1765085827946942923)

over the issues:

> "At it stands, I'm not okay with shipping the 7900XTX platform. What should we do?
> ...
> We are not AMD's QA team, and we have no relationship with them. I saw some stuff last year that gave me hope, but this platform has been out for 14 months now, and there's still serious issues.
> 
> It upsets me that the MES isn't open source. While more stuff is open source than NVIDIA, if there's blobs we do not own the hardware, and I don't feel great about investing time into this.
> 
> The compiler bug today is icing on the cake. At first I thought it was the `launch_bounds` feature, but it looks like it can be triggered without that. Not being able to trust a compiler undermines so much trust in the entire platform.
> 
> It would set us back, but maybe we should switch to either 3090s or @intel
> GPUs. Either way, we aren't shipping the tinybox (or ordering bulk 7900XTX) until this is figured out.
> ...
> I'm sure @AMD doesn't want these bugs either, but they are focused in the wrong place.
> 
> They should immediately stop development of high end ML libraries and fix their basic shit. Their compiler and driver have bugs, why should anyone spend a minute building anything on top until that stuff is addressed.
> 
> It looks like a problem with testing methodology. Fuzzers would catch these issues. What a waste to invest in higher level crap until the basics are good."

To which was then

[another tweet](https://twitter.com/LisaSu/status/1765209899418423751?s=09)

:

> "If AMD open sources their firmware, I'll fix their LLVM spilling bug and write a fuzzer for HSA. Otherwise, it's not worth putting tons of effort into fixing bugs on a platform you don't own."

Specifically the firmware they are after is currently at least the MES firmware for the Micro Engine Scheduler "MES" of the GPU.

Lisa

[responded](https://twitter.com/LisaSu/status/1765209899418423751)

:

> "Thanks for the collaboration and feedback. We are all in to get you a good solution. Team is on it."

We'll see what comes of this and apparently a call is scheduled for today between Tiny Corp and AMD. It will be interesting to see if AMD ends up open-sourcing their MES firmware or has some interim solution. Due to legal/code review it's unlikely to be a quick process and even more unlikely they would open up large swaths of their firmware. But due to customer demand they have been working on more open-source firmware in general such as with Sound Open Firmware support, the interesting openSIL efforts on the CPU side, and last year did

[publish their SEV firmware as open-source](https://www.phoronix.com/news/AMD-SEV-Firmware-Open-Source)

. Stay tuned.

**Update:** [Tiny Corp At "70%" Confidence For AMD To Open-Source Some Relevant GPU Firmware](https://www.phoronix.com/news/Tiny-Corp-70p-MES-Firmware)