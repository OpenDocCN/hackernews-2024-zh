<!--yml
category: 未分类
date: 2024-05-29 12:30:31
-->

# Inversion: fast, reliable structured LLMs - Rysana

> 来源：[https://rysana.com/inversion](https://rysana.com/inversion)

# **Inversion**: fast, reliable structured LLMs

At Rysana, we've built **Inversion** - our family of structured language models designed to solve the speed, reliability, and reasoning issues in previous AI systems.

Inference speed in characters per second, avg/90/99th percentile across 600 extraction & reasoning tests. (higher is better)

Our first generation models are state of the art in structured tasks such as extraction and function calling while running up to over **100× faster** and **10× lower latency**, outputting **100% reliable structure** with **10,000× less overhead** than the best alternatives, and boasting the deepest support for typed JSON output available anywhere.*

Inversion models do more with less - they use less compute, less time, and less data to produce outputs with higher quality, reliability, and reasoning.

Be among the first to try Inversion. [Sign up for early access](/request-access).

## 100× faster typed LLM inference

We started building Inversion back in February 2023, inspired by the capability leap in general-purpose language models to understand systems and glue together human intent and machine action through natural language and structured data.

As we built products on top of these models, we found that the models were far too unreliable, expensive, and slow for production use in such tasks. We needed to create a new kind of model that could handle our workloads in the real world at scale.

The key insight is that **structured inference** is fundamentally accelerative - and that if we build models that can always reliably output structured data with constraints, we can massively improve both the speed and quality of the outputs.

Time-to-first-token in milliseconds, min/avg/max across 600 extraction & reasoning tests. (lower is better)

We set ourselves to the task of taking the same level quality of outputs from state of the art LLMs for workloads like function calling, [actions/workflows](/docs/lusat), and dynamic UI generation - down from around one minute to under 100 ms, which is roughly the speed at which a human feels a response is instant.

It was important to ensure the outputs would **always** match the data types we expected, in our case this meant being valid against a JSON schema for function arguments or component props.

Such a system would unlock **a Cambrian explosion of new viable AI-native applications**, with reliable real-time feel - everything from humanoid robots and game NPCs that react to complex dynamic environments to natural language interfaces and agents that can understand and act on complex human intent in the blink of an eye.

This means:

1.  We needed to process grammars and schemas in nearly no time,
2.  We needed to bring down the model's latency to nearly instant,
3.  We needed to accelerate inference to over 10,000 Hz.

Time to compile output constraints in microseconds, min/avg/max across 400 JSON schema tests. No caching, fully dynamic, identical hardware. Ratios are average of test ratios, not ratio of test averages. (lower is better)

The first set of components we built were the systems we use to process data structures and constrain model outputs with them. We invented a new kind of compiler and physics-based projection model that achieves **stricter** constraints than the best comparable libraries for typed JSON generation, with around **10,000× faster compilation**.

Our Inversion compiler processes a typical never-before-seen schema/grammar in around 100 μs (microseconds) and samples model constraints at runtime in around 10 μs, supporting up to over **100,000 tokens per second** inference with perfectly structured output.

Percentage of invalid output data structure across 600 extraction & reasoning tests. (lower is better)

Read more about the supported types and constraints in [the docs](/docs/api/json-schema).

The developer experience of knowing you're going to get exactly the type you ask for is bliss.

Always-valid outputs are a game changer for structured workloads, dramatically improving the reliability and reasoning level of LLMs across most tasks. Inversion models often match or beat all other models we've tested against, even compared to models with over 10× as many parameters.

Percentage of correct outputs across 600 tests, grouped by: actions/functions, extraction, and typed data generation. (higher is better)

We're expanding access to the first generation of Inversion models currently, and have begun building the next generation of models targeting on the order of 100,000 Hz inference.

## Accelerating structured inference

You might be wondering - how does any of this actually work?

Why is Inversion so far ahead on multiple fronts that might otherwise seem at odds with each other, demanding a trade-off, like constraint and speed?

When we started with Inversion, it was actually slightly *slower* than similar models - barely faster than the strongest model and barely more reliable, as of early last spring. The first step was to achieve guaranteed structure on common JSON types, which is where the compiler originated.

Next, we leveraged the compiled structures to "invert" the inference process, using the constraint of the output to dynamically scale up and down the amount of compute required to produce each token.

You can think of this as switching on and off large clusters of individual neurons in the model based on the position within the output structure, instead of only modifying token sampling. Mathematically, this is a *projection* from the full model onto a smaller model with undesired sublayers pruned away.

Throughput in characters per second across tests in Inversion models. Third party models to compare in grey. (higher is better)

We built a simple version of this last summer that gained a ~10× inference speed boost, but to achieve the current level of performance we had to completely rewrite our inference and learning systems from scratch, and train new kinds of neural networks to augment the transformers with dynamic acceleration.

Today, our models combine dozens of major improvements and hundreds of minor optimizations over the status quo at every addressable level of the stack, and we have been consistently improving their efficiency month by month.

**Update**: Since we posted this announcement about a month ago, we've made another 114% improvement in inference speed, 100× improvement in sampling, and 4× improvement in compilation speed, among many more reliability and reasoning improvements.

## Inversion v2 & onwards

We're also working on a fundamentally new class of models for the next generation of Inversion - they are not ready yet, but so far we expect another several orders of magnitude improvement across the board, with as many heavy workloads as possible completing in single or double digit milliseconds for fractions of a cent in compute cost and at unprecedented reliability and quality.

We're looking forward to:

*   Breaking the trend of pre-trained generic models, by creating an architecture that can adapt to every human on the fly and constantly improve every day.
*   Generative UI composition on the fly, with typed sandboxed code generation in real time.
*   Improved multilingual support, with deep understanding of dialect and personal nuance.
*   Much, much more!

We're excited to share more about it in the coming months.

## Built for developers

We're aiming to deliver the best possible developer experience, see an example of various approaches to making a request through JS/TS (with `zod`), Python (with `pydantic`), cURL (with JSON Schema):

## Our shared future

Together we'll create a future where technology augments human genius, trivializes mundane tasks, and empowers everyone to live better lives & pursue their passions.

Join us on this journey as we share insights & access to the technology we're building.

Reach us on X [@RysanaAI](https://x.com/RysanaAI)

## Subscribe to our newsletter

Receive an email when we publish a new post. No spam, just the good stuff.

### Summary

We've created **Inversion** - a family of structured language models designed to solve the speed, reliability, and reasoning issues in traditional AI systems.

Our first generation models are state of the art in structured tasks such as extraction and function calling while running up to over **100× faster**, with **10× lower latency**, outputting **100% reliable structure** with **10,000× less overhead** than the best alternatives, and boasting the deepest support for typed JSON output available anywhere.

We're expanding access to the first generation of Inversion models shortly, and have begun building the next generation of models targeting on the order of 100,000 Hz inference.

Be among the first to try Inversion. [Sign up for early access](/request-access).

*All approximate numbers based on 1000 tests as of March-May 2024\. Results may vary. Experimental models not yet finalized. Models are tested by a single client making sequential requests to each model API server in rapid succession. Inference speed is calculated as the total output characters divided by the total request time. Throughput is calculated as the total output characters divided by the total request time minus the time to first tokens. Latency is calculated as the time from start of request to receiving the first tokens on the requesting client. Type error rate is the percentage of outputs that fail to parse according to the requested schema.