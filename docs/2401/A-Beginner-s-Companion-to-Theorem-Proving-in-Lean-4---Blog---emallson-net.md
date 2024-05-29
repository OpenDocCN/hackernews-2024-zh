<!--yml
category: 未分类
date: 2024-05-27 14:33:25
-->

# A Beginner's Companion to Theorem Proving in Lean 4 | Blog | emallson.net

> 来源：[https://emallson.net/blog/a-beginners-companion-to-theorem-proving-in-lean/](https://emallson.net/blog/a-beginners-companion-to-theorem-proving-in-lean/)

This year, one of my hobby projects has been implementing some basic properties and conversions between [Matroids](https://en.wikipedia.org/wiki/Matroid) in [Lean](https://lean-lang.org/).

My primary resource for doing this has been [*Theorem Proving in Lean 4*](https://lean-lang.org/theorem_proving_in_lean4/title_page.html) (TPiL), which is an incredibly detailed walk through using Lean as a proof assistant. While useful, it has...gaps—as does the main Lean documentation. A lot of this gap appears to be [filled by the Zulip chat](https://leanprover-community.github.io/archive/).

The title of this post is a bit of a double entendre: I am still a relative beginner, writing what I intend as a companion to TPiL; and this is is intended for beginners to Lean, covering that I wish I had known as a beginner myself.

A final caveat emptor: I haven't really hung out in Zulip and don't have a good grasp on good code organization and style in Lean. These may not be best practices, but they helped me.

* * *

This is organized in small sections covering individual things that I learned the hard way while working on my hobby project.

 <details><summary>Running Snippets</summary> 

You can run the code snippets below in Lean or on the [Lean website](https://live.lean-lang.org/) if you paste this prelude at the top:

```
import Mathlib.Data.Finset.Basic import Mathlib.Data.Finset.Card 
```</details> 

```
example (S T : Finset α) : S ⊆ T -> T ⊆ S -> T = S := by 
 exact? 
```

One of the biggest pain points early in my Lean experience was *finding* theorems to use. The [`mathlib`](https://leanprover-community.github.io/mathlib4_docs/foundational_types.html) documentation is great, but lacking [hoogle](https://hoogle.haskell.org/)-style search.

EDIT: Apparently not! `LeahNeukirchen` points out on [lobste.rs](https://lobste.rs/s/hwjv0f/beginner_s_companion_theorem_proving#c_7rypt5) two different hoogle-like search engines that I missed:

The `rw?`, `simp?`, `apply?` and `exact?` tactics give you specialized hoogle-like access within the Lean environment, finding candidate theorems that satisfy a specific goal and listing them in the editor UI. You can pick one of the solutions via code actions if using the LSP.

By default, all target the current goal, but `rw?` and `simp?` can have `at` forms:

```
example (S T : Finset α) : S ⊆ T -> T ⊆ S -> T = S := by 
 intro S_subs T_subs rw? at S_subs 
```

TPiL explains what each of these tactics do in technical terms, but it took time for me to grok what each do in practice.

It is important to understand that these tactics are all fundamentally performing *pattern rewriting*. The patterns for each are:

**Input Goal:** `a`
**Assumption:** `a = b` or `a ↔ b`
**Output Goal:** `b`

`rw` is the most direct of the 3\. It applies a theorem or assumption that looks like `a = b` or `a <-> b` to a goal that looks like `a`, substituting it with `b`.

`rw`, unlike `apply` and `exact`, can be applied to non-goals to transform them. This is very useful to massage theorems from different sources into alignment to prove your goal.

```
example (S T X : Finset α) : S = X -> X = T -> S = T := by
 intro S_eq X_eq rw [S_eq, <-X_eq] 
```

**Input Goal:** `a`
**Assumption:** `a`
**Output Goal:** *No Goal*

`exact` takes an assumption, along with any needed parameters, and uses it to resolve the current goal. It is simplistic, but effective.

```
example (S T : Finset α) : S ⊆ T -> T ⊆ S -> S = T := by
 intro S_subs T_subs  /- Finset.Subset.antisymm looks like: S ⊆ T -> T ⊆ S -> S = T -/
  /- the S_subs and T_subs parameters leave us with S = T, which matches the goal. -/
 exact Finset.Subset.antisymm S_subs T_subs 
```

**Input Goal:** `b`
**Assumption:** `a → b`
**Output Goal:** `a`

`apply` transforms your goal into a different goal using an implication. If a proof of `a` exists in the current scope, it is applied (in which case you basically get `exact`). Otherwise, you get a new goal `a`.

```
example (S T : Finset α) : S ⊆ T ∧ T ⊆ S := by
  /- Finset.Subset.antisymm_iff.mp looks like S = T -> S ⊆ T ∧ T ⊆ S -/
 apply Finset.Subset.antisymm_iff.mp  /- new goal: S = T -/ 
```

Importantly (so importantly I'll bring it up again later): an `a ↔ b` or `a = b` assumption can be converted to `a → b` (with `.mp`) or `b → a` (with `.mpr`).

TPiL mentions sub-goals at several points and does briefly discuss `have`, but I feel this area was glossed over. Adding sub-goals toward your larger goal is an extremely helpful method of making larger proofs tractable.

The first (and seemingly most common) way to introduce sub-goals is by way of `have`, which generally has two purposes. First: you can introduce smaller goals that are easier to prove—sometimes even possible for Lean to prove automatically.

```
/- While lean cannot auto-solve the outer statement, it can trivially solve the both steps with `exact?` -/ example [DecidableEq α] (S : Finset α) (e1 e2 : α) : e2 ∈ S → insert e2 (insert e1 S) = (insert e1 S) := by
 intro e2_mem  have : e2 ∈ (insert e1 S) := by exact? exact? 
```

Second: you can use it to instantiate other theorems for re-use:

```
/- (not runnable) -/ have T_card := ind_rank_eq_largest_ind_subset_card ind (insert e2 S) T T_subs T_ind T_max 
```

In contrast, `if` / `else` is generally useful in cases where you need to introduce a goal that relies on the [law of the excluded middle](https://lean-lang.org/theorem_proving_in_lean4/axioms_and_computation.html?highlight=propext#the-law-of-the-excluded-middle).

```
/- there is a theorem in Mathlib for this, but we use `have` to create smaller goals to bypass it -/ example [DecidableEq α] (S : Finset α) (e : α) : (insert e S).card ≤ S.card + 1 := by
  if h : e ∈ S then
  have : insert e S = S := Finset.insert_eq_self.mpr h rw [this]
 exact Nat.le_add_right (Finset.card S) 1
  else
  have : S.card + 1 = (insert e S).card := (Finset.card_insert_of_not_mem h).symm rw [this] 
```

Note that this, along with helpful theorems like `not_not` rely on classical logic and as such are not strictly constructive (as far as my knowledge indicates), but `if`/`else` in particular is such a useful means of organizing my thoughts in the implementation of a proof that I opted to use it quite heavily.

EDIT: It appears that Lean will only allow non-constructive uses of `if` / `else` if you have used `open Classical` in the module. [More info](https://lobste.rs/s/hwjv0f/beginner_s_companion_theorem_proving#c_jc4wbc)

TLiP discusses handling cases with dots and `cases _ with` and `match _ with`, but it wasn't clear to me how to handle the (labelled) sub-goals generated other tools (like `apply Iff.intro`).

You can use `case <label> => <proof>` to handle a specific named proof.

This couples well with `refine`, which is like `exact` but generates sub-goals for (named or anonymous) placeholders:

```
example (S T : Finset α) : S ⊆ T -> T ⊆ S -> S = T := by
 intros refine Finset.Subset.antisymm ?left ?right case left => assumption case right => assumption 
```

As noted above, you can transform inferential rules with dot accessors:

Beyond this, many operators expose helpful utilities. Using `a > b` as an example:

*   `le` produces `b < a`
*   `not_le` produces `¬ a <= b`
*   `asymm` produces `¬ a < b`
*   `trans_le` produces `a <= c -> b < c`

and many more. These are super handy for cases where (for example) you've got your hands on a proof of `a > b` and need to prove `¬ a <= b`, which happens often with contraposition.

These tactics are all negation-related. Much like `rw` / `apply` / `exact`, they do different yet similar things.

**Input State:** an assumption `h`, for which you can prove `¬ h` from other assumptions
**Output State:** the goal is replaced with `¬ h`

This is useful in cases where you have constructed a contradiction in your assumptions for a proof by contradiction, but the contradiction is not obvious to Lean.

```
example (a b : ℕ) : a > b -> b > a -> True := by
 intro left right absurd right exact left.asymm 
```

**Input State:** a pair of assumptions `h` and `h'` which *blatantly* contradict
**Output State:** *No Goal*

`contradiction` cuts out the extra steps of `absurd` in cases where the contradiction is obvious to Lean. Typically, this means that you have hypothesis `h : a` and `h' : ¬ a`.

```
example (a b : ℕ) : a > b -> ¬ a > b -> True := by
 intros contradiction 
```

**Input State:** an assumption `h`
**Output State:** the goal is replaced with `¬ h`, and `h` is replaced with `¬ goal`

Unlike `absurd` / `contradiction`, this is *actually* doing something different: applying [contraposition](https://en.wikipedia.org/wiki/Contraposition). I'm including it mostly because the tactic wasn't mentioned in TLiP and I often find that `contrapose` is useful to simplify goals that involve negation in combination with `rw [not_not]`.

```
/- a theorem for this exists in Mathlib, but again we're ignoring it -/ example (S : Finset α) : ¬ S.card = 0 -> S ≠ ∅ := by
 intro neq_zero contrapose neq_zero rw [not_ne_iff] at neq_zero rw [not_not, neq_zero] rfl 
```

In a conventional language with algebraic data types, there are broadly two groups of types:

In a language like Rust, these are `struct`s or tuples:

```
struct Foo {
  a: usize;
 b: usize; }   struct Bar(usize, usize); 
```

The `And` type in Lean, while usually written `A ∧ B`, is actually a structure that looks something like:

```
structure And where
  left : α
 right : β 
```

Since this is a structure, that means you can destructure it in an assumption:

```
have : a ∧ b := sorry  have 〈 a, b 〉 := this 
```

You can deal with it in a goal in two ways:

*   Instantiating the structure: `exact 〈 a, b 〉` (closes the goal)
*   Converting to sub-goals: `apply And.intro` (creates new goals for proving the left and right sides)

In a language like Rust, these are `enum`s:

```
enum Foo {
 A(usize),
 B(usize) } 
```

The `Or` type in Lean is a sum type. While written `A ∨ B`, it would actually look something like this:

```
inductive Or where | inl : α | inr : β 
```

As a sum type, you deal with it by *pattern matching* in assumptions:

```
have : a ∨ b := sorry  cases this with | inl a => sorry  | inr b => sorry 
```

However, in goals you generally do one of two things:

*   Instantiate one side of the structure: `exact Or.inr b` (which closes the goal). This pairs well with `have` to create a sub-goal for one side of the `Or`.
*   Convert to an implication: for example `rw [or_iff_not_imp_left]` (which convers `a ∨ b` to `¬ a -> b`, and given a `¬a` assumption changes the goal to `b`)

The single thing that gave me the most trouble in my hobby project was the proving of theorems involving *existential quantifiers.* It is important to understand the constructive elements of Lean to fully understand the design of existentials, and I think that TPiL does a pretty good job of explaining that. However, that explanation doesn't really tell you how to prove existentials in more complicated settings.

First off: existentials have the type `Exist α β`. This is a product type, much like `And`, and can be destructured in a similar way:

```
have : ∃ x : N, x < 4 := sorry  have 〈x, x_lt〉 := this /- note that `x` is actually an N that exists satisfying x_lt! -/ 
```

This also means that your options for *proving* an existential are much like an `And`. Principally: construct it with `exact 〈x, h〉`, `x` is some value that satisfies `h`.

The question then: for complex types, how do you get an `x`? The answer: get it from a method. `bex` seems like the common name for it. For example, to obtain an element of a finite set `S`:

```
example (S : Finset α) : S ≠ ∅ -> ∃ x : α, x ∈ S := by
 intro h  /- before getting an element from S, we have to construct the Nonempty type -/
  have : S.Nonempty := Finset.nonempty_of_ne_empty h exact this.bex 
```

Once you're used to this, it starts becoming fairly natural. Retrieving elements from collections generally requires proving that they are nonempty (otherwise there may be no element to retrieve!), but that this was a totally separate type (`Finset.Nonempty` in my case) was opaque and difficult for me to discover.

Above, I discussed `have` as a means of introducing sub-goals to solve. Alongside it exists a syntactically-similar tool: `let`. Specifically:

```
have <name> : <assumption> := <proof of assumption> 
```

may be restated as:

```
have <name> : <type> := <construction> 
```

which is then (almost!) equivalent to:

```
let <name> : <type> := <construction> 
```

There is one key difference: `let` definitions are *transparent*, while `have` definitions are *opaque*. In other words: if you want the rewriter to be able to operate on the construction of a term, use `let`. Otherwise, `have`.

To see the difference, consider this example:

```
example [DecidableEq α] (a b : Finset α) (e : α) : ¬ e ∈ a -> insert e a = b -> b.card = a.card + 1 := by
 intro mem h  let x := insert e a  have : x.card = a.card + 1 := by
 rw [Finset.card_insert_of_not_mem mem] rw [← this, ← h] 
```

If `let x` is replaced by `have x`, then the inner sub-goal is no longer provable—the definition of `x` has become opaque.

There is one other difference: which fields are optional. `have` requires *only* the `construction` / `proof` field, while `let` requires at least the `name` and `construction` fields (the type may be inferred).

My final note—and the only one on code organization—is to use small theorems. If you example much of mathlib, you will find small proofs: a handful of lines, at most. This is obviously helpful when you are familiar with the library internals, but opaque as a reader on how exactly the proof is constructed.

Don't let that discourage you from focusing on small proofs, though! There are two *strong* reasons beyond vague "code architecture" considerations for this:

1.  Small proofs can often be constructed automatically by Lean through `exact?`, `rw?` or `simp?` (if not more advanced tools like [`aesop`](https://github.com/JLimperg/aesop)), which means less work for you.
2.  The theorem lookup tactics start to perform noticeable worse with large proof states, which makes the interactive experience worse.

This is not *too* much of a problem—my proof of submodularity of Matroid rank functions derived from independence predicates is around 100 lines long and remained at least *usable*—but certainly an element of UX that coincides with good practice: small theorems, like small functions, are generally easier to work with and this aligns with the practical performance of Lean's interactive systems.

Overall, working through these proofs in Lean was a very instructive experience for me. Being a relative beginner to proof assistants in general (having only light experience toying with Emacs' [Proof General](https://proofgeneral.github.io/) + [Coq](https://coq.inria.fr/) during grad school), the quality of the LSP and other tools were huge boons. I wish this had existed while I was working on theory stuff more actively, as it did help me find issues with on-paper proofs that I'd done which held hidden assumptions.

That said: it definitely was not a replacement for working on paper for me. The degree to which Lean encourages small transformations caused me to miss the forest for the trees in a very direct way. It is useful in verification of theory, but not a replacement for sitting down with pen, paper, and tea and working through a proof.

Hopefully these notes will be useful for others looking to use Lean as a proof assistant (and, perhaps, people who are less committed to constructive theory and are more than willing to embrace practical niceties like `if`/`else` and `not_not`).