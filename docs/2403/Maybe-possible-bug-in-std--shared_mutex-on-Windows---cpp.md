<!--yml

类别：未分类

日期：2024-05-28 18:11:19

-->

# 可能是`std::shared_mutex`在Windows上的潜在bug：cpp

> 来源：[https://old.reddit.com/r/cpp/comments/1b55686/maybe_possible_bug_in_stdshared_mutex_on_windows/](https://old.reddit.com/r/cpp/comments/1b55686/maybe_possible_bug_in_stdshared_mutex_on_windows/)

我公司的一个团队遇到了`std::shared_mutex`的一种奇特且意外的行为。这种行为仅在使用Windows的MSVC时发生，而在MinGW或其他平台上则不会发生。

到了这一点，行为已经非常清楚。问题不是"如何解决这个问题"。问题是：

1.  这是`std::shared_mutex`的bug吗？

1.  这是[Windows SlimReaderWriter](https://learn.microsoft.com/en-us/windows/win32/sync/slim-reader-writer--srw--locks)实现中的bug吗？

我大胆地声称"肯定是"和"是的，或者SRW的行为需要被记录下来"。你的反应肯定是"这绝对不是bug，这总是用户的错误"。我理解这种观点。请暂时保持这种看法，并继续阅读。

这里是情景：

1.  主线程获取了**排他**锁

1.  主线程创建了N个子线程。

1.  每个子线程：

    1.  获取了**共享**锁

    1.  直到所有子进程都获取了共享锁之后，才会产生结果。

    1.  释放了**共享**锁

1.  主线程释放了**排他**锁。

这通常***大部分***时间都有效。然而，大约1次/1000次会发生"死锁"。当它发生时，确实有一个子进程成功获取了共享锁，而所有其他子进程则永远阻塞在`lock_shared()`中。无论是使用`std::shared_mutex`、`std::shared_lock`/`std::unique_lock`，还是直接调用`SRW`函数，都可以观察到这种行为。

如果唯一成功调用`unlock_shared()`的子进程，则其他子进程将会唤醒。然而，如果我们正在等待所有读者获取他们的共享锁，那么我们将永远等待下去。是的，我们可以通过其他方式实现这种行为，但这不是问题的关键。

我在[StackOverflow上发表了一个帖子](https://stackoverflow.com/questions/78090862/stdshared-mutexunlock-shared-blocks-even-though-there-are-no-active-exclus)，得到了一些很好的讨论。行为已被确认。然而，现在我们需要一个语言专家，[u/STL](/u/STL)，或者坦率地说是Raymond Chen，来宣布这是"设计缺陷"还是一个bug。

这里有一段代码，可以轻松编译以重现这个错误。

```
#include <atomic>
#include <cstdint>
#include <iostream>
#include <memory>
#include <shared_mutex>
#include <thread>
#include <vector>

struct ThreadTestData {
    int32_t numThreads = 0;
    std::shared_mutex sharedMutex = {};
    std::atomic<int32_t> readCounter = 0;
};

int DoStuff(ThreadTestData* data) {
    // Acquire reader lock
    data->sharedMutex.lock_shared();

    // wait until all read threads have acquired their shared lock
    data->readCounter.fetch_add(1);
    while (data->readCounter.load() != data->numThreads) {
        std::this_thread::yield();
    }

    // Release reader lock
    data->sharedMutex.unlock_shared();

    return 0;
}

int main() {
    int count = 0;
    while (true) {
        ThreadTestData data = {};
        data.numThreads = 5;

        // Acquire write lock
        data.sharedMutex.lock();

        // Create N threads
        std::vector<std::unique_ptr<std::thread>> readerThreads;
        readerThreads.reserve(data.numThreads);
        for (int i = 0; i < data.numThreads; ++i) {
            readerThreads.emplace_back(std::make_unique<std::thread>(DoStuff, &data));
        }

        // Release write lock
        data.sharedMutex.unlock();

        // Wait for all readers to succeed
        for (auto& thread : readerThreads) {
            thread->join();
        }

        // Cleanup
        readerThreads.clear();

        // Spew so we can tell when it's deadlocked
        count += 1;
        std::cout << count << std::endl;
    }

    return 0;
} 
```

就我个人而言，我认为在没有排他锁的情况下，函数`lock_shared()`不应该被允许永远阻塞。对我来说，这是一个bug。这种情况只出现在基于`SRW`的Windows MSVC实现中的`std::shared_mutex`中。也许这在语言规范上是允许的？我不是一个语言专家。

我倾向于将`SRW`行为称为错误或者应该被记录的事情。有一篇[2017 年 Raymond Chen](https://devblogs.microsoft.com/oldnewthing/20170301-00/?p=95615)的博文讨论了**确切**的这种行为。他暗示这是用户错误。因此，我倾向于大胆地，也许是错误地，称这是一个`SRW`的 bug。

你们觉得呢？

编辑：更新以明确地将`readCounter`设置为 0。问题并非如此。
