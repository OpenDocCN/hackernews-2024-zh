<!--yml
category: 未分类
date: 2024-05-29 13:20:18
-->

# /tmp | Python Generators Are Underutilized

> 来源：[https://www.slashtmp.io/posts/generators/](https://www.slashtmp.io/posts/generators/)

<main>

#### January 12, 2024

# Python Generators Are Underutilized

One of my favorite features in the python ecosystem is one I don’t often see utilized:

*Generators*

## Introduction

Python generators *generate* values upon reaching yield statements. Unlike loops, generators yield values one at a time. Consider the following code snippet:

```
def square_vals(x : list):
    return [val * val for val in x]

nums_to_square = list(range(500))
print(square_vals(nums_to_square)) 
```

This will yield the following output:

```
[0, 1, 4, 9, 16, 25, 36, ...] 
```

Let’s bring in a utility library called [tracemalloc](https://docs.python.org/3/library/tracemalloc.html#tracemalloc.get_traced_memory). This is a python built-in library with some very simple utilities for tracking memory allocations in python programs. Let’s modify our program to track the amount of memory allocated for this entire program:

```
def square_vals(x : list):
    to_return = [val * val for val in x]
    return to_return

tracemalloc.start()
nums_to_square = list(range(500))
squares = square_vals(nums_to_square)
initial_mem, peak_mem = tracemalloc.get_traced_memory()
snapshot = tracemalloc.take_snapshot()
pprint.pprint(snapshot.statistics('lineno'))
pprint.pprint("Peak usage:"+ str(peak_mem))
tracemalloc.stop() 
```

This will output:

```
[<Statistic traceback=<Traceback (<Frame filename='<filepath>' lineno=<2>>,)> size=19616 count=484>,
 <Statistic traceback=<Traceback (<Frame filename='<filepath>' lineno=<5>>,)> size=11832 count=245>]
<Statistic traceback=<Traceback (<Frame filename='<filepath>' lineno=7>,)> size=64 count=2>]
'Peak usage:31648 
```

Here we’re seeing the amount of memory allocated at each line, as well as the peak memory. So in this code block, we’re performing the following allocations:

*   `py to_return = [val * val for val in x]` allocates 19616 bytes
*   `py squares = square_vals(nums_to_square)` allocates 11832 bytes
*   `py to_return = initial_mem, peak_mem = tracemalloc.get_traced_memory()` allocates 64 bytes

In total, we utilized a maximum of 31,468 bytes in this program.

Now to add a generator.

Consider the following code:

```
def gen_square_vals(x : list):
    for val in x:
        yield val * val

tracemalloc.start()

nums_to_square = list(range(500))
squares = gen_square_vals(nums_to_square)
initial_mem, peak_mem = tracemalloc.get_traced_memory()
snapshot = tracemalloc.take_snapshot()
pprint.pprint(snapshot.statistics('lineno'))
pprint.pprint("Peak usage:"+ str(peak_mem))
tracemalloc.stop() 
```

This will output:

```
[<Statistic traceback=<Traceback (<Frame filename='<filepath>' lineno=<7>>,)> size=11932 count=484>,
 <Statistic traceback=<Traceback (<Frame filename='<filepath>' lineno=<8>>,)> size=208 count=245>]
<Statistic traceback=<Traceback (<Frame filename='<filepath>' lineno=9>,)> size=64 count=2>]
'Peak usage:12040 
```

This is interesting! Note that, compared to our first example, the act of providing the squared numbers `yield val * val` does not actually lead to any memory allocation, whereas `to_return = [val * val for val in x]` does. Instead, in the latter code block, we end up allocating 208 bytes on the line that calls the `gen_square_values()`. This is the beauty of a python generator!

So what’s happening here?

In our first example, when we call `square_vals()`, we create the list containing all the squared values in memory and return that memory block from the method. When we turn this into a generator, we are instead creating a *generator object*. This generator object creates values on-the-fly, that is: we won’t allocate any of the squared values until we’re ready to use them.

If we go back to our codeblock, we can see this in action by adding a new loop:

```
def gen_square_vals(x : list):
    for val in x:
        yield val * val

tracemalloc.start()
nums_to_square = list(range(500))
squares = gen_square_vals(nums_to_square)
gen_to_list = [square for square in squares]
initial_mem, peak_mem = tracemalloc.get_traced_memory()
snapshot = tracemalloc.take_snapshot()
pprint.pprint(snapshot.statistics('lineno'))
pprint.pprint("Peak usage:"+ str(peak_mem))
tracemalloc.stop() 
```

This just block just takes our generator and yields all the values into a list called `gen_to_list`. When we run this, we get additional output:

```
[<Statistic traceback=<Traceback (<Frame filename='<file_path> lineno=5>,)> size=15456 count=483>,
 <Statistic traceback=<Traceback (<Frame filename='<file_path> lineno=2>,)> size=11832 count=245>,
 <Statistic traceback=<Traceback (<Frame filename='<file_path> lineno=7>,)> size=4160 count=1>,
 <Statistic traceback=<Traceback (<Frame filename='<file_path> lineno=6>,)> size=208 count=1>,
 <Statistic traceback=<Traceback (<Frame filename='<file_path> lineno=8>,)> size=64 count=2>]
'Peak usage:31856' 
```

We’ve managed to get our peak memory usage back up! Notice that the line that generates values is, once again, allocating 11832 bytes when we yield all of our values into a list. This reveals the underlying functionality of the generator: it exists as a small, lightweight object that *generates* values one by one.

## Scaling Up

So what’s the utility here? Well that becomes more obvious when we change things around a little. Let’s modify our code to generate 100,000 numbers instead of 500\. Additionally I’m going to change the line:

```
nums_to_square = list(range(100000)) 
```

to:

```
nums_to_square = range(100000) 
```

in our generator example. (The range builtin in Python acts very similarly to a generator but is slightly different)

Running our example using lists give us the following benchmarks:

```
[<Statistic traceback=<Traceback (<Frame filename='<file_path> lineno=6>,)> size=4000384 count=99984>,
 <Statistic traceback=<Traceback (<Frame filename='<file_path> lineno=14>,)> size=3991832 count=99745>,
 <Statistic traceback=<Traceback (<Frame filename='<file_path> lineno=16>,)> size=64 count=2>]
'Peak usage:7992416' 
```

7,992,416 is a lot of bytes for not doing a lot of work. We’re already at 7MB of memory usage. Let’s instead look at a generator implementation:

```
[<Statistic traceback=<Traceback (<Frame filename='<file_path> lineno=8>,)> size=208 count=1>,
 <Statistic traceback=<Traceback (<Frame filename='<file_path> lineno=7>,)> size=80 count=2>,
 <Statistic traceback=<Traceback (<Frame filename='<file_path> lineno=9>,)> size=64 count=2>]
'Peak usage:288' 
```

288 bytes? Okay, but we’re not doing anything with those values, let’s actually do something with the generator:

```
def gen_square_vals(x : list):
    for val in x:
        yield val * val

tracemalloc.start()

nums_to_square = range(100000)
squares = gen_square_vals(nums_to_square)
for square in squares:
    print(square)
initial_mem, peak_mem = tracemalloc.get_traced_memory()
snapshot = tracemalloc.take_snapshot()
pprint.pprint(snapshot.statistics('lineno'))
pprint.pprint("Peak usage:"+ str(peak_mem))
tracemalloc.stop() 
```

This prints:

```
[<Statistic traceback=<Traceback (<Frame filename='<file_path> lineno=8>,)> size=208 count=1>,
 <Statistic traceback=<Traceback (<Frame filename='<file_path> lineno=7>,)> size=80 count=2>,
 <Statistic traceback=<Traceback (<Frame filename='<file_path> lineno=11>,)> size=64 count=2>,
 <Statistic traceback=<Traceback (<Frame filename='<file_path> lineno=3>,)> size=32 count=1>]
'Peak usage:784' 
```

That’s still tiny!

## Why?

This is obviously just an exceptionally simple example of using generators to put data to the console, but generators are exceptionally useful in many applications. Python is often used as a glue language for ETL pipelines and batch jobs, and many engineers implement these by creating massive list objects within their application. Consider a simple JSON schema that we parse into a python object:

```
{
  "user_id": 123456789,
  "username": "john_doe",
  "full_name": "John Doe",
  "email": "john.doe@example.com",
  "age": 30,
  "gender": "male",
  "country": "United States",
  "city": "New York",
  "occupation": "Software Engineer",
  "education": "Bachelor's Degree",
  "interests": ["programming", "hiking", "reading"],
  "languages": ["English", "Spanish"],
  "last_login": "2024-02-22T08:30:00Z",
  "subscription": {
    "type": "premium",
    "start_date": "2023-01-01",
    "end_date": "2024-12-31"
  }
} 
```

If we’re ingesting hundreds of thousands of these objects, storing those all in memory is going to get expensive quickly. Generators enable us to avoid creating these objects all in memory at once. This can be invaluable in reducing costs and increasing performance in memory limited cloud compute environments.

## Gotchas

Although convenient, generators *do* have some limitations that must be considered before using them:

### Generators can’t be rewound or peeked without additional code

Once a value has been generated, it cannot be regenerated from the same generator. Likewise, there is no way to compute the next value of a generator without also consuming it.

### Generators can be tricky to debug

Line-by-line debuggers often struggle inspecting the underlying object as they are unrewindable by nature. I am not currently aware of a python visual debugging application that allows for the inspection of generated values without also consuming the value. This can be worked around by implementing custom code on top of generators.

### Generators can increase mental load of understanding code

The under utilization of generators means that they are often difficult to understand for those with limited experience with python. It can take some time for people to develop the mental abstraction of “this effectively becomes a list when iterated over”. The apparent “out-of-order” execution can be hard to parse at first. Take the example:

```
def do_hello():
    print("hello")
    yield 5
    print("world")

hi = do_hello()
print("goodbye")
for test in hi:
    pass 
```

It can be confusing, without a solid understanding of generators, to parse the order of output in this script.

</main>