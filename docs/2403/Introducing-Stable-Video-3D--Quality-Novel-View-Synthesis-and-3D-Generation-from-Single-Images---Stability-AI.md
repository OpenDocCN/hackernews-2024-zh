<!--yml

category: 未分类

date: 2024-05-27 15:04:25

-->

# Introducing Stable Video 3D: Quality Novel View Synthesis and 3D Generation from Single Images — Stability AI

> 来源：[https://stability.ai/news/introducing-stable-video-3d](https://stability.ai/news/introducing-stable-video-3d)

*SV3D以单个对象图像作为输入，并输出该对象的新视角。然后，我们可以使用这些新视角和SV3D来生成3D网格。*

当我们发布[稳定视频扩散](https://stability.ai/news/stable-video-diffusion-open-ai-video-model)时，我们强调了我们的视频模型在各种应用中的多功能性。基于这一基础，我们很高兴地发布了Stable Video 3D。这款新模型推动了3D技术领域的进步，与之前发布的[Stable Zero123](https://stability.ai/news/stable-zero123-3d-generation)以及其他开源替代品如[Zero123-XL](https://objaverse.allenai.org/docs/zero123-xl/)相比，在提供的质量和多视角上有了显著改进。

此版本包含两种变体：

+   SV3D_u：这种变体基于单张图像输入生成轨道视频，无需摄像机条件。

+   SV3D_p：扩展了SVD3_u的能力，此变体可同时处理单张图像和轨道视图，允许沿指定摄像机路径创建3D视频。

现在可以通过[Stability AI Membership](https://stability.ai/membership)进行商业用途。非商业用途可以在[Hugging Face](https://huggingface.co/stabilityai/sv3d)下载模型权重，并查看我们的研究论文[这里](http://arxiv.org/abs/2403.12008)。

**视频扩散的优势**

通过将我们的Stable Video扩散图像到视频扩散模型与摄像机路径调节的结合，Stable Video 3D能够生成对象的多视角视频。与Stable Zero123使用的图像扩散模型相比，视频扩散模型在生成输出的泛化能力和视角一致性方面带来了显著优势。此外，我们提出了利用Stable Video 3D这一强大能力进行改进的3D优化，包括解耦的照明优化以及新的掩膜分数蒸馏采样损失函数。这些技术进一步确保了Stable Video 3D能够从单张图像输入可靠地输出高质量的3D网格。

查看技术报告[这里](https://sv3d.github.io/)，了解Stable Video 3D模型和实验比较的更多细节。

**新视角生成**

Stable Video 3D 引入了在 3D 生成方面的显著进展，特别是在新视角合成（NVS）方面。与以往常常受限于有限视角和输出不一致的方法不同，Stable Video 3D 能够从任何给定角度提供连贯的视角，并具有良好的泛化能力。这种能力不仅增强了姿态可控性，还确保了多视角下一致的物体外观，进一步改善了现实和准确的 3D 生成的关键方面。
