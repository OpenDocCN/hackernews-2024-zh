<!--yml
category: 未分类
date: 2024-05-29 12:35:05
-->

# Redict is an independent, copyleft fork of Redis® | Redict

> 来源：[https://redict.io/posts/2024-03-22-redict-is-an-independent-fork/](https://redict.io/posts/2024-03-22-redict-is-an-independent-fork/)

##### March 22, 2024

Like many of you, I was disappointed when I learned that Redis® ^(was changing to [a non-free licensing model](https://github.com/redis/redis/pull/13157). This is a betrayal of the free software community, but perhaps not an entirely surprising one. Forks are likely to start appearing in the coming days, and today, I would like to offer [Redict](/) to you as a possible future home for your needs, and present its trade-offs as compared to the other forks you’re likely to be choosing from soon.)

In short, Redict is an independent, non-commercial fork of Redis® OSS 7.2.4. ^(It is based on the BSD 3-Clause source code of Redis® OSS, and all changes from this point onwards are licensed under the Lesser GNU General Public license, LGPL-3.0-only.)

## Why LGPL? [#](#why-lgpl)

The choice of the LGPL as the license for Redict is a deliberate one which balances a number of concerns. Most importantly, it is an irrevocable promise that Redict will always be free, much stronger than the [promise made by RedisLabs Co-founder and ex-CEO Yiftach in 2018](https://news.ycombinator.com/item?id=17819392). By using a copyleft license, all changes to Redict are required to be distributed using the same LGPL free software license, guaranteeing that modified versions of the software will be free.

Additionally, we will not be making use of any sort of Contributor License Agreement which gives one entity special privileges with respect to the copyright and licensing of Redict in a manner similar to that employed by Redis®. As a consequence, the copyright over Redict is held in common by all contributors, who would all have to agree to a future change of license, making it next to impossible that a similar change of license is in Redict’s future. The provenance of contributions is verified with the [Developer Certificate of Origin](https://developercertificate.org/).

There are many copyleft licenses and LGPL was chosen as the most suitable for Redict. The Affero GNU General Public License (AGPL) is a common choice for projects of this nature, particularly those run by stewards who, like Redis® Ltd, would prefer that cloud providers do not sell their software. The AGPL is a fine license, but we want to make it as easy as possible for users to comply with the Redict license and we do not see any reason to discourage cloud providers from making use of Redict. EUPL was considered, but was not selected for the same reasons.

LGPL was chosen over the GNU General Public License to reduce concerns that integrations with Redis® compatible Modules or Lua plugins would be subject to the “virality” of the GNU GPL.

The choice of the LGPL protects the future of the Redict project while balancing each of these concerns to the best effect possible.

<details><summary>Wait, how did you change the license? Are you allowed to do that?</summary>

Good question! Redis® OSS was based on the BSD 3-Clause license, which is a permissive license and allows for sublicensing provided that its terms are met. Redict complies with these terms by retaining the original license and copyright disclaimer. All changes made by Redict are licensed using the LGPL and the combined work as a whole is distributed under the terms of the LGPL. The licensing situation is clearly indicated in the repository by making use of the [REUSE](https://reuse.software) specification.

By the way, Redis® Ltd *does not* hold the copyright over the Redis® code (though they do own the trademark on the name). They're sublicensing it using the same permissive BSD license that everyone else gets, same as Redict, only they're using the non-free SSPL whereas we're using the free LGPL.</details> 

## Changes with respect to Redis® [#](#changes-with-respect-to-redis)

Presently, the changes from Redis® 7.2.4 are limited in scope. The main concern is changing the name in a backwards-compatible manner and establishing a technical foundation for an independent future. User facing changes implemented so far include the following:

*   The executables have been renamed to redict-*, e.g. redict-cli.
*   The Lua API provides a “redict” global compatible with the Redis® OSS API, available through the “redis” global for backwards compatibility.
*   The Module API symbols have been renamed, however, Redict retains ABI compatibility with Redis® OSS Modules up to version 7.2.4.

Work is underway to complete the renaming process, and a migration guide will be available when the first version, 7.3.0, is released, which will ideally be available next week. Redict will function as a drop-in replacement for Redis® OSS 7.2.4, though you may take steps to update your local configuration for Redict if it pleases you (such as migrating your database to /var/lib/redict).

We have also updated the repository to be conformant with the [REUSE](https://reuse.software/) specification, to ease the license conformance process and clarify the various software licenses that apply to Redict, including the BSD 3-Clause license of the original Redis® OSS code and the new LGPL changes, but also vendored dependencies such as Lua.

### Future changes [#](#future-changes)

The intention of Redict is to provide a continuation of development for a free software distribution of Redis® OSS compatible software, with minimally disruptive changes for the time being. Discussions are ongoing around the following changes:

*   Using this opportunity to remove some long-deprecated features, such as “redis-trib”
*   Eliminating vendored dependencies and moving to upstream Lua, jemalloc
*   Becoming more downstream agnostic, removing e.g. example systemd or upstart services

We will also be forking [Hiredis](https://github.com/redis/hiredis), as it is an internal dependency of Redict.

No effort will be made to remain compatible with future versions of Redis® SAL.

### Infrastructure changes [#](#infrastructure-changes)

This opportunity is being used to establish a community independent of proprietary infrastructure, such as GitHub and Slack. The source code is hosted on [Codeberg](https://codeberg.org/redict/redict), a Forgejo instance operated by a German non-profit, which should provide a comfortable and familiar user experience for anyone comfortable with the GitHub-based community of Redis® OSS. Additionally, we have established an IRC channel, [#redict on libera.chat](https://web.libera.chat/?channel=#redict), where the burgeoning community is being organized.

### Relationship to other forks [#](#relationship-to-other-forks)

Prior to the change of the Redis® license, a number of forks were already established, such as [KeyDB](https://docs.keydb.dev/). You should definitely check these out – maybe you’ve been missing out! However, these forks are more opinionated than Redict aims to be: Redict will provide a more conservative continuation of the Redis® OSS codebase.

In the coming weeks we are likely to see a few other forks of Redis® OSS appear. We may pull changes from forks which are permissively licensed, or use a copyleft license which is compatible with the LGPL.

## Help wanted! [#](#help-wanted)

Please join the project and help out! Established Redis® OSS contributors need only make themselves known and they will be granted push access to our upstream repository. If you had any outstanding pull requests against Redis® OSS, please take some time to rebase them against Redict. New contributors are also encouraged to participate in development.

Another easy place to help is building documentation for Redict, as the Redis® documentation does not use a free license and therefore cannot be adapted for Redict. We are also interested in participation from Linux distributions looking to replace their “redis” package with a free software alternative; please drop by and let us know your needs, concerns, and feedback.

Join us on IRC to get started: [#redict on libera.chat](https://web.libera.chat/?channel=#redict).

We will be establishing additional community infrastructure in the foreseeable future, including a security mailing list and an updated code of conduct. Keep an eye on this blog for updates.