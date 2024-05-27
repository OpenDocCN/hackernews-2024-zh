<!--yml
category: 未分类
date: 2024-05-27 13:18:57
-->

# Causality, Interaction, and Complexity

> 来源：[https://www.alexahn.com/2024/04/causality-interaction-and-complexity.html](https://www.alexahn.com/2024/04/causality-interaction-and-complexity.html)

In a highly chaotic system, such as a high temperature gas, it is not ideal to use an atomic causal model. Instead, the effective causal model is to approximate to what extent each atom is interacting with every other atom. If we increase the temperature, then the number of atoms each atom interacts with should increase. As the temperature decreases, the number of atoms each atom interacts with should decrease.

If we were to randomly sample any atom, then on average, the atom should interact with a set of atoms of a certain size. Instead of thinking in terms of conditional probabilities and causal implications, we think in terms of sets of interconnected events. And this is because it is not computationally effective to analyze chaotic systems in a linear manner.

We can apply the same line of reasoning to sampling. If a system has a particular sampling rate, the inputs to the system are batched according to the sampling rate. In other words, the system cannot discern the ordering of events within a single sample, only between samples. The coupling of events (according to a sampling rate) is not done for computational efficiency, but it is a quality of the system.

It is strange that the predominant way to reason about causality is through atomic implications, such as p implies q. If we lower the temperature of a chaotic system such that each atom only interacts with one other element, have we not achieved the same thing, just without identity? Or if we increase the sampling rate of a system such that every event can be represented as an atomic sample, have we not achieved the same thing?

It seems causality is a consequence of two systems having different levels of entropy, or two systems having different sampling rates, because it is the difference in entropy or sampling rate that allows a pattern to be captured. If two systems have the same level of entropy, then neither can introspect the other because they look the same to each other. Likewise, if two systems have the same sampling rate, neither can decode the ordering of events within a sample. But if two systems have two different levels of entropy, then the one that has less entropy will look more ordered to the one that has more. Likewise, if two systems have two different sampling rates, then the one that has a higher sampling rate will look more ordered to the one that has a lower sampling rate. And a causal model of the events that are either interactively connected (due to entropy) or batched (due to sampling) can be constructed due to the difference.