<!--yml
category: 未分类
date: 2024-05-27 14:43:12
-->

# Elisp in Replacement String - Susam Pal

> 来源：[https://susam.net/maze/elisp-in-replacement-string.html](https://susam.net/maze/elisp-in-replacement-string.html)

<main>

# Elisp in Replacement String

By **Susam Pal** on 09 Jan 2024

It is likely well known among Emacs users that the following key sequence starts a search-and-replace operation to match strings with the regular expression pattern `f..` and replace the matches with `bar`.

```
C-M-% f.. RET bar RET
```

Similarly, the following key sequence looks for strings that match a pattern that has two capturing groups and replaces each match with a new string that swaps the substrings matched by the capturing groups:

```
C-M-% \(f..\)-\(b..\) RET \2-\1
```

For example, this operation matches a string like `foo-bar` and replaces it with `bar-foo`. A string like `postfix-boxing` becomes `postbox-fixing`. The backreference `\1` refers to the string matched by the first capturing group `\(f..\)` and similarly `\2` refers to the string matched by the second capturing group `\(b..\)`. The replacement string then swaps the positions of the matches in both capturing groups.

What may be less commonly known is the ability to utilise Elisp expressions to dynamically compute portions of the replacement strings. To employ this feature, simply write `\,` (i.e., backslash and comma) followed by the Elisp expression in the replacement string. Consider the following key sequence:

```
C-M-% f.. RET \,(upcase \&) RET
```

The backreference `\&` refers to the whole match. We pass it as the argument to the Elisp function `upcase`. This function converts its argument to upper-case. This example searches for strings that match the pattern `f..` and replaces each match with the upper-case form of the match. A string like `foo-bar` is replaced with `FOO-bar`.

Here is another slightly more sophisticated example:

```
C-M-% host:\([0-9]+\) RET host:\,(+ 1000 \#1)
```

The backreference `\#1` refers to the string matched by the first capturing group `\([0-9]+\)` as a *number*. The Elisp expression in the replacement pattern simply adds `1000` to that number and replaces the matched string with the result. A string like `host:80` becomes `host:1080`. Another string like `localhost:8000` becomes `localhost:9000`.

Finally, here is an example from the real world of text editing where this feature was useful to me recently while solving a text editing problem. Consider the following text buffer with a list of numbered items:

```
### 1) apple
### 2) ball
### 3) bat
### 4) cat
### 5) dog
### 6) elephant
### 7) fish
### 8) grapes
### 9) hen
### 10) ink
### 11) jug
### 12) kite
### 13) lion

```

While this is a toy example presented here for the sake of simplicity and clarity, this example is based on an actual text editing problem I encountered recently. In my actual problem though, there were more words on each line and there were some arbitrary paragraphs of text between every consecutive pair of items. Further, the list was long with 50 or so items. The problem now was to remove item number 3 and renumber all the lines below it.

It is quite straightforward to remove item 3\. Just move the point (cursor) to that line and type `C-S-<backspace>` or `C-a C-k` to kill that line. We get this:

```
### 1) apple
### 2) ball
### 4) cat
### 5) dog
### 6) elephant
### 7) fish
### 8) grapes
### 9) hen
### 10) ink
### 11) jug
### 12) kite
### 13) lion

```

How do we now renumber all the items starting from `4) cat`? This is where the support for Elisp expressions in replacement strings turns out to be useful. First move the point to the beginning of that line. Then type the following key sequence:

```
C-M-% ^### \([0-9]+\) RET ### \,(1- \#1) RET
```

The search pattern captures the item number on each line in a capturing group. The replacement string contains an Elisp expression that subtracts one from this number. Thus a string like `### 4` gets replaced with `### 3`. After completing the replacement, the buffer looks like this:

```
### 1) apple
### 2) ball
### 3) cat
### 4) dog
### 5) elephant
### 6) fish
### 7) grapes
### 8) hen
### 9) ink
### 10) jug
### 11) kite
### 12) lion

```

I hope this was useful. Do you have an interesting Elisp-in-replacement-string story? Please share it in the comments.

</main>