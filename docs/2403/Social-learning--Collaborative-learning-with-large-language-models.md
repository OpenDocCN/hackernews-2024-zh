<!--yml
category: 未分类
date: 2024-05-27 14:41:22
-->

# Social learning: Collaborative learning with large language models

> 来源：[https://blog.research.google/2024/03/social-learning-collaborative-learning.html?m=1](https://blog.research.google/2024/03/social-learning-collaborative-learning.html?m=1)

Large language models (LLMs) have significantly improved the state of the art for solving tasks specified using natural language, often reaching performance close to that of people. As these models increasingly enable assistive agents, it could be beneficial for them to learn effectively from each other, much like people do in social settings, which would allow LLM-based agents to improve each other’s performance.

To discuss the learning processes of humans, Bandura and Walters [described](https://books.google.ch/books/about/Social_Learning_Theory.html?id=IXvuAAAAMAAJ&redir_esc=y) the concept of *social learning* in 1977, outlining different models of observational learning used by people. One common method of learning from others is through a *verbal instruction* (e.g., from a teacher) that describes how to engage in a particular behavior. Alternatively, learning can happen through a *live model* by mimicking a live example of the behavior.

Given the success of LLMs mimicking human communication, in our paper “[Social Learning: Towards Collaborative Learning with Large Language Models](https://arxiv.org/pdf/2312.11441.pdf)”, we investigate whether LLMs are able to learn from each other using social learning. To this end, we outline a framework for social learning in which LLMs share knowledge with each other in a privacy-aware manner using natural language. We evaluate the effectiveness of our framework on various datasets, and propose quantitative methods that measure privacy in this setting. In contrast to previous approaches to collaborative learning, such as common [federated learning](https://blog.research.google/2017/04/federated-learning-collaborative.html) approaches that often rely on gradients, in our framework, agents teach each other purely using natural language.