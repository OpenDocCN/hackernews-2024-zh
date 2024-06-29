<!--yml

category: 未分类

date: 2024-05-29 13:20:28

-->

# Qdrant 24代码之夏 - Qdrant

> 来源：[https://qdrant.tech/blog/qdrant-summer-of-code-24/](https://qdrant.tech/blog/qdrant-summer-of-code-24/)

Google Summer of Code（#GSoC）今年庆祝其20周年，推出了2024年的计划。在过去的20年中，有19K新贡献者通过这个计划在800多个开源组织的指导下介入了#opensource。Qdrant去年成功参与了该计划。UI 仪表板与非结构化数据可视化以及高级地理过滤两个项目都及时完成，并已成为引擎的一部分。两位年轻贡献者中的一位加入了团队并继续推动项目。

我们非常高兴地宣布，Qdrant 由于未知原因未能进入 GSoc 2024 计划，而是推出了我们自己的**Qdrant 代码之夏**项目，并为贡献者提供津贴！为了不重复造轮子，我们遵循官方计划的所有时间表和规则。

## 我们的项目想法。

我们已准备好一些优秀的项目想法。看看并选择是否要为Rust或基于Python的项目做出贡献。

➡ *基于WASM的维度降维可视化* 📊

在Rust中实现降维算法，编译成WASM，并将WASM代码与Qdrant Web UI集成。

➡ *高效的BM25和Okapi BM25，使用BERT Tokenizer* 🥇

BM25和Okapi BM25是流行的排名算法。Qdrant的FastEmbed支持密集嵌入模型。我们需要一个快速、高效和大规模并行的Rust实现，并为这些实现创建Python绑定。

➡ *Python中的ONNX交叉编码器* ⚔️

将交叉编码器排名模型导出到ONNX运行时，并将此模型与Qdrant的FastEmbed集成，以支持高效的重新排名。

➡ *Rust 中的排名融合算法实现* 🧪

开发各种排名融合算法的Rust实现，包括但不限于互惠排名融合（RRF）。完整列表请见：[https://github.com/AmenRa/ranx](https://github.com/AmenRa/ranx)，并为实现的Rust模块创建Python绑定。

➡ *设置Jepsen来测试Qdrant的分布式保证* 💣

基于其他数据库的实现，设计并编写Jepsen测试，并创建发现的报告或博客。

查看我们Notion页面上的所有细节：[https://www.notion.so/qdrant/GSoC-2024-ideas-1dfcc01070094d87bce104623c4c1110](https://www.notion.so/qdrant/GSoC-2024-ideas-1dfcc01070094d87bce104623c4c1110)

贡献者申请周期将从3月18日开始。我们将通过电子邮件接受申请。让我们一起贡献和庆祝！

在开源中，我们信任！ 🦀🤘🚀
