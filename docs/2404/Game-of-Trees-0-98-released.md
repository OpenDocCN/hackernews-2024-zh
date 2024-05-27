<!--yml
category: 未分类
date: 2024-05-27 13:30:29
-->

# Game of Trees 0.98 released

> 来源：[https://www.undeadly.org/cgi?action=article;sid=20240424043509](https://www.undeadly.org/cgi?action=article;sid=20240424043509)

Contributed by [Peter N. M. Hansteen](http://bsdly.blogspot.com/) on 2024-04-23 from the are these all trees dept.

The version control system

[gameoftrees](https://gameoftrees.org/) [0.98](https://gameoftrees.org/releases/CHANGES)

has been released and should soon show up in

[OpenBSD](https://www.openbsd.org/) `-current` [packages](https://marc.info/?l=openbsd-ports-cvs&m=171387192004593&w=2)

. An update for the

`-portable`

version will follow as well.

The main improvements in the new release are listed in the release notes as

> ```
> - speed up got tag -l by caching timestamps in got_ref_cmp_tags()
> - provide a macro for vi(1) path for use by -portable at compile time
> - avoid a rename/stat race when gotd installs a new pack and then uses it
> - make 'got ref -l' output consistent when packed references exist
> - make 'got ref -l' work consistently when a reference argument is given
> - add initial support for notifications to gotd(8), via email and http/json
> 
> ```

> ```
> - display process title in syslog when a gotd child process exits
> - hide a pointless end-of-file error on imsg pipe in libexec helpers
> - plug a memory leak in 'got blame'
> - add support for topological sorting to the commit graph
> - add log -t option which enables topological sorting of commits
> - make 'got rebase' find a merge base with topological sorting if needed
> - call unveil(2) earlier during import, commit, histedit, and tag commands
> - make 'got status' display interrupted rebase, histedit, and merge operations
> - got.1: escape Eq since it's a GNU roff macro, to fix rendering in -portable
> - regress: use seq instead of jot, for portability reasons
> - get rid of unnecessary "dns inet" pledge promises while fetching via git://
> - add http clone/fetch support using a new got-fetch-http helper
> - drop git+ssh protocol name from documentation; Git has done the same
> - require -R option for staging or unstaging directory contents
> - got patch: fix applying on empty files
> 
> ```

See the [release notes](https://gameoftrees.org/releases/CHANGES) for details. Stefan Sperling (`stsp@`) provided some further description in a fediverse [post](https://bsd.network/@stsp/112320391033588976), which may be well worth signing up to a friendly Mastodon instance for.