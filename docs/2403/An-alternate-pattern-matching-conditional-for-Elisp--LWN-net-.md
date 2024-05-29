<!--yml
category: 未分类
date: 2024-05-27 15:01:43
-->

# An alternate pattern-matching conditional for Elisp [LWN.net]

> 来源：[https://lwn.net/Articles/961682/](https://lwn.net/Articles/961682/)

| **LWN.net needs you!**Without subscribers, LWN would simply not exist. Please consider [signing up for a subscription](/subscribe/) and helping to keep LWN publishing |

By **Jake Edge**
March 1, 2024

One of the outcomes of the (extremely) lengthy discussion about using Common Lisp features in Emacs Lisp (Elisp), which we [looked at](/Articles/951090/) back in November, was an effort to start removing some of those uses from Emacs. The rewrite of some of the Elisp in Emacs that uses the Common Lisp library (cl-lib) was [started by Richard Stallman](/ml/emacs-devel/E1qvq5s-0003LC-O6@fencepost.gnu.org/) as a way to reduce the cognitive load needed for maintaining Emacs itself. Since then, he has broadened his efforts to simplify Elisp by adding a new [pattern-matching conditional](https://www.gnu.org/software/emacs/manual/html_node/elisp/Pattern_002dMatching-Conditional.html) that would be a competitor to [`pcase`](https://www.gnu.org/software/emacs/manual/html_node/elisp/pcase-Macro.html), which is a longstanding macro that he finds overly complex.

#### Complexity

Back in mid-November, Stallman [noted](/ml/emacs-devel/E1r3SgH-0007Rb-BU@fencepost.gnu.org/) that he found the "<q>little language</q>" that `pcase` defines to be "<q>so concise it is downright cryptic</q>". He recognizes that trying to solve the same set of problems combining simpler Elisp constructs, such as [`cond`](https://www.gnu.org/software/emacs/manual/html_node/elisp/Conditionals.html#index-cond) and [`let`](https://www.gnu.org/software/emacs/manual/html_node/eintr/let.html), is "<q>long-winded and cumbersome</q>", but `pcase` has taken the desire for conciseness to an undesirable extreme. That imposes a cost on all Emacs developers who have to maintain code using `pcase`, he said, so he decided to adapt some `pcase` features in other constructs.

Predictably, that led to a long thread—standard fare for the emacs-devel mailing list—discussing whether there is a need for a `pcase` alternative, what one might look like, and more. Stallman [started a new sub-thread](/ml/emacs-devel/E1r4Bci-00043P-VJ@fencepost.gnu.org/) to investigate his ideas for a new macro, `cond*`, which would provide a simpler pattern-matching construct that is still more concise than using "<q>old-fashioned Lisp</q>". His new macro is meant to combine the conditional `cond` form, which handles checking for multiple different values—something like [switch constructs](https://en.wikipedia.org/wiki/Switch_statement) in other languages—with `let`, which temporarily binds values to variables within a limited scope. `pcase` and `cond*` are both designed to provide an [ML-style pattern-matching conditional](https://en.wikibooks.org/wiki/Standard_ML_Programming/Expressions#Case_expressions_and_pattern-matching) mechanism for Elisp.

A simple `pcase` example may give the general flavor of these constructs, but there is a great deal more that both can do, including pattern matching and pulling lists and other data structures apart, which is known as "destructuring". This example, taken from the `pcase` documentation, handles several different types (e.g. string, symbol) for a return code, producing an appropriate message for each:

```
    (pcase (get-return-code x)
      ;; string
      ((and (pred stringp) msg)
       (message "%s" msg))

      ;; symbol
      ('success       (message "Done!"))
      ('would-block   (message "Sorry, can't do it now"))
      ('read-only     (message "The schmilblick is read-only"))
      ('access-denied (message "You do not have the needed rights"))

      ;; default
      (code           (message "Unknown return code %S" code)))

```

Stallman characterized his approach as trying to "<q>avoid the kludginess of pcase's bells-and-whistles-for-everything approach</q>". Naturally, that led to further arguments in favor of `pcase`. For example, Michael Heerdegen [said](/ml/emacs-devel/878r6u3s7f.fsf@web.de/): "<q>In my opinion `pcase' comes very close to the optimal solution for its task.</q>"

By mid-December, Stallman was [asking](/ml/emacs-devel/E1rF4wk-0006QG-G6@fencepost.gnu.org/) about features needed for `cond*` "<q>so that it is rare to encounter a pcase that can't be replaced cleanly</q>". That conversation took place in [another branch](/ml/emacs-devel/E1rF4wi-0006Q3-Ne@fencepost.gnu.org/) of the original `pcase`-replacement discussion, which makes it a little hard(er) to follow. In that part of the thread, others were working with Stallman on `cond*`; once again, there were numerous helpful suggestions and clarification queries, amidst a few grumbles. Stallman was clearly making progress on the feature, however.

In mid-November, Alan Mackenzie had [raised an issue](/ml/emacs-devel/ZVp_3zG-W5EnYfRT@ACM/) that has seemingly lingered with `pcase` since it was [added](https://git.savannah.gnu.org/cgit/emacs.git/commit/?id=d02c9bcd096c44b4e3d5e2834c75967b56cdecdd) in 2010: documentation. He pointed to a [post](https://lists.gnu.org/archive/html/emacs-devel/2015-12/msg00741.html) he had made in 2015 that described the problems that existed in the `pcase` documentation; many of those were addressed at the time, but there is an [ongoing effort](/ml/emacs-devel/19a64b7a-cec0-ed8a-d413-096451cc7413@gmail.com/), led by Jim Porter, to improve the documentation for the macro.

Porter listed multiple areas that need attention, including moving the presentation of the backquote ("```") operator, which is used for pattern-matching and destructuring, earlier in the `pcase` [doc string](https://www.emacswiki.org/emacs/DocString). As Mackenzie had noted, one of the confusing things about `pcase` is its use of two punctuation marks, backquote and comma ("`,`"), that already have established uses in Elisp macro definitions: "<q>pcase complicated the meaning of ` and ,. Before pcase these had definite meanings. Afterwards, they became highly context dependent.</q>"

There were a few responses to Porter's message, largely in favor of his ideas; eventually, Emacs co-maintainer Stefan Kangas [copied the post](/ml/emacs-devel/CADwFkm=aZB29-Jc_WGBWj+SdwrQJ1KRCqvJUPgaqR=JpWDLVGA@mail.gmail.com/) to Stefan Monnier, who is a former Emacs maintainer and the developer of `pcase`. Monnier was [generally in favor](/ml/emacs-devel/jwvbkaejfzz.fsf-monnier+emacs@gnu.org/) as well; he thought that the doc string was not really the place for detailed backquote information, though some reorganization made sense. Stallman [took exception](/ml/emacs-devel/E1rILvo-0006PN-Ka@fencepost.gnu.org/) to Porter's other suggestion to possibly mention `pcase` in "[An Introduction to Programming in Emacs Lisp](https://www.gnu.org/software/emacs/manual/html_node/eintr/index.html)". "<q>We should not encourage people learning Emacs Lisp to use pcase.</q>"

#### First draft

In mid-January, Stallman [posted a first draft of `cond*`](/ml/emacs-devel/E1rQJDv-0001UG-2E@fencepost.gnu.org/), asking for more testing, "<q>constructive comments, bug reports, patches, and suggestions</q>". Andrea Corallo [asked](/ml/emacs-devel/yp1ttnave7z.fsf@fencepost.gnu.org/) about one particular feature of `cond*`:

> what is the reason for some of these cond* clauses to keep the binding in effect outside the clause itself and for the whole cond* construct? At first glance it doesn't look very idiomatic in Lisp terms to me.

Corallo is referring to the `bind*` sub-clause that can appear anywhere in the body of the `cond*` to bind variables with a scope that does not end with the `bind*` that contains them. Instead, the scope of those variables is the body of the `cond*` from that point onward, which is a bit of an oddity from the usual expectation in Lisp.

```
    (cond*
         (CONDITION FORM)
         ((bind* (x 42)))  ; create a binding of 42 to x
         (CONDITION-using-x FORM-using-x)
         ...)

```

As João Távora [noted](/ml/emacs-devel/CALDnm51H-3vPJLEm40YbX5K559O7mRTxeXg5hG0no73y2Wu4gw@mail.gmail.com/), others had already asked about that behavior, but he is "<q>not sure we eventually clarified it</q>". He also wondered if `cond*` could be built using `pcase`; if it is a strict subset of the features of `pcase`, it might help to do so. It "<q>could actually facilitate the adoption path for 'cond*' (as questionable as that path may still be, at least for some parties)</q>". But Stallman [does not see things that way](/ml/emacs-devel/E1rRO8o-0001R8-Mx@fencepost.gnu.org/); he listed the advantages he sees with `cond*` and said that he plans to add the new macro to Emacs:

> cond* has four basic advances over pcase: making bindings that cover the rest of the body, matching patterns against various data objects (not forcibly the same one), use of ordinary Lisp expressions as conditions in clauses, and the [ability] to make bindings and continue with further clauses.
> 
> I'm going to do some more testing and then install cond*.

Adam Porter [asked](/ml/emacs-devel/86a97153-94eb-4df4-b135-4127d2a00057@alphapapa.net/) Stallman to reconsider installing `cond*` into Emacs proper, suggesting that he should consider enhancing `pcase` rather than add a whole new facility that developers will need to learn. One of the complaints about `pcase` is that it is a burden to learn; "<q>How will that burden be helped by having to learn both Pcase and cond*?</q>" (Adam) Porter pointed out that `pcase` could handle many of the advances Stallman had listed, so there may be a path to enhance `pcase`:

> Your stated reasons for writing cond* were various shortcomings of Pcase. Some of those, e.g. the documentation, have already had volunteers step up to address. The others could also be addressed in various ways. I've suggested a few, but you haven't explained the reasons for rejecting them.

As might be guessed, Stallman [did not agree](/ml/emacs-devel/E1rSH01-00025m-B1@fencepost.gnu.org/):

> If pcase lacked features for certain specific jobs, it would be [easy] to fix that by adding a few features. However, the problem with pcase is that it has too many features for the job it does. cond* does the same jobs with fewer features because they work together better.

#### ELPA?

Kangas [said](/ml/emacs-devel/CADwFkmmCYZqGrF5KtSED2RHJuqR-nbo0CDL9gSrHFVBD7zUGfQ@mail.gmail.com/) that he was not closely following the discussion, but was surprised to hear that "<q>there was a plan to install `cond*', or I would have spoken up sooner</q>". Adding `cond*` will necessarily make the job of maintaining Emacs harder, since `pcase` is not going away, thus there will be "<q>not one, but two relatively complex macros</q>" that he will have to know and understand. That might be worth doing if `cond*` offered substantial benefits, but he does not see that; "<q>What I see instead is a mere _version_ of `pcase'.</q>" So, he recommended making a new package for the [GNU Emacs Lisp Package Archive](https://elpa.gnu.org/) (ELPA), which will provide "<q>a good way of exploring an alternative version of an existing macro</q>".

The lack of a plan to wholesale replace `pcase` with `cond*` was seen as a good thing by Kangas. That avoids a bunch of code churn and bugs that would naturally result. But Mackenzie [disagreed](/ml/emacs-devel/ZbD_WHup2fSmtYwi@ACM/); he thinks that fully replacing `pcase` would be a good goal for "<q>improving the readability and maintainability of our code at a stroke</q>". He thinks that putting it into an ELPA library is "<q>a way of ensuring it never comes to anything and just gets forgotten about</q>"; a feature branch with an eventual merge into the Emacs mainline would be a better course.

Like several others, Monnier [believes](/ml/emacs-devel/jwvv87kvtey.fsf-monnier+emacs@gnu.org/) that `cond*` is simply further complicating Elisp. On the other hand, he said, it would be a bit hypocritical for him to oppose it:

> So, I'm not super enthusiastic about adding such a new form, but being responsible for the introduction of several such new constructs in ELisp over the years, I do feel a bit like the pot calling the kettle black.
> 
> So, rather than oppose it, I'll just point out some things which I think could be improved (or which offend my taste, as the case may be).

What followed was an extended back-and-forth between Stallman and Monnier about `cond*`, its overlaps with `pcase` (and how the two could perhaps "meet in the middle"), some deficiencies in the regular `cond` form with respect to binding inside of the conditional, and more. Much of the discussion revolved around Monnier's concern that the two constructs had overlapping, but still different, pattern languages:

> [...] I see a fairly extensive but hardcoded pattern language, which seems like a regression compared to Pcase where the pattern language is built from a few primitives only (with the rest defined on top via `pcase-defmacro`). The worst part, tho, is that the two pattern languages are very similar, and I can't see any good reason for the differences.

Monnier [asserted](/ml/emacs-devel/jwvttmt8x38.fsf-monnier+emacs@gnu.org/) that Stallman could just reuse the low-level pattern language of `pcase` for `cond*`, which would have a number of benefits: "<q>Less work, less code duplication, less documentation duplication, less to learn for coders. And presumably you'd then improve Pcase, so everyone wins.</q>"

The discussion continues as of this writing, though it is unclear which direction Stallman will take with the pattern language. That language, which is implemented by the `match*` form that would be added for `cond*`, could easily be changed to use the `pcase` machinery, Monnier [said](/ml/emacs-devel/jwvcysksgl3.fsf-monnier+emacs@gnu.org/); a patch to do so has [already been sent](/ml/emacs-devel/jwvmsroqxhc.fsf-monnier+emacs@gnu.org/). Mackenzie [objected](/ml/emacs-devel/Zdt3TM4AzmcDSrVq@ACM/) to `cond*` using the `pcase` pattern handling, in part because of a lack of documentation of the low-level `pcase` machinery. Alfred M. Szmidt [hoped](/ml/emacs-devel/E1reIZh-0000Z1-56@fencepost.gnu.org/) that eventually `pcase` could be written to use `cond*`, instead of the reverse, but Monnier [said](/ml/emacs-devel/jwvbk84qvii.fsf-monnier+emacs@gnu.org/) that would not be technically feasible.

#### Adding `cond*`

At the end of January, Emacs co-maintainers Eli Zaretskii and Kangas [announced](/ml/emacs-devel/CADwFkm=p5QWY4Quy1ihrd8u3FX+NALis+OAOnGG0RzWPMihbsQ@mail.gmail.com/) that `cond*` would be installed in the Emacs core as an alternative to `pcase`; there would be no effort to switch away from `pcase`, however, as it is "<q>to be considered a matter of stylistic preference</q>". In the post, Kangas makes it clear that political, rather than strictly technical, considerations were part of the decision-making process:

> Our responsibility as maintainers is first and foremost to ensure that we can all work together, and unite under a common banner. Our success as a project depends on it. Thus, the last thing we want to do is to alienate any group of contributors, big or small.
> 
> We believe that this is a more important concern than the arguments for or against cond* or pcase. The simple fact is that we have different backgrounds and experiences, which have tended to land us on either side of this discussion. This diversity is a strength, and not a weakness.

Even that has not satisfied everyone as there are apparently still some scars from `pcase` being installed in Emacs 14 years ago, at least [for Mackenzie](/ml/emacs-devel/ZbZ5m1AO5Ltqglvc@ACM/). He believes that the middle path chosen by the maintainers will not help resolve the problems that he and others have in understanding some `pcase` uses; he would still advocate a wholesale replacement using `cond*`. Stallman, on the other hand, [does not think that `pcase` should be replaced](/ml/emacs-devel/E1rUfF3-00068E-Jd@fencepost.gnu.org/), but does "<q>hope to discourage its use inside Emacs in favor of cond*</q>". And, of course, the decision did not stop yet another [discussion of "`cond*` versus `pcase`"](/ml/emacs-devel/DU2PR02MB10109F1CBA5F7A8D78A597A5D96472@DU2PR02MB10109.eurprd02.prod.outlook.com/) from breaking out and, naturally, continuing at length.

At the end of January, Stallman [said](/ml/emacs-devel/E1rVO51-00077C-A0@fencepost.gnu.org/) that he was close to ready to commit the code to the [Emacs Git repository](/ml/emacs-devel/E1rVO51-00077C-A0@fencepost.gnu.org/), though that has not happened as of this writing. Zaretskii [asked](/ml/emacs-devel/86cytg1vcu.fsf@gnu.org/) that documentation be added to the [Elisp reference manual](https://www.gnu.org/software/emacs/manual/html_node/elisp/index.html) at the same time, so that may have slowed the process some. Before long, though, `cond*` should make an appearance in Emacs, so there will be two options for conditionals with pattern-matching and destructuring capabilities—for good or ill.

* * *

(

[Log in](https://lwn.net/Login/?target=/Articles/961682/)

to post comments)