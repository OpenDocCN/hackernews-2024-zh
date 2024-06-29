<!--yml

category: 未分类

date: 2024-05-27 14:41:44

-->

# Pile

> 来源：[https://pile.eleuther.ai/](https://pile.eleuther.ai/)

## Pile 是什么？

Pile 是一个包含 22 个较小的高质量数据集组合在一起的 825 GiB 多样化、开源的语言建模数据集。

## 下载

Pile 是由[Eye](https://the-eye.eu/)托管的。

有使用或评估 Pile 的模型？[告诉我们](mailto:contact@eleuther.ai)!

## Pile 为什么是一个好的训练集？

最近的研究表明，特别是对于大型模型，数据源的多样性可以提高模型的跨领域知识和下游泛化能力。在我们的评估中，不仅经过 Pile 训练的模型在传统语言建模基准上显示出适度的改进，它们在 Pile BPB 上也显示出显著的提高。

## Pile 为什么是一个很好的基准？

要在 Pile BPB（每字节比特）上得分高，模型必须能够理解包括书籍、GitHub 仓库、网页、聊天记录以及医学、物理、数学、计算机科学和哲学论文在内的许多不同领域。Pile BPB 是在这些领域中的世界知识和推理能力的度量，使其成为大型语言模型通用跨领域文本建模能力的稳健基准。

## 引用

如果您使用 Pile 或其任何组件，请引用我们！

```
@article{pile,
  title={The {P}ile: An 800GB Dataset of Diverse Text for Language Modeling},
  author={Gao, Leo and Biderman, Stella and Black, Sid and Golding, Laurence and Hoppe, Travis and Foster, Charles and Phang, Jason and He, Horace and Thite, Anish and Nabeshima, Noa and Presser, Shawn and Leahy, Connor},
  journal={arXiv preprint arXiv:2101.00027},
  year={2020}
}

```
