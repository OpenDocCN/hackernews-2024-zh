<!--yml

category: 未分类

日期：2024年5月27日 14:55:52

-->

# 更强大的Go执行追踪 - Go编程语言

> 来源：[https://go.dev/blog/execution-traces-2024](https://go.dev/blog/execution-traces-2024)

# 更强大的Go执行追踪

Michael Knyszek

2024年3月14日

[runtime/trace](/pkg/runtime/trace)包含了一个强大的工具，用于理解和排查Go程序。其功能允许产生每个goroutine在一段时间内执行的追踪。通过[`go tool trace`命令](/pkg/cmd/trace)（或优秀的开源[gotraceui工具](https://gotraceui.dev/)），可以可视化和探索这些追踪数据。

追踪的魔力在于它可以轻松地揭示程序中难以以其他方式看到的事物。例如，一个并发瓶颈，在这个瓶颈中许多goroutine都在同一个通道上阻塞，可能在CPU分析中很难看到，因为没有执行可供抽样。但在执行追踪中，*缺少*执行将非常清晰地显示出来，而被阻塞goroutine的堆栈跟踪将迅速指出罪魁祸首。

Go开发者甚至能够使用[任务](/pkg/runtime/trace#Task)、[区域](/pkg/runtime/trace#WithRegion)和[日志](/pkg/runtime/trace#Log)来为其自己的程序进行仪器化，这些可以用来将高层次关注点与低层次执行细节相关联。

## 问题

不幸的是，执行追踪中的丰富信息往往难以触及。追踪历史上的四大问题经常妨碍其使用。

+   追踪有很高的开销。

+   追踪并没有很好地扩展，可能变得过于庞大以至于难以分析。

+   通常不清楚何时开始追踪以捕获特定的不良行为。

+   仅有最有冒险精神的gopher才能在缺乏用于解析和解释执行追踪的公共包的情况下进行程序化分析。

如果你在过去几年中使用过追踪功能，你可能已经对其中一个或多个问题感到沮丧。但是我们很高兴地宣布，在过去两个Go版本中，我们在所有这四个领域都取得了重大进展。

## 低开销追踪

在Go 1.21之前，对于许多应用程序而言，追踪的运行时开销大约在10-20%的CPU之间，这限制了追踪的使用，而不像CPU分析那样可以持续使用。事实证明，追踪的大部分成本归结于回溯。运行时产生的许多事件都附带有堆栈跟踪，这对于实际上识别关键时刻goroutine的执行情况非常宝贵。

由于Felix Geisendörfer和Nick Ripley在优化回溯效率方面的工作，执行追踪的运行时CPU开销大幅减少，对于许多应用程序而言仅为1-2%。关于这项工作的更多信息可以在[Felix的博客文章](https://blog.felixge.de/reducing-gos-execution-tracer-overhead-with-frame-pointer-unwinding/)中找到。

## 可扩展的追踪

跟踪格式及其事件的设计围绕着相对高效的发射，但需要工具来解析并保留跟踪的整体状态。几百兆字节的跟踪可能需要几个吉字节的内存来进行分析！

不幸的是，这个问题与跟踪生成的方式密切相关。为了保持运行时开销低，所有事件都被写入等效于线程本地缓冲区。但这意味着事件以不正确的顺序出现，跟踪工具必须弄清楚真正发生的事情。

使跟踪数据规模化并保持开销低的关键见解是偶尔分割正在生成的跟踪。每个分割点会像同时禁用和重新启用跟踪一样运行。到目前为止的所有跟踪数据都代表了一个完整且独立的跟踪，而新的跟踪数据会无缝地从上次中断的地方继续。

修复这个问题需要[重新思考和重写跟踪实现的许多基础部分](/issue/60773)。我们很高兴地宣布，这项工作已经在 Go 1.22 中完成，并且现在已经普遍可用。重写带来了[许多不错的改进](/doc/go1.22#runtime/trace)，包括对[`go tool trace`命令](/doc/go1.22#trace)的一些改进。如果你感兴趣，所有细节都在[设计文档](https://github.com/golang/proposal/blob/master/design/60773-execution-tracer-overhaul.md)中。

（注意：`go tool trace`仍会将完整的跟踪加载到内存中，但对于由 Go 1.22+ 程序生成的跟踪，[移除此限制](/issue/65315)现在是可行的。）

## 飞行记录

假设你正在开发一个网络服务，某个 RPC 花了很长时间。由于慢请求的根本原因已经发生且未记录，你无法在知道 RPC 已经花费了很长时间的点开始跟踪。

有一种技术可以帮助解决这个问题，这就是飞行记录，你可能已经在其他编程环境中熟悉了。飞行记录的见解是持续进行跟踪，并始终保留最近的跟踪数据，以备不时之需。然后，一旦发生有趣的事情，程序就可以写出它所拥有的任何内容！

在能够分割跟踪之前，这几乎是行不通的。但由于低开销使连续跟踪现在成为可能，而且运行时现在可以在任何时候分割跟踪，因此实施飞行记录实际上是非常简单的。

因此，我们很高兴地宣布飞行记录器实验，现在在[golang.org/x/exp/trace 包](/pkg/golang.org/x/exp/trace#FlightRecorder)中可用。

请尝试使用它！下面是一个示例，设置飞行记录以捕获长时间的 HTTP 请求，帮助你入门。

```

    fr := trace.NewFlightRecorder()
    fr.Start()

    var once sync.Once
    http.HandleFunc("/my-endpoint", func(w http.ResponseWriter, r *http.Request) {
        start := time.Now()

        doWork(w, r)

        if time.Since(start) > 300*time.Millisecond {

            once.Do(func() {

                var b bytes.Buffer
                _, err = fr.WriteTo(&b)
                if err != nil {
                    log.Print(err)
                    return
                }

                if err := os.WriteFile("trace.out", b.Bytes(), 0o755); err != nil {
                    log.Print(err)
                    return
                }
            })
        }
    })
    log.Fatal(http.ListenAndServe(":8080", nil))

```

如果您有任何反馈，无论是正面的还是负面的，请分享到[提案问题](/issue/63185)！

## 跟踪读取器 API

伴随着追踪实现的重写，还进行了清理其他追踪内部结构的努力，比如`go tool trace`。这引发了一个尝试创建足够好且可以共享的追踪读取器API，可以使追踪更加可访问。

就像飞行记录仪一样，我们很高兴地宣布，我们还有一个实验性的追踪读取器API，我们希望能分享给大家。它可以在[与飞行记录仪相同的包中找到，golang.org/x/exp/trace](/pkg/golang.org/x/exp/trace#Reader)。

我们认为它已经足够好，可以开始在其上构建东西了，请尝试一下！以下是一个示例，用于测量等待网络的goroutine阻塞事件的比例。

```

    r, err := trace.NewReader(os.Stdin)
    if err != nil {
        log.Fatal(err)
    }

    var blocked int
    var blockedOnNetwork int
    for {

        ev, err := r.ReadEvent()
        if err == io.EOF {
            break
        } else if err != nil {
            log.Fatal(err)
        }

        if ev.Kind() == trace.EventStateTransition {
            st := ev.StateTransition()
            if st.Resource.Kind == trace.ResourceGoroutine {
                from, to := st.Goroutine()

                if from.Executing() && to == trace.GoWaiting {
                    blocked++
                    if strings.Contains(st.Reason, "network") {
                        blockedOnNetwork++
                    }
                }
            }
        }
    }

    p := 100 * float64(blockedOnNetwork) / float64(blocked)
    fmt.Printf("%2.3f%% instances of goroutines blocking were to block on the network\n", p)

```

就像飞行记录仪一样，有一个[提议问题](/issue/62627)可以让您留下反馈！

我们想要快速提到Dominik Honnef，他在早期尝试了这个工具，提供了很好的反馈，并为API提供了对旧版追踪的支持。

## 谢谢！

这项工作的完成，在很大程度上要感谢[诊断工作组](/issue/57175)中的那些人，这个工作组是一年多前由来自Go社区各方利益相关者共同开展的合作项目，并向公众开放。

我们想要花一点时间感谢过去一年中定期参加诊断会议的社区成员：Felix Geisendörfer、Nick Ripley、Rhys Hiltner、Dominik Honnef、Bryan Boreham、thepudds。

大家所做的讨论、反馈和工作对我们今天所处的位置至关重要。谢谢！
