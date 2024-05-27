<!--yml
category: 未分类
date: 2024-05-27 13:31:01
-->

# Llamafile’s progress, four months in - Mozilla Hacks - the Web developer blog

> 来源：[https://hacks.mozilla.org/2024/04/llamafiles-progress-four-months-in/](https://hacks.mozilla.org/2024/04/llamafiles-progress-four-months-in/)

When Mozilla’s Innovation group [first launched](https://hacks.mozilla.org/2023/11/introducing-llamafile/) the [llamafile project](https://github.com/Mozilla-Ocho/llamafile) late last year, we were thrilled by the immediate positive response from open source AI developers. It’s become one of Mozilla’s top three most-favorited repositories on GitHub, attracting a number of contributors, some excellent PRs, and a growing community on our [Discord server](https://discord.gg/YTgM42NZEr).

Through it all, lead developer and project visionary [Justine Tunney](https://github.com/jart) has remained hard at work on a wide variety of fundamental improvements to the project. Just last night, Justine [shipped the v0.8 release of llamafile](https://github.com/Mozilla-Ocho/llamafile/releases/tag/0.8), which includes not only support for the very latest open models, but also a number of big performance improvements for CPU inference.

As a result of Justine’s work, today llamafile is both the easiest *and fastest* way to run a wide range of open large language models on your own hardware. See for yourself: with llamafile, you can run Meta’s just-released [LLaMA 3 model](https://huggingface.co/jartine/Meta-Llama-3-8B-Instruct-llamafile)–which rivals the very best models available in its size class–on an everyday Macbook.

How did we do it? To explain that, let’s take a step back and tell you about everything that’s changed since v0.1.

**tinyBLAS: democratizing GPU support for NVIDIA** ***and*** **AMD**

llamafile is built atop the now-legendary [llama.cpp](https://github.com/ggerganov/llama.cpp) project. llama.cpp supports GPU-accelerated inference for NVIDIA processors via the cuBLAS linear algebra library, but that requires users to install NVIDIA’s CUDA SDK. We felt uncomfortable with that fact, because it conflicts with our project goal of building a fully open-source and transparent AI stack that anyone can run on commodity hardware. And besides, getting CUDA set up correctly can be a bear on some systems. There had to be a better way.

With the community’s help (here’s looking at you, [@ahgamut](https://ahgamut.github.io/) and [@mrdomino](https://github.com/mrdomino)!), we created our own solution: it’s called tinyBLAS, and it’s llamafile’s brand-new and highly efficient linear algebra library. tinyBLAS makes NVIDIA acceleration simple and seamless for llamafile users. On Windows, you don’t even need to install CUDA at all; all you need is the display driver you’ve probably already installed.

But tinyBLAS is about more than just NVIDIA: it supports AMD GPUs, as well. This is no small feat. While AMD commands a respectable 20% of today’s GPU market, poor software and driver support have historically made them a secondary player in the machine learning space. That’s a shame, given that AMD’s GPUs offer high performance, are price competitive, and are widely available.

One of llamafile’s goals is to democratize access to open source AI technology, and that means getting AMD a seat at the table. That’s exactly what we’ve done: with llamafile’s tinyBLAS, you can now easily make full use of your AMD GPU to accelerate local inference. And, as with CUDA, if you’re a Windows user you don’t even have to install AMD’s ROCm SDK.

All of this means that, for many users, llamafile will automatically use your GPU right out of the box, with little to no effort on your part.

**CPU performance gains for faster local AI**

Here at Mozilla, we are keenly interested in the promise of “local AI,” in which AI models and applications run directly on end-user hardware instead of in the cloud. Local AI is exciting because it opens up the possibility of more user control over these systems and greater privacy and security for users.

But many consumer devices lack the high-end GPUs that are often required for inference tasks. llama.cpp has been a game-changer in this regard because it makes local inference both possible and usably performant on CPUs instead of just GPUs. 

Justine’s recent work on llamafile has now pushed the state of the art even further. As documented in [her detailed blog post](http://justine.lol/matmul/) on the subject, by writing 84 new matrix multiplication kernels she was able to increase llamafile’s prompt evaluation performance by an astonishing 10x compared to our previous release. This is a substantial and impactful step forward in the quest to make local AI viable on consumer hardware.

This work is also a great example of our commitment to the open source AI community. After completing this work we immediately [submitted a PR](https://github.com/ggerganov/llama.cpp/pull/6414) to upstream these performance improvements to llama.cpp. This was just the latest of a number of enhancements we’ve contributed back to llama.cpp, a practice we plan to continue.

**Raspberry Pi performance gains**

Speaking of consumer hardware, there are few examples that are both more interesting and more humble than the beloved Raspberry Pi. For a bargain basement price, you get a full-featured computer running Linux with plenty of computing power for typical desktop uses. It’s an impressive package, but historically it hasn’t been considered a viable platform for AI applications.

Not any more. llamafile has now been optimized for the latest model (the Raspberry Pi 5), and the result is that a number of small LLMs–such as Rocket-3B ([download](https://huggingface.co/jartine/rocket-3B-llamafile/resolve/main/rocket-3b.Q5_K_M.llamafile?download=true)), TinyLLaMA-1.5B ([download](https://huggingface.co/jartine/TinyLlama-1.1B-Chat-v1.0-GGUF/resolve/main/TinyLlama-1.1B-Chat-v1.0.Q5_K_M.llamafile?download=true)), and Phi-2 ([download](https://huggingface.co/jartine/phi-2-llamafile/resolve/main/phi-2.Q5_K_M.llamafile?download=true))–run at usable speeds on one of the least expensive computers available today. We’ve seen prompt evaluation speeds of [up to 80 tokens/sec](https://twitter.com/JustineTunney/status/1776440470152867930) in some cases!

**Keeping up with the latest models**

The pace of progress in the open model space has been [stunningly fast](https://twitter.com/maximelabonne/status/1779123021480865807). Over the past few months, hundreds of models have been released or updated via fine-tuning. Along the way, there has been a clear trend of ever-increasing model performance and ever-smaller model sizes.

The llama.cpp project has been doing an excellent job of keeping up with all of these new models, frequently rolling-out support for new architectures and model features within days of their release.

For our part we’ve been keeping llamafile closely synced with llama.cpp so that we can support all the same models. Given the complexity of both projects, this has been no small feat, so we’re lucky to have Justine on the case.

Today, you can today use the very latest and most capable open models with llamafile thanks to her hard work. For example, we were able to roll-out llamafiles for Meta’s newest LLaMA 3 models–[8B-Instruct](https://huggingface.co/jartine/Meta-Llama-3-8B-Instruct-llamafile) and [70B-Instruct](https://huggingface.co/jartine/Meta-Llama-3-70B-Instruct-llamafile)–within a day of their release. With yesterday’s 0.8 release, llamafile can also run Grok, Mixtral 8x22B, and Command-R.

**Creating your own llamafiles**

Since the day that llamafile shipped people have wanted to create their own llamafiles. Previously, this required a number of steps, but today you can do it with a single command, e.g.:

`llamafile-convert [model.gguf]`

In just moments, this will produce a “model.llamafile” file that is ready for immediate use. Our thanks to community member [@chan1012](https://github.com/chand1012) for contributing this helpful improvement.

In a related development, Hugging Face recently added official support for llamafile within their model hub. This means you can now [search and filter](https://huggingface.co/models?library=llamafile&sort=trending) Hugging Face specifically for llamafiles created and distributed by other people in the open source community.

**OpenAI-compatible API server**

Since it’s built on top of llama.cpp, llamafile inherits that project’s server component, which provides OpenAI-compatible API endpoints. This enables developers who are building on top of OpenAI to switch to using open models instead. At Mozilla we very much want to support this kind of future: one where open-source AI is a viable alternative to centralized, closed, commercial offerings.

While open models do not yet fully rival the capabilities of closed models, they’re making rapid progress. We believe that making it easier to pivot existing code over to executing against open models will increase demand and further fuel this progress.

Over the past few months, we’ve invested effort in extending these endpoints, both to increase functionality and improve compatibility. Today, llamafile can serve as a drop-in replacement for OpenAI in a wide variety of use cases.

We want to further extend our API server’s capabilities, and we’re eager to hear what developers want and need. What’s holding you back from using open models? What features, capabilities, or tools do you need? [Let us know](https://discord.gg/YTgM42NZEr)!

**Integrations with other open source AI projects**

Finally, it’s been a delight to see llamafile adopted by independent developers and integrated into leading open source AI projects (like [Open Interpreter](https://github.com/OpenInterpreter/open-interpreter)). Kudos in particular to our own [Kate Silverstein](https://github.com/k8si) who landed PRs that add llamafile support to [LangChain](https://github.com/langchain-ai/langchain) and [LlamaIndex](https://github.com/run-llama/llama_index) (with [AutoGPT](https://github.com/Significant-Gravitas/AutoGPT) coming soon).

If you’re a maintainer or contributor to an open source AI project that you feel would benefit from llamafile integration, [let us know how we can help](https://discord.gg/YTgM42NZEr).

**Join us!**

The llamafile project is just getting started, and it’s also only the first step in a major new initiative on Mozilla’s part to contribute to and participate in the open source AI community. We’ll have more to share about that soon, but for now: I invite you to join us on the llamafile project!

The best place to connect with both the llamafile team at Mozilla and the overall llamafile community is over at our Discord server, which has [a dedicated channel just for llamafile](https://discord.gg/YTgM42NZEr). And of course, your enhancement requests, issues, and PRs are always welcome over at our [GitHub repo](https://github.com/Mozilla-Ocho/llamafile).

I hope you’ll join us. The next few months are going to be even more interesting and unexpected than the last, both for llamafile and for open source AI itself.

Stephen leads open source AI projects (including llamafile) in Mozilla's Innovation group. He previously managed social bookmarking pioneer del.icio.us; co-founded Storium, Blockboard, and FairSpin; and worked on Yahoo Search and BEA WebLogic.

[More articles by Stephen Hood…](https://hacks.mozilla.org/author/slangtonhoodmozilla-com/)