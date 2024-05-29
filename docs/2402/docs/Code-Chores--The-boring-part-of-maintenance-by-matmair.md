<!--yml
category: 未分类
date: 2024-05-27 14:52:44
-->

# Code Chores: The boring part of maintenance by matmair

> 来源：[https://polar.sh/matmair/posts/code-chores-the-boring-part-of-maintenance](https://polar.sh/matmair/posts/code-chores-the-boring-part-of-maintenance)

What kind of chores need to happen to keep a self-hosted project long-term viable? And why “base” funding is often more important than one-off funds.

InvenTree 0.14.0 (still WIP but the roadmap [is public](https://github.com/inventree/InvenTree/milestone/56)) is shaping up to be a big one again. We will probably have to stop saying that in every release blog, now that we accumulate nearly 200 issues/pull requests in a 77-day window.

> [InvenTree](https://github.com/inventree/inventree) is an Open Source (MIT), self-hosted inventory and part library management system written mainly in Python. It uses Django and DRF to expose a REST API that can be used from the web, iOS and Android apps, a Python library and IOT devices.

## Explaining by sample

A theme for me in this window was code chores. The changes that are not flashy, do not make the release notes and just keep things working properly. There were lots of things that fall into this category:

*   Adding a way to check if the API schema is changing with a PR and blocking a merge if the API version was not bumped ([PR 6440](https://github.com/inventree/InvenTree/pull/6440)).
*   Upgrading major framework versions. We had to migrate to Django 4 LTS as Django 3’s End of Life was approaching fast. We know that InvenTree is being run in highly restricted/regulated environments so addressing this was paramount. It has been in my ToDo [since April 2022](https://github.com/inventree/InvenTree/issues/2853) and took quite a lot of detective work and collaboration to get across the finish line. We had to patch multiple dependencies, most notably [django-mptt](https://github.com/django-mptt/django-mptt/pull/836) to get everything working right. It took me multiple attempts and the final commit number is representative of its age. Fortunately, we squash commits before they are merged into upstream ([PR 6173](https://github.com/inventree/InvenTree/pull/6173)).
*   Refactoring the code base to be as easy to read/understand as possible ([PR 6251](https://github.com/inventree/InvenTree/pull/6251)) or to address depreciation ([PR 6210](https://github.com/inventree/InvenTree/pull/6210)).
*   Bumping dependencies also has to be done regularly to keep the stack secure ([PR 6421](https://github.com/inventree/InvenTree/pull/6421)).

Exciting changes, right?

## Code Chores

All these things do not qualify for flashy blogs, my best description for them is code (base) chores.
These are what maintenance is really about. Keeping the project running (safe, reliable, maintainable, …) so that others can build the fun, simple feature requests. Both are important in their own way to keep projects alive. Marketing the latter is a lot simpler. Chores in a decent-size project are also not limited to code. You have to manage translations, social media, new security settings, CI integrations, a website and a host of other things.

## Funding the boring stuff

Constant funding for projects and maintainers is important to keep all these things happening. Not a lot of people enjoy chores, me neither.
Companies are often more willing to sponsor specific feature requests - they are easy to communicate to management, I get it. This presents a problem because the reality of InvenTree is that Oliver (who does a lot more maintenance than me) and I spend more time fixing bugs, answering (setup) questions and chores than building new features. From conversations with other developers, it seems like that is mostly the case in smaller self-hosted OSS projects. But not doing the chores ends up with you wearing dirty clothes (or insecure projects in our case).

So how do you attract money if it is a hard sell? There are many possible avenues, every big project is trying to balance its answer between easier/specialised distribution, open core, support contracts and a myriad of other options.
I am currently leaning in favour of this platform, Polar. Here we are able to easily split money coming in for feature requests between contributors and the project to help with maintenance efforts. It is sort of implied with the default setup and I appreciate that. It reflects the reality of a mature OSS project. A feature won't maintain itself magically after it is merged.

I am not sure if this (platform or model) will work but not trying is a guaranteed way to fail. Funding was never easy for OSS, I do not suspect this will change any time soon.

Subscribe if you want to get background on InvenTree and its ecosystem. Sponsor InvenTree [here](https://polar.sh/inventree/issues) or [on GitHub](https://github.com/sponsors/inventree), take a look at [InvenHost](https://polar.sh/invenhost/issues) and thank a maintainer.