<!--yml

类别：未分类

日期：2024-05-27 14:51:12

-->

# 以五千种方式看玫瑰

> 来源：[https://cs.stanford.edu/~yzzhang/projects/rose/](https://cs.stanford.edu/~yzzhang/projects/rose/)

<main class="main">

以五千种方式看玫瑰

斯坦福大学

牛津大学

康奈尔科技，康奈尔大学

斯坦福大学

*你所在的地方的人们…在一个花园里种植五千朵玫瑰…然而，他们所寻找的可能在一朵玫瑰中找到。*

*--- 安托万·德·圣埃克苏佩里《小王子》*

摘要

视觉上，玫瑰是什么？一朵玫瑰包含其固有属性，包括几何、纹理和材料的分布，特定于其对象类别。有了这些固有属性的知识，我们可以呈现不同大小和形状、不同姿态和不同光照条件下的玫瑰。在这项工作中，我们构建了一个生成模型，从单个图像（如花束的照片）中学习捕捉这样的对象固有属性。这样的图像包括对象类型的多个实例。这些实例都共享相同的内在属性，但由于这些内在属性的变异以及外部因素（如姿态和光照）的差异，它们看起来不同。实验表明，我们的模型成功地学习了多个对象的内在属性（几何、纹理和材料的分布），每个对象来自一个单独的互联网图像。我们的方法在多个下游任务上取得了优异的结果，包括内在图像分解、形状和图像生成、视图合成和重新照明。

从互联网图像中学习

在仅对包含一组相似对象的单个互联网图像进行训练后，我们的方法可以稳健地恢复对象类的几何和纹理，并捕捉所观察到的实例之间的变化。请注意，我们的方法无法访问相机内参或外参参数、对象姿态或光照条件。

下面的第一列显示了训练输入。第二列和第三列显示了渲染结果和相应的几何。

试试自己：移动滑块以改变视角、光照和身份。

BibTeX

```
@InProceedings{zhang2023rose,
  author    = {Yunzhi Zhang and Shangzhe Wu and Noah Snavely and Jiajun Wu},
  title     = {Seeing a Rose in Five Thousand Ways},
  booktitle = {CVPR},
  year      = {2023}
}
```

致谢

我们要感谢 Angjoo Kanazawa 和 Josh Tenenbaum 的详细评论和反馈，感谢 Ruocheng Wang 和 Kai Zhang 的深入讨论，感谢 Yiming Dou 和 Koven Yu 在数据收集方面的建议。这项工作部分得到了斯坦福人类中心人工智能研究所 (HAI)、NSF CCRI #2120095、NSF RI #2211258、ONR MURI N00014-22-1-2740、亚马逊、博世、福特、谷歌和三星的支持。

网站设计改编自[SunStage](https://grail.cs.washington.edu/projects/sunstage/)。此网页的模板源代码可在[此处](https://github.com/zzyunzhi/template)找到。

</main>
