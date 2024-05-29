<!--yml
category: 未分类
date: 2024-05-27 14:51:12
-->

# Seeing a Rose in Five Thousand Ways

> 来源：[https://cs.stanford.edu/~yzzhang/projects/rose/](https://cs.stanford.edu/~yzzhang/projects/rose/)

<main class="main">

Seeing a Rose in Five Thousand Ways

Stanford University

University of Oxford

Cornell Tech, Cornell University

Stanford University

*People where you live ... grow five thousand roses in one garden ... And yet what they’re looking for could be found in a single rose.*

*--- "The Little Prince" by Antoine de Saint-Exupery*

Abstract

What is a rose, visually? A rose comprises its intrinsics, including the distribution of geometry, texture, and material specific to its object category. With knowledge of these intrinsic properties, we may render roses of different sizes and shapes, in different poses, and under different lighting conditions. In this work, we build a generative model that learns to capture such object intrinsics from a single image, such as a photo of a bouquet. Such an image includes multiple instances of an object type. These instances all share the same intrinsics, but appear different due to a combination of variance within these intrinsics and differences in extrinsic factors, such as pose and illumination. Experiments show that our model successfully learns object intrinsics (distribution of geometry, texture, and material) for a wide range of objects, each from a single Internet image. Our method achieves superior results on multiple downstream tasks, including intrinsic image decomposition, shape and image generation, view synthesis, and relighting.

Learning from Internet Images

After training on a single Internet image containing a group of similar objects, our method can robustly recover the geometry and texture of the object class, and capture the variation among the observed instances. Note that our method has no access to camera intrinsic or extrinsic parameters, object poses, or illumination conditions.

The first column below shows training inputs. The second and third columns show rendered results and corresponding geometry.
Try it yourself: Move the slider to change viewpoints, lighting, and identities.

BibTeX

```
@InProceedings{zhang2023rose,
  author    = {Yunzhi Zhang and Shangzhe Wu and Noah Snavely and Jiajun Wu},
  title     = {Seeing a Rose in Five Thousand Ways},
  booktitle = {CVPR},
  year      = {2023}
}
```

Acknowledgement

We would like to thank Angjoo Kanazawa and Josh Tenenbaum for detailed comments and feedbacks, thank Ruocheng Wang and Kai Zhang for insightful discussions, and thank Yiming Dou and Koven Yu for their advice in data collection. This work is supported in part by the Stanford Institute for Human-Centered AI (HAI), NSF CCRI #2120095, NSF RI #2211258, ONR MURI N00014-22-1-2740, Amazon, Bosch, Ford, Google, and Samsung.
The website design is adapted from [SunStage](https://grail.cs.washington.edu/projects/sunstage/). The template source code for this webpage is available [here](https://github.com/zzyunzhi/template).

</main>