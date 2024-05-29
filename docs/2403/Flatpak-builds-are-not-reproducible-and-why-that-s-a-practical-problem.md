<!--yml
category: 未分类
date: 2024-05-29 12:43:16
-->

# Flatpak builds are not reproducible and why that's a practical problem

> 来源：[https://ranfdev.com/blog/flatpak-builds-are-not-reproducible/](https://ranfdev.com/blog/flatpak-builds-are-not-reproducible/)

<hgroup>

# Flatpak builds are not reproducible and why that's a practical problem

## 6 minutes read. Published: 2022-05-09

</hgroup>

<details class="toc"><summary>Table of contents</summary></details>

## *Update 2022-05-10*

I've just talked to a flatpak contributor, who gave me more information on the topic. Flatpaks *technically* **are reproducible!** All the metadata needed to reproduce the build is present **inside** the built package.

> there is a lockfile that's generated already and its part of the end build. You can find them under /usr/manifest.json and /app/manifest.json for runtime and application respectively.

Indeed, these files are fixing the exact commits of the runtime, sdk and other dependencies:

```
[📦 org.gnome.Podcasts ~]$ cat /app/manifest.json {
 "id" : "org.gnome.Podcasts", "runtime" : "org.gnome.Platform", "runtime-version" : "41", "runtime-commit" : "a306cb5f76bba5e87223de4c98881ff65f32c2fba3dfac7af2673507d18dd741", "sdk" : "org.gnome.Sdk", "sdk-commit" : "be925b0b2a0092bd1372f130c72e7d2fffb53b238953469bb7943e75a0808bd7", ... } 
```

Also, they acknowledge everything is not perfect yet, but this is just because getting reproducibility everywhere is an hard problem (for everyone). There are [issues keeping track of reproducibility status](https://gitlab.com/freedesktop-sdk/freedesktop-sdk/-/issues/107) and they are also working with other technologies designed to respect archiving regulations:

> Buildstream, which we use for the runtimes but its also used for things like building endlessos and a lot embedded systems, does better in that regard. Its designed to be able to deal with archiving regulations in automotive industry for example, which doesn't get much more reproducible than that

### BUT...

While the information to reproduce the build is indeed present inside the built package itself, **that information is not committed to the source tree**, which **brings the problems I listed below in the original article**. Flatpaks are reproducible, but the tooling around them could be improved to get a better developer experience when building from source.

* * *

## Introduction

According to [flatpak.org](https://flatpak.org), "Flatpak is a next-generation technology for building and distributing desktop applications on Linux". I believe flatpak is becoming the de-facto standard app distribution format: Just open [flathub.org](https://flathub.org) to see how many flatpaks are already out there; It has broad distro support and it's completely open source (even server-side); overall, it's well appreciated by both users and developers. I want flatpaks to succeed, but I think they got some fundamental things wrong. Fortunately, these things can be fixed.

To create a flatpak, as a developer, you need two tools: `flatpak` and `flatpak-builder`.

> "Flatpak-builder is a tool for building flatpaks from sources."

The `flatpak` tool instead is mostly used to install flatpaks and managing flatpak repositories.

## The flatpak manifest

To package a flatpak app, you need to declare a manifest, a JSON file which will be read by `flatpak-builder`. This file must declare every dependency your app needs: a runtime (a common software bundle), runtime extensions (common software to extend a runtime), and custom dependencies.

Let's see a (simplified) example taken from an application of mine, [Geopard](https://ranfdev.com/projects/geopard):

```
{
  "app-id" : "com.ranfdev.Geopard.Devel",
  "runtime" : "org.gnome.Platform",
  "runtime-version" : "42",
  "sdk" : "org.gnome.Sdk",
  "sdk-extensions" : [
  "org.freedesktop.Sdk.Extension.rust-stable"
 ],  "command" : "geopard",
  "finish-args" : [
  "--share=ipc",
  "--socket=fallback-x11",
  "--socket=wayland",
  "--device=dri",
  "--share=network",
  "--filesystem=xdg-download/Geopard"
 ],  ...   "modules" : [
 {  "name" : "Geopard",
  "buildsystem" : "meson",
  "sources" : [
 {  "type" : "git",
  "url" : "../",
  "branch": "master"
 } ] } ] } 
```

## Problem 1: where should I get the runtime from?

`flatpak-builder` will parse the previous manifest and complain... The runtime `org.gnome.Sdk` is not installed! Ok... but where should I download the runtime from? Flatpak software is downloaded from a repository, but in this case, which repository? It's impossible to tell certainly, but when you are unsure, you should always try with `https://flathub.org`, which is the biggest public repository.

Let's setup the repository and install the other required software (notice, we have to do this manually!):

```
flatpak --user remote-add flathub https://flathub.org/repo/flathub.flatpakrepo flatpak --user install org.gnome.Platform//42 # I've manually added the version (stated in the manifest) as a suffix. # ... 
```

### Solution

There should be a "repos" field in the manifest, so that people (and tools) actually know where to get things from: Note: this field isn't optional. **A manifest without the required repos should be considered invalid**.

```
{
 "app-id" : "com.ranfdev.Geopard.Devel", "repos" : ["https://flathub.org/repo/flathub.flatpakrepo"], ... } 
```

## Problem 2: what version is it?

While installing the required runtimes and extensions, we've arrived at this step:

```
$ flatpak --user install org.freedesktop.Sdk.Extension.rust-stable Looking for matches… Similar refs found for ‘org.freedesktop.Sdk.Extension.rust-stable’ in remote ‘flathub’ (user):   1) runtime/org.freedesktop.Sdk.Extension.rust-stable/x86_64/1.6 2) runtime/org.freedesktop.Sdk.Extension.rust-stable/x86_64/18.08 3) runtime/org.freedesktop.Sdk.Extension.rust-stable/x86_64/19.08 4) runtime/org.freedesktop.Sdk.Extension.rust-stable/x86_64/20.08 5) runtime/org.freedesktop.Sdk.Extension.rust-stable/x86_64/21.08   Which do you want to use (0 to abort)? [0-5]: 
```

Good question... Which "ref" do I want to use?... I don't know.

### Solution

A ref without a version suffix *must* be considered invalid.

Invalid:

```
{"sdk-extensions" : [
  "org.freedesktop.Sdk.Extension.rust-stable" ]} 
```

Valid:

```
{"sdk-extensions" : [
  "org.freedesktop.Sdk.Extension.rust-stable//21.08" ]} 
```

## Problem 3: reproducibility

Since there isn't enough information to know the needed dependencies, it's clear there's no reproducibility. The text on the official flatpak website is misleading: "Develop and test your application in an environment that’s identical to the one users have.". Not even the developers have a consistent dev environment.

### Solution

Once `flatpak-builder` succesfully builds a manifest, it should write the exact dependencies it used inside a lockfile. This pattern is common to every package manager who cares about reproducibility: Nix, cargo, npm... I understand `flatpak-builder` isn't a package manager, but *something* in the flatpak build chain should create the lockfile.

I also understand why people may actually *don't want* to lock their runtime to a specific version: if the runtime is locked, it can't be updated without programmer intervention, so apps can't be automatically rebuilt to get security updates. This is a problem, indeed, but that can be easily solved by software repositories by simply overriding the lockfile. In this way, developers have reproducible debug builds (so they can actually track down bugs) and users can receive security updates. It's still better than what we have now.

## Conclusions

Those are (conceptually) simple solutions, but I think they can greatly improve the flatpak ecosystem. Nix, and Guix, the state of the art package managers, have already solved these issues. Flatpak is trying to be both a package manager and an app distribution system. The latter part is good, the former part is just... bad.

* * *

Copyright: Ranfdev 2019-2023