<!--yml
category: 未分类
date: 2024-05-27 14:40:16
-->

# Nix-shell shebang - NixOS Wiki

> 来源：[https://nixos.wiki/wiki/Nix-shell_shebang](https://nixos.wiki/wiki/Nix-shell_shebang)

You can use `nix-shell` as a script interpreter to

*   run scripts in arbitrary languages
*   provide dependencies with Nix

To do this, start the script with multiple shebang (`#!`) lines.
The first shebang line is always `#! /usr/bin/env nix-shell`.
The second shebang line declares the script language and the script dependencies.

As of Nix 2.19.0 you can also use the new CLI `nix shell` and flakes to define shebangs. See [docs](https://nixos.org/manual/nix/stable/command-ref/new-cli/nix.html?highlight=shebang#shebang-interpreter).

## Examples

### Bash

To run bash scripts, set the interpreter with `-i bash`

```
#! /usr/bin/env nix-shell
#! nix-shell -i bash -p bash

echo  hello  world 
```

You can use `nix-shell -p ...` to add dependencies:

```
#! /usr/bin/env nix-shell
#! nix-shell -i bash -p imagemagick cowsay

# scale image by 50%
convert  "$1"  -scale  50%  "$1.s50.jpg"  &&
cowsay  "done $1.q50.jpg" 
```

### Python

```
#! /usr/bin/env nix-shell
#! nix-shell -i python3 -p python3

print("hello world") 
```

```
#! /usr/bin/env nix-shell
#! nix-shell -i python3 -p python3Packages.pillow python3Packages.ansicolor

# scale image by 50%
import sys, PIL.Image, ansicolor
path = sys.argv[1]
image = PIL.Image.open(path)
factor = 0.5
image = image.resize((round(image.width * factor), round(image.height * factor)))
path = path + ".s50.jpg"
image.save(path)
print(ansicolor.green(f"done {path}")) 
```

### Rust

#### No dependencies

```
#!/usr/bin/env nix-shell
#![allow()] /*
#!nix-shell -i bash -p rustc
rsfile="$(readlink  -f  $0)"
binfile="/tmp/$(basename  "$rsfile").bin"
rustc  "$rsfile"  -o  "$binfile"  --edition=2021  &&  exec  "$binfile"  $@  ||  exit  $?
*/
fn  main()  {
  for  argument  in  std::env::args().skip(1)  {
  println!("{}",  argument);
  };
  println!("{}",  std::env::var("HOME").expect(""));
} 
```

#### With dependencies

uses [rust-script](https://github.com/fornwall/rust-script)

```
#!/usr/bin/env nix-shell
//!  ```cargo
//!  [dependencies]
//!  time  =  "0.1.25"
//!  ```
/*
#!nix-shell -i rust-script -p rustc -p rust-script -p cargo
*/
fn  main()  {
  for  argument  in  std::env::args().skip(1)  {
  println!("{}",  argument);
  };
  println!("{}",  std::env::var("HOME").expect(""));
  println!("{}",  time::now().rfc822z());
} 
```

### Haskell

```
#!  /usr/bin/env  nix-shell
#!  nix-shell  -p  "haskellPackages.ghcWithPackages (p: with p; [turtle])"  -i  runghc

{-# LANGUAGE OverloadedStrings #-}

import  Turtle

main  =  echo  "Hello world!" 
```

## Pinning nixpkgs

To pin nixpkgs to a specific version, add a third shebang line:

```
#! /usr/bin/env nix-shell
#! nix-shell -i bash
#! nix-shell -I nixpkgs=https://github.com/NixOS/nixpkgs/archive/aed4b19d312525ae7ca9bceb4e1efe3357d0e2eb.tar.gz

echo  hello  world 
```

## Flake

It is also possible to make it work for flake like in:

```
#!/usr/bin/env -S nix shell nixpkgs#bash nixpkgs#hello nixpkgs#cowsay --command bash

hello  |  cowsay 
```

The [doc](https://nixos.org/manual/nix/stable/command-ref/new-cli/nix.html?highlight=shebang#shebang-interpreter) mentions that it should be possible to run more complex commands using multiple lines, but it does not work for me as reported [here](https://github.com/NixOS/nixpkgs/issues/280033).

## Performance

TODO ... why the startup delay? how to make it faster?

## See also