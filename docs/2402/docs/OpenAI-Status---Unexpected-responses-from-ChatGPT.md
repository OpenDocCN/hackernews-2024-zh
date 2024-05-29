<!--yml
category: 未分类
date: 2024-05-29 13:18:35
-->

# OpenAI Status - Unexpected responses from ChatGPT

> 来源：[https://status.openai.com/incidents/ssg8fh7sfyz3](https://status.openai.com/incidents/ssg8fh7sfyz3)

On February 20, 2024, an optimization to the user experience introduced a bug with how the model processes language.

LLMs generate responses by randomly sampling words based in part on probabilities. Their “language” consists of numbers that map to tokens.

In this case, the bug was in the step where the model chooses these numbers. Akin to being lost in translation, the model chose slightly wrong numbers, which produced word sequences that made no sense. More technically, inference kernels produced incorrect results when used in certain GPU configurations.

Upon identifying the cause of this incident, we rolled out a fix and confirmed that the incident was resolved.