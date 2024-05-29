<!--yml
category: 未分类
date: 2024-05-27 14:46:54
-->

# When "blocked indefinitely" is not indefinite - Well-Typed: The Haskell Consultants

> 来源：[https://well-typed.com/blog/2024/01/when-blocked-indefinitely-is-not-indefinite/](https://well-typed.com/blog/2024/01/when-blocked-indefinitely-is-not-indefinite/)

Consider a Haskell thread trying to read from a [TMVar](https://hackage.haskell.org/package/stm-2.5.3.0/docs/Control-Concurrent-STM-TMVar.html):

```
x <- atomically $ takeTMVar v
```

If the `TMVar` is currently empty and there are no other threads that could write to the `TMVar`, then this thread will never be able to make progress. The GHC runtime detects such situations, and this call to `atomically` will throw a [BlockedIndefinitelyOnSTM](https://hackage.haskell.org/package/base-4.19.0.0/docs/Control-Exception.html#t:BlockedIndefinitelyOnSTM) exception, rendered as

```
thread blocked indefinitely in an STM transaction
```

**Occasionally, however, the runtime will throw this exception even when progress *is* possible.**

This blog post is not the first to make this observation; Simon Marlow’s book [Parallel and Concurrent Programming in Haskell](https://simonmar.github.io/pages/pcph.html) discusses it, and it occasionally comes up in various tickets (e.g. [GHC #9401](https://gitlab.haskell.org/ghc/ghc/-/issues/9401), [GHC #10241](https://gitlab.haskell.org/ghc/ghc/-/issues/10241), [Async #14](https://github.com/simonmar/async/issues/14)). Nonetheless, the problem is not as widely known as perhaps it should be, and it can lead to very confusing behaviour. In this blog post we will therefore examine when and how this can arise, and what we can do about it.

*Note on terminology.* In the Haskell literature a thread that cannot make progress in a situation like this is often referred to as “deadlocked” (for example, section [Detecting Deadlock](https://www.oreilly.com/library/view/parallel-and-concurrent/9781449335939/ch15.html#sec_deadlock) in Simon Marlow’s book, or the section on [Deadlock](https://hackage.haskell.org/package/base-4.19.0.0/docs/Control-Concurrent.html#g:14) in the documentation of `Control.Concurrent`). However, [traditionally](https://en.wikipedia.org/wiki/Deadlock) the term deadlock refers to a situation in which a group of threads cannot make progress because they are all waiting *on each other*; that need not be the case here, and so we will avoid the term “deadlock” in this blog post, instead referring to a thread that cannot make progress as “stalled”.

## Example

Consider an application with three threads.

1.  The first thread mimicks a low-level network library (such as [http2](https://hackage.haskell.org/package/http2)); we’ll assume it decodes messages from a network interface and makes them available on a `TMVar`.
2.  The second thread mimicks a higher-level networking library (such as [grapesy](https://github.com/well-typed/grapesy), a [gRPC](https://grpc.io/) library); for our purposes we’ll just assume that this reads the messages from the `TMVar` and writes them to a `TQueue`, providing some kind of buffering.
3.  The third thread mimicks the application layer; here we’ll just assume it’s reading from the `TQueue` and writes all messages to the terminal.

In other words, the setup looks something like this:

```
/--------\          /--------\            /-------\
|        |   TMVar  |        |   TQueue   |       |
| decode | <------> | buffer | <--------> | write |
|        |          |        |            |       |
\--------/          \--------/            \-------/
```

For the implementation of `decode`, we will simply write a new “message” every two seconds:

```
decode :: TMVar Int -> IO ()
decode decoderOutput =
 forM_ [1..] $ \i -> do
 threadDelay 2_000_000
 atomically $ putTMVar decoderOutput i
```

For `buffer` we want to wait for each message, enqueue it, and loop; however, we also want to detect if any exceptions are thrown, and if so, write those to the queue as well:

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

Finally, in `write` we wait for a message, print it to the terminal, and loop, unless we see an exception reported:

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

## When the decoder dies

Suppose we start all three threads running: `decode` is writing a new message to the `TMVar` every two seconds, `buffer` is copying all of these to the `TQueue`, and `write` is dequeueing them and writing them to the terminal – *and then we kill the decoder thread* (this might simulate a network failure, for example).

Since nobody is writing to the `TMVar` anymore, the `takeTMVar` in `buffer` cannot make progress and will throw a “blocked indefinitely” exception; this exception is caught, written to the `TQueue` as `NetworkFailure BlockedIndefinitelyOnSTM`, and `buffer` terminates cleanly. So far so good.

However, the `write` thread does not manage to dequeue this `NetworkFailure`; instead, it is killed with a `BlockedIndefinitelyOnSTM` of its own when it reads from the `TQueue`. This is extremely confusing; we can *see* that the write to the `TQueue` happens, so why is the `readTQueue` considered to be blocked indefinitely? Indeed, if we replace the call to `atomically` in `write` by

```
atomicallyStubborn :: forall a. STM a -> IO a
atomicallyStubborn stm = persevere
 where
 persevere :: IO a
 persevere =
 catch (atomically stm) $ \BlockedIndefinitelyOnSTM ->
 persevere
```

which simply attempts the transaction again when it receives the “blocked indefinitely” exception, then the `write` thread *does* see the `NetworkFailure` being reported and terminates cleanly.

To re-iterate, we have a `readTQueue` from a queue and we *see* something being written to that `TQueue`. Yet, that `readTQueue` throws a “blocked indefinitely” exception, and if we try to read *again* after receiving that exception, the read succeeds. This clearly illustrates that the thread was *not* blocked indefinitely, and *can* make progress. So why was it killed?

## Detection of stalled threads

The detection of stalled threads [happens as part of garbage collection](https://www.oreilly.com/library/view/parallel-and-concurrent/9781449335939/ch15.html#sec_deadlock). The default garbage collector (GC) in `ghc` uses a mark-and-sweep algorithm: it first traverses the heap, starting at a set of *roots*, marking everything that is reachable from that set of roots, and then collects (“deletes”) everything that was not marked. As a first approximation, the set of the roots is the set of *running* (non-blocked) threads.

To understand how this relates to the detection of stalled threads, we need to know two additional pieces of information. First, threads and `TVar`s are themselves heap allocated objects (and, by extension, so are `TMVar`s and `TQueue`s, which are built from `TVar`s). Second, `TVar`s have associated lists of threads that are blocked on them.

Let’s first consider the non-exceptional situation where the `buffer` thread is blocked (i.e., not running), waiting on the `TMVar`, and the `decode` thread is running and is therefore a GC root. As we start traversing the heap, starting at the `decode` thread, the GC will encounter the `TMVar`; from there, it will mark the `buffer` thread, which is recorded as one of the threads blocked on that `TMVar`. Since the `buffer` thread gets marked, the runtime concludes that it is not stalled.

However, now consider what happens when the `decode` thread is killed. When GC marks the heap, *it never encounters the `buffer` thread* (there is no path from any running thread to the `buffer` thread). After everything is marked, the GC therefore concludes that the `buffer` thread is unreachable, and hence that it must be stalled, and it sends it the `BlockedIndefinitelyOnSTM` exception. Put another way:

> A thread that is blocked on a `TVar` is considered blocked indefinitely *if there is no reference to that `TVar` from a **running** thread*.

The problem is that the `buffer` thread is not the only thread that is unreachable: the `write` thread is *also* blocked, and similarly cannot be reached from any running thread. The GC therefore concludes that it is *also* stalled, and sends it too a `BlockedIndefinitelyOnSTM` exception. The fact that the `buffer` thread can *recover* from this exception, and then in turn unblocks the `write` thread, is invisible to the GC. The `write` thread is sent the exception *before* the exception handler in the `buffer` thread even gets a chance to run.

Whether or not this is the *correct* behaviour can of course be argued. [It is certainly *expected* behaviour](https://gitlab.haskell.org/ghc/ghc/-/issues/9401#note_86720), and changing it may be non-trivial. In the remainder of this blog post we will consider how we can work *with* this behaviour to get the results we want.

## Workaround

As the library author, *we* know that a read from the `TQueue` in the `write` thread can *never* be blocked indefinitely (provided we stop reading after receiving a `NetworkFailure`). It would therefore be good if we could somehow exclude that thread from stall detection.

One option is to use something like `atomicallyStubborn` in the `write` thread. This works, but has two important downsides. First, in our example the `write` thread is the “application layer”; if we are the author of the library in the middle (the `buffer` thread), we would not have control over the `write` thread. But perhaps instead of exposing the `TQueue` directly, we could offer an API such as

```
dequeue :: TQueue (Either NetworkFailure Int) -> IO (Either NetworkFailure Int)
dequeue queue = atomicallyStubborn $ readTQueue queue
```

However, this doesn’t address the second, more important, problem. Suppose that instead of writing the messages to the terminal, the `write` thread writes those messages to a `TMVar` of its own:

```
/--------\          /--------\            /-------\          /-----\
|        |   TMVar  |        |   TQueue   |       |   TMVar  |     |
| decode | <------> | buffer | <--------> | write | <------> | ... |
|        |          |        |            |       |          |     |
\--------/          \--------/            \-------/          \-----/
```

If we now have a *fourth* thread reading from this `TMVar`, *it too* must use `atomicallyStubborn` to read from that `TMVar`, or else be subject to the exact same problem again.

A better workaround is to consider the `write` thread to be a GC root when it is blocked on the `TQueue`; this way it will not be considered as stalled, and nor will any other threads that depend on it (such as the fourth thread in the example above). We can do this by providing creating a [stable pointer](https://hackage.haskell.org/package/base-4.19.0.0/docs/Foreign-StablePtr.html) to the thread (this workaround is [due to Simon Marlow](https://gitlab.haskell.org/ghc/ghc/-/issues/10793)):

```
write :: TQueue (Either NetworkFailure Int) -> IO ()
write queue = do
 void $ newStablePtr =<< myThreadId
 loop
 where
 loop = .. -- as before
```

Alternatively, if `write` is considered part of the application layer again and we are developing the middle layer, we could provide a `dequeue` function such as

```
dequeue :: TQueue (Either NetworkFailure Int) -> IO (Either NetworkFailure Int)
dequeue queue = do
 tid <- myThreadId
 bracket (newStablePtr tid) freeStablePtr $ \_ ->
 atomically $ readTQueue queue
```

Here we make sure to free the stable pointer again, because we don’t want to turn off stall detection *entirely* in the user’s application.

## Conclusions

Arguably the *real* problem in our running example is that a network failure in the `decode` thread results in a stall in the first place. Indeed, [Simon Marlow writes](https://www.oreilly.com/library/view/parallel-and-concurrent/9781449335939/ch15.html#sec_deadlock):

> You should not rely on deadlock detection for the correct working of your program. Deadlock detection is a debugging feature; in the event of a deadlock, you get an exception rather than a silent hang, but you should aim to never have any deadlocks in your program.

In practice this is not always easy to achieve; maybe because the stall originates in a library we have no control over, or simply because fixing the problem is difficult (e.g. see [Http2 #97](https://github.com/kazu-yamamoto/http2/pull/97), [Http2 #104](https://github.com/kazu-yamamoto/http2/pull/104)); in cases like this it is important to have a reliable workaround.

For the reasons we explained above, we do not consider `atomicallyStubborn` to be a good workaround. However, variants of this pattern are occasionally useful and do appear in the wild; for example, it gets used in `async` ([Async #14](https://github.com/simonmar/async/issues/14)), although it is not entirely clear why the stable pointer workaround did not work there.

For completeness sake, we want to mention two further complications. First, the interaction between finalizers and stalled threads is subtle; see the [section on Deadlock in the documentation of Control.Concurrent](https://hackage.haskell.org/package/base-4.19.0.0/docs/Control-Concurrent.html#g:14) (and [GHC #11001](https://gitlab.haskell.org/ghc/ghc/-/issues/11001)). Second, in the case of *both* infinite recursion *and* stalling, you might not get the exception you prefer; see [Shake #294](https://github.com/ndmitchell/shake/issues/294) / [GHC #10793](https://gitlab.haskell.org/ghc/ghc/-/issues/10793).

This work was sponsored by Anduril as part of the development of [grapesy](https://github.com/well-typed/grapesy), a new Haskell library for gRPC. This library is currently still under development, but we expect to make a first release relatively soon. When that happens it will of course be accompanied by a blog post on the [Well-Typed blog](https://well-typed.com/blog/).

With thanks to Justin Le, Ryan Brown and Kazu Yamamoto for their comments on a draft of this blog post.