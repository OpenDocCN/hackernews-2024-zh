<!--yml
category: 未分类
date: 2024-05-29 12:38:43
-->

# CUDA is Still a Giant Moat for NVIDIA - by James Wang

> 来源：[https://weightythoughts.com/p/cuda-is-still-a-giant-moat-for-nvidia](https://weightythoughts.com/p/cuda-is-still-a-giant-moat-for-nvidia)

In a way, this is a post-**[GTC](https://www.nvidia.com/gtc/)** update. However, **[there are already great overviews of the event](https://www.fabricatedknowledge.com/p/jensens-world-compressing-reality-be4?utm_campaign=email-post&r=4de3d&utm_source=substack&utm_medium=email)**. Here, I’ll explain more about why I’ve been a somewhat lonely voice (until recently) about why other players like **[SMIC/Huawei/etc](https://weightythoughts.com/p/fear-and-loathing-in-7nm)** improving on their hardware still doesn’t displace NVIDIA at all.

That view mainly relates to CUDA’s ecosystem and NVIDIA’s proprietary interconnects.

CUDA remains as dominant as ever, and while the announcements about **[Blackwell](https://nvidianews.nvidia.com/news/nvidia-blackwell-platform-arrives-to-power-a-new-era-of-computing)** are interesting, it’s a big incremental step forward rather than something truly groundbreaking. The much more compelling announcement related to how much more NVIDIA is creating a lead in interconnects for large-scale computing (NVLink especially).

I’ll get into a deeper post about interconnects, which is quite involved, but I think I should finally more fully explain my views on CUDA. Instead of a pure technical explanation, let me tell a story to make it more visceral.

I once took a graduate class at UC Berkeley on high-performance and parallel computing. The students were mainly PhDs in computer science, computer engineering, and statistics (machine learning). I was a weirdo taking it during my MBA—nothing in the rules *technically* said I couldn’t fulfill my electives with this and PhD seminars in computer science and statistics. This was before I got my own Master’s degree in computer science.

Anyway, we went through the concepts and techniques involved in parallel computing, in terms of memory architecture, deep dives in matrix multiplication (which is fundamental to most modern computing, including current AI), locality, SIMD/CPU vectorization, lock/etc primitives. These were quite theoretical, but we also discussed and implemented specific technologies including OpenMP, OpenCL/GL, and the like.

All of them were a pain in the ass.

Then, still fairly early in the term, a former PhD student of the professor who now worked at NVIDIA came in to guest lecture on GPUs and CUDA.

It’s really difficult to explain to non-programmers—or even programmers who don’t deal with massive compute problems—just how night-and-day the difference is between CUDA and its alternatives.

Some of it has extremely nice primitives for standard math and atomic memory operations, while also offering easy explicit memory allocation… and a ton of other stuff that for most of my audience is completely meaningless. I’ll just say it gives you super pleasant tools.

But beyond that, because NVIDIA controls CUDA *and* their GPUs, they control the entire stack. Software, firmware, and hardware. This is the same thing that gives Apple a killer advantage on their platforms. NVIDIA has the same thing, and even aside from programmer productivity/ease (which, as we’ve learned over the years, is itself huge), stuff in CUDA just magically works much better. Behind the scenes, CUDA performs significant optimizations.

Additionally, as new GPUs come out, as they inevitably do, CUDA continues to be forward and backward-compatible (depending on the features you use), and you continue to reap the benefits of even better hardware and firmware that NVIDIA continues to pump out.

One of the primary arguments about why the hardware is what matters in the AI compute war is that it’s easy to see why that is a “hard” barrier. If you can’t make the hardware, well, you’re just out of luck.

However, if that were the case, we shouldn’t be seeing such tepid adoption (if it exists at all) of AMD in AI workloads. AMD is worse, sure, but it’s not *that* far behind, and if you’re stuck with massive waitlists and impossible-to-get NVIDIA GPUs, why wouldn’t you do AMD?

The answer is CUDA.

AMD has its own “drop-in” replacement for CUDA called **[ROCm](https://www.amd.com/en/products/software/rocm.html)**, which for the longest time didn’t even support floating-point operations (meaning, for non-technical folks, decimals). It has a lot of the same primitives, it just sucks.

Even if you were an ambitious PhD in computer science who is also an outstanding programmer who somehow wants to write a new AI-related library in OpenCL or Vulcan or whatever, you still wouldn’t get the performance you could with CUDA (at least not without a massive amount of extra low-level programming effort).

In reality, although I know this is strange to non-PhD/technical/academic readers, most PhDs in computer science, engineering, and statistics actually kind of suck at programming, unless their specialty is programming languages. They aren’t motivated by trying to do fancy stuff with writing code. Additionally, they have enough headaches and time pressure, why would they deliberately choose to make life harder on themselves?

This is why a lot of bleeding-edge, prototype machine learning algorithms are written in and only available in R (which can make one internally scream, but I’ve been forced to get extremely familiar with it), which offers no performance benefits at all. But for stuff that is meant to run on “real” workloads to try to demonstrate cutting-edge performance on massive datasets, that’s in the CUDA ecosystem.

One can liken it to Adobe/Microsoft/AutoCAD software being used extensively in educational institutions, and that makes them also very sticky in industry (which in turn reinforces why schools use them). Of course, it’s even more painful to switch from CUDA in this case.

Tensorflow and PyTorch nowadays support platforms apart from CUDA and (in turn) NVIDIA GPUs. However, for the longest time, they didn’t—or, at least, they didn’t if you wanted to finish your workloads anytime this century and not run them on CPUs.

Now that these fundamental tools support other platforms may sound like it’s countering my argument, but it’s more to illustrate that years of tools, low-level libraries, and the like assumed that everyone was using NVIDIA GPUs.

There were PhD blog posts/guides during the great GPU shortage in 2020-2022 about buying used RTX 2090s for poor graduate students. **[Today, more updated guides still exclusively talk about NVIDIA GPUs](https://timdettmers.com/2023/01/30/which-gpu-for-deep-learning/comment-page-1/)**.

I’ve heard arguments about how China has infinite money and resources to throw at conquering AI (… which is an assertion that is also debatable, especially now), they can just port everything to a new platform like **[Huawei’s CANN](https://github.com/opencv/opencv/wiki/Huawei-CANN-Backend)**.

Aside from the hand-wavy nature of the rather insane proposal (which I think is mainly “simple” only to hardware people) to just “port everything,” that still doesn’t solve the problem. The ***[leading edge](https://weightythoughts.com/p/compute-is-overrated-as-ais-bottleneck)*** **[of AI software and models, which is](https://weightythoughts.com/p/compute-is-overrated-as-ais-bottleneck)** ***[really](https://weightythoughts.com/p/compute-is-overrated-as-ais-bottleneck)*** **[what’s driving the AI revolution, not hardware/compute](https://weightythoughts.com/p/compute-is-overrated-as-ais-bottleneck)**, is still within the CUDA ecosystem.

If you’re ok with always lagging, sure, that’s ok, but if AI’s frontier moves as fast as everyone seems to think it will (which I do too), it’ll be similar to Moore’s Law pre-2010\. Being generations behind constantly is a *bad thing* in that case and not the key to any kind of technological innovation, let alone leadership.

**[Chip War](https://en.wikipedia.org/wiki/Chip_War:_The_Fight_for_the_World%27s_Most_Critical_Technology)** has a great section on how the Soviet Union tried a “just copy/steal” strategy in semiconductors and fell hopelessly behind because of it. It’s a great theoretical idea to just copy/steal and fast-follow, but semiconductors, AI, and other “harder technologies” require building human and intellectual capital that will get better with time. From there, you need to have the prior generation to keep up with ever-increasing complexity and difficulty as these things get more advanced.

The idea that NVIDIA is dominant isn’t exactly a controversial take **[immediately after a GTC that has been likened to a Taylor Swift concert](https://www.businessinsider.com/nvidia-ceo-taylor-swift-nod-shows-aware-of-tech-reputation-2024-3)**.

However, the *reasons* continue to be the same, and I think are worth emphasizing. Will CUDA one day lose its advantage? Will Huawei’s CANN or various open-source projects like Vulcan be able to take away its sheer dominance in the AI field, and thus degrade NVIDIA’s absolute stranglehold on AI compute?

Maybe. But I still haven’t seen a distinctive shift. The reasons aren’t in numbers or simplistic top-down thought experiments about how NVIDIA can lose its leadership.

The reasons for NVIDIA’s dominance are years and billions of dollars in investment in the CUDA ecosystem, evangelism, and education of the community that builds AI. Even if it were fading, it would take quite a while for the impacts of such a heavy, long-term strategy to go away.

As it stands, NVIDIA is only retrenching its strength within the space. And the reason (apart from interconnects—which, as said, I promise a piece at some point) is CUDA and its ecosystem.