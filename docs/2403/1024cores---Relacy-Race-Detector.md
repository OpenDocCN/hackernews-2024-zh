<!--yml
category: 未分类
date: 2024-05-29 12:36:59
-->

# 1024cores - Relacy Race Detector

> 来源：[https://www.1024cores.net/home/relacy-race-detector](https://www.1024cores.net/home/relacy-race-detector)

Relacy Race Detector is a tool for efficient execution of unit-tests for synchronization algorithms written in C++0x.

Every user thread is represented as a fiber (ucontext). Every time only one fiber is running, and special scheduler controls interleaving between fibers.

With random scheduler it just executes numerous amount of various interleavings between threads. With full search scheduler or context-bound scheduler it systematically executes all possible interleavings between threads. While executing particular interleaving it makes exhaustive verification of various aspects of execution (races, accesses to freed memory etc).

If no errors found then verification terminates when particular number of interleavings are verified (for random scheduler), or when all possible interleavings are verified (for full search scheduler). If error is found then tool outputs execution history which leads to error and terminates.

Physically Relacy Race Detector is a header-only library for C++98.

## 

*   *   Relaxed ISO C++0x Memory Model. Relaxed/acquire/release/acq_rel/seq_cst memory operations. The only non-supported feature is memory_order_consume, it's simulated with memory_order_acquire.

    *   Exhaustive automatic error checking (including ABA detection).

    *   Full-fledged atomics library (with spurious failures in compare_exchange()).

    *   Memory fences.

    *   Arbitrary number of threads.

    *   Detailed execution history for failed tests.

    *   No false positives.

    *   Before/after/invariant functions for test suites.

## 

*   MSVC 7/8/9, Windows 2000 and above, 32-bit

*   GCC 3.4 and above, Linux, 32-bit

## 

1\. Add

#include <relacy/relacy_std.hpp>

2\. For atomic variables use type std::atomic<T>:

std::atomic<void*> head;

3\. For usual non-atomic variables use type rl::var<T>:

rl::var<int> data;

Such vars will be checked for races and included into trace.

4\. All accesses to std::atomic<T> and rl::var<T> variables postfix with '($)':

std::atomic<void*> head;

rl::var<int> data;

head($).store(0);

data($) = head($).load();

5\. Strictly thread-private variables use can leave as-is:

for (int i = 0; i != 10; ++i)

Such vars will be NOT checked for races NOR included into trace. But they will accelerate verification.

6\. Describe test-suite: number of threads, thread function, before/after/invariant functions. See example below.

7\. Place asserts:

int x = g($).load();

RL_ASSERT(x > 0);

8\. Start verification:

rl::simulate<test_suite_t>();

## 

#include <relacy/relacy_std.hpp>

// template parameter '2' is number of threads

struct race_test : rl::test_suite<race_test, 2>

{

std::atomic<int> a;

rl::var<int> x;

// executed in single thread before main thread function

void before()

{

a($) = 0;

x($) = 0;

}

// main thread function

void thread(unsigned thread_index)

{

if (0 == thread_index)

{

x($) = 1;

a($).store(1, rl::memory_order_relaxed);

}

else

{

if (1 == a($).load(rl::memory_order_relaxed))

x($) = 2;

}

}

// executed in single thread after main thread function

void after()

{

}

// executed in single thread after every 'visible' action in main threads

// disallowed to modify any state

void invariant()

{

}

};

int main()

{

rl::simulate<race_test>();

}

## 

struct race_test

DATA RACE

iteration: 8

execution history:

[0] 0: <00366538> atomic store, value=0, (prev value=0), order=seq_cst, in race_test::before, test.cpp(14)

[1] 0: <0036655C> store, value=0, in race_test::before, test.cpp(15)

[2] 0: <0036655C> store, value=1, in race_test::thread, test.cpp(23)

[3] 0: <00366538> atomic store, value=1, (prev value=0), order=relaxed, in race_test::thread, test.cpp(24)

[4] 1: <00366538> atomic load, value=1, order=relaxed, in race_test::thread, test.cpp(28)

[5] 1: <0036655C> store, value=0, in race_test::thread, test.cpp(29)

[6] 1: data race detected, in race_test::thread, test.cpp(29)

thread 0:

[0] 0: <00366538> atomic store, value=0, (prev value=0), order=seq_cst, in race_test::before, test.cpp(14)

[1] 0: <0036655C> store, value=0, in race_test::before, test.cpp(15)

[2] 0: <0036655C> store, value=1, in race_test::thread, test.cpp(23)

[3] 0: <00366538> atomic store, value=1, (prev value=0), order=relaxed, in race_test::thread, test.cpp(24)

thread 1:

[4] 1: <00366538> atomic load, value=1, order=relaxed, in race_test::thread, test.cpp(28)

[5] 1: <0036655C> store, value=0, in race_test::thread, test.cpp(29)

[6] 1: data race detected, in race_test::thread, test.cpp(29)

You can download the library here: http://sites.google.com/site/1024cores/home/downloads/relacy_2_4.zip

The old deprecated site for the tool: [http://groups.google.com/group/relacy](http://groups.google.com/group/relacy)