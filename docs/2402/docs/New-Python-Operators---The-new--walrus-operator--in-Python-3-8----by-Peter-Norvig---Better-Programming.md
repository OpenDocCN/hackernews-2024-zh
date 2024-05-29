<!--yml
category: 未分类
date: 2024-05-27 14:59:44
-->

# New Python Operators!. The new “walrus operator” in Python 3.8… | by Peter Norvig | Better Programming

> 来源：[https://betterprogramming.pub/new-python-operators-9f31b56ddcc7?gi=7918b2c3ccf0](https://betterprogramming.pub/new-python-operators-9f31b56ddcc7?gi=7918b2c3ccf0)

Walrus Operator `:=`

The new “walrus operator” in Python 3.8 is written as `:=` and has been the topic of much [discussion](https://docs.python.org/3/whatsnew/3.8.html). This post describes a few of Python’s other whimsically-named but less-well-known multi-character operators.

These operators are announced today, April 1, 2023, but, much like Dorothy with her ruby slippers, you always had the power to use them, you just had to learn it for yourself!

# Ski Hat Operator

The “ski hat” operator is written as `*=0` and can be used to empty out a variable, be it a list, string, tuple, or numeric value.

For example, after executing the following code, `skiers` is an empty list.

```
skiers = ["Lindsey", "Alberto", "Bode"]

skiers *=0
```

Ski Hat Operator `*=0`

# Dumbbell Operator

The “dumbbell operator” is written as `[:]=[]` and can also be used to empty a list, but is not as versatile as the ski hat operator, as it doesn’t work for most other types.

After executing the following code, `reps` will be an empty list:

```
reps = [1, 2, 3, 4, 5]

reps [:]=[]
```

Dumbbell Operator `[:]=[]`

# Lapping Cat Operator

The “lapping cat” operator is written as `,=` and picks out the first element of an iterable. Like a finicky cat, it complains if there are other bothersome elements in the iterable.

After executing the following code, `water` is `'HHO'`.

```
water ,= ['H'*2+'O']
```

Lapping Cat Operator `,=`

# Starship Operator

The “starship” operator is written as `, *_=` and depicts a dual-nacelle starship (such as the Enterprise) alongside a photon torpedo. It has a similar effect to the lapping cat operator in picking out the first element of an iterable, but it allows the iterable to have more than one element.

After executing the following code, `NCC` is `1`:

```
NCC    , *_=    [1, 7, 0, 1]
```

Starship operator , *_=

# Flying Saucer Operator

The “flying saucer” operator is written as `--0--` and “beams up” an integer division, making it round up rather than round down. (I learned about it from [Mark Dickinson](https://www.enthought.com/team/mark-dickinson-2/).)

The following expression evaluates to 5, not 4:

```
--0--   42 // 10
```

Flying Saucer Operator `— 0 —`

# Emphasis Operator

The “emphasis operator” is written by surrounding an integer-valued expression with asterisks and is used to emphasize the following sequence by repeating it. For example,

```
sigh = 3
['oh', 'good', 'grief', *sigh* '!']
```

evaluates to `['oh', 'good', 'grief', '!', '!', '!']`.

The emphasis operator *sigh*

# Factorial Operator

Math fans will be pleased to learn that the factorial operator `n!` has been partially incorporated into Python.

Unfortunately, the implementation is incomplete. But these unit tests all pass, so that’s good enough, right?

```
assert 0!=1
assert 3!=6
assert 4!=24
assert 5!=120
assert 6!=720
assert 7!=5040
assert 8!=40,320
assert 9!=362,880
```

# Abstract keyword

Python also allows you to use the keyword `abstract` to indicate that a method of an abstract class must be implemented in a subclass for any instantiated object.

In the following code, the method `name` is defined as `abstract`, so a call to an object of the class results in an error message pointing out that the method is not defined.

```
class AnAbstractClass:
    def name(self): 
        abstract

>>> AnAbstractClass().name()
NameError: name 'abstract' is not defined
```

# More

The astute reader will recognize that these new operators rely on the placement of spaces in a way that violates [PEP 8](https://peps.python.org/pep-0008/).

To that I say: but my way is more fun! Especially today, April 1\. For more operators, including implementations of the `++` and `<<` operators from C++, see my [old pos](https://norvig.com/python-iaq.html)t.

PS: Everything in this post is true, except for “that’s good enough, right?”