<!--yml
category: 未分类
date: 2024-05-27 14:40:43
-->

# Wiki - ElixirNitpicks

> 来源：[https://wiki.alopex.li/ElixirNitpicks](https://wiki.alopex.li/ElixirNitpicks)

So in mid 2023 I wrote a [small but nontrivial web app](https://linkrot.cc/) with Elixir, which also served as [an exercise in learning the language](ElixirForCynicalCurmudgeons). This is my list of gotchas and weirdnesses that I ran into with the language. This is primarily comparing vs. Rust since that’s what I consider my tool of choice these days, and this is all using Elixir 1.14.

Last updated in April 2024.

# Error handling

Elixir’s error handling is basically the combination of two different worlds: exception-style “raised” errors, and Rust-style returned tuples of `{:ok, val}` or `{:error, e}`. The problem is where these things intersect.

Most functions that can fail return the tuple `{:ok, val}` or `{:error, thing}`. Some things instead `raise` an exception when they fail, and most functions that return an error tuple have a a version that throws an error with the same function name plus a `!`. This is handy in interactive use so you don’t have to constantly pattern match to get the result from running a function, which gets tedious. So like, `DB.update` returns `{:ok, val}` or `{:error, e}`, and `DB.update!` return `val` or raises `e` as an exception. So far so good I guess. You can always write `try do thing!() rescue e -> {:error, e} end` to convert a raised error into a returned one, and you can always do `{:ok, val} = thing()` as a somewhat quick-and-dirty way to raise a pattern-match error if an error occurs.

The problem is sequencing and chaining these things, which Rust just has far better idioms for. Though Elixir’s `with` statement does a *pretty* decent job, it doesn’t nest well and causes issues with nesting other blocks inside it, like if expressions. But because Elixir has no early returns, you literally can’t write a macro like Rust’s `?` or `try!()` which is a single expression that bails early. (Exercise for the reader: prove me wrong.) There’s also the issue with using Elixir’s `with` function when you chain multiple steps together that all return error values, but their error values all look similar and you want to know which one actually failed in your error handling. [https://dev.to/martinthenth/using-elixirs-with-statement-5e36](https://dev.to/martinthenth/using-elixirs-with-statement-5e36) has a fairly clever solution in the section titled “Differentiating non-matching clauses”, where you have something like this:

```
with true <- is_email_address?(email),
 true <- EmailAddresses.is_available(email) do
 ...
else
 false ->
 # Return either '{:error, :bad_request} or '{:error, :conflict}'
end
```

and handle it by instead writing something like this:

```
with {:is_email, true} <- {:is_email, is_email_address?(email)},
 {:is_available, true} <- {:is_available, EmailAddresses.is_available(email)} do
 ...
else
 {:is_email, false} ->
 {:error, :bad_request}

 {:is_available, false} ->
 {:error, :conflict}
end
```

This turns calls two functions in series that each return a `bool` indicating success or error, and turns them into two different return types that you can distinguish in a pattern match. That’s honestly not terrible, but feels wonky and fragile. Not certain I’d consider it anywhere near as good as:

```
is_email_address(email).map_err(bad_request)?;
is_available(email).map_err(conflict)?;
```

It just feels like a lot of ceremony for something that should be common.

Also a function might return `{:ok, thing}`, or if it has no value to return then it might just return `:ok`. You just have to know if you need to do `:ok = thing()` or `{:ok, _val} = thing()` ’cause if you do the wrong one it’s an error. So it’s a little tricky to make general-purpose error combinator code in Elixir because it has to check for both of these possibilities. And if you have to return `{:ok, thing1, thing2}`? Fuggedaboudit; no error combinator around will try too hard to deal with every possible tuple length. Just return `{:ok, {thing1, thing2}}` and move on with life, which makes more sense anyway when you think about it but also isn’t as obvious.

Another error nitpick I’ve found, “Let it fail” doesn’t work when validating user input where you need to present a good error message back to the user. Sometimes the validation can’t happen at the edge too, so you have some deep call-chains of code that have to explicitly handle errors, and some other adjacent deep call-chains that are fine just logging something and dying.. (Hmmm, it would be much easier if validation did always happen at the edge, that may be part of my problem…)

# State management

Just about any global state change in an Elixir program can and will be hidden behind a function call. There’s no way to really express that this is the case, unlike Rust where you almost always have to pass `&mut something` into functions. For a purely immutable functional language, Elixir programs sure have *lots* of global mutable state in them, too.

The common idiom in general is to hide your mutable state in a process, and then instead of sending messages directly to it you write wrapper functions to do it for you. A great example is a database connection. In Rust you tend to have something like this:

```
let db = Db.open("...")?;
db.query("stuff");
db.insert("other stuff");
stash_into_mutable_global(db);

// in some other function
let local_db_reference = get_db_from_mutable_global()
local_db_reference.query("...")
// local_db goes out of scope
```

In Elixir you have:

```
DB.open("...")
DB.query("stuff")
DB.insert("other stuff")

// in some other function
DB.query("...")
```

There’s a global process registry, you can attach arbitrary names to them, and libraries use this a *lot*. So even if you personally don’t write it, you end up with a *ton* of global mutable state. It’s very weird and a little unsettling. Sure the state is all encapsulated into processes, but then those processes are hidden behind an abstraction layer that makes them invisible, so really you’re just touching global variables.

# Imports

There’s just too many heckin’ ways to import a module. You can:

*   Not import it, and refer to its contents through fully-qualified names, `Foo.Util.frobnosticate()`
*   `import` it, which stuffs all its symbols into the local namespace. So doing `import Foo` lets you call `Util.frobnosticate()`, while `import Foo.Util` lets you call `frobnosticate()`
*   `alias` it, which lets you rename some module path to some other, presumably shorter, module path. So doing `alias Foo.Util, as: U` lets you call `U.frobnosticate()`.
*   `require` it, which lets you call macros in it, which can’t normally be called. So you can’t normally call `Foo.Util.frob_macro()` until after you do `require Foo.Util`, after which you can call it as `Foo.Util.frob_macro()`. Though apparently if you do `import Foo.Util` you also can call `frob_macro()` directly, so there’s overlapping behavior there that I apparently don’t fully understand.
*   `use` it, which which calls a callback in it which may be a macro that generates code. So if you do `use Foo.Util` it is the equivalent of calling `require Foo.Util; Foo.Util.__using__()` and splicing whatever code it returns into the current module.

Contrast Rust, which makes `use` do all these things:

*   No imports result in no imports, and like Elixir you can refer to a thing through fully-qualified names.
*   `import Foo` is done with `use foo;`
*   `import Foo.Util` is done with `use foo::util::*;`
*   `alias Foo.Util, as: U` is done with `use foo::util as u;`
*   The more common `alias Foo.Util, as: Util` is just `use foo::util;`
*   `require Foo.Util` doesn’t need to exist because since Rust 2018??? or so, macro namespaces are treated basically the same as function namespaces with regards to imports. Rust visually distinguishes its macros with a trailing `!` so there’s no risk of calling something and having it sneakily turn out to be a macro that does something unexpected.
*   `use Foo.Util` is the only one that no real equivalent; the closest thing you have is a procedural macro… which is treated the same way by the language as any other macro.

`use` is weird enough that it’s pretty easy to distinguish from the others, but the exact differences between `import`, `alias` and `require` are a *constant* headache to try to remember. You could just have one thing do all of these:

*   `import Foo` -> you can call `Util.whatever()`
*   `import Foo.Util` -> you can call `whatever()`
*   `import Foo.Util, as: U` -> you can call `U.whatever()`
*   `import Foo.Util, macros: true` -> you can call `whatever_macro()` (can combine with `all:` or `as:` keywords).
*   Leave `use` the way it is, it’s an odd enough special case it deserves to be different.

I tried to write a macro to implement something like this, but I’m not actually interested enough in the problem to spend too much time on it. Plus, to actually make this macro exist, you have to start each module with `require CoolMacro`! Argh! The requirement for `require` is honestly pretty difficult to paper over without changing how the language works, I think.

I considered suggesting making some explicit macro import unnecessary, which I still think is a good idea, but Elixir is way more macro-heavy than Rust so there’s probably good reasons for it being the way it is. Still, Elixir macros are fairly… leaky, which gets pretty frustrating. `Integer.is_odd()` is a macro, for example, so if you try to call it directly the compiler will helpfully inform you to call `require Integer` first. I appreciate that, but generally if the compiler is smart enough to tell you the exact solution to your problem, it should be smart enough to just do it for you. Why do they make `is_odd()` a macro in the first place, you might ask? Because that way it can be used in match guards, which functions cannot be since match guards are a limited subset of the language that does not have side-effects. That all makes sense, but is surprising as *hell*. We’re in a four-deep chain of non-obvious causes and effects involving the dangers of arbitrary code generation, when what you *really* want is either a way to write pure functions that can be used as pattern guards, or a way to visually distinguish functions from macros a la Rust’s sigils so that it’s more obvious what is going on.

# Mixed messages

The common lore I’ve seen disbursed to newbies in the Elixir Discord chat includes:

*   Don’t use umbrella projects if you can avoid it
*   Don’t use live upgrade of code if you can avoid it – ie prefer to upgrade by restarting nodes, rather than hot-reloading new code while migrating state
*   Don’t use macros if you can avoid it
*   The Mnesia database is more of a Redis replacement than a PostgreSQL replacement. This is more about Erlang’s messaging thing than Elixir’s, since Mnesia is distributed with the Erlang runtime, but still relevant. I’ve also heard of Mnesia being used as a control-plane/config database rather than a user-data DB, which it is also probably pretty good at.

It would be nice if the Elixir tutorials or FAQ included more of this wisdom, instead of listing them as neat features that it should tell you all about. LOTS of people seem to run `mix help new`, see the single line saying `An --umbrella option can be given to generate an umbrella project`, and go down the rabbit-hole there figuring out how umbrella projects work instead of doing something useful. Even heckin’ *describing* what an umbrella project is and saying “most projects except for the largest don’t need this” would help.

# Unit tests are more of a pain

…though I really have a hard time expressing why.

# Domain Specific Languages

This is more of a problem with libraries than with the core language itself, but those core libraries (Phoenix, Plug, Ecto, etc) tend to be the first things you encounter and you use them a lot. Then other programs people write tend to follow the idioms that those core libraries teach. Basically those libs have lots of new syntax that they build out of macros, which suffers the same problems as macro DSL’s *always* suffer, which is poorly defined semantics.

For example, using Phoenix I have a slightly weird issue with a component that has a couple private subcomponents implemented as functions… Basically I have:

```
defmodule LinkrotWeb.Components.Navbar do
  use Phoenix.Component
  ...

  attr :page_title, :string, required: false, default: ""
  defp navitem(%{title: _, path: _, name: _} = assigns) do: ...
  defp rnavitem(%{title: _, path: _, name: _, label: _, first: _} = assigns) do: ...

  def navbar(assigns) do
    links = ...
    assigns = assign(assigns, :links, links)
    ~H"""
    <ul class="navbar">
      <%= for {name, path} <- @links do %>
        <.navitem title={@title} name={name} path={path}/>
      <% end %>

      <.rnavitem title={@title} path={~p"/users/register"} name="Register" label="Register" first={true} />
      <.rnavitem title={@title} path={~p"/users/log_in"} name="Log in" label="Log in" first={false} />
    </ul>
    """
  end
end
```

and everything works fine, but I get warnings for the navitem function:

```
warning: undefined attribute "name" for component LinkrotWeb.Components.Navbar.navitem/1
  lib/linkrot_web/components/navbar.ex:61: (file)

warning: undefined attribute "path" for component LinkrotWeb.Components.Navbar.navitem/1
  lib/linkrot_web/components/navbar.ex:61: (file)

warning: undefined attribute "title" for component LinkrotWeb.Components.Navbar.navitem/1
  lib/linkrot_web/components/navbar.ex:61: (file)
```

But I don’t get those for the rnavitem function, and everything actually renders properly, so I’m confused about where those are coming from. Turns out the `attr` macro applies to the function right after it, not the module it’s in. On the one hand this makes sense ’cause it’s the same as Elixir’s normal type annotations, but Elixir’s docs also tend to treat the “component” as the module as a whole, not specific functions inside that module. It’s documented, so to some extent my fault for not reading the docs, but it’s also non-obvious to me at the time since Phoenix has lots of other macro DSL’s that *do* just slot into the module enclosing them and treat things magically, like `plug`, `pipeline` and `embed_templates`.

“Let’s just create a declarative DSL out of macros to make life easier, what could go wrong?” It’s not actually declarative and involves implicit assumptions about how the code is structured, that’s what. You COULD write the code like this:

```
 def navbar(assigns) do
    attr(assigns, [
      [:page_title, :string, required: false, default: ""],
      [:current_user, :string, required: false, default: nil],
    ])
    links = ...
```

And it would be *just fine*. And also make clear that `attr` is associated with the function, not the module. I’ve yet to see any use of `attr` that can’t be instead done by normal Dialyzer `@spec` decls either, though it’s possible there are some out there. Maybe the default values? idk, the whole `assigns` system is kinda overly-magical in my opinion anyway. Assigns are there for good reasons but I have issues trying to juggle Elixir, HEEx templates, and the various DSL fragments associated with them all at once.

# Misc

*   Ecto, the de-facto standard database query builder, needs more love. It’s not bad by any stretch of the imagination, but a lot of the rough edges I encountered with docs and API design were in Ecto compared to other projects. `Repo.one` returns `value | nil` while `Repo.insert` returns `{:ok, value} | {:error, changeset}`, the `Repo.transaction` function [could really use some honestly quite basic improvements](https://tomkonidas.com/repo-transact/), I’m really still not sure of the best way to do joins, keeping your migrations and schemas in sync is kinda painful with two separate DSL’s in their own config files, stuff like that. However, all of these are also pretty superficial problems. Operationally, Ecto has been fast, flexible, generates good queries, and as far as I can tell with my limited database experience tends to guide you towards good design patterns. Its query DSL is one of the few times that learning the DSL has been totally worth it and has been a very useful and low-leakage abstraction.
*   Hot code reloading is a little squirrelly at times. Sometimes you’ll save a file and it will get hot-code-reloaded instantly automatically, other times it will take a few seconds, some errors will be fixed when the system automatically reloads the file and some won’t. It’s a bit of a nuisance. Monitoring files for changes in general is harder than it looks, but I do wish it was better.
*   Erlang/Elixir’s vaunted reliability only kind of involves the language itself. Most of the time I don’t feel like it’s particularly easier or harder to write robust, fault-tolerant code in Elixir or Erlang than any other immutable-first functional language like OCaml or Clojure (which is probably its closest cousin, being dynamically typed). Errors happen, and they make things fail in ugly ways, and sometimes you really wish you just had Rust’s `Result` type and `?` operator. What BEAM/OTP gives you is tools to *deal with* the errors when they happen, and the ecosystem uses those things pervasively. For a while I’ve had the hunch that `systemd` and Unix process trees are really just a weaksauce and poorly-implemented version of OTP’s supervisor trees, and now that I’ve used it for real a little that has proven to be spot on. So far anyway.
*   There’s a cultural split between Erlang and Elixir that makes life harder than it needs to be. It really doesn’t need to be there, and on a technical level the languages interoperate so well through BEAM that the split *isn’t* there. For example lots of things are missing from Elixir libs because Erlang libs have them, and calling Erlang code from Elixir (or vice versa) takes essentially zero effort. But Elixir has the `mix` build tool and Erlang has `rebar3`, cross-building packages so you can include both langs in the same project isn’t as painless as it should be, you can’t really invoke the Elixir compiler from Erlang last I checked, etc. Anecdotally, when Elixir started off there was some bad blood between them and the Erlang community, which is the origin of this schism. But nobody seems to hold any grudges anymore and the languages and communities seem to cohabitate well. It’s just annoying having two different, mostly-incompatible build tools when they proceed to build 100% compatible artifacts.
*   For some weird reason the Elixir Discord community has a distinct lack of programmer-socks-wearing queer furries, at least compared to Rust, or even most other tech-y Discord servers I’ve seen. It caused some weird cognitive dissonance. Why do I feel vaguely strange hanging out online with all these kind, knowledgeable, friendly and compassionate techbro’s? Then I see a name I recognized from elsewhere and my hindbrain goes “oh thank gods, I know for a fact she’s actually a snow leopard in her free time”. Okay, this nitpick is **firmly** tongue-in-cheek, but the Rust user-base continues to be a fascinating case study in how many weirdos you can get together in one place when you *very explicitly* say it’s ok to be a weirdo. This isn’t a complaint; nice techbro’s need their place too, but it’s an interesting contrast. In the last Rustbelt Rust I went to pre-COVID, the most common answer to “what other languages are you interested in” was “Elixir”, by far, so I feel like Elixir heavily overlaps with Rust in time and cognitive-space, so it’s a weird cultural split. Nobody I’ve met in the Elixir community is even remotely unfriendly or exclusive, but the vibe is distinctly more corporate-professional, so I guess we weirdos aren’t nearly as comfortable letting the masks down. Maybe it’s because Elixir targets itself more towards fairly mainstream web-tech, while most other things I’m interested in (programming languages, graphics, robotics, gamedev) are rather more niche?

# Positive notes

Things that surprised me in good ways:

*   So, Elixir is dynamically typed everywhere. There’s a program called Dialyzer that will check optional type annotations for you, but sometimes it feels a little like a second-class citizen, at least compared to the amount of work that gets put into Rust’s type system and error messages. But somewhat to my surprise, in practice it ends up working *really well*. Dialyzer will catch lots of type errors for you if you let it, and almost all the libraries I’ve used have complete and correct type annotations for everything. Dialyzer isn’t foolproof, and if given incomplete information can fail in fairly mysterious ways, but on the whole it’s very much worth using from day one. (Erlang/Elixir+Dialyzer is a fascinating point of reference to contrast with other gradually-typed languages like Typescript or Python, I could go on about it far too long.)
*   Similarly, Elixir is very much dynamically *bound*. If I understand correctly, most function calls are semantically treated as call-by-symbol-name, same as running `apply(Modulename, :function_name, [args])`. Then the compiler inlines and optimizes it down to more direct calls if the names are always known at compile-time. This results in some things that feel very weird to a Rustacean, like calls to functions or modules that don’t exist being a compiler *warning* rather than an error. In fact the Elixir compiler almost *never* gives you an outright error, basically it only fails if a file can’t be parsed. This feels spooky as hell… but its warnings are basically always correct and seldom miss anything, so in reality you just treat Elixir’s warnings the way you’d treat `rustc`’s errors and everything is fine.
*   When talking about reliable code and system-building in Elixir/Erlang, the first thing any doc starts telling you about is supervision trees. But lots of the time, you mostly don’t need to touch them. Every library that starts a long-running process has its own supervision tree, and Phoenix starts your app in its own supervision tree, and there’s one hook you write to connect them together. Almost no configuration or tweaking is actually needed, you just let them do their thing, and let it stay that way is until you scale up enough that you really need to start growing your own sub-tree of processes for parallelism or error-isolation. With Phoenix at least, your program can get pretty big before you actually need to consider splitting it apart into different processes, because a lot of that is already done.

# Conclusions

Elixir is pretty darn cool. You should use it.

Phoenix scourges my soul a little bit but its heart is in the right place, and it works well in practice. It just suffers from Web-Framework-Itis where it tries to make simple things too simple and ends up making complicated things too complicated.

Ecto is an extremely nice query-builder that makes lots of the PITA parts of SQL actually less painful without introducing more sharp edges. I do wish migrations were just generated from schemas though, a la Django.

There’s lots of other great tools in the ecosystem; Oban, Mix, Telemetry/Logger, and most of the Erlang stdlib are all on the “10/10, would use again” list. Lots of the more niche ones are a little short of polish but do the basics very well.

Lots of other tools are a bit short of the “first-class” level of polish; Image/Vix, ExAws, and other more minor things are entirely usable but don’t get their entire API surface banged on as much or as hard, so if you’re getting started in an unfamiliar space or doing something odd they can take a little more wrestling than maybe they should to do what you want them to.

I avoided Phoenix LiveView with a passion, but I probably shouldn’t; it seems like a very interesting take on a lot of the same same principles that go into [HTMX](https://htmx.org/). It just triggers my buzzword-avoidance and Shiny Webtech Phobia real damn good. Maybe someday I’ll write “liveview for cynical curmudgeons”. Not sure that’s a brand I should especially pursue though…

I need to write a video game using Elixir. Someday!

# References

Minor things and conversations that follow similar trends.