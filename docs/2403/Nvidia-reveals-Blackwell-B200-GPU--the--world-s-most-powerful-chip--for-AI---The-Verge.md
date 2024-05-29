<!--yml
category: 未分类
date: 2024-05-27 15:04:12
-->

# Nvidia reveals Blackwell B200 GPU, the ‘world’s most powerful chip’ for AI - The Verge

> 来源：[https://www.theverge.com/2024/3/18/24105157/nvidia-blackwell-gpu-b200-ai](https://www.theverge.com/2024/3/18/24105157/nvidia-blackwell-gpu-b200-ai)

Nvidia’s must-have H100 AI chip made it [a multitrillion-dollar company](/2024/2/23/24080975/nvidia-ai-chips-h100-h200-market-capitalization), one that may be worth [more than Alphabet and Amazon](/2024/2/14/24073384/nvidia-market-cap-passes-amazon-alphabet), and competitors have been [fighting to catch up](/2024/2/1/24058186/ai-chips-meta-microsoft-google-nvidia). But perhaps Nvidia is about to extend its lead — with the new Blackwell B200 GPU and GB200 “superchip.”

*Nvidia CEO Jensen Huang holds up his new GPU on the left, next to an H100 on the right, from the GTC livestream.*

Image: Nvidia

Nvidia says the new B200 GPU offers up to 20 *petaflops* of FP4 horsepower from its 208 billion transistors. Also, it says, a GB200 that combines two of those GPUs with a single Grace CPU can offer 30 times the performance for LLM inference workloads while also potentially being substantially more efficient. It “reduces cost and energy consumption by up to 25x” over an H100, says Nvidia, though there’s a questionmark around cost — [Nvidia’s CEO has suggested](/2024/3/19/24106141/nvidia-ceo-says-his-blackwell-b200-ai-gpu-will-cost-30k-40k-a-pop) each GPU might cost between $30,000 and $40,000\.

Training a 1.8 trillion parameter model would have previously taken 8,000 Hopper GPUs and 15 megawatts of power, Nvidia claims. Today, Nvidia’s CEO says 2,000 Blackwell GPUs can do it while consuming just four megawatts.

On a GPT-3 LLM benchmark with 175 billion parameters, Nvidia says the GB200 has a somewhat more modest seven times the performance of an H100, and Nvidia says it offers four times the training speed.

*Here’s what one GB200 looks like. Two GPUs, one CPU, one board.*

Image: Nvidia

Nvidia told journalists one of the key improvements is a second-gen transformer engine that doubles the compute, bandwidth, and model size by using four bits for each neuron instead of eight (thus, the 20 petaflops of FP4 I mentioned earlier). A second key difference only comes when you link up huge numbers of these GPUs: a next-gen NVLink switch that lets 576 GPUs talk to each other, with 1.8 terabytes per second of bidirectional bandwidth.

That required Nvidia to build an entire new network switch chip, one with 50 billion transistors and some of its own onboard compute: 3.6 teraflops of FP8, says Nvidia.

*Nvidia says it’s adding both FP4 and FP6 with Blackwell.*

Image: Nvidia

Previously, Nvidia says, a cluster of just 16 GPUs would spend 60 percent of their time communicating with one another and only 40 percent actually computing.

Nvidia is counting on companies to buy large quantities of these GPUs, of course, and is packaging them in larger designs, like the GB200 NVL72, which plugs 36 CPUs and 72 GPUs into a single liquid-cooled rack for a total of 720 petaflops of AI training performance or 1,440 petaflops (aka 1.4 *exaflops*) of inference. It has nearly two miles of cables inside, with 5,000 individual cables.

*The GB200 NVL72\.*

Image: Nvidia

Each tray in the rack contains either two GB200 chips or two NVLink switches, with 18 of the former and nine of the latter per rack. In total, Nvidia says one of these racks can support a 27-trillion parameter model. GPT-4 is rumored to be around a 1.7-trillion parameter model.

The company says Amazon, Google, Microsoft, and Oracle are all already planning to offer the NVL72 racks in their cloud service offerings, though it’s not clear how many they’re buying.

And of course, Nvidia is happy to offer companies the rest of the solution, too. Here’s the DGX Superpod for DGX GB200, which combines eight systems in one for a total of 288 CPUs, 576 GPUs, 240TB of memory, and 11.5 exaflops of FP4 computing.

Nvidia says its systems can scale to tens of thousands of the GB200 superchips, connected together with 800Gbps networking with its new Quantum-X800 InfiniBand (for up to 144 connections) or Spectrum-X800 ethernet (for up to 64 connections).

We don’t expect to hear anything about new gaming GPUs today, as this news is coming out of Nvidia’s GPU Technology Conference, which is usually almost entirely focused on GPU computing and AI, not gaming. But the Blackwell GPU architecture will [likely also power a future RTX 50-series lineup](https://videocardz.com/newz/nvidia-blackwell-gb203-gpu-to-feature-256-bit-bus-gb205-with-192-bit-claims-leaker) of desktop graphics cards.

***Update, March 19th:** Added Nvidia CEO estimate that the new GPUs might cost up to $40K each.*