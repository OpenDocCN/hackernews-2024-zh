<!--yml

类别：未分类

日期：2024 年 05 月 27 日 14:43:51

-->

# 一个拥有不可能大尺寸的向量的情况 - The Old New Thing

> 来源：[`devblogs.microsoft.com/oldnewthing/20240105-00/?p=109242`](https://devblogs.microsoft.com/oldnewthing/20240105-00/?p=109242)

# 一个拥有不可能大尺寸的向量的情况

一个客户的程序崩溃了，并且出现了以下堆栈：

```
contoso!Widget::GetCost
contoso!StandardWidgets::get_TotalCost+0x12f
rpcrt4!Invoke+0x73
rpcrt4!Ndr64StubWorker+0xb9b
rpcrt4!NdrStubCall3+0xd7
combase!CStdStubBuffer_Invoke+0xdb
combase!ObjectMethodExceptionHandlingAction<<lambda_...> >+0x47
combase!DefaultStubInvoke+0x376
combase!ServerCall::ContextInvoke+0x6f3
combase!ComInvokeWithLockAndIPID+0xacb
combase!ThreadInvoke+0x103
rpcrt4!DispatchToStubInCNoAvrf+0x18
rpcrt4!RPC_INTERFACE::DispatchToStubWorker+0x1a9
rpcrt4!RPC_INTERFACE::DispatchToStubWithObject+0x1a7
rpcrt4!LRPC_SCALL::DispatchRequest+0x308
rpcrt4!LRPC_SCALL::HandleRequest+0xdcb
rpcrt4!LRPC_SASSOCIATION::HandleRequest+0x2c3
rpcrt4!LRPC_ADDRESS::HandleRequest+0x183
rpcrt4!LRPC_ADDRESS::ProcessIO+0x939
rpcrt4!LrpcIoComplete+0xff
ntdll!TppAlpcpExecuteCallback+0x14d
ntdll!TppWorkerThread+0x4b4
kernel32!BaseThreadInitThunk+0x18
ntdll!RtlUserThreadStart+0x21

```

他们纳闷是否最近对 Windows 进行了一些更改导致了问题的源头，因为在早期版本的 Windows 中并没有发生这种情况。

堆栈跟踪指向了 `Widget::<wbr>IsEnabled`，它在第一条指令上崩溃，因为它被给了一个无效的 `this` 指针。

```
00007fff`73a8a59f mov edx,dword ptr [rcx+40h]
                                    ds:00000000`00000040=????????

```

`Widget` 指针来自于 `Standard­Widgets` 类的成员 `std::vector`。

```
using namespace Microsoft::WRL;

class StandardWidgets : RuntimeClass<IStandardWidgets, FtmBase>
{
    IFACEMETHODIMP get_TotalCost(INT32* result);
    ⟦ other methods not relevant here ⟧

private:
    HRESULT LazyInitializeWidgetList();

    static constexpr PCWSTR standardWidgetNames[] = {
        L"Bob", L"Carol", L"Ted", L"Alice" };
    static constexpr int standardWidgetCount =
        ARRAYSIZE(standardWidgetNames);

    std::vector<ComPtr<Widget>> m_widgets;
};

```

代码在对 `Widget::<wbr>GetCost` 的调用处崩溃：

```
IFACEMETHODIMP StandardWidgets::get_TotalCost(INT32* result)
{
    *result = 0;

    RETURN_IF_FAILED(LazyInitializeWidgetList());

    INT32 totalCost = 0;
    for (int i = 0; i < standardWidgetCount; i++) {
        totalCost += m_widgets[i]->GetCost(); // here
    }
    *result = totalCost;
    return S_OK;
}

```

客户的调试显示，在崩溃点，不仅 `widget` 是垃圾，而且 `m_widgets` 向量有着不可能的大量元素。预期 `m_widgets` 只有四个小部件，但它不知何故发现自己有十个，有时甚至有一百个小部件。当然，它们几乎都是损坏的。

这是懒惰初始化小部件列表的代码：

```
HRESULT StandardWidgets::LazyInitializeWidgetList()
{
    // Early-out if already initialized.
    if (!m_widgets.empty()) {
        return S_OK;
    }

    // Lazy-create the vector of standard widgets
    try {
        m_widgets.reserve(standardWidgetCount);
        for (auto name : standardWidgetNames) {
            ComPtr<Widget> widget;
            RETURN_IF_FAILED(
                MakeAndInitialize<Widget>(&widget, name));
            m_widgets.push_back(widget);
        }
    } catch (std::bad_alloc const&) {
        return E_OUTOFMEMORY;
    }
    return S_OK;
}

```

客户指出，`reserve` 方法始终以值 4 调用，代码从未向向量推送超过四个项目。他们承认，如果存在创建所有四个标准小部件的问题，则向量可能最终只有少于四个小部件，但永远不应该有 *超过* 四个。

你已经有了多个线索，指向客户问题的所在。我会给你一些时间去思考它。

与此同时，让我们看看代码懒惰初始化小部件列表的其他问题。

正如客户所指出的，如果有任何四个标准小部件中的一个创建问题，失败就会传播给 `Lazy­Initialize­Widget­List` 的调用者，并且 `get_TotalCost` 反过来将错误传播给自己的调用者，它永远不会到达遍历所有成本并将它们相加的地步。

如果在 `reserve()` 处发生内存分配失败，或者第一个标准小部件出现问题，则向量保持空白，第二次调用 `Lazy­Initialize­Widget­List` 将尝试新的初始化。

然而，如果第二个或后续标准小部件存在问题，情况就会变得奇怪。`Lazy­Initialize­Widget­List` 函数返回失败，导致 `get_TotalCost` 返回失败。但是第二次有人调用 `get_TotalCost` 时，`Lazy­Initialize­Widget­List` 将看到一个非空向量，并假设一切都已经初始化。这次，`get_TotalCost` 方法将继续进行求和，并在到达未能创建的小部件时执行越界数组访问。

糟糕。

这个特定问题归结为在惰性初始化失败时留下了一个部分初始化的`m_widgets`。为了避免这个问题，我们应该在一个局部变量中创建向量，并且只有在确保所有小部件都成功创建后才将其传输到成员变量中。

```
HRESULT StandardWidgets::LazyInitializeWidgetList()
{
    // Early-out if already initialized.
    if (!m_widgets.empty()) {
        return S_OK;
    }

    // Lazy-create the vector of standard widgets
    try {
        std::vector<ComPtr<Widget>> widgets;
        widgets.reserve(standardWidgetCount);
        for (auto name : standardWidgetNames) {
            ComPtr<Widget> widget;
            RETURN_IF_FAILED(
                MakeAndInitialize<Widget>(&widget, name));
            widgets.push_back(widget);
        }
        m_widgets.swap(widgets);
    } catch (std::bad_alloc const&) {
        return E_OUTOFMEMORY;
    }
    return S_OK;
}

```

这确保了`m_widgets`要么完全为空，要么完全初始化。它永远不处于半初始化状态。

顺便说一下，我们可能应该将`get_TotalCost`中的循环转换为一个范围`for`循环。现在，`get_<wbr>Total­Cost`对`Lazy­Initialize­Widgets`有一个隐藏的依赖性：它假设`Lazy­Initialize­Widgets`总是创建与`standard­Widget­Names`中的小部件数量完全相同的小部件。也许在将来，你可能想根据某些配置设置抑制一些标准小部件。如果你添加了该配置设置并忘记更新`get_<wbr>Total­Cost`以考虑被抑制的小部件，你将会有一个超出界限的索引。决定哪些小部件是标准的所有逻辑应该局限于`Lazy­Initialize­Widgets`。

```
IFACEMETHODIMP StandardWidgets::get_TotalCost(INT32* result)
{
    *result = 0;

    RETURN_IF_FAILED(LazyInitializeWidgetList());

    INT32 totalCost = 0;
    for (auto&& widget : m_widgets) { 
        totalCost += widget->GetCost();
    } 
    *result = totalCost;
    return S_OK;
}

```

或者如果你想搞酷一点，

```
IFACEMETHODIMP StandardWidgets::get_TotalCost(INT32* result)
{
    *result = 0;

    RETURN_IF_FAILED(LazyInitializeWidgetList());

    *result = std::transform_reduce( 
        m_widgets.begin(), m_widgets.end(), 0, std::plus<>(),
    [](auto&& w) { return w->GetCost(); }); 
    return S_OK;
}

```

好的，但回到崩溃。我认为我添加了足够的填充内容让你有时间考虑发生了什么。

当我查看这个崩溃时，我注意到该类是使用[Microsoft::<wbr>WRL::<wbr>RuntimeClass`模板类](https://learn.microsoft.com/en-us/cpp/cppcx/wrl/runtimeclass-class?view=msvc-170)实现的，并且实现明确将`FtmBase`列为模板参数，将此类标记为自由线程（也称为“灵活”），这意味着它可以同时从多个线程使用。¹

对象可以用于多线程使用，但没有互斥锁来保护两个线程同时修改`m_widgets`。我怀疑是竞争条件。

您还可以观察到该类正在以自由线程方式使用，因为导致崩溃的堆栈跟踪表明它正在一个线程池线程（`TppWorkerThread`）上运行，而线程池线程默认为多线程公寓。² 在`TppWorkerThread`和应用程序代码之间的堆栈上的唯一代码都是 COM 和 RPC，因此没有应用程序代码潜入并将线程初始化为单线程公寓模式。

当我查看崩溃转储时，我抓住了代码的犯罪现场：还有另一个线程也在调用此代码。

```
// Crashing thread
0:006> .frame 1
01 contoso!StandardWidgets::get_TotalCost+0x12f
0:006> dv
    this = 0x000001b7`492833b0
    ...

// Another thread running at the time of the crash
0:004> kn
 # Call Site
00 ntdll!ZwDelayExecution+0x14
01 ntdll!RtlDelayExecution+0x4c
02 KERNELBASE!SleepEx+0x84
04 kernel32!WerpReportFault+0xa4
05 KERNELBASE!UnhandledExceptionFilter+0xd3a02
06 ntdll!TppExceptionFilter+0x7a
07 ntdll!TppWorkerpInnerExceptionFilter+0x1a
08 ntdll!TppWorkerThread$filt$3+0x19
09 ntdll!__C_specific_handler+0x96
0a ntdll!__GSHandlerCheck_SEH+0x6a
0b ntdll!RtlpExecuteHandlerForException+0xf
0c ntdll!RtlDispatchException+0x2d4
0d ntdll!KiUserExceptionDispatch+0x2e
0e contoso!StandardWidgets::get_TotalCost+0x12f
0f rpcrt4!Invoke+0x73
10 rpcrt4!Ndr64StubWorker+0xb9b
11 rpcrt4!NdrStubCall3+0xd7
12 combase!CStdStubBuffer_Invoke+0xdb
14 combase!ObjectMethodExceptionHandlingAction<<lambda_...> >+0x47
16 combase!DefaultStubInvoke+0x376
1a combase!ServerCall::ContextInvoke+0x6f3
1f combase!ComInvokeWithLockAndIPID+0xacb
21 combase!ThreadInvoke+0x103
22 rpcrt4!DispatchToStubInCNoAvrf+0x18
23 rpcrt4!RPC_INTERFACE::DispatchToStubWorker+0x1a9
25 rpcrt4!RPC_INTERFACE::DispatchToStubWithObject+0x1a7
27 rpcrt4!LRPC_SCALL::DispatchRequest+0x308
29 rpcrt4!LRPC_SCALL::HandleRequest+0xdcb
2a rpcrt4!LRPC_SASSOCIATION::HandleRequest+0x2c3
2b rpcrt4!LRPC_ADDRESS::HandleRequest+0x183
2c rpcrt4!LRPC_ADDRESS::ProcessIO+0x939
2d rpcrt4!LrpcIoComplete+0xff
2e ntdll!TppAlpcpExecuteCallback+0x14d
2f ntdll!TppWorkerThread+0x4b4
30 kernel32!BaseThreadInitThunk+0x18
31 ntdll!RtlUserThreadStart+0x21
0:004> .frame 0xe
0e contoso!StandardWidgets::get_TotalCost+0x12f
0:004> dv
    this = 0x000001b7`492833b0
    ...

```

注意`this`指针对于两个线程是相同的，所以我们可以证明这个对象正在同时被多个线程使用。

修复多线程问题的方法是确保只有一个线程尝试一次性初始化小部件向量。我们可以通过添加互斥锁来实现这一点，但我将进一步使用`std::once_flag`，其生命周期的目的是与`std::<wbr>call_once`一起使用以执行线程安全的一次性初始化，这正是我们想要的。

```
class StandardWidgets : RuntimeClass<IStandardWidgets, FtmBase>
{
    IFACEMETHODIMP get_TotalCost(INT32* result);
    ⟦ other methods not relevant here ⟧

private:
    HRESULT LazyInitializeWidgetList();

    static constexpr PCWSTR standardWidgetNames[] = {
        L"Bob", L"Carol", L"Ted", L"Alice" };
    static constexpr int standardWidgetCount =
        ARRAYSIZE(standardWidgetNames);

    std::once_flag m_initializeFlag;
    std::vector<ComPtr<Widget>> m_widgets;
};

HRESULT StandardWidgets::LazyInitializeWidgetList()
{
    try {
        std::call_once(m_initializeFlag, [&] {
            std::vector<ComPtr<Widget>> widgets;
            widgets.reserve(standardWidgetCount);
            for (auto name : standardWidgetNames) {
                ComPtr<Widget> widget;
                THROW_IF_FAILED(
                    MakeAndInitialize<Widget>(&widget, name));
                widgets.push_back(widget);
            }
            m_widgets = std::move(widgets);
        });
    } catch (std::bad_alloc const&) {
        return E_OUTOFMEMORY;
    }
    return S_OK;
}

```

**更新**：我们不得不将`RETURN_<wbr>IF_<wbr>FAILED`更改为`THROW_<wbr>IF_<wbr>FAILED`，因为（1）没有进行更改，编译器会抱怨不是所有代码路径都返回值，因为 lambda 也会在结尾处结束，更重要的是，（2）`call_once`不关心 lambda 的返回值；它使用异常来检测错误。**更新结束**。

这个版本还处理了一个极端情况，即根本没有标准小部件的情况。原始代码会不断尝试重新初始化向量，因为它无法确定空向量是“尚未初始化”还是“成功初始化（且为空）”。

**额外的闲聊**：这个多线程竞争条件是如何导致一个大小荒谬的向量的呢？好吧，我们有些时候看到过，[`std::vector`的内部结构](https://devblogs.microsoft.com/oldnewthing/20230802-00/?p=108524 "STL 内幕：vector")是三个指针，一个用于向量数据的起始位置，一个用于有效数据的结束位置，一个用于分配数据的结束位置。如果两个线程同时调用`reserve()`，它们都将分配新数据，然后它们会竞争更新这三个指针。你可能会得到一个“起始”指针，它指向第一个线程分配的数据，但“结束”指针指向第二个线程分配的数据，从而得到一个异常大小的向量。

¹ 实际上，实现中明确列出了`FtmBase`是很方便的，因为默认行为取决于是否设置了`__WRL_<wbr>CONFIGURATION_<wbr>LEGACY__`。

| 模板参数 | 标准模式 | 传统模式 |
| --- | --- | --- |
| 未指定任何内容 | 自由线程 | 非自由线程 |
| `FtmBase` | 自由线程 |
| `InhibitFtmBase` | 非自由线程 |
| `InhibitFtmBase` + `FtmBase` | 非自由线程 |

伪代码中：

```
bool isFreeThreaded = !InhibitFtmBase && (FtmBase || Standard mode);

```

明确包含`FtmBase`省去了我查找客户项目使用的模式的麻烦。

² 假设多线程公寓真的存在。
