<!--yml

category: 未分类

date: 2024-05-29 12:48:59

-->

# Dart中的JS互操作历史。Support for Wasm just landed in the… | 作者：Sigmund Cherem | Dart | Medium

> 来源：[https://medium.com/dartlang/history-of-js-interop-in-dart-98b06991158f](https://medium.com/dartlang/history-of-js-interop-in-dart-98b06991158f)

# Dart中JS互操作的历史

## 在当前的Flutter beta版中，由于Dart 3.3达到了令人振奋的JavaScript互操作里程碑，因此对Wasm的支持已经到位。为了庆祝这一事件，让我们回顾一下Dart和JS互操作的十年之旅。

由Gemini生成的AI图像

从Dart开始，互操作性一直是核心关注点。当Dart首次在2011年发布时，它被设计为*可嵌入*和*多平台*。它运行在独立的虚拟机上，嵌入在浏览器中，并编译为JavaScript。当Flutter在2015年出现时，我们也准备好将其嵌入其中。现在，我们对于[目标WasmGC运行时](https://docs.flutter.dev/platform-integration/web/wasm)感到兴奋。

最初，我们迅速工作，以暴露Dart嵌入的每个平台的功能。这就是我们的SDK平台特定库出现的方式：`[dart:io](https://api.dart.dev/stable/dart-io/dart-io-library.html)`在VM上暴露文件系统，`[dart:html](https://api.dart.dev/stable/dart-html/dart-html-library.html)`在Web上暴露浏览器API等等。这些库看起来和感觉像常规的Dart库，但背后隐藏着一些复杂的低级原语使它们工作。这是我们发明的第一种互操作形式。它很富有表现力，但仅限于SDK库。

在Web上，开发人员需要访问的不仅仅是浏览器API。因此，我们开始研究如何扩展互操作性以覆盖更多目标。作为起点，我们在2013年引入了`[dart:js](https://codereview.chromium.org//15782009)`来启用对JavaScript库的访问。

```
// Short example JavaScript code to illustrate Dart/JS interop
window.myTopLevel = {
  field1: 0,
  method2() {
    return this.field1;
  }
}
```

```
// Access via `dart:js` (2013)
import 'dart:js' as js;

void main() {
  // This line has a typo! oops :(
  var object = js.context['myTopLevl'];
  object['field1'] = 1;
  // This call fails with a noSuchMethod because method2
  // returns an int, oops
  object.callMethod('method2', []).substr(1);
}
```

我们当时知道`dart:js`不是我们想要的编程模型。你必须使用字符串来访问JavaScript中的名称——别想在编译时找到问题，更别提代码补全了！而且实现成本也很高。它在大多数操作中严重依赖盒子和深复制。因此，我们在2014年和2015年继续[草拟](https://github.com/dart-archive/js-interop-deprecated)想法，直到发布了`package:js`的v0.6版本。

```
// Access via `package:js` (2015)
import 'package:js/js.dart';

// Magic annotations allow us to declare API signatures:
@JS()
class MyObject {
  external int get field1;
  external void set field1(int value);
  external String method2();
}

@JS()
external MyObject get myTopLevel;

void main() {
  // Access to code is less error prone: analyzer can check that
  // these symbols match a declaration, and we get code-completion too!
  var object = myTopLevel;
  object.field1 = 1;
  // But types are not checked, this unsoundly invokes substring on an int
  object.method2().substring(1);
}
```

使用 `[package:js](https://pub.dev/documentation/js/latest/)`，我们最终拥有了一个高效且用户友好的开放式 API。您可以在抽象类上添加一些注解，*voila*，即可访问 JavaScript API。一切都像魔术一样运行良好，直到不行为止。使用 `package:js` 有很多限制：无法直接访问浏览器 API、重命名成员、转换、附加 Dart 逻辑及其他 [更多](https://github.com/dart-lang/sdk/issues/35084)。为了弥补这些不足，我们还发布了 `[dart:js_util](https://codereview.chromium.org/2150313003/)` —— 一种轻量且高效的低级 API，类似于 `dart:js`，作为备用方案。`package:js` 中的所有限制确实让我们很困扰，但我们束手无策。我们需要 Dart 语言的更多支持才能做得更好。

大约在那个时候，我们已经在进行我们有史以来最大的语言改进 — 我们正在使 Dart 变得 [sound](https://dart.dev/language/type-system#what-is-soundness)。具有讽刺意味的是，当我们在2018年发布了新的类型系统 Dart 2.0 时，互操作性变得 *更差*！超出了这些早期的限制，使 `package:js` 特殊的魔力也有了一个阴暗面 — 它无法检查类型的有效性。这意味着我们的互操作性在我们本来就是完全类型安全的语言中成为了不安全的源头。

然后，我们的旅程转向专注于共同改进 Dart 和 JS-interop。遵循明确的原则（成为惯用的、表达力强的、组合的、精确的、平易近人的、务实的、非魔法的和完整的），我们朝着一种依赖于类型和静态调度的设计前进，挑战 Dart 语言。接下来是并行的演进。

+   在2019年，Dart 2.7 添加了静态扩展方法。您可以将自定义的 Dart 逻辑附加到 JS-interop 类上，并转换值，例如将 JS 的 `Promise` 转换为 Dart 的 `Future`，而无需使用包装器。

+   在2021年，我们发布了 `@staticInterop` 与 `package:js` v0.6.4。最后，JS-interop 变得足够表达力强 —— 您可以公开之前由 SDK 库如 `dart:html` 独家管理的浏览器 API。

+   在2023年，当我们在 Dart 3.0 中放弃了不安全的空安全性时，我们终于看到了我们所取得的进展，我们的设计和 `@staticInterop` 的工作清楚地表明，我们已经准备好解决我们长期以来存在的声音差距。

那一年，我们引入了对 WasmGC 的编译，并利用 JS-interop 在其上运行像 [Flutter web](/flutter/whats-next-for-flutter-b94ce089f49c) 这样的丰富框架。这促使了 [JS Types](https://dart.dev/interop/js-interop/js-types) 的工作，明确定义了 Dart 和 JS 在编程模型中的边界，并找到了在 Wasm 和 JS 编译目标中与 JS 一致工作的方法。我们还开始了 [extension types](https://dart.dev/language/extension-types) 的语言实验 —— 这是 Dart 3.3 中推出的一个功能，它弥合了 Dart 语言与 JS-interop 之间的差距。多年来，JS-interop 有一些行为，如类型擦除，与 Dart 中的其他内容不匹配。有了扩展类型，JS-interop 最终可以成为惯用的，得到在 Dart 开发工具中应有的支持。

尽管历经多次变化，但整个十年中有一件事始终如一：我们 Dart 社区的积极参与。社区成员早期就开始测试和贡献 `dart:js`，后来影响了 `package:js` 的设计。他们编写了工具来解决功能差距（`[package:js_wrapping](https://github.com/a14n/dart-js-wrapping)`），并尝试通过自动生成 Dart API（`[package:js_facade_gen](https://github.com/dart-archive/js_facade_gen)`、`[package:js_bindings](https://pub.dev/packages/js_bindings)`、`[package:typings](https://pub.dev/packages/typings)`）来提高生产力。每一份贡献都帮助改进了 Dart 的互操作设计。感谢你们每一个人，让这段旅程如此激动人心！

最后，现在是 2024 年。我们在 Dart 3.3 中发布了 `[dart:js_interop](https://dart.dev/interop/js-interop)`，连同 `[package:web](https://dart.dev/interop/js-interop/package-web)`，这是 Dart 中用于 JS 互操作的最新解决方案，使得将 [Flutter 编译到 Wasm](https://docs.flutter.dev/platform-integration/web/wasm) 成为可能。

```
// Access via `dart:js_interop` (2024)
import 'dart:js_interop';

// Declarations use extension types, which are very similar to package:js
// declarations. The main difference: they are statically dispatched.
extension type MyObject._(JSObject _) implements JSObject {
  external int get field1;
  external void set field1(int value);
  external String method2();
}

@JS()
external MyObject get myTopLevel;

void main() {
  var object = myTopLevel;
  object.field1 = 1;
  // At last, access is sound - this line fails with a type error
  // when returning from method2.
  object.method2().substring(1);
}
```

+   `dart:js_interop` 是一种基于扩展类型的静态、健全、表达力强且一致的互操作形式，能够暴露任何 JavaScript 或浏览器 API。

+   `package:web` 使用 `dart:js_interop` 来完成 13 年前 `dart:html` 的工作，但以一种既支持 JavaScript 又支持 WasmGC 的方式进行。

今天，我们很高兴庆祝 Dart/JS 互操作的新形式及其所带来的未来。了解我们的过去，我们可以肯定，这不是旅程的终点，而是我们历史上一个激动人心的时刻。

我们迫不及待想看到你们将用它来构建什么！
