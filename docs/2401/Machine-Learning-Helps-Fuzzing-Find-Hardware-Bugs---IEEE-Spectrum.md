<!--yml
category: 未分类
date: 2024-05-27 14:37:04
-->

# Machine Learning Helps Fuzzing Find Hardware Bugs - IEEE Spectrum

> 来源：[https://spectrum.ieee.org/hardware-hacking](https://spectrum.ieee.org/hardware-hacking)

In the age of the global [chip-supply shortage](https://spectrum.ieee.org/chip-shortage-rewiring-tech), any speedup in chip manufacturing and quality-assurance testing is a potential lifeline. So a technique first developed to [find instabilities in UNIX command-line prompts in the 1980s](https://en.wikipedia.org/wiki/Fuzzing#History) is now being retooled to automate chip tests on the assembly line—and discover bugs that could ultimately lead to hardware vulnerabilities like the sort that led to the [Meltdown and Spectre flaws](https://spectrum.ieee.org/how-the-spectre-and-meltdown-hacks-really-worked) and waves of hacks that sprung from them.

It’s difficult to patch hardware bugs, so catching them early in the product development cycle is important, says Texas A&M University engineering associate professor Jeyavijayan Rajendran. A coauthor [on the new study](https://arxiv.org/abs/2311.14594), Rajendran likened chip manufacturers to the car industry, which has to issue recalls to fix security issues. Ideally, flaws are found before vehicles are rolled out to consumers in the first place.

“When people design hardware, they do not think about security up front.... And because of this, a lot of hardware vulnerabilities creep into the system.”
**—Jeyavijayan Rajendran, Texas A&M**

Rajendran’s work—to be presented at the Design, Automation and Test in Europe (DATE) conference—relies on a technique called “fuzzing,” which, in this case, introduces commands and prompts to a chip that are not quite correct. They aren’t complete nonsense, but they contain enough correct syntax to make the system behave erratically and unpredictably. Studying those erratic responses to “fuzzed” commands can then point researchers—or hackers—to potential weak links in the system.

This is why fuzzing is increasingly popular for hardware testing. It uncovers flaws by running the hardware with edge cases and unexpected inputs—like random data and machine instructions—to stress the system and see if something breaks. If the system does something unexpected, researchers zero in to determine if there is a security flaw that hackers could take advantage of.

“We do a comparison between the expectation of what the processor should do and the reality of what the processor is actually doing,” says Rajendran.

These tests save time because they can be automated and executed multiple times during a product-development cycle, and performed in parallel with other engineering work. But researchers are still looking for ways to make hardware fuzzing techniques faster and more efficient. Currently, fuzzing algorithms employ a rigid strategy for selecting new random inputs. This rigidness slows down the process of discovering vulnerabilities because it doesn’t take advantage of promising leads.

In this study, researchers used [reinforcement learning](https://spectrum.ieee.org/tag/reinforcement-learning) to select inputs for fuzz testing. They adapted an algorithm used to solve the multi-armed bandit (MAB) problem—the dilemma of how to optimize rewards when faced with the choice of accepting known rewards or exploring rewards that may be greater or lower. In this case, the algorithm—called MABFuzz—is used to decide whether to try a new random input or stick with one that works well. Researchers found that MABFuzz achieved significant speedup in detecting vulnerabilities and covering the testing space.

Hardware vulnerabilities have attracted more attention recently because processors are increasingly complex and designed to optimize for performance, said Rajendran. That presents more places for security flaws to hide.

“When people design hardware, they do not think about security up front,” he says. “They think about things like power, performance—but security is not their first design metric. And because of this, a lot of hardware vulnerabilities creep into the system.”

Traditional hardware testing strategies mostly consist of manual testing by hardware security experts, but that strategy can’t scale up to meet the needs of modern processors. Manual testing is time consuming and expensive, and limited by the availability of security experts.

Automated hardware-testing techniques like fuzz testing aren’t meant to replace manual testing by experts, Rajendran says. Instead, it’s a first line of defense that can uncover a large number of relatively easy-to-find vulnerabilities easily, he said. That frees up security experts’ time to uncover the really tricky bugs that still require expertise to find.

Ahmad-Reza Sadeghi, professor of computer science at the Technical University of Darmstadt, who is a coauthor on the study, says improved security testing for hardware components will be important for the future of chip engineering. Strategies that can make the process of quickly uncovering vulnerabilities easier are needed for a healthy chip industry, just as supply chains and manufacturing capabilities are.

From Your Site Articles

Related Articles Around the Web