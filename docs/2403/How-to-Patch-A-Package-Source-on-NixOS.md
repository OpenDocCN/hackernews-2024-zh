<!--yml
category: 未分类
date: 2024-05-27 15:00:23
-->

# How to Patch A Package Source on NixOS

> 来源：[https://drakerossman.com/blog/how-to-patch-a-package-source-on-nixos](https://drakerossman.com/blog/how-to-patch-a-package-source-on-nixos)

This post is part of a series about NixOS. The series is done in support of my upcoming Opus Magnum, "Practical NixOS: the Book", and you can read the [detailed post here](/blog/practical-nixos-the-book).

Should you find this article lacking some information, be sure to tell me in the comments or elsewhere, since it is a living document and I am very keen on expanding it.

This article's full code is [available on NixOS Musings GitHub](https://github.com/drakerossman/nixos-musings/tree/main/how-to-patch-a-package-source-on-nixos).

<details open=""><summary class="pt-2 pb-2 ml-6 text-xl font-bold">Table of Contents (Click to Hide/Show)</summary></details>

# [](#introduction)Introduction

To get an overview of nix formatters ecosystem, please refer to a [previous article](/blog/overview-of-nix-formatters-ecosystem) of mine.

To format my nix code, I prefer the `alejandra` formatter (though I'd be switching to [nixfmt](https://github.com/serokell/nixfmt) once it's accepted as the standard). `alejandra` does not provide any configuration options, however I would really like to have the indent set to 4 spaces. `alejandra` is uncompromising, and so am I.

# [](#identifying-the-code-of-interest-to-be-patched)Identifying the Code of Interest to be Patched

We're going to achieve that in a rather simple way: we will replace the hard-coded formatting option of 2 spaces with 4\. For this, let's check the [sourcecode of `alejandra`](https://github.com/kamadorueda/alejandra/tree/main/src).

By searching for `indentation` throughout the codebase, we identify the file that has the indent size definition - [`/src/alejandra/src/builder.rs`](https://github.com/kamadorueda/alejandra/tree/main/src/alejandra/src/builder.rs).

Here's the [relevant snippet (line 49)](https://github.com/kamadorueda/alejandra/blob/e53c2c6c6c103dc3f848dbd9fbd93ee7c69c109f/src/alejandra/src/builder.rs#L49C1-L50C1):

```
match step {
 crate::builder::Step::Comment(text) => { let mut lines: Vec<String> = text.lines().map(|line| line.trim_end().to_string()).collect();   lines = lines .iter() .enumerate() .map(|(index, line)| { if index == 0 || line.is_empty() { line.to_string() } else { format!( "{0:<1$}{2}", "", 2 * build_ctx.indentation, line, ) } }) .collect();   add_token( builder, rnix::SyntaxKind::TOKEN_COMMENT, &lines.join("\n"), ); } 
```

To achieve the 4-space indentation, we will replace `2` with `4` in the line containing `2 * build_ctx.indentation`. Obvious!

# [](#modifying-the-source-via-postpatch)Modifying the Source via `postPatch`

But first, let's take a look at how `alejandra` is packaged on `nixpkgs`. Search for `alejandra` at [https://search.nixos.org/packages](https://search.nixos.org/packages?channel=unstable&from=0&size=50&sort=relevance&type=packages&query=alejandra), and you'll find `alejandra` as the first result, and from the web UI you can find the link to the nix package definition:

Here's the [source code](https://github.com/NixOS/nixpkgs/blob/nixos-unstable/pkgs/tools/nix/alejandra/default.nix), and the full definition fits just a couple of lines, so I copy it below:

```
{ lib , rustPlatform , fetchFromGitHub , testers , alejandra }:   rustPlatform.buildRustPackage rec {
 pname = "alejandra"; version = "3.0.0";   src = fetchFromGitHub { owner = "kamadorueda"; repo = "alejandra"; rev = version; hash = "sha256-xFumnivtVwu5fFBOrTxrv6fv3geHKF04RGP23EsDVaI="; };   cargoHash = "sha256-tF8E9mnvkTXoViVss9cNjpU4UkEsARp4RtlxKWq55hc=";   passthru.tests = { version = testers.testVersion { package = alejandra; }; };   meta = with lib; { description = "The Uncompromising Nix Code Formatter"; homepage = "https://github.com/kamadorueda/alejandra"; changelog = "https://github.com/kamadorueda/alejandra/blob/${version}/CHANGELOG.md"; license = licenses.unlicense; maintainers = with maintainers; [ _0x4A6F kamadorueda sciencentistguy ]; mainProgram = "alejandra"; }; } 
```

What do we see here?

We build a Rust package (`alejandra` is written in Rust) via `rustPlatform.buildRustPackage`. For that, we declare the package name and its version with `pname = "alejandra"` and `version = "3.0.0";`. Then, we fetch the source code of `alejandra` from its repository via `fetchFromGitHub`, pinned down to a specific commit with `hash = "sha256-xFumnivtVwu5fFBOrTxrv6fv3geHKF04RGP23EsDVaI=";`.

Next, we see `cargoHash = "sha256-tF8E9mnvkTXoViVss9cNjpU4UkEsARp4RtlxKWq55hc=";`. It is needed to guarantee the full reproducibility. To be completely sure that you're using the same source code all the time, and there are no changes introduced inbetween different builds, we specify the hash in the build instructions. This hash is derived from the tarball - the downloaded file containing source code - itself. Should the source code change, so will the the hash, and the build would fail upon a detected mismatch.

> `buildRustPackage` requires either the `cargoSha256` or the `cargoHash` attribute which is computed over all crate sources of this package.

Full documentation for `rustPlatform.buildRustPackage` is available at [Nixpkgs Manual](https://nixos.org/manual/nixpkgs/unstable/#compiling-rust-applications-with-cargo).

The remaining things are self-explanatory: `passthru.tests` runs the tests, and the `meta =` declares the metadata of this package, such as description, mainteners' names, and license.

Since we're working with the source code, we're going to add the following lines:

```
postPatch = ''  substituteInPlace src/alejandra/src/builder.rs \ --replace "2 * build_ctx.indentation" "4 * build_ctx.indentation" ''; 
```

Nix comes with a neat built-in function called `substituteInPlace`. It allows us to replace a line with some other one in a file where necessary, and do it without the need to write some shell script ourselves.

`postPatch` is a build step that instructs our builder to "apply patches", or, in this case, to modify the fetched source in place before compiling the Rust package.

Since we have modified `alejandra`'s indent level, we should get broken tests now. But it is safe to assume if `alejandra` passed tests successfully for `2` spaces indent already, `alejandra` should also be okay with `4` spaces after such a minor change. We're not going to modify the entire testing suite. Instead, we're just going to comment out the now-irrelevant parts: by commenting out `passthru.tests`, we will prevent the builder from running the tests, and that will save us some package compilation and realization time, as well as will not fail the actual build.

We're also going to remove the folder containing tests by adding the following line to `postPatch`, so it won't be copied to the nix store, saving us some storage space:

```
rm -r src/alejandra/tests 
```

Here's the final result:

```
{
 lib, rustPlatform, fetchFromGitHub,   }: rustPlatform.buildRustPackage rec {
 pname = "alejandra"; version = "3.0.0";   src = fetchFromGitHub { owner = "kamadorueda"; repo = "alejandra"; rev = version; sha256 = "sha256-xFumnivtVwu5fFBOrTxrv6fv3geHKF04RGP23EsDVaI="; };   cargoSha256 = "sha256-tF8E9mnvkTXoViVss9cNjpU4UkEsARp4RtlxKWq55hc=";      postPatch = '' substituteInPlace src/alejandra/src/builder.rs \ --replace "2 * build_ctx.indentation" "4 * build_ctx.indentation"   rm -r src/alejandra/tests '';   meta = with lib; { description = "The Uncompromising Nix Code Formatter"; homepage = "https://github.com/kamadorueda/alejandra"; changelog = "https://github.com/kamadorueda/alejandra/blob/${version}/CHANGELOG.md"; license = licenses.unlicense; maintainers = with maintainers; [_0x4A6F kamadorueda sciencentistguy]; mainProgram = "alejandra"; }; } 
```

# [](#using-patchfiles-in-builder)Using Patchfiles in Builder

Once we build this via `nix build`, we discover that the resulting binary is named `./result/bin/alejandra`. We would like to change that to `alejandra4`. You may try to adjust the `pname = "alejandra";`, but you'll soon discover that:

*   change to `pname = "alejandra4` has no effect on the name of the compiled binary;
*   you would also receive a hash mismatch for `cargoSha256` attribute. Supplying the proper hash still has no effect on the binary's name.

The actual name of the binary is coming from the inner workings of the `rustPlatform.buildRustPackage`, which defines it to match the name of crate in `Cargo.toml`. `alejandra`'s source repo uses a Rust workplace layout with 2 crates - `alejandra` and `alejandra_cli`, with latter depending on the former. After some trial and error we learn, that we would have to change the name in [src/alejandra_cli/Cargo.toml](https://github.com/kamadorueda/alejandra/blob/main/src/alejandra_cli/Cargo.toml).

With such a necssesity arising, we can now also learn how to apply a "true" patch.

`rustPlatform.buildRustPackage` provides a `cargoPatches`, which you may use with `Cargo.lock` (not `Cargo.toml`!) to provide newer versions of dependencies for a potentially outdated `Cargo.lock`.

However, to patch `Cargo.toml` you would need to use an "escape hatch" - a `patches = [ ./some.patch ];` inside the `rustPlatform.buildRustPackage` attrset.

To get the actual patch, we would do the following:

*   clone the `alejandra`'s repo - `git clone https://github.com/kamadorueda/alejandra`
*   modify the 2nd line `src/alejandra_cli/Cargo.toml` - change `name = "alejandra"` to `name = "alejandra4"`
*   `git add src/alejandra_cli/Cargo.toml`, so git includes the modified file to uncomitted tracked changes
*   `git diff` to produce the diff, which looks like the following:

```
diff --git a/src/alejandra_cli/Cargo.toml b/src/alejandra_cli/Cargo.toml index 34b2b16..f796dec 100644 --- a/src/alejandra_cli/Cargo.toml +++ b/src/alejandra_cli/Cargo.toml @@ -1,5 +1,5 @@
 [[bin]] -name = "alejandra" +name = "alejandra4"  path = "src/main.rs"  [dependencies] 
```

This is how a [standard Unix patch file](https://en.wikipedia.org/wiki/Patch_%28Unix%29) should be structured, except we should also remove the top 2 lines attached by git:

```
--- a/src/alejandra_cli/Cargo.toml +++ b/src/alejandra_cli/Cargo.toml @@ -1,5 +1,5 @@
 [[bin]] -name = "alejandra" +name = "alejandra4"  path = "src/main.rs"  [dependencies] 
```

You may have also produced such a patch file without `git` by simply copying the file, modifying the copy, and then running `diff original.toml modified.toml`, but you wouldn't get the header, which has the correct paths to patched files in it - including the arbitrary top-level differing directories names - `a` and `b`

Without these paths, build would error at the patching step with the message:

```
error: builder for '/nix/store/vh3s5cjznmisjva70k1pgf5ld36j53rd-alejandra-3.0.0.drv' failed with exit code 1;
 last 10 log lines: > Perhaps you used the wrong -p or --strip option? > The text leading up to this was: > > | src/alejandra_cli/Cargo.toml > |+++ src/alejandra_cli/Cargo.toml > > File to patch: > Skip this patch? [y] > Skipping patch. > 1 out of 1 hunk ignored For full logs, run 'nix log /nix/store/vh3s5cjznmisjva70k1pgf5ld36j53rd-alejandra-3.0.0.drv'. error: 1 dependencies of derivation '/nix/store/c2vqr9k3xi6cibaxj9yjvzzzgvxaq46c-nix-shell-env.drv' failed to build 
```

Save the patch as `patch-name.diff` in the same directory as the patched source builder definition, and then include it via `patches = [./name-patch.diff];` inside the builder:

```
{
 lib, rustPlatform, fetchFromGitHub,   }: rustPlatform.buildRustPackage rec {
 pname = "alejandra"; version = "3.0.0";   src = fetchFromGitHub { owner = "kamadorueda"; repo = "alejandra"; rev = version; sha256 = "sha256-xFumnivtVwu5fFBOrTxrv6fv3geHKF04RGP23EsDVaI="; };   cargoSha256 = "sha256-tF8E9mnvkTXoViVss9cNjpU4UkEsARp4RtlxKWq55hc=";    patches = [./name-patch.diff];   postPatch = '' substituteInPlace src/alejandra/src/builder.rs \ --replace "2 * build_ctx.indentation" "4 * build_ctx.indentation"   rm -r src/alejandra/tests '';   meta = with lib; { description = "The Uncompromising Nix Code Formatter"; homepage = "https://github.com/kamadorueda/alejandra"; changelog = "https://github.com/kamadorueda/alejandra/blob/${version}/CHANGELOG.md"; license = licenses.unlicense; maintainers = with maintainers; [_0x4A6F kamadorueda sciencentistguy]; mainProgram = "alejandra"; }; } 
```

That should do the work, and `alejandra4` now has the proper name.

# [](#conclusion)Conclusion

And that was it! You can now use `alejandra4`, e.g. as part of your devShell, or as part of your `environment.systemPackages`.

For the ease of access to the newly "created" `alejandra4` formatter, I am also providing a flake, which is published [at my GitHub](https://github.com/drakerossman/alejandra4).

Example of using said flake with a devShell:

```
{
 description = "A sample devshell flake with alejandra4";   inputs = { nixpkgs.url = "github:nixos/nixpkgs/nixos-unstable"; flake-utils.url = "github:numtide/flake-utils"; alejandra4.url = "github:drakerossman/alejandra4"; };   outputs = { self, nixpkgs, flake-utils, alejandra4, }: flake-utils.lib.eachDefaultSystem ( system: let pkgs = import nixpkgs { inherit system; }; in with pkgs; { devShells.default = pkgs.mkShell { packages = [ alejandra4.defaultPackage.${system} ]; }; } ); } 
```

All the Nix code in this article is formatted with `alejandra4` 🙂