<!--yml
category: 未分类
date: 2024-05-27 14:41:44
-->

# The Pile

> 来源：[https://pile.eleuther.ai/](https://pile.eleuther.ai/)

## What is the Pile?

The Pile is a 825 GiB diverse, open source language modelling data set that consists of 22 smaller, high-quality datasets combined together.

## Download

The Pile is hosted by [the Eye](https://the-eye.eu/).

Have a model that uses or evaluates on the Pile? [Let us know](mailto:contact@eleuther.ai)!

## Why is the Pile a good training set?

Recent work has shown that especially for large models, diversity in data sources improves general cross-domain knowledge of the model, as well as downstream generalization capability. In our evaluations, not only do models trained on the Pile show moderate improvements in traditional language modeling benchmarks, they also show significant improvements on Pile BPB.

## Why is the Pile a good benchmark?

To score well on Pile BPB (bits per byte), a model must be able to understand many disparate domains including books, github repositories, webpages, chat logs, and medical, physics, math, computer science, and philosophy papers. Pile BPB is a measure of world knowledge and reasoning ability in these domains, making it a robust benchmark of general, cross-domain text modeling ability for large language models.

## Citing

If you use the Pile or any of the components, please cite us!

```
@article{pile,
  title={The {P}ile: An 800GB Dataset of Diverse Text for Language Modeling},
  author={Gao, Leo and Biderman, Stella and Black, Sid and Golding, Laurence and Hoppe, Travis and Foster, Charles and Phang, Jason and He, Horace and Thite, Anish and Nabeshima, Noa and Presser, Shawn and Leahy, Connor},
  journal={arXiv preprint arXiv:2101.00027},
  year={2020}
}

```