<!--yml

分类：未分类

日期：2024-05-27 14:43:25

-->

# 如何在Go中编写HTTP服务已有13年 | Grafana Labs

> 来源：[https://grafana.com/blog/2024/02/09/how-i-write-http-services-in-go-after-13-years/](https://grafana.com/blog/2024/02/09/how-i-write-http-services-in-go-after-13-years/)

将近六年前，我写了一篇博文概述了[我如何在Go中编写HTTP服务](https://pace.dev/blog/2018/05/09/how-I-write-http-services-after-eight-years.html)，今天我再次来告诉你，我如何编写HTTP服务。

那篇原始文章有点火了，引发了一些很棒的讨论，这些讨论影响了我今天的做法。在主持了多年的[Go Time podcast](https://changelog.com/gotime)，在[Twitter](https://twitter.com/matryer)上讨论Go，并在多年来积累了更多维护这样的代码的经验后，我觉得是时候更新一下了。

（对于那些注意到Go并不完全13岁的学究，我开始写Go HTTP服务是在[版本 .r59](https://go.dev/doc/devel/pre_go1#r59)。）

这篇文章涵盖了与在Go中构建服务相关的各种主题，包括：

+   为了达到最大的可维护性，结构化服务器和处理程序。

+   优化快速启动和优雅关闭的技巧和窍门。

+   如何处理适用于许多类型请求的常见工作

+   深入探讨如何正确测试您的服务

从小项目到大项目，这些实践方法经过时间的考验，对我来说效果显著，我也希望对你们有所帮助。

## 这篇文章是为谁写的？

这篇文章适合你。它适合所有计划在Go中编写某种HTTP服务的人。如果你正在学习Go，你也可能会发现这篇文章有用，因为很多示例都遵循了良好的实践方法。有经验的gopher也可能会学到一些好的模式。

要想发现这篇文章最有用，你需要了解基本的Go知识。如果你觉得自己还没有达到这个水平，我非常推荐由Chris James编写的[用测试学习Go](https://quii.gitbook.io/learn-go-with-tests/)。如果你想听更多Chris的观点，还可以查看我们与Ben Johnson共同进行的Go Time节目，主题是[Go项目的文件和文件夹](https://changelog.com/gotime/278)。

如果你熟悉此前版本的这篇文章，这一节包含了现在有何不同的快速总结。如果你想从头开始阅读，请跳到下一节。

1.  我的处理程序过去是挂在服务器结构体上的方法，但我不再这样做了。如果一个处理函数需要一个依赖，它可以直截了当地通过参数来要求。不再有意外的依赖，当你只是想测试一个单一的处理程序时。

1.  我过去更喜欢`http.HandlerFunc`而不是`http.Handler` — 足够多的第三方库首先考虑`http.Handler`，这是一个很好的选择。`http.HandlerFunc`仍然非常有用，但现在大多数事情都被表示为接口类型。无论选择哪个方式，都没什么大的区别。

1.  我增加了更多关于测试的内容，包括一些“观点”。

1.  我添加了更多的部分，所以建议每个人都进行全面阅读。

## `NewServer` 构造函数

让我们首先看看任何 Go 服务的骨干：服务器。 `NewServer` 函数生成了主要的 `http.Handler`。通常我每个服务只有一个，我依靠 HTTP 路由将流量引导到每个服务内部正确的处理程序，因为：

+   `NewServer` 是一个大构造函数，它将所有依赖项作为参数接受

+   如果可能，它返回一个 `http.Handler`，对于更复杂的情况可以是一个专用类型

+   它通常配置自己的 muxer 并调用 `routes.go`

例如，你的代码可能类似于这样：

```
func NewServer(
	logger *Logger
	config *Config
	commentStore *commentStore
	anotherStore *anotherStore
) http.Handler {
	mux := http.NewServeMux()
	addRoutes(
		mux,
		Logger,
		Config,
		commentStore,
		anotherStore,
	)
	var handler http.Handler = mux
	handler = someMiddleware(handler)
	handler = someMiddleware2(handler)
	handler = someMiddleware3(handler)
	return handler
}
```

在不需要所有依赖的测试用例中，我将 `nil` 传入作为一个信号，表示它不会被使用。

`NewServer` 构造函数负责适用于所有端点的所有顶级 HTTP 事务，比如 CORS、认证中间件和日志记录：

```
var handler http.Handler = mux
handler = logging.NewLoggingMiddleware(logger, handler)
handler = logging.NewGoogleTraceIDMiddleware(logger, handler)
handler = checkAuthHeaders(handler)
return handler
```

设置服务器通常是通过使用 Go 的内置 `http` 包来暴露它：

```
srv := NewServer(
	logger,
	config,
	tenantsStore,
	slackLinkStore,
	msteamsLinkStore,
	proxy,
)
httpServer := &http.Server{
	Addr:    net.JoinHostPort(config.Host, config.Port),
	Handler: srv,
}
go func() {
	log.Printf("listening on %s\n", httpServer.Addr)
	if err := httpServer.ListenAndServe(); err != nil && err != http.ErrServerClosed {
		fmt.Fprintf(os.Stderr, "error listening and serving: %s\n", err)
	}
}()
var wg sync.WaitGroup
wg.Add(1)
go func() {
	defer wg.Done()
	<-ctx.Done()
	// make a new context for the Shutdown (thanks Alessandro Rosetti)
	shutdownCtx := context.Background()
	shutdownCtx, cancel := context.WithTimeout(ctx, 10 * time.Second)
	defer cancel()
	if err := httpServer.Shutdown(shutdownCtx); err != nil {
		fmt.Fprintf(os.Stderr, "error shutting down http server: %s\n", err)
	}
}()
wg.Wait()
return nil
```

### 长参数列表

必须有一个限制，这时它停止做正确的事情，但大多数时间我很高兴添加依赖项列表作为参数。虽然有时它们会变得很长，但我发现它仍然是值得的。

是的，它使我免于创建一个结构体，但真正的好处是我从参数中得到了稍微更多的类型安全。我可以制作一个跳过任何我不喜欢的字段的结构体，但一个函数迫使我按我的需求去查找字段。我必须查找字段才能知道如何在结构体中设置它们，而我如果不传递正确的参数就无法调用一个函数。

如果你将它格式化为垂直列表，像我在现代前端代码中看到的那样，它并不坏：

```
srv := NewServer(
	logger,
	config,
	tenantsStore,
	commentsStore,
	conversationService,
	chatGPTService,
)
```

## 在 `routes.go` 中映射整个 API 表面

这个文件是你的服务中所有路由都列出的地方。

有时你不能帮助但有些东西散落在周围，但能够去一个文件中看到它的 API 表面非常有帮助。

因为在 `NewServer` 构造函数中有大量依赖参数列表，所以你通常会在你的路由函数中得到相同的列表。但这并不坏。再说一遍，多亏了 Go 的类型检查，你很快就会知道是否忘记了某些内容或者顺序有误。

```
func addRoutes(
	mux                 *http.ServeMux,
	logger              *logging.Logger,
	config              Config,
	tenantsStore        *TenantsStore,
	commentsStore       *CommentsStore,
	conversationService *ConversationService,
	chatGPTService      *ChatGPTService,
	authProxy           *authProxy
) {
	mux.Handle("/api/v1/", handleTenantsGet(logger, tenantsStore))
	mux.Handle("/oauth2/", handleOAuth2Proxy(logger, authProxy))
	mux.HandleFunc("/healthz", handleHealthzPlease(logger))
	mux.Handle("/", http.NotFoundHandler())
}
```

在我的例子中，`addRoutes` 并不会返回错误。任何可能抛出错误的操作都移到了 `run` 函数中，在到达这一点之前进行了整理，使得这个函数可以保持简单和平坦。当然，如果你的任何处理程序由于任何原因返回错误，那么可以返回错误。

## `func main()` 只调用 `run()`

`run` 函数类似于 `main` 函数，不同之处在于它将操作系统的基本功能作为参数，并返回，你猜对了，一个错误。

我希望 `func main()` 可以是 `func main() error`。或者像在 C 语言中那样，可以返回退出码： `func main() int`。通过拥有一个超级简单的 `main` 函数，你也可以实现你的梦想：

```
func run(ctx context.Context, w io.Writer, args []string) error {
	ctx, cancel := signal.NotifyContext(ctx, os.Interrupt)
	defer cancel()

	// ...
}

func main() {
	ctx := context.Background()
	if err := run(ctx, os.Stdout, os.Args); err != nil {
		fmt.Fprintf(os.Stderr, "%s\n", err)
		os.Exit(1)
	}
}
```

> 编辑：我曾经在 `main` 函数中执行 `signal.NotifyContext` 的一部分，但是戴夫·亨德森（和其他几个人）指出 `cancel` 将不会被调用，因此我已将其移动到 `run` 函数中。

上面的代码直接调用了 `run` 函数，该函数创建一个上下文，可以通过 `Ctrl+C` 或等效方式取消。如果 `run` 返回 `nil`，函数会正常退出。如果返回错误，我们会将其写入到 stderr 并以非零代码退出。如果我在写一个命令行工具，退出码很重要，我也会返回一个整数，这样我就可以编写测试来验证正确的退出码。

操作系统基础知识被作为参数传递给了 `run` 函数。例如，您可以传递 `os.Args` 如果它支持标志，甚至 `os.Stdin`、`os.Stdout`、`os.Stderr` 依赖项。这使得您的程序更容易进行测试，因为测试代码可以调用 `run` 来执行您的程序，通过传递不同的参数来控制参数和所有流。

下表显示了运行函数的输入参数示例：

| **值** | **类型** | **描述** |
| --- | --- | --- |
| `os.Args` | `[]string` | 在执行程序时传递的参数。也用于解析标志。 |
| `os.Stdin` | `io.Reader` | 用于读取输入 |
| `os.Stdout` | `io.Writer` | 用于写入输出 |
| `os.Stderr` | `io.Writer` | 用于写入错误日志 |
| `os.Getenv` | `func(string) string` | 用于读取环境变量 |
| `os.Getwd` | `func() (string, error)` | 获取工作目录 |

如果您远离任何全局范围的数据，通常可以在更多地方使用 `t.Parallel()` 来加速您的测试套件。一切都是自包含的，因此多次调用 `run` 不会相互干扰。

我经常会得到像这样的 `run` 函数签名：

```
func run(
	ctx    context.Context,
	args   []string,
	getenv func(string) string,
	stdin  io.Reader,
	stdout, stderr io.Writer,
) error
```

现在我们进入 `run` 函数内部，可以回到编写正常的 Go 代码，可以像没事人一样地返回错误。我们的 gopher 们真的很喜欢返回错误，早点承认这一点，网上那些人就能赢了，然后走开。

### 优雅地关闭

如果您正在运行大量的测试，程序在每个测试完成时停止是很重要的。（或者，您可能决定保持一个实例运行所有测试，但这取决于您。）

上下文会被传递。如果终止信号进入程序，它将被取消，因此在每个层级尊重它非常重要。至少，将其传递给您的依赖项。最好，在任何长时间运行或循环代码中检查 `Err()` 方法，如果返回错误，停止正在进行的操作并将其返回。这将有助于服务器优雅地关闭。如果启动了其他 goroutine，也可以使用上下文来决定是否该停止它们。

### 控制环境

`args` 和 `getenv` 参数为我们提供了通过标志和环境变量控制程序行为的几种方式。通过使用 args 处理标志（只要你不使用全局空间版本的 flags，而是在 `run` 内部使用 `flags.NewFlagSet`），因此我们可以使用不同的值调用 run：

```
args := []string{
	"myapp",
	"--out", outFile,
	"--fmt", "markdown",
}
go run(ctx, args, etc.)
```

如果您的程序使用环境变量而不是标志（或者两者都使用），那么 `getenv` 函数允许您插入不同的值而不更改实际环境。

```
getenv := func(key string) string {
	switch key {
	case "MYAPP_FORMAT":
		return "markdown"
	case "MYAPP_TIMEOUT":
		return "5s"
	default:
		return ""
}
go run(ctx, args, getenv)
```

对我来说，使用这种 `getenv` 技术比使用 `t.SetEnv` 控制环境变量更胜一筹，因为您可以通过调用 `t.Parallel()` 持续并行运行您的测试，而 `t.SetEnv` 不允许。

写命令行工具时，这种技术甚至更加有用，因为您经常希望以不同的设置运行程序以测试其所有行为。

在 `main` 函数中，我们可以传入真实的东西：

```
func main() {
	ctx := context.Background()
	if err := run(ctx, os.Getenv, os.Stderr); err != nil {
		fmt.Fprintf(os.Stderr, "%s\n", err)
		os.Exit(1)
	}
}
```

## 制造函数返回处理程序

我的处理程序函数不直接实现 `http.Handler` 或 `http.HandlerFunc`，它们返回它们。具体来说，它们返回 `http.Handler` 类型。

```
// handleSomething handles one of those web requests
// that you hear so much about.
func handleSomething(logger *Logger) http.Handler {
	thing := prepareThing()
	return http.HandlerFunc(
		func(w http.ResponseWriter, r *http.Request) {
			// use thing to handle request
			logger.Info(r.Context(), "msg", "handleSomething")
		}
	)
}
```

这种模式为每个处理程序提供了其自己的闭包环境。您可以在此空间中进行初始化工作，并且在调用处理程序时数据将可用。

确保只读取共享数据。如果处理程序修改任何内容，您将需要互斥锁或其他保护机制。

在这里存储程序状态通常不是您想要的。在大多数云环境中，您不能信任代码会长时间运行。根据您的生产环境，服务器通常会关闭以节省资源，或者仅仅因其他原因而崩溃。可能还会有许多实例在负载均衡的情况下运行，请求会以不可预测的方式分布。在这种情况下，一个实例只能访问自己的本地数据。因此，最好使用数据库或其他存储 API 在真实项目中持久化数据。

## 在一个地方处理解码/编码

每个服务都需要解码请求体并编码响应体。这是一个经得起时间考验的合理抽象。

我通常有一对名为 encode 和 decode 的辅助函数。使用泛型的示例版本向您展示，您实际上只是包装了几行基本代码，我通常不会这样做，但是在需要为所有 API 更改这里时，这变得非常有用。（例如，假设您的新老板困在20世纪90年代，他们想要添加 XML 支持。）

```
func encode[T any](w http.ResponseWriter, r *http.Request, status int, v T) error {
	w.Header().Set("Content-Type", "application/json")
	w.WriteHeader(status)
	if err := json.NewEncoder(w).Encode(v); err != nil {
		return fmt.Errorf("encode json: %w", err)
	}
	return nil
}

func decode[T any](r *http.Request) (T, error) {
	var v T
	if err := json.NewDecoder(r.Body).Decode(&v); err != nil {
		return v, fmt.Errorf("decode json: %w", err)
	}
	return v, nil
}
```

有趣的是，编译器能够从参数推断出类型，因此在调用 encode 时不需要传递它：

```
err := encode(w, r, http.StatusOK, obj)
```

但由于它是 decode 的返回参数，你需要指定你期望的类型：

```
decoded, err := decode[CreateSomethingRequest](r)
```

我试图不要过度使用这些函数，但过去我对一个简单的验证接口非常满意，它很好地契合了 decode 函数。

## 验证数据

我喜欢简单的接口。实际上非常喜欢。单方法接口很容易实现。所以当涉及到验证对象时，我喜欢这样做：

```
// Validator is an object that can be validated.
type Validator interface {
	// Valid checks the object and returns any
	// problems. If len(problems) == 0 then
	// the object is valid.
	Valid(ctx context.Context) (problems map[string]string)
}
```

`Valid`方法接受一个上下文（可选，但过去对我有用），并返回一个映射。如果字段有问题，其名称用作键，并设置问题的人类可读说明作为值。

该方法可以执行任何必要的操作来验证结构体的字段。例如，它可以检查确保：

+   必填字段不为空。

+   具有特定格式（如电子邮件）的字符串是正确的。

+   数字在可接受范围内。

如果您需要做更复杂的事情，比如在数据库中检查字段，那应该在其他地方进行；这可能太重要了，不能被视为快速验证检查，你不会希望在这样的函数中找到这种事情，因此它很容易被隐藏起来。

然后我使用类型断言来查看对象是否实现了该接口。或者在泛型世界中，我可能选择通过更改解码方法来明确说明正在发生的事情。

```
func decodeValid[T Validator](r *http.Request) (T, map[string]string, error) {
	var v T
	if err := json.NewDecoder(r.Body).Decode(&v); err != nil {
		return v, nil, fmt.Errorf("decode json: %w", err)
	}
	if problems := v.Valid(r.Context()); len(problems) > 0 {
		return v, problems, fmt.Errorf("invalid %T: %d problems", v, len(problems))
	}
	return v, nil, nil
}
```

在此代码中，`T`必须实现`Validator`接口，并且`Valid`方法必须返回零问题，以便将对象视为成功解码。

安全地返回`nil`以表示问题，因为我们将检查`len(problems)`，对于`nil`映射，它将为`0`，但不会发生恐慌。

## 中间件的适配器模式

中间件函数接受一个`http.Handler`并返回一个可以在调用原始处理程序之前和/或之后运行代码的新处理程序 — 或者它可以决定根本不调用原始处理程序。

一个示例是检查用户是否为管理员的检查：

```
func adminOnly(h http.Handler) http.Handler {
	return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		if !currentUser(r).IsAdmin {
			http.NotFound(w, r)
			return
		}
		h(w, r)
	})
}
```

处理程序内部的逻辑可以选择性地决定是否调用原始处理程序。在上面的示例中，如果`IsAdmin`为false，则处理程序将返回`HTTP 404 Not Found`并返回（或中止）；注意没有调用`h`处理程序。如果`IsAdmin`为true，则允许用户访问路由，因此执行权传递给`h`处理程序。

通常我在`routes.go`文件中列出中间件：

```
package app

func addRoutes(mux *http.ServeMux) {
	mux.HandleFunc("/api/", handleAPI())
	mux.HandleFunc("/about", handleAbout())
	mux.HandleFunc("/", handleIndex())
	mux.HandleFunc("/admin", adminOnly(handleAdminIndex()))
}
```

通过查看端点映射，很清楚地显示了哪些中间件应用于哪些路由。如果列表开始变得更长，请尝试跨多行拆分它们 — 我知道，我知道，但你会习惯的。

## 有时我返回中间件

以上方法适用于简单情况，但如果中间件需要大量依赖项（记录器、数据库、一些API客户端、包含“永远不会放弃你”的数据的字节数组以供稍后恶作剧使用），那么我已知道有一个返回中间件函数的函数。

问题在于，最终你会得到这样的代码：

```
mux.Handle("/route1", middleware(logger, db, slackClient, rroll []byte, handleSomething(handlerSpecificDeps))
mux.Handle("/route2", middleware(logger, db, slackClient, rroll []byte, handleSomething2(handlerSpecificDeps))
mux.Handle("/route3", middleware(logger, db, slackClient, rroll []byte, handleSomething3(handlerSpecificDeps))
mux.Handle("/route4", middleware(logger, db, slackClient, rroll []byte, handleSomething4(handlerSpecificDeps))
```

这会使代码膨胀，并且实际上并没有提供任何有用的东西。相反，我会让中间件函数接受依赖项，但只返回一个接下来的处理程序函数。

```
func newMiddleware(
	logger Logger,
	db *DB,
	slackClient *slack.Client,
	rroll []byte,
) func(h http.Handler) http.Handler
```

返回类型`func(h http.Handler) http.Handler`是我们设置路由时将调用的函数。

```
middleware := newMiddleware(logger, db, slackClient, rroll)
mux.Handle("/route1", middleware(handleSomething(handlerSpecificDeps))
mux.Handle("/route2", middleware(handleSomething2(handlerSpecificDeps))
mux.Handle("/route3", middleware(handleSomething3(handlerSpecificDeps))
mux.Handle("/route4", middleware(handleSomething4(handlerSpecificDeps))
```

有些人，但不包括我，喜欢这样形式化函数类型：

```
// middleware is a function that wraps http.Handlers
// proving functionality before and after execution
// of the h handler.
type middleware func(h http.Handler) http.Handler
```

这很好。如果你喜欢，可以这样做。我不会到处转悠等着你，然后一副令人害怕的姿态跟在你身边，搭着你的肩膀，问你自己满意吗。

我不这么做的原因是因为它增加了额外的间接性。当你看上面的`newMiddleware`函数签名时，很清楚发生了什么。如果返回类型是中间件，你需要做一些额外的工作。基本上，我优化于阅读代码，而不是写代码。

### 隐藏请求/响应类型的机会

如果一个端点有它自己的请求和响应类型，通常它们只对特定的处理程序有用。

如果是这样，你可以在函数内定义它们。

```
func handleSomething() http.HandlerFunc {
	type request struct {
		Name string
	}
	type response struct {
		Greeting string `json:"greeting"`
	}
	return func(w http.ResponseWriter, r *http.Request) {
		...
	}
}
```

这保持了全局空间的清晰，并且防止其他处理程序依赖于你可能不考虑稳定的数据。

当你的测试代码需要使用相同类型时，有时会遇到这种方法的摩擦。公平地说，这是打破它们的一个很好的理由。

## 在测试中使用内联请求/响应类型来进行额外的故事讲述

如果您的请求/响应类型隐藏在处理程序内部，您只需在测试代码中声明新类型。

这是一个向未来需要理解你的代码的人讲述一些故事的机会。

例如，假设我们的代码中有一个`Person`类型，并且我们在许多端点上重复使用它。如果我们有一个`/greet`端点，我们可能只关心他们的名字，所以我们可以在测试代码中表达这一点：

```
func TestGreet(t *testing.T) {
	is := is.New(t)
	person := struct {
		Name string `json:"name"`
	}{
		Name: "Mat Ryer",
	}
	var buf bytes.Buffer
	err := json.NewEncoder(&buf).Encode(person)
	is.NoErr(err) // json.NewEncoder
	req, err := http.NewRequest(http.MethodPost, "/greet", &buf)
	is.NoErr(err)
	//... more test code here
```

这个测试清楚地表明，我们唯一关心的是`Name`字段。

## `sync.Once`来推迟设置

如果在准备处理程序时需要执行任何昂贵的工作，我会推迟到首次调用该处理程序时。

这提高了应用程序的启动时间。

```
func handleTemplate(files string...) http.HandlerFunc {
	var (
		init    sync.Once
		tpl     *template.Template
		tplerr  error
	)
	return func(w http.ResponseWriter, r *http.Request) {
		init.Do(func(){
			tpl, tplerr = template.ParseFiles(files...)
		})
		if tplerr != nil {
			http.Error(w, tplerr.Error(), http.StatusInternalServerError)
			return
		}
		// use tpl
	}
}
```

`sync.Once`确保代码只执行一次，并且其他调用（其他人发出相同请求）将阻塞，直到它完成。

+   错误检查在`init`函数之外，因此如果出现问题，我们仍然会显示错误，并且不会在日志中丢失它

+   如果处理程序未被调用，则昂贵的工作永远不会完成——这可能会根据代码部署方式带来很大的好处

请记住，通过这样做，您将初始化时间从启动移到运行时（当首次访问端点时）。我经常使用Google App Engine，所以这对我来说是有意义的，但您的情况可能不同，因此值得考虑何时以及何地以这种方式使用`sync.Once`。

## 设计用于可测试性

这些模式的演变部分是因为测试代码的简易性。`run` 函数是从测试代码中直接运行程序的简单方法。

在 Go 中进行测试时，您有很多选择，这与对错无关，更多地是关于：

+   通过查看测试，了解程序的功能有多容易？

+   在不担心破坏事物的情况下更改代码有多容易？

+   如果所有测试都通过，您可以推送到生产环境，还是需要覆盖更多内容？

### 单元测试时的单元是什么？

遵循这些模式，处理程序本身也可以独立进行测试，但我倾向于不这样做，我将在下面解释原因。您必须考虑对于您的项目来说最佳的方法是什么。

要仅测试处理程序，您可以：

1.  调用函数以获取 `http.Handler` — 您必须传入所有必需的依赖项（这是一个特性）。

1.  在返回的 `http.Handler` 上调用 `ServeHTTP` 方法，使用真实的 `http.Request` 和 `httptest` 包中的 `ResponseRecorder`（参见 [https://pkg.go.dev/net/http/httptest#ResponseRecorder](https://pkg.go.dev/net/http/httptest#ResponseRecorder)）

1.  对响应进行断言（检查状态码，解码主体并确保正确，检查任何重要的标头等）。

如果这样做，您可以剔除任何像身份验证这样的中间件，直接进入处理程序代码。如果有一些特定的复杂性需要构建一些测试支持，这很好。然而，当您的测试代码以与用户相同的方式调用 API 时，就会有一个优势。在这个层面上，我更倾向于进行端到端测试，而不是在内部对所有部分进行单元测试。

我更愿意调用 `run` 函数来尽可能接近在生产环境中运行程序。这将解析任何参数，连接到任何依赖项，迁移数据库，以及在实际环境中执行的其他操作，并最终启动服务器。然后当我从测试代码中调用 API 时，我会经过所有层，并与真实数据库进行交互。我还同时测试 `routes.go`。

我发现通过这种方法可以更早地发现更多问题，并且可以避免专门测试样板内容。这也减少了我的测试中的重复。如果我勤奋地测试每一层，最终可能会以稍微不同的方式多次重复相同的内容。您必须维护所有这些内容，因此如果要更改某些内容，则更新一个函数和三个测试并不会感到非常高效。通过端到端测试，您只需拥有一组描述用户与系统之间交互的主要测试。

在适当的情况下，我仍然在其中使用单元测试。如果我使用 TDD（我经常这样做），那么通常已经完成了很多测试，我很乐意维护。但是，如果重复了与端到端测试相同的内容，我会回去删除测试。

这个决定将取决于很多因素，从身边人的意见到项目的复杂性，所以像本文中所有建议一样，如果对你来说行不通，不要勉强去做这件事。

### 使用`run`函数进行测试

我喜欢从每个测试中调用`run`函数。每个测试都会得到自己独立的程序实例。对于每个测试，我可以传递不同的参数、标志值、标准输入和输出管道，甚至环境变量。

因为`run`函数接受`context.Context`，而且我们所有的代码都尊重上下文（对吧，大家？它尊重上下文，对吧？）我们可以通过调用`context.WithCancel`得到一个取消函数。通过延迟`cancel`函数，当测试函数返回（即测试完成运行时），上下文将被取消，程序将优雅地关闭。在Go 1.14中，他们添加了`t.Cleanup`方法，它是对自己使用`defer`关键字的替代，如果你想了解更多原因，可以查看这个问题：[https://github.com/golang/go/issues/37333](https://github.com/golang/go/issues/37333)。

所有这些都在惊人少量的代码中实现。当然，你也必须随时检查`ctx.Err`或`ctx.Done`：

```
func Test(t *testing.T) {
	ctx := context.Background()
	ctx, cancel := context.WithCancel(ctx)
	t.Cleanup(cancel)
	go run(ctx)
	// test code goes here
```

### 等待准备就绪

因为`run`函数在一个goroutine中执行，我们实际上不知道它何时启动。如果我们要像真实用户一样开始使用API，我们需要知道它何时准备就绪。

我们可以设置某种方式来表示就绪，比如一个通道或其他东西 —— 但我更喜欢在服务器上运行`/healthz`或`/readyz`端点。正如我年迈的祖母过去常说的，布丁的真正实力在于实际的HTTP请求（她的时代先见之明）。

这是一个例子，我们努力使代码更具可测试性，为我们提供了了解用户需求的洞察力。他们可能也想知道服务是否就绪，那么为什么不有一个官方的方式来了解这一点呢？

要等待服务准备就绪，你可以简单地写一个循环：

```
// waitForReady calls the specified endpoint until it gets a 200 
// response or until the context is cancelled or the timeout is 
// reached.
func waitForReady(
	ctx context.Context, 
	timeout time.Duration, 
	endpoint string,
) error {
	client := http.Client{}
	startTime := time.Now()
	for {
		req, err := http.NewRequestWithContext(
			ctx, 
			http.MethodGet, 
			endpoint, 
			nil,
		)
		if err != nil {
			return fmt.Errorf("failed to create request: %w", err)
		}

		resp, err := client.Do(req)
		if err != nil {
			fmt.Printf("Error making request: %s\n", err.Error())
			continue
		}
		if resp.StatusCode == http.StatusOK {
			fmt.Println("Endpoint is ready!")
			resp.Body.Close()
			return nil
		}
		resp.Body.Close()

		select {
		case <-ctx.Done():
			return ctx.Err()
		default:
			if time.Since(startTime) >= timeout {
				return fmt.Errorf("timeout reached while waiting for endpoint")
			}
			// wait a little while between checks
			time.Sleep(250 * time.Millisecond)
		}
	}
}
```

## 将所有这些付诸实践

使用这些技巧编写简单的API仍然是我最喜欢的方式。它符合我追求的代码易读、易扩展（通过复制模式）、易于新人使用、易于无需担忧地修改，并且明确地没有任何魔法。即使我使用像我们自己的[Oto包](https://github.com/pacedotdev/oto)这样的代码生成框架，根据我定制的模板为我编写样板，这仍然是正确的。

在更大的项目或较大的组织中，特别是像Grafana Labs这样的组织，你经常会遇到影响决策的具体技术选择。gRPC就是一个很好的例子。在有已建立的模式和经验，或者其他广泛使用的工具或抽象的情况下，你经常会发现自己做出顺其自然的选择，就像他们说的那样，尽管我怀疑（还是希望？）这篇文章对你来说仍然有用。

我的日常工作是与Grafana Labs内的一组才华横溢的人员共同打造新的[Grafana IRM](/products/cloud/irm/)套件。本文讨论的模式帮助我们提供可靠的工具。“告诉我更多关于这些伟大工具的信息！”，我听见你在电脑显示器前尖叫。

大多数人使用Grafana来可视化其系统的运行情况，并通过Grafana警报功能在指标超出可接受范围时发出警报。使用Grafana OnCall，您的计划和升级规则会自动化联系适当的人员在事情出错时联系。

Grafana事故处理功能让你管理那些不可避免的紧急情况，我们大多数人都太熟悉了。它为你创建Zoom会议室来讨论问题，一个专用的Slack频道，并跟踪事件时间轴，让你专注于解决问题。在Slack中，你标记为机器人面部表情符号的任何内容将被添加到时间轴中。这使得在进行事后总结或事故回顾讨论时非常容易收集关键事件。

*[Grafana Cloud](/products/cloud/?pg=blog&plcmt=body-txt)是开始使用度量、日志、跟踪、仪表板等功能最简单的方式。我们有一个慷慨的永久免费层和适用于各种用例的计划。[立即免费注册](/auth/sign-up/create-user/?pg=blog&plcmt=body-txt)!*
