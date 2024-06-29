<!--yml

category: 未分类

日期：2024-05-29 13:26:01

-->

# 关于无法组合终结器和FFI — wingolog

> 来源：[https://wingolog.org/archives/2024/02/26/on-the-impossibility-of-composing-finalizers-and-ffi](https://wingolog.org/archives/2024/02/26/on-the-impossibility-of-composing-finalizers-and-ffi)

前些天试图为[Harfbuzz](https://harfbuzz.org/)做一个Guile绑定时，我想起了为什么我不再这样做了：无法将GC与显式所有权组合起来。

让我举个例子。Harfbuzz有一个“blob”的概念，它们是引用计数的字节序列。它在许多地方使用这些概念，例如加载OpenType字体时。你可以用[`hb_blob_get_data`](https://harfbuzz.github.io/harfbuzz-hb-blob.html#hb-blob-get-data)来查看blob的内容，它会给你一个指针和一个长度。

假设你使用LuaJIT。（想想几年前，我整天都在写LuaJIT；现在我几乎忘记了。）你从某处获取一个blob并希望获取其数据。你为`hb_blob_get_data`定义了一个包装器：

```
local hb = ffi.load("harfbuzz")
ffi.cdef [[
typedef struct hb_blob_t hb_blob_t;

const char *
hb_blob_get_data (hb_blob_t *blob, unsigned int *length);
]]

```

假设当GC收集了一个blob的Lua包装器时，你安排释放LuaJIT对blob的引用：

```
ffi.cdef [[
void hb_blob_destroy (hb_blob_t *blob);
]]

function adopt_blob(ptr)
  return ffi.gc(ptr, hb.hb_blob_destroy)
end

```

好的，假设我们从某处获取了一个blob，并且想将其内容作为字节字符串复制出来。

```
function blob_contents(blob)
   local len_out = ffi.new('unsigned int')
   local contents = hb.hb_blob_get_data(blob, len_out)
   local len = len_out[0];
   return ffi.string(contents, len)
end

```

问题在于，这段代码尽可能地正确，但还不够。在调用`hb_blob_get_data`之后以及任何其他操作之前，GC可能会运行，如果`blob`在程序执行的未来（延续）中不再使用，那么它可能会被收集，导致`hb_blob_destroy`终结器释放对blob的最后引用，从而释放`contents`：我们随后会访问无效内存。

在GC实现者中，普遍认为包含终结器的程序很可能导致段错误。LuaJIT的语义并不规定GC何时发生以及哪些值将保持活动状态，因此GC和编译器不受限制地扩展`blob`的存活期到其词法作用域的全部。在`blob`的最后使用后立即收集它是完全有效的，因此在某个时间点，GC将会演变成这样做。

我选择LuaJIT并非是要挑它的毛病，而是因为它的FFI非常简单明了。据我所知，所有其他带有GC的语言都有同样的问题。只有两种解决方法，但都不尽人意：要么深入且正确地了解编译器和运行时对于给定代码段的行为，然后祈祷这些知识不会过时，要么尝试手动延长可终结对象的生命周期，然后祈祷编译器和GC不会学到新的技巧来使你的把戏失效。

这后一种策略采用了[“记住这个”程序设计，旨在智胜编译器](https://github.com/ivmai/bdwgc/blob/master/include/gc/gc.h#L1469-L1491)。在过去的几十年里，它们大多数时候都很有效，但未来我不会押注它们。

另一种看待这个问题的方式是，一旦你有一个运行中的系统——不过，你怎么知道它是正确的呢？——你要么从不更新编译器和运行时，要么你要成为那个维护你的 GC（垃圾回收器）以及可能是你的编译器的人的快速朋友。

想要深入了解这个主题，像往常一样，汉斯·伯姆（Hans Boehm）总是第一个和最后一个发言；例如，参见2002年的[*析构函数、终结器和同步*](https://citeseerx.ist.psu.edu/document?repid=rep1&type=pdf&doi=ac6669e60ef9ae9adbf3e596377d731584c27049)。这些考虑实际上并不适用于*析构函数*，它们通常用于具有所有权的语言，并且一般是同步运行的。

祝你编程愉快，并注意安全！
