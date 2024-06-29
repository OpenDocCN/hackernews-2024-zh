<!--yml

类别：未分类

日期：2024-05-27 13:30:29

-->

# 游戏0.98版本已发布

> 来源：[https://www.undeadly.org/cgi?action=article;sid=20240424043509](https://www.undeadly.org/cgi?action=article;sid=20240424043509)

由[Peter N. M. Hansteen](http://bsdly.blogspot.com/)贡献于2024-04-23，来自这些都是树部门。

版本   版本控制系统

[gameoftrees](https://gameoftrees.org/) [0.98](https://gameoftrees.org/releases/CHANGES)

已发布，并应很快显示在

[OpenBSD](https://www.openbsd.org/) `-current` [软件包](https://marc.info/?l=openbsd-ports-cvs&m=171387192004593&w=2)

. 对于

`-portable`

版本也将随之而来。

新版本的主要改进在发布说明中列出

> ```
> - speed up got tag -l by caching timestamps in got_ref_cmp_tags()
> - provide a macro for vi(1) path for use by -portable at compile time
> - avoid a rename/stat race when gotd installs a new pack and then uses it
> - make 'got ref -l' output consistent when packed references exist
> - make 'got ref -l' work consistently when a reference argument is given
> - add initial support for notifications to gotd(8), via email and http/json
> 
> ```
> 
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

详细信息请参阅[发布说明](https://gameoftrees.org/releases/CHANGES)。Stefan Sperling (`stsp@`) 在一个fediverse [帖子](https://bsd.network/@stsp/112320391033588976)中提供了进一步的描述，这可能值得注册一个友好的Mastodon实例。
