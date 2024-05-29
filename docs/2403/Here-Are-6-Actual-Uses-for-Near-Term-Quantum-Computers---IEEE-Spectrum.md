<!--yml
category: 未分类
date: 2024-05-29 12:32:39
-->

# Here Are 6 Actual Uses for Near-Term Quantum Computers - IEEE Spectrum

> 来源：[https://spectrum.ieee.org/what-are-quantum-computers-used-for](https://spectrum.ieee.org/what-are-quantum-computers-used-for)

**Although [recent finding](https://cacm.acm.org/research/disentangling-hype-from-practicality-on-realistically-achieving-quantum-advantage/#R5)[s](https://cacm.acm.org/research/disentangling-hype-from-practicality-on-realistically-achieving-quantum-advantage/#R5) have [poured cold water](https://spectrum.ieee.org/quantum-computing-skeptics) on [quantum computing](https://spectrum.ieee.org/tag/quantum-computing) hype, don’t count the technology out yet. On 4 March, [Google](https://spectrum.ieee.org/tag/google) and XPrize announced a [US $5 million prize](https://www.xprize.org/prizes/qc-apps) to anyone who comes up with use cases for quantum computers. If that sounds like an admission that use cases don’t already exist, it isn’t, says Ryan Babbush, head of quantum algorithms at Google. “We do know of some applications that these devices would be quite impactful for,” he says.**

****“A quantum computer is a special purpose accelerator,” says Matthias Troyer, corporate vice president of [Microsoft](https://spectrum.ieee.org/tag/microsoft) Quantum and member of the Xprize competition’s advisory board. “It can have a huge impact for special problems where quantum mechanics can help you solve them.”****

****The kinds of problems for which quantum computers could be useful hark back to their historical roots. In 1981, physicist [Richard Feynman](https://en.wikipedia.org/wiki/Richard_Feynman) proposed the [idea of a quantum computer](https://s2.smu.edu/~mitch/class/5395/papers/feynman-quantum-1981.pdf) as a means of simulating the full complexity of the quantum world.****

******“The commercial impact of solving quantum systems is in chemistry, material science, and pharma.” **—Matthias Troyer, Microsoft Quantum********

****Since then, scientists have come up with ingenious algorithms to make quantum computers useful for non-quantum things, such as [searching databases](https://learning.quantum.ibm.com/course/fundamentals-of-quantum-algorithms/grovers-algorithm) or [breaking cryptography](https://spectrum.ieee.org/encryptionbusting-quantum-computer-practices-factoring-in-scalable-fiveatom-experiment). However, the database search algorithms [don’t promise viable speedups](https://journals.aps.org/prxquantum/abstract/10.1103/PRXQuantum.2.010103) in the foreseeable future, and destroying Internet security seems like a dubious reason to build a new machine.**But a [recent study](https://arxiv.org/pdf/2401.16317.pdf) suggests that quantum computers will be able to simulate quantum phenomena of interest to several industries well before they can make headway in those other applications.******

******“The commercial impact of solving quantum systems is in chemistry, material science, and pharma,” Troyer says. And these are industries of significance, Troyer adds. “From the Stone Age to the Bronze Age, the Iron Age, the Steel Age, the Silicon Age, we define progress through materials progress.”******

******On that path to the possible new Quantum Age, here are a few examples with proven quantum advantage on machines that quantum computing researchers expect within the coming decade. And with any luck, Troyer hopes that the $5 million prize will incentivize the scientific community to find even more ways to put quantum algorithms to use in the real world. “The goal [of the prize] is that we want to have more quantum scientists get interested in not just developing quantum algorithms and the theory of them but also asking: Where can they be applied? How can we use quantum computers to tackle the world’s big problems?”******

## ******Drug Metabolism******

******In a 2022 paper [published in](https://www.pnas.org/doi/10.1073/pnas.2203533119) *[PNAS](https://www.pnas.org/doi/10.1073/pnas.2203533119)*, a collaboration between pharmaceutical company Boehringer Ingelheim, Columbia University, Google Quantum AI, and quantum simulation company QSimulate examined an enzyme called cytochrome P450\. This particular enzyme is responsible for metabolizing roughly 70 percent of the drugs that enter the human body.**The oxidation process by which the enzyme metabolizes drugs is inherently quantum, in a way that is difficult to simulate classically (classical methods work well when there are not a lot of quantum correlations at different scales).********

******They found that a quantum computer of a few million qubits would be able to simulate the process faster and more precisely than state-of-the-art classical techniques. “We find that the higher accuracy offered by a quantum computer is needed to correctly resolve the chemistry in this system, so not only will a quantum computer be better, it will be necessary,” the researchers (including Babbush) wrote in a [blog post](https://blog.research.google/2023/10/developing-industrial-use-cases-for.html).******

## ******CO[2] Sequestration******

******One strategy to lower the amount of carbon dioxide in the atmosphere is [sequestration](https://www.nationalgrid.com/stories/energy-explained/what-carbon-sequestration#:~:text=Carbon%20sequestration%20is%20the%20capturing,carbon%20from%20the%20earth's%20atmosphere.)—using a catalyst to react with the carbon dioxide and form a compound that can be stored for a long time. Sequestration strategies exist, but are not cost or energy efficient enough to make a significant dent in the current carbon emissions.******

******[Several](https://pubs.aip.org/avs/aqs/article/5/1/013801/2879052/Description-of-reaction-and-vibrational-energetics) [recent](https://link.springer.com/content/pdf/10.1140/epjqt/s40507-022-00155-w.pdf?pdf=button) [studies](https://journals.aps.org/prresearch/abstract/10.1103/PhysRevResearch.3.033055) have suggested that quantum computers of the near future should be able to model carbon dioxide reactions with various catalysts more accurately than classical computers. If true, this would allow scientists to more effectively estimate the efficiency of various sequestration candidates.******

## ******Agricultural Fertilization******

******Most farmland today is fertilized with ammonia produced under high temperature and pressure in large plants via the [Haber-Bosch process](https://en.wikipedia.org/wiki/Haber_process). In 2017, a team at Microsoft Research and ETH Zurich [considered](https://www.pnas.org/doi/abs/10.1073/pnas.1619152114) an alternative ammonia production method—nitrogen fixation by way of the enzyme nitrogenase—that would work at ambient temperature and pressure.******

******This reaction cannot be accurately simulated by classical methods, the researchers showed, but it is within the reach of a classical and quantum computer working in tandem. “If, for example, you could find a chemical process for nitrogen fixation is a small scale in a village on a farm, that would have a huge impact on the food security,” says Troyer, who was involved in the research.******

## ******Alternate Battery Cathodes******

******Many lithium-ion batteries [use cobalt](https://www.aquametals.com/recyclopedia/lithium-ion-anode-and-cathode-materials/#:~:text=Cathode%20active%20materials%20(CAM)%20are,oxide%20(LiNiMnCoO2%20or%20NMC).) in their cathodes. Cobalt mining has some practical drawbacks, including [environmental concerns](https://spectrum.ieee.org/seafloor-cobalt) and [unsafe labor practices](https://www.npr.org/sections/goatsandsoda/2023/02/01/1152893248/red-cobalt-congo-drc-mining-siddharth-kara). One alternative to cobalt is nickel. In a [study](https://journals.aps.org/prxquantum/abstract/10.1103/PRXQuantum.4.040303) published in 2023, a collaboration between chemical producer BASF, Google Quantum AI, Macquarie University in Sydney, and QSimulate considered what it would take to simulate a nickel-based cathode, lithium nickel oxide, on a quantum computer.******

******Pure lithium nickel oxide, the researchers said, is unstable in production, and its basic structure is poorly understood. Having a better simulation of the material’s ground state may suggest methods for making a stable version. The quantum computing requirements to adequately simulate this problem are “out of reach of the first error-corrected quantum computers,” the authors wrote in a [blog post](https://blog.research.google/2023/10/developing-industrial-use-cases-for.html), “but we expect this number to come down with future algorithmic improvements.”******

## ******Fusion Reactions******

******In 2022, the National Ignition Facility [made headlines](https://spectrum.ieee.org/national-ignition-facility-impractical) with the first inertial fusion reaction to produce more energy than was put directly into it. In an inertial fusion reaction, a tritium-deuterium mixture is heated with lasers until it forms a plasma that collapses into itself, initiating the fusion reaction. This plasma is extremely difficult to simulate, says Babbush, who was involved with the study. “The Department of Energy is already spending hundreds of millions of CPU hours if not billions of CPU hours, just computing one quantity,” he says.******

******In a [preprint](https://arxiv.org/abs/2308.12352v1), Babbush and his collaborators outlined an algorithm that a quantum computer could use to model the reaction in its full complexity. This, like the battery cathode research, would require more qubits than are currently available, but the authors believe future hardware and algorithmic improvements may close this gap.******

## ******Improving Quantum Sensors******

******Unlike quantum computers, [quantum sensors](https://spectrum.ieee.org/quantum-sensors) are already having an impact in the real world. These sensors can measure magnetic fields more precisely than any other technology, and are being used for brain scans, gravity measurements for mapping geological activity, and more. The output of the quantum sensor is quantum data, but it’s currently read out as classical data, traditional ones and zeros that miss some of the full quantum complexity.******

******A [2022 study](https://www.science.org/doi/10.1126/science.abn7293) from a collaboration between Google, Caltech, Harvard, UC Berkeley, and Microsoft has shown that if the output of a quantum sensor is instead funneled into a quantum computer, they can use a clever algorithm to learn relevant properties with exponentially fewer copies of the data from the sensor, speeding up readout. They demonstrated their quantum algorithm on a simulated sensor, showing that this algorithms is within reach for even currently existing quantum computers.******

## ******And More******

******There are also advantageous quantum algorithms still in search of definitive use cases, and prize money is being offered to also motivate that search. Among those algorithms are [solving certain types of linear differential equations](https://journals.aps.org/prx/abstract/10.1103/PhysRevX.13.041041), and [finding patterns in data](https://www.nature.com/articles/s41567-021-01287-z) that are not accessible classically. In addition, classically, many algorithms can’t be proven to work efficiently with pencil and paper, says Jay Gambetta, vice president at IBM Quantum. Instead, people try heuristic algorithms out on real hardware, and some of them perform surprisingly well. With quantum computers, Gambetta argues, the hardware state of the art is on the cusp of being good enough to test out many more heuristic algorithms, so many more use cases should be forthcoming.******

******“I think we can finally start to do algorithm discovery using hardware,” Gambetta says. “And to me that’s opening a different avenue for accelerated scientific discovery. And I think that’s what’s most exciting.”******

******From Your Site Articles******

******Related Articles Around the Web******