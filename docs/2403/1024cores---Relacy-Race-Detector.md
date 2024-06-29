<!--yml

category: 未分类

date: 2024-05-29 12:36:59

-->

# 1024cores - Relacy Race Detector

> 来源：[https://www.1024cores.net/home/relacy-race-detector](https://www.1024cores.net/home/relacy-race-detector)

Relacy Race Detector 是一个用于 C++0x 编写的同步算法单元测试的工具。

每个用户线程都表示为一个纤程（ucontext）。每次只有一个纤程在运行，特殊调度程序控制纤程之间的交错。

使用随机调度程序，它只执行线程之间大量不同的交错。使用全搜索调度程序或上下文绑定调度程序时，它系统地执行线程之间的所有可能交错。在执行特定的交错时，它对执行的各个方面（竞争、访问释放内存等）进行彻底验证。

如果未发现错误，则在验证特定数量的交错（对于随机调度程序）或验证所有可能的交错（对于全搜索调度程序）后终止验证。如果发现错误，则工具输出导致错误的执行历史并终止。

物理上，Relacy Race Detector 是一个头文件库，适用于 C++98。

+   +   放松的 ISO C++0x 内存模型。Relaxed/acquire/release/acq_rel/seq_cst 内存操作。唯一不支持的功能是 memory_order_consume，它通过 memory_order_acquire 进行模拟。

    +   彻底的自动错误检查（包括 ABA 检测）。

    +   完整的原子库（包括比较交换中的虚假失败）。

    +   内存栅栏。

    +   任意数量的线程。

    +   失败测试的详细执行历史。

    +   无误报。

    +   测试套件的 before/after/invariant 函数。

+   MSVC 7/8/9，Windows 2000 及以上，32 位

+   GCC 3.4 及以上版本，Linux，32 位

1\. 添加

#include <relacy/relacy_std.hpp>

2\. 对于原子变量使用类型 std::atomic<T>：

std::atomic<void*> head;

3\. 对于通常的非原子变量使用类型 rl::var<T>：

rl::var<int> data;

这些变量将被检查是否存在竞争，并包含在跟踪中。

4\. 所有对 std::atomic<T> 和 rl::var<T> 变量的访问都要加上 '($)' 后缀：

std::atomic<void*> head;

rl::var<int> data;

head($).store(0);

data($) = head($).load();

5\. 严格的线程私有变量可以保留为原样：

for (int i = 0; i != 10; ++i)

这些变量不会被检查是否存在竞争，也不会被包含在跟踪中。但它们将加速验证过程。

6\. 描述测试套件：线程数，线程函数，before/after/invariant 函数。参见下面的示例。

7\. 放置断言：

int x = g($).load();

RL_ASSERT(x > 0);

8\. 开始验证：

rl::simulate<test_suite_t>();

#include <relacy/relacy_std.hpp>

// 模板参数 '2' 是线程数

struct race_test : rl::test_suite<race_test, 2>

{

std::atomic<int> a;

rl::var<int> x;

// 主线程函数前在单线程中执行

void before()

{

a($) = 0;

x($) = 0;

}

// 主线程函数

void thread(unsigned thread_index)

{

if (0 == thread_index)

{

x($) = 1;

a($).store(1, rl::memory_order_relaxed);

}

else

{

如果 (1 == a($).load(rl::memory_order_relaxed))

x($) = 2;

}

}

// 执行完主线程函数后在单线程中执行

void 后()

{

}

// 执行完主线程中每个“可见”操作后在单线程中执行

// 禁止修改任何状态

void 不变()

{

}

};

`int main()`

{

rl::simulate<race_test>();

}

结构 race_test

数据竞争

迭代：8

执行历史：

[0] 0: <00366538> 原子存储，值=0，(之前的值=0)，顺序=seq_cst，在race_test::before，test.cpp(14)

[1] 0: <0036655C> 存储，值=0，在race_test::before，test.cpp(15)

[2] 0: <0036655C> 存储，值=1，在race_test::thread，test.cpp(23)

[3] 0: <00366538> 原子存储，值=1，(之前的值=0)，顺序=relaxed，在race_test::thread，test.cpp(24)

[4] 1: <00366538> 原子加载，值=1，顺序=relaxed，在race_test::thread，test.cpp(28)

[5] 1: <0036655C> 存储，值=0，在race_test::thread，test.cpp(29)

[6] 1: 检测到数据竞争，在race_test::thread，test.cpp(29)

线程 0：

[0] 0: <00366538> 原子存储，值=0，(之前的值=0)，顺序=seq_cst，在race_test::before，test.cpp(14)

[1] 0: <0036655C> 存储，值=0，在race_test::before，test.cpp(15)

[2] 0: <0036655C> 存储，值=1，在race_test::thread，test.cpp(23)

[3] 0: <00366538> 原子存储，值=1，(之前的值=0)，顺序=relaxed，在race_test::thread，test.cpp(24)

线程 1：

[4] 1: <00366538> 原子加载，值=1，顺序=relaxed，在race_test::thread，test.cpp(28)

[5] 1: <0036655C> 存储，值=0，在race_test::thread，test.cpp(29)

[6] 1: 检测到数据竞争，在race_test::thread，test.cpp(29)

你可以在这里下载该库：http://sites.google.com/site/1024cores/home/downloads/relacy_2_4.zip

工具的旧弃用站点：[http://groups.google.com/group/relacy](http://groups.google.com/group/relacy)
