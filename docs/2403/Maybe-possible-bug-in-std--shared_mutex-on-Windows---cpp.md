<!--yml
category: 未分类
date: 2024-05-28 18:11:19
-->

# Maybe possible bug in std::shared_mutex on Windows : cpp

> 来源：[https://old.reddit.com/r/cpp/comments/1b55686/maybe_possible_bug_in_stdshared_mutex_on_windows/](https://old.reddit.com/r/cpp/comments/1b55686/maybe_possible_bug_in_stdshared_mutex_on_windows/)

A team at my company ran into a peculiar and unexpected behavior with `std::shared_mutex`. This behavior only occurs on Windows w/ MSVC. It does not occur with MinGW or on other platforms.

At this point the behavior is pretty well understood. The question isn't "how to work around this". The questions are:

1.  Is this a bug in `std::shared_mutex`?
2.  Is this a bug in the [Windows SlimReaderWriter](https://learn.microsoft.com/en-us/windows/win32/sync/slim-reader-writer--srw--locks) implementation?

I'm going to boldly claim "definitely yes" and "yes, or the SRW behavior needs to be documented". Your reaction is surely "it's never a bug, it's always user error". I appreciate that sentiment. Please hold that thought for just a minute and read on.

Here's the scenario:

1.  Main thread acquires **exclusive** lock
2.  Main thread creates N child threads
3.  Each child thread:
    1.  Acquires a **shared** lock
    2.  Yields until all children have acquired a shared lock
    3.  Releases the **shared** lock
4.  Main thread releases the **exclusive** lock

This works ***most*** of the time. However 1 out of ~1000 times it "deadlocks". When it deadlocks exactly 1 child successfully acquires a shared lock and all other children block forever in `lock_shared()`. This behavior can be observed with `std::shared_mutex`, `std::shared_lock`/`std::unique_lock`, or simply calling `SRW` functions directly.

If the single child that succeeds calls `unlock_shared()` then the other children will wake up. However if we're waiting for all readers to acquire their shared lock then we will wait forever. Yes, we could achieve this behavior in other ways, that's not the question.

I made a [StackOverflow post](https://stackoverflow.com/questions/78090862/stdshared-mutexunlock-shared-blocks-even-though-there-are-no-active-exclus) that has had some good discussion. The behavior has been confirmed. However at this point we need a language lawyer, [u/STL](/u/STL), or quite honestly Raymond Chen to declare whether this is "by design" or a bug.

Here is code that can be trivially compiled to repro the error.

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

Personally I don't think the function `lock_shared()` should ever be allowed to block forever when there is not an exclusive lock. That, to me, is a bug. One that only appears for `std::shared_mutex` in the `SRW`-based Windows MSVC implementation. *Maybe* it's allowed by the language spec? I'm not a language lawyer.

I'm also inclined to call the `SRW` behavior either a bug or something that should be documented. There's a [2017 Raymond Chen](https://devblogs.microsoft.com/oldnewthing/20170301-00/?p=95615) post that discusses EXACTLY this behavior. He implies it is user error. Therefore I'm inclined to boldly, and perhaps wrongly, call this is an `SRW` bug.

What do y'all think?

Edit: Updated to explicitly set `readCounter` to 0\. That is not the problem.