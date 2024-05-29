<!--yml
category: 未分类
date: 2024-05-29 13:22:23
-->

# Gemma, Ollama and LangChainGo - Eli Bendersky's website

> 来源：[https://eli.thegreenplace.net/2024/gemma-ollama-and-langchaingo/](https://eli.thegreenplace.net/2024/gemma-ollama-and-langchaingo/)

Yesterday Google released Gemma - an open LLM that folks can run locally on their machines (similarly to `llama2`). I was wondering how easy it would be to run Gemma on my computer, chat with it and interact with it from a Go program.

Turns it - thanks to [Ollama](https://ollama.com/download) - it's extremely easy! Gemma was already [added to Ollama](https://ollama.com/library/gemma), so all one has to do is run:

And wait for a few minutes while the model downloads. From this point on, my previous post about [using Ollama locally in Go](https://eli.thegreenplace.net/2023/using-ollama-with-langchaingo/) applies with pretty much no changes. Gemma becomes available through a REST API locally, and can be accessed from ollama-aware libraries like [LangChainGo](https://github.com/tmc/langchaingo).

I went ahead and added a `--model` flag to all my [code samples from that post](https://github.com/eliben/code-for-blog/tree/main/2023/ollama-go-langchain), and they can all run with `--model gemma` now. It all just works, due to the magic of standard interfaces:

*   Gemma is packaged in a standard interface for inclusion in Ollama
*   Ollama then presents a standardized REST API for this model, just like it does for other compatible models
*   LangChainGo has an Ollama provider that lets us write code to interact with any model running through Ollama

So we can write code like:

```
package  main import  ( "context" "flag" "fmt" "log" "github.com/tmc/langchaingo/llms" "github.com/tmc/langchaingo/llms/ollama" ) func  main()  { modelName  :=  flag.String("model",  "",  "ollama model name") flag.Parse() llm,  err  :=  ollama.New(ollama.WithModel(*modelName)) if  err  !=  nil  { log.Fatal(err) } query  :=  flag.Args()[0] ctx  :=  context.Background() completion,  err  :=  llms.GenerateFromSinglePrompt(ctx,  llm,  query) if  err  !=  nil  { log.Fatal(err) } fmt.Println("Response:\n",  completion) } 
```

And then run it as follows:

```
$ go run ollama-completion-arg.go --model gemma "what should be added to 91 to make -20?"
Response:
 The answer is -111.

91 + (-111) = -20 
```

Gemma seems relatively fast for a model running on a CPU. I find that the default 7B model, while much more capable than the default 7B llama2 based on published benchmarks - also runs about 30% faster on my machine.

## Without LangChainGo

While LangChainGo offers a conveneint API that's standardized across LLM providers, its use is by no means required for this sample. Ollama itself has a [Go API](https://pkg.go.dev/github.com/jmorganca/ollama/api) as part of its structure and it can be used externally as well. Here's an equivalent sample that doesn't require LangChainGo:

```
package  main import  ( "context" "flag" "fmt" "log" "github.com/jmorganca/ollama/api" ) func  main()  { modelName  :=  flag.String("model",  "",  "ollama model name") flag.Parse() client,  err  :=  api.ClientFromEnvironment() if  err  !=  nil  { log.Fatal(err) } req  :=  &api.GenerateRequest{ Model:  *modelName, Prompt:  flag.Args()[0], Stream:  new(bool),  // disable streaming } ctx  :=  context.Background() var  response  string respFunc  :=  func(resp  api.GenerateResponse)  error  { response  =  resp.Response return  nil } err  =  client.Generate(ctx,  req,  respFunc) if  err  !=  nil  { log.Fatal(err) } fmt.Println("Response:\n",  response) } 
```