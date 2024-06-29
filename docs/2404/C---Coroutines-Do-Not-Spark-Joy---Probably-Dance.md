<!--yml

category: 未分类

date: 2024-05-27 12:57:32

-->

# C++ Coroutines Do Not Spark Joy | Probably Dance

> 来源：[https://probablydance.com/2021/10/31/c-coroutines-do-not-spark-joy/](https://probablydance.com/2021/10/31/c-coroutines-do-not-spark-joy/)

C++20增加了对协程的最小支持。我认为它们的实现方式在很大程度上并不适合C++，主要是因为它们不遵循零开销原则。调用协程可能非常昂贵（需要调用new()和delete()），这种开销并非完全在您的控制之下，而且它们设计得使您难以控制它们的开销。我认为它们受到了C#协程的启发，设计在C#中确实更合适。但在C++中，我不知道它们是为谁设计的，或者谁要求添加这些功能...

在我们开始之前，我必须解释它们是什么，以及它们有什么用处。简而言之，它们对于具有并发性的代码非常有用。经典示例是，如果您的代码有多个状态机在不同时间改变状态：例如，从网络读取数据的状态机正在等待更多字节，而提供字节的代码也是一个状态机（也许正在解压缩），依次从另一个状态机获取其字节（也许正在处理TCP/IP层）。当所有这些状态机都可以假装独立运行时（也许通过管道通信），这样做就更容易。但是，如果流处理器可以通过正常的同步函数调用调用解压器返回字节，代码看起来更美观。协程允许您在函数中间暂停和恢复，而不会在更多字节不立即可用时阻塞整个系统。

C++标准所做的最好的事情之一是将“协程”这个词定义为与“纤程”或“绿色线程”等相关概念不同的词汇。（这与现有用法非常不同，例如Lua协程与C++协程不同。我认为这很好，因为C++添加的东西可能很有用，并且有很多理由证明现有用法是错误的）。在标准中，协程简单地是一个在多次调用时表现不同的函数：它不会在每次调用时从头开始，而是在上次返回的返回语句后继续运行。为了实现这一点，它需要一些状态来存储上次返回时的信息，以及那时本地变量的状态。在现有用法中，这意味着您需要一个程序堆栈来存储这种状态，但在C++中，这只是存储在一个结构体中。

为了说明这一切，让我们在普通的C++中构建一个协程，而不使用语言协程：

## 手动构建协程

我们将以一个非常简单的例子为例。C++协程将转换此代码：

```
generator_coro<int> range(int stop, int step = 1)
{
    for (int i = 0; i < stop;)
    {
        co_yield i;
        i += step;
    }
}

```

Into this code:

```
struct range_struct
{
    int i;
    int stop;
    int step;
    enum {
        At_start,
        In_loop,
        Done
    } state;

    explicit range_struct(int stop, int step = 1)
        : stop(stop), step(step), state(At_start)
    {
    }

    int resume()
    {
    switch (state)
    {
    case At_start:  for (i = 0; i < stop;)
                    {
                        state = In_loop;
                        return i;
    case In_loop:       i += step;
                    }
                    state = Done;
    case Done:      return 0;
    }
    }
};

```

第一个清单中的“co_yield”是一个新关键字。我说重复调用协程会在最后的return语句后继续执行，但它们实际上在最后的“co_yield”语句后继续执行。（而co_yield在语法上与“return”相同）。我认为这是因为你想要函数有两种不同的返回方式：

+   co_return，意味着“返回并不再被调用”

+   co_yield，意味着“在下一次调用时返回并继续执行此处”。

第二个清单是一个结构体，看起来与编译器为第一个清单生成的完全相同。转换如下：

1.  将所有堆栈变量转换为结构体的成员变量

1.  将函数重命名为resume()并将其作为结构体的成员函数

1.  在函数整体的主体周围加上一个switch/case，就像Duff's Device中那样

1.  为每个co_yield在你的枚举中添加一个新的case。

1.  添加一个Done case来指示函数已经结束。在结尾返回什么并不重要，从Done返回的值永远不应该被使用

这些转换甚至适用于更复杂的函数。这可能来自于Simon Tatham经典文章[在C中使用协程](https://www.chiark.greenend.org.uk/~sgtatham/coroutines.html)，该文章使用静态变量而非类成员变量进行了这种转换。

旁注：作为“协程”一词现有用法的一个例子，汤姆·达夫（Duff's Device的作者）[说](https://brainwagon.org/2005/03/05/coroutines-in-c/#comments)这不是实现协程的好方法，因为使用这种技巧时，无法从嵌套函数中产生yield。C++标准将其颠倒过来，并表示“只有在不能从嵌套函数中产生yield时才算是协程。如果可以，它将会是一个fiber。”

让我们继续构建这个协程，使其做一些有用的事情。比如说，我们想让这个测试工作：

```
TEST(range, coro_and_struct)
{
    std::vector<int> range_result;
    std::vector<int> range_struct_result;
    for (int i : range(10, 2))
    {
        range_result.push_back(i);
    }
    for (int i : range_struct(10, 2))
    {
        range_struct_result.push_back(i);
    }
    ASSERT_EQ((std::vector<int>{0, 2, 4, 6, 8}), range_result);
    ASSERT_EQ((std::vector<int>{0, 2, 4, 6, 8}), range_struct_result);
}

```

所以在这里我分配了两个向量，然后我循环遍历range()函数（来自第一个清单）的结果，并且也循环遍历range_struct()构造函数（来自第二个清单）的结果，两者都应该给出相同的结果。知道range_struct迄今为止是如何实现的，你会为了使其工作而添加什么？

我们需要迭代器。这是使range_struct工作的最小实现：

```
struct range_struct
{
    // ... unchanged top half of the struct

    struct end_iterator {};
    struct iterator
    {
        range_struct * self;
        bool operator==(const end_iterator&) const
        {
            return self->state == Done;
        }
        void operator++()
        {
            self->resume();
        }
        int operator*() const
        {
            return self->i;
        }
    };

    iterator begin()
    {
        resume();
        return { this };
    }
    end_iterator end()
    {
        return {};
    }
};

```

迭代器需要operator++来前进，operator*来获取值，operator==来检查是否到达末尾。每个的实现都很简单，但可能需要一些思考才能看出它是否正确。一个奇怪的地方是，在构造迭代器时，在begin()中我们还必须调用一次resume()。这只是C++迭代器接口的一个偶然。这些协程实际上与Rust迭代器接口更匹配，其中next()直接包装resume()。要看到调用的必要性，思考一下如果我们没有调用一次resume()，并且end==0时会发生什么。operator==的第一次调用会返回错误的结果。

可以避免读取函数的内部状态。我们可以存储 resume() 的返回值并记住它，而不是读取 self->i。然后，这将适用于按照我上述转换步骤操作的任何函数，但出于教育目的，我希望保持实现的简单性。

如果这是编译器所做的一切，那么问题是什么？问题在于他们并没有停在这一步。

## 堆分配

上述方法的一个问题是我转换步骤中的步骤1：“将所有堆栈变量转换为结构体的成员变量。”在更复杂的函数中，这显然是浪费的。如果在 for 循环结束后还有更多工作要做，编译器可以重用曾经由变量 'i' 占用的堆栈空间。因此，C++ 标准不希望描述这些协程结构体的布局。因此，你无法完全看到这个结构体。语言完全将其隐藏起来，甚至比 lambda 函数类型更多。你甚至无法对其使用 sizeof()。如何将一种类型隐藏得比 lambda 函数类型还多？它们将其隐藏在指针背后。并且该指针指向一个堆分配。

因此，每次调用协程都会包含一个 new() 的调用。在协程结束时，会调用 delete()。他们是如何将这个引入到标准中的呢？谁相信了“我们想要这个编译器优化，而我们只需引入一次 new() 和一次 delete()”的故事，显然这不是优化？将这个引入标准中的方法是[承诺](https://www.youtube.com/watch?v=8C8NnE1Dg4A)协程可以内联，并且如果内联了，就不需要调用 new() 和 delete()。

这听起来不错，简单的生成器示例确实可以内联处理，但对于更复杂的内容似乎行不通。我尝试实现[Differential Dataflow](https://www.microsoft.com/en-us/research/publication/differential-dataflow/)，希望代码看起来大多数时候都很正常。因此，这是在构建一个由嵌套协程图组成的图形，每个协程操作来自其他协程的管道。这类似于上面的简单范围生成器，只是每个函数都在无限输入管道上操作，并且存在连接、分割和聚合。生命周期通常不明确，但有时它们非常明确，即使如此，我也无法实现内联处理。

这在协程寿命较长时并非问题。一旦分配完成，挂起和恢复协程的速度很快。回顾我们上面创建的结构体，这是有道理的：堆分配仅影响构造函数和析构函数。但是，如果你有小型实用协程，堆分配可能会带来很大成本。

这是语言上的一个重大变化。语言从未想要控制内联。"inline"关键字故意模糊不清。有非标准的方法来表示“不要内联”和“请尽力内联”，但没有保证。在大多数代码中，内联和不内联的区别并不大。但如果不内联意味着增加堆分配，那么突然之间不内联就可能导致代码运行慢100倍。有了这么大的差异，我们突然需要真正控制是否内联。由于协程只是一个结构体，我们应该已经能够控制这一点：只需将其放在堆栈上，或者将其作为不同结构的成员，但语言故意禁止这样做，坚持让编译器来处理。

所以如果需要内联某些东西，你会用什么？

## 宏

我的协程代码很快积累了宏。而且不仅仅是为了性能原因。有几种方式协程不好组合。例如，让我们坚持上面的生成器代码并稍微复杂化它。假设它首先发出从零到九的数字，然后调用另一个也返回生成器的东西：

```
generator_coro<int> foo()
{
    for (int i = 0; i < 10; ++i)
        co_yield i;
    generator_coro<int> rest = bar();
    // ... how do I return all the values from rest?
    return rest;
}

```

这是不能编译的。rest的类型是`generator_coro<int>`，看起来函数返回`generator_coro<int>`，但实际上那是协程包装器的类型，而包装器期望我们返回整数（正如我们在之前产生整数时所看到的）。

所以你必须写成这样：

```
generator_coro<int> foo()
{
    for (int i = 0; i < 10; ++i)
        co_yield i;
    generator_coro<int> rest = bar();
    for (int i : rest)
        co_yield i;
}

```

好吧，这并不太糟糕。我们只需将另一个协程存储在我们的堆栈上，然后从中产生所有值。在我的代码中，我经常像这样组合协程，这都很好，直到我需要做出改变。我想稍微修改如何像这样转发值。举个例子，假设我想在执行此循环之前设置一个标志“is_draining”。所以它应该看起来像这样：

```
generator_coro<int> foo()
{
    for (int i = 0; i < 10; ++i)
        co_yield i;
    generator_coro<int> rest = bar();
    rest.set_is_draining(true);
    for (int i : rest)
        co_yield i;
}

```

不幸的是，代码被手动内联到各个地方，比如这样：完全耗尽并产生生成器时，我总是遍历列表，所以当我需要在循环之前设置标志时，我必须改变每个地方。在正常代码中我该如何修复这个问题？编写一个封装重复代码的函数：

```
generator_coro<int> drain_and_yield_all(generator_coro<int> & coro)
{
    coro.set_is_draining(true);
    for (int i : coro)
        co_yield i;
}

```

这看起来很合理：现在我可以调用这个函数来耗尽生成器。但你不能这样做。这是一个新的协程。如果我试图在外部函数中使用这个，我会得到这个：

```
generator_coro<int> foo()
{
    for (int i = 0; i < 10; ++i)
        co_yield i;
    generator_coro<int> rest = bar();
    generator_coro<int> rest2 = drain_and_yield_all(rest);
    // ... what now?
}

```

函数`drain_and_yield_all`返回一个全新的协程，我不能返回它。所以我回到了最初的问题。

唯一解决这个问题的方法是使用宏。协程没有宏就无法组合。我不能强制函数“drain_and_yield_all”内联化，因此它生成一个新的协程（可能在堆上分配），因此我不能编写这个小巧的可重用辅助程序。如果我想要一种标准化的方法来做这件事，我可以轻松地更改，我需要使用宏：

```
#define DRAIN_AND_YIELD_ALL(x) if (true)\
{\
    auto && coro = x;\
    coro.set_is_draining(true);\
    for (auto && i : coro)\
        co_yield i;\
} else static_cast<void>(0)

```

不太漂亮，但这是唯一的方法。

## 产出值

在我上面的手动代码转换中，我只是从协程的堆栈中读取值‘i’。为什么要把它变得更加复杂呢？

标准稍微抽象了一些。当我在上面返回类型“generator_coro<int>”时，无论函数体如何，都是相同的类型。我没有展示这种类型的样子，因为它很凌乱，但我必须稍微解释一下。理论上它只是一个包装器，提供迭代器接口，所以我们可以在循环中使用这种类型，但它必须适用于所有函数。在我上面的手动转换中，我们看到对于每个函数我们得到一个不同的结构体，那么一个包装器如何适用于可能获得的所有不同结构体？堆分配使得这更容易，但主要原因是包装器不能依赖于函数中的任何内容。我们不能访问它的堆栈变量。那么 co_yield 如何工作？

协程和调用者之间有一种称为“promise type”的通信渠道。这是 co_yield 可以存储值的地方，也是包装器可以从中读取的地方。对于返回相同包装器的所有协程，这个 promise_type 都是相同的。因此，在 co_yield 中，我们实际上只是将值存储在 promise_type 中，然后包装器从 promise_type 中读取它。在我的示例中，这显然效率低下，因为我只是在产出一个堆栈变量。我们真的需要复制它吗？

一个改进是在 promise_type 中存储一个指针。对于 int 来说这有点过度，但通常来说更快。我们可以安全地指向协程体中的任何内容，因为这些内容都只是结构体的成员，而结构体一直存在。这样可以节省一次复制。

但是我们可以更进一步：事实证明，在一个特殊情况下允许直接从堆栈中读取：promise_type 存储在协程的堆栈上。因此，所有返回相同包装器的协程将生成一个结构体，其中 promise_type 是其成员之一。只要我们从 promise_type 中读取，我们就可以从结构体中读取。问题在于协程无法向 promise_type 写入。如果可以的话，我会把我的循环变量‘i’放入 promise_type 中。

幸运的是，David Mazieres [找到了方法](https://www.scs.stanford.edu/~dm/blog/c++-coroutines.html#the-promise-object) 来做一个理智的事情。不幸的是，代码看起来有点疯狂。还有另一个新操作符，co_await，我还没解释过。当你的协程等待长时间操作的结果时，比如网络调用，就可以使用 co_await，这意味着“现在返回，等对象告诉我何时可以再次调用我”。实际上你可以 co_await 一些指示网络请求完成的对象。但我们并不打算用它来做那个。事实证明，co_await 可以作为获取承诺对象的一种方法，因为可等待对象允许从 promise_type 中读取，就像包装器一样。所以我们将创建一个可等待对象，看起来像这样：

```
template<typename PromiseType>
struct GetPromise
{
  PromiseType * p;
  bool await_ready()
  {
      return false; // says we're not ready, call await_suspend
  }
  bool await_suspend(costd::coroutine_handle<PromiseType> h)
  {
    p = &h.promise();
    return false; // says no don't suspend coroutine after all
  }
  PromiseType * await_resume()
  {
      return p;
  }
};

```

每个这些函数根据标准都有意义。但在这里真正相关的是，在 await_suspend 中，我们可以访问 promise_type。我们只需存储一个指针，然后在 await_resume 中返回它。所以在协程体中，你可以这样做：

```
generator_coro<int> range(int stop, int step)
{
    generator_coro<int>::promise_type * promise = co_await GetPromise<generator_coro<int>::promise_type>{};
    for (promise->i = 0; promise->i < stop;)
    {
        co_yield promise->i;
        promise->i += step;
    }
}

```

这看起来有点恶心，但我们确信承诺存储在此协程的自动生成结构中的“stop”和“step”旁边。所以这就是你如何直接从栈中产生值而不进行复制且不必使用指针的方式。co_yield 调用应该被编译为自我赋值，即 i=i。

看起来有些疯狂，但如果你考虑实际生成的指令，这是在协程和其句柄之间传递数据的唯一理智的方法。一旦你理解了协程的工作原理，从包装器直接从栈上读取值就是有意义的，而协程则将值写入栈中。而唯一允许执行此操作的地方就是在 promise_type 中。尽管它位于我们的栈上，我们不应该访问它，但这只是标准行为很奇怪的一个例子。

一件好事是将其封装成一个包装函数，这样我就可以写成这样：

```
generator_coro<int> range(generator_coro<int>::promise_type * promise, int stop, int step)
{
    for (promise->i = 0; promise->i < stop;)
    {
        co_yield promise->i;
        promise->i += step;
    }
}

```

所以我只需将承诺作为我的其中一个参数即可。可惜，这个包装器无法写出来，因为我们必须强制内联才能使其工作，而这是不可能的。不过，你可以用宏来稍微减少其丑陋程度……

## 用于 Fibers 的运算符

有件事我一直期待的是终于在语言中拥有 co_yield 和 co_await 操作符，这样我就可以在 Fibers 中使用它们。一切都足够可定制，这应该是可能的。在 C++20 之前，我必须在我的 Fibers 中创建这样的宏：

```
#define FIBER_YIELD(x) if (fiber_yield(x)) static_cast<void>(0) else return false

```

在这里我假设“fiber_yield()”如果纤程应该继续运行则返回true，如果纤程应该提前结束则返回false（可能是因为正在析构，所以我们需要拆解堆栈，因此立即返回是绝对必要的）。co_yield运算符内置了这种行为，我也想在纤程中使用它。不幸的是，一旦你使用了这个关键字，你的函数不再是函数，而是一个协程。这意味着它需要一个包装器并且需要在堆上分配内存。这很糟糕，因为纤程不会遇到我上面提到的组合问题：写一些小的帮助函数更容易，因为在纤程中你可以从嵌套函数中yield。但是如果这些小的嵌套函数必须是协程，那就糟糕了。

## 结论和建议

总体上，我仍然很高兴协程存在。我喜欢尝试它们。但每个人的反应似乎都是一样的：这些协程很快就消耗掉你的精力。它们没有激发我的兴奋点。一切都有点笨拙和粗糙。你总是要担心是否有东西被内联，因为内联协程和非内联协程之间的差距比普通函数的差距要大得多。而且它们使用起来并不方便。

如果你正在为其他语言编写协程，请做几件简单的事情：

1.  给我访问生成的结构体的权限。允许我将其放在另一个函数的栈上，或者作为另一个结构体的成员。然后我可以将它存储在堆上，但不要强制我这样做。

1.  思考这些如何组合。如果我对结构有控制权，情况会好很多，因为如果我想写一个小的实用函数，我可以将它放在栈上，而不必担心堆分配的问题。

1.  允许我在纤程中使用相同的操作符。不要仅因为我使用了“co_yield”而将我的函数转变为协程。

1.  给我一个更明智的方式来访问栈变量。也许只是允许我在栈变量上写“public”。因此，初始代码看起来会像这样：

```
generator_coro<int> range(int stop, int step = 1)
{
    public int i;
    for (i = 0; i < stop;)
    {
        co_yield i;
        i += step;
    }
}

```

我将这些内容单独放在一行，以便更清晰，但可以将任何栈变量都变为公共的，无论它如何声明。因为这些变量最终都变成了结构体成员，使用“public”关键字甚至合理，因为它用于同样的目的：我使某些东西对结构体的用户可访问。

就目前的协程来说，我不太明白它们的用途。它们似乎可能有用，但没有人似乎对它们感到兴奋。那些有长时间并行任务的人很久以前就已经转向使用纤程了。参见[这个好的讲座](https://www.youtube.com/watch?v=JDcip-SRgVE)（2012年）或[这个更好的讲座](https://www.gdcvault.com/play/1022186/Parallelizing-the-Naughty-Dog-Engine)（2015年）。我不知道在这里如何使用协程。当你的内联器改变主意时，有堆分配的威胁会毁掉它。（或者更可能对于像这样的复杂代码，内联根本不起作用）

我很好奇是否有人真的能找到这些东西有用。它们只是用于简单的生成函数吗？
