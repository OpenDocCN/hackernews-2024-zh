<!--yml
category: 未分类
date: 2024-05-29 12:47:19
-->

# Announcing Grok-1.5

> 来源：[https://x.ai/blog/grok-1.5](https://x.ai/blog/grok-1.5)

## Long Context Understanding

A new feature in Grok-1.5 is the capability to process long contexts of up to 128K tokens within its context window. This allows Grok to have an increased memory capacity of up to 16 times the previous context length, enabling it to utilize information from substantially longer documents.

Furthermore, the model can handle longer and more complex prompts, while still maintaining its instruction-following capability as its context window expands. In the Needle In A Haystack (NIAH) evaluation, Grok-1.5 demonstrated powerful retrieval capabilities for embedded text within contexts of up to 128K tokens in length, achieving perfect retrieval results.

## Grok-1.5 Infra

Cutting-edge Large Language Model (LLMs) research that runs on massive GPU clusters demands robust and flexible infrastructure. Grok-1.5 is built on a custom distributed training framework based on JAX, Rust, and Kubernetes. This training stack enables our team to prototype ideas and train new architectures at scale with minimal effort. A major challenge of training LLMs on large compute clusters is maximizing reliability and uptime of the training job. Our custom training orchestrator ensures that problematic nodes are automatically detected and ejected from the training job. We also optimized checkpointing, data loading, and training job restarts to minimize downtime in the event of a failure. If working on our training stack sounds interesting to you, [apply to join the team](https://x.ai/careers).

## Looking Ahead

Grok-1.5 will soon be available to early testers, and we look forward to receiving your feedback to help us improve Grok. As we gradually roll out Grok-1.5 to a wider audience, we are excited to introduce several new features over the coming days.

 *Note that the GPT-4 scores are taken from the March 2023 release. For MATH and GSM8K, we present maj@1 results. For HumanEval, we report pass@1 benchmark scores.*