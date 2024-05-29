<!--yml
category: 未分类
date: 2024-05-27 15:20:01
-->

# How to Learn Nix, Part 49: nix-direnv is a huge quality of life improvement

> 来源：[https://ianthehenry.com/posts/how-to-learn-nix/nix-direnv/](https://ianthehenry.com/posts/how-to-learn-nix/nix-direnv/)

The *reason* I [discovered an ancient blog post](/posts/how-to-learn-nix/installing-nix-on-macos/) the other day was that I had something new to say about Nix for the first time in over two years.

The thing I want to say is this: [`nix-direnv`](https://github.com/nix-community/nix-direnv) is great. It fixes roughly every problem that I’ve had with `nix-shell`, and does so in a much nicer way than my previous ad-hoc solutions.

This is important because I *mostly* just use Nix to document and install per-project native dependencies. I do use it to install “global” tools as well, but that is [rarely very interesting](https://ianthehenry.com/posts/janet-game/how-to-patch-emacs/), and most of my interaction with Nix these days consists of editing small `shell.nix` files.

But it took a bit of doing to get to the point that I felt *good* about using Nix for this. For one thing, shells don’t register GC roots, which means that every time you collect garbage you have to re-download all the dependencies for the project you were working on. We overcame that hurdle in [part 37](/posts/how-to-learn-nix/saving-your-shell/), by making a custom wrapper around `nix-shell` that sets up GC roots correctly, but it was surprisingly difficult.

For another thing, Nix is pretty insistent that you use *bash* as your interactive shell. I figured out a workaround for that in [Nix classic](/posts/how-to-learn-nix/nix-zshell/), but [essentially failed](/posts/how-to-learn-nix/nix-develop/) to make `nix develop` similarly usable.

[`nix-direnv`](https://github.com/nix-community/nix-direnv) solves both of these problems. Instead of spawning a new shell, it just adds environment variables to your existing shell. And when it evaluates `shell.nix`, it automatically registers the result as a GC root.

It also only re-evaluates `shell.nix` when it actually changes, which means that it in the typical case there’s no startup time. In contrast, my GC-root-installing wrapper takes about 750ms to open a typical shell (raw `nix-shell`, without the GC root evaluation dance, takes only 400ms). This doesn’t sound very long, because it’s not – I’m running Nix on what I can only characterize as a supercomputer. But I originally installed Nix on a laptop that pre-dated germ theory, and its startup latency was a lot more annoying.

`nix-direnv` also automatically updates the environment when `shell.nix` changes, so you don’t have to close and re-open your `nix-shell` whenever you add a dependency. Not only is this ergonomically better, but it also means that you don’t mess up your shell history every time you add a dependency or exit a project.

I had never used [`direnv`](https://direnv.net/) before, and to this date the only thing I’ve used it for is managing my Nix shells. But it’s a general tool for managing per-directory environment variables, which is *essentially* all that `nix-shell` is. `nix-shell` can also register bash functions – if you’re using bash – which is useful if you want to use it to debug a derivation. But for my purposes, environment variables are all I really need.

`direnv` has some built-in support for Nix, but it isn’t great; [direnv publishes a table outlining some of the advantages](https://github.com/direnv/direnv/wiki/Nix#some-factors-to-consider) of using `nix-direnv`. `nix-direnv` is some sort of plugin(?) that replaces the native Nix support with something much better. And it’s great. It makes the “reproducible developer environment” aspect of Nix just work™. And it’s pretty easy to use:

First off, install `nixpkgs.direnv` and `nixpkgs.nix-direnv`.

I installed them with `nix-env`, using the same declarative wrapper that I wrote in [part 22](/posts/how-to-learn-nix/declarative-user-environment/). If you install `nix-direnv` in a different way, the following will be different.

Installing `nix-direnv` doesn’t “enable” the plugin; you have to separately tell `direnv` about it:

```
mkdir -p ~/.config/direnv
echo 'source ~/.nix-profile/share/nix-direnv/direnvrc' > ~/.config/direnv/direnvrc
echo 'eval "$(direnv hook zsh)"' >> ~/.zshrc 
```

Once you do that, you have to run the following commands in every directory that you want to nix-shellify:

```
echo 'use nix' > .envrc
direnv allow 
```

And you’re done. That’s it! Now every time you navigate to that directory, you’ll have&mldr;

```
$ cd ~src/project
direnv: loading ~/src/project/.envrc
direnv: using nix
direnv: nix-direnv: using cached dev shell
direnv: export +CONFIG_SHELL +HOST_PATH +IN_NIX_SHELL +MACOSX_DEPLOYMENT_TARGET +NIX_BUILD_CORES +NIX_CFLAGS_COMPILE +NIX_COREFOUNDATION_RPATH +NIX_DONT_SET_RPATH +NIX_DONT_SET_RPATH_FOR_BUILD +NIX_ENFORCE_NO_NATIVE +NIX_IGNORE_LD_THROUGH_GCC +NIX_INDENT_MAKE +NIX_NO_SELF_RPATH +NIX_STORE +PATH_LOCALE +SOURCE_DATE_EPOCH +XDG_DATA_DIRS +__darwinAllowLocalNetworking +__impureHostDeps +__propagatedImpureHostDeps +__propagatedSandboxProfile +__sandboxProfile +buildInputs +builder +configureFlags +depsBuildBuild +depsBuildBuildPropagated +depsBuildTarget +depsBuildTargetPropagated +depsHostHost +depsHostHostPropagated +depsTargetTarget +depsTargetTargetPropagated +doCheck +doInstallCheck +dontAddDisableDepTrack +gl_cv_func_getcwd_abort_bug +name +nativeBuildInputs +nobuildPhase +out +outputs +patches +phases +propagatedBuildInputs +propagatedNativeBuildInputs +shell +shellHook +stdenv +strictDeps +system -PS2 ~PATH
```

Oh. Well that’s not great.

By default `direnv` prints every environment variable that it adds, removes, or changes. Which makes sense if you’re using it for, like, credentials or something, but for Nix shells it’s just a waste of scrollback.

There’s not really a simple way to suppress printing that giant `export` line, but you can hack it away by adding something like this to your `.zshrc`:

```
export DIRENV_LOG_FORMAT="$(printf "\033[2mdirenv: %%s\033[0m")"
eval "$(direnv hook zsh)"
_direnv_hook() {
  eval "$(direnv export zsh 2> >(egrep -v -e '^....direnv: export' >&2))"
}; 
```

(The `.`s in the regex exclude the “dim text” control characters at the beginning of the line.)

That removes the giant export line without removing the rest of the input. And now:

```
$ cd ~src/project
direnv: loading ~/src/project/.envrc
direnv: using nix
direnv: nix-direnv: using cached dev shell
```

Ahhh. That’s better.

I’ve been using `nix-direnv` for a few months now, and I must say: I wish that I had installed it sooner. It’s a *much* nicer experience than the default `nix-shell`, and I’m happy that I can get rid of the bespoke hacks that I’ve accrued over the years.

&mldr;almost. The one thing this does not help with is `nix-shell -p`. `nix-shell -p` is a useful way to “temporarily” install packages without actually putting them on your PATH, and I still use my zsh hack so that `nix-shell -p` doesn’t drop me into a bash session. Although I do this rarely enough that I could probably just suffer through it.