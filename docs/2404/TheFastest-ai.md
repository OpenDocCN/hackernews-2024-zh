<!--yml
category: 未分类
date: 2024-05-27 13:26:47
-->

# TheFastest.ai

> 来源：[https://thefastest.ai](https://thefastest.ai)

Definitions

===========

*   **Model:** The LLM used.
*   **TTFT:** Time To First Token. This is how quickly the model can process the incoming request and begin to output text, and translates directly into how quickly the UI starts to update. Lower values = lower latency/faster performance.
*   **TPS:** Tokens Per Second. This is how quickly the model can produce text and controls how quickly the full response shows up in the UI. Higher values = more throughput/faster performance.
*   **Total:** The total time from the start of the request until the response is complete, i.e., the last token has been generated. Total time = TTFT + TPS * Tokens. Lower values = lower latency/faster performance.

Methodology

===========

**Distributed Footprint:** We run our tools daily in multiple data centers using [Fly.io](https://fly.io/docs/reference/regions/). Currently we run in cdg, iad, and sea.

**Connection Warmup:** A warmup connection is made to remove any HTTP connection setup latency.

**TTFT Measurement:** The TTFT clock starts when the HTTP request is made and stops when the first token is received in the response stream.

**Max Tokens:** The number of output tokens is set to 20 (~100 chars), the length of a typical conversational sentence.

**Try 3, Keep 1:** For each provider, three separate inferences are performed, and the best result is kept (to remove any outliers due to queuing etc).

Sources

======

**Raw Data:** All data is in this public [GCS bucket](https://storage.googleapis.com/thefastest-data).

**Benchmarking Tools:** The full test suite is available in the [ai-benchmarks repo](https://github.com/fixie-ai/ai-benchmarks).

**Website:** Full source code for this site is on [GitHub](https://github.com/fixie-ai/fastest.ai).