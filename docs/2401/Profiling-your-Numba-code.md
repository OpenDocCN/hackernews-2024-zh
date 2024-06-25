<!--yml

类别：未分类

日期：2024-05-27 15:20:44

-->

# 对你的 Numba 代码进行性能分析

> 来源：[https://pythonspeed.com/articles/numba-profiling/](https://pythonspeed.com/articles/numba-profiling/)

如果你正在编写数值 Python 代码，[Numba](/articles/numba-faster-python/) 可能是加速程序的好方法。通过将 Python 的子集编译为机器代码，Numba 允许你编写在普通 Python 中速度太慢的循环和其他结构。换句话说，它类似于 Cython、C 或 Rust，因为它允许你为 Python 编写编译扩展。

然而，Numba 代码并不总是像它可能的那样快。这就是性能分析有用的地方：它可以找出你代码中至少一些的瓶颈。

在本文中，我们将讨论：

+   Profila，我发布的一个新性能分析器，专门设计用于 Numba 代码。

+   性能分析的局限性。性能分析器无法发现和帮助你发现的潜在的许多性能增强。 

## 介绍 Profila：一个专门为 Numba 设计的性能分析器

[Profila](https://github.com/pythonspeed/profila) 是一个专门设计的性能分析器，用于识别你的 Numba 代码中哪些行代码运行较慢。它通过采样工作：每隔大约 10ms 运行一个 `gdb` 进程，连接到你的进程，获取回溯信息，然后使用它来确定哪些行代码正在运行。获取足够的样本，你将了解到程序中花费时间最多的地方。

让我们看看 Profila 的实际效果！我们将对以下程序进行性能分析，该程序将图像抖动，将其从灰度转换为黑白。（这是最初的朴素版本，我随后在[另一篇文章中对其进行了性能优化](/articles/optimizing-dithering/)。）

```
from numba import njit
import numpy as np
from skimage import io
from time import time

IMAGE = io.imread("hallway.jpg")

# Code that is decorated with numba.njit looks like Python code,
# but is actually compiled into machine code, at runtime. This
# is fast, low-level code! @njit
def dither(img):
    # Allow negative values and wider range than a uint8 has:
    result = img.astype(np.int16)
    y_size = img.shape[0]
    x_size = img.shape[1]
    last_y = y_size - 1
    last_x = x_size - 1
    for y in range(y_size):
        for x in range(x_size):
            old_value = result[y, x]
            if old_value < 0:
                new_value = 0
            elif old_value > 255:
                new_value = 255
            else:
                new_value = np.uint8(np.round(old_value / 255.0)) * 255
            result[y, x] = new_value
            # We might get a negative value for the error:
            error = np.int16(old_value) - new_value
            if x < last_x:
                result[y, x + 1] += error * 7 // 16
            if y < last_y and x > 0:
                result[y + 1, x - 1] += error * 3 // 16
            if y < last_y:
                result[y + 1, x] += error * 5 // 16
            if y < last_y and x < last_x:
                result[y + 1, x + 1] += error // 16

    return result.astype(np.uint8)

# Run a first time, to compile the code: dither(IMAGE)

# Make sure we run this for long enough that the profiler
# gets sufficient samples: start = time()
runs = 0
while time() - start < 5:
    runs += 1
    dither(IMAGE)

elapsed = time() - start
print(f"Processed {int(round(runs / elapsed))} images / sec") 
```

然后，我们安装 `gdb` 和 `profila`：

```
$ sudo apt-get install -y gdb
$ pip install profila 
```

现在我们可以使用 Profila 来对代码进行性能分析：

```
$ python -m profila annotate -- dither.py
Processed 255 images / sec
# Total samples: 805 (37.8% non-Numba samples, 1.9% bad samples)

## File `dither.py`
Lines 14 to 40:

  0.1% |     result = img.astype(np.int16)
       |     y_size = img.shape[0]
       |     x_size = img.shape[1]
       |     last_y = y_size - 1
       |     last_x = x_size - 1
       |     for y in range(y_size):
  0.2% |         for x in range(x_size):
       |             old_value = result[y, x]
       |             if old_value < 0:
       |                 new_value = 0
  1.0% |             elif old_value > 255:
       |                 new_value = 255
       |             else:
 30.2% |                 new_value = np.uint8(np.round(old_value / 255.0)) * 255
       |             result[y, x] = new_value
       |             # We might get a negative value for the error:
  2.0% |             error = np.int16(old_value) - new_value
  0.6% |             if x < last_x:
 11.3% |                 result[y, x + 1] += error * 7 // 16
       |             if y < last_y and x > 0:
  0.9% |                 result[y + 1, x - 1] += error * 3 // 16
       |             if y < last_y:
  2.4% |                 result[y + 1, x] += error * 5 // 16
       |             if y < last_y and x < last_x:
  1.6% |                 result[y + 1, x + 1] += error // 16
       |
  0.2% |     return result.astype(np.uint8)

## File `numba/np/arraymath.py`
Lines 3157 to 3157:

  9.8% |                         return _np_round_float(a) 
```

### 优化代码

首先，从输出的顶部注意到，37.8 + 1.9 = 大约 40% 的样本要么是不好的，要么不是运行 Numba 代码。这包括花费在导入模块、运行常规 Python 代码、编译 Numba 代码等的时间。这意味着 60% 的时间花费在运行实际的 Numba 代码上。

然后，我们可以看到 `np.round()` 和 `new_value = np.uint8(np.round(old_value / 255.0)) * 255` 的组合占总样本的 30.2 + 9.8 = 40%。所以这意味着 40/60，三分之二的时间，都花费在了这一行代码上，将 `old_value` 转换为 `new_value`。

正如我们在[原文](/articles/optimizing-dithering/)中看到的这个示例，我们可以将这段代码替换为：

```
if old_value < 0:
    new_value = 0
elif old_value > 255:
    new_value = 255
else:
    new_value = np.uint8(np.round(old_value / 255.0)) * 255 
```

用一个更简单的版本：

```
new_value = 0 if old_value < 128 else 255 
```

如果我们对这个优化版本进行分析，我们将看到：

```
$ python -m profila annotate -- dither2.py
Processed 758 images / sec
# Total samples: 713 (30.3% non-Numba samples, 2.2% bad samples)

## File `/home/itamarst/devel/sandbox/numba-profiling-article/dither2.py`
Lines 14 to 35:

  0.1% |     result = img.astype(np.int16)
       |     y_size = img.shape[0]
       |     x_size = img.shape[1]
       |     last_y = y_size - 1
       |     last_x = x_size - 1
       |     for y in range(y_size):
  0.3% |         for x in range(x_size):
  0.1% |             old_value = result[y, x]
  4.5% |             new_value = 0 if old_value < 128 else 255
  2.0% |             result[y, x] = new_value
       |             # We might get a negative value for the error:
  5.5% |             error = np.int16(old_value) - new_value
  5.3% |             if x < last_x:
 34.9% |                 result[y, x + 1] += error * 7 // 16
       |             if y < last_y and x > 0:
  3.6% |                 result[y + 1, x - 1] += error * 3 // 16
       |             if y < last_y:
  6.0% |                 result[y + 1, x] += error * 5 // 16
       |             if y < last_y and x < last_x:
  4.5% |                 result[y + 1, x + 1] += error // 16
       |
  0.6% |     return result.astype(np.uint8) 
```

就像在[原文](/articles/optimizing-dithering/)中一样，我们的代码现在运行速度提高了 3 倍！

这次我们不必猜测，我们有一个性能分析器指向主要的瓶颈。

## 性能分析的局限性

虽然分析非常有帮助，但它并不总是能告诉您加速代码所需的所有信息。要了解更多关于这些问题的信息，请查看我的[即将发布的关于编写高性能低级代码的书籍](/products/lowlevelcode/)。

### 限制 #1：编译后的代码并不总是匹配源代码。

考虑以下示例：

```
@njit
def add(arr):
    result = np.empty_like(arr)
    for i in range(len(arr)):
        orig = arr[i]
        orig += 123_000_000
        orig += 456_000
        orig += 789
        result[i] = orig
    return result 
```

如果我们对其进行分析，我们会得到：

```
## File `combo.py`
Lines 13 to 14:

  2.9% |         orig += 789
  7.5% |         result[i] = orig 
```

为什么 `orig += 789` 被列出来，但添加 `123_000_000` 和 `456_000` 的其他行却没有？因为由于编译器优化，它们在编译后的版本中不再作为不同的操作存在。例如，我们可以检查函数生成的汇编代码，并搜索 789：

```
>>> for line in next(iter(add.inspect_asm().values())).splitlines():
...     if "789" in line: print(line)
...
        .long   123456789
        movl    $123456789, %edi
        movl    $123456789, %edx 
```

编译器将三个添加的数字（`123_000_000`、`456_000`、`789`）合并为一个单一数字（`123_456_789`），这个数字很可能是与单一的加法操作一起使用的，而不是三个。

### 限制 #2：你需要一个关于 CPU 执行方式的心智模型。

记住，在我们优化的抖动函数中，我们看到了以下分析结果：

```
 5.3% |             if x < last_x:
 34.9% |                 result[y, x + 1] += error * 7 // 16
       |             if y < last_y and x > 0:
  3.6% |                 result[y + 1, x - 1] += error * 3 // 16
       |             if y < last_y:
  6.0% |                 result[y + 1, x] += error * 5 // 16
       |             if y < last_y and x < last_x:
  4.5% |                 result[y + 1, x + 1] += error // 16 
```

为什么第一次计算比后面的要昂贵得多？它们似乎做的事情差不多。

一个假设是指令级并行性：CPU 能够预先并行地执行后面的代码。所以第一行昂贵不是因为它与众不同，而是因为它是*第一个*。我们可以部分测试这个假设，通过改变语句的顺序并再次进行分析。这是结果：

```
 |             if y < last_y and x > 0:
 20.3% |                 result[y + 1, x - 1] += error * 3 // 16
 14.2% |             if x < last_x:
  8.8% |                 result[y, x + 1] += error * 7 // 16
       |             if y < last_y:
  3.7% |                 result[y + 1, x] += error * 5 // 16
       |             if y < last_y and x < last_x:
  3.7% |                 result[y + 1, x + 1] += error // 16 
```

现在，`result[y, x + 1] += error * 7 // 16` 这行代码在之前版本的代码中的样本比例要低得多，即使它运行的次数相同，并且执行的计算也完全相同。唯一改变的是执行顺序。

一个仅仅执行代码并且没有并行性的 CPU 的心智模型是不足以理解这些结果的。

### 限制 #3：更快的代码可能需要更全面的视角。

查看代码哪些行速度较慢可能非常有帮助，但有些优化是对代码进行更广泛重新思考的结果。例如，[原文](/articles/optimizing-dithering/)中的最终优化版本将内存使用减少了三分之二，同时比第二版本运行速度快了50%。这是通过改变算法的结构来实现的，而不仅仅是专注于一两行代码。

## 试试 Profila 吧！

Profila 是一个新项目：我在写这篇文章的当天发布了它。所以试试它，并请在 [GitHub issue 跟踪器](https://github.com/pythonspeed/profila/issues/new) 或通过电子邮件告诉我您发现的任何错误，您的成功故事，或任何反馈。
