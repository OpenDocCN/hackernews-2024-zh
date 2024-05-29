<!--yml
category: 未分类
date: 2024-05-29 12:30:04
-->

# LaTeX3 Automatic Labels for Fun and No Profit

> 来源：[https://commutative.xyz/~miguelmurca/blog/x/autoref.html](https://commutative.xyz/~miguelmurca/blog/x/autoref.html)

# LaTeX3 Automatic Labels for Fun and No Profit

I’m a PhD student in physics, which means I’ve spent the better part of the last 10 years writing LaTeX. For those not in the know, LaTeX is a 40-year old superset of a 46-year old typesetting system – i.e., a macro based programming language to produce print documents. Notably, it’s mostly intended *not* as a programming language; its strongest suit is arguably the way it beautifully typesets mathematics, and its solution to express complex mathematical expressions. For example,

```
\sqrt{\frac{a+b}{\vert{c}\vert} 
```

refers to <math xmlns="http://www.w3.org/1998/Math/MathML" display="inline"><mrow><msqrt><mrow><mfrac><mrow><mi>a</mi> <mo>+</mo> <mi>b</mi></mrow> <mrow><mi>|</mi> <mrow><mi>c</mi></mrow> <mi>|</mi></mrow></mfrac></mrow></msqrt></mrow></math> . Indeed, if you’ve ever needed to write a mathematical expression into a computer, you’re likely to have used either TeX or some form of pidgin TeX.

But, (La)TeX really is Turing complete – it’s just extremely convoluted. This makes (La)TeX a reasonably fun esoteric programming language to play around with. On the other hand, being able to wrangle (La)TeX’s macro system lets you automate repetitive tasks, or generally extend (La)TeX’s functionality, which end up being of practical use. This is why there is also a community effort to improve programming ("macro writing") in LaTeX.

LaTeX3 is a “new kernel for LaTeX [… based on] its own consistent interface to all the functions needed to control TeX."^([[@expl3.pdf]](https://mirrors.up.pt/pub/CTAN/macros/latex/contrib/l3kernel/expl3.pdf)) The “new” qualifier is arguable, seeing as it has been in development since 1989, but, in short, we now have access to a set of base macros in LaTeX which are more sophisticated and behave more predictably. Unfortunately, however, due to Knuth’s own [fondness of literate programming](https://en.wikipedia.org/wiki/Literate_programming) (the man invented the concept, after all), and because of the nature of LaTeX’s output,^([citation needed]) most information about LaTeX3’s functionality is buried deep in long PDFs with interspersed prose and code and accessible only via `texdoc <designator you must guess>`. A notable (and welcome) exception is [this article by Alan Xiang, which I recommend reading](https://www.alanshawn.com/latex3-tutorial/). In any case, this post is my attempt to make a small contribution to practical and digestible LaTeX3 materials, so that you, too, can procrastinate writing your document by writing very convoluted LaTeX macros.

## Problem statement and goal

In LaTeX, you may `\label{...}` sections, equations, …, and later refer to their identifier with `\ref{...}`. So, for example,

```
\documentclass{article}
\begin{document}

\section{First section}
\label{sec:first-section}

Some text here.

This is actually section~\ref{sec:first-section}.

\end{document} 
```

yields

```
1 First section

    Some text here.
    This is actually section 1. 
```

As said, this also works for equations, and is most useful in mathematical documents, where you want to reference equations in the body of text. LaTeX2e provides the `equation` environment, which automatically typesets a nice `(eqno)` next to your equation, and which you can label and reference:

```
\documentclass{article}
\let\implies\Longrightarrow % \implies == \Longrightarrow
\begin{document}

\begin{equation} \label{eq:a-implies-b}
    A \implies B
\end{equation}
\begin{equation} \label{eq:b-implies-c}
    B \implies C
\end{equation}

From eqs.~$(\ref{eq:a-implies-b}, \ref{eq:b-implies-c})$, we conclude that $A$ implies
$C$.

\end{document} 
```

yields something as

```
 A ⇒ B       (1)
            B ⇒ C       (2)

From eqs. (1,2), we conclude that A implies C. 
```

However, labels must be unique throughout the document. And so, when writing out a long document, it quickly becomes quite upsetting to think up a good label for an equation you are certain you will only reference in the next line of text. Wouldn’t it be nice to have some `\AutoLabel` and `\AutoRef` macros that would let you just

```
\begin{equation} \AutoLabel
    1 + 1 = 2
\end{equation}

Equation~\AutoRef{} can be proven with set theory. 
```

## First steps

The simplest approach to this problem is to have `\AutoLabel` generate a different label every time it’s called in a systematic way, and have `\AutoRef` reference that label when it’s called.

```
\documentclass{article}
\let\implies\Longrightarrow

\ExplSyntaxOn
\int_new:N \g_autolabel_int
\NewDocumentCommand{\AutoLabel}{}{
    \int_gincr:N \g_autolabel_int
    \exp_args:Ne \label {autolabelprefix- \int_use:N \g_autolabel_int}
}
\NewDocumentCommand{\AutoRef}{}{
    \exp_args:Ne \ref {autolabelprefix-\int_use:N \g_autolabel_int}
}
\ExplSyntaxOff

\begin{document}

\begin{equation} \AutoLabel
    A \implies B
\end{equation}

I have just defined eq.~\AutoRef{}.

\end{document} 
```

Let’s break this down line by line:

Anything between `\ExplSyntaxOn` and `\ExplSyntaxOff` obeys LaTeX3’s syntax rules, rather than LaTeX2e’s rules. Indeed, (La)TeX allows you to (roughly speaking) re-define syntax rules on the fly; in particular, the “function” of each character as it is parsed. So, for example, while `\` is usually a special character (indicating that what follows is a control sequence), you can change its value halfway through the document to function as a regular character. A typical (ab)use of this mechanism in LaTeX2e was the modification of the “character type” (the “character code”, or “catcode") of `@` between special character and regular letter to guard internal commands from the end-user. This procedure was summarized with the `\makeatletter` and `\makeatother` commands (to, respectively, set the catcode of `@` to “regular letter” and “other character"), and you’ll often find, in LaTeX2e code, code that looks like:

```
\makeatletter
\newcommand{\@internalcommand}{Do something}
\newcommand{\endusercommand}{\@internalcommand\ and something else.}
\makeatother 
```

Calling `\@internalcommand` after `\makeatother` yields an error, because, after `\makeatother` is called, LaTeX doesn’t know how to interpret the character `@` anymore. This also highlights the fact that the catcode switching happens as these declarations are parsed, even though the expressions in the declarations themselves are *not* evaluated at the time of declaration. This finds expression in the following example, a mistake I’ve incurred in more than once:

```
\newcommand{\boom}{\makeatletter\@destroytheworld\makeatother} 
```

Calling `\boom` later in the document errors out, since at the time of declaration, `\@` is parsed as the command in the declaration, not `\@destroytheworld`. At macro declaration time, the parser sees `\makeatletter`, but only as a macro that is part of the declaration – it doesn’t make sense to “run” the definition at declaration time, after all. It thus parses the rest of the declaration based on its current rules, which do *not* expect `@` to be a regular character. So, as far as the parser cares, `\boom` reads `[macro: \makeatletter] [macro: \@] [letters: destroytheworld] [macro: \makeatother]`. When `\boom` is finally called, LaTeX complains about the use of `\@`.

This is a reasonably long tangent to say that `\ExplSyntaxOn` modifies catcodes, to favour better and more consistent macro names, and to provide a nicer “programming” environment. There are two notable effects: first, within `expl3` rules, spaces are ignored. Really, dealing with spaces within macro declarations is a true pain point of LaTeX2e; not just spaces you type out, but also spaces implied by new lines. Here’s an example; what do you expect the differences to be between `\a`, `\b`, `\c`, and `\d`?

```
\newcommand{\a}{foobar}
\newcommand{\b}{
    foobar
}
\newcommand{\c}{
    foo
    bar%
}
\newcommand{\d}{foo%
    bar} 
```

I tested these out by calling `.\a..\b..\c..\d.` within a LaTeX document; the periods are there to highlight eventual spacing. The results are as follows:

```
.foobar..␣foobar␣..␣foo␣bar..foobar. 
```

(I’ve highlighted spaces for you, with the `␣` character.) Essentially, what you would expect happens in the body of text – that a single newline becomes a space – is also happening within the macro declaration, inserting spurious spaces everywhere. These spaces can only be avoided by inserting comment characters, `%`, before the end of the line, which the parser interprets as an explicit instruction to ignore the newline. This results in a mess of `%` everywhere in macro declarations, since, at some point, macro writers start putting a `%` at the end of every line where they don’t *explicitly* want some space. While, in the example given, the spaces are not too critical (although unwanted), within complicated macro declarations spurious spaces will cause some really nasty bugs and crashes. The situation is even worse with double newlines, which get transformed into a paragraph, and that will *really* mess up your command’s parsing.

The other notable catcode switch is making `:` and `_` “regular letters”. So, while in LaTeX2e, macro names are generally composed of `a-zA-Z` (and, eventually, `@`), in LaTeX3, macros names are expected to be of the form (`texdoc expl3`, sec. 3.2.),

```
\⟨module⟩_⟨description⟩:⟨arg-spec⟩ 
```

where `module` would be a unique per-macro package prefix, to avoid name collisions, and `description` would be the actual name of the macro. `arg-spec` is a bit more special, telling you what *kind* of argument the macro expects, and, notably, if the command will expand it or not. Think of it as only slightly more than type hints. Macro expansion is a big topic – too big to cover here – but here’s an illustrative example:

```
\newcommand{\foobar}{A winner is you!}
\newcommand{\myname}{foobar}

\csname\myname\endcsname % The macro named `\myname`
                         % (so, `\\myname`, if it were possible.)

\expandafter\csname\myname\endcsname % \foobar 
```

Read more about expansion [here](https://www.overleaf.com/learn/latex/Articles/How_does_%5Cexpandafter_work%3A_The_meaning_of_expansion).

In `expl3`, the `arg-spec` bit of a macro will tell you what the macro consumes, and if it will expand what it consumes. The main ones to care about (though you can find a complete list in `texdoc expl3`, chapter 3, and throughout the main reference for programming in LaTeX3, `texdoc interface3`) are:

*   `n` – a braced set of characters ("token list"),
*   `N` – a single character ("token"); a macro is a single token,
*   `c` – a macro *name* (`foobar`, rather than `\foobar`); thus, a token list,
*   `V` – a variable, i.e., a macro containing a value (read on),
*   `v` – a variable’s *name*; like `c`, but for `V`,
*   `e` – a braced set of characters to be fully expanded before consumption,
*   `o` – a braced set of characters to be expanded once before consumption,
*   `T` and `F` – a braced set of characters to be inserted in case some condition evaluates to true or false, respectively.

The notion of a “variable” also appears here, as LaTeX3 tries to separate the notion of a macro that *does* something (a function) from a macro that merely stores some value (a variable). So, in LaTeX3 terms, `\foo` below would be a variable, whereas `\baz` is a function:

```
\newcommand{\foo}{winner}
\newcommand{\baz}[1]{A #1 is you!}

\baz{\foo} 
```

Though I should really say,

```
\ExplSyntaxOn
\cs_new:Nn \baz:N {
    A #1 is you!
}
\tl_new:Nn \foo {winner}
\baz:N \foo
\ExplSyntaxOff 
```

Even more correct would be

```
\ExplSyntaxOn
\cs_new:Nn \baz:N {
    A #1 is you!
}
\cs_generate_variant:Nn \baz:N {V}
\tl_new:Nn \foo {winner}
\baz:V \foo
\ExplSyntaxOff 
```

And, finally, heck it, `texdoc expl3` specifies that variables should be named as `\⟨scope⟩_⟨module⟩_⟨description⟩_⟨type⟩`, so here’s some good, honest to God, LaTeX3:

```
\ExplSyntaxOn
\cs_new:Nn \custom_baz:N {
    A #1 is you!
}
\cs_generate_variant:Nn \custom_baz:N {V}
\tl_new:Nn \l_custom_foo_tl {winner}
\custom_baz:N \l_custom_foo_tl
\ExplSyntaxOff 
```

This is the price to pay for the magical expansion control tools that LaTeX3 gives.

Finally, we move to the following line,

```
\int_new:N \g_autolabel_int 
```

Here, we’re declaring a new integer variable. As good LaTeX3 netizens, we use the correct variable naming scheme, and declare the variable to be global with `g_` (labels will need to be globally unique to the document, so this makes sense). We can expect this variable to be initialized to 0; from page 168 of `texdoc interface3`:

> The ⟨integer⟩ is initially equal to 0.

The idea will be that we can generate infinite unique labels by choosing a unique prefix, and suffixing it with an increasing integer; `\g_autolabel_int` will hold this integer.

Next line:

```
\NewDocumentCommand{\AutoLabel}{}{
    \int_gincr:N \g_autolabel_int
    \exp_args:Ne \label {autolabelprefix- \int_use:N \g_autolabel_int}
} 
```

Here we declare a user-facing command, with `\NewDocumentCommand` (LaTeX3’s version of `\newcommand`, much nicer to use and about which you can read with `texdoc xparse`). Thus, there is no need to adhere to LaTeX3’s naming conventions. The declaration furthermore specifies that it takes no arguments from the user, and leaves in its place, when called, two instructions: a global increment of the `\g_autolabel_int` variable, and `\exp_args:Ne \label {autolabelprefix- \int_use:N \g_autolabel_int}`. Here is what `interface3.pdf` has to say about `\exp_args:Ne`:

> This function absorbs two arguments (the ⟨function⟩ name and the ⟨tokens⟩) and exhaustively expands the ⟨tokens⟩. The result is inserted in braces into the input stream after reinsertion of the ⟨function⟩. Thus the ⟨function⟩ may take more than one argument: all others are left unchanged.

Ideally, we would have some labelling function `\label:n` with a variant `\label:e`, that would expand whatever would be given to it before taking the result as the actual label text. But, we don’t, since `label` is a LaTeX2e command (`texdoc latex2e`, chap. 7.1). So, we use `LaTeX3`’s `\exp_args:Ne`, to leave the first token (`\label`) alone while the braced tokens that follow are not fully expanded. Only then is this token list passed as argument to `\label`.

Inside the token list, we find `autolabelprefix- \int_use:N \g_autolabel_int`: since `\int_use:N` will “[recover] the content of an ⟨integer⟩ and [place] it directly in the input stream”, we get, after full expansion, our unique label, of the form

```
autolabelprefix-⟨label number⟩ 
```

Finally, we have the declaration of the command that references this label:

```
\NewDocumentCommand{\AutoRef}{}{
    \exp_args:Ne \ref {autolabelprefix-\int_use:N \g_autolabel_int}
} 
```

It’s fairly similar to `\AutoLabel`, with the exception that the relevant LaTeX2e function is now `\ref`, rather than `\label`. Since we know the form of the generated labels, we reconstruct the label at referencing time, by once again using the `\g_autolabel_int` variable and `\exp_args:Ne`.

## Multiple Labels

The above solution works fine, but it quickly breaks down if we wish to reference two equations after *both* their declarations. To recover a previous example,

```
% snip
\begin{equation} \AutoLabel
    A \implies B
\end{equation}
\begin{equation} \AutoLabel
    B \implies C
\end{equation}

Eqs.~\AutoRef\ and \AutoRef\ imply $A \implies C$. 
```

will not work, as the output will read

```
Eqs. (2) and (2) imply A ⇒ C. 
```

We can deal with this quite simply, by counting the number of labels and references separately:

```
\ExplSyntaxOn
\int_new:N \g_autolabel_int
\int_new:N \g_autoref_int
\tl_const:Nn \c_autoprefix_tl {autolabelprefix-}

\NewDocumentCommand{\AutoLabel}{} {
    \int_gincr:N \g_autolabel_int
    \exp_args:Ne \label{ \tl_use:N \c_autoprefix_tl \int_use:N \g_autolabel_int }
}
\NewDocumentCommand{\AutoRef}{} {
    \int_gincr:N \g_autoref_int
    \exp_args:Ne \ref{ \tl_use:N \c_autoprefix_tl \int_use:N \g_autoref_int}
}
\ExplSyntaxOff 
```

I’ve sneaked in `\c_autoprefix_tl` as a constant token list holding the prefix, just because it looks that much nicer than a repeated arbitrary constant string throughout the source, and I’ve also done away with the scope for the sake of readability. Otherwise everything is still quite similar to the previous definition.

Things start going awry when using, for example, the `gather` environment from the `amsmath` package:

```
% snip
\begin{gather} 
    A \implies B \AutoLabel \\
    B \implies C \AutoLabel
\end{gather} 
```

produces

```
Eqs. ?? and ?? imply A ⇒ C. 
```

What’s going on? LaTeX3 does have some console debugging capabilities, with commands such as `tl_show:N`, but it’s easier, here, to just go with LaTeX’s version of print-debugging: placing values directly in the text. We thus modify the `\AutoLabel` definition to not only label the equations, but also place the label’s name in the text stream:

```
% snip
\NewDocumentCommand{\AutoLabel}{} {
    \int_gincr:N \g_autolabel_int
    % 👇 new addition
    {\tl_use:N \c_autoprefix_tl \int_use:N \g_autolabel_int}
    \exp_args:Ne \label{ \tl_use:N \c_autoprefix_tl \int_use:N \g_autolabel_int }
}
% snip 
```

Once again compiling, we find that the equations now read:

```
 A ⇒ Bautolabelprefix−3   (1)
    B ⇒ Cautolabelprefix−4   (2) 
```

Huh?! Where did labels 1 and 2 go? The problem seems to happen only with `amsmath` environments, and so a little bit of trolling its documentation (`texdoc amsmath`) reveals the following paragraph:

> `ifmeasuring@` – All display environments get typeset twice—once during a “measuring” phase and then again during a “production” phase; `\ifmeasuring@` will be used to determine which case we’re in, so we can take appropriate action.
> 
> `\newif\ifmeasuring@`

Uh oh. Our `\AutoLabel` function is getting called twice: once in a box that `amsmath` uses to perform measurements, and then discards, and then again when the actual typesetting happens. To make matters worse, `amsmath` is using a Plain TeX Boolean, `\ifmeasuring@`, *and* the Boolean’s name has a `@` in it. Here’s what `interface3.pdf` has to say on its Booleans, and Plain TeX’s Booleans:

> TeXhackers note: The `bool` data type is not implemented using the `\iffalse`/`\iftrue` primitives, in contrast to `\newif`, etc., in plain TeX, LaTeX2e and so on. Programmers should not base use of `bool` switches on any particular expectation of the implementation.

Plain TeX Booleans are very fickle, and likely to misbehave if the Boolean test looks anything other than

We might get away with giving some arguments to `\a` or `\b`, but no more than that; certainly, something like `\ifbool\a\b\c\else\d\e\f\fi` is bound to cause you trouble. So, our first step is moving the whole of `\AutoLabel` into a single macro of its own:

```
\ExplSyntaxOn
\int_new:N \g_autolabel_int
\int_new:N \g_autoref_int
\tl_const:Nn \c_autoprefix_tl {autolabelprefix-}

\cs_new:Nn \autolabel: {
    \int_gincr:N \g_autolabel_int
    \exp_args:Ne \label{ \tl_use:N \c_autoprefix_tl \int_use:N \g_autolabel_int }
}
\NewDocumentCommand{\AutoLabel}{}{
    \autolabel:
}
\NewDocumentCommand{\AutoRef}{}{
    \int_gincr:N \g_autoref_int
    \exp_args:Ne \ref{ \tl_use:N \c_autoprefix_tl \int_use:N \g_autoref_int}
}
\ExplSyntaxOff 
```

Now, we need only to test `\ifmeasuring@` inside `\AutoLabel`, and behave accordingly; i.e., if, indeed, measuring, we don’t want to do anything at all. But, recall that LaTeX3 and LaTeX2e’s relationship with `@` is different, and LaTeX3 won’t correctly interpret `\ifmeasuring@` at all (it’s not expecting macro names to have `@`s in them). We may get around this by first using a `c`-type argument, and an auxiliary definition:

```
% snip
\NewDocumentCommand{\AutoLabel}{}{
    \cs_set_eq:Nc \amsmath_ifmeasuring {ifmeasuring@}
    \amsmath_ifmeasuring\else\autolabel:\fi
}
% snip 
```

We’re using the `Nc` variant of `\cs_set_eq:NN`, about which `interface3.pdf` tells us:

> Globally creates ⟨control sequence1⟩ and sets it to have the same meaning as ⟨control sequence2⟩ or ⟨token⟩. The second control sequence may subsequently be altered with- out affecting the copy.

Is this very kosher? Absolutely not. Is this the least cursed LaTeX macro ever written? Not even close, and it works, to boot. Run your compilation steps again, and you’ll find the document now correctly reads

```
 A ⇒ B               (1)
        B ⇒ C               (2)

Eqs. 1 and 2 imply A ⇒ C. 
```

## Some final improvements

The last thing that’s missing is the ability to repeatedly reference the same equation. As it stands, there is no way to typeset something like

```
Eqs. 1 and 2 imply A ⇒ C, but eq. 1 does not imply B ⇒ A. 
```

Let’s fix that. Specifically, let’s modify `\AutoRef` to take in an optional argument. If the argument is present, and equal to `n`, then that command should reference the `n`th equation before it. Thus, our previous example would correspond to something like

```
Eqs.~\AutoRef\ and \AutoRef imply $A \implies C$, but \AutoRef[2] does not imply $B \implies A$. 
```

I’ll once again give you the final answer, and then break down any new elements:

```
\ExplSyntaxOn
\int_new:N \g_autolabel_int
\int_new:N \g_autoref_int
\tl_const:Nn \c_autoprefix_tl {autolabelprefix-}

\cs_new:Nn \autolabel: {
    \int_gincr:N \g_autolabel_int
    \exp_args:Ne \label{ \c_autoprefix_tl \int_use:N \g_autolabel_int}
}
\NewDocumentCommand{\AutoLabel}{}{
    \cs_set_eq:Nc\amsmath_ifmeasuring{ifmeasuring@}
    \amsmath_ifmeasuring\else\autolabel:\fi
}
\NewDocumentCommand{\AutoRef}{o}{
    \cs_set_eq:Nc\amsmath_ifmeasuring{ifmeasuring@}
    \IfValueF{#1}
        {\amsmath_ifmeasuring\else\int_gincr:N \g_autoref_int\fi}
    \IfValueTF{#1}
        {\int_set_eq:NN \l_tmpa_int \g_autolabel_int}
        {\int_set_eq:NN \l_tmpa_int \g_autoref_int}
    \IfValueT{#1}
        {\int_sub:Nn \l_tmpa_int {#1 - 1}}
    \exp_args:Ne \ref {\tl_use:N \c_autoprefix_tl \int_use:N \l_tmpa_int}
}
\ExplSyntaxOff 
```

First, note that I’ve also guarded the `\g_autoref_int` counter against double incrementation due to measuring – something we’d neglected to previously do, but it’s plausible `\AutoRef` is called within an `amsmath` environment. Otherwise, the main difference to the previous definitions has to do with the inclusion of the optional (`o`) argument in the declaration of `\AutoRef`.When no value is given for these arguments, they take a special flag value (usually denoted `-NoValue-` in the documentation). `xparse` provides the nice `IfValue(TF)` macros to deal with the cases where, respectively, the argument takes some value, or no value.

So, when we find that the user passed some optional value, we calculate the corresponding “absolute label number”; this is given by the current auto-labelling number, minus the number the user gave minus one (in other words: if `g_autolabel_int` is currently `2`, implying `autoprefix-1` and `-2` were already defined, and the user passes in `\AutoRef[2]`, they are referencing `autoprefix-1`, and `1 = 2-(2-1)`).

If, on the other hand, the user did not pass in any value, then they are referencing the most recent label; as before, we increment `g_autoref_int`, and set `\l_tmpa_int` to hold this number:

```
\amsmath_ifmeasuring\else\int_gincr:N \g_autoref_int\fi
\int_set_eq:NN \l_tmpa_int \g_autoref_int 
```

Then, the final line correctly references the most recent label:

```
\exp_args:Ne \ref {\tl_use:N \c_autoprefix_tl \int_use:N \l_tmpa_int} 
```

This *almost* works, except for the following edge case: what do you expect the following code to produce?

```
\begin{equation} \AutoLabel
    A = A
\end{equation}

\AutoRef[1] constitutes a ``tautology''.

\begin{equation}
    A = B
\end{equation}

Eq.~\AutoRef{} does not constitute a tautology. 
```

As it stands, the first call to `\AutoRef[1]` correctly references the label of the first equation, but it does *not* advance `\g_autoref_int`, because a value is given. Therefore, when `\AutoRef` is called a second time, in the second paragraph, it again references the first equation. This is, however, the behaviour we might expect in the following situation (which, with apologies, is more synthetic):

```
\begin{gather}
    A \AutoLabel \\
    B \AutoLabel \\
    C \AutoLabel
\end{gather}

Eq~\AutoRef[1] and \AutoRef{} and \AutoRef{}. 
```

Indeed, here, we do *not* expect `\AutoRef[1]` to advance `\g_autoref_int`.

The bottom line is that, to get the intended behaviour, calling `\AutoRef[1]` should be indifferent from simply calling `\AutoRef`; but only in the case where the two would produce the same result. This is not currently the case. But!, LaTeX3 has us covered with some pretty comprehensive arithmetic tools. The following modification to `\AutoRef` is sufficient:

```
% snip
\NewDocumentCommand{\AutoRef}{o}{
    \cs_set_eq:Nc\amsmath_ifmeasuring{ifmeasuring@}
    % 👇 main difference
    \tl_set:Nn \l_tmpb_tl {#1}
    \IfValueT{#1}
        {\int_compare:nNnT {#1}={1} {
            \int_compare:nNnT {\g_autoref_int}={\g_autolabel_int - 1} 
                {\tl_set:NV \l_tmpb_tl \c_novalue_tl}}}
    % 👆
    % 👇 But notice also the use of \l_tmpb_tl below, now:
    \exp_args:Ne \IfValueF{\tl_use:N \l_tmpb_tl}
        {\amsmath_ifmeasuring\else\int_gincr:N \g_autoref_int\fi}
    \exp_args:Ne \IfValueTF{\tl_use:N \l_tmpb_tl}
        {\int_set_eq:NN \l_tmpa_int \g_autolabel_int}
        {\int_set_eq:NN \l_tmpa_int \g_autoref_int}
    \exp_args:Ne \IfValueT{\tl_use:N \l_tmpb_tl}
        {\exp_args:NNe \int_sub:Nn \l_tmpa_int {\tl_use \l_tmpb_tl - 1}}
    \exp_args:Ne \ref {\tl_use:N \c_autoprefix_tl \int_use:N \l_tmpa_int}
}
% snip 
```

The idea is not so different; but now, instead of addressing the optional argument directly (`#1`), we start by assigning it to `\l_tmpb_tl`. Then, if we find that the user did pass in some value, but that this value is 1, and would result in the same as not having provided any value at all, we put the empty value into `\l_tmpb_tl`. The rest of the command is therefore processed, in that case, as though the user had not supplied a value.

Using `\l_tmpb_tl` instead of `#1` requires some further care with `\exp_args:Ne` to make sure the value of `\l_tmpb_tl` is expanded before it’s tested, but nothing special.

Thus, we have our final set of LaTeX3 macros:

```
\ExplSyntaxOn
\int_new:N \g_autolabel_int
\int_new:N \g_autoref_int
\tl_const:Nn \c_autoprefix_tl {autolabelprefix-}

\cs_new:Nn \autolabel: {
    \int_gincr:N \g_autolabel_int
    \exp_args:Ne \label{ \c_autoprefix_tl \int_use:N \g_autolabel_int}
}
\NewDocumentCommand{\AutoLabel}{}{
    \cs_set_eq:Nc\amsmath_ifmeasuring{ifmeasuring@}
    \amsmath_ifmeasuring\else\autolabel:\fi
}
\NewDocumentCommand{\AutoRef}{o}{
    \cs_set_eq:Nc\amsmath_ifmeasuring{ifmeasuring@}
    \tl_set:Nn \l_tmpb_tl {#1}
    \IfValueT{#1}
        {\int_compare:nNnT {#1}={1} {
            \int_compare:nNnT {\g_autoref_int}={\g_autolabel_int - 1} 
                {\tl_set:NV \l_tmpb_tl \c_novalue_tl}}}
    \exp_args:Ne \IfValueF{\tl_use:N \l_tmpb_tl}
        {\amsmath_ifmeasuring\else\int_gincr:N \g_autoref_int\fi}
    \exp_args:Ne \IfValueTF{\tl_use:N \l_tmpb_tl}
        {\int_set_eq:NN \l_tmpa_int \g_autolabel_int}
        {\int_set_eq:NN \l_tmpa_int \g_autoref_int}
    \exp_args:Ne \IfValueT{\tl_use:N \l_tmpb_tl}
        {\exp_args:NNe \int_sub:Nn \l_tmpa_int {\tl_use \l_tmpb_tl - 1}}
    \exp_args:Ne \ref {\tl_use:N \c_autoprefix_tl \int_use:N \l_tmpa_int}
}
\ExplSyntaxOff 
```

While quite a modest contribution to the [very hefty history of LaTeX packages](https://ctan.math.illinois.edu/), it nonetheless served as a basis for discussing important points of LaTeX3 macro writing, such as syntax differences, token list manipulation, controlled expansion, variables and functions, Boolean tests, and LaTeX2e interfacing. Not too shabby. Hopefully this post is also a good enough introduction that you may now directly reference important reference documents, such as `interface3.pdf`, yourself.

Personally, I also find the macro itself quite useful in practice, and it’s something I’ve been using in my scientific writing. I am yet to find out if Physical Review will be upset by it.

As a parting gift, and to point the interested user towards property lists, here is an exercise for the reader: the `revtex4-2` document class, mandated by Physical Review, does not support the use of `\footnotetext` and `\footnotenumber`. This means that any footnote text really must be in the middle of the source of the body of text. Can you write `\FootnoteLater` and `\FootnoteNow` to rectify this?

* * *

**A Post-Scriptum for HackerNews readers:** I like to submit these posts to HN, as I feel like the average HN user fits the intended audience. But, 1\. famously, HN can be quite predictable in some of their responses (by what I expect is, essentially, a meme effect), and 2\. I’ve had some unexpected experiences resulting from previously reaching FP in HN. So, consider this a preemptive response to some points I expect to be raised. If you’re coming in from HN and you see someone fail to account for these answers, you’ll know they haven’t even read the whole thing. So:

1.  It’s weird I even have to say this, but don’t stalk me and email me at my personal address. This is genuinely something that has happened, inexplicably. If you wish to contact me by email, by all means do so to `miguelmurca+autoref [æt] cumperativa.xyz`.
2.  Again, strange that I would need to point this out, but do not assume my nationality, or language. This post is in English. If you wish to write me, please do so in English.
3.  Yes, we are aware of [typst](https://github.com/typst/typst). I think it’s cool, but C++ hasn’t replaced C, Rust hasn’t replaced C++, Typst is unlikely to replace LaTeX. Likewise, many are aware of [LuaTeX](https://www.luatex.org/), but, again, the entrenching of a 40-odd year system is not to be underestimated. I am rooting for `typst`, anyway, and hope it finds its place. A good place to start would be to provide a compilation toolchain from `typst` to TeX, if they really want to replace TeX.
4.  There are, in fact, reasons why someone would not want to just use your favourite form of Markdown plus pidgin TeX, not least of all because not everyone is just taking notes, and also because not everyone (dare I say, most people) who are using LaTeX are of the computer science/technology subject. Also, on a purely personal level, if you were taking your class notes in Markdown+TeX, either the syllabus was too easy or you were making it way harder for yourself.
5.  Yes, LaTeX is ugly and antiquated. It *is* old. It’s an evolving, niche, very precocious thing, and it suffers from this. And, yet, it seems to have its place. But, if you just look at it like an esoteric programming language, why should this be a problem?
6.  Even though I strived to be correct, it’s likely I’ve incurred in some error. There are a lot of people who are *really* good with LaTeX in HN, and I expect them to point out any errors (for which I will be very grateful, and will duly correct). But, at the end of the day, LaTeX is far from my primary occupation, and I felt like the sort of introduction that this text would be able to give to the average programmer w.r.t. LaTeX would outweigh any eventual minor error, easily corrected by consulting the official references.
7.  Yes, it’s `$YEAR` and we’re still producing PDFs. Again, historical reasons plus the fact that most people doing maths are not necessarily very interested in computers.
8.  99% of the time that you’re writing LaTeX, you need to know literally nothing of what’s contained in this blog post.
9.  (La)TeX does not have a grammar. Parsing TeX is Turing complete. This does not mean that you could not write a grammar for, roughly speaking, “much” of TeX, especially “many” mathematical expressions. I suspect this is what quite a few pidgin TeX-related programs do.

* * *