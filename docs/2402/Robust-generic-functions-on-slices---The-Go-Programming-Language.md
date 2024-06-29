<!--yml

category: 未分类

date: 2024-05-29 13:21:15

-->

# 切片上的健壮通用函数 - Go编程语言

> 来源：[https://go.dev/blog/generic-slice-functions](https://go.dev/blog/generic-slice-functions)

# 切片上的健壮通用函数

瓦伦丁·德莱普拉斯

2024年2月22日

[切片](/pkg/slices)包提供了适用于任何类型切片的函数。在本博文中，我们将讨论如何通过理解切片在内存中的表示以及对垃圾回收器的影响，更有效地使用这些函数，并且我们将介绍我们最近如何调整这些函数以使其更少令人惊讶。

借助[类型参数](/blog/deconstructing-type-parameters)，我们可以为所有可比较元素类型的切片编写像[slices.Index](/pkg/slices#Index)这样的函数：

```
// Index returns the index of the first occurrence of v in s,
// or -1 if not present.
func Index[S ~[]E, E comparable](s S, v E) int {
    for i := range s {
        if v == s[i] {
            return i
        }
    }
    return -1
} 
```

不再需要为每种不同类型的元素重新实现`Index`。

[切片](/pkg/slices)包包含许多这样的辅助函数，用于执行切片的常见操作：

```
 s := []string{"Bat", "Fox", "Owl", "Fox"}
    s2 := slices.Clone(s)
    slices.Sort(s2)
    fmt.Println(s2) // [Bat Fox Fox Owl]
    s2 = slices.Compact(s2)
    fmt.Println(s2)                  // [Bat Fox Owl]
    fmt.Println(slices.Equal(s, s2)) // false 
```

几个新函数（`Insert`、`Replace`、`Delete`等）修改了切片。为了理解它们的工作原理以及如何正确使用它们，我们需要检查切片的基本结构。

切片是数组的一部分视图。[内部](/blog/slices-intro)，一个切片包含一个指针、长度和容量。两个切片可以有相同的底层数组，并且可以查看重叠的部分。

例如，这个切片`s`是一个大小为6的数组的4个元素的视图：

如果一个函数改变了作为参数传递的切片的长度，则需要向调用者返回一个新的切片。如果不需要增长底层数组，它可能保持不变。这解释了为什么[append](/blog/slices)和`slices.Compact`返回一个值，但仅仅重新排序元素的`slices.Sort`则不返回值。

考虑删除切片部分的任务。在泛型之前，从切片`s`中删除部分`s[2:5]`的标准方法是调用[append](/ref/spec#Appending_and_copying_slices)函数将末端部分复制到中间部分：

```
s = append(s[:2], s[5:]...) 
```

语法复杂且容易出错，涉及子切片和可变参数。我们添加了[slice.Delete](/pkg/slices#Delete)以使删除元素变得更容易：

```
func Delete[S ~[]E, E any](s S, i, j int) S {
       return append(s[:i], s[j:]...)
} 
```

这个一行的函数`Delete`更清晰地表达了程序员的意图。让我们考虑一个长度为6且容量为8的切片`s`，包含指针：

这个调用从切片`s`中删除了索引为`2`、`3`、`4`的元素：

```
s = slices.Delete(s, 2, 5) 
```

索引为2、3、4的间隙由将元素`s[5]`向左移动来填充，并将新长度设置为`3`。

`Delete`不需要为新数组分配空间，因为它在原地移动元素。与`append`类似，它返回一个新切片。`slices`包中的许多其他函数也遵循这种模式，包括`Compact`、`CompactFunc`、`DeleteFunc`、`Grow`、`Insert`和`Replace`。

在调用这些函数时，我们必须考虑原始切片已无效，因为底层数组已经被修改。调用函数但忽略返回值将是一个错误：

```
 slices.Delete(s, 2, 5) // incorrect!
    // s still has the same length, but modified contents 
```

## 一个不想要的活跃性问题

在 Go 1.22 之前，`slices.Delete` 没有修改切片新旧长度之间的元素。虽然返回的切片不会包含这些元素，但是在原始、现在无效的切片末尾创建的“间隙”仍然保留着它们。这些元素可能包含指向大对象（比如一个 20MB 的图像）的指针，垃圾收集器不会释放与这些对象相关联的内存。这导致了可能引起显著性能问题的内存泄漏。

在上面的例子中，我们通过将一个元素向左移动成功地从`s[2:5]`中删除了指针`p2`、`p3`、`p4`。但是`p3`和`p4`仍然存在于底层数组中，超出了`s`的新长度。垃圾收集器不会对它们进行回收。更不明显的是，`p5`不是被删除的元素之一，但是由于数组的灰色部分中保留了`p5`指针，它的内存可能仍然泄漏。

如果开发人员没有意识到“看不见”的元素仍在使用内存，这可能会让他们感到困惑。

所以我们有两个选择：

+   要么保留`Delete`的高效实现。如果用户想确保指向的值可以被释放，让他们自己将过时的指针设置为`nil`。

+   或者将`Delete`改为始终将过时元素设置为零。这是额外的工作，使`Delete`稍微不那么高效。将指针清零（将它们设置为`nil`）使得当它们变得无法访问时，对象可以被垃圾回收。

哪一个选择更好并不明显。第一个默认提供了性能，第二个默认提供了内存节约。

## 解决方法

一个关键观察是，“将过时指针设置为`nil`”并不像看起来那么简单。事实上，这个任务非常容易出错，我们不应该让用户自己来写。出于实用主义的考虑，我们选择修改这五个函数的实现：`Compact`、`CompactFunc`、`Delete`、`DeleteFunc`、`Replace`，以“清除尾部”的方式。作为一个好的副作用，认知负担减少了，现在用户不需要再担心这些内存泄漏问题。

在 Go 1.22 中，在调用`Delete`之后，内存看起来是这样的：

这五个函数的代码更改使用了新的内置函数[clear](/pkg/builtin#clear)（Go 1.21）来将过时元素设置为`s`元素类型的零值：

当`E`为指针、切片、映射、通道或接口类型时，`E`的零值是`nil`。

## 测试失败

这个变化导致一些在 Go 1.21 中通过的测试在错误使用切片函数时在 Go 1.22 中失败。这是个好消息。当你有 bug 时，测试应该告诉你。

如果你忽略`Delete`的返回值：

```
slices.Delete(s, 2, 3)  // !! INCORRECT !! 
```

如果你假设 `s` 不包含任何空指针，那么你可能会做出错误的假设。[在 Go Playground 中的示例](/play/p/NDHuO8vINHv)。

如果你忽略了 `Compact` 的返回值：  

```
slices.Sort(s) // correct
slices.Compact(s) // !! INCORRECT !! 
```

如果你错误地假设 `s` 已经正确排序并且已压缩。[示例](/play/p/eFQIekiwlnu)。  

如果你将 `Delete` 的返回值分配给另一个变量，并且继续使用原始切片：  

```
u := slices.Delete(s, 2, 3)  // !! INCORRECT, if you keep using s !! 
```

如果你假设 `s` 不包含任何空指针，那么你可能会做出错误的假设。[示例](/play/p/rDxWmJpLOVO)。  

如果你意外地遮蔽了切片变量，并且继续使用原始切片：  

```
s := slices.Delete(s, 2, 3)  // !! INCORRECT, using := instead of = !! 
```

如果你假设 `s` 不包含任何空指针，那么你可能会做出错误的假设。[示例](/play/p/KSpVpkX8sOi)。

## 结论  

`slices` 包的 API 相比传统的无泛型语法删除或插入元素有显著改进。  

我们鼓励开发者使用新的函数，同时避免上述的“坑”。  

由于最近实现中的变更，一类内存泄漏已经自动避免，而无需更改 API，并且开发者也无需额外工作。  

## 进一步阅读

`slices` 包中函数的签名受到内存中切片表示特定细节的影响。我们建议阅读  

关于清零过时元素的 [原始提案](/issue/63393) 包含许多细节和评论。  
