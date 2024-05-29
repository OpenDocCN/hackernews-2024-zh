<!--yml
category: 未分类
date: 2024-05-27 15:04:25
-->

# Introducing Stable Video 3D: Quality Novel View Synthesis and 3D Generation from Single Images — Stability AI

> 来源：[https://stability.ai/news/introducing-stable-video-3d](https://stability.ai/news/introducing-stable-video-3d)

*SV3D takes a single object image as input and output novel multi-views of that object. We can then use those novel-views and SV3D to generate 3D meshes.*

When we released [Stable Video Diffusion](https://stability.ai/news/stable-video-diffusion-open-ai-video-model), we highlighted the versatility of our video model across various applications. Building upon this foundation, we are excited to release Stable Video 3D. This new model advances the field of 3D technology, delivering greatly improved quality and multi-view when compared to the previously released [Stable Zero123](https://stability.ai/news/stable-zero123-3d-generation), as well as outperforming other open source alternatives such as [Zero123-XL](https://objaverse.allenai.org/docs/zero123-xl/).

This release features two variants: 

*   SV3D_u: This variant generates orbital videos based on single image inputs without camera conditioning. 

*   SV3D_p: Extending the capability of SVD3_u, this variant accommodates both single images and orbital views, allowing for the creation of 3D video along specified camera paths. 

Stable Video 3D can be used now for commercial purposes with a [Stability AI Membership](https://stability.ai/membership). For non-commercial use, you can download the model weights on [Hugging Face](https://huggingface.co/stabilityai/sv3d) and view our research paper [here](http://arxiv.org/abs/2403.12008).

**Advantages of Video Diffusion**

By adapting our Stable Video Diffusion image-to-video diffusion model with the addition of camera path conditioning, Stable Video 3D is able to generate multi-view videos of an object. The use of video diffusion models, in contrast to image diffusion models as used in Stable Zero123, provides major benefits in generalization and view-consistency of generated outputs. Additionally, we propose improved 3D optimization leveraging this powerful capability of Stable Video 3D to generate arbitrary orbits around an object. By further implementing these techniques with disentangled illumination optimization as well as a new masked score distillation sampling loss function, Stable Video 3D is able to reliably output quality 3D meshes from single image inputs.

See the technical report [here](https://sv3d.github.io/) for more details on the Stable Video 3D models and experimental comparisons.

**Novel-View Generation**

Stable Video 3D introduces significant advancements in 3D generation, particularly in novel view synthesis (NVS). Unlike previous approaches that often grapple with limited perspectives and inconsistencies in outputs, Stable Video 3D is able to deliver coherent views from any given angle with proficient generalization. This capability not only enhances pose-controllability, but also ensures consistent object appearance across multiple views, further improving critical aspects of realistic and accurate 3D generations.