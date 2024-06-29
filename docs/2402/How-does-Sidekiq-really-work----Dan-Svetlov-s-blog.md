<!--yml

类别：未分类

日期：2024-05-27 14:37:12

-->

# Sidekiq到底是如何工作的？| 丹·斯维特洛夫的博客

> 来源：[https://dansvetlov.me/sidekiq-internals/](https://dansvetlov.me/sidekiq-internals/)
> 
> 自发布以来，[Mike Perham已背书这篇文章](https://www.mikeperham.com/2024/02/22/how-does-sidekiq-work/)，Sidekiq的创造者。
> 
> [Hacker News讨论](https://news.ycombinator.com/item?id=39257174)

[Sidekiq](https://github.com/sidekiq/sidekiq)是最常见的Ruby后台作业处理器之一。对于任何有经验的Ruby开发者而言，它无需介绍。Sidekiq以其高效、经过实战检验且易于使用的解决方案，拥有超过10年的记录，用于将应用程序逻辑的执行转移到后台。

它利用线程模型进行作业处理，使用Redis作为后端，并声称在免费的开源版本中在处理作业时具有“至少一次”的语义（带有一个警告）。Sidekiq还提供了另外两个付费版本 - Pro和Enterprise，每个版本都引入了额外的功能和扩展。由于明显的原因，我不会详细介绍这些版本的细节。

本文将深入探讨Sidekiq的内部结构，突出其关键方面以及我个人认为有趣或奇特的设计和实现决策，通过直接深入源代码并跟踪作业的完整生命周期。

我不会涵盖Sidekiq的用户界面API和一般的“如何操作”。它的[wiki](https://github.com/sidekiq/sidekiq/wiki/Getting-Started)提供了更好的资源。预期具备对该库及Ruby的基本了解。

本文中检查的代码来自Sidekiq 7.2版本和Ruby 3.2，运行在[MRI/cruby](https://github.com/ruby/ruby)上。尽管所讨论的代码可能会随着Sidekiq的更新而过时，但除非维护者在哲学上做出 drast 的变化，否则其一般原则和架构不会过时。尽管如此，本文可能对希望通过构建作业处理器练习其系统编程技能的任何人都是宝贵的资源。

值得注意的是，我既不是Sidekiq的维护者，也不是贡献者。我所有的观察和结论都源于阅读源代码、注释以及通过`git blame`获取的问题和拉取请求的讨论。

## 启动过程

我认为熟悉代码的最佳方式是首先检查其入口点。作为*后台*作业处理器的Sidekiq必须作为单独的进程初始化。`bin/sidekiq`是用于启动此过程的脚本，其主要职责是实例化`Sidekiq::CLI`单例类。

`CLI` 对象使用 Ruby 的 [`OptionParser`](https://github.com/sidekiq/sidekiq/blob/4ec059d53dbf1de67e41e3bd1687c7d90c12d580/lib/sidekiq/cli.rb#L26-L32) 解析传递给进程的命令行参数配置，并初始化了[全局默认的 `Sidekiq::Config`](https://github.com/sidekiq/sidekiq/blob/4ec059d53dbf1de67e41e3bd1687c7d90c12d580/lib/sidekiq/config.rb#L11-L35)。

如果通过 `--config/-C` CLI 参数传递了有效的 YAML 配置路径，它将被使用；否则，假定该配置可以在相对路径 `./config` 下找到。此外，也可以提供自定义的 `--require/-R` 参数，该参数应指向作业类所在的应用程序根目录。在这种情况下，配置将从 `<require 指定的路径>/config` 获取。

如果配置文件存在，Sidekiq 将使用 `ERB` 进行评估。这允许将配置文件定义为模板。然而，一个显著的注意事项是，在需要 Rails 应用程序或通过 `--require` 指定的文件之前，将会对此文件进行评估，这使得引用应用程序代码中定义的常量成为不可能。

然后，如果它们的值未明确设置，默认值的情况下，将填充 `queues` 和 `concurrency` 配置选项。使用默认值时，Sidekiq 仅处理 `default` 队列，并将其 `concurrency` 设置为 `RAILS_MAX_THREADS` 环境变量的值（如果 Sidekiq 与 Rails 应用程序一起使用）。

作为配置过程中的倒数第二步，每个胶囊都设置了 `queues` 和 `concurrency` 选项。胶囊将在后续部分详细介绍，但目前可以将胶囊视为配置选项的分区化组。在基本的 Sidekiq 设置中，胶囊不对用户公开；但是，默认胶囊会隐式使用。可以在配置 YAML 文件和通过 `Sidekiq.configure_server` 中定义自定义胶囊。

最后，`CLI` 实例验证 `--require` 参数指向的是现有文件，或者如果它指向目录，则 `config/application.rb` 存在。还检查 `concurrency` 和 `timeout` 是否为正整数。

## 进入主循环

加载和验证配置后，`bin/sidekiq` 可执行文件在 CLI 实例上调用 `run`。在这一点上，应用程序被加载。

如果 `require` 配置参数中的路径是一个目录，默认情况下是 `.`，除非明确配置，Sidekiq 假定它正在[用于 Rails 应用程序](https://github.com/sidekiq/sidekiq/blob/4ec059d53dbf1de67e41e3bd1687c7d90c12d580/lib/sidekiq/cli.rb#L294-L308)，并要求 `rails` 和 `sidekiq/rails`，随后通过 `require File.expand_path("#{@config[:require]}/config/environment.rb")`。

否则，如果 `require` 指向一个文件，该文件将被加载。这意味着如果在非 Rails 应用程序中使用，`require` 配置参数必须始终明确提供，并且指向一个加载应用程序代码的入口点。

调用 `Sidekiq.configure_server` 和 `Sidekiq.configure_client` 也会在应用程序启动期间进行评估，因为它们通常是可以急加载的应用程序代码的一部分。这种设置最常见于 `config/initializers/sidekiq.rb` Rails 初始化程序中。

### 信号处理

在 `run` 方法的下一步之前，值得回顾一下 [信号](https://man7.org/linux/man-pages/man7/signal.7.html) 是什么。

本质上，信号可以被视为一种进程间通信的类型。可以为大多数信号定义自定义处理程序，一旦进程接收到相应的信号，它们将执行任意逻辑。信号的一个用例是控制长时间运行的守护进程，例如 Sidekiq。例如，Kubernetes 发送 SIGTERM 到 Pod，以便它们可以优雅地关闭并执行必要的清理工作，这对于像 Sidekiq 这样的作业处理器是相关的。大多数现代编排工具和托管平台通过发送 SIGINT 或 SIGTERM 给运行中的进程来支持优雅终止。

理论讲解结束后，让我们看看 Sidekiq 如何在这里与信号交互 [here](https://github.com/sidekiq/sidekiq/blob/4ec059d53dbf1de67e41e3bd1687c7d90c12d580/lib/sidekiq/cli.rb#L49-L74) ：

```
self_read, self_write = IO.pipe
sigs = %w[INT TERM TTIN TSTP]
# ...
sigs.each do |sig|
  old_handler = Signal.trap(sig) do
    if old_handler.respond_to?(:call)
      begin
        old_handler.call
      rescue Exception => exc
        # signal handlers can't use Logger so puts only
        puts ["Error in #{sig} handler", exc].inspect
      end
    end

    self_write.puts(sig)
  end
rescue ArgumentError
  puts "Signal #{sig} not supported"
end 
```

值得注意的第一件事是使用 Ruby 提供的 `IO.pipe` API 创建一个未命名管道。它的 [返回值](https://github.com/ruby/ruby/blob/a7335e11e354d1ee2e15233f32f087230069ad5c/io.c#L408-L440) 是一个包含由底层系统调用（在大多数类 UNIX 系统上为 `pipe2` 或 `pipe`）返回的文件描述符的 `IO` 实例数组：

```
int
rb_cloexec_pipe(int descriptors[2])
{
#ifdef HAVE_PIPE2
    int result = pipe2(descriptors, O_CLOEXEC | O_NONBLOCK);
#else
    int result = pipe(descriptors);
#endif 
    if (result < 0)
        return result;

// ...
} 
```

未命名管道在分叉环境中非常有用，因为父进程可以与其子进程及其反之进行通信，而不需要进行复杂的设置。但是，Sidekiq 在免费操作系统版本中不使用分叉模型，而是广泛使用线程，所有线程共享进程的内存。那么为什么要使用管道？在回答这个问题之前，让我们来看看 Sidekiq 的信号处理程序中发生了什么。

从上面的 Ruby 片段可以看出，Sidekiq 设置了对 SIGINT、SIGTERM、SIGTTIN 和 SIGTSTP 的处理程序。每个处理程序都是使用 `Signal.trap` 按顺序定义的，在类 UNIX 系统上，它内部构造了一个处理程序并调用了 [`sigaction`](https://man7.org/linux/man-pages/man2/sigaction.2.html) 系统调用：

```
static sighandler_t
ruby_signal(int signum, sighandler_t handler)
{
    struct sigaction sigact, old;

  // Code that populates the sigact struct is omitted

  // Register the signal handler
    if (sigaction(signum, &sigact, &old) < 0) {
        return SIG_ERR;
    }

    if (old.sa_flags & SA_SIGINFO)
        handler = (sighandler_t)old.sa_sigaction;
    else
        handler = old.sa_handler;
    ASSUME(handler != SIG_ERR);

    // Return the previous handler of the same signal
    return handler;
} 
```

如观察所示，此函数返回了先前为同一信号注册的处理程序的实例。

让我们重新审视 `Sidekiq::CLI` 中的信号注册：

```
old_handler = Signal.trap(sig) do
  if old_handler.respond_to?(:call)
    begin
      old_handler.call
    rescue Exception => exc
      puts ["Error in #{sig} handler", exc].inspect
    end
  end
  self_write.puts(sig)
end 
```

这段代码乍一看可能会令人困惑 - `old_handler`如何在块内部调用，而`old_handler`本身又是该块的返回值？为了解开这个小的“盗梦空间”时刻，考虑以下事实：

+   正如我们所见，`Signal.trap`返回先前为信号注册的处理程序，该处理程序从`ruby_signal` C函数传播而来。这可以是`Proc`的实例或字符串。有关可能的字符串返回值，请参考`trap`函数的[这部分](https://github.com/ruby/ruby/blob/a7335e11e354d1ee2e15233f32f087230069ad5c/signal.c#L1310-L1323)，负责将旧处理程序返回到Ruby环境。

+   传递给`Signal.trap`的块不会立即评估；相反，它仅在进程接收到相应的信号时才被调用。proc对象存储在全局`vm->trap_list`结构中。

+   由于Ruby中绑定的工作方式，局部变量`old_handler`在块内部是可用的；`old_handler`只是通过块的内部作用域继承而来。这种行为类似于即使在引发异常之前未定义它们，也可以在救援块中引用局部变量。

所以，`Signal.trap`将传递的块转换为`Proc`实例，不会立即评估它，并在全局`trap_list`中注册。当接收到信号时，相应的处理程序将被调用。`old_handler`将评估为表示处理程序行为的字符串（例如，默认或忽略），如果之前未定义自定义信号陷阱，则为此情况，或者是旧的自定义处理程序。Sidekiq礼貌地处理了以前定义的陷阱处理程序，因为它无法对其运行的环境进行假设。开发人员或其他库可能已经声明了预期被调用的信号处理程序。

然而，陷阱上下文以信号名称被写入先前创建的未命名管道结束。异常直接发出到STDOUT而不是使用`Sidekiq.logger`记录可能是提示处理程序逻辑未直接评估的原因。关于为何会发生这种情况以及SIGINT、SIGTERM和其他处理程序的实际行为，请继续阅读。

`run`方法的其余执行时间用于急切加载资源，即[Redis连接池](https://github.com/sidekiq/sidekiq/blob/4ec059d53dbf1de67e41e3bd1687c7d90c12d580/lib/sidekiq/cli.rb#L73-L76)和[服务器中间件链](https://github.com/sidekiq/sidekiq/blob/4ec059d53dbf1de67e41e3bd1687c7d90c12d580/lib/sidekiq/cli.rb#L101-L102)。这种模式通常用于在多线程环境中初始化全局资源时避免竞态条件。`run`方法是这样做的绝佳机会，因为此时只有一个主线程，使得在创建状态时同步变得多余。或者，如果资源是惰性加载的，那么其访问器必须用同步原语（如互斥锁）包装，这会导致在每次触发正常路径时都会产生性能损失，而资源已分配并且只需要被引用。

最后，`run`调用`launch`并将管道的读取端作为参数传递。`launch`方法是主线程在Sidekiq进程运行时所花费其余时间的地方：

```
def launch(self_read)
  @launcher = Sidekiq::Launcher.new(@config)

  begin
    launcher.run

    while self_read.wait_readable
      signal = self_read.gets.strip
      handle_signal(signal)
    end
  rescue Interrupt
    launcher.stop

    exit(0)
  end
end 
```

创建了一个`Sidekiq::Launcher`实例，并调用了`run`方法。这个过程发生的地方被包裹在一个`rescue Interrupt`块中。

默认情况下，仅当进程[没有注册自定义`SIGINT`处理程序时](https://github.com/ruby/ruby/blob/a7335e11e354d1ee2e15233f32f087230069ad5c/signal.c#L1105-L1109)才会引发`Interrupt`，而这里的情况并非如此，因为`Signal.trap`已经显式调用了SIGINT。在这种情况下，让我们看看它可能是从哪里引发的。

在启动器被命令运行之后，主线程进入一个无限循环，调用`self_read.wait_readable`。这个方法在没有超时参数的情况下无限期地等待，直到底层文件描述符有可读数据可用。当之前使用`Signal.trap`陷阱捕获的任何信号被发送到进程时，它最终将被写入管道的写入端并在此处读取。`handle_signal`将调用[在`Sidekiq::CLI::SIGNAL_HANDLERS`哈希表中定义的相应处理程序](https://github.com/sidekiq/sidekiq/blob/4ec059d53dbf1de67e41e3bd1687c7d90c12d580/lib/sidekiq/cli.rb#L190-L217)。

```
SIGNAL_HANDLERS = {
  "INT" => ->(cli) { raise Interrupt },
  "TERM" => ->(cli) { raise Interrupt },
  "TSTP" => ->(cli) {
    cli.logger.info "Received TSTP, no longer accepting new work"
    cli.launcher.quiet
  },
  "TTIN" => ->(cli) {
    Thread.list.each do |thread|
      cli.logger.warn "Thread TID-#{(thread.object_id ^ ::Process.pid).to_s(36)}  #{thread.name}"
      if thread.backtrace
        cli.logger.warn thread.backtrace.join("\n")
      else
        cli.logger.warn "<no backtrace available>"
      end
    end
  }
}
UNHANDLED_SIGNAL_HANDLER = ->(cli) { cli.logger.info "No signal handler registered, ignoring" }
SIGNAL_HANDLERS.default = UNHANDLED_SIGNAL_HANDLER

def handle_signal(sig)
  logger.debug "Got #{sig} signal"
  SIGNAL_HANDLERS[sig].call(self)
end 
```

需要注意的关键信号处理程序是SIGINT、SIGTERM和SIGTSTP。^(前两个处理程序仅包含`raise Interrupt`，这将在之前我们查看过的`CLI.launch`循环中被捕获。SIGTSTP有些不同 - 它调用`Sidekiq::Launcher`实例上的`quiet`方法。)

在我们深入探讨启动器之前，让我们首先回答为什么信号处理程序不直接从陷阱上下文执行，而是被发送到管道并在主线程循环中处理。

从陷阱上下文调用的任何代码必须是可重入的。实际上意味着一些Ruby结构和方法不能在`Signal.trap`块中使用，尤其是`Mutex`。

之所以不允许在陷阱上下文中进行此类操作很简单：在Ruby中定义的自定义信号处理程序（即带有块的`Signal.trap`调用）可以在Ruby程序的任何时点执行。这意味着，即使是线程安全的代码，在并发执行时可能被认为是“正确”的，但如果信号陷阱代码访问任何不仅由其他线程共享，而且还由主线程本身共享的同步资源，将会遇到死锁。

Sidekiq的任何信号处理程序都不直接引用互斥锁。然而，几乎所有信号处理程序都使用`Sidekiq.logger`发出消息，默认情况下是纯Ruby的`Logger`类的子类。`Logger`又使用互斥锁来在内部同步写入：

```
def write(message)
  begin
    # ...
    synchronize do # A convenience method that calls Monitor#synchronize
      @dev.write(message)
    end
    # ...
  rescue Exception => ignored
    warn("log writing failed. #{ignored}")
  end
end 
```

这意味着在主线程拥有互斥锁期间调用信号处理程序时，可能会访问相同的日志记录器实例。

保持信号陷阱简短，并通过通知主线程的方式推迟实际逻辑，允许解除通常应用于信号处理程序的限制。

### 管理生命周期

如前一节所述，`Sidekiq::CLI`实例化一个`Sidekiq::Launcher`实例，调用其`run`方法，并进入一个无限循环，等待传入信号。SIGINT和SIGTERM调用启动器实例的`stop`，而SIGTSTP调用其`quiet`方法，到目前为止，这就是我们对启动器的知识的全部。让我们来剖析它。

值得注意的是，在启动器初始化期间创建管理器：

```
# Launcher

def initialize(config)
  # ...
  @managers = config.capsules.values.map do |cap|
    Sidekiq::Manager.new(cap)
  end
  # ...
end 
```

这里是胶囊完全进入聚光灯下的地方。它们的配置是在`Sidekiq::CLI`实例中完成的，现在它们被用来实例化`Sidekiq::Manager`的实例。胶囊可以被看作是Sidekiq进程内的隔间。它们被表示为包含个别`concurrency`和`queues`配置参数的Ruby对象。

如果没有自定义配置，Sidekiq隐式地使用单一的“默认”胶囊，而且可以在需要时定义自定义胶囊。

在我们继续研究启动器生命周期方法之前，让我们先深入了解一下栈的更深层次，并检查与主处理循环相关的关键组件。

每个胶囊都表现为一个管理器，充当监督一组处理器生命周期的容器。处理器的数量由`concurrency`设置确定。管理器通过公开的`start`、`quiet`和`stop`等方法来控制其处理器。

处理器作为Sidekiq内的执行单元，负责执行作业。其公共API包括`terminate`、`kill`和`start`等方法，提供对其执行生命周期的控制给其管理器。

#### 启动

现在让我们回到启动器。以下是其简化的 [`run`](https://github.com/sidekiq/sidekiq/blob/4ec059d53dbf1de67e41e3bd1687c7d90c12d580/lib/sidekiq/launcher.rb#L38-L44) 方法：

```
# Launcher

def run
  @thread = safe_thread("heartbeat", &method(:start_heartbeat))
  @managers.each(&:start)
end 
```

它最初启动一个心跳线程，每 10 秒将一些统计信息转储到 Redis 中。

`safe_thread` 本质上是 Sidekiq 提供的一个辅助方法，它在传递的 proc 周围包装了一个 rescue 块，通过 `Config#handle_exception` 调用内部异常处理程序。默认情况下，此处理程序仅记录异常而不会重新引发它。

管理器的 `start` 方法如下所示：

```
# Manager

def start
  @workers.each(&:start)
end 
```

`@workers` 包含了 `Processor` 类的实例。

深入到一个堆栈帧更深处，这里是处理器的 `start` 方法：

```
# Processor

def start
  @thread ||= safe_thread("#{config.name}/processor", &method(:run))
end 
```

与启动器类似，处理器启动一个执行 `Processor#run` 方法的线程。目前，让我们将其视为一个黑盒子，只假设它启动一个从队列中提取作业并执行它们的无限循环。

此时，Sidekiq 进程实际开始执行排队的作业。然而，这对于稳定性来说是不够的，因为关机是每个进程生命周期中不可避免的一部分。在诸如 Sidekiq 这样的作业处理器中优雅地处理它们可能比在 Web 服务器中更重要，这些原因将在接下来的章节中讨论。

#### 安静下来

正如我们已经看到的，某些信号会转换为对启动器的调用 - 即 `quiet` 和 `stop`。我们先从 `quiet` 开始，因为它通常应该在 `stop` 之前。

这是 `Launcher#quiet` 的修剪版本：

```
# Launcher

def quiet
  return if @done

  @done = true
  @managers.each(&:quiet)
end 
```

早期的返回语句是为了处理进程接收到多个额外的 SIGTSTP 信号的情况。

`Manager#quiet` 采取了同样的预防措施：

```
# Manager

def quiet
  return if @done
  @done = true

  @workers.each(&:terminate)
end 
```

这就是处理 `terminate` 的处理器方法：

```
# Processor

def terminate
  @done = true
end 
```

就是这样。尽管名字如此，该方法并没有执行更多比设置实例变量更激烈的事情。这就是我们揭示 `Processor#run` 方法的地方：

```
# Processor

def run
  process_one until @done
  @callback.call(self)
rescue Sidekiq::Shutdown
  @callback.call(self)
rescue Exception => ex
  @callback.call(self, ex)
end 
```

这与我们先前的假设一致：`run` 方法在处理器的 `start` 方法中由一个线程执行，并进入一个循环，其中断开条件是 `@done` 实例变量为 false。

需要明白的是，调用 `terminate` 并不会立即停止处理器处理当前作业。相反，它会继续处理当前作业直到完成。

因此，当调用 `terminate` 时，`run` 方法将在完成当前作业后退出循环。此时，会调用一个 `@callback`。此回调实际上是一个 `Method` 实例，它包装了 `Manager#processor_result`，在每个处理器初始化期间由管理器传递。

```
class Manager
  def initialize(capsule)
    @done = false
    @workers = Set.new
    @plock = Mutex.new
    @count.times do
      @workers << Processor.new(@config, &method(:processor_result))
    end
  end

  # ...

  def processor_result(processor, reason = nil)
    @plock.synchronize do
      @workers.delete(processor)
      unless @done
        p = Processor.new(@config, &method(:processor_result))
        @workers << p
        p.start
      end
    end
  end
end 
```

通过阅读 `processor_result` 的源代码，可以明确地看到处理器本身负责检出 `Manager` 的 `@workers` 集合。因此，如果处理器遇到 `Sidekiq::Shutdown` 或任何其他未捕获的异常，它将从集合中移除自己，以与 `Manager#initialize` 中所做的方式创建新处理器相似，将该处理器添加到集合，并在其上调用 `start`。

但是，在我们正在检查的代码路径中，在调用 `quiet` 后，回调只会从工作集中移除处理器。因此，调用 `quiet` 将导致处理器停止接收新任务运行，并且 *最终* 工作集将变为空。正如我们已经注意到的那样，当前由处理器执行的作业不会突然停止；相反，它们将继续自然完成。

这使得 SIGTSTP 信号成为在进程终止之前一段时间发送的完美选择。`Launcher#quiet` 是通知 Sidekiq 处理器在处理完当前作业后避免接收新作业的方法，因为进程即将终止。这最大程度地减少了进程需要完全停止时的侵入性动作的需求。

#### 停止

`Launcher#stop` 方法从使用 POSIX 的 `clock_gettime` 开始计算截止时间。使用 `CLOCK_MONOTONIC` 来获取当前的绝对经过时间，而不是墙上时间，因为墙上时间可能无法预测地变化：

```
# Launcher

def stop
  deadline = ::Process.clock_gettime(::Process::CLOCK_MONOTONIC) + @config[:timeout]

  quiet
  stoppers = @managers.map do |mgr|
    Thread.new do
      mgr.stop(deadline)
    end
  end

  stoppers.each(&:join)
end 
```

默认的 `timeout` 配置设置为 25 秒，这个数字并非随意选择；在历史上，许多托管平台和编排器使用 30 秒的宽限期，以使进程能够优雅地响应 SIGTERM。Sidekiq 将这个超时时间设置得稍低，以便更有机会完成必要的清理工作。

`quiet` 作为 `stop` 方法的第一步被调用，以减少下一步需要等待的工作量。调用 `quiet` 的另一个目的是确保，如果在 SIGINT 或 SIGTERM 之前没有明确发送 SIGTSTP，处理器停止接收新作业：

```
# Manager

def stop(deadline)
  quiet

  return if @workers.empty?

  logger.info { "Pausing to allow jobs to finish..." }
  wait_for(deadline) { @workers.empty? }
  return if @workers.empty?

  hard_shutdown
ensure
  capsule.stop
end 
```

如果处理器设法及时完成其当前工作，它们将在完成当前作业后从其管理器的 `@workers` 数组中移除自己。`stop` 中的两个条件语句用于处理通过预先终止执行来处理快乐路径的情况，避免不必要的额外工作。这就是为什么确保首先调用 `quiet` 是重要的原因。

如果经理发现在第一个 `return if @workers.empty?` 检查之前有任何处理器无法立即完成，它必须在其 `stop` 方法中等待它们。请记住，默认情况下截止时间为 25 秒，由启动器传递。`wait_for` 方法会在循环中执行块，直到块的返回值为真：

```
# Manager

def wait_for(deadline, &condblock)
  remaining = deadline - ::Process.clock_gettime(::Process::CLOCK_MONOTONIC)
  while remaining > PAUSE_TIME
    return if condblock.call
    sleep PAUSE_TIME
    remaining = deadline - ::Process.clock_gettime(::Process::CLOCK_MONOTONIC)
  end
end 
```

如果截止时间后仍有活动处理器滞留，将调用 `hard_shutdown`。这里是这个方法的 [实现](https://github.com/sidekiq/sidekiq/blob/4ec059d53dbf1de67e41e3bd1687c7d90c12d580/lib/sidekiq/manager.rb#L87-L119)，并保留了原始注释：

```
# Manager

def hard_shutdown
  # We've reached the timeout and we still have busy threads.
  # They must die but their jobs shall live on.
  cleanup = nil
  @plock.synchronize do
    cleanup = @workers.dup
  end

  if cleanup.size > 0
    jobs = cleanup.map { |p| p.job }.compact
    # ...
    capsule.fetcher.bulk_requeue(jobs)
  end

  cleanup.each do |processor|
    processor.kill
  end

  # when this method returns, we immediately call `exit` which may not give
  # the remaining threads time to run `ensure` blocks, etc. We pause here up
  # to 3 seconds to give threads a minimal amount of time to run `ensure` blocks.
  deadline = ::Process.clock_gettime(::Process::CLOCK_MONOTONIC) + 3
  wait_for(deadline) { @workers.empty? }
end 
```

我们暂时不会探讨 `capsule.fetcher.bulk_requeue` 的功能。这是 Sidekiq 中最关键的逻辑之一，暂时假设它根据方法名称的暗示重新排列作业。

每个处理器在调用 `hard_shutdown` 时未从 `@workers` 数组中检出自身，都将被终止。这里是 [`Processor#kill`](https://github.com/sidekiq/sidekiq/blob/4ec059d53dbf1de67e41e3bd1687c7d90c12d580/lib/sidekiq/processor.rb#L49-L59) 的执行内容：

```
# Processor

def kill(wait = false)
  @done = true
  return unless @thread

  @thread.raise ::Sidekiq::Shutdown
end 
```

它类似于 `quiet`，但额外在处理器线程上引发 `Sidekiq::Shutdown`，这是 `Interrupt` 的子类。

然而，引发此异常并不意味着线程将在上限时间内真正完成。因此，每个进程被杀死后，管理器会等待 3 秒钟，以便它们有机会优雅地终止。稍后我们将详细讨论这种优雅的终止方式。

我们已经探讨了 Sidekiq 如何建立必要的类层次结构，以确保处理过程的顺利进行。启动器监督管理器，管理器又管理处理作业的处理器。现在让我们来看看处理器如何获取要执行的作业。

### 队列处理

Sidekiq 作业会路由到队列。至少每个 Sidekiq 安装将使用 `default` 队列。应该处理的 Sidekiq 队列可以在全局范围或每个 capsule 基础上定义。您可以使用 `config/sidekiq.yml` 配置文件、`-q` 命令行参数或 `Sidekiq.configure_server` 指定队列集。此配置会填充 `default` capsule 的 `queues` 参数，除非在配置文件或服务器配置中为特定 capsule 提供了特定配置。

最终，所有提供的队列列表都会传递到 `Capsule#queues=` 设置器方法中，这对于队列处理逻辑至关重要。这里是 [其内容](https://github.com/sidekiq/sidekiq/blob/4ec059d53dbf1de67e41e3bd1687c7d90c12d580/lib/sidekiq/capsule.rb#L48-L70) 的逐字复制，保留了原始注释：

```
# Sidekiq checks queues in three modes:
# - :strict - all queues have 0 weight and are checked strictly in order
# - :weighted - queues have arbitrary weight between 1 and N
# - :random - all queues have weight of 1
def queues=(val)
  @weights = {}
  @queues = Array(val).each_with_object([]) do |qstr, memo|
    arr = qstr
    arr = qstr.split(",") if qstr.is_a?(String)
    name, weight = arr
    @weights[name] = weight.to_i
    [weight.to_i, 1].max.times do
      memo << name
    end
  end
  @mode = if @weights.values.all?(&:zero?)
    :strict
  elsif @weights.values.all? { |x| x == 1 }
    :random
  else
    :weighted
  end
end 
```

有三种队列轮询方式可选。如果没有队列定义权重（使用 `default,10` 标记），模式将设置为 `strict`。如果所有队列都明确设置了权重为 1，模式将设置为 `random`。在任何其他情况下，模式将设置为 `weighted`。

值得注意的一点是，`@queues`数组将包含每个队列的`m`个副本，其中`m`是它们的权重。例如，如果队列按照`default,10 queue_one,5 queue_two,3`的权重配置，`@queues`将由10个`:default`元素，5个`:queue_one`元素和3个`:queue_two`元素组成。

让我们回到`Processor#run`。正如我们已经看到的，此方法使处理器进入一个循环，直到调用`terminate`或`kill`为止调用`Processor#process_one`：

```
# Processor

def get_one
  uow = capsule.fetcher.retrieve_work
  if @down
    logger.info { "Redis is online, #{::Process.clock_gettime(::Process::CLOCK_MONOTONIC) - @down} sec downtime" }
    @down = nil
  end
  uow
rescue Sidekiq::Shutdown
rescue => ex
  handle_fetch_exception(ex)
end 
```

条件分支是为了防止Sidekiq在`handle_fetch_exception`触发的回退后恢复。如果与Redis联系时存在瞬态连接问题，则该方法简单地`sleep` 1秒。

`Sidekiq::Shutdown`在此处被静音以减少由从`handle_fetch_exception`调用的错误处理程序引起的噪音。由于此时`@done`实例变量被设置为`false`，新作业的处理停止。

`Capsule`类中的`fetcher`方法提取获取器对象。以下是其实现：

```
def fetcher
  @fetcher ||= begin
    inst = (config[:fetch_class] || Sidekiq::BasicFetch).new(self)
    inst.setup(config[:fetch_setup]) if inst.respond_to?(:setup)
    inst
  end
end 
```

Sidekiq的操作系统版本带有单个获取器选项`BasicFetch`。

Pro版本包括[`SuperFetch`](https://github.com/sidekiq/sidekiq/wiki/Reliability#using-super_fetch)，为Sidekiq作业提供了更强大的耐久性保证。但是，本文未涉及此获取器的具体信息。

让我们来看看`BasicFetch`的内部。以下是相关方法，保留了原始注释：

```
class BasicFetch
  # We want the fetch operation to timeout every few seconds so the thread
  # can check if the process is shutting down.
  TIMEOUT = 2

  UnitOfWork = Struct.new(:queue, :job, :config) {
    def acknowledge
      # nothing to do
    end

    def queue_name
      queue.delete_prefix("queue:")
    end

    def requeue
      config.redis do |conn|
        conn.rpush(queue, job)
      end
    end
  }

  # ...

  def retrieve_work
    qs = queues_cmd
    # 4825 Sidekiq Pro with all queues paused will return an
    # empty set of queues
    if qs.size <= 0
      sleep(TIMEOUT)
      return nil
    end

    queue, job = redis { |conn| conn.blocking_call(conn.read_timeout + TIMEOUT, "brpop", *qs, TIMEOUT) }
    UnitOfWork.new(queue, job, config) if queue
  end

  # ...

  # Creating the Redis#brpop command takes into account any
  # configured queue weights. By default Redis#brpop returns
  # data from the first queue that has pending elements. We
  # recreate the queue command each time we invoke Redis#brpop
  # to honor weights and avoid queue starvation.
  def queues_cmd
    if @strictly_ordered_queues
      @queues
    else
      permute = @queues.shuffle
      permute.uniq!
      permute
    end
  end
end 
```

正如其名，这个类中的逻辑很简单：构造一个队列数组（请记住，队列权重会导致`@queues`数组中出现重复条目），如果没有严格的顺序，则对其进行洗牌，然后将其传递给Redis的`BRPOP`命令。

当存在加权队列时，在结果数组上调用`#uniq!`可能看起来违反直觉，但事实并非如此。数组中存在的相同条目越多，其中一个条目出现在数组头部的几率就越高。`uniq!`保留了元素的顺序。

[`BRPOP`](https://redis.io/commands/brpop/)接受多个[list](https://redis.io/docs/data-types/lists/)名称作为参数，并以阻塞方式弹出第一个非空列表的尾元素。这意味着`@queues`数组中的第一个队列具有优先级。这里一个重要的细节是超时。正如`TIMEOUT`常量上面的注释所述，Sidekiq希望这些操作定期超时，以便进程可以检查是否在此期间调用了`terminate`或`kill`。

如果命令成功从列表中获取作业，则返回值将封装在`UnitOfWork` DTO中。

`BasicFetch` 的关键在于 `BRPOP` 从列表中移除作业。尽管 Sidekiq 尽力使自身尽可能弹性，并覆盖大多数失败场景，但这带来一个重要影响：作业可能会丢失。稍后我们将探讨可能发生这种情况的一些场景，以及 Sidekiq 如何在后续章节中试图减少这些影响。

正如你可能已经注意到的，对于 `BasicFetch` 的 `UnitOfWork`，`acknowledge` 方法是空的。虽然我自己还没有探索过 Sidekiq 的 Pro 或 Enterprise 版本，但合理地假设该方法对于 `SuperFetch` 提取器是相关的。`SuperFetch` 在从 Redis 获取后不立即移除作业，而是以某种形式保留它，这提高了可靠性，但也使得必须以某种方式标记作业已成功处理。`SuperFetch` 的 `acknowledge` 很可能正是用于此目的。随着我们深入探讨，`acknowledge` 方法将会变得更加重要。

在 `Processor#process_one` 中，如果获取成功，则调用 `Processor#process`。由于这个方法很长，我们将分段检查修改后的较小段落。

```
# Processor

def process(uow)
  jobstr = uow.job
  queue = uow.queue_name

  job_hash = nil
  begin
    job_hash = JSON.parse(jobstr)
  rescue => ex
    handle_exception(ex, {context: "Invalid JSON for job", jobstr: jobstr})
    now = Time.now.to_f
    redis do |conn|
      conn.multi do |xa|
        xa.zadd("dead", now.to_s, jobstr)
        xa.zremrangebyscore("dead", "-inf", now - @capsule.config[:dead_timeout_in_seconds])
        xa.zremrangebyrank("dead", 0, - @capsule.config[:dead_max_jobs])
      end
    end
    return uow.acknowledge
  end

  # ...
end 
```

从 Redis 获取的作业是表示为字符串的 JSON 负载，因此需要首先解析它。在这里，我们遇到了尸房和无效作业的概念。试图处理格式错误的作业无论重试多少次都不会产生积极的结果，因此将作业标记为‘无效’是一个实际的方法。

这通过 [`ZADD`](https://redis.io/commands/zadd/) 命令实现，该命令将成员添加到 [排序集合](https://redis.io/docs/data-types/sorted-sets/) 中，其中排序键是作业被宣告为无效的时间。

下面的两个命令用于删除在尸房中超过默认允许时间的无效作业，即 6 个月，并仅保留集合中最新的 10000 个作业。所有这些操作都在 Redis 事务中使用 [`MULTI`](https://redis.io/commands/multi/) 命令完成。

作业负载成功解析后，需要执行它。`#process` 方法的其余部分正是如此（保留了注释）：

```
# Processor

def process(uow)
  # ...

  ack = false
  Thread.handle_interrupt(Sidekiq::Shutdown => :never) do
    Thread.handle_interrupt(Sidekiq::Shutdown => :immediate) do
      dispatch(job_hash, queue, jobstr) do |inst|
        config.server_middleware.invoke(inst, job_hash, queue) do
          execute_job(inst, job_hash["args"])
        end
      end
      ack = true
    rescue Sidekiq::Shutdown
      # Had to force kill this job because it didn't finish
      # within the timeout.  Don't acknowledge the work since
      # we didn't properly finish it.
    rescue Sidekiq::JobRetry::Handled => h
      # this is the common case: job raised error and Sidekiq::JobRetry::Handled
      # signals that we created a retry successfully.  We can acknowlege the job.
      ack = true
      e = h.cause || h
      handle_exception(e, {context: "Job raised exception", job: job_hash})
      raise e
    rescue Exception => ex
      # Unexpected error!  This is very bad and indicates an exception that got past
      # the retry subsystem (e.g. network partition).  We won't acknowledge the job
      # so it can be rescued when using Sidekiq Pro.
      handle_exception(ex, {context: "Internal exception!", job: job_hash, jobstr: jobstr})
      raise ex
    end
  ensure
    if ack
      uow.acknowledge
    end
  end
end 
```

首先显眼的是 `Thread.handle_interrupt` 的使用。简而言之，它允许修改内部 Ruby 异步事件的处理方式，例如 `Thread#raise` 和 `Thread#kill` - 前者正是 `Processor#kill` 已经使用的方法。

在这种特定情况下，首先使用 `Sidekiq::Shutdown => :never` 调用 `handle_interrupt`，并包装调用工作单元上的 `#acknowledge` 的 `ensure` 块。第二个调用直接嵌套在第一个之后，并将行为重置回 `:immediate`。

这样做的好处是：如果处理器被终止（使用 `Thread#raise(Sidekiq::Shutdown)`），而 ensure 块正在评估时，抛出的异常将被忽略，并且工作将被确认。如果行为未设置为 `:never`，在这种情况下将错过确认。内部块立即将其重置为 `:immediate`，因为这是期望的行为 - `Sidekiq::Shutdown` 应该中断作业代码的执行。

这在 OS 版本中并不那么重要，因为 `BasicFetch` 的 `UnitOfWork#acknowledge` 是一个无操作，但对于 `SuperFetch` 来说，它的 `acknowledge` 版本很可能会从 Redis 中移除作业，以标记其已处理。

救援块由注释自解释。一个值得注意的方面是，当成功创建重试时，`Sidekiq::JobRetry::Handled` 异常被重新抛出并传播到 `Processor#run`。这反过来触发了 `Manager` 提供的回调，该回调删除处理器并创建新的处理器。因此，作业重试将强制重新创建处理器，以新的 Ruby 线程替代原始线程。

让我们分解 `Processor#dispatch` 和 `Processor#execute_job` 方法：

```
# Processor
# Relevant bits from #dispatch, #process & #execute_job combined together

@job_logger.prepare(job_hash) do
  @retrier.global(jobstr, queue) do
    @job_logger.call(job_hash, queue) do
      stats(jobstr, queue) do
        @reloader.call do
          klass = Object.const_get(job_hash["class"])
          inst = klass.new
          inst.jid = job_hash["jid"]
          @retrier.local(inst, jobstr, queue) do
            config.server_middleware.invoke(inst, job_hash, queue) do
              inst.perform(*job_hash["args"])
            end
          end
        end
      end
    end
  end
end 
```

`@job_logger` 变量是 `Sidekiq::JobLogger` 的一个实例。它的 `prepare` 方法负责将作业元数据放入线程局部上下文中，确保在堆栈更深处发出的日志包含有用的相关信息。

这个管道中的另一个关键步骤是 `@reloader.call` 的调用。这是 Sidekiq 与 Rails 框架集成的地方。如其名称所示，[reloader](https://guides.rubyonrails.org/threading_and_code_execution.html#reloader) 负责代码重载。这很重要，因为作业通常驻留在 Rails 应用程序的 `app/` 目录中，应该可以重新加载。此外，reloader 处理 [ActiveRecord 连接清理](https://github.com/rails/rails/blob/a255742b2eb711baa8fd7a8937852851ddc8a679/activerecord/lib/active_record/railtie.rb#L326-L331) 和其他相关问题。

Sidekiq 通过 `Object.const_get(job_hash["class"])` 提取作业类，并在调用服务器中间件链之后调用其 `#perform` 方法。Sidekiq 的中间件类似于 Rack 的中间件 - 是响应 `#call` 并在链不应停止时执行的任意类。这是希望增强或改变 Sidekiq 处理逻辑的集成的公共 API，这正是像 [Sentry](https://github.com/getsentry/sentry-ruby/blob/master/sentry-sidekiq/lib/sentry/sidekiq/sentry_context_middleware.rb)、[Datadog](https://github.com/DataDog/dd-trace-rb/blob/e4498741b7e85b7f886a8feb72ec62e64f86ad25/lib/datadog/tracing/contrib/sidekiq/server_tracer.rb) 等监控库所做的。

### 处理异常

当然，作业内调用的应用程序代码可能会引发异常。Sidekiq提供了几种处理它们的方式，默认行为是使用指数退避重试失败的作业最多25次。

让我们来看看`Processor`的`@retrier`实例变量的`#local`方法，它是`Sidekiq::JobRetry`的实例，包装了每个作业类的`#perform`方法：

```
class JobRetry
  class Handled < ::RuntimeError; end
  class Skip < Handled; end

  # ...

  def local(jobinst, jobstr, queue)
    yield
  rescue Handled => ex
    raise ex
  rescue Sidekiq::Shutdown => ey
    raise ey
  rescue Exception => e
    raise Sidekiq::Shutdown if exception_caused_by_shutdown?(e)

    msg = JSON.parse(jobstr)
    if msg["retry"].nil?
      msg["retry"] = jobinst.class.get_sidekiq_options["retry"]
    end

    raise e unless msg["retry"]
    process_retry(jobinst, msg, queue, e)

    raise Skip
  end
end 
```

由于此方法可以直接与特定实例化的作业实例相关联，因此称为“local”。一旦我们到达`#process_retry`，我们将看到它为何相关。

`Sidekiq::Shutdown`被重新引发，因为它应该绕过重试系统，`Handled`也被重新引发，以防开发人员决定从作业内部手动引发它。

救援`Exception`（即所有异常）是有趣的部分。

首先，Sidekiq会检测是否由于线程从`Processor#kill`接收到`Sidekiq::Shutdown`而引发异常。通过此检测，只有由应用程序代码内的错误或其他瞬态错误引起的错误才会触发重试。这可以避免不必要地污染监控工具，并且不会强制良好行为的作业进入重试集。出于简洁起见，我们将跳过`#exception_caused_by_shutdown?`的内部实现，只提到其秘密酱汁是Ruby的[`exception#cause`](https://rubyapi.org/3.3/o/exception#method-i-cause)方法。

接下来，Sidekiq检查是否明确配置了不重试作业 - 如果是这样，它将简单地重新引发异常。默认情况下，每个Sidekiq作业最多重试25次。

在我们继续执行`#process_retry`之前，我们需要看最后一行：`raise Skip`。Sidekiq不会重新引发原始异常，而是引发`JobRetry::Handled`的子类。一旦我们到达`#global`方法，我们将看到它与之相关。

原始的`#process_retry`方法处理元数据管理和格式化；以下代码片段是其精简版本：

```
# JobRetry

def process_retry(jobinst, msg, queue, exception)
  max_retry_attempts = retry_attempts_from(msg["retry"], @max_retries)

  msg["queue"] = (msg["retry_queue"] || queue)

  # Code that updates the `retry_count` job payload attribute, saves the `retried_at` and `failed_at` timestamps, filters and saves the backtrace, etc
  count = msg["retry_count"] # Increment logic of this value is omitted

  return retries_exhausted(jobinst, msg, exception) if count >= max_retry_attempts

  rf = msg["retry_for"]
  return retries_exhausted(jobinst, msg, exception) if rf && ((msg["failed_at"] + rf) < Time.now.to_f)

  strategy, delay = delay_for(jobinst, count, exception, msg)
  case strategy
  when :discard
    return
  when :kill
    return retries_exhausted(jobinst, msg, exception)
  end

  jitter = rand(10) * (count + 1)
  retry_at = Time.now.to_f + delay + jitter
  payload = JSON.parse(msg)
  redis do |conn|
    conn.zadd("retry", retry_at.to_s, payload)
  end
end 
```

首先，请注意Sidekiq提供了一个能力，即在作业失败时将其重新路由到不同的队列；这就是代码的第二行负责的内容。

立即之后，如果作业超过其最大重试次数或应该重试的时间段已过去，将调用`#retries_exhausted`。

如果Sidekiq应尝试重试作业，则调用`#delay_for`。此方法返回一个策略和延迟（以秒为单位）。我们将看到策略值来自何处，但现在理解作业可以是`discard`或`kill`，前者导致Sidekiq完全忘记作业而不是重试。

如果确实应该重试作业，则根据获得的延迟计算应该发生重试的时间戳。对此值应用抖动以避免雷鸣般的大群作业在同一秒钟安排重试的问题。

最后，使用`ZADD`将作业添加到`retry`排序集中，遵循与死作业类似的过程。

```
# JobRetry
def delay_for(jobinst, count, exception, msg)
  rv = begin
    block = jobinst&.sidekiq_retry_in_block
    # ...
    block&.call(count, exception, msg)
  rescue Exception => e
    # ...
    nil
  end

  rv = rv.to_i if rv.respond_to?(:to_i)
  delay = (count**4) + 15
  if Integer === rv && rv > 0
    delay = rv
  elsif rv == :discard
    return [:discard, nil]
  elsif rv == :kill
    return [:kill, nil]
  end

  [:default, delay]
end 
```

`#delay_for`从开发人员在作业类上定义的可选`sidekiq_retry_in_block`块中获取`strategy`。它可以返回表示策略的符号，或表示延迟秒数的整数值。

如果延迟值不是由开发人员提供的，则使用指数补偿公式计算，其中`count`是已经执行的重试次数。默认的最大重试次数为25次，这意味着一个作业（没有定义自定义`sidekiq_retry_in_block`）将在大约20天内重试25次。

下面是`#retries_exhausted`方法中发生的事情：

```
# JobRetry

def retries_exhausted(jobinst, msg, exception)
  rv = begin
    block = jobinst&.sidekiq_retries_exhausted_block
    # ...
    block&.call(msg, exception)
  rescue => e
    # Log the error and do not reraise it
    # ...
  end

  return if rv == :discard
  unless msg["dead"] == false
    payload = Sidekiq.dump_json(msg)
    now = Time.now.to_f

    redis do |conn|
      conn.multi do |xa|
        xa.zadd("dead", now.to_s, payload)
        xa.zremrangebyscore("dead", "-inf", now - @capsule.config[:dead_timeout_in_seconds])
        xa.zremrangebyrank("dead", 0, - @capsule.config[:dead_max_jobs])
      end
    end
  end
end 
```

另一个在此处使用的可选用户提供的作业级配置是`sidekiq_retries_exhausted_block`，它允许在其重试耗尽时丢弃作业。如果未提供，作业将以与`Processor#process`中作业有效载荷格式错误时相同的方式被送入太平间。

重试器中的最后一个相关方法是`#global`。值得记住的是，它包装了对`#local`、`Processor#reloader`、中间件调用以及`Processor#process`管道中的几乎每一步的调用。它需要捕获在处理作业时引发的任何异常，包括那些不是从应用程序代码中引发的异常，尽力而为。它与`#local`的唯一区别在于，如果作业没有设置`retry`属性，或者该属性在作业类上设置为`false`，它会丢弃该作业：

```
# JobRetry

def global(jobstr, queue)
  yield
rescue Handled => ex
  raise ex
rescue Sidekiq::Shutdown => ey
  raise ey
rescue Exception => e
  raise Sidekiq::Shutdown if exception_caused_by_shutdown?(e)

  msg = Sidekiq.load_json(jobstr)
  if msg["retry"]
    process_retry(nil, msg, queue, e)
  else
    @capsule.config.death_handlers.each do |handler|
      handler.call(msg, e)
    rescue => handler_ex
      handle_exception(handler_ex, {context: "Error calling death handler", job: msg})
    end
  end

  raise Handled
end 
```

你可能会注意到，与`#local`类似，这种方法也会拯救`JobRetry::Handled`，并且在这种情况下不会重试，因为这些异常已经在引发`#local`救援块的过程中得到处理，而引发了`JobRetry::Skip` - 一个`Handled`的子类。这避免了重复的重试处理。

我们现在已经探讨了Sidekiq在服务器上处理作业的主要过程。但是，那些需要重试的作业呢？此外，Sidekiq是否提供了一种机制来延迟执行作业直到将来的某个指定时间？我们将在下一部分中学习Sidekiq如何处理这些情况。

## 排队作业

到目前为止，我们已经看到了Sidekiq服务器的内部工作：`Launcher`、`Manager`和`Processor`结构，它们在负责作业处理的后台进程中运行。现在，让我们探索作业如何进入队列，这将引导我们进入`Sidekiq::Client`。

尽管大多数Sidekiq用户通常不直接与`Sidekiq::Client`交互，而更喜欢像[作业API](https://github.com/sidekiq/sidekiq/blob/c1607198815d68f60d138010907dd3426d6521bb/lib/sidekiq/job.rb)这样的高级抽象，但是该模块提供的所有便利方法都依赖于基本构建块：`Client#push`和`Client#push_bulk`，它们接受不同的参数组合。

让我们看看`Client#push`做了什么：

```
# Client

def push(item)
  normed = normalize_item(item)
  payload = middleware.invoke(item["class"], normed, normed["queue"], @redis_pool) do
    normed
  end
  if payload
    verify_json(payload)
    raw_push([payload])
    payload["jid"]
  end
end 
```

`normalize_item` 对有效负载执行多个操作，包括参数验证和生成唯一作业标识符。

随后，有效负载通过客户端中间件链进行处理，反映了 `Processor` 将作业有效负载通过服务器中间件发送的方式。这使得外部集成可以通过额外属性增强作业有效负载，并在作业分派周围实现自定义逻辑。

`verify_json` 可能不需要额外的解释，因为它确实做了其名称所示的事情，但它是如何做到的使得它值得深入研究：

```
def verify_json(item)
  job_class = item["wrapped"] || item["class"]
  args = item["args"]
  mode = Sidekiq::Config::DEFAULTS[:on_complex_arguments]

  if mode == :raise || mode == :warn
    if (unsafe_item = json_unsafe?(args))
      msg = <<~EOM Job arguments to #{job_class} must be native JSON types, but #{unsafe_item.inspect} is a #{unsafe_item.class}.
        See https://github.com/sidekiq/sidekiq/wiki/Best-Practices
        To disable this error, add `Sidekiq.strict_args!(false)` to your initializer.
 EOM

      if mode == :raise
        raise(ArgumentError, msg)
      else
        warn(msg)
      end
    end
  end
end

# ...

RECURSIVE_JSON_UNSAFE = {
  Integer => ->(val) {},
  Float => ->(val) {},
  TrueClass => ->(val) {},
  FalseClass => ->(val) {},
  NilClass => ->(val) {},
  String => ->(val) {},
  Array => ->(val) {
    val.each do |e|
      unsafe_item = RECURSIVE_JSON_UNSAFE[e.class].call(e)
      return unsafe_item unless unsafe_item.nil?
    end
    nil
  },
  Hash => ->(val) {
    val.each do |k, v|
      return k unless String === k

      unsafe_item = RECURSIVE_JSON_UNSAFE[v.class].call(v)
      return unsafe_item unless unsafe_item.nil?
    end
    nil
  }
}

RECURSIVE_JSON_UNSAFE.default = ->(val) { val }
RECURSIVE_JSON_UNSAFE.compare_by_identity
private_constant :RECURSIVE_JSON_UNSAFE

def json_unsafe?(item)
  RECURSIVE_JSON_UNSAFE[item.class].call(item)
end 
```

简而言之，`verify_json` 方法验证由 `#perform` 方法期望的作业参数，以确保它们符合有效的 JSON 格式。在 Sidekiq 7.0 之前的版本中，`json_unsafe?` 方法通过验证作业在转储和解析为 JSON 后是否保持不变来进行简单检查：`JSON.parse(JSON.dump(item["args"])) == item["args"]`。然而，这种方法涉及对每个作业的可能大参数进行转储和解析，显然需要性能优化。

为了解决这个问题，引入了 `RECURSIVE_JSON_UNSAFE` 常量。额外的改进在于 `RECURSIVE_JSON_UNSAFE.compare_by_identity`，它使得哈希基于它们的 `object_id` 来解析键对象，而不是调用 [`hash`](https://rubyapi.org/3.3/o/object#method-i-hash)，从而加快了处理速度。提取一个在对象生命周期内保持不变的静态 `object_id`，尽管是递归的，比经历两次 JSON 处理要快得多。

在这个新版本中，已知为“JSON-safe”的每个 Ruby 类返回一个执行时返回 `nil` 的过程。数组和哈希表接受特殊处理，因为它们包含其他对象，使得这个哈希表递归。如果作业有效负载包含未在哈希表中声明为键的任何其他类的对象，则认为参数不安全，并将采取适当的操作，其默认行为是引发错误。

`raw_push` 是作业放入 Redis 的地方：

```
# Client

def raw_push(payloads)
  @redis_pool.with do |conn|
    retryable = true
    begin
      conn.pipelined do |pipeline|
        atomic_push(pipeline, payloads)
      end
    rescue RedisClient::Error => ex
      if retryable && ex.message =~ /READONLY|NOREPLICAS|UNBLOCKED/
        conn.close
        retryable = false
        retry
      end
      raise
    end
  end
  true
end

def atomic_push(conn, payloads)
  if payloads.first.key?("at")
    conn.zadd("schedule", payloads.flat_map { |hash|
      at = hash.delete("at").to_s
      # ...
      [at, JSON.generate(hash)]
    })
  else
    queue = payloads.first["queue"]
    now = Time.now.to_f
    to_push = payloads.map { |entry|
      entry["enqueued_at"] = now
      JSON.generate(entry)
    }
    conn.sadd("queues", [queue])
    conn.lpush("queue:#{queue}", to_push)
  end
end 
```

这个方法接受一个有效负载数组来支持 `#push_bulk`。`#push` 以单个数组元素作为参数。

救援措施被包含在内，以处理需要重新打开连接套接字的 Redis 故障转移，尝试进行一次透明重试。

随后，`atomic_push` 调用的 Redis 命令是通过 [pipelining](https://redis.io/docs/manual/pipelining/) 进行的，减少了往返时间（RTT）。需要注意的是，当作业批量推送时，所有作业都使用单个 `LPUSH` 命令提交，因此此处的流水线处理并不像可能的那样关键。

`atomic_push`有两个分支。第二个分支是为了尽快处理作业的立即调度。[`SADD`](https://redis.io/commands/sadd/)命令用于存储用于监视目的的队列集合，并不一定与作业处理本身相关。然而，随后的[`LPUSH`](https://redis.io/commands/lpush/)命令非常关键，因为它将作业放入列表中，这些列表会由服务器端的`Processor`实例定期获取，这一点我们已经讨论过了。一个重要的细节是，`LPUSH`将元素添加到列表的前面 - `BasicFetch#retrieve_work`从尾部弹出元素，使得Sidekiq队列采用FIFO模型。

在这里必须强调的是，尽管队列的执行顺序是先进先出的，但Sidekiq不保证作业执行的顺序。在另一个处理器处理完之前，后续的作业可能会从队列列表中弹出。然而，可以通过设置特定胶囊的`concurrency`设置为1来确保每次只有一个`Processor`是活动的，使得作业获取是顺序的。然而，这仍然不能保证在作业失败并发送到重试集合，或者前面的作业丢失的情况下的顺序。我们将在下一节中探讨后一种情况可能发生的时机。无论如何，如果确实需要严格的作业顺序，Sidekiq并不是最佳选择。

`atomic_push`的第一个分支用于容纳预定的作业，这是我们到目前为止还没有涉及过的部分。预定的作业允许用户推迟作业的执行，直到将来的特定时间点。为了实现这一点，再次使用`ZADD`与排序集合类似于`dead`和`retry`集合。

## 重试与调度

到目前为止，我们已经探索了大部分服务器的内部结构，也已经检查了`Sidekiq#client`如何将作业推送到队列中，包括将作业安排在将来执行的选项。然而，我们还没有探索能够处理预定作业和需要重试的作业的最终组件。我们注意到，这两种类型的作业都使用`ZADD`添加到相应的排序集合中，但它们如何被处理呢？

当我们之前检查`Launcher`时，我们有意忽略了`Sidekiq::Scheduled::Poller`。每当一个Sidekiq进程启动时，它调用`Launcher#run`，这将启动轮询线程。

```
def start
  @thread ||= safe_thread("scheduler") {
    initial_wait

    until @done
      enqueue
      wait
    end
  }
end 
```

它的主循环与`Processor`非常相似，因此我不会详细介绍`@done`的设置在哪里。

在我们深入研究 `initial_wait`、`enqueue` 和 `wait` 方法的细节之前，让我们退一步。基于我们迄今所见，我们可以自信地说多个 Sidekiq 进程根本不需要协调。使用单个 Redis 实例并启动多个 Sidekiq 进程是常见做法，每个进程共享相同的队列集，以提高吞吐量和可用性。这些进程只需原子地从队列列表中 `BRPOP` 作业，每个处理器都会得到其工作的份额。

然而，`scheduled` 和 `retry` 集合的操作方式不同。这些集合中的作业不应立即处理；它们会在处理时间到来时由 `Processor` 推送到各自的队列中。`Poller` 处理这个任务，但如果有多个 Sidekiq 进程怎么办？每个进程都有自己的轮询线程竞争访问共享的 Redis 集合。虽然这对于正常处理队列列表不是问题，因为 `BRPOP` 操作需要恒定的时间，但从 `scheduled` 或 `retry` 排序集合中提取所需键具有不同的计算复杂度。从多个进程向 Redis 连续发送轮询器使用的命令会对其健康和性能产生负面影响。因此，轮询器必须要么协调以确保只有一个运行，要么它们的执行必须人为地分散开来。Sidekiq 选择了后者，并且我们将探讨它是如何实现的。

`Sidekiq::Scheduled` 是 Sidekiq 中被评论最多的模块之一，所以所有原始评论都在以下片段中保留。这里是每个 Sidekiq 进程调用一次的 `#initial_wait`：

```
# Scheduled::Poller

INITIAL_WAIT = 10

# ...

def initial_wait
  # Have all processes sleep between 5-15 seconds. 10 seconds to give time for
  # the heartbeat to register (if the poll interval is going to be calculated by the number
  # of workers), and 5 random seconds to ensure they don't all hit Redis at the same time.
  total = 0
  total += INITIAL_WAIT unless @config[:poll_interval_average]
  total += (5 * rand)

  @sleeper.pop(total)
rescue Timeout::Error
ensure
  # periodically clean out the `processes` set in Redis which can collect
  # references to dead processes over time. The process count affects how
  # often we scan for scheduled jobs.
  cleanup
end 
```

心跳在第一个评论中提到 - 与轮询器类似，它是另一个定期更新存储在 Redis 中的某些进程元数据的线程。对于轮询器而言，唯一相关的是更新包含每个 Sidekiq 进程唯一标识符的 `processes` 集合，我们很快会看到它如何确切地发挥作用。

`@sleeper.pop(total)` 简单地使线程等待指定的时间量。

```
# Scheduled::Poller

def cleanup
  # dont run cleanup more than once per minute
  return 0 unless redis { |conn| conn.set("process_cleanup", "1", "NX", "EX", "60") }

  count = 0
  redis do |conn|
    procs = conn.sscan("processes").to_a
    heartbeats = conn.pipelined { |pipeline|
      procs.each do |key|
        pipeline.hget(key, "info")
      end
    }

    # the hash named key has an expiry of 60 seconds.
    # if it's not found, that means the process has not reported
    # in to Redis and probably died.
    to_prune = procs.select.with_index { |proc, i|
      heartbeats[i].nil?
    }
    count = conn.srem("processes", to_prune) unless to_prune.empty?
  end

  count
end 
```

如图所示，轮询器负责清理从 Redis 中删除死进程的元数据。尽管 `#initial_wait` 中的注释说的是一旦轮询器启动就会发生，但在正常情况下，它只会在进程启动时启动。

为什么这是必要的，可以在被称为的 `#wait` 方法中看到，该方法在每次 `#enqueue` 后都被调用：

```
# Scheduled::Poller

def wait
  @sleeper.pop(random_poll_interval)
rescue Timeout::Error
  # expected
rescue => ex
  # if poll_interval_average hasn't been calculated yet, we can
  # raise an error trying to reach Redis.
  logger.error ex.message
  handle_exception(ex)
  sleep 5
end

def random_poll_interval
  # We want one Sidekiq process to schedule jobs every N seconds.  We have M processes
  # and **don't** want to coordinate.
  #
  # So in N*M second timespan, we want each process to schedule once.  The basic loop is:
  #
  # * sleep a random amount within that N*M timespan
  # * wake up and schedule
  #
  # We want to avoid one edge case: imagine a set of 2 processes, scheduling every 5 seconds,
  # so N*M = 10\.  Each process decides to randomly sleep 8 seconds, now we've failed to meet
  # that 5 second average. Thankfully each schedule cycle will sleep randomly so the next
  # iteration could see each process sleep for 1 second, undercutting our average.
  #
  # So below 10 processes, we special case and ensure the processes sleep closer to the average.
  # In the example above, each process should schedule every 10 seconds on average. We special
  # case smaller clusters to add 50% so they would sleep somewhere between 5 and 15 seconds.
  # As we run more processes, the scheduling interval average will approach an even spread
  # between 0 and poll interval so we don't need this artifical boost.
  #
  count = process_count
  interval = poll_interval_average(count)

  if count < 10
    # For small clusters, calculate a random interval that is ±50% the desired average.
    interval * rand + interval.to_f / 2
  else
    # With 10+ processes, we should have enough randomness to get decent polling
    # across the entire timespan
    interval * rand
  end
end

# We do our best to tune the poll interval to the size of the active Sidekiq
# cluster.  If you have 30 processes and poll every 15 seconds, that means one
# Sidekiq is checking Redis every 0.5 seconds - way too often for most people
# and really bad if the retry or scheduled sets are large.
#
# Instead try to avoid polling more than once every 15 seconds.  If you have
# 30 Sidekiq processes, we'll poll every 30 * 15 or 450 seconds.
# To keep things statistically random, we'll sleep a random amount between
# 225 and 675 seconds for each poll or 450 seconds on average.  Otherwise restarting
# all your Sidekiq processes at the same time will lead to them all polling at
# the same time: the thundering herd problem.
#
# We only do this if poll_interval_average is unset (the default).
def poll_interval_average(count)
  @config[:poll_interval_average] || scaled_poll_interval(count)
end

# Calculates an average poll interval based on the number of known Sidekiq processes.
# This minimizes a single point of failure by dispersing check-ins but without taxing
# Redis if you run many Sidekiq processes.
def scaled_poll_interval(process_count)
  process_count * @config[:average_scheduled_poll_interval]
end

def process_count
  pcount = Sidekiq.redis { |conn| conn.scard("processes") }
  pcount = 1 if pcount == 0
  pcount
end 
```

由于详尽的评论，轮询间隔逻辑得到了详细解释。在计算轮询器应该与 Redis 联系的间隔时，考虑了从同一 Redis 安装中消耗作业的活动 Sidekiq 进程数量。这归结为最小化并发轮询器操作。

最后，让我们看看所有这些努力是为了什么。`Poller#enqueue` 实例化 `Sidekiq::Scheduled::Enq`，封装了预定出队逻辑，并在其上调用 `#enqueue_jobs`：

```
class Enq
  LUA_ZPOPBYSCORE = <<~LUA local key, now = KEYS[1], ARGV[1]
    local jobs = redis.call("zrange", key, "-inf", now, "byscore", "limit", 0, 1)
    if jobs[1] then
      redis.call("zrem", key, jobs[1])
      return jobs[1]
    end LUA

  # ...

  def enqueue_jobs(sorted_sets = %w[retry schedule])
    # A job's "score" in Redis is the time at which it should be processed.
    # Just check Redis for the set of jobs with a timestamp before now.
    redis do |conn|
      sorted_sets.each do |sorted_set|
        # Get next item in the queue with score (time to execute) <= now.
        # We need to go through the list one at a time to reduce the risk of something
        # going wrong between the time jobs are popped from the scheduled queue and when
        # they are pushed onto a work queue and losing the jobs.
        while !@done && (job = zpopbyscore(conn, keys: [sorted_set], argv: [Time.now.to_f.to_s]))
          @client.push(JSON.generate(job))
        end
      end
    end
  end

  # ...

  private

  def zpopbyscore(conn, keys: nil, argv: nil)
    if @lua_zpopbyscore_sha.nil?
      @lua_zpopbyscore_sha = conn.script(:load, LUA_ZPOPBYSCORE)
    end

    conn.call("EVALSHA", @lua_zpopbyscore_sha, keys.size, *keys, *argv)
  rescue RedisClient::CommandError => e
    raise unless e.message.start_with?("NOSCRIPT")

    @lua_zpopbyscore_sha = nil
    retry
  end
end 
```

首先要注意的是`LUA_ZPOPBYSCORE`字符串，以及[`SCRIPT`](https://redis.io/commands/script-load/)和[`EVALSHA`](https://redis.io/commands/evalsha/) Redis命令。这是Sidekiq利用Redis的一个[特性](https://redis.io/docs/interact/programmability/eval-intro/)，允许在服务器上执行原子Lua脚本。该脚本通过`conn.script`预加载和缓存在Redis中，以便后续使用它的命令可以获得增加的性能。

脚本本身接受一组集合（`retry`和`schedule`）以及当前时间戳（请注意Lua数组索引从1开始）。然后调用[`ZRANGE`](https://redis.io/commands/zrange/)，返回应该分派到队列的具有最低时间戳的单个作业。然后使用[`ZREM`](https://redis.io/commands/zrem/)从集合中移除作业元素。

`ZRANGE`的复杂度为O(log(N) + M)，其中N是集合中的总元素数，M是返回的元素数。然而，如评论所示，Sidekiq每次只出列一个计划的或待重试的作业，以确保安全，这意味着总体复杂度将为O(M * log(N))。

`ZREM`本身已经是O(M * log(N))，所以整体复杂度不会因为整个脚本对每个相关作业的调用而改变。

对于每个成功出列的作业，将调用`Client#push`将作业分派到其目标队列。

除此之外，Redis大多数情况下是单线程的，这清楚地说明了为什么在时间上分散轮询器如此重要。通过几个进程和足够大的重试或计划集（根据我的经验，重试集大小很容易达到数千个甚至更多），并发执行这个逻辑可以显著地影响Redis的性能，甚至使其停滞不前。

## 作业丢失的潜力

现在我们已经探讨了作业在服务器处理过程中的完整生命周期，让我们检查可能导致作业丢失的潜在场景。

我们知道Sidekiq的操作系统版本只提供`BasicFetch`作为从Redis出列作业的方法，这有效地从相应的队列列表中删除它们。这意味着如果Sidekiq进程突然终止，例如使用SIGKILL，所有正在进行中的作业都将丢失。例如，如果进程耗尽内存，促使Kubernetes强制使用`kill -9`命令终止pod，或者如果其他编排器和托管提供商采取类似的操作。

那么在优雅终止期间，Sidekiq如何管理挂起的、长时间运行的或者简单地正在进行中的作业呢？

让我们重新审视`Manager#hard_shutdown`，当一个Sidekiq进程接收到SIGTERM或SIGINT信号时会调用它：

```
# Manager

def hard_shutdown
  cleanup = nil
  @plock.synchronize do
    cleanup = @workers.dup
  end

  if cleanup.size > 0
    jobs = cleanup.map { |p| p.job }.compact

    capsule.fetcher.bulk_requeue(jobs)
  end

  cleanup.each do |processor|
    processor.kill
  end

  # ...
end 
```

`capsule.fetcher.bulk_requeue(jobs)` 在处理器线程终止之前被调用。此时，等待它们处理`Sidekiq::Shutdown`异常并优雅地退出是不可行的。^(因此，最佳选择是重新排队所有正在进行的作业，即使这意味着一些在进程退出之前完成的作业也将被重新排队。)

`BasicFetch#bulk_requeue` 简单地将从活动处理器中提取的作业载荷放回相应的Redis列表：

```
# BasicFetch

def bulk_requeue(inprogress)
  return if inprogress.empty?

  # ...
  jobs_to_requeue = {}
  inprogress.each do |unit_of_work|
    jobs_to_requeue[unit_of_work.queue] ||= []
    jobs_to_requeue[unit_of_work.queue] << unit_of_work.job
  end

  redis do |conn|
    conn.pipelined do |pipeline|
      jobs_to_requeue.each do |queue, jobs|
        pipeline.rpush(queue, jobs)
      end
    end
  end
  # ...
end 
```

`BasicFetch`的重新排队逻辑是可能导致作业丢失的一个场景。Redis在处理作业时可能会耗尽内存，从而无法将作业放回队列。这对于普通的重试也是一个问题。

此外，虽然与Sidekiq没有直接关系，但由于特定的Redis服务器配置错误，作业可能会丢失。[驱逐策略](https://redis.io/docs/reference/eviction/)在此处起到了重要作用，它决定了Redis在内存满时的处理方式。除了`noevict`之外的任何设置，在特定情况下都可能导致作业从Redis中被驱逐出去。此外，Redis在重新启动后可能会遇到故障并丢失数据，特别是在某些[persistence](https://redis.io/docs/management/persistence/)配置下。这种情况可能导致Sidekiq客户端错误地假设成功推送后作业将被处理，而实际上它可能根本没有到达Sidekiq服务器。

还有可能是在预定义超时期间内未执行SIGTERM或SIGINT处理程序，Sidekiq运行的编排程序在发送SIGKILL之前等待这一超时。这里最可能的罪魁祸首是Redis连接问题。理论上，GC暂停和主机机器的状态也可能在此关键任务期间阻碍Sidekiq进程，但我个人从未遇到过，也从未听说过25秒内Ruby程序被卡住的情况。

毫无疑问，如果在未发送SIGINT或SIGTERM的情况下使用SIGKILL终止Sidekiq进程，所有正在进行的作业都将丢失。

应注意，当前设计`BasicFetch`的决策是有意为之的。通过使用单个`BRPOP`命令来获取作业，Sidekiq在Redis的负载方面保持极其简单和轻量化。然而，这也意味着如果您的作业绝对不能丢失，仅依赖于Sidekiq可能不是最佳选择。尽管如此，可能可以重构应用程序代码和业务逻辑，需要强大的耐久性保证，包括一个像是定期cron作业的活跃组件，它会重复地将作业加入队列，直到达到期望的最终状态。

这使得 Sidekiq 采用了“至少一次”交付语义，系统健康严重降级时可能会丢失作业。然而，专业版确保作业不会丢失，因为作业在未使用 `SuperFetch` 确认之前永远不会离开 Redis，有效地使其真正成为“至少一次”而没有任何附带条件。

Sidekiq 利用简单的 Redis 数据结构，如列表，具有常数时间的推送和弹出操作，以及可调整的线程数，实现了可伸缩性、效率和易管理性。在操作系统版本中，扩展可以涉及部署多个 Sidekiq 进程，每个进程从同一 Redis 安装中消耗作业。企业版提供了一个分叉模型用于多进程执行，提供了一种可扩展的方式。为了对队列进行精细控制，可以使用胶囊来组织具有特定并发设置的处理。此外，您可以优先处理某些队列，强制执行严格的队列处理顺序（与作业排序不同），或均匀轮询队列。但是，需要注意的是，队列内的作业排序不受支持。尽管如此，在每个队列使用单个处理器可能会提供一种基本的排序形式，这可能根据您的应用需求而足够。
