<!--yml

category: 未分类

date: 2024-05-29 13:22:23

-->

# Gemma，Ollama和LangChainGo - Eli Bendersky的网站

> 来源：[https://eli.thegreenplace.net/2024/gemma-ollama-and-langchaingo/](https://eli.thegreenplace.net/2024/gemma-ollama-and-langchaingo/)

昨天，Google发布了Gemma - 一个开放的LLM，用户可以在本地机器上运行（类似于`llama2`）。我想知道在我的计算机上运行Gemma，与之聊天并与之交互有多容易。

转而感谢[Ollama](https://ollama.com/download) - 这非常容易！Gemma已经被添加到了Ollama，因此所有人只需运行：

等待几分钟，模型下载完成。从这一点开始，我之前关于[在Go中本地使用Ollama](https://eli.thegreenplace.net/2023/using-ollama-with-langchaingo/)的帖子几乎不需要更改，Gemma通过本地的REST API可用，并且可以通过像[LangChainGo](https://github.com/tmc/langchaingo)这样的ollama感知库访问。

我已经为我所有的[来自该帖子的代码示例](https://github.com/eliben/code-for-blog/tree/main/2023/ollama-go-langchain)添加了一个`--model`标志，并且它们现在都可以使用`--model gemma`运行。这一切都很顺利，这要归功于标准接口的神奇：

+   Gemma是一个标准接口包装，用于在Ollama中进行包含。

+   然后，Ollama为该模型提供了标准化的REST API，就像它为其他兼容模型一样。

+   LangChainGo具有一个Ollama提供程序，让我们可以编写代码与通过Ollama运行的任何模型进行交互

因此，我们可以编写如下代码：

```
package  main import  ( "context" "flag" "fmt" "log" "github.com/tmc/langchaingo/llms" "github.com/tmc/langchaingo/llms/ollama" ) func  main()  { modelName  :=  flag.String("model",  "",  "ollama model name") flag.Parse() llm,  err  :=  ollama.New(ollama.WithModel(*modelName)) if  err  !=  nil  { log.Fatal(err) } query  :=  flag.Args()[0] ctx  :=  context.Background() completion,  err  :=  llms.GenerateFromSinglePrompt(ctx,  llm,  query) if  err  !=  nil  { log.Fatal(err) } fmt.Println("Response:\n",  completion) } 
```

然后按以下方式运行它：

```
$ go run ollama-completion-arg.go --model gemma "what should be added to 91 to make -20?"
Response:
 The answer is -111.

91 + (-111) = -20 
```

对于在CPU上运行的模型而言，Gemma似乎相对快速。我发现默认的7B模型，尽管在已发布的基准测试中比基于llama2的默认7B模型更强大，但在我的机器上也能运行快约30%。

## 没有LangChainGo

虽然LangChainGo提供了一个跨LLM提供商标准化的便捷API，但并不是必须使用本示例。Ollama本身作为其结构的一部分具有一个[Go API](https://pkg.go.dev/github.com/jmorganca/ollama/api)，也可以外部使用。以下是一个不需要LangChainGo的等效示例：

```
package  main import  ( "context" "flag" "fmt" "log" "github.com/jmorganca/ollama/api" ) func  main()  { modelName  :=  flag.String("model",  "",  "ollama model name") flag.Parse() client,  err  :=  api.ClientFromEnvironment() if  err  !=  nil  { log.Fatal(err) } req  :=  &api.GenerateRequest{ Model:  *modelName, Prompt:  flag.Args()[0], Stream:  new(bool),  // disable streaming } ctx  :=  context.Background() var  response  string respFunc  :=  func(resp  api.GenerateResponse)  error  { response  =  resp.Response return  nil } err  =  client.Generate(ctx,  req,  respFunc) if  err  !=  nil  { log.Fatal(err) } fmt.Println("Response:\n",  response) } 
```
