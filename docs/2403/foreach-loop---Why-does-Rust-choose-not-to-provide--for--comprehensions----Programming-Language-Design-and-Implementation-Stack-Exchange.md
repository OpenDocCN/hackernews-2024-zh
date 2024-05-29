<!--yml
category: 未分类
date: 2024-05-27 14:49:57
-->

# foreach loop - Why does Rust choose not to provide `for` comprehensions? - Programming Language Design and Implementation Stack Exchange

> 来源：[https://langdev.stackexchange.com/questions/2848/why-does-rust-choose-not-to-provide-for-comprehensions](https://langdev.stackexchange.com/questions/2848/why-does-rust-choose-not-to-provide-for-comprehensions)

### Python List Comprehensions

In Python, there are 3 blessed collections -- `list`, `set`, and `dict` -- for which syntactic sugar built into the language allow creating one, or the other, directly. The List Comprehension syntax available in Python allows creating either:

```
new_list = [i + 1 for i in old_list]
new_set = {i + 1 for i in old_set}
new_dict = {k:v+1 for (k, v) in old_dict.items()} 
```

This is great when you use them... and less so when you'd like another.

### Equality: no second-class citizen in Rust!

Rust is a Systems Programming Language.

In Python, most developers do not know, nor care, exactly how `dict` works under the hood: what's the memory layout? The type of hash-map used? The hashing algorithm used?

Systems Programmers, however, do care. They may care due to performance reasons, they may care due to the lack of memory allocation in their domain (embedded, kernel). This means that unlike Python developers, they are *much* more likely to need specialized collections, rather than the run-of-the-mill standard collections.

And therein lies the rub. The thought of all those specialized collections being second-class citizens, and much more awkward to use than 3 blessed standard collections, did not sit well with Rust developers.

Therefore, instead of creating a list-comprehension syntax that would only be usable with 3 standard collections, they instead focused on making the operations you are looking for work with any collection via Rust's powerful trait system:

1.  The `Iterator` trait allows iterating over anything, and provides many way to filter/transform the elements being yielded.
2.  The `FromIterator` trait allows any type implementing it to be built from an `Iterator`.

Therefore, it is perfectly possible to create one collection from another:

```
let new: Vec<_> = [1, 2, 3, 4, 5].into_iter().map(|i| i + 1).collect();
let new: VecDeque<_> = [1, 2, 3, 4, 5].into_iter().map(|i| i + 1).collect();
let new: LinkedList<_> = [1, 2, 3, 4, 5].into_iter().map(|i| i + 1).collect(); 
```

*Note: It is not possible to *directly* create an array due to fallibility concerns, though maybe one should submit a RFC to allow collecting into a `Result<[_; _], _>` as that would handle the fallibility concerns.*

Iterator chains are very powerful: take, skip, filter, filter_map, enumerate, ... so that list comprehensions would be a step down (they can only map and filter) in many cases.

Is it worth having special syntax for the easy case? Well, so far, the answer from Rust developers has been that they don't feel it would be.

* * *

On a tangent, the same iterators chains are available for *extending* an existing collection with more elements:

```
my_collection.extend([1, 2, 3, 4, 5].into_iter().map(|i| i + 1)); 
```

The `extend` method takes any iterator yielding elements of the same type as the collection.

### Macros!

It should also be noted that Rust has macros.

A quick search on [crates.io](https://crates.io) reveals that some Rust users felt the need for quick & easy, and they went ahead and created macros for it, such as the [`list_comprehension_macro`](https://crates.io/crates/list_comprehension_macro) crate (and a bunch of others). From the README:

```
let even_squares = comp![x.pow(2) for x in arr if x % 2 == 0];
let flatten_matrix = comp![x for row in arr for x in row];
let dict_comp = comp!{x: x.len() for x in arr}; 
```

This, in turn, is also a disincentive to include any such functionality in the standard library, as it's easy enough to make your own, or use a 3rd-party library.

*Note: a somewhat minimal standard library is also an explicit goal of Rust developers.*