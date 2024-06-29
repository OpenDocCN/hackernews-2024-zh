<!--yml

category: 未分类

date: 2024-05-27 14:39:21

-->

# 优化 LLM（大型语言模型） 的技术文档 | kapa.ai 文档

> 来源：[https://docs.kapa.ai/blog/optimizing-technical-documentation-for-llms](https://docs.kapa.ai/blog/optimizing-technical-documentation-for-llms)

我们看到很多 **前瞻性的技术公司**，如 [OpenAI](https://docs.kapa.ai/examples#-openai)、[CircleCI](https://docs.kapa.ai/examples#-circleci)、[Temporal](https://docs.temporal.io/)、[Mixpanel](https://docs.kapa.ai/examples#-mixpanel) 和 [Prisma](https://docs.kapa.ai/examples#-prisma) **采用了基于他们文档训练的大型语言模型（LLMs），以提升他们的开发者体验**。

在 [kapa.ai](https://www.kapa.ai) 我们与超过 80 个技术团队合作，包括上述提到的公司，为他们的开发者实施基于 LLM 的系统。在此过程中，我们了解了如何为 LLM 结构化文档，并希望分享一些考虑此方法的其他人的最佳实践。

### 1\. 接受页面结构和层次[​](#1-embrace-page-structure-and-hierarchy "直达 1\. 接受页面结构和层次")

LLM 擅长导航结构化内容，并依赖上下文提示理解更广泛的图景。一个页面上清晰的标题和子标题的层次结构，有助于 LLM 理解文档中不同部分之间的关系。

另一个很好的例子是 **Temporal** 如何为他们的 SDKs 结构化他们的 [文档](https://docs.temporal.io/dev-guide/java/durable-execution#add-replay-test)。在 Java SDK 中的 `Add a replay test`，这是一个与工作流执行相关的重要功能。文档的层次结构如下：

```
- Development
 - Java SDK - Develop for durability - Add a replay test - ... 
```

这种结构允许 LLM 更有效地导航和理解与 Java SDK 中 `replay tests` 相关的问题的上下文。尤其是因为 `replay tests` 也用于 [其他 SDKs](https://docs.temporal.io/dev-guide/go/durable-execution#why-replay-test)。

### 2\. 按子产品划分文档[​](#2-segment-documentation-by-sub-products "直达 2\. 按子产品划分文档")

为了避免 LLM 们将云端和开源版本等类似产品弄混，确保良好的 **文档层次结构延伸到产品级别** 是非常有帮助的。我们看到，为每个子产品维护单独的文档，可以显著提高 LLM 对上下文和用户意图的理解。

一个很好的例子是 **Prisma** 如何将他们的 [文档](https://www.prisma.io/docs/) 分成 **三大主要产品**：

+   `ORM`: 一个 Node.js 和 TypeScript ORM（核心产品）

+   `Accelerate`: 全球数据库缓存（新发布）

+   `Pulse`: 托管变更数据捕获（早期访问）

在某些情况下，产品文档的分割可以**允许为每个产品部署单独的LLM**，从而可以进一步针对特定用例进行优化。

### 3\. 包括故障排除FAQs[​](#3-include-troubleshooting-faqs "直达链接至3\. 包括故障排除FAQs")

作为LLM的有效信息源，格式为问答的故障排除部分，**反映了用户经常提出的问题**，这使得LLM更容易理解和回答类似的问题。

**OpenAI**的文档是一个很好的例子，特别是在他们的[能力页面](https://platform.openai.com/docs/guides/vision)，每个页面底部都有技术上的FAQ。

最适合LLM的格式是清晰的问题后跟简洁的答案。例如，一个**结构良好的常见问题解答（FAQ）部分**可能如下所示：

```
### [Common User Questions]   [Concise 1-2 Sentence Answer] 
```

当[查看指标](https://docs.kapa.ai/#how-does-kapa-work-)以了解LLM响应中特定来源使用频率时，我们发现**技术FAQs通常是使用最频繁的来源**。

### 4\. 提供自包含的示例代码片段[​](#4-provide-self-contained-example-code-snippets "直达链接至4\. 提供自包含的示例代码片段")

包含**小型、独立的代码片段**可能会很有帮助，特别是对于依赖于大型且复杂SDK或API的产品。

例如，**Mixpanel**在其[文档](https://docs.mixpanel.com/docs/tracking-methods/sdks/javascript#incrementing-numeric-properties)中有效地使用代码片段，其中包含大量跟踪和分析实现代码。例如，为了增加数值属性，他们提供以下代码片段展示`mixpanel.people.increment`方法：

在包含代码时，还有两个有用的提示是确保片段（1）**在代码上方有简要说明**以阐明其目的和用法，以及（2）**在代码内部有注释**以解释逻辑和功能。这两者都有助于LLM进一步理解代码片段的上下文和目的。

虽然与文档结构关联较少，但如果不提到建立**社区论坛作为开发者和LLM获取帮助的来源**，本指南将显得不完整，尤其是对于**未记录主题**。

例如，**CircleCI**有一个[活跃且维护良好的社区论坛](https://discuss.circleci.com/)，用户可以在那里提问并从其他用户和CircleCI员工获得帮助。

与FAQ类似，技术论坛非常有效，因为它**反映了用户经常提出的问题**。论坛也可作为暂时解决方案，用于尚未包含在官方文档中的问题。

注意在包含论坛内容时需要谨慎。应用筛选器，如**仅包括标记为** `resolved` **或** `accepted` **的响应**，有助于确保内容的相关性，并**包含指向原始论坛帖子的链接**，以确保作者得到适当的归属。

### 6\. 更多实用技巧[​](#6-a-few-more-practical-tips "直达链接到6\. 更多实用技巧")

除了以上内容，这里还有一些解决LLM常见文档相关问题的实用战术技巧：

+   **避免将文档存储在文件中：** 将相关内容直接放在文档中，而不是存储在诸如PDF之类的链接文件中，因为LLM难以解析这些文件。

+   **为图像编写文本描述：** 确保通过截图传达的信息也以文本形式描述，因为LLM更有效地解析文本。

+   **为REST API提供OpenAPI规范：** 提供结构化的OpenAPI规范可以利用自定义解析器，从而改进LLM的格式化能力。

+   **包括示例请求和响应：** 在API描述中包含这些内容，以便LLM具体了解如何使用你的API。

+   **定义特定的首字母缩写和术语：** 在文档中澄清所有首字母缩写和专业术语，以帮助LLM理解。

+   **在代码示例中包含必要的导入：** 这确保了代码示例可以在没有额外上下文的情况下运行。

这些技巧可以显著提高LLM理解和准确响应用户查询的能力。

* * *

遵循这些指南，你可以显著**增强技术文档对LLM的实用性和资源性**，最终改善开发者的体验。

如果你对在技术资源上测试 **LLM** 感兴趣，那么请[点击此处进行快速演示](https://www.kapa.ai/early-access)，或者如果你对如何**进一步优化你的技术文档**为LLM有疑问，请联系**kapa团队**（邮箱：[founders@kapa.ai](mailto:founders@kapa.ai)）。
