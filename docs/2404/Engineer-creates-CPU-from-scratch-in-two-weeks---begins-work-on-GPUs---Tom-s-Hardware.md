<!--yml
category: 未分类
date: 2024-05-27 13:12:57
-->

# Engineer creates CPU from scratch in two weeks — begins work on GPUs | Tom's Hardware

> 来源：[https://www.tomshardware.com/pc-components/cpus/engineer-creates-cpu-from-scratch-in-two-weeks-begins-work-on-gpus](https://www.tomshardware.com/pc-components/cpus/engineer-creates-cpu-from-scratch-in-two-weeks-begins-work-on-gpus)

An engineer has shared his experience of designing a CPU from scratch, over two weeks, “with no prior experience.” During this brief period, [Adam Majmudar](https://twitter.com/MajmudarAdam/status/1778235769150423121) claims to have learned the fundamentals of [chip architecture](https://www.tomshardware.com/tech-industry/artificial-intelligence/jim-keller-responds-to-sam-altmans-plan-to-raise-dollar7-billion-to-make-ai-chips), absorbed the finer points of [chip fabrication](https://www.tomshardware.com/news/new-us-fabs-everything-we-know), and prepared his first full chip layout using [EDA tools](https://www.tomshardware.com/news/ai-tools-take-chip-design-industry-by-storm-200-chips-tape-out). The next step in his “speed-running the chip stack” to-do list is designing a GPU from scratch. When finished this project is destined for production via [Matthew Venn’s](https://twitter.com/matthewvenn) TinyTapeout 6.

We’ve reported on enthusiast [DIY CPU designs](https://www.tomshardware.com/news/man-builds-own-silicon-chip-at-home) previously, as well as [DIY GPU projects](https://www.tomshardware.com/pc-components/gpus/new-open-source-gpu-is-free-to-all-supports-modern-windows-software-stack-runs-on-an-fpga-with-custom-pcb). However, some of those feats have eaten through years of spare time for the people involved. Majmudar must be on vacation and spending all his surplus time on this “speed run” project to have gotten as far as he has “from scratch.”

The fledgling chip designer, who describes himself as one of the founding engineers at a web3 development company, outlines the steps he has made so far in his quest. You can click and read through all the steps leading up to the current GPU focus via the embedded Tweet, above. We’ve also bullet-pointed the speed run steps which have been completed to date, below.

*   Learning the fundamentals of chip architecture – a strong understanding is a critical foundation
*   Learning the fundamentals of chip fabrication – materials, wafer prep, patterning and packaging
*   Starting electronic design automation by making a CMOS transistor, layer-by-layer
*   Creating my first full circuit in Verilog – “my first experience with programming hardware using software.”
*   Implementing simulation & formal verification for my circuit
*   Designing my first full chip layout – designing and optimizing using OpenLane, which is an open-source EDA tool

Image 1 of 5

(Image credit: Adam Majmudar )

(Image credit: Adam Majmudar )

(Image credit: Adam Majmudar )

(Image credit: Adam Majmudar )

(Image credit: Adam Majmudar )

As we mentioned in the intro, the significant step that Majmudar now faces is designing a GPU from scratch. He knows this will be a difficult task and admits that, after initial investigations, it is [harder than expected](https://twitter.com/MajmudarAdam/status/1778235788880420870). The fledgling chip designer explains that there simply aren’t the learning resources online for building a GPU. “Because GPU companies are all trying to keep their secrets from each other, most of the GPU architecture data is all proprietary and closed source,” the engineer finds.

Despite this hurdle, Majmudar says the big GPU-makers’ secrecy has made this part of the project “way more fun for me.” Interestingly, Anthropic’s Claude Opus AI tools have been useful during this GPU designing stage. “I've been proposing my ideas for how each unit must work to Claude, and then somehow it will guide me toward the right implementation approaches which I can then go and confirm with open-source repos,” explained the engineer. However, he observed that “if I search some of the things publicly, nothing shows up which is a testament to how well hidden the implementation details are.”

After taking just two weeks or so to get through three out of five legs of his speed run, the above concerns expressed regarding GPUs might make readers worry that Majmudar may have hit a speed bump, a snag, or even a brick wall. That doesn’t seem to be the case, as he optimistically predicts that his GPU design will be shipping “in the next few days,” and a cut-down version sent to be taped out.

It may be well worth keeping an eye open for this engineer’s next updates. However, we know it can take quite some time between submitting work to projects like TinyTapeout and the production run. The maker of the [Rickroll ASIC](https://www.tomshardware.com/maker-stem/rickroll-asic-heralded-as-a-world-first-this-chip-is-never-gonna-let-you-down), for example, said there were nine months between submitting his design and receiving the silicon. Please note that [TT06](https://tinytapeout.com/runs/tt02/145/) closes just eight days from now.

Get Tom's Hardware's best news and in-depth reviews, straight to your inbox.