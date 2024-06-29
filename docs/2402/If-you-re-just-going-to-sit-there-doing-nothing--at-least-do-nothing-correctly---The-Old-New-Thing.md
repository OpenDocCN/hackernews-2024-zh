<!--yml

category: 未分类

date: 2024-05-27 14:54:45

-->

# 如果你只是坐在那里什么也不做，至少要以正确的方式什么也不做 - 《The Old New Thing》

> 来源：[https://devblogs.microsoft.com/oldnewthing/20240216-00/?p=109409](https://devblogs.microsoft.com/oldnewthing/20240216-00/?p=109409)

# 如果你只是坐在那里什么也不做，至少要以正确的方式什么也不做

有时候你可能需要让一个 API 什么也不做。重要的是以正确的方式使其什么也不做。

例如，Windows 拥有庞大的打印基础设施。但这种基础设施在 Xbox 上不存在。如果一个应用程序试图在 Xbox 上打印会发生什么？

做的错误的事情是让打印函数抛出 `Not­Supported­Exception` 异常。用户在 Xbox 上安装的应用程序可能主要（如果不是唯一地）在 PC 上进行测试，而在 PC 上打印总是可用的。当在 Xbox 上运行时，异常可能不会被处理，应用程序会崩溃。即使应用程序试图捕获异常，它可能会显示像“哎呀。出了些问题。请联系支持并提供此事件代码。”这样的消息。

在 Xbox 上“支持”打印的更好设计是使打印功能成功，但报告没有安装打印机。使用此行为，当应用程序尝试打印时，它将要求用户选择打印机，并显示一个空列表。用户意识到，“哦，没有打印机”，然后取消打印请求。

为了处理那些变得复杂并说“哦，你没有安装打印机，让我帮你安装一个”的应用程序，安装打印机的函数可以立即返回一个结果代码，“用户取消了操作”。

这里的想法是所有打印函数都表现出与完全支持打印一致的方式，但神秘地从来没有打印机可以打印。

现在，您可能还希望添加一个功能来检查打印是否有效。如果应用程序在不支持打印的系统上运行，可以使用此功能从其用户界面隐藏“打印”按钮。但是，天真的应用程序假设打印功能正常运行，仍将表现出合理的行为：您只是在一个没有打印机的系统上，所有安装打印机的尝试都无效。

我们用来描述这种“什么也不做”的行为的名称是“惰性”。

API 表面仍然存在并根据其规范运行，但它也什么也不做。重要的是，它以与其文档一致且最不可能与现有代码创建问题的方式什么也不做。

另一个例子是退休的 API，该 API 具有用于创建小部件句柄的各种功能，接受小部件句柄的其他功能以及关闭小部件句柄的功能。最初建议将进行退休的 API 使其变得惰性如下：

```
HRESULT CreateWidget(_Out_ HWIDGET* widget)
{
    *widget = nullptr;
    return S_OK;
}

// Every widget is documented to have at least one alias,
// so we have to produce one dummy alias (empty string).
HRESULT GetWidgetAliases(
    _Out_writes_to_(capacity, *actual) PWSTR* aliases,
    UINT capacity,
    _Out_ UINT* actual)
{
    *actual = 0;

    RETURN_HR_IF(
        HRESULT_FROM_WIN32(ERROR_MORE_DATA),
        capacity < 1);

    aliases[0] = make_cotaskmem_string_nothrow(L"").release();
    RETURN_IF_NULL_ALLOC(aliases[0]);

    *actual = 1;
    return S_OK;
}

// Inert widgets cannot be enabled or disabled.
HRESULT EnableWidget(HWIDGET widget, BOOL value)
{
    return E_HANDLE;
}

HRESULT Close(HWIDGET widget)
{
    RETURN_HR_IF(E_INVALIDARG, widget != nullptr);
    return S_OK;
}

```

我指出，使`Create­Widget`成功但返回空指针会使应用程序混淆。“调用成功了，但我没有得到一个有效的句柄？”我甚至找到了他们自己的一些测试代码，检查句柄是否为空来确定调用是否成功，而不是检查返回值。

我还指出，使`Enable­Widget`返回“无效句柄”也会造成混淆。一个应用程序调用`Create­Widget`，它成功了，并且拿到了那个句柄（应该是有效的），然后尝试使用它来启用一个小部件，却被告知“那个句柄无效”。这怎么可能？“我要求一个小部件，你给了我一个，然后当我给你看它时，你说，‘那不是一个小部件。’这个API在[气灯]我！”

我查看了他们API的现有文档，发现一个记录的返回值是`ERROR_<wbr>CANCELLED`，表示用户取消了小部件的创建。因此，应用程序已经处理了由于其控制范围之外的条件而无法创建小部件的可能性，因此我们可以利用这一点：每次应用程序尝试创建一个小部件，只需说“不，嗯，用户取消了，是的，就是这样。”

```
HRESULT CreateWidget(_Out_ HWIDGET* widget)
{
    *widget = nullptr;
    return HRESULT_FROM_WIN32(ERROR_CANCELLED);
}

HRESULT GetWidgetAliases(
    _Out_writes_to_(capacity, *actual) PWSTR* aliases,
    UINT capacity,
    _Out_ UINT* actual)
{
    *actual = 0;
    return E_HANDLE;
}

HRESULT EnableWidget(HWIDGET widget, BOOL value)
{
    return E_HANDLE;
}

HRESULT Close(HWIDGET widget)
{
    return E_HANDLE;
}

```

现在我们有了一个合适的惰性API表面。

如果您试图创建一个小部件，我们会告诉您由于用户取消了，我们无法创建。由于所有创建小部件的尝试都失败了，因此不存在有效的小部件句柄，每次您尝试使用一个句柄时，我们会告诉您该句柄无效。

这还避免了为小部件产生虚拟别名的问题。因为*没有小部件*，不存在合法的情况，应用程序可以要求一个小部件的别名。

**额外的废话**: 为了消除一些混淆：这里的想法是打印API始终存在于支持打印的桌面上，而“获取打印机列表”的函数文件中没有抛出异常。如果您想要将打印API移植到Xbox上，该如何做到以便现有的桌面应用程序继续在Xbox上运行？惰性行为完全真实：Xbox上没有打印机。没有人期望问题的答案是，“有多少台打印机？”是“你竟敢问我这样的问题！”

另一种需要创建惰性API表面的场景是，如果您希望淘汰现有的API。在保持API行为与其契约一致的同时，如何确保仍然没有产生实际作用？
