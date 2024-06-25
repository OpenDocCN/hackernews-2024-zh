<!--yml

类别：未分类

日期：2024-05-27 14:40:05

-->

# Objective-C 的 BOOL 类型是布尔类型吗？这取决于具体情况。

> 来源：[https://www.jviotti.com/2024/01/05/is-objective-c-bool-a-boolean-type-it-depends.html](https://www.jviotti.com/2024/01/05/is-objective-c-bool-a-boolean-type-it-depends.html)

Objective-C 编程语言引入了自己的类型来表示布尔值：[`BOOL`](https://developer.apple.com/documentation/objectivec/bool)。该类型的实例是常量 [`YES`](https://developer.apple.com/documentation/objectivec/yes) 和 [`NO`](https://developer.apple.com/documentation/objectivec/no)。

这是一个简单的例子，调用接受`BOOL`值作为参数的函数：

```
void print_boolean(BOOL value) {
 NSLog(value ? @"True" : @"False");
}

print_boolean(YES);
print_boolean(NO);
```

虽然`BOOL`看起来很琐碎，但其定义却相当复杂。它取决于你正在针对哪个苹果平台和架构，这可能导致意外的行为。

> 本文基于运行在 2020 M1 MacBook Pro 上的 macOS Sonoma 14.2.1 上的 Xcode 15.1（15C65）。

## 意外行为的一个例子

我写了很多 Objective-C++。C++ 的一个有用特性是[函数重载](https://learn.microsoft.com/en-us/cpp/cpp/function-overloading?view=msvc-170)：即定义多个函数，它们只在接受的参数类型上有所不同而名称相同。当你调用这样的函数时，编译器会自动确定调用哪个重载。

最近，我遇到了一个情况，对于相同的代码，macOS Intel 和 macOS Apple Silicon 调用了不同的重载。以下是该问题的一个最小可复现的 Objective-C++ 示例：

```
// To locally run this example:
// $ xcrun clang++ main.mm -o objc-bool-test
// $ ./objc-bool-test

#include <objc/runtime.h>
#include <iostream>
#include <cstdlib>

void print(bool value) { std::cout << "(bool) " << value << "\n"; }
void print(int value) { std::cout << "(int) " << value << "\n"; }

int main() {
 std::cout << "YES: ";
 print(YES);
 std::cout << "NO: ";
 print(NO);
 return EXIT_SUCCESS;
}
```

在我的 Apple Silicon Mac 上（运行 Xcode 15.1），上述程序调用了`bool`重载：

```
YES: (bool) 1
NO: (bool) 0
```

但是，在我的 macOS Intel Mac 上（运行 Xcode 14.3.2），上述程序调用了`int`重载：

```
YES: (int) 1
NO: (int) 0
```

## 探索 Objective-C 运行时

苹果芯片和 Intel 之间的这种差异在文档中没有明确说明，文档在某种程度上似乎自相矛盾。[文档](https://developer.apple.com/documentation/objectivec/bool)表示`BOOL`的规范定义是`bool`的类型别名。然而，在*特殊注意事项*下，文档说明了*`BOOL`的类型实际上是`char`*。

当文档无法帮助时，我们可以深入了解。`BOOL`类型由 [Objective-C](https://developer.apple.com/documentation/objectivec/objective-c_runtime?language=objc) 运行时定义，其公共头文件由 Xcode 在以下目录中分发：

```
$ ls $(xcode-select --print-path)/Platforms/MacOSX.platform/Developer/SDKs/MacOSX.sdk/usr/include/objc
List.h               Protocol.h           objc-api.h           objc-runtime.h
NSObjCRuntime.h      hashtable.h          objc-auto.h          objc-sync.h
NSObject.h           hashtable2.h         objc-class.h         objc.h
Object.h             message.h            objc-exception.h     runtime.h
ObjectiveC.apinotes  module.modulemap     objc-load.h
```

我们可以通过快速搜索确认，与布尔值相关的头文件是`objc.h`，该文件被`runtime.h`所包含，后者是 Objective-C 运行时的入口点。

### `YES` 和 `NO` 常量

这些布尔常量有两种定义方式：

```
// $(xcode-select --print-path)/Platforms/MacOSX.platform/Developer/SDKs/MacOSX.sdk/usr/include/objc/objc.h
...
#if __has_feature(objc_bool)
#define YES __objc_yes
#define NO  __objc_no
#else
#define YES ((BOOL)1)
#define NO  ((BOOL)0)
#endif
...
```

[`__has_feature`](https://clang.llvm.org/docs/LanguageExtensions.html#has-feature-and-has-extension) 宏被 LLVM 定义为用于内省编译器特性的语言扩展。虽然不是标准，但 [GCC 也支持 `__has_feature`](https://gcc.gnu.org/onlinedocs/cpp/_005f_005fhas_005ffeature.html)。但是，Objective-C 运行时头文件引用的 `objc_bool` 特性从未被 LLVM 定义。因此，我们感兴趣的定义是第二个定义：

```
#define YES ((BOOL)1)
#define NO  ((BOOL)0)
```

如果你感兴趣，定义的第一个子句中的 `__objc_yes` 和 `__objc_no` 符号是由 LLVM 解析器识别的内置类型，并可供使用。例如，下面的程序是有效的：

```
void print_boolean(BOOL value) {
 NSLog(value ? @"True" : @"False");
}

print_boolean(__objc_yes);
print_boolean(__objc_no);
```

### `BOOL` 类型别名

现在我们知道 `YES` 和 `NO` 是通过将数字常量 `1` 和 `0` 强制转换为 `BOOL` 定义的，让我们把注意力转向 `BOOL` 类型。这个类型别名在 `objc.h` 头文件中定义如下：

```
// $(xcode-select --print-path)/Platforms/MacOSX.platform/Developer/SDKs/MacOSX.sdk/usr/include/objc/objc.h
...
#if OBJC_BOOL_IS_BOOL
 typedef bool BOOL;
#else
#   define OBJC_BOOL_IS_CHAR 1
 typedef signed char BOOL;
 // BOOL is explicitly signed so @encode(BOOL) == "c" rather than "C"
 // even if -funsigned-char is used.
#endif
...
```

正如我们所见，`BOOL` 类型是根据 `OBJC_BOOL_IS_BOOL` 预处理定义的值而别名为 `bool` 或别名为 `signed char`。此外，如果 `BOOL` 设置为后者，Objective-C 运行时将定义 `OBJC_BOOL_IS_CHAR`。该定义方便编写必须对底层 `BOOL` 类型做出假设的代码。例如：

```
#import <Foundation/Foundation.h>

int main() {
#if OBJC_BOOL_IS_CHAR
 NSLog(@"My BOOL is a signed char");
#else
 NSLog(@"My BOOL is a bool");
#endif
 return EXIT_SUCCESS;
}
```

### `OBJC_BOOL_IS_BOOL` 的定义

我们了解到 `BOOL` 是否是别名到 `bool` 或到 `signed char` 取决于 `OBJC_BOOL_IS_BOOL` 预处理定义的值。像我们迄今所探讨的其他常量和别名一样，这个定义也在 `objc.h` 头文件中声明：

```
// $(xcode-select --print-path)/Platforms/MacOSX.platform/Developer/SDKs/MacOSX.sdk/usr/include/objc/objc.h
...
#if defined(__OBJC_BOOL_IS_BOOL)
 // Honor __OBJC_BOOL_IS_BOOL when available.
#   if __OBJC_BOOL_IS_BOOL
#       define OBJC_BOOL_IS_BOOL 1
#   else
#       define OBJC_BOOL_IS_BOOL 0
#   endif
#else
 // __OBJC_BOOL_IS_BOOL not set.
#   if TARGET_OS_OSX || TARGET_OS_MACCATALYST || ((TARGET_OS_IOS || 0) && !__LP64__ && !__ARM_ARCH_7K)
#      define OBJC_BOOL_IS_BOOL 0
#   else
#      define OBJC_BOOL_IS_BOOL 1
#   endif
#endif
...
```

第一个子句很直接：如果存在 `__OBJC_BOOL_IS_BOOL` 预处理定义，则将 `OBJC_BOOL_IS_BOOL` 设置为它。

但是，第二个子句涉及的内容更加复杂。代码表明，在以下条件之一下，Objective-C 运行时将 `BOOL` 别名为 `signed char`：

+   如果你正在编写一个 macOS 应用程序（`TARGET_OS_OSX`），独立于目标架构

+   如果你正在编写一个 [Mac Catalyst](https://developer.apple.com/mac-catalyst/) 应用程序（`TARGET_OS_MACCATALYST`）

+   如果你正在编写一个不使用 [LP64 数据模型](https://en.wikipedia.org/wiki/64-bit_computing#64-bit_data_models) (`__LP64__`) 且不是 ARMv7 芯片（[`__ARM_ARCH_7K`](https://opensource.apple.com/source/xnu/xnu-4570.1.46/osfmk/arm/arch.h.auto.html)）的 iOS 应用程序（`TARGET_OS_IOS`）

我们可以推断出第二个子句在这里不适用，因为我们在 macOS Intel 和 macOS Apple Silicon 中看到了 `BOOL` 的定义存在差异。否则，两种架构都会将 `BOOL` 别名为 `signed char`，因为存在 `TARGET_OS_OSX`。因此，我们可以得出结论，`__OBJC_BOOL_IS_BOOL` 总是被设置的，我们可以通过以下简单的程序确认：

```
// main.m
#import <Foundation/Foundation.h>

int main() {
#if defined(__OBJC_BOOL_IS_BOOL)
 NSLog(@"__OBJC_BOOL_IS_BOOL is %i", __OBJC_BOOL_IS_BOOL);
#else
 NSLog(@"__OBJC_BOOL_IS_BOOL is not defined");
#endif
 return EXIT_SUCCESS;
}
```

如预期，`__OBJC_BOOL_IS_BOOL`在我的Apple Silicon Mac和我的Intel Mac上分别定义为1和0。 此外，[有一个测试](https://github.com/apple-oss-distributions/objc4/blob/objc4-906.2/test/bool.c#L23-L25)在[Objective-C runtime源代码](https://github.com/apple-oss-distributions/objc4)中断言`__OBJC_BOOL_IS_BOOL`始终被定义：

```
// https://github.com/apple-oss-distributions/objc4/blob/objc4-906.2/test/bool.c
...
#if __OBJC__ && !defined(__OBJC_BOOL_IS_BOOL)
#   error no __OBJC_BOOL_IS_BOOL
#endif
...
```

## 调整`__OBJC_BOOL_IS_BOOL`

现在我们知道`__OBJC_BOOL_IS_BOOL`是最终控制`BOOL`别名的基础类型的预处理器定义，我们可以尝试控制它。

> 我在这里这样做是为了实验目的，但我强烈不建议更改`__OBJC_BOOL_IS_BOOL`的定义（主要是在生产代码中！），因为我们不知道它对依赖Objective-C runtime的框架的影响有多深。 也许Apple为不同的平台和体系结构设置不同别名的好理由。 相反，如果您需要对`BOOL`进行内省，可能最好使用我们之前讨论过的`OBJC_BOOL_IS_CHAR`定义。

在我的Apple Silicon Mac上，我可以成功构建并运行本文开头的[*意外行为*示例](#an-example-of-unexpected-behavior)，强制 Objective-C runtime 将`BOOL`别名为`signed char`，方法是将 `__OBJC_BOOL_IS_BOOL` 明确设置为 `0`：

```
$ xcrun clang++ main.mm -o force-signed-char -D__OBJC_BOOL_IS_BOOL=0
In file included from <built-in>:444:
<command line>:1:9: warning: '__OBJC_BOOL_IS_BOOL' macro redefined [-Wmacro-redefined]
#define __OBJC_BOOL_IS_BOOL 0
 ^
<built-in>:34:9: note: previous definition is here
#define __OBJC_BOOL_IS_BOOL 1
 ^
1 warning generated.

$ ./force-signed-char
YES: (int) 1
NO: (int) 0
```

尽管它起作用并且我们成功地影响了别名，但让我们仔细查看编译警告。 编译器警告我们正在覆盖`__OBJC_BOOL_IS_BOOL`，而它（正如我们在Apple Silicon上所预期的那样）之前是这样定义的：

```
#define __OBJC_BOOL_IS_BOOL 1
```

但是，这个定义并非来自Objective-C runtime或任何其他头文件。 相反，它直接来自LLVM，正如编译器与我们共享的源位置所示：

```
<built-in>:34:9
```

使用 `grep(1)`，我们可以确认 Objective-C runtime 公共头文件从不声明（只使用）`__OBJC_BOOL_IS_BOOL`定义：

```
$ cd $(xcode-select --print-path)/Platforms/MacOSX.platform/Developer/SDKs/MacOSX.sdk
$ grep --recursive '__OBJC_BOOL_IS_BOOL' usr/include/objc
usr/include/objc/objc.h:#if defined(__OBJC_BOOL_IS_BOOL)
usr/include/objc/objc.h:    // Honor __OBJC_BOOL_IS_BOOL when available.
usr/include/objc/objc.h:#   if __OBJC_BOOL_IS_BOOL
usr/include/objc/objc.h:    // __OBJC_BOOL_IS_BOOL not set.
```

## 深入挖掘 LLVM

要找到神秘的`__OBJC_BOOL_IS_BOOL`定义，让我们深入挖掘 LLVM。 在我的主要机器上，我正在运行 AppleClang 1500.1.0.2.5（Xcode 15.1），它[对应 LLVM 16](https://en.wikipedia.org/wiki/Xcode)。 通过在这种 LLVM 版本中搜索`__OBJC_BOOL_IS_BOOL`的出现，我们可以发现[`clang/lib/Frontend/InitPreprocessor.cpp`](https://github.com/llvm/llvm-project/blob/llvmorg-16.0.0/clang/lib/Frontend/InitPreprocessor.cpp)根据名为`useSignedCharForObjCBool`的方法设置`__OBJC_BOOL_IS_BOOL`：

```
// See https://github.com/llvm/llvm-project/blob/llvmorg-16.0.0/clang/lib/Frontend/InitPreprocessor.cpp#L859-L862
...
// Define a macro that describes the Objective-C boolean type even for C
// and C++ since BOOL can be used from non Objective-C code.
Builder.defineMacro("__OBJC_BOOL_IS_BOOL",
 Twine(TI.useSignedCharForObjCBool() ? "0" : "1"));
...
```

反过来，`TargetInfo`类的`useSignedCharForObjCBool`方法在[`clang/include/clang/Basic/TargetInfo.h`](https://github.com/llvm/llvm-project/blob/llvmorg-16.0.0/clang/include/clang/Basic/TargetInfo.h)中定义为简单地返回`UseSignedCharForObjCBool`布尔类成员：

```
// See https://github.com/llvm/llvm-project/blob/llvmorg-16.0.0/clang/include/clang/Basic/TargetInfo.h#L848-L855
...
/// Check if the Objective-C built-in boolean type should be signed
/// char.
///
/// Otherwise, if this returns false, the normal built-in boolean type
/// should also be used for Objective-C.
bool useSignedCharForObjCBool() const {
 return UseSignedCharForObjCBool;
}
...
```

默认情况下，`UseSignedCharForObjCBool`布尔类成员设置为`true`：

```
// See https://github.com/llvm/llvm-project/blob/llvmorg-16.0.0/clang/lib/Basic/TargetInfo.cpp#L135
...
TargetInfo::TargetInfo(const llvm::Triple &T) : Triple(T) {
...
 UseSignedCharForObjCBool = true;
...
}
...
```

### Objective-C 到 C++ 重写器

在`TargetInfo`类中有一个直接影响`UseSignedCharForObjCBool`布尔类成员值的方法。此方法称为`noSignedCharForObjCBool`，正如其名称所示，将`UseSignedCharForObjCBool`设置为`false`。其定义如下：

```
// See https://github.com/llvm/llvm-project/blob/llvmorg-16.0.0/clang/include/clang/Basic/TargetInfo.h#L856-L858
...
void noSignedCharForObjCBool() {
 UseSignedCharForObjCBool = false;
}
...
```

有趣的是，此方法被调用的唯一位置是[`clang/lib/Frontend/CompilerInstance.cpp`](https://github.com/llvm/llvm-project/blob/llvmorg-16.0.0/clang/lib/Frontend/CompilerInstance.cpp)，在这个位置上，我们广泛讨论了 Objective-C 到 C++ 重写器，将`BOOL`无条件别名为`bool`。

```
// See https://github.com/llvm/llvm-project/blob/llvmorg-16.0.0/clang/lib/Frontend/CompilerInstance.cpp#L1023-L1025
...
// rewriter project will change target built-in bool type from its default.
if (getFrontendOpts().ProgramAction == frontend::RewriteObjC)
 getTarget().noSignedCharForObjCBool();
...
```

如果你好奇，你可以阅读[LLVM 提交](https://github.com/llvm/llvm-project/commit/29898f45657314c7fa7f61e8157a36c832015fcc)，该提交最初引入了`noSignedCharForObjCBool`方法，并选择了 Objective-C 到 C++ 重写器中的本地`bool`类型。

### Objective-C 编译器

撇开 Objective-C 实验性 C++ 重写器不谈，产品级准备就绪的 Objective-C 编译器支持的目标在从`TargetInfo`派生时设置`UseSignedCharForObjCBool`布尔类成员。

我们可以使用`grep(1)`来查找提及`UseSignedCharForObjCBool`的每个目标子类，如下所示：

```
$ grep --recursive 'UseSignedCharForObjCBool' clang/lib/Basic/Targets | uniq
clang/lib/Basic/Targets/AArch64.cpp:  UseSignedCharForObjCBool = false;
clang/lib/Basic/Targets/X86.h:      UseSignedCharForObjCBool = false;
clang/lib/Basic/Targets/ARM.cpp:    UseSignedCharForObjCBool = false;
```

让我们仔细看看每个匹配项。

#### `AArch64`

AArch64 目标对应于 64 位 ARM 芯片，例如 Apple Silicon。对于这个目标，`UseSignedCharForObjCBool`无条件地设置为`false`，因此`__OBJC_BOOL_IS_BOOL`预处理器定义始终为`1`：

```
// See https://github.com/llvm/llvm-project/blob/llvmorg-16.0.0/clang/lib/Basic/Targets/AArch64.cpp#L1432
...
DarwinAArch64TargetInfo::DarwinAArch64TargetInfo(const llvm::Triple &Triple,
 const TargetOptions &Opts)
 : DarwinTargetInfo<AArch64leTargetInfo>(Triple, Opts) {
 ...
 UseSignedCharForObjCBool = false;
 ...
}
...
```

正如预期的那样，这与我们在文章开头看到的行为相匹配：对于 Apple Silicon Mac，Objective-C 运行时将`BOOL`别名为`bool`。

#### `X86`

对于 32 位英特尔芯片，`BOOL`始终是`signed char`的别名，除了 watchOS 外。在其前 8 个版本（直到 2022 年），watchOS 附带了 [32 位支持](https://en.wikipedia.org/wiki/WatchOS#watchOS_8)，因此此配置适用于在英特尔 Mac 上运行旧 watchOS 模拟器：

```
// See https://github.com/llvm/llvm-project/blob/llvmorg-16.0.0/clang/lib/Basic/Targets/X86.h#L533-L536
...
DarwinI386TargetInfo(const llvm::Triple &Triple, const TargetOptions &Opts)
 : DarwinTargetInfo<X86_32TargetInfo>(Triple, Opts) {
 ...
 // The watchOS simulator uses the builtin bool type for Objective-C.
 llvm::Triple T = llvm::Triple(Triple);
 if (T.isWatchOS())
 UseSignedCharForObjCBool = false;
 ...
}
...
```

对于 64 位英特尔芯片，`BOOL`始终是`signed char`的别名，除了 iOS 外。正如注释所解释的那样，此配置用于在英特尔 Mac 上运行 iOS 模拟器的情况：

```
// See https://github.com/llvm/llvm-project/blob/llvmorg-16.0.0/clang/lib/Basic/Targets/X86.h#L915-L918
...
DarwinX86_64TargetInfo(const llvm::Triple &Triple, const TargetOptions &Opts)
 : DarwinTargetInfo<X86_64TargetInfo>(Triple, Opts) {
 ...
 // The 64-bit iOS simulator uses the builtin bool type for Objective-C.
 llvm::Triple T = llvm::Triple(Triple);
 if (T.isiOS())
 UseSignedCharForObjCBool = false;
 ...
}
...
```

正如预期的那样，这与我们在文章开头看到的行为相匹配：对于英特尔 Mac，Objective-C 运行时将`BOOL`别名为`signed char`。

#### `ARM`

最后，对于 32 位 ARM 芯片，`BOOL`始终是`signed char`的别名，除了 watchOS 外。watchOS 产品始终定位到 ARM，因此此配置可能用于生产 watchOS 部署，直到版本 8（之后移除了 32 位支持）：

```
// See https://github.com/llvm/llvm-project/blob/llvmorg-16.0.0/clang/lib/Basic/Targets/ARM.cpp#L1412-L1417
...
DarwinARMTargetInfo::DarwinARMTargetInfo(const llvm::Triple &Triple,
 const TargetOptions &Opts)
 : DarwinTargetInfo<ARMleTargetInfo>(Triple, Opts) {
 ...
 if (Triple.isWatchABI()) {
 ...
 // BOOL should be a real boolean on the new ABI
 UseSignedCharForObjCBool = false;
 ...
}
...
```

如果你想要更高层次的视角，Objective-C 运行时的测试套件对于确定在受测试平台上是否应该应用“真实”布尔类型有一个[有趣的案例](https://github.com/apple-oss-distributions/objc4/blob/objc4-906.2/test/bool.c#L7-L21)。

## 概要

如果你已经读到这里，你可能想知道`BOOL`是如何变得如此复杂的。虽然没有苹果内部人士的知识，我们无法确定，但我认为原因是历史性的。

早在1984年，Objective-C被设计为C语言的严格超集。当时，C语言没有内置支持布尔值，而Objective-C将`signed char`重新用于保存布尔值的决定是合理的。你可以在这里看到一个古老的`BOOL`定义，它无条件地别名为`signed char`。[这里](https://opensource.apple.com/source/objc4/objc4-237/runtime/objc.h.auto.html)。

十多年后，作为C99规范的一部分，C语言通过[`<stdbool.h>`](https://pubs.opengroup.org/onlinepubs/9699919799/basedefs/stdbool.h.html)头文件发布了对布尔值的支持。然后，Objective-C运行时的更新版本开始有条件地将`BOOL`别名为现代苹果产品中的新`bool`类型。老的平台和架构组合可能仍然出于传统原因使用`signed char`。

> 如果你想继续了解`BOOL`类型的奇闻趣事，你可能也会喜欢[BOOL的尖锐边缘](https://bignerdranch.com/blog/bools-sharp-corners/)，作者是Big Nerd Ranch，以及Google的[Objective-C样式指南](https://google.github.io/styleguide/objcguide.html#bool-pitfalls)。

**HN 讨论**：[https://news.ycombinator.com/item?id=38909377](https://news.ycombinator.com/item?id=38909377)。
