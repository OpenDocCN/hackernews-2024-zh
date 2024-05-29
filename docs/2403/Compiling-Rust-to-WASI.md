<!--yml
category: 未分类
date: 2024-05-29 12:31:51
-->

# Compiling Rust to WASI

> 来源：[https://benw.is/posts/compiling-rust-to-wasi](https://benw.is/posts/compiling-rust-to-wasi)

Recently I've been taking a bit of a break from building and interacting with Rust compiled to the Webassembly target `wasm32-unknown-unknown`, and instead spending [a lot of time](https://github.com/fermyon/leptos-spin) working with Rust compiled to [WASI](https://wasi.dev/), or the `wasm32_wasi` target.

You'd be forgiven if you're not sure of the distinction between these two, they're both webassembly right? The answer is yes, but they're different in several crucial ways. I like to think of the `wasm32-unknown-unknown` as analogous to `no_std`. It has no access to anything outside of the webassembly virtual machine. None of the standard system calls for file access, network requests, or the kernel will be available in the traditional way. This is the target that is supported natively by the browser, and is used by Leptos for client side interactivity.

`wasm32-wasi` is `wasm32-unknown-unkown` plus the missing traditional stuff. It reimplements the traditional system calls like `read()` and provide access to things C expects like `File*` objects. It also fixes a crucial ABI incompatibility with `wasm32-unknown-unknown`, due to an [error in the ABI](https://github.com/rust-lang/rust/issues/83788) used by `wasm-bindgen` that has become a Rust standard. If you want to hear more about that, read on, otherwise you can skip to the next section. I spent far too much time figuring this out, so y'all get to hear about it too.

Way back when, when webassembly support in Rust was young, the Rust developer working on it made a mistake involving how Rust handles structs in wasm-bindgen. LLVM and Clang say passing a struct by value should inline the fields, while for Rust it should be passed by pointer. This becomes a problem when you have Rust code that interacts with C and does this. Whether compiled together with Rust or as a seperate C webassembly module compiled to emscripten, this issue rears it's ugly head.

> This is also why Rust code compiled to `wasm32_wasi` and `wasm32-unknown-emscripten` can have difficulties talking to each other.

At some point this issue was fixed for Rust's C ABI, but fixing it for the `wasm32-unknown-unknown` target would break compatibility with Rust apps compiled to webassembly with wasm-bindgen under the old behavior. Which leaves us in a bit of a bind. wasm-bindgen now has [*forward* compatibility](https://github.com/rustwasm/wasm-bindgen/pull/3595) with the possible new ABI, meaning it no longer has an issue, but how to introduce that in the Rust ecosystem has been a matter of debate. Should you create a new target like `wasm32-unknown-unknown2`? Switch it one day? Switch it across an edition boundary?

For now at least, Rust has decided to eventually incorporate an unstable flag that fixes the ABI by switching to the existing C ABI. The [PR](https://github.com/rust-lang/rust/pull/117919) for that is here, and the MCP for it has been approved, so it's likely to get into the language sometime in the future. So that's fun news. Avoiding passing any structs across the ABI boundary is another option, but requires editing all the parties involved. Personally I have no appetite for editing all C dependencies I might want to use, but you do you.

As a fun fact, the `wasm32-wasi` and `wasm32-unknown-emscripten` targets have fixed this incompatibility. I suppose the first is new enough and the second is unknown/purpose built enough that it was deemed appropriate to do so. For `wasm32-wasi`, here was [the fix](https://github.com/rust-lang/rust/pull/79998).

One of the sellings points of Webassembly and WASI is that things compiled to it are supposed to be portable, but the ABI spec for each language is left up to the languages themselves. Perhaps because C is the largest/oldest language, we seem to be coalescing arounding [whatever the C/Clang/llvm folks](https://github.com/WebAssembly/tool-conventions/blob/main/BasicCABI.md) decide to do. This can have [other implications](https://faultlore.com/blah/c-isnt-a-language/), as this lovely article describes for existing systems. At any rate, the Rust folks haven't totally commited to this, with a "wasm" ABI option for Rust compiled to Webassembly in [the works](https://github.com/rust-lang/lang-team/blob/master/design-meeting-minutes/2021-04-21-wasm-abi.md), and I have no idea what any other languages

Now that you all have learned so much about webassembly and webassembly in Rust(or skipped straight here), let's talk about building a crate. I started out on this journey because I wanted a better editor for blog posts on my blog. For that I use [femark](https://crates.io/crates/femark), a crate that I wrote combining treesitter and pulldown-cmark to compile markdown to HTML with syntax highlighted code blocks. Only trouble with this is that treesitter(and its parsers) are written in C/C++.

Historically I've been converting to html on the server, where the crate can be compiled by Rust quite easily. This time I realized that to get the live preview experience I desired, I'd have to run it in the browser. And I'm happy to say, after much finagling, I was able to get there with the help of Joel Dice and several lovely folks on the WASI Zulip.

1.  Compile Femark to WASI

    You'd think this part would be easy, just run

    bash

    ```
    cargo build --target=wasm32-wasi

    ```

    and it'd spit out a wasm file right? Wrong! Instead you get this

    C code

    ```
      *error* *occurred*: *Command* *"clang"* *"-O3"* *"-ffunction-sections"* *"-fdata-sections"* *"-fPIC"* *"--target=wasm32-wasi"* *"-I"* *"./typescript/src"* *"-Wall"* *"-Wextra"* *"-o"* *"/celCluster/projects/femark/target/wasm32-wasi/release/build/tree-sitter-typescript-db2920a2728a6446/out/./typescript/src/parser.o"* *"-c"* *"./typescript/src/parser.c"* *with* *args* *"clang"* *did* *not* *execute* *successfully* 

    ```

Turns out the Rust compiler does not know how to handle the C/C++ in the parsers for this target. So I went and downloaded the [wasi-sdk](https://github.com/WebAssembly/wasi-sdk) and install the released binaries to `/opt/wasi-sdk-21.0`. And then set some RUSTFLAGs and env vars to point to things in it.

bash

```
RUSTFLAGS='-L /opt/wasi-sdk-21.0/share/wasi-sysroot/lib/wasm32-wasi -lstatic=c++ -lstatic=c++abi' CXXSTDLIB=c++ CC=/opt/wasi-sdk-21.0/bin/clang CXX=/opt/wasi-sdk-21.0/bin/clang++ CXXFLAGS="-fno-exceptions"  cargo component build --release

```

We use a special version of clang and other c libs they provide to make wasi/C interop easier. Messing with C/C++ compilers is definitely not my forte, so I was very thankful to have help figuring that out. Thankfully this builds.

2.  Convert a WASI preview 1 file to a WASI preview 2 file

If you've been following the development of WASI, you'd know that fairly recently we saw the release of [WASI preview 2](https://component-model.bytecodealliance.org/). WASI preview 2 provides a component model for WASI components, allowing them to talk to each other. Currently, the `wasm32-wasi` target for Rust aims at WASI preview 1\. We want preview 2 to get WASI to run in the browser with the newly released [jco](https://bytecodealliance.org/articles/jco-1.0) transpiler. The linked article mentions allowing WASI preview 2 components to run in node.js, but it also has a bit more experimental support for running them in the browser.

The easiest way to convert a wasi preview 1 module to a wasi preview 2 component is to use [cargo-component](https://github.com/bytecodealliance/cargo-component), which basically replaces the command above. You do still need all the RUSTFLAGS and other env vars up there. If you don't use `cargo component new` to handle this for you, I found it works by adding this to your Cargo.toml. The key here is that the part after the colon matches your world defined in the wit file.

TOML markup

```
*[**package**.**metadata**.**component**]*
*package* *=* *"benwis:femark"*
*[**package**.**metadata**.**component**.**dependencies**]*

```

and then setting up the wit bindings. WASI2 defines functions and types that are exported and imported from other components using the [WIT language](https://component-model.bytecodealliance.org/design/wit.html). Cargo component will take those bindings, and then match them to Rust functions defined in a `Guest` trait implementation. For this crate, we only want to export two Rust functions:

wit

```
// wit/femark.wit
package benwis:femark;

world femark {

  record owned-code-block {
  language: option<string>,
  source: string,
  }
  record owned-frontmatter {
        title: option<string>,
        code-block: option<owned-code-block>
  }
  record html-output{
        toc: option<string>,
        content: string,
        frontmatter: option<owned-frontmatter>
    }

  /// The set of errors which may be raised by the function
  variant highlight-error {
		nolang,
		nohighlighter,
		could-not-build-highlighter(string),
        string-generation-error(string)
  }

  export process-markdown-to-html: func(input: string) -> result<html-output, highlight-error>; 
  export process-markdown-to-html-with-frontmatter: func(input: string) -> result<html-output, highlight-error>; 
}

```

I won't get too far into the WIT language here, but it's fairly Rust like, and I found it functional and fairly easy to pick up. The part that threw me while writing this is that while the functions are mapped to Rust functions, the types defined here will be automatically converted into Rust types. So existing versions of `HighlightError`, `HtmlOutput` and more in Rust needed to be deleted and replaced with generated versions from the autogenerated bindings.rs file.

To map the functions, we define a `Component` struct and implement the `Guest` trait for it in `src/lib.rs`.

Rust code

```
*// Define a custom type and implement the generated `Guest` trait for it which*
*// represents implementing all the necessary exported interfaces for this*
*// component.*
*struct* *Component**;*

*impl* *Guest* *for* *Component* *{*
    *fn* *process_markdown_to_html**(**input**:* *String**)* -> std*::*result*::**Result**<**HtmlOutput**,* *HighlightError**>* *{*
        *process_markdown_to_html**(**&*input*)*
    *}*
    *fn* *process_markdown_to_html_with_frontmatter**(*
        *input**:* *String**,*
    *)* -> std*::*result*::**Result**<**HtmlOutput**,* *HighlightError**>* *{*
        *process_markdown_to_html_with_frontmatter**(**&*input*,* *true**)*
    *}*
*}*

```

With all that done, we can run `cargo component build` and get a WASI preview 2 component `.wasm` file with these exported functions and types built in.

> Remember all that text I wrote about ABI incompatibility earlier? Well, WASI preview 2 defines a [Canonical ABI](https://github.com/WebAssembly/component-model/blob/main/design/mvp/CanonicalABI.md) so that all the languages compiled to WASI can talk to eother WASI components. However, how these types and functions are repesented by their respective languages in the webassembly itself is up to each language. So we'd still have issues with ABI mismatches compiling Rust and C into the same WASI component, but if they were compiled into seperate ones, there'd be a common interface language.

3.  Transpile it for the web with jco

    [jco](https://github.com/bytecodealliance/jco) can transpile the `.wasm` file we generated above into a JS ESM module that node.js and the browser can understand. Using it is actually pretty easy. Before we transpile it, jco has a command to see the imports and exports of a WASI2 module.

    bash

    ```
    jco wit target/wasm32-wasi/release/femark.wasm

    ```

    The output for my module is seen below.

    wit

    ```
    package root:component;

    world root {
      import wasi:cli/environment@0.2.0;
      import wasi:cli/exit@0.2.0;
      import wasi:io/error@0.2.0;
      import wasi:io/streams@0.2.0;
      import wasi:cli/stdin@0.2.0;
      import wasi:cli/stdout@0.2.0;
      import wasi:cli/stderr@0.2.0;
      import wasi:cli/terminal-input@0.2.0;
      import wasi:cli/terminal-output@0.2.0;
      import wasi:cli/terminal-stdin@0.2.0;
      import wasi:cli/terminal-stdout@0.2.0;
      import wasi:cli/terminal-stderr@0.2.0;
      import wasi:clocks/monotonic-clock@0.2.0;
      import wasi:clocks/wall-clock@0.2.0;
      import wasi:filesystem/types@0.2.0;
      import wasi:filesystem/preopens@0.2.0;
      import wasi:random/random@0.2.0;

      record owned-code-block {
        language: option<string>,
        source: string,
      }

      record owned-frontmatter {
        title: option<string>,
        code-block: option<owned-code-block>,
      }

      record html-output {
        toc: option<string>,
        content: string,
        frontmatter: option<owned-frontmatter>,
      }

      variant highlight-error {
        nolang,
        nohighlighter,
        could-not-build-highlighter(string),
        string-generation-error(string),
      }

      export process-markdown-to-html: func(input: string) -> result<html-output, highlight-error>;
      export process-markdown-to-html-with-frontmatter: func(input: string) -> result<html-output, highlight-error>;
    }

    ```

    Looks very similar to our wit file right? The only new stuff here we can see is all of the wasi function imports automatically added by wasi. Pretty neat huh?

    bash

    ```
    jco transpile --minify femark.wasm -o femark-wasi

    ```

    So now it'll get transpiled into a folder containing a few wasm files and some js glue

    bash

    ```
    ❯ tree femark-wasi
    dist/femark-wasi
    ├── femark.core2.wasm
    ├── femark.core.wasm
    ├── femark.d.ts
    ├── femark.js
    └── interfaces
        ├── wasi-cli-environment.d.ts
        ├── wasi-cli-exit.d.ts
        ├── wasi-cli-stderr.d.ts
        ├── wasi-cli-stdin.d.ts
        ├── wasi-cli-stdout.d.ts
        ├── wasi-cli-terminal-input.d.ts
        ├── wasi-cli-terminal-output.d.ts
        ├── wasi-cli-terminal-stderr.d.ts
        ├── wasi-cli-terminal-stdin.d.ts
        ├── wasi-cli-terminal-stdout.d.ts
        ├── wasi-clocks-monotonic-clock.d.ts
        ├── wasi-clocks-wall-clock.d.ts
        ├── wasi-filesystem-preopens.d.ts
        ├── wasi-filesystem-types.d.ts
        ├── wasi-io-error.d.ts
        ├── wasi-io-streams.d.ts
        └── wasi-random-random.d.ts

    ```

    Let's take a peek in the JS file and see what we see. Probably without the --minify to make it a bit easier to read. It's quite long, so I'll link to it here

    JavaScript code

    ```
    *import* *{* *environment**,* *exit* *as* *exit$1**,* *stderr**,* *stdin**,* *stdout**,* *terminalInput**,* *terminalOutput**,* *terminalStderr**,* *terminalStdin**,* *terminalStdout* *}* *from* *'@bytecodealliance/preview2-shim/cli'**;*
    *import* *{* *monotonicClock**,* *wallClock* *}* *from* *'@bytecodealliance/preview2-shim/clocks'**;*
    *import* *{* *preopens**,* *types* *}* *from* *'@bytecodealliance/preview2-shim/filesystem'**;*
    *import* *{* *error**,* *streams* *}* *from* *'@bytecodealliance/preview2-shim/io'**;*
    *import* *{* *random* *}* *from* *'@bytecodealliance/preview2-shim/random'**;*
    ...
    *function* *processMarkdownToHtml**(**arg0**)* *{*
      *var* *ptr0* *=* *utf8Encode**(**arg0**,* *realloc1**,* *memory0**)**;*
      *var* *len0* *=* *utf8EncodedLen**;*
      *const* *ret* *=* *exports1**[**'process-markdown-to-html'**]**(**ptr0**,* *len0**)**;*
      *let* *variant14**;*
      *switch* *(**dataView**(**memory0**)**.**getUint8**(**ret* *+* 0*,* *true**)**)* *{*
        *case* 0: *{*
          *let* *variant2**;*
          *switch* *(**dataView**(**memory0**)**.**getUint8**(**ret* *+* 4*,* *true**)**)* *{*
            *case* 0: *{*
              *variant2* *=* *undefined**;*
              *break**;*
            *}*
            *case* 1: *{*
              *var* *ptr1* *=* *dataView**(**memory0**)**.**getInt32**(**ret* *+* 8*,* *true**)**;*
              *var* *len1* *=* *dataView**(**memory0**)**.**getInt32**(**ret* *+* 12*,* *true**)**;*
              *var* *result1* *=* *utf8Decoder**.**decode**(**new* Uint8Array*(**memory0**.**buffer**,* *ptr1**,* *len1**)**)**;*
              *variant2* *=* *result1**;*
              *break**;*
            *}*
            *default*: *{*
              *throw* *new* TypeError*(**'invalid variant discriminant for option'**)**;*
            *}*
          *}*
          *var* *ptr3* *=* *dataView**(**memory0**)**.**getInt32**(**ret* *+* 16*,* *true**)**;*
          *var* *len3* *=* *dataView**(**memory0**)**.**getInt32**(**ret* *+* 20*,* *true**)**;*
          *var* *result3* *=* *utf8Decoder**.**decode**(**new* Uint8Array*(**memory0**.**buffer**,* *ptr3**,* *len3**)**)**;*
          *let* *variant10**;*
          *switch* *(**dataView**(**memory0**)**.**getUint8**(**ret* *+* 24*,* *true**)**)* *{*
            *case* 0: *{*
              *variant10* *=* *undefined**;*
              *break**;*
            *}*
            *case* 1: *{*
              *let* *variant5**;*
              *switch* *(**dataView**(**memory0**)**.**getUint8**(**ret* *+* 28*,* *true**)**)* *{*
                *case* 0: *{*
                  *variant5* *=* *undefined**;*
                  *break**;*
                *}*
                *case* 1: *{*
                  *var* *ptr4* *=* *dataView**(**memory0**)**.**getInt32**(**ret* *+* 32*,* *true**)**;*
                  *var* *len4* *=* *dataView**(**memory0**)**.**getInt32**(**ret* *+* 36*,* *true**)**;*
                  *var* *result4* *=* *utf8Decoder**.**decode**(**new* Uint8Array*(**memory0**.**buffer**,* *ptr4**,* *len4**)**)**;*
                  *variant5* *=* *result4**;*
                  *break**;*
                *}*
                *default*: *{*
                  *throw* *new* TypeError*(**'invalid variant discriminant for option'**)**;*
                *}*
              *}*
              *let* *variant9**;*
              *switch* *(**dataView**(**memory0**)**.**getUint8**(**ret* *+* 40*,* *true**)**)* *{*
                *case* 0: *{*
                  *variant9* *=* *undefined**;*
                  *break**;*
                *}*
                *case* 1: *{*
                  *let* *variant7**;*
                  *switch* *(**dataView**(**memory0**)**.**getUint8**(**ret* *+* 44*,* *true**)**)* *{*
                    *case* 0: *{*
                      *variant7* *=* *undefined**;*
                      *break**;*
                    *}*
                    *case* 1: *{*
                      *var* *ptr6* *=* *dataView**(**memory0**)**.**getInt32**(**ret* *+* 48*,* *true**)**;*
                      *var* *len6* *=* *dataView**(**memory0**)**.**getInt32**(**ret* *+* 52*,* *true**)**;*
                      *var* *result6* *=* *utf8Decoder**.**decode**(**new* Uint8Array*(**memory0**.**buffer**,* *ptr6**,* *len6**)**)**;*
                      *variant7* *=* *result6**;*
                      *break**;*
                    *}*
                    *default*: *{*
                      *throw* *new* TypeError*(**'invalid variant discriminant for option'**)**;*
                    *}*
                  *}*
                  *var* *ptr8* *=* *dataView**(**memory0**)**.**getInt32**(**ret* *+* 56*,* *true**)**;*
                  *var* *len8* *=* *dataView**(**memory0**)**.**getInt32**(**ret* *+* 60*,* *true**)**;*
                  *var* *result8* *=* *utf8Decoder**.**decode**(**new* Uint8Array*(**memory0**.**buffer**,* *ptr8**,* *len8**)**)**;*
                  *variant9* *=* *{*
                    *language*: *variant7**,*
                    *source*: *result8**,*
                  *}**;*
                  *break**;*
                *}*
                *default*: *{*
                  *throw* *new* TypeError*(**'invalid variant discriminant for option'**)**;*
                *}*
              *}*
              *variant10* *=* *{*
                *title*: *variant5**,*
                *codeBlock*: *variant9**,*
              *}**;*
              *break**;*
            *}*
            *default*: *{*
              *throw* *new* TypeError*(**'invalid variant discriminant for option'**)**;*
            *}*
          *}*
          *variant14**=* *{*
            *tag*: *'ok'**,*
            *val*: *{*
              *toc*: *variant2**,*
              *content*: *result3**,*
              *frontmatter*: *variant10**,*
            *}*
          *}**;*
          *break**;*
        *}*
        *case* 1: *{*
          *let* *variant13**;*
          *switch* *(**dataView**(**memory0**)**.**getUint8**(**ret* *+* 4*,* *true**)**)* *{*
            *case* 0: *{*
              *variant13**=* *{*
                *tag*: *'nolang'**,*
              *}**;*
              *break**;*
            *}*
            *case* 1: *{*
              *variant13**=* *{*
                *tag*: *'nohighlighter'**,*
              *}**;*
              *break**;*
            *}*
            *case* 2: *{*
              *var* *ptr11* *=* *dataView**(**memory0**)**.**getInt32**(**ret* *+* 8*,* *true**)**;*
              *var* *len11* *=* *dataView**(**memory0**)**.**getInt32**(**ret* *+* 12*,* *true**)**;*
              *var* *result11* *=* *utf8Decoder**.**decode**(**new* Uint8Array*(**memory0**.**buffer**,* *ptr11**,* *len11**)**)**;*
              *variant13**=* *{*
                *tag*: *'could-not-build-highlighter'**,*
                *val*: *result11*
              *}**;*
              *break**;*
            *}*
            *case* 3: *{*
              *var* *ptr12* *=* *dataView**(**memory0**)**.**getInt32**(**ret* *+* 8*,* *true**)**;*
              *var* *len12* *=* *dataView**(**memory0**)**.**getInt32**(**ret* *+* 12*,* *true**)**;*
              *var* *result12* *=* *utf8Decoder**.**decode**(**new* Uint8Array*(**memory0**.**buffer**,* *ptr12**,* *len12**)**)**;*
              *variant13**=* *{*
                *tag*: *'string-generation-error'**,*
                *val*: *result12*
              *}**;*
              *break**;*
            *}*
            *default*: *{*
              *throw* *new* TypeError*(**'invalid variant discriminant for HighlightError'**)**;*
            *}*
          *}*
          *variant14**=* *{*
            *tag*: *'err'**,*
            *val*: *variant13*
          *}**;*
          *break**;*
        *}*
        *default*: *{*
          *throw* *new* TypeError*(**'invalid variant discriminant for expected'**)**;*
        *}*
      *}*
      *postReturn0**(**ret**)**;*
      *if* *(**variant14**.**tag* *===* *'err'**)* *{*
        *throw* *new* ComponentError*(**variant14**.**val**)**;*
      *}*
      *return* *variant14**.**val**;*
    *}*
    *.**.**.*
    *export* **{* *processMarkdownToHtml**,* *processMarkdownToHtmlWithFrontmatter**,*  *}** 
    ```

     *It's definitely interesting to see the shenanigans JS has to jump through to match the type system of Wit or Rust. Returning a `result<htmloutput, highlighterro>` creates a ton of switch cases blocks! At the top we can see a number of imports and at the bottom exports of the named functions we're looking for!

    Remember how WASI provides all those interfaces to common system functions? Well those are provided through a `@bytecodealliance/preview2-shim`. package If you're using Node or using a JS bundler, you can pretty easily pull in the [npm package](https://www.npmjs.com/package/@bytecodealliance/preview2-shim?activeTab=versions). Since I'm doing neither of those things, I need to serve that library from my web server, and change all the paths in the js file to point towards a local [copy of it](https://github.com/bytecodealliance/jco/tree/main/packages/preview2-shim).

    JavaScript code

    ```
    *import**{**environment* *as* *e**,**exit* *as* *r**,**stderr* *as* *a**,**stdin* *as* *t**,**stdout* *as* *s**,**terminalInput* *as* *o**,**terminalOutput* *as* *c**,**terminalStderr* *as* *n**,**terminalStdin* *as* *i**,**terminalStdout* *as* *l**}**from**"/components/preview2-shim-browser/cli.js"**;**import**{**monotonicClock* *as* *d**,**wallClock* *as* *b**}**from**"/components/preview2-shim-browser/clocks.js"**;**import**{**preopens* *as* A*,**types* *as* *k**}**from**"/components/preview2-shim-browser/filesystem.js"**;**import**{**error* *as* *f**,**streams* *as* *p**}**from**"/components/preview2-shim-browser/io.js"**;**import**{**random* *as* *u**}**from* *"/components/preview2-shim-browser/random.js"**;*

    ```

    This is from the minified version, but you can see I'm setting absolute paths to the folder served by my server. With this in place, we should be ready to move on!* 
**   Serve all the files

    As previously mentioned, we need to serve the `preview2-shim/lib/browser` folder as well as the femark wasm and js files we generated earlier. On Leptos for SSR apps, this is pretty easy, just put them in your assets dir. The more interesting part is how to use it.

    *   Use wasm_bindgen to generate Rust bindings to the exported J'S functions for Leptos

    wasm_bindgen to the rescue! Here we see the wasm functions, exported to JS, and then passed into extern C blocks as required by wasm_bindgen. Since the return types are structs, I redefine the returned types and create helper functions to parse them from `JsValue`. There's probably a better way to do this, but the types are defined in a crate that won't compile to wasm32_unknown_unknown, and I figured this would be easier than defining a seperate types crate.

    Rust code

    ```
    *use* *crate**::*errors*::*BenwisAppError*;*
    *use* serde*::**{*Deserialize*,* Serialize*}**;*
    *use* wasm_bindgen*::*prelude*::*****;*

    *///Recreate HtmlOutput so we don't have to import it from server only crates*
    *#*[*derive*(*Default, Debug, Clone, Serialize, Deserialize*)**]**
    *pub* *struct* *HtmlOutput* *{*
        *pub* *toc**:* *Option**<**String**>**,*
        *pub* *content**:* *String**,*
        *pub* *frontmatter**:* *Option**<**OwnedFrontmatter**>**,*
    *}*
    */// An owned version of the CodeBlock used for the frontmatter. Makes it much easier to return.*
    *#*[*derive*(*Default, Debug, Clone, Serialize, Deserialize*)**]**
    *pub* *struct* *OwnedCodeBlock* *{*
        *pub* *language**:* *Option**<**String**>**,*
        *pub* *source**:* *String**,*
    *}*

    */// An owned verison of the Frontmatter, returned from our functions*
    *#*[*derive*(*Default, Debug, Clone, Serialize, Deserialize*)**]**
    *pub* *struct* *OwnedFrontmatter* *{*
        *pub* *title**:* *Option**<**String**>**,*
        *pub* *code_block**:* *Option**<**OwnedCodeBlock**>**,*
    *}*

    *#*[*wasm_bindgen*(*raw_module = *"/components/femark-wasi/femark.js"**)**]**
    *extern* *"C"* *{*
        *pub* *fn* *processMarkdownToHtml**(**input**:* *String**)* -> *JsValue**;*

        *pub* *fn* *processMarkdownToHtmlWithFrontmatter**(**input**:* *String**)* -> *JsValue**;*
    *}*

    *pub* *fn* *process_markdown_to_html**(**input**:* *String**)* -> *Result**<**HtmlOutput**,* *BenwisAppError**>* *{*
        *let* jsval = *processMarkdownToHtml**(*input*)**;*
        Ok*(*serde_wasm_bindgen*::**from_value**(*jsval*)*?*)*
    *}*

    *pub* *fn* *process_markdown_to_html_with_frontmatter**(*
        *input**:* *String**,*
    *)* -> *Result**<**HtmlOutput**,* *BenwisAppError**>* *{*
        *let* jsval = *processMarkdownToHtmlWithFrontmatter**(*input*)**;*
        Ok*(*serde_wasm_bindgen*::**from_value**(*jsval*)*?*)*
    *}*

    ```

    *   Use it in a Leptos component

    Finally, the moment of truth! We've compiled our library to WASI preview 2, transpiled it with JCO to generate JS bindings, and shimmed the wasi calls with a locally served copy of the preview2-shim package. The good news is that since those are now regular Rust functions, we can call it as if it was one.

    Because I'm using it in wasm_bindgen to process the returned values, you can see I've gated the function imports for our markdown processing functions behind a feature flag

    Rust code

    ```
    *use* *crate**::*functions*::*post*::**{*get_post*,* UpdatePost*}**;*

    *#*[*cfg*(*not*(*feature = *"ssr"**)**)**]**
    *use* *crate**::*js*;*

    *use* *crate**::*models*::**{*Post*,* SafeUser*}**;*
    *use* *crate**::*providers*::*AuthContext*;*
    *use* *crate**::*routes*::*blog*::*PostParams*;*
    *use* cfg_if*::*cfg_if*;*
    *use* leptos*::*****;*
    *use* leptos_meta*::*****;*
    *use* leptos_router*::*****;*

    *#*[*island*]**
    *pub* *fn* EditPostForm*(**user**:* *Option**<**SafeUser**>**,* *post**:* *Post**)* -> *impl* *IntoView* *{*
        *let* update_post = *create_server_action**::**<**UpdatePost**>**(**)**;*
        *let* content = *create_rw_signal**(**String**::**new**(**)**)**;*
        *let* toc = *create_rw_signal**::**<**Option**<**String**>**>**(*None*)**;*
        *let* show_post_metadata = *create_rw_signal**(**false**)**;*
        *view**!* *{*
          <ActionForm action=update_post class=*"w-full text-black dark:text-white"*>
            <div class=*"grid min-h-full w-full grid-cols-2"*>
              <section class=*"text-left flex-col w-full justify-between col-span-2 gap-4 dark:bg-gray-900 bg-slate-50 rounded mb-4 border-2 dark:border-yellow-400 border-gray-300 "*>
                <div on:click=move |_e| *{*
                    show_post_metadata.update*(*|b| *b = !*b*)*;
                *}*>

              //...
                <textarea
                  *type*=*"text"*
                  id=*"raw_content"*
                  name=*"raw_content"*
                  rows=*50*
                  class=*"w-full text-black border border-gray-500"*
                  on:input=move |ev| *{*
                      cfg_if! *{*
                          *if* #*[*cfg*(*not*(*feature = *"ssr"**)**)**]* *{* 
                              *let* new_value = event_target_value*(*& ev*)*; 
                              *let* output = js::process_markdown_to_html_with_frontmatter*(*new_value.into*(**)**)*; 
                              *match* output *{* 
                                  Ok*(*o*)* => *{* 
                                      content.set*(*o.content*)*;
                          			  toc.set*(*o.toc*)*; 
                                  *}*, 
                                  Err*(*e*)* => leptos::logging::log!*(**"{}"*, e*)*
                          *}* *}*
                      *}*
                  *}*
                >
                  *{*post.raw_content.clone*(**)**}*
                </textarea>
                <input
                  *type*=*"hidden"*
                  name=*"content"*
                  id=*"content"*
                  value=move || content.get*(**)*
                />
              </div>
              <section class=*"shadow-md rounded"*>
                <div
                  class=*"prose text-black prose dark:prose-invert dark:text-white text-base p-4 bg-slate-200 dark:bg-gray-800 w-full h-full rounded"*
                  inner_html=move || content.get*(**)*
                ></div>
              </section>
            </div>
          </ActionForm>
        *}*
    *}*

    ```* 

 *If you'll notice, we crate an `on:input` event handler that will read the value of the textarea, and calls our `process_markdown_to_html_with_frontmatter()` function that we imported from WASI, but only on the client, because it's only defined there.

Success! You can even try it yourself by adding `/edit` to the end of this post url like [so](https://benw.is/posts/compiling-rust-to-wasi/edit). If I setup auth right, you won't be able to actually commit changes, and if I didn't I'm sure someone will let me know :).

On this journey I learned a lot about webassembly, wasi, and wasi tooling in the Rust ecosystem. I have to say I'm excited, both for what's possible now and what's coming in the future. There idea that we could write programs in our own favorite language, and compile it to a universal format that can executed on almost any platform is truly revolutionary. I'm reminded of this [talk](https://www.destroyallsoftware.com/talks/the-birth-and-death-of-javascript) from DestroyAllSoftware in which he posits that these kinds of virtualized environments might take over the world, and it feels they were somewhat prescient.

The tooling itself is a bit wonky and experimental, but I expect that'll be resolved in time. If you happen to have a Rust only project, I expect compiling to WASI preview 2 would be somewhat smoother.The Rust project and the people behind the Bytecode Alliance seem to working diligently to make it as easy as possible.

For those who want to take a deeper look at the generated wasi or the Leptos app that uses it, feel free to check out the github repo [here](https://github.com/benwis/benwis_spin). In theory there should be a place where I can post the WASI file as a module for others to use, like crates.io, and if I find one I'll post it there as well. If you're working on a cool WASI project, I'd love to hear about it! Feel free to message me on my Mastodon at @benwis@hachyderm.io or through my other social media methods*