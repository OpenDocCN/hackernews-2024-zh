<!--yml
category: 未分类
date: 2024-05-27 14:53:41
-->

# Lambda Functions

> 来源：[https://www.lambdafunctions.com/articles/elixir-and-rust](https://www.lambdafunctions.com/articles/elixir-and-rust)

# Managing mutable data in Elixir with Rust

One of Elixir’s core benefits, and the secret to its robustness and scalability, is its foundation on immutable data. Sometimes, though, immutability is just not a great fit for a particular task—but that task is only one part of a large project. Is it possible to enjoy the benefits of Elixir’s data model everywhere else, but carve out a little mutable exception for one area?

Yes!

A long-running Elixir project I’m involved in has just this problem. The project is delivered over the web, so moving away from Elixir as a whole is not on the cards because [Phoenix](https://www.phoenixframework.org/) is quite simply a cheat code for web development. We *could* hive off the mutable section into a microservice, but that would require significant architectural and management overhead. All we really need is a little escape hatch for a limited chunk of code, while still being within the same VM and able to interact normally with the rest of the service.

This is just what [Rustler](https://github.com/rusterlium/rustler) offers.

Rustler is “a library for writing Erlang NIFs in safe Rust code”—in other words, you can write code that looks like standard Elixir functions, but behind the scenes is actually implemented in Rust.

[NIFs](https://www.erlang.org/docs/17/tutorial/nif) have been a feature of the Erlang VM since long before either Elixir or Rust arrived on the scene. What Rust and Rustler add is:

*   safety—this is critical since a crash in a NIF will bring down the whole VM

*   a lot of polish and interface glue that makes it feasible to write more ambitious integrations that you might be inclined to attempt with C and Erlang’s standard NIF support

*   access to all of Rust’s libraries

Unfortunately, most of the Rustler examples on the web focus on the speed benefits and show the implementation of a trivial `add` function and then stop there. While that’s fine for demonstrating the bare minimum integration required, for me the interesting part of Rustler is the chance to escape in a controlled way from the immutable world—I want to explore how to manage a little mutable chunk of memory in a safe way. Although Rustler is certainly capable of this, there’s very little available in the way of tutorials or examples.

Hopefully this article will help.

## Goal

As mentioned above, I want to explore memory management. More specifically, I want to be able to hold a chunk of data in the Rust world that persists between multiple calls to different “Elixir” (Rustler) functions. These functions should allow the Elixir world to pass data into the Rust world, mutate the data held there, and then retrieve results.

To give us something substantial to play with and avoid having to implement our own data store for this demo, I’ll use Oxigraph.

[Oxigraph](https://github.com/oxigraph) is a Rust graph database library implementing the SPARQL standard. Let’s suppose that we want to wrap it, so that we can have access to a fast graph database from within Elixir. We’ll call our wrapper `FeGraph`.

We want to be able to:

1.  Make a new in-memory database

    ```
    db = FeGraph.new()
    ```

2.  Add data to it

    ```
    FeGraph.set(db, "http://foo.bar.com")
    FeGraph.set(db, "http://foo.baz.com")
    ```

3.  Export the database as a [Turtle](https://en.wikipedia.org/wiki/Turtle_(syntax)) string:

    ```
    FeGraph.dump_db(db) |> IO.puts()
    ```

To keep this to a reasonable length, we’re not going to implement everything that would be required to expose all the capabilities of Oxigraph; just enough to demonstrate holding data on the Rust side and acting on it from Elixir.

## Implementation

(If you want to follow along, I recommend working through one of the many Rustler `add` tutorials I mentioned before getting into the code, so that you have a basic project up and running and have worked through how Rust and Elixir functions link together. Everything below here assumes you’re already at that point.)

The key to this whole approach is the ability to pass a [Resource](https://erlang.org/doc/man/erl_nif.html#resource_objects) between the two worlds. This acts as a handle to a piece of memory; it can be returned from a NIF and then passed back into another call. Exactly what we need.

A BEAM `Resource` is represented in Rustler by a `rustler::resource::ResourceArc<T>` struct. To get started with our implementation, let’s define a new type that we can use as a handle to represent the state of our graph store.

In a production scenario we’re likely to want to manage more state than this, but for now it will suffice to define a `MyGraph` struct that just contains (via a mutex) the Oxigraph data store; this represents the mutable data we want to manage outside Elixir. In the future, more fields could be added to `MyGraph` as necessary.

To turn this into something that can be passed back and forth between Elixir and Rust, we need to wrap it in a `ResourceArc`. In order to make our function signatures a bit more readable we’ll define a new type of `GraphArc` to represent a `MyGraph` struct in a `ResourceArc`.

In `lib.rs`:

```
use std::sync::Mutex;
use oxigraph::store::Store;
use rustler::resource::ResourceArc;
use rustler::OwnedBinary;
use rustler::{Env, Term};

struct MyGraph { store: Mutex<Store> }

type GraphArc = ResourceArc<MyGraph>;
```

A bit of additional plumbing is required to tell Rustler that a `MyGraph` is something that can be used as a `Resource`:

```
fn on_load(env: Env, _info: Term) -> bool {
    rustler::resource!(MyGraph, env);
    true
}

rustler::init!("Elixir.FeGraph", [], load = on_load);
```

With these definitions in place, we can write a `new` function that allocates a new data store and returns a handle to it:

```
#[rustler::nif]
fn new() -> GraphArc {
    ResourceArc::new(
        MyGraph {
            store: Mutex::new(Store::new().unwrap()),
        }
    )
}
```

And on the Elixir side, in `fe_graph.ex`:

```
defmodule FeGraph do
  use Rustler, otp_app: :myapp, crate: "fegraph"

  def new(), do: :erlang.nif_error(:nif_not_loaded)
end
```

At this point we can test in `iex`, and see that the Elixir stub above has been replaced by the Rust NIF we defined, which we can run and which gives us back a reference:

```
iex(1)> FeGraph.new
#Reference<0.2659174309.2607677441.115195>
```

Granted we can’t yet *do* anything with it, but we’re already defining a data store in Rust and seeing evidence of it in Elixir; and behind the scenes the BEAM and Rustler are taking care of all of the heavy lifting for us.

How about a simple function to add some data to our new store? In a way that will feel very familiar to Elixir code, it will need to both take and return a `GraphArc` handle. We’ll also have it accept a single string to use for all three parts of the triple to store (normally of course we’d take different strings for the subject, predicate, and object parts of the triple, but our focus here isn’t on SPARQL—we just want some data to store.)

```
#[rustler::nif]
fn set(state: GraphArc, iri: &str) -> GraphArc {
    let store = state.store.lock().unwrap();

    let ex = NamedNode::new(iri).unwrap();
    let quad =
        Quad::new(ex.clone(), ex.clone(), ex.clone(), GraphName::DefaultGraph);
    (*store).insert(&quad).unwrap();

    drop(store);

    state
}
```

Within the function we can use our `GraphArc` state argument to get a hold of the Oxigraph store that we created back in the `new` function. Once we’ve got it we can add some test data to the graph as normal, then return the unchanged `state`.

The final piece of the puzzle is to retrieve some data from our store. Rather than running a query (which would require getting into more SPARQL) we’ll just dump the whole database and return it as a string. As before, our new function will need to accept a `GraphArc`, but this time we’ll return an `OwnedBinary`, which allows us to send a binary back to the BEAM and then wash our hands of it.

```
#[rustler::nif]
fn dump_db(state: GraphArc) -> OwnedBinary {
    let store = state.store.lock().unwrap();

    let mut buffer = Vec::new();
    (*store)
        .dump_graph(
            &mut buffer,
            GraphFormat::Turtle,
            GraphNameRef::DefaultGraph,
        )
        .unwrap();

    let mut result = OwnedBinary::new(buffer.len()).unwrap();
    result.as_mut_slice().copy_from_slice(&buffer);

    result
}
```

The majority of this function turns out to be messing around getting the data out of Oxigraph into a buffer, and then from the buffer into the `OwnedBinary`; the Rustler wrapper has become mostly invisible which is what I was originally hoping for.

With this in place we can now demonstrate allocating some memory in Rust, returning a handle to that memory, then using it to store data outside the BEAM memory model and finally fetch the data back into the Elixir world:

```
iex(1)> db = FeGraph.new
#Reference<0.2749498138.3684302852.140448>
iex(2)> FeGraph.set(db, "http://foo.com")
#Reference<0.2749498138.3684302852.140448>
iex(3)> FeGraph.dump_db(db)
"<http://foo.com> <http://foo.com> <http://foo.com> .\n"
```

Note the important part; the reference is the same both times despite the data changing and we are *not* storing it after the `set` call; normally we’d need to do something like `db = FeGraph.set(db, "http://foo.com")` instead. The only reason `set` returns the reference is for convenient use with the pipe operator or similar.

## Conclusion

While the code shown above does skip past most of the error handling, hopefully it’s clear just how accessible Rustler makes it to link Rust code into Elixir projects in a way that allows you to combine the strengths of both.

Rustler is a tremendous addition to the Elixir ecosystem, and it opens up far more opportunities than just calculating things more quickly. Being able to opt out of the standard BEAM memory model for specific sections of code can open the doors to custom data stores and other features that would not generally be a good fit for Elixir, while still allowing you to use the power of Phoenix for the majority of your application… all with virtually seamless integration.