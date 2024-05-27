<!--yml
category: 未分类
date: 2024-05-27 13:06:07
-->

# AI Coding Is Going From Copilot to Autopilot - IEEE Spectrum

> 来源：[https://spectrum.ieee.org/ai-code-generator](https://spectrum.ieee.org/ai-code-generator)

A new breed of [AI-powered coding](https://spectrum.ieee.org/ai-programming) tools have emerged—and they’re claiming to be more autonomous versions of earlier assistants like [GitHub Copilot](https://github.com/features/copilot), [Amazon CodeWhisperer](https://aws.amazon.com/codewhisperer/), and [Tabnine](https://www.tabnine.com/).

One such new entrant, [Devin AI](https://devinai.ai/), has been dubbed an “AI software engineer” by its maker, applied AI lab [Cognition](https://www.cognition-labs.com/). According to Cognition, Devin can [perform all these tasks unassisted](https://www.cognition-labs.com/introducing-devin): build a website from scratch and deploy it, find and fix bugs in codebases, and even train and fine-tune its own [large language model](https://spectrum.ieee.org/tag/llms).

Following its launch, open-source alternatives to Devin have cropped up, including [Devika](https://github.com/stitionai/devika) and [OpenDevin](https://github.com/OpenDevin/OpenDevin). Meanwhile makers of established assistants have not been standing still. Researchers at [Microsoft](https://spectrum.ieee.org/tag/microsoft), GitHub Copilot’s developer, recently uploaded a [paper to the arXiv preprint server](https://arxiv.org/abs/2403.08299) introducing AutoDev, which uses autonomous AI agents to generate code and test cases, run tests and check the results, and fix bugs within the test cases.

“It’s exciting to see more versions of AI coding assistants with new capabilities,” says [Ben Dechrai](https://bendechrai.com/about), a coder and developer advocate at software company [Sonar](https://www.sonarsource.com/). “They validate the need for [generative AI](https://spectrum.ieee.org/what-is-generative-ai) tools in developers’ workflows.”

Dechrai adds that these coding copilots can help software engineers write code faster, allowing them to focus on more strategic and creative tasks. Another advantage of these [programming](https://spectrum.ieee.org/tag/programming) tools is the ability to create a template for code, notes [Saurabh Bagchi](https://www.cs.purdue.edu/people/faculty/sbagchi.html), a professor of electrical and computer engineering at [Purdue University](https://www.purdue.edu/). Much as with [prompt engineering](https://spectrum.ieee.org/prompt-engineering-is-dead), developers must provide these assistants with “the right kind of software requirements to produce a template, and then a software engineer can fill in the gaps,” he says.

“To develop intuitive systems, you need an iterative process with humans in the loop to provide feedback” **—Saurabh Bagchi, Purdue University**

These gaps include safety and reliability considerations. Software engineers must look out for security vulnerabilities in AI-generated code, as well as the types of corner cases that could cause it to crash.

“Developers still need to ensure rigorous quality standards are in place when analyzing and reviewing code written with generative AI, just as they would with code developed by a human,” says Dechrai. “AI coding assistants are good at suggesting code, reflecting on the code, and reasoning about its effectiveness, but even then it’s not 100 percent accurate.”

Dechrai cautions that autonomous coders are “still so new that developers are just learning which use cases will be most beneficial.” And they’ll need to be “ironed out in the real world to see how much they’re able to deliver on their promise,” says Bagchi.

## AI Coders vs. the Humans

Doom-and-gloom predictions of replacing human software engineers are also bound to follow the emergence of these “AI software engineers,” but that won’t be happening anytime soon. Devin, for instance, [resolved only 14 percent of a subset of GitHub issues](https://www.cognition-labs.com/post/swe-bench-technical-report) from real-world code repositories. “There’s still a long way to go for it to become something I can rely on blindfolded,” says Bagchi.

He notes that these autonomous programming tools have another blind spot: the fact that [software development](https://spectrum.ieee.org/tag/software-development) happens in collaboration. Coding copilots try to do everything, and they might do it reasonably well. On the other hand, different software engineers have their own specialties—be it front end, back end, full stack, or data, to name a few—and they all work together to build a cohesive product.

“To develop intuitive systems, you need an iterative process with humans in the loop to provide feedback,” Bagchi says. “The fundamental human intuition, depth, and imagination has to be brought to bear.”

That’s why Bagchi believes these unassisted versions won’t be dominating the space that coding assistants hold—at least for now. “The models running underneath are similar in architecture, and as technology continues to evolve, both of them will get better,” he says. “But the Copilot or CodeWhisperer model seems most promising and is better suited to complex software development where humans work with the assistance of AI.”

Yet programmers “should start using these tools if they haven’t already, or they’ll risk getting left behind,” says Dechrai. “If you want to know if an AI coding assistant is truly beneficial, you have to use it yourself, get to know it, and see where it fails.”

Bagchi echoes the sentiment: “Try them out with the use cases you have and stress them with the kinds of software you’re creating.” But because unassisted coding copilots are a nascent technology, they are likely to improve rapidly. “So you have to track them,” he adds.

Moreover, software engineers will have to “consistently ensure code is secure, reliable, and maintainable throughout its life cycle,” Dechrai says. “It will always be up to the developer to properly understand the output and how it was generated.”