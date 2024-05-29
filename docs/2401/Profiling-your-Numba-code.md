<!--yml
category: 未分类
date: 2024-05-27 15:20:44
-->

# Profiling your Numba code

> 来源：[https://pythonspeed.com/articles/numba-profiling/](https://pythonspeed.com/articles/numba-profiling/)

If you’re writing numeric Python code, [Numba](/articles/numba-faster-python/) can be a great way to speed up your program. By compiling a subset of Python to machine code, Numba lets you write for loops and other constructs that would be too slow in normal Python. In other words, it’s similar to Cython, C, or Rust, in that it lets you write compiled extensions for Python.

Numba code isn’t always as fast as it could be, however. This is where profiling is useful: it can find at least some of the bottlenecks in your code.

In this article we’ll cover:

*   Profila, a new profiler I’ve released that is specifically designed for Numba code.
*   The limits of profiling. There are many potential performance enhancements that a profiler can’t and won’t help you discover.

## Introducing Profila: a profiler for Numba

[Profila](https://github.com/pythonspeed/profila) is a profiler specifically designed to identify which lines of code in your Numba code are slow. It works via sampling: it runs a `gdb` process that connects to your process every 10ms or so, gets a backtrace, and uses that to identify which lines of code were running. Get enough samples, and you’ll get a sense of where the most time was spent in your program.

Let’s see Profila in action! We’re going to profile the following program, which dithers an image, switching it from grayscale to black and white. (This is the initial naive version that I then [optimized for performance in a different article](/articles/optimizing-dithering/).)

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

We then install `gdb` and `profila`:

```
$ sudo apt-get install -y gdb
$ pip install profila 
```

And now we can use Profila to profile the code:

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

### Optimizing the code

First, notice from the top of the output that 37.8 + 1.9 = about 40% of the samples were either bad, or not running Numba code. This includes time spent importing modules, running regular Python code, compiling the Numba code, and so on. That means 60% of the time was spent running actual Numba code.

Then, we can see the combination of `np.round()` and `new_value = np.uint8(np.round(old_value / 255.0)) * 255` is 30.2 + 9.8 = 40% of total samples. So that means 40/60, two thirds of the time, was spent in that one line of code, turning `old_value` into `new_value`.

As we saw in [the original article](/articles/optimizing-dithering/) featuring this example, we can replace this code:

```
if old_value < 0:
    new_value = 0
elif old_value > 255:
    new_value = 255
else:
    new_value = np.uint8(np.round(old_value / 255.0)) * 255 
```

With a much simper version:

```
new_value = 0 if old_value < 128 else 255 
```

If we profile this optimized version we’ll see:

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

As was the case in the [original article](/articles/optimizing-dithering/), our code now runs 3× as fast!

And this time we didn’t have to guess, we had a profiler to point us at the main bottleneck.

## The limits of profiling

While profiling is extremely helpful, it won’t always tell you everything you need to know to speed up your code. To learn more about all of these issues, check out my [upcoming book on writing high-performance low-level code](/products/lowlevelcode/).

### Limit #1: Compiled code doesn’t always match the source code

Consider the following example:

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

If we profile it, we get:

```
## File `combo.py`
Lines 13 to 14:

  2.9% |         orig += 789
  7.5% |         result[i] = orig 
```

Why is `orig += 789` listed, but not the other lines adding `123_000_000` and `456_000`? Because thanks to compiler optimizations, they no longer exist in the compiled version as distinct operations. We can for example inspect the generated assembly for the function and search for 789:

```
>>> for line in next(iter(add.inspect_asm().values())).splitlines():
...     if "789" in line: print(line)
...
        .long   123456789
        movl    $123456789, %edi
        movl    $123456789, %edx 
```

The three added numbers (`123_000_000`, `456_000`, `789`) have been combined by the compiler into a single number (`123_456_789`) which presumably is being used with a single addition operation, instead of three.

### Limit #2: You need a mental model of how CPU execution works

Recall that in our optimized dithering function we saw the following profiling results:

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

Why is the first calculation so much more expensive than the later ones? They seem to be doing more or less the same thing.

One hypothesis is instruction-level parallelism: the CPU is able to go ahead and speculatively execute the later code in parallel. So the first line is expensive not because it’s any different, but because it’s *first*. We can partially test this hypothesis by changing the order of the statements and profiling again. Here’s the result:

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

Now the line `result[y, x + 1] += error * 7 // 16` has a much lower percentage of samples than in the previous version of the code, even though it’s running the exact same number of times and doing the exact same calculation. The only thing that has changed is the order of execution.

A mental model of a CPU that just executes code linearly, with no parallelism, is not sufficient to understand these results.

### Limit #3: Faster code may require a more holistic view

Seeing which lines of code are slow can be very helpful, but some optimizations are a result of rethinking the code more broadly. For example, the final optimized version in the [original article](/articles/optimizing-dithering/) cuts memory usage by two thirds, and also runs 50% even faster than the second version. It does so by changing the structure of the algorithm, rather than just focusing on a single line or two of code.

## Give Profila a try!

Profila is a new project: I released it the same day I wrote this article. So try it out, and please let me know about any bugs you find, your success stories, or any feedback at all, [in the GitHub issue tracker](https://github.com/pythonspeed/profila/issues/new) or by email.