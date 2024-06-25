<!--yml

分类：未分类

日期：2024-05-27 14:49:21

-->

# 什么也不传递出奇地困难

> 来源：[https://davidben.net/2024/01/15/empty-slices.html](https://davidben.net/2024/01/15/empty-slices.html)

# 什么也不传递出奇地困难

[David Benjamin](/)（2024-01-15）

我的日常工作是在[浏览器](https://www.chromium.org/)和[加密](https://boringssl.googlesource.com/boringssl/)方面，而不是编译器，然而我经常发现我需要花更多的时间来解决编程语言的语义问题而不是使用它们。本文讨论了 C、C++ 和 Rust 之间的一个棘手的跨语言问题。简而言之：

+   C 语言关于指针和 `memcpy` 的规则没有好的方法来表示空的内存切片。

+   C++ 的指针规则是可以的，但是 C++ 中的 `memcpy` 继承了 C 的行为。

+   Rust FFI 并非[零成本](https://blog.rust-lang.org/2015/04/24/Rust-Once-Run-Everywhere.html)。Rust 选择了一种与 C/C++ 不兼容的切片表示，每个方向都需要进行转换。忘记转换是一个容易犯的错误，而且是不安全的。

+   Rust 切片似乎*也*与 Rust 指针算术不兼容，以至于标准库的切片迭代器是不安全的。（更新 2024-01-16：听起来这正在[修复过程中](https://github.com/rust-lang/rust/issues/117945)！）

由于 FFI 的挑战本质上涉及多种语言，我大部分是为了有一个共同的参考来描述不匹配。

## 切片

这三种语言都允许使用 *切片*，或者内存中连续的对象序列。（在这里也称为[跨度](https://en.cppreference.com/w/cpp/container/span)，但我们这里将使用“切片”）切片通常是一个指针和一个长度，`(start, count)`，从 `start` 开始，有 `count` 个对象，类型为 `T`。

切片也可以用两个指针 `(start, end)` 来指定，从 `start`（包含）到 `end`（不包含）的对象。这对于迭代来说更好，因为只需要调整一个值来推进，但长度不太容易得到。C++ 的迭代器对就是这种形式的一般化，而 Rust 的切片迭代器在内部使用这种形式。这两种形式可以用 `end = start + count` 和 `count = end - start` 进行转换，使用 C 风格的指针算术，其中一切都按照对象的大小来缩放。我们主要讨论 `(start, count)`，但这种双重性意味着切片与指针算术密切相关。

在 C 和 C++ 中，切片最多是由指针和长度构建的库类型。有时函数只是单独获取或返回指针和长度，但仍然使用它来表示内存切片。在 Rust 中，切片是语言原语，但底层组件对于不安全代码和 FFI 是暴露的。要处理每个这些，我们必须理解哪些指针和长度的组合是有效的。

对于正长度的切片来说这很简单：`start` 必须指向一个至少包含 `count` 个 `T` 类型对象的分配内存。但是假设我们想要一个空（长度为零）切片。**空切片的有效表示是什么？**

当然，我们可以将 `start` 指向（或刚好超出）某些 `T` 类型数组，并将 `count` 设置为零。但我们可能希望创建一个没有现有数组的空切片。例如，在 C++ 中默认构造的 `std::span<T>()` 或者在 Rust 中的 `&[]`。那么我们有哪些选项呢？特别地：

1.  空切片可以是 `(nullptr, 0)` 吗？

1.  空切片可以是 `(alignof(T), 0)`，或者是一些不对应于分配的对齐地址吗？

第二个问题对 C 和 C++ 的人来说可能看起来有点奇怪，但 Rust 的人可能会将其识别为 `[std::ptr::NonNull::dangling](https://doc.rust-lang.org/std/ptr/struct.NonNull.html#method.dangling)`。

## C++

C++ 是最容易讨论的，因为它有一个正式的规范并且（几乎）是自洽的。

首先，`(nullptr, 0)` 是 C++ 中的一个有效的空切片。STL 的类型 [经常返回它](https://eel.is/c++draft/span.cons#2)，而且语言与之兼容：

+   `(T*)nullptr + 0` 被 [定义](https://eel.is/c++draft/expr.add#4.1) 为 `(T*)nullptr`

+   `(T*)nullptr - (T*)nullptr` 被 [定义](https://eel.is/c++draft/expr.add#5.1) 为 `0`

C++ 用 `(start, count)` 形式的 [指针加法](https://eel.is/c++draft/span.cons#4.1) 定义了像 `std::span` 这样的 API，用迭代器对在 `(start, end)` 形式的 [指针减法](https://eel.is/c++draft/iterator.operations#5) 定义了迭代器对，因此这既是必要的也是充分的。

此外，C++ 禁止 `(nullptr, 0)` 是不现实的。C++ 代码经常需要与指定切片为单独组件的 API 进行交互。给定这样的 API，*没有人*会编写这样的代码：

```
void takes_a_slice(const uint8_t *in, size_t len);

uint8_t placeholder;
takes_a_slice(&placeholder, 0);

```

使用 `nullptr` 更加自然：

```
void takes_a_slice(const uint8_t *in, size_t len);

takes_a_slice(nullptr, 0);

```

这意味着，为了实用，诸如 `takes_a_slice` 这样的函数必须接受 `(nullptr, 0)`。为了实现这样的函数是实用的，底层语言原语也必须接受 `(nullptr, 0)`。

至于 `(alignof(T), 0)` 的问题，指针 [加法](https://eel.is/c++draft/expr.add#4.2) 和 [减法](https://eel.is/c++draft/expr.add#5.2) 要求指针指向某个对象并且操作保持在该对象的边界内（或者超出末尾的一个位置）。C++ 并没有定义在 `alignof(T)` 处有一个对象，因此这是不允许的，而是产生未定义的行为。这并没有直接的可用性问题（没有人会写 `reinterpret_cast<uint8_t*>(1)` 来调用 `takes_a_slice`），但我们将会在后面看到它对 Rust FFI 有一些后果。

（2024-01-16 更新：添加了以下段落，因为这种简写似乎不太清楚。）

当然，**原则上**，`takes_a_slice` 可以定义其自己独特的规则来处理这些边缘情况。除了类型系统自然提供的内容之外，用户代码很少会正式定义语义，因此我们必须转而寻求更模糊的惯例和期望。不幸的是，在 C 和 C++ 中，这些更模糊的方面通常包括明确定义的特性。当所有与切片相邻的语言原语一致地使用相同的切片组件前提条件时，一个简单写的函数将匹配。这时，对于 `takes_a_slice` 的前提条件来说，这是一个合理的默认解释。当本文讨论 C++ 或 C 的切片规则时，部分意思是这种出现的惯例的简写。

然而，C++ 只是*几乎*自洽的。C++ 从 C 的标准库中获取 `memcpy` 和其他函数，包括 C 的语义...

## C

C 更混乱。出于上述相同的原因，拒绝 `(NULL, 0)` 是不切实际的。然而，C 的语言规则使函数难以接受它。C 没有 C++ 对于 `(T*)NULL + 0` 和 `(T*)NULL - (T*)NULL` 的特殊情况。参见[N2310](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n2310.pdf)第 6.5.6 节的第 8 和 9 条款。`memcpy` 和 C 标准库的其余部分同样[禁止](https://www.imperialviolet.org/2016/06/26/nonnull.html)`(NULL, 0)`。

我认为**这应该被视为 C 规范中的一个 bug**，编译器不应该基于此进行[优化](https://gcc.gnu.org/gcc-4.9/porting_to.html)。在[BoringSSL](https://boringssl.googlesource.com/boringssl/)中，我们发现 C 规则如此难以使用，以至于我们不得不[包装标准库](https://boringssl.googlesource.com/boringssl/+/17cf2cb1d226b0ba2401304242df7ddd3b6f1ff2%5E%21/)并使用 `n != 0` 检查。指针算术规则同样是[开发](https://boringssl.googlesource.com/boringssl/+/6be491b7bb57c3950d4fbb97fdd4a141e3fa4d63%5E%21/)的[负担](https://boringssl.googlesource.com/boringssl/+/4984e4a6325e9c6302f846c7bf2b75e8ea3fd9dd%5E%21/)。此外，C++ 继承了 C 的标准库（但不是它的指针规则），包括这种行为。在 Chromium 的 C++ 代码中，`memcpy` 是阻止[启用 UBSan](https://bugs.chromium.org/p/chromium/issues/detail?id=1394755)的最大障碍。

幸运的是，还有希望。[Nikita Popov](https://www.npopov.com/2024/01/01/This-year-in-LLVM-2023.html#zero-length-operations-on-null) 和 Aaron Ballman 提出了一份[提案](https://docs.google.com/document/d/1guH_HgibKrX7t9JfKGfWX2UCPyZOTLsnRfR6UleD1F8/edit)来修复这个问题。 （谢谢！）虽然这不会使 C 和 C++ 安全到任何程度，但这是一个简单的步骤来修复一个非强制性的错误。

请注意，除了带有删除的空检查的人为示例外，当前的规则实际上并没有帮助编译器有意义地优化代码。`memcpy` 的实现不能依赖指针的有效性进行推测性读取，因为即使 `memcpy(NULL, NULL, 0)` 是未定义的，缓冲区末尾的切片也是可以的。

```
char buf[16];
memcpy(dst, buf + 16, 0);

```

如果 `buf` 在页面的末尾，之后没有分配任何内容，从 `memcpy` 进行的推测性读取会中断。

## Rust

Rust 不允许 `(nullptr, 0)`。诸如 `std::slice_from_raw_parts` 这样的函数[要求指针是非空的](https://doc.rust-lang.org/std/slice/fn.from_raw_parts.html)。这来自于 Rust 将类型像 `&[T]` 和 `*[T]` 视为类似于 `&T` 和 `*T`。它们是“引用”和“指针”，表示为 `(start, count)` 对。Rust 要求每种指针类型在其引用类型之外都有一个“空”值。这在 `enum` 布局优化中使用。例如，`Option::<&[T]>` 与 `&[T]` 具有相同的大小，因为 `None` 使用此空值。

不幸的是，Rust 选择了 `(nullptr, 0)` 作为空切片指针，这意味着空切片 `&[]` 不能使用它。这使得 Rust 必须发明一种不寻常的约定：一些非空的、对齐的、但是否则悬空的指针，通常是 `(alignof(T), 0)`。

这个切片是否定义了指针算术？从我所了解的情况来看，答案似乎是否定的！（更新 2024-01-16：听起来这个问题正在[被定义中](https://github.com/rust-lang/rust/issues/117945)！）

Rust 中的指针算术用方法 `[add](https://doc.rust-lang.org/std/primitive.pointer.html#method.add)`、`[sub_ptr](https://doc.rust-lang.org/std/primitive.pointer.html#method.sub_ptr)` 和 `[offset_from](https://doc.rust-lang.org/std/primitive.pointer.html#method.offset_from)` 表示，标准库以 [allocated objects](https://doc.rust-lang.org/std/ptr/index.html#allocated-object) 的术语定义了它们。这意味着，为了使指针算术与 `alignof(T)` 协同工作，必须在每个非零地址分配零大小的切片。此外，`offset_from` 要求从同一切片派生的两个悬空指针指向这些对象的“相同”部分。虽然[这里](https://doc.rust-lang.org/std/ptr/index.html#safety)的第三条，第二句，说将字面量转换为指针是“有效的零大小访问”，但它没有提到分配的对象或指针算术。

最终，这些语义来自 LLVM。Rustonomicon 在[这方面有更多内容](https://doc.rust-lang.org/nomicon/vec/vec-alloc.html#:~:text=The%20other%20corner%2Dcase%20we%20need%20to%20worry%20about%20is%20empty%20allocations)（开始于“The other corner-case…”）。它得出的结论是，虽然在`0x01`处有无限多个*零大小*类型，但 Rust 保守地假设别名分析不允许对*正大小*类型的`alignof(T)`进行零偏移。这意味着**Rust 指针算术规则与 Rust 空切片不兼容。**但请记住，切片迭代和指针算术是密切相关的。Rustonomicon 的[示例迭代器](https://doc.rust-lang.org/nomicon/vec/vec-into-iter.html)使用指针算术，但需要在`into_iter`中用`cap == 0`进行保护，并在`size_hint`中进行`usize`转换。

这对程序员来说太容易忘记了。实际上，真正的 Rust 切片迭代器无条件地进行指针算术（[指针加法](https://github.com/rust-lang/rust/blob/76101eecbe9aa80753664bbe637ad06d1925f315/library/core/src/slice/iter.rs#L94)，[指针减法](https://github.com/rust-lang/rust/blob/76101eecbe9aa80753664bbe637ad06d1925f315/library/core/src/slice/iter/macros.rs#L57)，在[一些](https://github.com/rust-lang/rust/blob/76101eecbe9aa80753664bbe637ad06d1925f315/library/core/src/slice/iter/macros.rs#L141) [宏](https://github.com/rust-lang/rust/blob/76101eecbe9aa80753664bbe637ad06d1925f315/library/core/src/slice/iter.rs#L132)的后面）。这表明**Rust 切片迭代器是不安全的。**

## FFIs

除了自洽性的问题之外，这一切意味着 Rust 和 C++ 的切片是不兼容的。并非所有的 Rust `(start, count)` 对都可以传递到 C++ 中，反之亦然。C 的问题使得情况不太明确，但自然的解决方法是使其与 C++ 保持一致。

这意味着 Rust FFI 不是[“零成本”](https://blog.rust-lang.org/2015/04/24/Rust-Once-Run-Everywhere.html)。**在 C/C++ 和 Rust 之间传递切片需要双向转换，以避免未定义行为。**

对我来说比性能更重要的是安全性和人性化问题。程序员不能指望记住语言规范。如果给定一个`&[T]`并尝试调用一个 C API，自然的选择是使用[as_ptr](https://doc.rust-lang.org/std/primitive.slice.html#method.as_ptr)，但那会返回一个不兼容 C/C++ 的输出。我见过的大多数包装 C/C++ API 的 Rust 包都不进行转换，因此是不安全的。

这尤其是一个问题，因为 C 和 C++（更严重的）安全问题会导致[真实用户受到损害](https://security.googleblog.com/2021/09/an-update-on-memory-safety-in-chrome.html)，需要解决。但是存在半个世纪的现有 C 和 C++ 代码。我们无法实际上通过一个新语言来解决这个问题，而没有良好的 FFI。什么是良好的 FFI？至少，我认为**从 Rust 调用 C 或 C++ 函数的安全性不应该明显低于从 C 或 C++ 中调用该函数的安全性。**

## 愿望清单

空列表不应该如此复杂。我们可以通过几种方式改进 C、C++ 和 Rust：

### 使 C 接受 `nullptr`

参见 Nikita Popov 和 Aaron Ballman 的[提案](https://docs.google.com/document/d/1guH_HgibKrX7t9JfKGfWX2UCPyZOTLsnRfR6UleD1F8/edit)。

### 修复 Rust 的切片表示

所有 `alignof(T)` 问题最终都来自 Rust 不寻常的空切片表示。这是 Rust 需要一个不是 `&[T]` 值的 “null” `*[T]` 值的结果。Rust 可以选择任意数量的明确未使用的表示，例如 `(nullptr, 1)`、`(nullptr, -1)` 或 `(-1, -1)`。

虽然这现在将是一个重大变化，需要解决兼容性问题，但我认为值得认真考虑。这将解决这个混乱的根本原因，在不仅是 Rust FFI 中，而且是 Rust 本身的 soundness 风险。这种风险足以让 Rust 的标准库受到影响。

这也是我看到的唯一完全符合 Rust “零成本” FFI [目标](https://blog.rust-lang.org/2015/04/24/Rust-Once-Run-Everywhere.html)的选项。即使我们让 C 和 C++ 接受 Rust 的 `(alignof(T), 0)`（见下文），从 C/C++ 传递给 Rust 的任何切片仍可能是 `(nullptr, 0)`。

### 为无效指针定义指针算术

如果 Rust 保留其切片表示，我们应该为 `NonNull::dangling()` 定义指针算术。期望低级 Rust 代码保护所有指针偏移量是不切实际的。

更新 2024-01-16：令人高兴的是，听起来这已经[正在定义过程中](https://github.com/rust-lang/rust/issues/117945)!

在 `nullptr` 是一个单一值可以特殊处理的情况下，有许多 `alignof(T)` 值。似乎需要根据实际分配的对象来定义它。这远远超出了我的专业知识范围，所以我可能把所有细节和术语都搞错了，但一个可能性是，沿着 [PNVI-ae-udi](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n2364.pdf) 的路线：

+   `cast_ival_to_ptrval` 在转换垃圾值时返回特殊的 `@empty` 追溯（与 PNVI-ae-udi 保持不变）

+   将 `@empty` 追溯的指针加零是有效的，并且返回原始指针

+   如果两个具有相同地址的 `@empty` 追溯的指针，它们可以相减为零。

但有一个微妙之处，`cast_ival_to_ptrval`如果该地址上有对象，则可能不会返回`@empty`。 返回具体的来源意味着采用[指针清除](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n2443.pdf)语义，这在这里是不可取的。 对于`alignof(T)`，如果最大对齐小于一页且底部页面从未被分配，则不应发生这种情况。 但是Rust不仅允许`alignof(T)`而且允许[任何非零整数字面量](https://doc.rust-lang.org/std/ptr/index.html#safety)，*即使该地址上存在某些分配*。（也许我们可以利用“用户消歧义”方面，并说所有整数到指针的转换可能额外消除歧义为`@empty`？那会影响编译器的别名分析吗？）

我认为这种复杂性说明了为什么`nullptr`比悬空指针更好地选择为空切片。 使用`nullptr`进行指针算术很容易定义，并且`nullptr`不能与真实分配别名。

如果Rust（和LLVM）接受无效指针，它将修复Rust内部的完整性问题，但不包括FFI。 如果C和C++标准*也*采用了这一点，那么这将*部分*修复FFI。 然后我们可以直接将Rust切片传递给C和C++，但反之则不行。 直接将C和C++切片传递给Rust只能通过改变Rust以接受`(nullptr, 0)`形式来修复。

~~（除了Rust FFI之外，在C/C++中使用`alignof(T)`作为指针没有理由，所以我不知道C/C++接受这一点的可能性有多大。）~~ 更新2024-01-16：Nelson Elhage提醒我，有时会使用非空标记指针来[分配零字节](https://github.com/torvalds/linux/blob/ffc253263a1375a65fa6c9f62a893e9767fbebfa/include/linux/slab.h#L167-L178)。 虽然C禁止`malloc`这样做（`malloc(0)`必须返回`nullptr`或*唯一*非空指针），但其他分配器可能合理地选择此选项。 这样做可以使错误检查更加统一，而不必实际保留地址空间。 因此，存在一个非Rust的原因可以允许这些指针在C和C++中使用。

### Rust标准库中的FFI辅助工具

如果语言的切片表示不能兼容，那么我们仍然会在Rust FFI中留下安全隐患。 在这种情况下，Rust的标准库应该更多地帮助程序员选择正确的操作：添加类似于`slice::from_raw_parts`、`slice::as_ptr`等的模拟，使用C和C++表示法，根据需要进行内部转换。 对于无法用于FFI的现有函数，文档中应清楚警告。 最后，在crates.io中审计所有现有的调用，因为现有调用的大多数可能是为了FFI。

对于`slice::from_raw_parts`，我们可以进一步修复现有的函数本身。这将是向后兼容的，但会向非 FFI 用途添加不必要的转换。也就是说，如果 crates.io 的审核显示主要是 FFI 用途，那么这种转换可能是必要的。对于非 FFI 用途，一个包含`[std::ptr::NonNull](https://doc.rust-lang.org/std/ptr/struct.NonNull.html)`的类型签名本来更合适。

这将改善事情，但这是一个不完美的解决方案。我们仍然会牺牲零成本 FFI，并且我们仍然依赖程序员阅读警告并意识到自然选项是不正确的。

## 致谢

感谢 Alex Chernyakhovsky、Alex Gaynor、Dana Jansens、Adam Langley 和 Miguel Young de la Sota 在审阅本文早期版本时提供的帮助。这里的任何错误都是我自己的。
