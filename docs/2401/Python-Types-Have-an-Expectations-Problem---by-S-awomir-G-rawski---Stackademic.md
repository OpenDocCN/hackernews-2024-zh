<!--yml

分类: 未分类

日期：2024-05-27 15:18:59

-->

# Python类型存在期望问题 | 作者：Sławomir Górawski | Stackademic

> 来源：[https://medium.com/@sgorawski/python-types-have-an-expectations-problem-ea71a8645ce8](https://medium.com/@sgorawski/python-types-have-an-expectations-problem-ea71a8645ce8)

# Python类型存在期望问题

照片由[Hitesh Choudhary](https://unsplash.com/@hiteshchoudhary?utm_source=medium&utm_medium=referral)提供在[Unsplash](https://unsplash.com/?utm_source=medium&utm_medium=referral)

# 背景

在过去的约10年左右，许多流行的脚本语言都增加了可选的静态类型：JavaScript（通过TypeScript），PHP，Python，甚至Ruby据我所知都有类似的东西。

它已经在每种语言的社区中相当普及，甚至被认为是应用程序的最佳实践，绝对是库的最佳实践。

看到类型注释有多么有用很容易 - 让我们看看这个没有类型注释的函数：

```
def check_permission(user, perm, obj):
    ...
```

它可能看起来很清楚它的作用 - 它检查`user`是否对`obj`拥有`perm`权限。

但是，如果我们想在某处使用它，立即会有一些问题浮现：

1.  `user`是应用程序的User模型的实例吗？（我猜的。）

1.  `perm`是某种权限模型的实例还是只是一些表示，比如字符串？这有点难，可能取决于所使用的框架的约定和具体的项目。

1.  `obj`是任何模型的实例，还是可能是模型的*类型*？如果检查的权限更广泛：我应该传递null还是在某处有不同的函数？

1.  如果用户具有/不具有权限，函数是否返回true/false？或者如果检查为负面，它是否不返回任何内容并抛出异常？

将其与类型版本进行比较：

```
def check_permission(user: User, perm: str, obj: BaseModel | None) -> bool:
    ...
```

它有点长，但它回答了我们所有的问题。（我们并不 *完全* 确定代码会做什么，但我们可以合理猜测。）此外，它可以捕获错误并帮助IDE进行自动完成。

那么，如果类型注释如此有用，为什么我会说在Python中它们存在“期望问题”呢？

好吧，让我们比较一下其他语言是如何做的。

# 比较

## JavaScript

JavaScript本身没有任何类型，你必须使用TypeScript。这是一个非常受欢迎的选择，所以开始起来不应该很难。

在源代码中我们可以有类似这样的东西：

```
function checkPermission(user: User, perm: string, obj?: BaseModel) {
	console.log(user, perm, obj);
}

checkPermission(1, 2, 3);
```

现在，TypeScript不会像这样在浏览器（或Node.js）中运行，我们必须编译它：

```
$ node_modules/.bin/tsc example.ts
error TS2345: Argument of type 'number' is not assignable to parameter of type 'User'.
```

它不会让我们通过。所以，如果我们成功编译并运行它，我们可以确信类型不匹配已经被修复，程序类型检查已经通过。

## PHP

现在，让我们在PHP中做类似的事情：

```
<?php
function checkPermission(User $user, string $perm, ?BaseModel $obj) {
    var_dump($user, $perm, $obj);
}

checkPermission(1, 2, 3);
```

在PHP中没有编译步骤。但是，如果我们运行这段代码，它将在运行时崩溃：

```
$ php -f example.php
Uncaught TypeError: checkPermission(): Argument #1 ($user) must be of type User, int given
```

是的，也许抛出异常不如早点捕获好，但至少我们可以确信如果类型不对，函数代码不会被执行。

## Python

进入Python：

```
def check_permission(user: User, perm: str, obj: BaseModel | None):
    print(user, perm, obj)

check_permission(1, 2, 3)
```

```
$ python3 example.py
1 2 3
```

我们运行了这个代码，类型注释毫无作用！它使用了错误类型的参数调用了函数，就好像我们没有做任何事情来防止这种情况发生。

现在，对于熟悉 Python 的人来说，这将是非常明显的 —— 类型提示在运行时不起作用，它们只是供类型检查器程序使用的信息，比如 MyPy，你必须单独运行它。

但根据我们在其他语言中看到的情况，这可能一点也不明显。

在我所有的示例中，我使用了给定语言最流行的静态类型解决方案编写了代码，然后我尽可能简单地使代码在我的应用程序中运行。在 TypeScript 的情况下，我必须将其编译为 JS 来运行代码，然后它就能捕获错误了。在 PHP 中，我“只是”运行它，它在运行时捕获了类型不匹配。在 Python 中，我也可以直接运行它 —— 不需要编译步骤 —— 然后我的类型提示在运行时什么也没做！

因此，如果我有给定语言的源代码和一个正在运行的应用程序，那我可以从中得知：

现在，如果我知道有人在运行代码之前对代码库进行了正确配置的类型检查，那么 Python 的最后答案将会有所不同 —— 但我怎么能知道呢？我必须设置一些带有类型检查步骤的 CI，并且确保无法部署未通过检查的代码，或许是这样。如果我没有那样的基础设施，我就无能为力了。

Python 类型提示是语言的核心部分，它们甚至有标准库模块（`typing`），但是在该语言中使用时却没有任何外部工具的情况下它们什么也不做。

对我来说，这有点期望不符。

# 结论

现在，如果 Python 类型注释实际上看起来像它们所代表的信息 —— 供外部工具使用，而不是语言的“工作”部分 —— 那可能会更清晰。就像注释一样。而事实上，确实*有*一种官方的语法！

```
def check_permission(user, perm, obj):
    # type: (User, str, BaseModel | None) -> bool
    return False
```

它与 MyPy 配合使用，但在这一点上它相当小众（旨在与旧版 Python 兼容），与 PyCharm 等工具的兼容性不是太好，而且可能无法表达注释所能表达的一切。

真可惜，因为对我来说它传达了更好的期望 —— 它是一条注释，所以它是供外部工具使用的信息，而且在运行时不起作用。所有这些都是可以预料的。

# Stackademic 🎓

感谢您一直阅读到最后。在您离开之前：
