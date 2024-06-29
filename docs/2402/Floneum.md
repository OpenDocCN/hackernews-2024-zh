<!--yml

类别：未分类

日期：2024-05-29 13:26:57

-->

# Floneum

> 来源：[https://floneum.com/blog/kalosm_0_2/](https://floneum.com/blog/kalosm_0_2/)

我们很高兴地宣布Kalosm v0.2.0的发布！此版本包括许多新功能、改进和错误修复，包括：

+   任务和代理

+   任务评估

+   提示自动调整

+   正则表达式验证

+   Surreal Database Integration

+   RAG改进

+   性能改进

+   新模型

Kalosm现在包括用于运行、评估和改进任务和代理的实用程序。

让我们构建一个简单的任务和代理来演示新功能。

```
use kalosm::language::*;   #[tokio::main] async fn main() {   // You can use tasks with llama, mistral or phi models.
  let llm = Phi::new_chat();    // Now we can create a task. We will create a simple assistant that identifies keywords in a sentence.
  let task = Task::builder(&llm, "You are an assistant who identifies keywords in a sentence. When identifying keywords, you will output the keywords in a comma-separated list.")   // You can optionally add constraints to the task.
  // accept 2-5 keywords of 2-15 characters each
 .with_constraints(RegexParser::new(r#"The keywords are:([ a-z]{2,15},){2,5}"#).unwrap())   /// You can also add examples to the task to help the agent learn how to solve math problems.
 .with_example("What is the weather like in New York?", "The keywords are: weather, new york, ")
 .with_example("What is the capital of France?", "The keywords are: capital, france, ")
 .build();    // The first time we use the task, it will load the model and prompt which will take a bit longer.
 task.run("What is the temperature in Chicago?", &llm)
 .await .unwrap()
 .to_std_out()
 .await .unwrap(); } 
```

或者您可以使用更复杂的约束条件来

```
// First, we need to create a Llama instance. We will use the default chat model (open chat) for this example. Tasks work with chat and text generation models. let mut llm = Llama::new_chat();   // Next (optionally), we can define constraints for the task. In this example, we will use a regex to validate the output of the task. let constraints =  RegexParser::new(r"(Step \d: \d+ [+\-*/] \d+ = \d+\n){1,3}Output: \d+").unwrap();   // Now we can create a task. We will create a simple math problem solving task. let task = Task::builder(&llm, "You are an assistant who solves math problems. When solving problems, you will always solve problems step by step with one step per line. Once you have solved the problem, you will output the result in the format 'Output: <result>'.")  .with_constraints(constraints)   // You can also add examples to the task to help the agent learn how to solve math problems.  .with_example("What is 1 + 2?", "Step 1: 1 + 2 = 3\nOutput: 3")  .with_example("What is 3 + 4?", "Step 1: 3 + 4 = 7\nOutput: 7")  .with_example("What is (4 + 8) / 3?", "Step 1: 4 + 8 = 12\nStep 2: 12 / 3 = 4\nOutput: 4")  .build();   // The first time we use the task, it will load the model and prompt which will take a bit longer. task.run("What is 2 + 2?", &llm)  .await  .unwrap()  .to_std_out()  .await  .unwrap(); 
```

任务可以在运行之间高效地重复使用会话，这可以显著加快流程：

```
question 1 Step 1: 2 + 2 = 4 Output: 4. first question took: 18.371939709s question 2 Step 1: 4 + 4 = 8 Output: 8 second question took: 5.723529959s question 3 Step 1: 7 + 5 = 12 Step 2: 12 / 2 = 6 Output: 6 third question took: 9.303650625s 
```

第三个问题需要更复杂的计算，并且需要更多的令牌来解决，但解决问题的时间仍然明显快于第一个问题。第一个问题的会话被用于第二个和第三个问题，这使得第二个和第三个问题运行更快。

+   **评估抽象：**引入评估抽象，提供增强功能。([#113](https://github.com/floneum/floneum/pull/113))

```
+-----------------+-------+ | Statistic       | Value | +=========================+ | Mean            | 0.75 | |-----------------+-------| | Median          | 0.77 | |-----------------+-------| | Min             | 0.49 | |-----------------+-------| | Max             | 0.94 | |-----------------+-------| | 25th Percentile | 0.67 | |-----------------+-------| | 75th Percentile | 0.81 | +-----------------+-------+ +------------------------------------------------------------------------------------------+-------------------------------------------------------------------------------------------+--------------------+ | Expected Output                                                                          | Actual Output                                                                             | Score              | +===========================================================================================================================================================================================================+ | What are Floneum plugins?                                                                | What is designed to support an expanding ecosystem of plugins?                            | 0.49 (low outlier) | |------------------------------------------------------------------------------------------+-------------------------------------------------------------------------------------------+--------------------| | What is supervised learning in machine learning?                                         | What is required for machine learning models in order to make accurate predictions?       | 0.50 (low outlier) | |------------------------------------------------------------------------------------------+-------------------------------------------------------------------------------------------+--------------------| | What distinguishes open-source software from proprietary software?                       | What type of access does open source software provide for its users?                      | 0.64 | |------------------------------------------------------------------------------------------+-------------------------------------------------------------------------------------------+--------------------| | How does blockchain contribute to the security of cryptocurrencies?                      | What is the advantage of cryptocurrency over traditional currencies in terms of security? | 0.65 | |------------------------------------------------------------------------------------------+-------------------------------------------------------------------------------------------+--------------------| | What are the tradeoffs of using chat GPT?                                                | What is an example of how using ChatGPT may become challenging in certain situations?     | 0.65 | |------------------------------------------------------------------------------------------+-------------------------------------------------------------------------------------------+--------------------| | What is the relationship between DevOps, continuous integration and continuous delivery? | What is the main goal of DevOps?                                                          | 0.67 | |------------------------------------------------------------------------------------------+-------------------------------------------------------------------------------------------+--------------------| | ... 18 more                                                                              |                                                                                           | 0.80 (average)     | +------------------------------------------------------------------------------------------+-------------------------------------------------------------------------------------------+--------------------+ | Score Histogram        | | 0.00 - 0.10:           | | 0.10 - 0.20:           | | 0.20 - 0.30:           | | 0.30 - 0.40:           | | 0.40 - 0.50: *         | | 0.50 - 0.60: *         | | 0.60 - 0.70: ******    | | 0.70 - 0.80: ********* | | 0.80 - 0.90: *****     | | 0.90 - 1.00: **        | 
```

+   **提示自动调整：**现在可以自动调整提示以获得更好的性能。([#132](https://github.com/floneum/floneum/pull/132))

让我们通过一个示例来看看新的提示自动调整功能。作为[RAG改进](#Improved-Chunking-Strategies)的一部分，kalosm包括一个任务，该任务生成关于文本的假设问题，用于嵌入模型，该模型可根据文本的含义为该部分找到相似文档。我们可以调整该任务以找到任务的最佳示例，使用PromptAnnealer：

```
const EXAMPLES: &[(&str, &str)]= &[
 ("An example input to the task", "An example output to the task"),
  // ...more examples ];   let llm = Phi::v2(); const PREFIX: &str = "Questions that are answered by the previous text: "; const QUESTION_STARTERS: [&str; 9] = [
 "Who", "What", "When", "Where", "Why", "How", "Which", "Whom", "Whose", ]; let constraints = LiteralParser::new(PREFIX).then(
 IndexParser::new(  QUESTION_STARTERS
 .iter()
 .copied()
 .map(LiteralParser::new)
 .collect::<Vec<_>>(), ) .then(StopOn::new("?").filter_characters(
 |c| matches!(c, ' ' | '?' | 'a'..='z' | 'A'..='Z' | '0'..='9' | ','),
 )) .repeat(1..=5), ); let task = Task::builder("You generate hypothetical questions that may be answered by the given text. The questions restate any information necessary to understand the question")
 .with_constraints(constraints); let mut annealing = kalosm::PromptAnnealer::builder(&llm, EXAMPLES, task)
 .with_initial_temperature(0.6)
 .with_initial_choice_range(1..4)
 .build()
 .await;   let result = annealing.run().await;   println!("Result: {:?}", result); 
```

这是提示退火器为任务找到的最佳示例集：

| 输入 | 输出 |
| --- | --- |
| 传统数据库依赖固定模式，而像MongoDB这样的NoSQL数据库提供灵活的结构，允许您以更动态的方式存储和检索数据。这种灵活性对于具有不断变化数据需求的应用程序特别有益。 | “MongoDB与传统数据库有何不同？ |
| 区块链技术，超越加密货币，正在被探索用于智能合约等应用。智能合约是具有协议条款直接编写成代码的自动执行合同。 | “区块链技术在智能合约概念中如何被利用？ |

将这两个示例输入任务中，与从任务中选择两个随机示例相比，可以实现所有其他示例的相似度得分为0.71，而选择两个随机示例只能实现所有其他示例的相似度得分为0.62。

我们从kalosm的初始版本发布中得到的一些反馈是，受限生成的约束条件过于复杂。Kalosm中的约束条件有两个目的：

1.  验证：约束可以用来验证模型的输出。模型只会输出可以被约束解析的文本。这样可以确保模型的输出符合您的预期格式。

例如，您可能希望强制模型响应始终以指导模型的前缀开头：

```
use kalosm::language::*;   #[tokio::main] async fn main() {
  let llm = Llama::default();  let prompt = "Generate a list of 10 words you use to describe yourself: ";    let validator = LiteralParser::new("(Responding as a pirate) ").then(OneLine);
  let stream = llm.stream_structured_text(prompt, validator).await.unwrap();   stream.to_std_out().await.unwrap(); } 
```

1.  解析：约束可用于解析模型的输出。当您想要从LLM生成特定结构而不必编写单独的验证和解析逻辑时，这将非常有用。

例如，您可能希望生成一个包含10个数字的列表：

```
use kalosm::language::*;   #[tokio::main] async fn main() {
  let llm = Llama::default();  let prompt = "Prime numbers: ";    let validator = <[f32; 10] as HasParser>::new_parser();
  let words = llm.stream_structured_text(prompt, validator).await.unwrap();    let result: [f32; 10] = words.result().await.unwrap();
 println!("Prime numbers: {:?}", result); } 
```

如果您只需要验证模型的输出，现有的约束可能比您需要的更复杂。在此版本中，我们增加了对正则表达式验证的支持。这样可以更轻松地验证模型的输出，而无需处理解析：

```
use kalosm::language::*;   #[tokio::main] async fn main() {
  let llm = Llama::default();  let prompt = "Generate a list of 10 words you use to describe yourself: ";    let validator = RegexParser::new(r"\(Responding as a pirate\) ([a-z]{1,10}, ){10}").unwrap();
  let stream = llm.stream_structured_text(prompt, validator).await.unwrap();   stream.to_std_out().await.unwrap(); } 
```

结合LLM使用时，向量数据库非常有用。它们可以基于文本的含义而不仅仅是使用的单词来存储和检索类似文档。然而，向量数据库仅处理非常有限的用例。在此版本中，我们为更传统的数据库用例添加了Surreal DB的支持。Surreal DB可以嵌入到您的应用程序中，用于本地存储和通过网络存储和检索数据。

Kalosm 0.2 允许您在Surreal DB中创建由向量索引的表。然后，您可以将文档（或其他嵌入）插入到表中，并根据文本的含义查询类似的文档。

让我们看看如何使用Surreal DB集成来存储和检索基于文本含义的类似文档：

```
use comfy_table::{Cell, Color, Row, Table}; use kalosm::language::*; use kalosm::*; use std::path::PathBuf; use surrealdb::{engine::local::RocksDb, Surreal};   let exists = std::path::Path::new("./db").exists();   // Create or open a new database at ./db/temp.db let db = Surreal::new::<RocksDb>("./db/temp.db").await.unwrap();   // Select a specific namespace / database within the database db.use_ns("test").use_db("test").await.unwrap();   // Create a new document table that uses arroy for fast vector search let document_table = db .document_table_builder("documents")
  // Store the embedding database in the same directory as the document table
 .at("./db/embeddings.db")
 .build()
 .unwrap();   // If the database doesn't exist, create it and insert some documents if !exists {
 std::fs::create_dir_all("documents").unwrap();
  // Load some files from a directory
  let documents = DocumentFolder::try_from(PathBuf::from("./documents")).unwrap();    // Convert the folder into a vector of documents
  let documents = documents.into_documents().await.unwrap();
  for document in documents {  // And insert the documents into the table (this will automatically chunk and embed the documents before they are inserted into the table and vector database)
 document_table.insert(document).await.unwrap();
 } }   // Now we can query the table for similar documents based on the meaning of the text loop {
  // Get a query from the user and embed that query into a vector
  let user_question = prompt_input("Query: ").unwrap();
  let user_question_embedding = document_table .embedding_model_mut()
 .embed(&user_question)
 .await .unwrap();    // Select the 5 most similar documents to the user's query
  let nearest_5 = document_table .select_nearest_embedding(user_question_embedding, 5)
 .await .unwrap();    // Display the results in a formatted table with colors based on the distance from the query
  let mut table = Table::new(); table.set_header(vec!["Score", "Value"]);    for result in nearest_5 {  let mut row = Row::new();  let color = if result.distance < 0.25 {
 Color::Green } else if result.distance < 0.75 {
 Color::Yellow } else {
 Color::Red }; row.add_cell(Cell::new(result.distance).fg(color))
 .add_cell(Cell::new(result.record.body()[0..50].to_string() + "..."));
 table.add_row(row);
 }   println!("{}", table); } 
```

RAG（检索增强生成）是生成包含最新或专有信息的文本的强大工具。检索增强生成通常遵循以下步骤：

1.  从一些本地文件、数据库或网络数据中收集上下文。在kalosm中，您可以从任何实现 [`IntoDocument`](https://docs.rs/kalosm/0.2.0/kalosm/language/trait.IntoDocument.html) 或 [`IntoDocuments`](https://docs.rs/kalosm/0.2.0/kalosm/language/trait.IntoDocuments.html) 的源中检索数据。您可以从 [本地文档](https://docs.rs/kalosm/0.2.0/kalosm/language/struct.DocumentFolder.html)、[搜索术语](https://docs.rs/kalosm/0.2.0/kalosm/language/struct.SearchQuery.html)、[特定网页](https://docs.rs/kalosm/0.2.0/kalosm/language/struct.Url.html)、[RSS feed](https://docs.rs/kalosm/0.2.0/kalosm/language/struct.RssFeed.html) 或甚至 [自定义网络爬虫](https://docs.rs/kalosm/0.2.0/kalosm/language/enum.Page.html#method.crawl) 收集您的数据源。

1.  将上下文插入可搜索的数据库中。Kalosm包括一个 [向量数据库](https://docs.rs/kalosm/0.2.0/kalosm/language/struct.VectorDB.html)，可用于存储和检索类似文档。

向量数据库使用基于文本片段（通常小于整个文档）生成向量的嵌入模型。然后将向量存储在数据库中。当您想要检索类似文档时，可以嵌入一个查询并在数据库中搜索相似的向量。

这些向量代表文本的含义，因此您可以基于文本的含义而不仅仅是使用的单词搜索类似的文档。

1.  使用上下文生成文本。您可以找到与LLM生成的问题或搜索类似的文本，然后基于上下文生成响应。

在这个版本中，我们对RAG进行了几项改进！（[#126](https://github.com/floneum/floneum/pull/126))

当您将文档插入向量数据库时，需要将其拆分为较小的片段，然后将文本嵌入其中。您选择的这些片段可能会对从向量数据库获得的结果的性能产生重大影响。在此版本中，我们为向量数据库添加了两种新的分块策略：

与基于文档内容生成嵌入相反，这种分块策略基于关于文档生成的假设问题生成嵌入。当构建需要查找与问题相关的上下文的聊天机器人时，这非常有用。

例如，如果你有一份关于美国历史的文档，你可以生成假设问题，比如“美国的首都是什么？”和“美国的第一任总统是谁？”然后基于这些问题生成嵌入。

然后，如果您用问题“美国的领导者是谁？”查询向量数据库，您可以找到关于美国历史的文档。

> 注意到问题“美国的领导者是谁？”与假设问题中的许多单词不同，但它确实传达了类似的意思，因此向量数据库仍然可以找到相关的文档。

这种分块策略生成基于文档摘要的嵌入。当你有一份大型文档并且想要根据文档的主要要点找到类似的文档时，这是很有用的。

基于文档摘要生成嵌入可以创建更好的嵌入，其中包含比仅包含文档的一个小片段更多的信息。

除了新的分块策略，我们还增加了对增量索引的支持。这意味着您可以向向量数据库添加新文档，而无需重新创建整个数据库。当您拥有大型数据库或者希望向LLM提供不断更新的上下文时，这非常有用。

Kalosm的向量数据库现在由[arroy](https://github.com/meilisearch/arroy)支持，这是一种空间高效且增量索引的向量数据库，由MeiliSearch支持！

Llama 实现已重写并优化，以获得更好的性能和模块化。 新实现现在比以前的版本快 7-25%。 在未来的发布中，我们计划添加对模型微调和为现有模型训练新头部的支持。 ([#122](https://github.com/floneum/floneum/pull/122))

类似 Llama 和 Phi 的语言模型会为词汇表中的每个令牌输出概率。 要生成文本，您需要从概率分布中进行抽样。 从概率分布中抽样可能会很慢，特别是在词汇量大时。 Kalosm 0.2 使用了在[llm-samplers](https://github.com/KerfuffleV2/llm-samplers/pull/9) 中引入的优化，只对前 512 个令牌进行抽样。 这种优化可以使抽样速度提高多达 2 倍。 ([#123](https://github.com/floneum/floneum/pull/123))

在结构化生成的约束内静态的大段文本现在被一次性加载，这可以显著加快处理速度。

例如，如果您有以下约束条件：

```
let constraints = RegexParser::new(r#"(The title of the book is [a-z]{2,15}\n)*"#).unwrap(); 
```

文本“书名为“将一次性加载而不是逐个令牌加载。 在受限生成中恢复了批量加载。 ([#131](https://github.com/floneum/floneum/pull/131))

Kalosm 0.2 添加了对几个新模型的支持，包括：

+   **海豚 Phi v2** 一个微小的聊天模型

+   **太阳-11b 模型** 一组用于聊天、文本和代码生成的模型

+   **微型 Llama 1.0** 一组微小的聊天和文本生成模型

有关 v0.1.0 和 v0.2.0 之间更改的详细列表，请参阅[完整的更改日志](https://github.com/floneum/floneum/compare/v0.2.0...v0.2.0-kalosm)。

希望您喜欢使用 Kalosm v0.2.0！ 您的反馈对我们非常宝贵，所以请不要犹豫分享您的想法并报告您遇到的任何问题。

在下一个版本中，我们计划添加对模型微调和为现有模型训练新头部的支持。 我们还计划继续改进语言模型的性能，并增加对更多模型的支持。

如果其中任何功能听起来有趣，或者您想提出一个新功能，请考虑在[Github](https://github.com/floneum/floneum/tree/main/interfaces/kalosm)上进行贡献。

如果您有兴趣使用 Kalosm 构建应用程序，请[加入 Discord](https://discord.gg/dQdmhuB8q5) 并参与社区！
