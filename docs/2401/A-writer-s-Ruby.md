<!--yml
category: 未分类
date: 2024-05-27 14:31:32
-->

# A writer's Ruby

> 来源：[https://world.hey.com/dhh/a-writer-s-ruby-2050b634](https://world.hey.com/dhh/a-writer-s-ruby-2050b634)

Programmers at large seem eternally skeptical of style. And I’m not just talking about the stereotype of nerds in uncoordinated outfits or using pocket protectors. But style in the broad sense of aesthetics. Many appear imbued with fundamental opposition to the idea that how something looks should even matter. That somehow such a focus is conflict with where it ought to be: on substance. This is very apparent in the discussion about linting code.

Linting is a term for applying a consistent style to source code. Like following

[Strunk & White’s grammatical guidelines](https://www.amazon.com/Elements-Style-Fourth-William-Strunk/dp/020530902X)

for prose, but automatically enforced by a linting program. It’s a technique that’s been around for a long time, but one that’s seen broader adoption with the advent of cloud-based continuous integration setups, which run tests, and now linting, on every pull request prepared.

Some languages, like Go, have

[a built-in linter](https://go.dev/blog/gofmt)

, which applies a universal style that’s been set by the language designers. That’s the most totalitarian approach. A decree on what a program should look like that’s barely two feet removed from a compile error. I don’t like that one bit.

It reminds me of

[Newspeak](https://bookanalysis.com/1984/newspeak/)

, the new INGSOC language from Orwell’s 1984\. Not because of any sinister political undertones, but in the pursuit of a minimalist language, with no redundant terms or ambiguities or flair. Imagine every novel written in the same style, Hemingway indistinguishable from Dickens, Tolkien from Rowling. It would be awfully gray to enjoy the English language if there was only a single shade of prose.

I can already hear the objection: Code isn’t prose! Its compile target is the computer, not the imagination! Oh do I dare to differ. The best code to me is indeed its own form of poetry, and style is an integral part of such expression. And in no language more so than Ruby.

Whereas Python’s motto has long been “there should be one – and preferably only one – obvious way to do it”, Ruby is the exact opposite. There are many ways to do almost everything, and it’s why reading and writing Ruby is such a delight to people like me who

[strive to write code as an author](https://www.youtube.com/watch?v=9LfmrkyP81M)

more often than as an engineer.

Let me give you the most basic example:

```
def ensure_privileged_access
  if !person.admin?
    redirect_to root_url,
  end
end
```

I’ve always found that “if not” expression to be clumsy, even though it’s a mainstay of most programming languages. But Ruby also allows you to write like this:

```
def ensure_privileged_access
  redirect_to root_url unless person.admin?
end
```

Now I don’t know if I’d call that poetry, but I do find it far more pleasant to read. And that’s just a single sentence. Compose a few of those into a paragraph (method), then into an essay (class), and soon you’ll have a whole book (system). It matters how that's written.

There are probably people who would prefer the more conventional, literal style, and they could encode that preference in a lint rule. I’d have absolutely no problem with that, as long as they’re not trying to force me to abide by their stylistic preferences.

Which gets to the crux of the question with linters. I think they’re excellent tools for authors and teams to enforce

*their*

style with consistency on

*their*

codebases. That’s why we’re shipping a default linter, cleverly called RuboCop, in the next version of Rails. Not such that I, or anyone, can mandate what style your codebase ought to be written in, but such that you can find and enforce your own.

Not everyone has yet developed a personal or team style, though, and for them, we’re we’re including

[the preferences I’ve developed](https://github.com/rails/rubocop-rails-omakase/)

during the last two decades, writing hundreds of thousands of lines of Ruby code with the team at 37signals. That’s the omakase menu. But it really is just a starting point for Rails programmers and teams to develop their own sensibilities, not a permanent edict on what those should be.

That to me is the essence of style, it’s invariably idiosyncratic. We wouldn’t even be talking about style if it was all the same for everyone all the time. Ruby affords us such rich and expressive syntax that it would be a damn shame if we narrowed all possible paths down to a single, pre-approved trail. Ruby is for writers.