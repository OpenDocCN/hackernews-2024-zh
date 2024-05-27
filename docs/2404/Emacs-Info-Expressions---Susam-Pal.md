<!--yml
category: 未分类
date: 2024-05-27 13:13:25
-->

# Emacs Info Expressions - Susam Pal

> 来源：[https://susam.net/emacs-info-expressions.html](https://susam.net/emacs-info-expressions.html)

<main>

# Emacs Info Expressions

By **Susam Pal** on 12 Apr 2024

On `#emacs` IRC or Matrix channels, we often share references to the built-in Emacs documentation as Elisp expressions that look like this:

```
(info "(emacs) Basic Undo")
```

Here is another example:

```
(info "(emacs) Word Search")
```

This is a common practice in the Emacs channels even though all of the Emacs manual is available on the world wide web too. For example, the section referred to in the above expression is available here: [GNU Emacs Manual: Word Search](https://www.gnu.org/software/emacs/manual/html_node/emacs/Word-Search.html). The reason for sharing Elisp expressions like this is likely partly tradition and partly convenience. Many Emacs users are logged into IRC networks via Emacs itself, so once the recipient sees an Elisp expression like the above one in their chat buffer, visiting the corresponding manual page is a simple matter of placing the cursor right after the closing parenthesis and typing `C-x C-e`.

But isn't it clumsy for the sender to type Elisp expressions like this merely to share a pointer to a section of a manual with others? Turns out, it is not. This is Emacs! So of course there are key-bindings to do this.

## Copy Info Node Name

Say, while helping another Emacs user we type `M-x info-apropos RET version control RET` and land on the section "Branches" and realise that this is the section that the person we are trying to help should read. Now when we are on this section, we can simply type `c` and Emacs will copy the name of the current Info node to the kill ring. This name looks like this:

```
(emacs) Branches
```

Now we can go to the `*scratch*` buffer (or any buffer), copy the node name, and complete the `info` expression manually. For example, we could type the following key sequence on a fresh new line to create the Elisp expression and copy it to the kill ring:

```
" " C-b C-y C-a C-SPC C-e M-( info C-a C-k C-/
```

On vanilla Emacs, the above rather long key sequence first types two double-quotes next to each other (`" "`), then moves the cursor back to go within the double-quotes (`C-b`), then pastes the text `(emacs) Branches` from the kill ring (`C-y`), then selects the pasted text (`C-a C-SPC C-e`), then surrounds it within parentheses (`M-(`), then inserts the text `info` just after the opening parentheses, and finally copies the resulting expression to the kill ring (`C-a C-k C-/`). The expression copied to the kill ring looks like this:

```
(info "(emacs) Branches")
```

Can we avoid constructing the `info` expression manually and have Emacs do it for us? Turns out we can as we see in the next section.

## Copy Info Expression

I recently learnt from [Karthink](https://karthinks.com/) and [Mekeor Melire](https://mastodon.social/@mekeor@catgirl.cloud) that we can ask Emacs to create the entire `info` expression automatically for us. All we need to do is use the zero prefix argument with the `c` key. So when we are on section "Branches", if we type `C-0 c`, the following expression is copied to the kill ring:

```
(info "(emacs) Branches")
```

I should have known this because indeed while we are in the Info documentation browser, if we type `C-h k c` to describe the key sequence `c`, we see the following documentation:

```
c runs the command Info-copy-current-node-name (found in
Info-mode-map), which is an interactive byte-compiled Lisp function in
‘info.el’.

It is bound to c, w, <menu-bar> <info> <Copy Node Name>.

(Info-copy-current-node-name &optional ARG)

Put the name of the current Info node into the kill ring.
The name of the Info file is prepended to the node name in parentheses.
With a zero prefix arg, put the name inside a function call to ‘info’.

  Probably introduced at or before Emacs version 22.1.

[back]

```

So indeed Emacs has a convenient key sequence to create the complete `info` expression for the current Info node. The person who receives this `info` expression can visit the corresponding section of the manual simply by evaluating it. For example, after copying the expression in Emacs, they could simply type `C-y C-x C-e` to paste the expression into a buffer and evaluate it immediately. Alternatively, they might want to type `M-: C-y RET` to bring the `eval-expression` minibuffer, paste the expression, and evaluate it.

</main>