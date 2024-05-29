<!--yml
category: 未分类
date: 2024-05-27 15:00:54
-->

# Serving a Website From a Git Repo Without Cloning It - mediocregopher's lil web corner

> 来源：[https://mediocregopher.com/posts/git-proxy](https://mediocregopher.com/posts/git-proxy)

```
                .%*.                                                        .-.
               .%@@@+.                                                .--=%@@@-
               =@@@@@@-                                         :--+@@@@@@@@@*
               *@%@@@@@%:                               :--=#@@@@@@@@@@@@#@@+
               @@::%@@@@@@-:                   :::-=%@@@@@@@@@@@@@@%#*-  :@*
              .@@   =@@@@@@@@@+-:::::::-=*@@@@@@@@@@@@@@@@@@@%##-        @@:
              #@#     +%@@@@@@@@@@@@@@@@@@@@@@@@@@@@@%%#*:              -@@
              #@-        *%@@@@@@@@@@@@@@@@@%%%#=                       %@=
              @@:             -=+++==     ..                            @@:
              @@        .               -@@@                           +@@
             #@%     =@@@@-            +@@@@#              *+          %@#
             #@-   -@@@@@@@.          +@@@@@%           .%@@@@=        @@:
             @@:  +@@@@@@@@@         +@+ +@@@         %@@@@@@@@@      -@@
             @@  *@%@*  :@@@        *@+  -@@@       %@@@@@@@@@@@#     @@@
            #@% %@ @%    %@@       *@*   .@@@     =@@@*@@   :@@@@     @@:
            %@:## *@+    :@@      :@@     @@@    %@@: @#     .@@@    .@@
            @@:.  @@:    .@@      @@      @@@   %@:  @@       @@%    @@%
            @@   -@@.    =@@     @@.      @@@  %@.  #@-       @@#    @@.
           %@%   +@@.    @@.    %@-       @@@ #@    @@.      :@@    -@@
           %@:   +@@+   .@%    #@=        @@@.@    *@@       +@+    @@*
           @@:   +@@%. .%%    *@+         @@@      %@@      -@#     @@
           @@    :@@@@@@%    :@@          @@%      %@@      @#     *@@
          %@@     #@@@@*     @@           @@%      %@@.    %@.     @@:
          @@:      ===:     @@.           @@%      %@@:  =@%.     .@@
          @@.              #@=            @@%      %@@%+%@#.      @@%
          @@              +@*             @@#      -@@@@@*       .@@
         %@@             :@@              @@#       %@@#:        =@@
         @@:            .@@               @@#        :           @@=
         @@.            @@.               @@#                   :@@
         @@            #@=                @@%                   %@%
        #@@           +@+                 #@%                  .@@
        @@:          =@#                  +@%                  =@%
        @@          :@%                   -@@                  @@+
        @@         .@@                    -@:                 -@@
       -@@         @%                     .:                  *@#
       @@*         #                                          @@.
       @@                                                    =@@
      .@@                                                    @@*
      .@@                                                   .@@
      *@@                                                   =@#
      @@*                                                   @@*
      @@                                                   -@@
     .@@                                                   +@*
     .@@                                                   @@-
     -@@                                                  =@@
     %@%                                                  #@*
     @@                                                   @@:
     @@                                                  +@@
    .@@                                                  %@+
    :@%                                                  @@.
    :@%                                                 *@%
    -@.                                                 %@=
    %@                                                  @@.
    @@                                                 *@%
    @@                                                 %@=
    @%                                                 @@:
   :@%                                                +@@
   :@*                =+-                             #@+
   :@:              .%@@@.                            @@-
   -@              =@@@@@#                           :@@
   =@             *@#.=@@%                           #@%
   *@            #@-  -@@%                           %@-
   @@           #@:   :@@%                           @@:
   @@          #%     .@@%               .#@@*      -@@
   @%         #%      .@@%              +@@@@@-     %@%
   @#        %@       :@@%            .@@@@@@@@     %@:
   @#       %@        :@@%           +@@% .@@@@     @@:
   @#      %@         :@@%          #@@:   :@@@    -@@
   @#     @@.         .@@@        .@@%     .@@@=   %@%
   @#    @@.          .@@@:      %@@-       @@@%   %@:
   @#   @@             @@@@    %@@@         *@@@   @@:
   @# .@@              @@@@@@@@@@=          :@@@   @@
   * @@@               :@@@@@@@@.           :@@@  *@@
.@@@@@%                 -@@@@@.             .@@@  @@=
@@@@@:                    ..                 @@@  @@.
-@%:                                         @@@  @@.
                                             @@@  @@
                                             -@@ =@@
                                             .@@ @@+
                                             .@@:@@.
                                              @@#@-
                                              @@@.
                                              +@-
                                               -

```

**mediocregopher**'s lil web corner

[Home](/)  //  [Posts](/posts/)  /  [Follow](/follow)  /  [RSS](/feed.xml)  //  [Source](https://dev.mediocregopher.com/mediocre-blog/)  /  [License](/static/wtfpl.txt)

# Serving a Website From a Git Repo Without Cloning It

* * *

It's fairly common to use git repositories as a vehicle for serving websites. The webdev pushes their changes to some branch of a publicly available git repository, and some web server somewhere serves the current tip of that branch as the website. Github Pages would be the most famous example of this.

The domani reverse proxy also supports serving a website from a git repository. It previously did so by automatically cloning the repository locally, and periodically pulling changes down. This worked fine enough, but I figured it could be simplified further such that no local state is required except the current hash of the desired branch. This post is going to explain how this can be done by first guiding you through git's internals a bit.

[Domani](https://code.betamike.com/micropelago/domani)

## Git Branches: What Are They?

The first thing to understand is what git branches actually are; they are a kind of "ref" (reference). Git tags are another kind of ref. A ref is nothing more than a name which points to an object hash, most likely a commit hash. You can easily inspect the refs of a git project, even without the git tool itself. For example, the current tip of the `main` branch can be found in the `.git/refs/heads/main` file:

```
# cat .git/refs/heads/main 
3651fd0f0cb8d2a0dd62b1ab096db5e1c23dde3a 
```

Ref files, and therefore branch files, are just plaintext files containing a single object hash. When the current tip of a branch is changed the only real change which takes place on the filesystem is to change the content of its ref file. At its core, git is actually quite a simple tool.

## Git Objects: Friend or Foe?

We've established that a git branch points to a git object via the hash of that object, but what does that really mean?

There are four kinds of git object: blob, tree, commit, and tag. Regardless of the object's kind, it is stored in a file named after the SHA1 of the object within the `.git/objects` directory. Objects are always stored compressed using zlib, but it is their uncompressed form which is used for hashing.

This example computes the hash of the object pointed to from our previous example. You can see the output SHA1 is the same as the object's file name (with the first two characters used for a directory name, otherwise the `objects` directory would get too big.) Note the usage of `pigz -d`, which does the zlib decompression.

```
# cat .git/objects/36/51fd0f0cb8d2a0dd62b1ab096db5e1c23dde3a | pigz -d | sha1sum 
3651fd0f0cb8d2a0dd62b1ab096db5e1c23dde3a  - 
```

The body of a git commit object is more or less plaintext, save for a single null byte which separates a header string from the object's contents, so we can just look at it directly:

```
# cat .git/objects/36/51fd0f0cb8d2a0dd62b1ab096db5e1c23dde3a | pigz -d | tr '\0' '\n'
commit 264
tree 15b9b5c79989caee0ee45748e699c470b934ebc5
parent 498a33533f1a1cf00639e99cabcc11c25c330fd7
author Brian Picciano <me@mediocregopher.com> 1708196552 +0100
committer Brian Picciano <me@mediocregopher.com> 1708196552 +0100

Remove full gix project as a dependency 
```

As we can see from the header, the object in question is a commit (but we knew that). The `264` denotes the size of the rest of the file after the header. Following the `264` would be a null byte, except that the null byte was replaced with a newline by the `tr '\0' '\n'`.

Most of the rest of the file should be familiar. The author is listed, along with the commit's description which makes up the tail of the file. The parent refers to the previous commit in the chain by its object hash. The tree is perhaps unfamiliar, unless you've dove into git's internals before, but it is the most interesting bit for our purposes.

Contrary to how git usually presents commits to its users, git commits do not actually contain a diff from the previous to the current state of the repository. Each commit instead contains a reference to a tree, where each tree fully describes the state of the repository's files at that commit. What you see when you do something like `git show` is actually a diff generated in real-time between the previous and current trees.

As mentioned, trees are themselves another kind of git object, and so are referenced and queried just like commits. Let's look at the tree for our commit:

```
# cat .git/objects/15/b9b5c79989caee0ee45748e699c470b934ebc5 | pigz -d | tr '\0' '\n'
tree 535
100644 .gitignore
�_���/�j���jS$��
                �100644 Cargo.lock
��"��AJ���?��100644 Cargo.toml

...(plus more garbage) 
```

Welp, it looks like trees aren't so simple to look at as commits. Luckily git ships with a handy utility, `cat-file`, for directly viewing objects and pretty-printing their contents:

```
# git cat-file -p 15b9b5c79989caee0ee45748e699c470b934ebc5
100644 blob a45fb6cad22fb76a9ae0e3bb6a5324a90f800cf2    .gitignore
100644 blob 771896770dfaf8227fb0e1414ae5a5edd43fe6fc    Cargo.lock
100644 blob 8e3a31cf00a4a91661adc400afaf7c1d2abe5e39    Cargo.toml
100644 blob be3f7b28e564e7dd05eaf59d64adba1a4065ac0e    LICENSE.txt
100644 blob 0a39e5d12a2040e764b13924dd1955a61d10ff5f    README.md
100644 blob d161f2e1298586fd01e000cad6788f01e8ec446e    config-dev.yml.tpl
100644 blob 9dcb733174c2b7a8842062eeeeb204efb4f6d5b4    config.yml
100644 blob 39bacff615979221b0d50c43eca260b647f6fdd1    default.nix
100644 blob 0ce9e7605a533667693385bbee6fadacbad4a242    flake.lock
100644 blob 26134bf04deed36bcd3e61a24eb01f44cbdd405f    flake.nix
100644 blob e01adca51af5f42b438f0ff184070a7eebe32241    rust-toolchain.toml
100644 blob 77db547fb0079adc2f160af8f0ff50acfae98157    shell.nix
040000 tree 7dca9943ec53acb19678ac753b5ea6b8ae9b07aa    src
040000 tree df6d134b7c8ad6638a14db1d0e23b6686e34680e    static 
```

Much better. We can see that the tree contains a list of entries, where each entry denotes a file (blob) or sub-directory (tree), along with the permissions and name of the file/sub-directory. Looking at the contents of the `static` sub-directory's object we find yet another tree:

```
# git cat-file -p df6d134b7c8ad6638a14db1d0e23b6686e34680e
100644 blob 3417794e08925a6a8164297575430f3a67a24c98    foo.gmi
100644 blob 96a73f5d009a32c50b3980117907808d51076082    foo.html
100644 blob a10841a0b027398ed8ebd79f3d81f7146f7613ad    index.gmi
100644 blob a4317bb1bf03636bf7c3ce1c22942e98576ad922    index.html
100644 blob e67581f93fbdcf2157773ea4b3e8cdb47810c172    penguin.jpg
040000 tree ad8bbea913647a1392cb3073259cc2638516c4c1    subdir 
```

We can see from this how git trees are used to describe the full contents of a repository from just a single hash. If any file in the repository were to change then the hash of its associated blob object would change, which would change the associated entry in the tree the file falls in, which changes the hash of the tree's object, which changes the entry for the tree in the tree's parent directory, and so on. The change propagates all the way upward to the root tree object and its hash. This hash is then stored in a commit, which allows each commit to easily denote the entire state of the repository.

(This pattern of using a recursively hashed tree to uniquely identify an arbitrarily large amount of hierarchical data is called a Merkle tree.)

Anyway, let's get to the goods: how do we view the files themselves? Each file stored as a blob object. Let's check out the `foo.html` blob we found in our last example:

```
# cat .git/objects/34/17794e08925a6a8164297575430f3a67a24c98 | pigz -d | tr '\0' '\n'
blob 19
# FOO

This is foo 
```

Blob objects are like commit objects, nice and easy to parse; just a header, a null byte, and then the content of the file as-is.

That's all there really is to git objects (ignoring tags, we don't need them today). Armed with this knowledge we can continue on towards our ultimate goal.

## Remote Repositories

When cloning a git repository you've probably done something like:

```
git clone https://code.betamike.com/micropelago/domani.git 
```

Git then went and did a bunch of magical stuff, and afterwards the repo was fully cloned locally. But how did your git tool do that, given just a URL?

The answer is: it's complicated. There are actually two different protocols with which git might clone a repo over HTTPS: the smart protocol and the dumb protocol. The smart protocol is fast but requires a special purpose HTTP client in order to work. The dumb protocol is slower than the smart one, but it does not require a special HTTP client; it just serves files as they are without any special logic.

We're going to use the dumb protocol.

Let's return to the beginning and remember our actual goal here: we want to serve a website using the contents of a git repository, and specifically the contents of the tip of a specific branch of a git repository. To do this we need to know which commit is currently being pointed to by that branch. We can discover this by making a simple GET request:

```
# curl -sS 'https://code.betamike.com/micropelago/domani.git/info/refs'
3651fd0f0cb8d2a0dd62b1ab096db5e1c23dde3a        refs/heads/main 
```

This repo only has a single branch (and no tags), so only a single line is returned. And would you look at that, it's the same commit hash as we saw in our local copy! Let's now query that commit object, and see how objects are queried in general:

```
# curl -sS 'https://code.betamike.com/micropelago/domani.git/objects/36/51fd0f0cb8d2a0dd62b1ab096db5e1c23dde3a' | pigz -d | tr '\0' '\n'
commit 264
tree 15b9b5c79989caee0ee45748e699c470b934ebc5
parent 498a33533f1a1cf00639e99cabcc11c25c330fd7
author Brian Picciano <me@mediocregopher.com> 1708196552 +0100
committer Brian Picciano <me@mediocregopher.com> 1708196552 +0100

Remove full gix project as a dependency 
```

As you can see objects are returned in the exact same way they are stored locally. No surprises. While trees are difficult to look at without the `git cat-file` tool (which won't work for remote objects), blobs are still easy:

```
# curl -sS 'https://code.betamike.com/micropelago/domani.git/objects/34/17794e08925a6a8164297575430f3a67a24c98' | pigz -d | tr '\0' '\n'
blob 19
# FOO

This is foo 
```

It works exactly like the local git repo.

## Putting It All Together

Given all this, how would my special reverse proxy handle a request for `/static/foo.html`? Well, first it would need to query the repository server for the commit (using `/info/refs`), and then fetch the commit object in order to pull out the root tree hash, and then fetch the root tree object.

From there the server would need to look in the root tree object and check that it sees a tree entry called `static`, and fetch its tree object. The server would then check `static`'s tree object for a blob entry called `foo.html`, and finally it would fetch that blob, passing its contents back as the response to the original request (after stripping off the git object header).

This sounds like a lot of steps to serve a single file, but there's two key optimizations which can be made. The first is to cache the root tree's hash in memory, which skips two lookups right at the beginning. The root tree's hash will only change when the latest commit of the branch changes, so it's enough to cache it in memory and have a separate background process periodically re-check the latest commit.

The second optimization is to cache tree objects in-memory using their hash as a key. The object identified by a hash never changes, so this cache is easy to manage, and by caching the tree objects in memory (perhaps with an LRU cache if memory usage is a concern) all round-trips to the remote server can be eliminated, save for the final round-trip for the file itself.

If you'd like to see an example implementation of this idea you can check out my rust implementation for Domani:

[git.rs](https://code.betamike.com/micropelago/domani/src/commit/e416a766682af3b78538854da09eee62baaf3762/src/origin/git.rs)

Note that I was able to take advantage of the excellent gix crate to help me with decoding git objects. If your language of choice doesn't have a git object parsing library available you'll have to parse the objects manually, but honestly it shouldn't be too difficult anyway.

Finally, here's a sample website being served using this technique:

[Link](https://test.domani.micropelago.net/)

That's it! Even if this is a pretty niche use-case and doesn't change the world, I hope you still found it useful as an introduction to git's internals, and perhaps as a jumping off point for your own ideas of how git can be abused to do interesting things.

*Published 2024-02-17*

* * *

This site can also be accessed via the gemini protocol: [gemini://mediocregopher.com/](gemini://mediocregopher.com/)

[What is gemini?](/posts/gemspace-tour)