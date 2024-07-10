<!--yml

类别：未分类

日期：2024-05-27 14:40:16

-->

# Nix-shell shebang - NixOS Wiki

> 来源：[https://wiki.nixos.org/wiki/Nix-shell_shebang](https://wiki.nixos.org/wiki/Nix-shell_shebang)

您可以使用 `nix-shell` 作为脚本解释器来

+   在任意语言中运行脚本

+   使用 Nix 提供依赖项

要做到这一点，请使用多个井号（`#!`）开头的脚本。

第一行井号始终是 `#! /usr/bin/env nix-shell`。

第二行井号声明脚本语言和脚本依赖项。

从 Nix 2.19.0 开始，您还可以使用新的 CLI `nix shell` 和 flakes 来定义 shebangs。详见 [文档](https://nixos.org/manual/nix/stable/command-ref/new-cli/nix.html?highlight=shebang#shebang-interpreter)。

## 示例

### Bash

要运行 bash 脚本，请使用 `-i bash` 设置解释器

```
#! /usr/bin/env nix-shell
#! nix-shell -i bash -p bash

echo  hello  world 
```

您可以使用 `nix-shell -p ...` 添加依赖项：

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

#### 没有依赖项

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

#### 带有依赖项

使用 [rust-script](https://github.com/fornwall/rust-script)

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

## 固定 nixpkgs

要将 nixpkgs 固定到特定版本，请添加第三行井号。

```
#! /usr/bin/env nix-shell
#! nix-shell -i bash
#! nix-shell -I nixpkgs=https://github.com/NixOS/nixpkgs/archive/aed4b19d312525ae7ca9bceb4e1efe3357d0e2eb.tar.gz

echo  hello  world 
```

## Flake

也可以使其适用于 flake，如下所示：

```
#!/usr/bin/env -S nix shell nixpkgs#bash nixpkgs#hello nixpkgs#cowsay --command bash

hello  |  cowsay 
```

[doc](https://nixos.org/manual/nix/stable/command-ref/new-cli/nix.html?highlight=shebang#shebang-interpreter) 提到可以使用多行运行更复杂的命令，但如 [此处](https://github.com/NixOS/nixpkgs/issues/280033) 所述，对我来说不起作用。

## 性能

TODO …… 为什么启动会延迟？如何加快速度？

## 另请参阅
