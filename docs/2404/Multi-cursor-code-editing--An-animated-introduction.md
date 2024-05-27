<!--yml
category: 未分类
date: 2024-05-27 13:22:52
-->

# Multi-cursor code editing: An animated introduction

> 来源：[https://alexharri.com/blog/multi-cursor-code-editing-animated-introduction](https://alexharri.com/blog/multi-cursor-code-editing-animated-introduction)

<main class="css-fkkl8v">

# Multi-cursor code editing: An animated introduction

January 28, 2023

When editing text, especially structured text, the need occurs to make repeated changes in multiple locations. A common case is renaming a variable.

Select and type, select and type.

For a small block of code it's fine. A bit tedious, but fine. Banging this out doesn't take a lot of time.

However, the number of keystrokes grows linearly. Increasing the number of references to a few dozens already makes the task quite taxing.

We want the effort for repeated changes to grow in a non-linear fashion. We can do that using Command `D`.

Note: This post is focused on VS Code, but Command `D` can be used in other text editors such as Sublime Text. Other text editors will have analogous keyboard shortcuts.

Command `D` selects the next instance of whatever you have selected, which enables multi-cursor editing.

Using Command `D` seems deceptively simple. Find a pattern to match, and then make the change:

There's already a lot of value in using Command `D` for simple transformations, such as the above, but we're just scratching the surface. Combined with smart text navigation techniques, we can take Command `D` quite far.

## Navigating text

First off, the basics.

*   **Arrow keys** to move the cursor
*   **Shift** to select text while moving the cursor.

Use Option to jump over words.

Jumping over words allows us to navigate text containing words of different lengths.

Use Command to jump to the beginning or end of a line.

Jumping to line boundaries allows us to navigate text that contains a variable number of words.

With text navigation locked down, let's do some cool stuff.

## Finding the pattern

Take this example of converting a series of `if` statements to a switch statement.

The `if` statements all have the exact same structure, so matching them is somewhat trivial. These sorts of patterns are the bread and butter of Command `D`, they're very common.

But Command `D` is still very effective for non-uniform patterns. Those more complex patterns can come in the form of

*   a variable number of arguments,
*   a variable number of words in a string, or
*   different argument types.

Let's take a look at an example.

### Non-uniform patterns

Let's say that we're developing a library for evaluating math expressions.

```
import  { evaluate }  from  "imaginary-mathlib";evaluate("2 * 4"); evaluate("[5, 10] / 2"); evaluate("1 > 1/2 ? 1 : 'err'"); 
```

In making testing the library less verbose, we made a utility function that takes an expression, and its expected value.

```
function  expectEqual(expression:  string, expectedValue:  any):  void;
```

We have some test code using it that looks like so:

```
expectEqual("2**4",  16);expectEqual("1/0",  ERR_DIV_ZERO);expectEqual("[1, 3, 5] * 2",  [2,  6,  10]);expectEqual("1/10 < 0.2 ? 'a' : 'b'",  "a");
```

However, we want to convert this test code into the following:

```
const tests =  [  { expression:  "2**4", value:  16  },  { expression:  "1/0", value:  ERR_DIV_ZERO  },  { expression:  "[1, 3, 5] * 2", value:  [2,  6,  10]  },  { expression:  "1/10 < 0.2 ? 'a' : 'b'", value:  "a"  },];
```

Since we have a lot of tests, doing this manually would be a lot of work. This is a prime case for using Command `D`, we just need to find a pattern to match.

If we match `expectEqual` and move in from there, we run into the problem of the expressions being of different lengths.

Matching the end runs into the same problem. The values are of different lengths.

If we try to match the commas `,` between the expression and the value, we also match commas within the expressions and expected values:

The expression and expected value can be of any length, so matching the start or end is of no use.

However, we can observe that the expression is always a string. The expression always ends with double quote `"` immediately followed by a comma `,`. That's a pattern we can match!

## Matching every instance

In the example above, we matched four tests. That's a pretty small number of tests, especially for a library that evaluates math expressions.

Pressing Command `D` three times is not a lot of work, but if the number of tests were increased to 1,000 we would need to press Command `D` 999 times. This goes against our goal of making repeated changes grow non-linearly.

This is a nice time to introduce Shift Command `L`, which is the keyboard shortcut for *Select All Matches*.

You have to be a bit more careful with Shift Command `L`, since it selects **every** match in a file. You may match something that you did not intend to, which can occur outside of the current viewport.

For this reason, I prefer Command `D` when working with a small number of matches. The matching feels more local, you visually see every match happen.

## Skipping an instance

When selecting matches, you may want to skip an instance. To skip a match, press Command `K` followed by Command `D`.

In order to skip a match, you first need to add the match to the selection. After you have added a match to your selection, press Command `K` and Command `D` to unselect it and select the next match.

Pressing Command `K` and Command `D` resolves to a command called **Move Last Selection to Next Find Match**. It's quite a technical name, but basically means

*   remove the most recent match, and
*   select the next match.

This is not very intuitive at first, but becomes second-nature given enough practice.

## Matching line breaks

Matching every line can be useful when working with arbitrary data.

Take this text file:

```
PythonJavaC++GoRustElixir
```

Let's say that we want to convert the lines of this file into a JSON array of strings:

```
[  "Python",  "Java",  "C++",  "Go",  "Rust",  "Elixir",]
```

There is no pattern across these lines, so matching each line seems impossible. However, Command `D` allows us to match newlines.

Matching newlines is occasionally useful when

*   matching every line, or
*   matching a pattern that only appears at the end of a line, or
*   matching a pattern that spans two or more lines.

For an example of matching a multi-line pattern, take this example of only matching the empty arrays:

## Case transformations

Translating between cases (such as changing snake-case to camelCase) comes up from time-to-time. I typically encounter this case when working across HTML, CSS and JavaScript.

VS Code has a handy `Transform to Uppercase` command that we can combine with Command `D` to make this happen.

There is not a direct keyboard shortcut for the `Transform to Uppercase` command. In VS Code, you can run it by opening the command prompt with Shift Command `P` and then typing the name of the command.

Note: Unfortunately, you will not be able to use the `Transform to Uppercase` method in this editor. This post uses Monaco Editor, which does not have VS Code's command prompt.

## That's a wrap!

There are many ways to do multi-cursor editing using VS Code, but I find Command `D` to be the simplest and most useful method.

Take what you learned in this post and apply it in your own work! There is a learning curve, but if you get past it then I promise that Command `D` will prove itself to be a really useful and productive tool.

Thanks for reading the post!

</main>