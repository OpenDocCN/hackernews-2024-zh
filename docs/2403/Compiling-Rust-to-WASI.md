<!--yml

category: 未分类

date: 2024-05-29 12:31:51

-->

# 将 Rust 编译为 WASI

> 来源：[https://benw.is/posts/compiling-rust-to-wasi](https://benw.is/posts/compiling-rust-to-wasi)

最近，我暂时放下了构建和与 Rust 编译为 Webassembly 目标 `wasm32-unknown-unknown` 进行交互的工作，而是花费了[大量时间](https://github.com/fermyon/leptos-spin)与 Rust 编译为 [WASI](https://wasi.dev/) 或 `wasm32_wasi` 目标进行交互。

如果你不确定这两者之间的区别，原谅你，它们都是 Webassembly 对吗？答案是肯定的，但它们在几个关键方面有所不同。我认为 `wasm32-unknown-unknown` 类似于 `no_std`。它无法访问 Webassembly 虚拟机之外的任何内容。传统的文件访问、网络请求或内核的标准系统调用都无法以传统方式使用。这是浏览器本地支持的目标，并且 Leptos 用于客户端交互。

`wasm32-wasi` 就是 `wasm32-unknown-unknown` 加上了缺失的传统内容。它重新实现了传统的系统调用，如 `read()` 并提供了 C 所期望的 `File*` 对象的访问。它还修复了 `wasm32-unknown-unknown` 中的一个关键 ABI 不兼容性，这是由于 `wasm-bindgen` 中的一个错误而成为 Rust 标准。如果你想了解更多，请继续阅读，否则你可以跳到下一节。我花了太多时间弄清楚这个问题，所以你们都要听我唠叨一下。

很久以前，当 Rust 中的 Webassembly 支持尚未成熟时，负责此工作的 Rust 开发人员在处理 wasm-bindgen 中的结构体时犯了一个错误。LLVM 和 Clang 声称通过值传递结构体应该内联字段，而对于 Rust，应该通过指针传递。当你的 Rust 代码与 C 交互并执行此操作时，这就成了一个问题。无论是与 Rust 一起编译还是作为单独的 C Webassembly 模块编译为 emscripten，此问题都会显现出来。

> Rust 代码编译为 `wasm32_wasi` 和 `wasm32-unknown-emscripten` 可能会相互通信存在困难。

曾经，Rust 的 C ABI 的一个问题被修复了，但修复 `wasm32-unknown-unknown` 目标可能会破坏与使用旧行为下的 wasm-bindgen 编译为 Webassembly 的 Rust 应用的兼容性。这让我们有些为难。wasm-bindgen 现在对可能的新 ABI 有[*向前*兼容性](https://github.com/rustwasm/wasm-bindgen/pull/3595)，意味着它不再存在问题，但如何在 Rust 生态系统中引入这一变化仍然存在争议。你应该像创建一个新的目标比如 `wasm32-unknown-unknown2` 吗？某一天切换它？或者跨版边界切换？

至少目前为止，Rust 已决定最终会引入一个不稳定标志，通过切换到现有的 C ABI 修复 ABI。[这里是 PR](https://github.com/rust-lang/rust/pull/117919)，MCP 已经批准，所以很可能在未来某个时候会加入到语言中。所以这是个好消息。避免在 ABI 边界传递任何结构体是另一个选择，但需要编辑所有相关方。个人而言，我不想编辑所有我可能想使用的 C 依赖库，但你可以自行决定。

有趣的是，`wasm32-wasi` 和 `wasm32-unknown-emscripten` 目标已经解决了这个不兼容性。我猜第一个足够新，第二个则是未知/专门构建，所以认为这样做是合适的。对于 `wasm32-wasi`，这里是[修复过程](https://github.com/rust-lang/rust/pull/79998)。

Webassembly 和 WASI 的卖点之一是编译到它的东西应该是可移植的，但每种语言的 ABI 规范由各种语言自行决定。也许是因为 C 是最大/最古老的语言，我们似乎正在围绕[ C/Clang/llvm 的决定](https://github.com/WebAssembly/tool-conventions/blob/main/BasicCABI.md)进行统一。正如这篇精彩的文章为现有系统所描述的那样，这可能会有[其他影响](https://faultlore.com/blah/c-isnt-a-language/)。无论如何，Rust 的人们还没有完全承诺，他们正在为 Rust 编译到 Webassembly 的"wasm" ABI 选项进行工作，并且我不知道其他语言。

现在大家对 Webassembly 和 Rust 中的 Webassembly 学到了这么多（或者直接跳到这里），让我们谈谈构建一个 crate。我开始这个旅程是因为我想要一个更好的编辑器来写博客。为此，我使用 [femark](https://crates.io/crates/femark)，一个我编写的 crate，结合了 treesitter 和 pulldown-cmark，将 markdown 编译为带有语法高亮代码块的 HTML。唯一的麻烦是 treesitter（及其解析器）是用 C/C++ 编写的。

从历史上看，我一直在服务器上转换为 html，这里 crate 可以很容易地由 Rust 编译。这次我意识到为了得到我想要的实时预览体验，我必须在浏览器中运行它。经过许多调试之后，我很高兴地说，通过 WASI Zulip 上 Joel Dice 和其他几位可爱的朋友的帮助，我终于实现了这一点。

1.  编译 Femark 到 WASI

    你可能会认为这部分很容易，只需运行

    bash

    ```
    cargo build --target=wasm32-wasi

    ```

    它会生成一个 wasm 文件，对吗？错了！相反，你会得到这个

    C 代码

    ```
      *error* *occurred*: *Command* *"clang"* *"-O3"* *"-ffunction-sections"* *"-fdata-sections"* *"-fPIC"* *"--target=wasm32-wasi"* *"-I"* *"./typescript/src"* *"-Wall"* *"-Wextra"* *"-o"* *"/celCluster/projects/femark/target/wasm32-wasi/release/build/tree-sitter-typescript-db2920a2728a6446/out/./typescript/src/parser.o"* *"-c"* *"./typescript/src/parser.c"* *with* *args* *"clang"* *did* *not* *execute* *successfully* 

    ```

结果 Rust 编译器不知道如何处理这个目标解析器中的 C/C++。所以我去下载了 [wasi-sdk](https://github.com/WebAssembly/wasi-sdk) 并将发布的二进制文件安装到 `/opt/wasi-sdk-21.0`。然后设置了一些 RUSTFLAGs 和环境变量来指向其中的内容。

bash

```
RUSTFLAGS='-L /opt/wasi-sdk-21.0/share/wasi-sysroot/lib/wasm32-wasi -lstatic=c++ -lstatic=c++abi' CXXSTDLIB=c++ CC=/opt/wasi-sdk-21.0/bin/clang CXX=/opt/wasi-sdk-21.0/bin/clang++ CXXFLAGS="-fno-exceptions"  cargo component build --release

```

我们使用他们提供的特殊版本的 clang 和其他 C 库，使 wasi/C 互操作更容易。弄清楚 C/C++ 编译器并不是我的强项，所以我非常感谢有人帮助弄清楚这一点。幸运的是这次编译成功了。

1.  将 WASI 预览 1 文件转换为 WASI 预览 2 文件

如果你一直在关注 WASI 的发展，你就会知道最近我们看到了 [WASI 预览 2 的发布](https://component-model.bytecodealliance.org/)。WASI 预览 2 为 WASI 组件提供了一个组件模型，允许它们相互通信。目前，Rust 的 `wasm32-wasi` 目标针对的是 WASI 预览 1。我们希望预览 2 能够让 WASI 在浏览器中运行，配合新发布的 [jco](https://bytecodealliance.org/articles/jco-1.0) 转换器。链接的文章提到允许 WASI 预览 2 组件在 node.js 中运行，但它还有更多实验性支持在浏览器中运行它们。

将 WASI 预览 1 模块转换为 WASI 预览 2 组件的最简单方法是使用 [cargo-component](https://github.com/bytecodealliance/cargo-component)，基本上替换了上面的命令。你仍然需要所有的 RUSTFLAGS 和其他环境变量。如果你不使用 `cargo component new` 来处理这些问题，我发现通过将以下内容添加到你的 Cargo.toml 文件中也能运行。关键在于冒号后面的部分要与你在 wit 文件中定义的世界匹配。

TOML 标记

```
*[**package**.**metadata**.**component**]*
*package* *=* *"benwis:femark"*
*[**package**.**metadata**.**component**.**dependencies**]*

```

然后设置 wit 绑定。WASI2 定义了使用 [WIT 语言](https://component-model.bytecodealliance.org/design/wit.html) 从其他组件导出和导入的函数和类型。Cargo component 将使用这些绑定，并将它们与 `Guest` trait 中定义的 Rust 函数匹配。对于这个 crate，我们只想导出两个 Rust 函数：

机智

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

我不会在这里深入讨论 WIT 语言，但它相当于 Rust，并且我发现它功能强大且易于掌握。在写这篇文章时让我感到困惑的部分是，虽然函数映射到 Rust 函数，但在这里定义的类型将自动转换为 Rust 类型。因此，需要删除 Rust 中现有版本的 `HighlightError`、`HtmlOutput` 等，并用自动生成的 bindings.rs 文件中的生成版本替换它们。

为了映射这些函数，我们在 `src/lib.rs` 中定义了一个 `Component` 结构体，并为其实现了 `Guest` trait。

Rust 代码

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

完成所有这些后，我们可以运行 `cargo component build` 并获得一个包含这些导出函数和类型的 WASI 预览 2 组件 `.wasm` 文件。

> 记得我之前写过的关于ABI不兼容性的所有文本吗？好吧，WASI预览2定义了一个[规范ABI](https://github.com/WebAssembly/component-model/blob/main/design/mvp/CanonicalABI.md)，以便所有编译为WASI的语言能够与其他WASI组件交流。然而，这些类型和函数如何在WebAssembly本身中由各自的语言表示取决于每种语言。因此，编译Rust和C到相同的WASI组件时仍然会有ABI不匹配的问题，但如果它们编译为不同的组件，则会有一个通用的接口语言。

1.  使用jco将其转译为Web

    [jco](https://github.com/bytecodealliance/jco)可以将我们上面生成的`.wasm`文件转译为Node.js和浏览器能理解的JS ESM模块。实际上使用它非常容易。在我们转译之前，jco有一个命令可以查看WASI2模块的导入和导出。

    bash

    ```
    jco wit target/wasm32-wasi/release/femark.wasm

    ```

    我的模块的输出如下。

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

    看起来非常像我们的wit文件对吧？这里唯一新的东西是自动添加的所有wasi函数导入。相当酷，对吧？

    bash

    ```
    jco transpile --minify femark.wasm -o femark-wasi

    ```

    现在它将被转换为一个包含几个wasm文件和一些js粘合剂的文件夹

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

    让我们来看看JS文件，看看我们能看到什么。可能没有--minify使它更容易阅读。它非常长，所以我会在这里链接到它

    JavaScript代码

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

    *看到JS为了匹配Wit或Rust的类型系统而跳跃的花絮绝对是有趣的。返回一个`result<htmloutput, highlighterro>`会创建大量的switch case块！顶部我们可以看到一些导入，在底部是我们正在寻找的命名函数的导出！*

    记得WASI如何提供所有那些常见系统函数的接口吗？那些通过`@bytecodealliance/preview2-shim`包提供。如果你使用Node或使用JS捆绑器，你可以很容易地引入[npm包](https://www.npmjs.com/package/@bytecodealliance/preview2-shim?activeTab=versions)。因为我两者都不使用，我需要从我的Web服务器上服务该库，并更改js文件中的所有路径指向一个本地的[副本](https://github.com/bytecodealliance/jco/tree/main/packages/preview2-shim)。

    JavaScript代码

    ```
    *import**{**environment* *as* *e**,**exit* *as* *r**,**stderr* *as* *a**,**stdin* *as* *t**,**stdout* *as* *s**,**terminalInput* *as* *o**,**terminalOutput* *as* *c**,**terminalStderr* *as* *n**,**terminalStdin* *as* *i**,**terminalStdout* *as* *l**}**from**"/components/preview2-shim-browser/cli.js"**;**import**{**monotonicClock* *as* *d**,**wallClock* *as* *b**}**from**"/components/preview2-shim-browser/clocks.js"**;**import**{**preopens* *as* A*,**types* *as* *k**}**from**"/components/preview2-shim-browser/filesystem.js"**;**import**{**error* *as* *f**,**streams* *as* *p**}**from**"/components/preview2-shim-browser/io.js"**;**import**{**random* *as* *u**}**from* *"/components/preview2-shim-browser/random.js"**;*

    ```

    这是来自缩小版本的，但你可以看到我正在设置绝对路径指向由我的服务器提供的文件夹。有了这个，我们应该准备好继续了！*

**   服务所有文件

    如前所述，我们需要服务`preview2-shim/lib/browser`文件夹以及我们之前生成的femark wasm和js文件。在Leptos上用于SSR应用程序，这相当简单，只需将它们放在您的资产目录中。更有趣的部分是如何使用它。

    +   使用wasm_bindgen为导出的Leptos J'S函数生成Rust绑定

    wasm_bindgen 来拯救！在这里我们看到了导出到JS的wasm函数，并按照wasm_bindgen的要求将其传递到extern C块中。由于返回类型是结构体，我重新定义了返回类型，并创建了帮助函数来从`JsValue`中解析它们。可能有更好的方法来做这个，但是类型是在一个无法编译到wasm32_unknown_unknown的crate中定义的，我想这比定义一个单独的类型crate要容易些。

    Rust 代码

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

    +   在Leptos组件中使用它

    最终，决定时刻到了！我们已经将我们的库编译成了WASI预览2版，用JCO进行了转译以生成JS绑定，并用本地提供的preview2-shim包来衬接wasi调用。好消息是，因为这些现在是普通的Rust函数，我们可以像调用普通函数一样调用它。

    因为我在wasm_bindgen中使用它来处理返回的值，你可以看到我已经将我们的markdown处理函数的函数导入放在了一个特性标志后面。

    Rust 代码

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

*如果你注意到的话，我们创建了一个`on:input`事件处理程序，它将读取文本区域的值，并调用我们从WASI导入的`process_markdown_to_html_with_frontmatter()`函数，但只在客户端上，因为它只在那里定义。

成功！你甚至可以尝试自己添加`/edit`到这篇文章URL的末尾，像这样[so](https://benw.is/posts/compiling-rust-to-wasi/edit)。如果我设置了正确的认证，你将无法提交更改，如果我没有，我相信会有人告诉我的 :）。

在这个旅程中，我对WebAssembly、WASI和Rust生态系统中的WASI工具有了很多了解。我不得不说我很兴奋，既为现在可能的事情，也为未来即将到来的事情。我们能够用自己喜欢的编程语言编写程序，并将其编译成几乎任何平台都可以执行的通用格式，这种想法确实是革命性的。我想起了DestroyAllSoftware的这个[演讲](https://www.destroyallsoftware.com/talks/the-birth-and-death-of-javascript)，他在其中提出这些虚拟化环境可能会主导世界，看来他们有点先见之明。

工具本身有点奇怪和实验性，但我预计随着时间的推移会解决这个问题。如果你碰巧有一个纯Rust项目，我预计编译到WASI预览2会顺利些。Rust项目和Bytecode Alliance背后的人似乎正在努力使其尽可能简单。

对于那些想深入了解生成的wasi或使用它的Leptos应用程序的人，欢迎查看GitHub仓库[这里](https://github.com/benwis/benwis_spin)。理论上应该有一个地方可以发布WASI文件作为模块供其他人使用，就像crates.io一样，如果我找到了，我也会在那里发布。如果你正在做一个酷炫的WASI项目，我很乐意听到。请随时通过我的Mastodon @benwis@hachyderm.io或其他社交媒体联系方式来联系我。
