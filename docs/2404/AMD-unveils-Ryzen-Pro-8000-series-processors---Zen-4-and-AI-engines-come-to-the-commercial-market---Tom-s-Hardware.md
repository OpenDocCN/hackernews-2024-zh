<!--yml
category: 未分类
date: 2024-05-27 13:15:05
-->

# AMD unveils Ryzen Pro 8000-series processors — Zen 4 and AI engines come to the commercial market | Tom's Hardware

> 来源：[https://www.tomshardware.com/pc-components/cpus/amd-unveils-ryzen-pro-8040-hawk-point-and-8000-series-phoenix-processors-ai-engines-come-to-the-commercial-market](https://www.tomshardware.com/pc-components/cpus/amd-unveils-ryzen-pro-8040-hawk-point-and-8000-series-phoenix-processors-ai-engines-come-to-the-commercial-market)

AMD announced its Ryzen Pro portfolio today as it extended the ['Hawk Point' 8040-series](https://www.tomshardware.com/pc-components/cpus/the-refresh-that-wasnt-amd-announces-hawk-point-ryzen-8040-series-with-zen-4-rdna3-and-xdna-teases-strix-point) to commercial laptop and workstation users while simultaneously offering its [Ryzen 8000 'Phoenix' APU models](https://www.tomshardware.com/pc-components/cpus/amd-launches-ryzen-8000g-phoenix-apus-brings-ai-to-the-desktop-pc-reveals-zen-4c-clocks-for-the-first-time) for commercial desktop PCs. As we've seen in the past, the Pro series is based on AMD's existing consumer-oriented processor models but comes with additional features that tailor them for the commercial market. With the consumer versions of these processors, AMD was the first x86 company to bring its AI-processing neural processing unit (NPU) to the mobile and desktop PC market. Those same AI acceleration features are now headed to commercial users, allowing AMD to lay claim to being the first with professional CPUs armed with NPUs for laptops and workstations.

AMD's in-built XNDA engine powers the NPU, and the company touts that its mobile processors have an advantage over Intel's competing Core Ultra processors with 16 TOPS of NPU performance, outperforming the Meteor Lake NPU's 11 TOPS. AMD also maintains a slimmer lead in overall system TOPS, which includes both the CPU and GPU AI processing power, taking the lead with 39 TOPS over Intel's 34 TOPS. AMD's desktop Ryzen 8000 APUs also have an in-built NPU engine that delivers 16 TOPS, whereas Intel has yet to release a processor for desktop PCs with an integrated AI engine.

Notably, neither AMD nor Intel's chips meet [Microsoft](https://www.tomshardware.com/tag/microsoft)'s next-gen [AI PC](https://www.tomshardware.com/tag/ai-pc) requirement for [45 TOPS of performance from the NPU](https://www.tomshardware.com/pc-components/cpus/intel-shares-new-ai-pc-definition-launches-ai-pc-acceleration-programs-and-core-ultra-meteor-lake-nuc-developer-kits-at-ai-conference), though both companies say their next-gen chips, [Strix Point and Lunar Lake](https://www.tomshardware.com/pc-components/cpus/intel-says-lunar-lake-will-have-100-tops-of-ai-performance-45-tops-from-the-npu-alone-meeting-requirement-for-next-gen-ai-pcs), respectively, will meet that bar. The 45 TOPS requirement is meant to enable Microsoft to run AI elements of Copilot locally, and it isn't clear how a future Windows update to enable that functionality will pan out with the current generations of AMD and Intel processors. [Qualcomm's Snapdragon X Elite](https://www.tomshardware.com/laptops/i-went-hands-on-with-two-different-qualcomm-snapdragon-x-elite-chips-as-the-company-claims-it-will-beat-intels-core-ultra) Arm chips will debut with 45 TOPS of performance from its NPU in the middle of the year. Apple's M3 processors provide 18 TOPS of NPU performance but obviously aren't impacted by Microsoft's requirements. 

The ability to run AI workloads locally helps alleviate critical privacy concerns in the commercial space. It also delivers latency, performance, and battery life advantages for AI applications (power efficiency is dramatically better with the NPU than running the same AI workload on the GPU). However, unleashing that performance requires tightly coupled software solutions, and in many respects, the early war for AI dominance will come down to developer partnerships.

The AI ecosystem is still raw as developers work to leverage NPU acceleration in their products, but AMD, Microsoft, and others are working to build the foundations that will enable a broad swath of AI-accelerated features in software and Windows. AMD cites that it will have 150+ ISV (independent software vendor) partnerships to develop AI-driven solutions in 2024, in contrast to [Intel's broadening AI Developer Program](https://www.tomshardware.com/pc-components/cpus/intel-shares-new-ai-pc-definition-launches-ai-pc-acceleration-programs-and-core-ultra-meteor-lake-nuc-developer-kits-at-ai-conference), which has 100 ISV and 100+ IHV (integrated hardware vendor) partnerships thus far.

Naturally, unleashing the power of locally-run AI starts with the hardware. Let's take a closer look at the silicon.

## AMD Ryzen Pro 8040-series processors

As noted, the Ryzen Pro 8040-series processors are based on the consumer-oriented 'Hawk Point' processors — you can [learn the details of this Zen 4-powered lineup here](https://www.tomshardware.com/pc-components/cpus/the-refresh-that-wasnt-amd-announces-hawk-point-ryzen-8040-series-with-zen-4-rdna3-and-xdna-teases-strix-point). The U-series processors slot in for the 15-28W TDP range, while the HS variants are carved into 20-28W and 35-54W swimlanes spanning the Ryzen 9, 7, and 5 product lines, with the 45W models geared for workstations.

Core counts range from six cores and 12 threads with Ryzen 5 up to eight cores and 16 threads with Ryzen 7 and 9\. Peak clock rates remain the same as the consumer variants. Notably, the Ryzen 5 Pro 8540U comes without an integrated NPU, just like the consumer variant. All other key criteria, like clock speeds, iGPU, and cache, remain the same.

Get Tom's Hardware's best news and in-depth reviews, straight to your inbox.

AMD shared [benchmarks](https://www.tomshardware.com/tag/benchmark) comparing its silicon to Intel's competing chips, but as with all vendor-provided benchmarks, we should take them with a grain of salt. AMD claims an average 30% advantage with the Ryzen 7 Pro 8840U at 15W over the 15W Intel Core Ultra 7 165U in a range of workloads spanning multitasking, rendering, productivity, graphics, and content creation workloads. The company also claims an 18% average advantage over the 28W Core Ultra 7 165H.

AMD also says its 45W Ryzen 9 Pro 8945HS for workstations delivers a 50%+ advantage in the Topaz Labs AI-powered video application over the 45W Intel Core Ultra 9 185H and up to 77% faster time to the first token with the Llama 2 LLM. AMD also shared benchmarks with a wider range of AI models, like facial and object recognition, among others. Claimed performance advantages range from 5% to 23% across a spate of more general CPU-centric applications. 

The company also claims a 46% battery life advantage over the Intel Core Ultra 7 165H during a Teams workflow and a slight advantage over the Apple M3 Pro CPU. However, the Apple processor is missing from any of AMD's direct performance comparisons.  

## AMD Ryzen Pro 8000-series APUs

The Ryzen Pro 8000-series for desktop PCs is built upon the foundation of the powerful 8000G APU series for the consumer market, which you can read more about [here](https://www.tomshardware.com/pc-components/cpus/amd-launches-ryzen-8000g-phoenix-apus-brings-ai-to-the-desktop-pc-reveals-zen-4c-clocks-for-the-first-time) and [here](https://www.tomshardware.com/pc-components/cpus/amd-ryzen-7-8700g-cpu-review). All of these models have an in-built NPU AI acceleration engine.

Once again, all relevant specs remain identical to the consumer models, with options spanning from four cores and eight threads up to eight cores and 16 threads across the Ryzen Pro 3, 5, and 7 families. The standard 8000G models have a configurable 45-65W TDP range, whereas the GE models slot in for a lower-power 35W TDP.

Overall, AMD claims a 19% average uplift over Intel's competing Core i7-14700, with a peak of a 3X advantage in the Time Spy graphics benchmark. That isn't surprising, given the powerful combination of the Zen 4 CPU with the RDNA 3 graphics engine. AMD also claims from 33% to 76% less power consumption across a span of different configurations.   

## AMD Ryzen Pro Technologies

The integrated AMD Pro Technologies suite is the key difference between AMD's Pro models and its standard consumer chips. This hardware and software suite includes features like AMD Pro [Security](https://www.tomshardware.com/tag/security), which includes multiple layers of [security](https://www.tomshardware.com/tag/security) that leverage proprietary OEM solutions and in-built [Windows 11](https://www.tomshardware.com/tag/windows-11) features, AMD memory guard, [Microsoft](https://www.tomshardware.com/tag/microsoft) Pluton, and the AMD Secure processor. The 8000-series family marks the first desktop PCs to integrate Microsoft Pluton security functionality and cloud-based remote manageability. 

The AMD Pro Manageability features simplify provisioning, system imaging, and deployment tasks, while the AMD Pro Business Ready suite ensures stability and includes a quality and reliability guarantee. AMD points out that it offers the full suite of Pro Technologies on all of its processor models, whereas Intel bifurcates support for segmentation purposes.

AMD's OEM platforms have a range of refreshed products built on the underpinnings of the 8040- and 8000-series processors. The Ryzen Pro series is available to AMD's OEM partners now.