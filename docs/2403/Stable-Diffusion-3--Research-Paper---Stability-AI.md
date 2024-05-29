<!--yml
category: 未分类
date: 2024-05-27 14:38:37
-->

# Stable Diffusion 3: Research Paper — Stability AI

> 来源：[https://stability.ai/news/stable-diffusion-3-research-paper](https://stability.ai/news/stable-diffusion-3-research-paper)

We have compared output images from Stable Diffusion 3 with various other open models including [SDXL](https://stability.ai/news/stable-diffusion-sdxl-1-announcement), [SDXL Turbo](https://stability.ai/news/stability-ai-sdxl-turbo), [Stable Cascade](https://stability.ai/news/introducing-stable-cascade), Playground v2.5 and Pixart-α as well as closed-source systems such as DALL·E 3, Midjourney v6 and Ideogram v1 to evaluate performance based on human feedback. During these tests, human evaluators were provided with example outputs from each model and asked to select the best results based on how closely the model outputs follow the context of the prompt it was given (“prompt following”), how well text was rendered based on the prompt (“typography”) and, which image is of higher aesthetic quality (“visual aesthetics”). 

From the results of our testing, we have found that Stable Diffusion 3 is equal to or outperforms current state-of-the-art text-to-image generation systems in all of the above areas. 

In early, unoptimized inference tests on consumer hardware our largest SD3 model with 8B parameters fits into the 24GB VRAM of a RTX 4090 and takes 34 seconds to generate an image of resolution 1024x1024 when using 50 sampling steps. Additionally, there will be multiple variations of Stable Diffusion 3 during the initial release, ranging from 800m to 8B parameter models to further eliminate hardware barriers.