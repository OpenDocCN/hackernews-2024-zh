<!--yml

类别：未分类

日期：2024-05-27 15:00:23

-->

# 如何在 NixOS 上修补软件包源码

> 来源：[https://drakerossman.com/blog/how-to-patch-a-package-source-on-nixos](https://drakerossman.com/blog/how-to-patch-a-package-source-on-nixos)

这篇文章是关于 NixOS 系列的一部分。该系列支持即将发布的我的大作，《实用 NixOS：本书》，你可以在[这里阅读详细文章](/blog/practical-nixos-the-book)。

如果你发现本文缺少某些信息，请务必在评论或其他地方告诉我，因为它是一个持续更新的文档，我非常愿意扩展它。

这篇文章的完整代码可以在[NixOS Musings GitHub](https://github.com/drakerossman/nixos-musings/tree/main/how-to-patch-a-package-source-on-nixos)上找到。

<details open=""><summary class="pt-2 pb-2 ml-6 text-xl font-bold">目录（点击隐藏/显示）</summary></details>

# [](#introduction)介绍

要了解 nix 格式化程序生态系统的概述，请参考我的[先前文章](/blog/overview-of-nix-formatters-ecosystem)。

为了格式化我的 nix 代码，我更喜欢`alejandra`格式化程序（尽管一旦[nixfmt](https://github.com/serokell/nixfmt)被接受为标准，我会转换过去）。`alejandra`不提供任何配置选项，但我真的很想将缩进设置为 4 个空格。`alejandra`毫不妥协，我也一样。

# [](#identifying-the-code-of-interest-to-be-patched)识别需要修补的感兴趣代码

我们将通过一个非常简单的方式来实现：我们将把硬编码的 2 个空格的格式选项替换为 4 个。为此，让我们查看[`alejandra`的源代码](https://github.com/kamadorueda/alejandra/tree/main/src)。

通过在代码库中搜索`indentation`，我们确定了定义缩进大小的文件 - [`/src/alejandra/src/builder.rs`](https://github.com/kamadorueda/alejandra/tree/main/src/alejandra/src/builder.rs)。

这里是[相关片段（第49行）](https://github.com/kamadorueda/alejandra/blob/e53c2c6c6c103dc3f848dbd9fbd93ee7c69c109f/src/alejandra/src/builder.rs#L49C1-L50C1)：

```
match step {
 crate::builder::Step::Comment(text) => { let mut lines: Vec<String> = text.lines().map(|line| line.trim_end().to_string()).collect();   lines = lines .iter() .enumerate() .map(|(index, line)| { if index == 0 || line.is_empty() { line.to_string() } else { format!( "{0:<1$}{2}", "", 2 * build_ctx.indentation, line, ) } }) .collect();   add_token( builder, rnix::SyntaxKind::TOKEN_COMMENT, &lines.join("\n"), ); } 
```

为了实现 4 个空格的缩进，我们将在包含`2 * build_ctx.indentation`的行中将`2`替换为`4`。显而易见！

# [](#modifying-the-source-via-postpatch)通过 `postPatch` 修改源码

但首先，让我们看看在 `nixpkgs` 上如何打包`alejandra`。在[https://search.nixos.org/packages](https://search.nixos.org/packages?channel=unstable&from=0&size=50&sort=relevance&type=packages&query=alejandra)搜索`alejandra`，你会发现它是第一个结果，通过 Web UI 你可以找到到 nix 包定义的链接：

这里是[源代码](https://github.com/NixOS/nixpkgs/blob/nixos-unstable/pkgs/tools/nix/alejandra/default.nix)，整个定义只有几行，我将其复制如下：

```
{ lib , rustPlatform , fetchFromGitHub , testers , alejandra }:   rustPlatform.buildRustPackage rec {
 pname = "alejandra"; version = "3.0.0";   src = fetchFromGitHub { owner = "kamadorueda"; repo = "alejandra"; rev = version; hash = "sha256-xFumnivtVwu5fFBOrTxrv6fv3geHKF04RGP23EsDVaI="; };   cargoHash = "sha256-tF8E9mnvkTXoViVss9cNjpU4UkEsARp4RtlxKWq55hc=";   passthru.tests = { version = testers.testVersion { package = alejandra; }; };   meta = with lib; { description = "The Uncompromising Nix Code Formatter"; homepage = "https://github.com/kamadorueda/alejandra"; changelog = "https://github.com/kamadorueda/alejandra/blob/${version}/CHANGELOG.md"; license = licenses.unlicense; maintainers = with maintainers; [ _0x4A6F kamadorueda sciencentistguy ]; mainProgram = "alejandra"; }; } 
```

我们在这里看到了什么？

我们通过 `rustPlatform.buildRustPackage` 构建了一个 Rust 包（`alejandra` 是用 Rust 编写的）。为此，我们用 `pname = "alejandra"` 和 `version = "3.0.0";` 声明了软件包名称和版本。然后，我们通过 `fetchFromGitHub` 从其仓库获取了 `alejandra` 的源代码，并使用 `hash = "sha256-xFumnivtVwu5fFBOrTxrv6fv3geHKF04RGP23EsDVaI=";` 将其固定到特定的提交版本。

接下来，我们看到 `cargoHash = "sha256-tF8E9mnvkTXoViVss9cNjpU4UkEsARp4RtlxKWq55hc=";`。这是为了确保完全的可重现性。为了确保始终使用相同的源代码，且在不同构建之间不会引入更改，我们在构建说明中指定了哈希值。这个哈希值是从 tarball（包含源代码的下载文件）中生成的。如果源代码发生变化，哈希值也会变化，构建将因检测到不匹配而失败。

> `buildRustPackage` 要求 `cargoSha256` 或 `cargoHash` 属性之一，它是计算此软件包所有 crate 源代码的哈希值。

`rustPlatform.buildRustPackage` 的完整文档可以在 [Nixpkgs 手册](https://nixos.org/manual/nixpkgs/unstable/#compiling-rust-applications-with-cargo) 找到。

其余的事情都很明了：`passthru.tests` 运行测试，而 `meta =` 则声明了此软件包的元数据，如描述、维护者姓名和许可证。

由于我们正在处理源代码，我们将添加以下行：

```
postPatch = ''  substituteInPlace src/alejandra/src/builder.rs \ --replace "2 * build_ctx.indentation" "4 * build_ctx.indentation" ''; 
```

Nix 自带一个很好的内置函数叫做 `substituteInPlace`。它允许我们在必要时在文件中用其他行替换一行，而无需自己编写 Shell 脚本。

`postPatch` 是一个构建步骤，指示我们的构建器“应用补丁”，或者在本例中，在编译 Rust 包之前就地修改获取的源代码。

由于我们修改了 `alejandra` 的缩进级别，现在应该会出现测试失败。但可以放心假设如果 `alejandra` 在 `2` 个空格缩进下已经成功通过了测试，那么在这样一个小改动后增加到 `4` 个空格也应该没问题。我们不打算修改整个测试套件。相反，我们只需注释掉现在不相关的部分：通过注释掉 `passthru.tests`，我们将阻止构建器运行测试，这将节省一些包编译和实现时间，也不会导致实际构建失败。

我们还将通过向 `postPatch` 添加以下行来删除包含测试的文件夹，以便它不会复制到 Nix 存储中，从而节省一些存储空间：

```
rm -r src/alejandra/tests 
```

这就是最终的结果：

```
{
 lib, rustPlatform, fetchFromGitHub,   }: rustPlatform.buildRustPackage rec {
 pname = "alejandra"; version = "3.0.0";   src = fetchFromGitHub { owner = "kamadorueda"; repo = "alejandra"; rev = version; sha256 = "sha256-xFumnivtVwu5fFBOrTxrv6fv3geHKF04RGP23EsDVaI="; };   cargoSha256 = "sha256-tF8E9mnvkTXoViVss9cNjpU4UkEsARp4RtlxKWq55hc=";      postPatch = '' substituteInPlace src/alejandra/src/builder.rs \ --replace "2 * build_ctx.indentation" "4 * build_ctx.indentation"   rm -r src/alejandra/tests '';   meta = with lib; { description = "The Uncompromising Nix Code Formatter"; homepage = "https://github.com/kamadorueda/alejandra"; changelog = "https://github.com/kamadorueda/alejandra/blob/${version}/CHANGELOG.md"; license = licenses.unlicense; maintainers = with maintainers; [_0x4A6F kamadorueda sciencentistguy]; mainProgram = "alejandra"; }; } 
```

# [](#using-patchfiles-in-builder)在构建器中使用补丁文件

一旦我们通过 `nix build` 构建了这个项目，我们发现生成的二进制文件被命名为 `./result/bin/alejandra`。我们希望将其改为 `alejandra4`。你可以尝试调整 `pname = "alejandra";`，但很快你会发现：

+   更改到 `pname = "alejandra4` 并不会影响编译后二进制文件的名称；

+   `cargoSha256` 属性的哈希值不匹配，你也会收到一个哈希值不匹配的通知。提供正确的哈希值仍然对二进制文件的名称没有影响。

实际的二进制文件名称来自 `rustPlatform.buildRustPackage` 的内部工作，它定义为与 `Cargo.toml` 中的 crate 名称匹配。 `alejandra` 的源仓库使用了一个包含 2 个 crate 的 Rust 工作空间布局 - `alejandra` 和 `alejandra_cli`，后者依赖于前者。经过一些试验和错误后，我们了解到，我们需要在 [src/alejandra_cli/Cargo.toml](https://github.com/kamadorueda/alejandra/blob/main/src/alejandra_cli/Cargo.toml) 中更改名称。

随着这种需要的出现，我们现在还可以学习如何应用一个“真正的”补丁。

`rustPlatform.buildRustPackage` 提供了 `cargoPatches`，您可以将其与 `Cargo.lock`（而不是 `Cargo.toml`！）一起使用，以为可能已过时的 `Cargo.lock` 提供更新的依赖项版本。

然而，要修补 `Cargo.toml`，你需要使用一个“逃生口”——在 `rustPlatform.buildRustPackage` 属性集内部的 `patches = [ ./some.patch ];`。

要获取实际的补丁，我们应该执行以下操作：

+   克隆 `alejandra` 的仓库 - `git clone https://github.com/kamadorueda/alejandra`

+   修改第二行 `src/alejandra_cli/Cargo.toml` - 将 `name = "alejandra"` 更改为 `name = "alejandra4"`

+   `git add src/alejandra_cli/Cargo.toml`，这样 git 就会包括修改后的文件到未提交的已跟踪更改中。

+   使用 `git diff` 生成补丁，它看起来像这样：

```
diff --git a/src/alejandra_cli/Cargo.toml b/src/alejandra_cli/Cargo.toml index 34b2b16..f796dec 100644 --- a/src/alejandra_cli/Cargo.toml +++ b/src/alejandra_cli/Cargo.toml @@ -1,5 +1,5 @@
 [[bin]] -name = "alejandra" +name = "alejandra4"  path = "src/main.rs"  [dependencies] 
```

这就是标准 Unix 补丁文件应该具备的结构，但我们还应该删除由 git 附加的前两行：

```
--- a/src/alejandra_cli/Cargo.toml +++ b/src/alejandra_cli/Cargo.toml @@ -1,5 +1,5 @@
 [[bin]] -name = "alejandra" +name = "alejandra4"  path = "src/main.rs"  [dependencies] 
```

您也可以通过简单复制文件、修改副本，然后运行 `diff original.toml modified.toml` 来生成此类补丁文件，而不使用 `git`，但是您不会获得正确的路径头部，其中包含了被补丁文件的正确路径，包括任意的顶级不同目录名称 - `a` 和 `b`

没有这些路径，构建会在补丁步骤中报错并显示以下消息：

```
error: builder for '/nix/store/vh3s5cjznmisjva70k1pgf5ld36j53rd-alejandra-3.0.0.drv' failed with exit code 1;
 last 10 log lines: > Perhaps you used the wrong -p or --strip option? > The text leading up to this was: > > | src/alejandra_cli/Cargo.toml > |+++ src/alejandra_cli/Cargo.toml > > File to patch: > Skip this patch? [y] > Skipping patch. > 1 out of 1 hunk ignored For full logs, run 'nix log /nix/store/vh3s5cjznmisjva70k1pgf5ld36j53rd-alejandra-3.0.0.drv'. error: 1 dependencies of derivation '/nix/store/c2vqr9k3xi6cibaxj9yjvzzzgvxaq46c-nix-shell-env.drv' failed to build 
```

将补丁保存为 `patch-name.diff`，与修补后的源构建器定义文件放在同一个目录下，并通过 `patches = [./name-patch.diff];` 在构建器内部包含它：

```
{
 lib, rustPlatform, fetchFromGitHub,   }: rustPlatform.buildRustPackage rec {
 pname = "alejandra"; version = "3.0.0";   src = fetchFromGitHub { owner = "kamadorueda"; repo = "alejandra"; rev = version; sha256 = "sha256-xFumnivtVwu5fFBOrTxrv6fv3geHKF04RGP23EsDVaI="; };   cargoSha256 = "sha256-tF8E9mnvkTXoViVss9cNjpU4UkEsARp4RtlxKWq55hc=";    patches = [./name-patch.diff];   postPatch = '' substituteInPlace src/alejandra/src/builder.rs \ --replace "2 * build_ctx.indentation" "4 * build_ctx.indentation"   rm -r src/alejandra/tests '';   meta = with lib; { description = "The Uncompromising Nix Code Formatter"; homepage = "https://github.com/kamadorueda/alejandra"; changelog = "https://github.com/kamadorueda/alejandra/blob/${version}/CHANGELOG.md"; license = licenses.unlicense; maintainers = with maintainers; [_0x4A6F kamadorueda sciencentistguy]; mainProgram = "alejandra"; }; } 
```

这应该能完成任务，并且 `alejandra4` 现在有了正确的名称。

# [](#conclusion)结论

就是这样了！现在你可以使用 `alejandra4`，例如作为你的 devShell 的一部分，或作为你的 `environment.systemPackages` 的一部分。

为了方便访问新创建的 `alejandra4` 格式化程序，我还提供了一个 flake，它发布在 [我的 GitHub](https://github.com/drakerossman/alejandra4) 上。

使用开发Shell的示例：

```
{
 description = "A sample devshell flake with alejandra4";   inputs = { nixpkgs.url = "github:nixos/nixpkgs/nixos-unstable"; flake-utils.url = "github:numtide/flake-utils"; alejandra4.url = "github:drakerossman/alejandra4"; };   outputs = { self, nixpkgs, flake-utils, alejandra4, }: flake-utils.lib.eachDefaultSystem ( system: let pkgs = import nixpkgs { inherit system; }; in with pkgs; { devShells.default = pkgs.mkShell { packages = [ alejandra4.defaultPackage.${system} ]; }; } ); } 
```

本文中的所有 Nix 代码都使用 `alejandra4` 进行格式化 🙂
