<!--yml

category: 未分类

date: 2024-05-27 12:57:26

-->

# 工具使用（函数调用）- Anthropic

> 来源：[https://docs.anthropic.com/claude/docs/tool-use](https://docs.anthropic.com/claude/docs/tool-use)

克劳德能够与外部客户端工具和功能进行交互，使您能够为克劳德配备自定义工具，以执行更广泛的任务。

**工具使用公测**

我们很高兴宣布，工具使用现已进入公测阶段！要访问此功能，您需要在 API 请求中包含`anthropic-beta: tools-2024-05-16`头。

在接下来的几周中，我们将对这个开放公测进行迭代，所以非常感谢您的所有反馈。请使用这个[表单](https://forms.gle/BFnYc6iCkWoRzFgk7)分享您的想法和建议。

这里是通过消息 API 为克劳德提供工具的示例：
