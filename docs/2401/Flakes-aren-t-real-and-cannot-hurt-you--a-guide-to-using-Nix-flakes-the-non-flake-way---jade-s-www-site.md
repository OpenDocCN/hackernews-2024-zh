<!--yml

类别：未分类

日期：2024-05-27 14:26:26

-->

# Flakes 不是真的，也不会伤害到你：一个使用 Nix flakes 的非 flakes 方法的指南 - jade 的 www 站点

> 来源：[https://jade.fyi/blog/flakes-arent-real/](https://jade.fyi/blog/flakes-arent-real/)

煽动性的标题已经讲过了，让我们开始吧。

我认为 Nix flakes 有一些相当大的好处，比如：

+   方便地锁定评估时依赖关系

+   通过仅在构建中包含跟踪文件来消除代码的无意义重建

+   通过普遍固定使 Nix 代码平均更具可重现性

+   据称缓存评估

+   可能通过减少对奇怪属性集的研究和一般的 `NIX_PATH` 错误性来使 Nix 更容易学习

然而，与此同时，有一些关于 flakes 的事情可能会让人产生不是最有效的方式。我个人在自己的工作中相对广泛地使用 flakes，但我使用的几种方式并不标准，有其原因。

Flakes 是 *可选的*，尽管一些薪水取决于此的人可能会说得不一样，但它们并不是 Nix 的（唯一）未来：它们只是具有内置固定系统的 Nix 代码的特殊入口点，不多不少。

Nix 继续因文档不佳而声名狼藉，部分原因是因为 nixpkgs 和 NixOS 的官方文档实际上不允许谈论 flakes，作为一种政策。这种情况部分是由于 Nix 开发人员和 nixpkgs 开发人员之间存在分歧，而这两个群体之间的重叠程度出奇的少。

Flakes 也是"支持 flakes"和"反对 flakes"派系之间产生很多社区内部冲突的症状或原因，但在某种程度上，这种情况表明了共识过程的破裂以及各种各样的参与者试图规避这些过程，很多人认为文档因不使用 flakes 而“过时”，以及 flakes 在博客文章或教程中的异常传播导致人们认为它们是必需的。

这篇文章是关于如何在一般情况下架构 Nix 项目的，特别关注如何在避免其限制的同时使用 flakes。它试图消除在这种单一文化中可能产生的误解。

# "Flakes 是 Nix 中的组合原语"

Nix 语言、函数和 nixpkgs 实用程序是一种有效的组合原语，更适合将项目的部分组合在一起，特别是如果它在单体库中。

使用 Nix 构建大型系统的最灵活方式只是将 flakes 作为入口点，并使用"旧"工具开发其余部分。这有多种原因：

+   flakes 结合了版本控制集成、依赖管理和锁定文件管理。在中等规模的项目中，即使在像我的 dotfiles 这样的规模上，锁定子项目的依赖关系通常也是非常不可取的。

    它们既不适合大规模工作，也不适合小规模工作：在小规模工作中，为了某些微小实用程序编写一个单独的`flake.nix`有太多的开销，而在大规模工作中，例如在 nixpkgs 中，如果实际使用 flake 进行依赖管理，`flake.nix`将会有 100,000 行`inputs`长。

+   在制作灵活的构建方面，flake 不支持配置[除非通过滑稽的滥用`--override-input`](https://github.com/boolean-option)。这意味着所有构建配置变体都必须提前预料到，或者必须使用传统的 nixpkgs/Nix 语言基元。

+   作为组合基元的 flake 与交叉编译完全不兼容。由于缺乏配置支持，`packages.${system}`无法用于交叉编译：没有地方可以指定要使用的体系结构进行构建。

因此，即使在 flake 的世界中，为了可重用和高效地组合软件，Nix 和 nixpkgs 提供的其他组合基元仍然是组装软件的最佳选择。然后，可以将 flake 归为仅仅是一个入口点和获取评估所需依赖项的方式（构建时依赖项应使用`pkgs.fetchurl`、`fetchFromGitHub`等）。

例如，要公开程序的多个配置，可以传统方式编写它，使用接受一些配置参数的 lambda，然后在 flake 本身内部多次调用该 lambda 以公开多个输出属性。这将软件的配置能力与软件的实际定义配置分离，并避免让 flake 的非系统配置定义构建定义的内部工作方式。

Nix 语言的最大优点和缺点之一是它是一种图灵完备语言，这给静态分析带来了痛苦，但也是其最大的资产之一：你可以对其进行编程。这可以看作是一个问题，但也很棒：你可以对软件进行程序化补丁，动态定义配置，读取任意格式的文件等等。

Nix 是一种函数式编程语言，这意味着它的基本组合基元是函数。甚至像 NixOS 模块或覆盖等“花哨”的对象只是可以移到单独的文件中、导入或通过其他函数的部分应用创建的函数（尽管由于模块中的`imports`按文件名去重，因此 NixOS 模块通常应通过路径导入而不是通过函数生成）。

请查看下一节，了解一起编写软件的具体方法。

# “Flakes 是你放置 Nix 代码的地方”

Flakes 只是一种制作 Nix 代码的标准化入口点的花哨方案。在任何规模的项目中，大部分 Nix 代码不应该位于`flake.nix`中，原因有几个。

将尽可能少的代码放入 `flake.nix` 的最微不足道的理由是可维护性：在 `flake.nix` 中存在与最近的德国和荷兰选举一样多的右倾漂移（令人担忧的多！），因此从仅此角度来看，将事物移出是有用的。

让我们谈谈在 flakes 出现之前就存在的一些标准模式，在 flakes 世界中仍然相关。

我正在使用 `package.nix` 来指代 nixpkgs 风格中编写包的标准方式，这些包使用 `callPackage` 调用。这与直接在 `flake.nix` 中使用 `pkgs` 编写的方式相对立。

一个 `package.nix` 文件大致如下所示：

```
 { 

  hello, stdenv,

  enableSomething ? false

}:

stdenv.mkDerivation (finalAttrs: {

}) 
```

如果可能，应该使用 `callPackage` 编写包定义，而不是在 `flake.nix` 中内联，因为使用 `package.nix` 会使它们成为小型、可组合、可配置和可移植的软件单元。此外，通过使用 `callPackage` 并以 nixpkgs 风格编写，将包移到项目之间，甚至将其上游到 nixpkgs，变得更加容易，因为它们看起来和工作方式都是熟悉的。

### 跨编译

`callPackage` 是跨编译中承担重要角色的一个较少被人知晓的事实。如果你在 `nativeBuildInputs` 中写入 `pkgs.foo`，这样的一个 Nix 表达式在跨编译时会失败，但是作为 `callPackage` 的参数的 `foo` 不会。这是因为 `callPackage` 会神奇地将出现在 `nativeBuildInputs` 内部的 `foo` 解析为意味着为构建计算机构建的 `pkgs.buildPackages.foo`；也就是说，一个为构建计算机构建的包。

`callPackage` 多次评估一个 Nix 文件，并使用不同的参数将结果拼接在一起，这样 `buildInputs` 神奇地接收到目标包，而 `nativeBuildInputs` 接收到构建包，即使相同的包名出现在两者中。魔法 ✨

也就是说，在以下故意有其他原因缺陷的 `flake.nix` 中：

```
 {

  outputs = { nixpkgs, ... }:

    let pkgs = nixpkgs.legacyPackages.x86_64-linux;

    in {

      packages.x86_64-linux.x = pkgs.callPackage ./package.nix { };

  };

} 
```

然后是 `package.nix`：

```
 { stdenv, hello, openssl }:

stdenv.mkDerivation {

  nativeBuildInputs = [ hello ];

  buildInputs = [ openssl ];

} 
```

顺便提一下，有注意到什么吗？是的，是 flakes 完全不支持跨编译。看下一个要点。:D

可以使用 `pkgs.buildPackages` 属性将东西拉入 `nativeBuildInputs`，并且使用 `pkgs` 来用于 `buildInputs` 但这样做并不常规，而且相当冗长。

[查看关于这些 callPackage 奇技淫巧的手册](https://nixos.org/manual/nixpkgs/stable/#ssec-cross-dependency-implementation) 以获取更多详情。另请参阅：[关于依赖类别的手册](https://nixos.org/manual/nixpkgs/stable/#ssec-stdenv-dependencies)。

[覆盖（overlay）](https://nixos.org/manual/nixpkgs/stable/#sec-overlays-definition) 是覆盖 nixpkgs 的函数，它在达到 [不动点（fixed point）](https://en.wikipedia.org/wiki/Fixed-point_combinator) 之前进行求值。覆盖接受两个参数，`final` 和 `prev`（有时也称为 `self` 和 `super`），并返回一个属性集，该属性集在 nixpkgs 顶部被浅层替换为 `//`。

叠加层对于在nixpkgs之外分发软件集合是有用的，并且在Flakes世界中仍然是有用的，因为叠加层是简单的函数，可以对任何版本的nixpkgs进行评估，并且如果使用`callPackage`编写，交叉编译也可以工作。人们可能会注意到`overlays`叠加层输出不是特定于架构的，这是因为它们被定义为接受包集并返回修改以进行的函数；这就是它们在这里正常工作的原因。

对于一个固定点的评估意味着它会被评估多次，直到它不再引用`final`参数（或者溢出堆栈）。这个想法出现在许多地方，包括LaTeX的目录、[Typst](https://typst.app)或其他排版程序中：通过生成目录，你可能会影响后续页面的布局并更改它们的页码，但在第一次运行之后，布局可能不会改变，因为唯一的变化是数字，所以*那次*迭代可能会收敛到最终结果。

`final`给出了属性集的*最终*版本，在叠加层被评估至其最大程度时；你的叠加层可能会在评估`final`中的属性时运行多次，甚至会导致无限递归。`prev`给出了当前叠加层或任何进一步叠加层之前的nixpkgs的版本。

例如，我们可以编写一个叠加层来重写GNU Hello，使其成为一个包装器，引用一个[优秀的复古计算机系列](https://www.youtube.com/watch?v=gQ6mwbTGXGQ)。`overlay.nix`的内容：

```
 final: prev: {

  hello = final.writeShellScriptBin "hello" ''

 ${prev.hello}/bin/hello -g "hellorld" "$@"

 '';

} 
```

然后：

```
» nix run --impure --expr '(import <nixpkgs> { overlays = [ (import ./overlay.nix) ]; }).hello'
hellorld 
```

在这里，我们修改的`nixpkgs`的属性`hello`现在是我们的脚本，调用原始的`hello`来说“hellorld”。

如果叠加层的惰性评估不正确，很容易意外导致无限递归。例如，属性集的属性名称是严格评估的，属性集中的所有名称都会立即评估，但属性的值是惰性评估的。[曾经有人尝试更改此行为](https://github.com/NixOS/nix/issues/4090)，但由于性能原因而被搁置。严格的属性名称可能是一个“足枪”，在某些情况下使用`mapAttrs`或类似机制在`prev`上生成要覆盖的事物集时可能导致令人困惑的无限递归。

如果一个叠加层实际上并没有替换任何东西或包含自引用，那么无限递归通常不是一个问题，就像叠加层分发非常简单的软件的情况一样，我们可以像下一节中所示的那样利用它。

### 叠加层在Flakes世界中的位置

*Flakes不支持交叉编译。*

我在这里使用措辞有点狡猾。Flakes并不会*阻止*你进行交叉编译，但你必须绕过Flakes，按照“老”方法来做。

由于 flakes 的这个设计缺陷，即不支持参数，编写 flake 项目中的包装最兼容的方法是首先将包定义写入一个覆盖层，然后从覆盖层中公开包。需要跨编译的消费者可以使用带有自己的 nixpkgs 副本的覆盖层，而不关心的消费者可以通过 `packages` 使用它。

考虑到["1000 个 nixpkgs 实例"](https://discourse.nixos.org/t/1000-instances-of-nixpkgs/17347/)，编写一个不修改 nixpkgs 中任何内容，只是添加内容的 flake 的合理方式是：

```
 {

  inputs.nixpkgs.url = "github:nixos/nixpkgs/nixos-unstable";

  inputs.flake-utils.url = "github:numtide/flake-utils";

  outputs = { self, nixpkgs, flake-utils, ... }:

    let

      out = system:

        let

          pkgs = nixpkgs.legacyPackages.${system};

          appliedOverlay = self.overlays.default pkgs pkgs;

        in

        {

          packages.myPackage = appliedOverlay.myPackage;

        };

    in

    flake-utils.lib.eachDefaultSystem out // {

      overlays.default = final: prev: {

        myPackage = final.callPackage ./package.nix { };

      };

    };

} 
```

需要跨编译的下游使用者可以使用覆盖层，而其他使用者可以像平常一样使用 `packages`。

这使用了一个巧妙的技巧，即调用覆盖层（overlay），它只是一个函数，使用 `final` 和 `prev` 作为最终的 nixpkgs 属性集。这绝对不适用于所有的覆盖层，因为覆盖层可以使用 `final` 进行自引用，实际上通常需要这样做，如果它们包含相互依赖的多个派生。

然而，通过稍微多做一点工作，这个问题可以非常干净地解决，同时也避免了任何可能的名称遮蔽问题！

如果确实需要使用一个真正的覆盖层来替换东西，请从 flake 中再次导入 nixpkgs，并将覆盖层作为参数。没关系。这只是多余的评估时间的一秒钟：

```
 let pkgs = import nixpkgs { inherit system; overlays = [ self.overlays.default ]; };

in 
```

### NixOS 模块

NixOS 模块，就像覆盖层和 `package.nix` 一样，基本上只是以一种高级方式调用的函数，并不是 flakes 的构造。

在 flakes 中与 `nixosModules.*` 输出一起使用时，它们是*与架构无关*的，因为它们只是函数，如果为由同一 flake 构建的软件定义一个模块，通常会想要在 [`nixpkgs.overlays`](https://nixos.org/manual/nixos/stable/options#opt-nixpkgs.overlays) 中使用覆盖层或上面的技巧，两次调用带有 `pkgs` 的覆盖层，以实际引入它（再次，以保持交叉编译兼容性）。

为了与将事物放在 `flake.nix` 外面以实现可重用性的主题保持一致，模块的代码可以放在一个单独的文件中，然后通过导入该模块并从其环境中注入依赖项来使用 `flake.nix`。

#### 注入依赖项

有几种方法可以从一个 flake 将依赖项注入到 NixOS 模块中，其中一种方法稍微有些丑陋。从 `flake.nix` 中注入值到 NixOS 需要有几个原因，其中最重要的是，要在 NixOS 配置中使用 flakes 管理的依赖项。同样，为了[正确配置 `NIX_PATH` 以便在 flake 配置中解析 `<nixpkgs>`](https://github.com/NixOS/nixpkgs/pull/254405)，从 `flake.nix` 获取实际输入是必要的，以获得适合创建对实际 flake 输入的依赖关系的 nixpkgs 的正确引用。

最简单（也是我认为最合理的）注入flakes中的依赖项的方法是编写一个内联模块，在`flake.nix`中将它们放在其词法闭包中。如果你想要花哨一点，甚至可以制作一个选项来存储注入的依赖项：

```
 let depInject = { pkgs, lib, ... }: {

  options.dep-inject = lib.mkOption {

    type = with lib.types; attrsOf unspecified;

    default = { };

  };

  config.dep-inject = {

    flake-inputs = inputs;

  };

};

in {

  nixosModules.default = { pkgs, lib, ... }: {

    imports = [ depInject ];

  };

} 
```

将依赖项注入到NixOS模块中的更丑陋，也许更为人所知的方法是[`specialArgs`](https://nixos.org/manual/nixos/unstable/options#opt-_module.args)。这更丑陋，因为它被转储到每个模块的参数中，这与NixOS中其他数据流工作方式不同，它也无法在实际调用`nixpkgs.lib.nixosSystem`的flake之外工作。后者是更加阴险的部分，也是我强烈推荐使用具有闭包的内联模块而不是`specialArgs`的原因：它们会破坏flake的组合。

话虽如此，*要么*使用`specialArgs` *要么*在`flake.nix`中内联模块，而不是在上面的选项中，是注入模块导入的唯一方法。也就是说，如果使用类似`imports = [ config.someOption ]`的选项，它会导致无限递归错误。对于这种情况，我们建议将导入放在`flake.nix`中的内联模块中。

要使用`specialArgs`，需要将属性集传递到`nixpkgs.lib.nixosSystem`，然后这些属性集会进入NixOS模块的参数中。

```
 nixosConfigurations.something = nixpkgs.lib.nixosSystem {

  system = "x86_64-linux";

  specialArgs = {

    myPkgs = nixpkgs;

  };

  modules = [

    ({ pkgs, lib, myPkgs }: {

    })

  ];

} 
```

#### 例子

这可以等效地通过上面对pkgs的覆盖调用技巧来完成。

例如，这定义了一个非常实用的NixOS模块，在启动时在控制台上对用户发出咪咪声：

```
 {

  inputs.nixpkgs.url = "github:nixos/nixpkgs/nixos-unstable";

  outputs = { self, nixpkgs, ... }: {

    overlays.default = final: prev: {

      meow = final.writeShellScriptBin "meow" ''

 echo meow

 '';

    };

    nixosModules.default = { pkgs, config, lib, ... }: {

      imports = [ ./nixos-module.nix ];

      config = lib.mkIf config.services.meow.enable {

        nixpkgs.overlays = [ self.overlays.default ];

        services.meow.package = lib.mkDefault pkgs.meow;

      };

    };

  };

} 
```

和包含实际代码的`nixos-module.nix`：

```
 { pkgs, config, lib, ... }:

let cfg = config.services.meow; in {

  options = {

    services.meow = {

      enable = lib.mkEnableOption "meow";

      package = lib.mkOption {

        description = "meow package to use";

        type = lib.types.package;

      };

    };

  };

  config = lib.mkIf cfg.enable {

    systemd.services.meow = {

      description = "meow at the user on the console";

      serviceConfig = {

        Type = "oneshot";

        ExecStart = "${cfg.package}/bin/meow";

        StandardOutput = "journal+console";

      };

      wantedBy = [ "multi-user.target" ];

    };

  };

} 
```

<details><summary>我如何测试上述内容</summary>

我将这些放入了`flake.nix`中：

```
 nixosConfigurations.test = nixpkgs.lib.nixosSystem {

  system = "x86_64-linux";

  modules = [

    self.nixosModules.default

    ({ pkgs, lib, ... }: {

      fileSystems."/" = {

        device = "/dev/sda1";

      };

      boot.loader.grub.enable = false;

      boot.initrd.enable = false;

      boot.kernel.enable = false;

      documentation.enable = false;

      environment.noXlibs = true;

      services.meow.enable = true;

      system.stateVersion = "23.05";

    })

  ];

}; 
```

然后我构建了系统配置：

```
» nix build .#nixosConfigurations.test.config.system.build.toplevel 
```</details>

# "Flakes是Nix的未来，也是唯一的CLI"

许多文字已经写了关于新CLI及其设计的文章，主要集中在flakes上。然而，这并不是新CLI的唯一模式：在任何有意义的地方，它实际上都完全支持非flakes的使用。

要与旧的CLI获得更精确的等价性，`-L`（`--print-build-logs`）和`--print-out-path`很有用。同样，旧的CLI可以通过传递`--log-format bar-with-logs`来改善其输出到新CLI的输出。我不得不提到[nix-output-monitor](https://github.com/maralorn/nix-output-monitor)作为观察Nix构建的更好方式。

下表显示了等价关系：

| **旧CLI** | **等效** |
| --- | --- |

|

```
nix-build -A hello
```

|

```
nix build -f . hello
```

|

|

```
nix-shell -A blah
```

|

```
nix develop -f . blah
```

|

|

```
-
```

|

```
nix run -f . hello
```

|

|

```
nix-build -E
'(import <nixpkgs> { config.allowUnfree = true; }).blah'
```

|

```
nix build --impure --expr
'(import <nixpkgs> { config.allowUnfree = true; }).blah'
```

|

|

```
nix-instantiate --eval --strict -E 'blah'
```

|

```
nix eval --impure --expr 'blah'
```

|

# "Flakes是管理外部依赖项的方法"

Flakes是管理外部依赖项的一种方式，但在该角色中它们有许多缺陷。

一个缺陷是所有依赖项都需要列在一个文件中，并且没有办法将它们限制在组中。

对于构建所需但不需要评估的内容，flake 输入存在一个文档不足的内置获取器的限制，这也是它们被禁止在 nixpkgs 中使用的原因之一（除了 [`restrict-eval`](https://nixos.org/manual/nix/stable/command-ref/conf-file.html#conf-restrict-eval) 使其无法工作之外），即它们在获取时阻止了进一步的评估。与此相对的选择是使用在构建时执行获取的固定输出派生物，例如使用 `pkgs.fetchFromGitHub`、`pkgs.fetchurl` 等方式。

阻塞并不一定是最大的问题，如果依赖项是用于评估构建所需的 Nix 代码，因为你无法避免与之相关的阻塞，但当依赖项不需要评估时，这可能会造成麻烦，因为它会 [减慢和串行化评估](https://jade.fyi/blog/nix-evaluation-blocking/)，一次只下载一个东西。如果依赖项是评估所需的，则没有太多方法可以改善这一点，但例如，对于需要许多构建时输入的构建，比如一堆 tree-sitter 语法、Haskell 包源码等，切换到固定输出派生物获取器将节省大量时间。

## 如果不是 flakes，那么是什么？

有一个完全合理的论点可以解释为什么要以与 nixpkgs 相同的方式处理依赖项，并在 Nix 源代码中直接调用 `pkgs.fetchurl` 等。这样做效果很好，是常规做法，并且避免了评估时构建依赖项（“从派生物导入”（IFD））的问题。

有自动更新这些并获取适当哈希的工具是很好的。

有几个工具可以维护带有 Nix 哈希的锁文件，例如 [Niv](https://github.com/nmattia/niv)、[npins](https://github.com/andir/npins) 和 [gridlock](https://github.com/lf-/gridlock)。遗憾的是，前两者都提供了使用内置获取器的 Nix 文件，因此存在评估性能问题，而后者则不提供任何 Nix 代码。这并不是对这些项目的抨击：它们的主要目的是固定 Nix 代码，对于这一目的，内置获取器是正确的选择，但这意味着它们提供的代码不应该用于构建依赖项。

因此，构建时依赖项的解决方案是忽略任何你选择使用的提供的 Nix 代码，并编写一些代码来读取工具的 JSON 文件，并从中提取软件包的 URL 和哈希值，并调用 `pkgs.fetchurl`。这样做非常容易，我们建议这样做。

# 为什么会有五种获取软件的方法？

很容易迷失在各种做事方式的数量中，并且忘记这其中的任何一个是为了什么。我认为这实际上是由 flakes 造成的 *因为* 它们使示例是自包含的，使它们更容易被直接拿起并运行而不需要深思熟虑。通常，许多博客以 flakes 形式提供示例，可能是作为 NixOS 模块或其他形式，这些形式可能具有很多可能对所需应用程序多余的花哨功能。

如果一个人对 Nix 还很陌生，那么很容易就会想到 "Nix Nix Nix 我要将世界化为虚无，Nix" 并将一切都转换为与 Nix 纠缠在一起，了解可用的机制以及为什么可能需要一个特定的机制是有帮助的。

安装事物的各种方式与可变性和 "效果" 有不同的关系。在这里，效果是指对计算系统的修改，其他系统可能使用后安装脚本进行。Nix 衍生物不支持后安装脚本，因为 "安装" 没有任何意义。通过持久性，我指的是它们对系统产生某种持久的影响，而不仅仅是将东西放入 Nix 存储中。

这一部分也许值得单独一篇文章，但我将简要总结一下：

## 项目目录中的 `flake.nix` (`default.nix`, `shell.nix`)

这些是项目的 *开发者* 打包方式：固定的工具版本，不太关心与系统统一依赖关系等。为此，它们提供了开发 shell 以用于项目工作，并且与项目一起进行版本控制。此外，它们可能提供打包以从 nixpkgs 单独安装工具。

与可能在 nixpkgs 中看到的打包相比，这些打包有几个显著之处：

+   在 nixpkgs 中，更多的构建时间可能是可以容忍的，而且很少有渴望进行增量编译的愿望。从 derivation 导入也被 nixpkgs 禁止。由于这些原因，nixpkgs 外的打包可能使用不同的框架，例如 [`crane`](https://github.com/ipetkov/crane)、[`callCabal2nix`](https://github.com/NixOS/nixpkgs/blob/bd645e8668ec6612439a9ee7e71f7eac4099d4f6/pkgs/development/haskell-modules/make-package-set.nix#L225) 和其他类似工具，以减轻维护 Nix 打包的负担或加速重建速度。

+   它与软件一起进行版本控制，因此通常会容忍更多的错误：可以固定库或其他类似的东西，并且不经常更新 nixpkgs。

+   它们包括 *工作* 项目所需的工具，这可能是构建所需工具的超集，例如，在生成的代码或其他类似内容已检入的情况下。

+   它们可能包括诸如检查、提交后钩子和其他项目基础设施等内容。

Shell 只是由环境变量构成的，并且（至少在不构建一个的情况下）不会创建一个包含所有 shell 中内容的单个 `bin` 文件夹，例如。另外，由于它们由环境变量构成，它们没有太多能力执行管理服务或磁盘状态等效果。

用于特定于一个项目的工具，例如该项目的编译器和库。 根据口味和情况，这些可能会或可能不会用于语言服务器。 通常不用于提供像`git`或`nix`这样的工具，因为这些工具预期已在系统上，除非它们确实用于实际编译软件。

## 临时外壳（`nix shell`，`nix-shell -p`）

临时外壳是 Nix 的超能力之一，因为它可以从虚无中呈现软件，而不必担心以后如何清除它。 从这个角度来看，它与项目特定的`flake.nix`文件具有完全相同的功能：你可以将软件包引入范围内，或者执行其他任何可以在那里完成的操作。

我认为项目外壳文件只是简单地保存了一个`nix-shell -p`调用，这样做的动机大致相同，只是通过固定等更多的软件工程可维护性提升。

用于临时获取任何可能有的工具。

## `nix profile`，`nix-env`

`nix profile`和`nix-env`构建了一个包含`bin`、`share`等的目录，并将其符号链接到提供文件的实际包。 在幕后，这些主要是`pkgs.buildEnv`，这是一个脚本，用符号链接构建这样的结构。 然后将其符号链接到 PATH 中的某个位置。

就我个人而言，我认为除非被说服以声明性方式运行，否则永远不应该使用`nix profile`和`nix-env`，因为在 Nix 生态系统中，它们作为同时具有命令和持久性的异常情况，通过完全指定意图来避免各种问题（它们在升级中有一些丑陋的边缘情况，通过简单地将指定软件包来自何处的 Nix 代码写入文件来解决）。

用于无用。 或者不用，我不是警察。

## [flakey-profile](https://github.com/lf-/flakey-profile)，`nix-env --install --remove-all --file`

旧的 Nix 配置文件 CLI 实际上支持声明性包安装，尽管我不建议使用，因为 [flakey-profile](https://github.com/lf-/flakey-profile) 在用户体验上更加愉悦，而且在实现上绝对是微不足道的。 从功能上讲，它们与`nix profile`和`nix-env`一样，构建了一个包含`bin`、`share`等的目录，并将其放置在 PATH 中。

用于轻量级声明性包管理，也许是在非 NixOS 系统上（在 NixOS 上使用它没有任何限制，但 NixOS 本身就在那里）。

## `home-manager`

那些使用`home-manager`的人通常将其用于替换 dotfile 管理器，以及配置服务和安装特定于用户的软件包。 我不使用它，因为基于符号链接到 git 存储库的方法避免了不必要的 Nix 构建，并大大简化了配置文件迭代周期的复杂性。

`home-manager` 本质上具有 NixOS 的功能，可以具有服务、激活脚本等效果，同时只针对一个用户。

用于在一个用户上安装软件包，可能不是在 NixOS 系统上，在声明方式下，以及配置用户范围的服务。请注意，这与上述描述的档案存在重叠；它只是使用相同工具构建的更重的机制。

## NixOS/`nix-darwin`

NixOS 和 `nix-darwin` 是建立在 Nix 档案配置文件和激活脚本之上的系统范围配置管理系统。它们允许以声明方式安装和配置服务，并以声明方式管理配置文件。在实现和使用方面，它们与 `home-manager` 做着类似的事情，但作用范围是系统范围的。

用于系统范围安装软件包和配置服务。

# 结论

我希望成功地说明了为什么 flakes 不是万能的，甚至也许说明了它们不是真实存在的。尽管 nixpkgs 并不总是一个令人赞叹的 Nix 项目架构的典范，但它是一个庞大的项目，从他们组织事物的方式中可以学到很多，这可能比在设计 flakes 时内化的还要多。

即使假设 flakes 在宏观层面的组合方面很好，它们经常伴随着微观层面组合的糟糕使用，而这些微观层面组合最好仍然是使用旧的原语来完成。

有了更好的架构，我们可以绕过 flakes 的限制，创建易于使用和可扩展的 Nix 代码。我们可以澄清所有“只需写一个 flake”博客文章中的含义，以了解其中的 Nix 工具，并避免对使代码难以理解的 flakes 产生错误、不必要的依赖。
