<!--yml
category: 未分类
date: 2024-05-27 14:50:06
-->

# Ushering in the Thermodynamic Future - Litepaper

> 来源：[https://www.extropic.ai/future](https://www.extropic.ai/future)

Mar 11th, 2024

/

Litepaper

# Ushering in the Thermodynamic Future

Message from the team

*We are very excited to finally share more about what Extropic is building: a full-stack hardware platform to harness matter's natural fluctuations as a computational resource for Generative AI.*

What does this novel paradigm of computing practically mean for the world?

*   *Extends hardware scaling well beyond the constraints of digital computing*
*   *Enables AI accelerators that are many orders of magnitude faster and more energy efficient than digital processors (CPUs/GPUs/TPUs/FPGAs)*
*   *Unlocks powerful probabilistic AI algorithms that are not feasible on digital processors*

*Our brief Litepaper (below) provides an early glimpse at our technologies. We hope the following excites you for the journey ahead. Join us as we accelerate towards the thermodynamically intelligent future.*

**- Gill and Trev**

The demand for computing power in the AI era is increasing at an unprecedented exponential rate. Luckily, for the past several decades, the miniaturization of CMOS transistor technology following Moore’s law [[1]](#references) has allowed much of this exponential growth to be accounted for by increasing computer efficiency.

Unfortunately, Moore’s law is starting to slow down [[2]](#references). The reason for this is rooted in fundamental physics: transistors are approaching the atomic scale where effects like thermal noise start to forbid rigid digital operation [[3, 4, 5]](#references).

As a result, the energy requirements of modern AI are beginning to take off. Major players are proposing measures as extreme as building nuclear reactor-powered data centers dedicated to large model training and inference. Continuing this scaling for a few more decades will require infrastructure engineering efforts of unprecedented scale and represents an arduous path forward for scaling humanity’s aggregate intelligence.

On the other hand, biology is neither rigid nor digital and hosts computing circuitry that is much more efficient than anything humanity has built to date. Inter-cellular chemical reaction networks drive computation in biological systems. Cells are small, and as a result, the number of reactants in these networks is countable [[6, 7]](#references). Therefore, reactions between reactants are genuinely discrete and intrinsically random. The relative effect of this intrinsic randomness scales inversely with the number of reactant molecules, and as such, fluctuations tend to dominate the dynamics of these systems.

From this, we can say with certainty that there is no fundamental reason for the constraints of digital logic to bind the efficiency of computing devices. The engineering challenge is clear: how can we design a complete AI hardware and software system from the ground up that thrives in an intrinsically noisy environment?

Energy-Based Models (EBMs) offer hints at a potential solution, as they are a concept that appears both in thermodynamic physics and in fundamental probabilistic machine learning. In physics, they are known as parameterized thermal states, arising from steady-states of systems with tunable parameters. In machine learning, they are known as exponential families.

Exponential families are known to be the optimal way to parameterize probability distributions, requiring the minimal amount of data to uniquely determine their parameters [[8]](#references). They are thus excellent in the low-data regime, which encompasses scenarios where one needs to model tail events in mission-critical applications, as depicted in figure 1\. The way they achieve this is by filling the blanks in data with noise; they seek to maximize their entropy while matching the statistics of the target distribution. This process of hallucinating every possibility that is not included in a dataset and penalizing such occurrences energetically requires the usage of a lot of randomness, both at training and inference time.

Figure 1: The principles of Extropic probabilistic AI accelerators A simple example of low-complexity distributional learning failing to capture the effect of a tail event. Low air pressure almost always means rain and high crop yields. However, every so often, very low air pressure corresponds to a hurricane.

This requirement of sampling has been the main limiter in production uses of EBMs. The fundamental reason for this is that sampling from generic energy landscapes is very difficult on digital hardware, which must expend substantial electrical energy to generate and shape the entropy required for the diffusion processes. From a hardware perspective, digital sampling seems quite contrived: why put so much effort into building increasingly complex pristine digital computers when the most common and compute-intensive algorithms turn around and pump them full of noise?

Extropic is shortcutting this inefficiency and unlocking the full potential of generative AI by implementing EBMs directly as parameterized stochastic analog circuits. Extropic accelerators will achieve many orders of magnitude of improvement over digital computers in terms of both runtime and energy efficiency for algorithms based on sampling from complex landscapes.

The operational principle of Extropic accelerators is analogous to Brownian motion. In Brownian motion, macroscopic but lightweight particles suspended in a fluid experience random forces due to many collisions with microscopic liquid molecules. These collisions lead to the random diffusion of the particles around the vessel. One could imagine anchoring the Brownian particles to the vessel walls and each other with springs, as depicted in Fig. 2 (a). In this case, the springs will resist the random forces, and the particles will prefer to reside in particular parts of the vessel more than others. If one repeatedly sampled the positions of the particles, waiting sufficiently long in between samples (as illustrated in Fig. 2 (b)), one would find that they follow a predictable *steady-state* probability distribution. If we changed the stiffness of the springs, this distribution would change. This simple mechanical system is a source of programmable randomness.

Figure 2: The operational principle of Extropic accelerators  **(a)** A simple mechanical analogy to Extropic accelerators. Since there are three masses in two dimensions, the steady state of this device would be a probability distribution over a 6-dimensional space. (b) Samples may be drawn from an Extropic accelerator by repeatedly observing the system, waiting at least the equilibration time teq between observations. teq is how long it takes for the noise in the system to destroy all correlations with the previous sample.

There is a direct connection between this mechanical picture and the parameterized stochastic analog circuits that make up Extropic accelerators. The lightweight particles represent electrons, and the liquid molecules represent the atoms of the conducting medium, which can transfer energy to the electrons upon collision. The springs represent circuit components that confine the motion of the electrons, such as inductors or transistors. Control voltages/currents can be applied to tune the values of these components, changing the distribution from which the circuit samples.

Although every circuit is noisy, not every circuit is useful as an Extropic accelerator. Making a noise-dominated yet well-behaved device is challenging from an engineering perspective. Thermal fluctuations are small, so devices must be physically small and low power to be strongly affected by them. For this reason, if one wanted to make an Extropic accelerator out of macroscopic components (on a PCB, for example), one would have to introduce synthetic noise. Doing so erodes the fundamental time and energy savings and ends up performing similarly to running the algorithm digitally.

Extropic’s first processors are nano fabricated from aluminum and run at low temperatures where they are superconducting. Fig. 3 shows an early device that tested several possible superconducting neuron designs. Some of these neurons are similar to existing super conducting flux qubits [[9]](#references). These neurons exploit the Josephson effect as a source of nonlinearity, which occurs when two superconductors are near one another. This nonlinearity is required for the device to access non-Gaussian probability distributions, which are necessary to model real-world applications with fat tails. Additionally, digital Gaussian sampling routines are ubiquitous and highly optimized. So, if an analog device is to provide a considerable speedup compared to a traditional processor, non-Gaussianity is required.

Figure 3: Microscope image of an Extropic chip. The inset shows two Josephson junctions, which are the devices that provide the processor with its critical nonlinearity.

These neurons provide the basic building blocks that are combined to form a larger superconducting system. In such a larger system, many linear and non-linear neurons are combined together to create a circuit that samples from a rich and high-dimensional distribution. The neuron biases and interaction strengths are all tunable parameters of the distribution, allowing a single device to embody a wide family of probability distributions.

Extropic’s superconducting chips are entirely passive, meaning we only expend energy when measuring or manipulating its state. This likely makes these neurons the most energy-efficient in the universe. These systems will be highly energy efficient at scale: Extropic targets low-volume, high-value customers like governments, banks, and private clouds with these systems.

Extropic is also building semiconductor devices that operate at room temperature to extend our reach to a larger market. These devices trade the Josephson junction for the transistor. Doing so sacrifices some energy efficiency compared to superconducting devices. In exchange, it allows one to build them using standard manufacturing processes and supply chains, unlocking massive scale. Since they operate at room temperature, it will be possible to pack them into a GPU-like expansion card form factor. This will allow us to put an Extropic accelerator in every home, enabling everyone to partake in the thermodynamic AI acceleration.

To support a wide range of hardware substrates, Extropic is also building a software layer that compiles from abstract specifications of EBMs to the relevant hardware control language. This compilation layer is built upon the theoretical framework of factor graphs [[8]](#references). Factor graphs specify how large distributions factorize into local chunks. This allows Extropic accelerators to breakdown and run programs that are too big to fit on any given analog core.

Many previous AI accelerator companies have struggled to find an advantage because of the memory-boundedness of deep learning; today’s algorithms spend around 25% of their time moving numbers around in memory. As a result of this, via Amdahl’s law, any chip that accelerates a particular operation (such as matrix multiplication) will struggle to achieve more than a 4x speedup. As Extropic chips natively accelerate a broad class of probabilistic algorithms by running them physically as a rapid and energy-efficient process in their entirety, we are bound to unlock a whole new regime of artificial intelligence acceleration well beyond what was previously thought achievable.