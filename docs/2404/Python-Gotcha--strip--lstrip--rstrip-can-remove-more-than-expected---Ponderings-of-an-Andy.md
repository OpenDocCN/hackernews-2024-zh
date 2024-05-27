<!--yml
category: 未分类
date: 2024-05-27 13:35:04
-->

# Python Gotcha: strip, lstrip, rstrip can remove more than expected · Ponderings of an Andy

> 来源：[https://andrewwegner.com/python-gotcha-strip-functions-unexpected-behavior.html](https://andrewwegner.com/python-gotcha-strip-functions-unexpected-behavior.html)

## Introduction

As a software engineer, you've cleaned your fair share of dirty strings. Removing leading or trailing spaces is probably one of the most common things done to user input.

In Python, this is done with the [`.strip()`](https://docs.python.org/3.10/library/stdtypes.html#str.strip), [`.lstrip()`](https://docs.python.org/3.10/library/stdtypes.html#str.lstrip) or [`.rstrip()`](https://docs.python.org/3.10/library/stdtypes.html#str.rstrip) functions and generally looks like this:

```
`>>> "     Andrew Wegner     ".lower().strip()
'andrew wegner'
>>> "     Andrew Wegner     ".lower().lstrip()
'andrew wegner     '
>>> "     Andrew Wegner     ".lower().rstrip()
'     andrew wegner'` 
```

That's pretty straightforward and nothing unexpected in going on.

## Gotcha

The Gotcha is that each of these functions take a list of characters that can be removed.

```
`>>> "Andrew Wegner".lower().rstrip(" wegner")
'and'` 
```

What happened? Why wasn't the result just

## Explaination

Read line from the documentation again, carefully:

> A list of **characters**

Not a list of strings.

This is explicitly spelled out in the documentation, with an example, showing what the implications are. However, for a new developer, it's unexpected behavior. After all, these seem like intutive functions.

The example with my does the following:

1.  Receives a list of characters to remove. In this case it is all letters in my last name, plus the space character: `wegner`
2.  Lower case all letters in the input string, resulting in `andrew wegner`
3.  From the right hand side of the string, begin removing characters that are in the input list. Stop when you encounter a character not in the list. In this case that means that `rengew wer` are removed (right to left) and then the `d` in `andrew` is encountered so that `rstrip` function stops.
4.  Return the remaining string of `and`

## Solution

Python has two functions that will correctly remove a **string** - [`.removesuffix()`](https://docs.python.org/3.10/library/stdtypes.html#str.removesuffix) and [`.removeprefix()`](https://docs.python.org/3.10/library/stdtypes.html#str.removeprefix) for right and left side removals.

```
`>>> "Andrew Wegner".lower().removesuffix(" wegner")
'andrew'` 
```

These two functions were introduced in Python 3.9 as part of [PEP-616](https://peps.python.org/pep-0616/). In the PEP, it explicitly calls out the confusion users have about the `*strip()` functions and how they behave. These two were introduced to allow the desired behavior.

One important note is that these two `remove*` functions will only remove *at most* one instance of the string.

```
`>>> "Andrew Wegner Wegner".lower().removesuffix(" wegner")
'andrew wegner'` 
```