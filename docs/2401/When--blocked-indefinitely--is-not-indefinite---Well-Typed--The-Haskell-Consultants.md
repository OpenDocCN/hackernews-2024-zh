<!--yml

类别：未分类

日期：2024-05-27 14:46:54

-->

# 当“无限期阻塞”不是无限期 - Well-Typed：Haskell 顾问

> 来源：[`well-typed.com/blog/2024/01/when-blocked-indefinitely-is-not-indefinite/`](https://well-typed.com/blog/2024/01/when-blocked-indefinitely-is-not-indefinite/)

考虑一个试图从[TMVar](https://hackage.haskell.org/package/stm-2.5.3.0/docs/Control-Concurrent-STM-TMVar.html)中读取的 Haskell 线程：

```
x <- atomically $ takeTMVar v
```

如果`TMVar`当前为空，并且没有其他可能写入`TMVar`的线程，那么这个线程将永远无法取得进展。 GHC 运行时检测到这种情况，并且对`atomically`的这个调用将抛出一个[BlockedIndefinitelyOnSTM](https://hackage.haskell.org/package/base-4.19.0.0/docs/Control-Exception.html#t:BlockedIndefinitelyOnSTM)异常，显示为

```
thread blocked indefinitely in an STM transaction
```

**然而，偶尔，运行时会在进展*可能*时抛出此异常。**

这篇博文并不是第一次提出这个观察；Simon Marlow 的书籍《[Haskell 并行与并发编程](https://simonmar.github.io/pages/pcph.html)》中讨论过这个问题，它偶尔会在各种票证中提到（例如[GHC #9401](https://gitlab.haskell.org/ghc/ghc/-/issues/9401)，[GHC #10241](https://gitlab.haskell.org/ghc/ghc/-/issues/10241)，[Async #14](https://github.com/simonmar/async/issues/14)）。尽管如此，这个问题并不像可能应该的那样广为人知，它可能导致非常令人困惑的行为。因此，在这篇博文中，我们将研究何时以及如何出现这种情况，以及我们可以对此采取什么措施。

*术语说明。* 在 Haskell 文献中，像这样无法在此类情况下取得进展的线程通常被称为“死锁”（例如，Simon Marlow 的书中的[检测死锁](https://www.oreilly.com/library/view/parallel-and-concurrent/9781449335939/ch15.html#sec_deadlock)一节，或者在`Control.Concurrent`文档中的[死锁](https://hackage.haskell.org/package/base-4.19.0.0/docs/Control-Concurrent.html#g:14)一节）。然而，[传统上](https://en.wikipedia.org/wiki/Deadlock)，“死锁”一词指的是一组线程无法取得进展，因为它们都在互相等待*对方*；但这种情况并不一定会发生在这里，因此我们将在这篇博文中避免使用“死锁”一词，而是将无法取得进展的线程称为“停滞”。

## 例子

考虑一个具有三个线程的应用程序。

1.  第一个线程模拟低级网络库（如[http2](https://hackage.haskell.org/package/http2)）；我们假设它从网络接口解码消息并将它们可用于`TMVar`。

1.  第二个线程模拟更高级的网络库（如[grapesy](https://github.com/well-typed/grapesy)，一个[gRPC](https://grpc.io/)库）；为了我们的目的，我们假设它读取`TMVar`中的消息并将其写入`TQueue`，提供某种缓冲。

1.  第三个线程模拟应用程序层；在这里，我们只是假设它从`TQueue`中读取并将所有消息写入终端。

换句话说，设置看起来像这样：

```
/--------\          /--------\            /-------\
|        |   TMVar  |        |   TQueue   |       |
| decode | <------> | buffer | <--------> | write |
|        |          |        |            |       |
\--------/          \--------/            \-------/
```

对于`decode`的实现，我们将简单地每两秒写入一条新的“消息”：

```
decode :: TMVar Int -> IO ()
decode decoderOutput =
 forM_ [1..] $ \i -> do
 threadDelay 2_000_000
 atomically $ putTMVar decoderOutput i
```

对于`buffer`，我们希望等待每个消息，将其入队，并循环；然而，我们还想检测是否有任何异常被抛出，如果是，则也将其写入队列：

```
newtype NetworkFailure = NetworkFailure SomeException
 deriving (Show)

buffer :: TMVar Int -> TQueue (Either NetworkFailure Int) -> IO ()
buffer decoderOutput queue =
 loop
 where
 loop :: IO ()
 loop = do
 mMsg <- try $ atomically $ takeTMVar decoderOutput
 case mMsg of
 Left err -> do
 atomically $ writeTQueue queue $ Left (NetworkFailure err)
 putStrLn $ "buffer exiting: " ++ show err
 Right msg -> do
 atomically $ writeTQueue queue $ Right msg
 loop
```

最后，在`write`中，我们等待消息，将其打印到终端，并循环，除非我们看到有异常报告：

```
write :: TQueue (Either NetworkFailure Int) -> IO ()
write queue =
 loop
 where
 loop :: IO ()
 loop = do
 mMsg <- atomically $ readTQueue queue
 case mMsg of
 Left (NetworkFailure err) ->
 putStrLn $ "Network failure: " ++ show err
 Right msg -> do
 putStrLn $ "Incoming message: " ++ show msg
 loop
```

## 当解码器死掉

假设我们启动所有三个线程运行：`decode`每两秒向`TMVar`写入一条新消息，`buffer`将所有这些消息复制到`TQueue`中，而`write`则将它们出队并写入终端——*然后我们杀死解码器线程*（例如，这可能模拟网络故障）。

由于没有人再向`TMVar`写入，因此`buffer`中的`takeTMVar`无法取得进展，并且会抛出“无限期阻塞”异常；此异常被捕获后，以`NetworkFailure BlockedIndefinitelyOnSTM`的形式写入`TQueue`，并且`buffer`干净地终止。到目前为止还不错。

但是，`write`线程无法成功出队这个`NetworkFailure`；相反，当它从`TQueue`中读取时，它自己被一个`BlockedIndefinitelyOnSTM`杀死。这非常令人困惑；我们*看到*将消息写入了`TQueue`，那么为什么`readTQueue`被认为是无限期阻塞呢？确实，如果我们用以下代码替换`write`中的`atomically`调用，情况会如何？

```
atomicallyStubborn :: forall a. STM a -> IO a
atomicallyStubborn stm = persevere
 where
 persevere :: IO a
 persevere =
 catch (atomically stm) $ \BlockedIndefinitelyOnSTM ->
 persevere
```

它仅在收到“无限期阻塞”异常时再次尝试事务，然后`write`线程*确实*看到了`NetworkFailure`被报告并干净地终止了。

重申一遍，我们从队列中读取一个`readTQueue`，我们*看到*有东西被写入了那个`TQueue`。然而，那个`readTQueue`却抛出了“无限期阻塞”异常，如果我们在收到异常后再次尝试读取*时*，读取成功了。这清楚地说明了线程*并没有*无限期阻塞，并且*可以*取得进展。那么为什么它被杀死了？

## 检测停滞线程

[停滞线程的检测是作为垃圾收集的一部分发生的](https://www.oreilly.com/library/view/parallel-and-concurrent/9781449335939/ch15.html#sec_deadlock)。`ghc`中的默认垃圾收集器（GC）使用标记-清除算法：它首先遍历堆，从一组*根*开始，标记从该组根可达的所有内容，然后收集（“删除”）未被标记的所有内容。作为第一个近似值，根的集合是*运行*（非阻塞）线程的集合。

要了解这与检测阻塞线程的关系，我们需要知道两个额外的信息。首先，线程和`TVar`本身是堆分配的对象（通过推论，`TMVar`和`TQueue`也是如此，它们是从`TVar`构建的）。其次，`TVar`有与之关联的线程列表，这些线程在其上被阻塞。

让我们首先考虑非异常情况，即`buffer`线程被阻塞（即不运行），正在等待`TMVar`，而`decode`线程正在运行，并且因此是 GC 根。当我们从`decode`线程开始遍历堆时，GC 将遇到`TMVar`；从那里，它将标记`buffer`线程，该线程记录为阻塞在该`TMVar`上的线程之一。由于标记了`buffer`线程，运行时得出结论它没有被阻塞。

然而，现在考虑一下当`decode`线程被终止时会发生什么情况。当 GC 标记堆时，*它永远不会遇到`buffer`线程*（从任何运行线程到`buffer`线程都没有路径）。在标记完所有内容后，GC 因此得出结论，`buffer`线程是不可访问的，因此必须被阻塞，它发送`BlockedIndefinitelyOnSTM`异常。换句话说：

> 如果没有来自**运行中**线程的引用，那么在`TVar`上被阻塞的线程被认为是无限期阻塞的。

问题在于`buffer`线程不是唯一无法访问的线程：`write`线程*也*被阻塞，同样无法从任何运行线程中访问。因此，GC 得出结论它*也*被阻塞，并因此发送了一个`BlockedIndefinitelyOnSTM`异常。`buffer`线程可以从这个异常中*恢复*，然后依次解除`write`线程的阻塞，对 GC 来说是看不见的。`write`线程在`buffer`线程的异常处理程序甚至有机会运行之前就收到了异常。

当然，*这是否是正确的*行为当然是可以争论的。[这当然是*预期的*行为](https://gitlab.haskell.org/ghc/ghc/-/issues/9401#note_86720)，并且更改它可能并不容易。在本博客的其余部分中，我们将考虑如何*与*这种行为一起工作以获得我们想要的结果。

## 解决方法

作为库的作者，*我们*知道`write`线程中从`TQueue`读取永远不会被无限期阻塞（只要我们在收到`NetworkFailure`后停止读取）。因此，如果我们可以从阻塞检测中排除该线程，那将是很好的。

一个选择是在`write`线程中使用类似`atomicallyStubborn`的东西。这样做是有效的，但有两个重要的缺点。首先，在我们的示例中，`write`线程是“应用层”；如果我们是中间库（`buffer`线程）的作者，我们将无法控制`write`线程。但也许我们可以提供一个 API，而不是直接暴露`TQueue`。

```
dequeue :: TQueue (Either NetworkFailure Int) -> IO (Either NetworkFailure Int)
dequeue queue = atomicallyStubborn $ readTQueue queue
```

但是，这并没有解决第二个更重要的问题。假设`write`线程不是将消息写入终端，而是将这些消息写入自己的`TMVar`：

```
/--------\          /--------\            /-------\          /-----\
|        |   TMVar  |        |   TQueue   |       |   TMVar  |     |
| decode | <------> | buffer | <--------> | write | <------> | ... |
|        |          |        |            |       |          |     |
\--------/          \--------/            \-------/          \-----/
```

如果现在我们有一个*第四个*线程从这个`TMVar`读取，*它也*必须使用`atomicallyStubborn`从那个`TMVar`读取，否则将再次面临完全相同的问题。

一个更好的解决方法是将`write`线程在被阻塞在`TQueue`上时视为 GC 根；这样它就不会被视为停顿，也不会有任何依赖于它的其他线程（例如上面示例中的第四个线程）被视为停顿。我们可以通过创建一个[稳定指针](https://hackage.haskell.org/package/base-4.19.0.0/docs/Foreign-StablePtr.html)到线程来做到这一点（这个解决方法是[due to Simon Marlow](https://gitlab.haskell.org/ghc/ghc/-/issues/10793)）：

```
write :: TQueue (Either NetworkFailure Int) -> IO ()
write queue = do
 void $ newStablePtr =<< myThreadId
 loop
 where
 loop = .. -- as before
```

或者，如果`write`被再次视为应用程序层的一部分，并且我们正在开发中间层，我们可以提供一个`dequeue`函数，例如

```
dequeue :: TQueue (Either NetworkFailure Int) -> IO (Either NetworkFailure Int)
dequeue queue = do
 tid <- myThreadId
 bracket (newStablePtr tid) freeStablePtr $ \_ ->
 atomically $ readTQueue queue
```

这里我们确保再次释放稳定指针，因为我们不想在用户的应用程序中*完全*关闭停顿检测。

## 结论

在我们运行的示例中，可以说*真正*的问题是`decode`线程中的网络故障导致了一次停顿。事实上，[Simon Marlow 写道](https://www.oreilly.com/library/view/parallel-and-concurrent/9781449335939/ch15.html#sec_deadlock)：

> 你不应该依赖死锁检测来保证程序的正确运行。死锁检测是一种调试特性；在发生死锁时，你会得到一个异常而不是静默挂起，但你应该尽量避免在你的程序中出现任何死锁。

在实践中，这并不总是容易实现；也许是因为停顿源自我们无法控制的库，或者仅仅因为修复问题很困难（例如，请参见[Http2 #97](https://github.com/kazu-yamamoto/http2/pull/97)，[Http2 #104](https://github.com/kazu-yamamoto/http2/pull/104)）；在这种情况下，有一个可靠的解决方法是很重要的。

出于我们上面解释的原因，我们不认为`atomicallyStubborn`是一个好的解决方法。然而，这种模式的变体偶尔是有用的，并且在实际中确实会出现；例如，它在`async`中被使用（[Async #14](https://github.com/simonmar/async/issues/14)），尽管不清楚为什么稳定指针解决方法在那里不起作用。

为了完整起见，我们想提及两个进一步的复杂情况。首先，终结器与停滞线程之间的交互是微妙的；请参阅[Control.Concurrent 文档中关于死锁的部分](https://hackage.haskell.org/package/base-4.19.0.0/docs/Control-Concurrent.html#g:14)（以及[GHC #11001](https://gitlab.haskell.org/ghc/ghc/-/issues/11001)）。其次，在*无限递归*和*停顿*的情况下，您可能无法获得您偏好的异常；请参阅[Shake #294](https://github.com/ndmitchell/shake/issues/294) / [GHC #10793](https://gitlab.haskell.org/ghc/ghc/-/issues/10793)。

这项工作由 Anduril 赞助，作为[grapesy](https://github.com/well-typed/grapesy)开发的一部分，这是一个新的用于 gRPC 的 Haskell 库。该库目前仍在开发中，但我们预计很快会发布第一个版本。届时，当然会在[Well-Typed 博客](https://well-typed.com/blog/)上发布相应的博客文章。

感谢 Justin Le、Ryan Brown 和 Kazu Yamamoto 对本博客文章草稿的评论。
