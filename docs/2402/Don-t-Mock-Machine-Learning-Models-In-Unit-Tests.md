<!--yml

category: 未分类

date: 2024-05-29 13:27:39

-->

# 不要在单元测试中嘲笑机器学习模型

> 来源：[https://eugeneyan.com/writing/unit-testing-ml/](https://eugeneyan.com/writing/unit-testing-ml/)

我一直在将典型的单元测试实践应用于机器学习代码，但并不简单。在软件中，单元是小的、孤立的逻辑片段，我们可以独立快速地测试。而在机器学习中，模型是从数据中学习到的逻辑块，机器学习代码是用来学习和使用这些派生逻辑块的逻辑。这种差异使得有必要重新思考我们如何对机器学习代码进行单元测试。

## 机器学习代码与常规软件的区别

**在软件中，我们编写包含逻辑的代码；在机器学习中，我们编写学习逻辑并使用该学习逻辑的代码。** 软件代码将输入数据 + 手工逻辑转换为预期输出。然后，我们可以根据断言测试这些输出。相反，机器学习代码将输入数据 + 预期输出转换为学习到的逻辑（即模型）。

\[\text{软件}: \text{输入数据} + 手工逻辑 = \text{预期输出}\] \[\text{机器学习}: \text{输入数据} + \text{预期输出} = 学习到的逻辑\]

因此，在机器学习中，我们不是写包含逻辑的代码，而是编写用于学习逻辑的代码，比如通过 [构建决策树](https://github.com/eugeneyan/testing-ml/blob/master/src/tree/decision_tree.py#L149) 或 [微调幻觉分类器](https://github.com/eugeneyan/visualizing-finetunes/blob/main/3_ft_usb_then_fib.ipynb)。由于作用于输入数据的逻辑嵌入在模型内部，如果我们要测试学习到的逻辑，我们需要加载模型，在一些样本输出上执行推理，然后断言输出是否符合预期输入。

**在软件中，我们通常模拟依赖关系如 API；在 ML 中，我们希望测试实际模型（有时）。** 在单元测试软件时，模拟数据库调用、文件系统访问、发送电子邮件/推送通知等是良好的实践。然而，在机器学习中，有些情况下我们希望针对实际模型进行测试。

例如，我们希望测试每个批次的损失是否减少，以及模型是否可能过拟合（在浪费计算资源之前）。如果模型是一个分类器，我们希望检查推理逻辑是否正确。例如，两个模型可能具有不同的输出类别：[Google 的 T5 NLI 模型](https://huggingface.co/google/t5_11b_trueteacher_and_anli) 将事实一致性分类为类别 1，而 [Meta 的 BART NLI 模型](https://huggingface.co/facebook/bart-large-mnli) 则分类为类别 2！

**机器学习/语言模型可能会很大且难以处理。** 一些神经网络可能有数十亿个参数，超出了笔记本电脑或标准开发环境可以加载的范围。即使我们有足够小型模型的内存，它们加载和推断速度也很慢，在编码过程中进行单元测试时会考验我们的耐心。

## 一些机器学习代码和模型单元测试的指导方针

（这些仍在进行中，我的思路仍在演变中——欢迎所有反馈！）

**使用小型、简单的数据样本。** 避免加载CSV或Parquet文件作为样本数据。（对于集成测试和评估是可以的，但不适合单元测试。）直接在单元测试代码中定义样本数据，使测试自包含，以测试关键功能，如：

+   当存在自定义逻辑时进行训练/测试集分割

+   自定义实现，例如Java中的余弦或欧几里得距离

+   预处理，例如数据增强或编码

+   后处理，例如多样化或筛选推荐

+   处理空或格式错误输入的错误处理

**在可行时，对随机或空权重进行测试。** 例如，我们可以使用随机权重初始化模型配置，以测试输出形状和设备移动（从CPU到GPU再回来）。以下是初始化模型而不必下载权重并断言输出形状的示例：

```
from transformers import AutoConfig, AutoModelForSequenceClassification

model_name = "valhalla/distilbart-mnli-12-1"
config = AutoConfig.from_pretrained("valhalla/distilbart-mnli-12-1")
model = AutoModelForSequenceClassification.from_config(config)
assert model.classification_head.out_proj.out_features == 3 
```

加速库还展示了[初始化空权重模型的示例](https://github.com/huggingface/accelerate/blob/main/tests/test_big_modeling.py#L955)：

```
def test_dispatch_model_bnb(self):
    """Tests that `dispatch_model` quantizes int8 layers"""
    from huggingface_hub import hf_hub_download
    from transformers import AutoConfig, AutoModel, BitsAndBytesConfig
    from transformers.utils.bitsandbytes import replace_with_bnb_linear

    with init_empty_weights():
        model = AutoModel.from_config(AutoConfig.from_pretrained("bigscience/bloom-560m"))

    quantization_config = BitsAndBytesConfig(load_in_8bit=True)
    model = replace_with_bnb_linear(
        model, modules_to_not_convert=["lm_head"], quantization_config=quantization_config
    )

    model_path = hf_hub_download("bigscience/bloom-560m", "pytorch_model.bin")

    model = load_checkpoint_and_dispatch(
        model,
        checkpoint=model_path,
        device_map="balanced",
    )

    assert model.h[0].self_attention.query_key_value.weight.dtype == torch.int8
    assert model.h[0].self_attention.query_key_value.weight.device.index == 0

    assert model.h[(-1)].self_attention.query_key_value.weight.dtype == torch.int8
    assert model.h[(-1)].self_attention.query_key_value.weight.device.index == 1 
```

**对实际模型编写关键测试。** 如果运行时间较长，[标记为慢速](https://docs.pytest.org/en/latest/how-to/mark.html#registering-marks)，仅在需要时运行（例如，预提交和预合并）。一些要点包括：

+   验证训练是否正确完成，例如损失减少、模型过拟合和在少量数据上训练到收敛

+   验证模型输出是否符合期望，例如0.99 = 不安全而非安全

+   验证模型服务器能够启动，接受批量输入，并返回预期输出

**不要测试外部库。** 我们可以假设外部库正常工作。因此，无需测试数据加载器、分词器、优化器等。

• • •

您在机器学习代码和模型单元测试中的最佳实践是什么？我很乐意听听您的意见。[请联系我！](https://twitter.com/eugeneyan)

## 进一步阅读

> 当所有单元测试通过而程序在集成中失败时，模拟发生的问题。模拟是一个理想的地方，其中你的库单元测试表现得像个冠军。是糟糕的模拟还是糟糕的库？还是两者都有问题。然后开发者们被派去调试单元测试… 有点过头。
> 
> 概率测试创建了一个棘手的问题：如果你严格定义你的断言，你会苦于无意义的失败测试，而习惯性地忽略测试失败；但如果你放松定义，你的测试实际上就不再有所断言了。而且没有一个愉快的平衡点：如果你选择中间某个位置，你最终会遇到两个问题。
> 
> 为什么我将我的测试数据内联到单元测试代码中，或者让我的单元测试代码从一个已检入的文件中加载同样的数据，这有什么关系呢？
> 
> > 这样做使得测试变得自包含且更易推理。作为副作用，随机测试在你改变看似无关的 csv 文件时不会意外地出错。按照经验法则，我也只在测试主体中显式定义的输入/输出值上进行断言。这样做节省了大量时间，不必追踪装置定义。
> > 
> 我原以为会有一篇关于在测试中伤害 LLM 感情副作用的文章。

如果您觉得这篇文章有用，请引用此文：

> Yan, Ziyou. (2024年2月). 不要在单元测试中模拟机器学习模型。eugeneyan.com。https://eugeneyan.com/writing/unit-testing-ml/。

或者

```
@article{yan2024unit,
  title   = {Don't Mock Machine Learning Models In Unit Tests},
  author  = {Yan, Ziyou},
  journal = {eugeneyan.com},
  year    = {2024},
  month   = {Feb},
  url     = {https://eugeneyan.com/writing/unit-testing-ml/}
}
```

分享至：
