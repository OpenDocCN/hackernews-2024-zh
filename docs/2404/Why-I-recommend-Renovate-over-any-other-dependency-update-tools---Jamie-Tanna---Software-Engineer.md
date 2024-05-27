<!--yml
category: 未分类
date: 2024-05-27 13:08:49
-->

# Why I recommend Renovate over any other dependency update tools · Jamie Tanna | Software Engineer

> 来源：[https://www.jvt.me/posts/2024/04/12/use-renovate/](https://www.jvt.me/posts/2024/04/12/use-renovate/)

If you've read my blog before, or interacted with me at work or in the Open Source world, you're likely to know that I'm a huge fan of [Renovate](https://docs.renovatebot.com/).

For those that aren't aware, Renovate is one of the big players in dependency updating tooling, commonly seen in comparisons with Dependabot or Snyk.

I've been using Renovate for the last ~5 years, alongside a mix of Dependabot and Snyk to keep me grounded. I *absolutely love* Renovate, and make a point of making it so I can run Renovate where possible.

I've also had the experience operating Renovate in self-hosted mode as well as used [the hosted Renovate app by Mend](https://developer.mend.io) and the Mend Enterprise SAAS, and [I've written about the lessons learned self-hosting Renovate](https://www.jvt.me/posts/2024/05/03/renovate-self-hosting-lessons/).

Renovate has some really key features that set it apart from the competition, and I'm sure in the time it's taken me to write this, they've shipped some new ones 😻

(Aside: I'm largely writing this blog post now, as I've recently been shouting the benefits of Renovate, and instead of writing another internal-only document at work (as I did at Deliveroo), I wanted to [write it as a form of blogumentation](https://www.jvt.me/posts/2017/06/25/blogumentation/)).

## Prior art

A few years back, I wrote about [some tips for using Renovate](https://www.jvt.me/posts/2021/09/26/whitesource-renovate-tips/) to make keeping your software up-to-date easier, which I still stand by.

Since then, I've worked across a number of different ecosystems, repository sizes, and levels of comfort merging dependency updates, and have learned a few more things about effectively using Renovate - but through it, I'm still very sure of Renovate being the best tool in the ecosystem.

## Configurability

Renovate is *extremely* configurable, with [dozens of configuration options](https://docs.renovatebot.com/configuration-options/) to tune your experience. But Renovate doesn't end up being "too" configurable, where you end up spending more time tweaking config than doing the changes, but configurable enough that it's very likely you can do what you need to with it.

## Centralised, shareable presets

This is something that really hit me when we were rolling out Dependabot at Deliveroo, where my team owned ~30 repositories, and so for each of these repos, we needed to hand-craft a `dependabot.yml`, as Dependabot needs to be told which directories contain which dependencies, and how often to update them.

After we'd had about a week of usage, we started needing to tweak the configuration in a few of them to reduce the noise, which then required us to raise PRs across all the repos and get them updated.

Because Dependabot requires a bit of a snowflake configuration per repository, this wasn't easily automatable, even with [great tools available to automate some of the bulk updates](https://www.jvt.me/posts/2023/01/21/bulk-git-repo-changes/), which made this a rather onerous and frustrating process.

Compare this to Renovate, where there's an excellent first-class support for [shareable config presets](https://docs.renovatebot.com/config-presets/), in which you can "extend" multiple configuration(s).

This allows a team that wants to have consistency (i.e. in how often they receive updates to the AWS and Google Cloud SDKs, or which labels they want on PRs) to create a shared preset for their team that defines this. Then, each of their repositories can "extend" this configuration, as well as defining their own configuration on top of it at a repo-specific level.

This also makes it possible to provide good guardrails for your organisation, providing a good set of defaults, such as a `base.json`:

```
{  "$schema": "https://docs.renovatebot.com/renovate-schema.json", "postUpdateOptions": [ "gomodTidy", "gomodUpdateImportPaths" ], "regexManagers": [ { "fileMatch": [ "^Makefile$" ], "matchStrings": [ "curl .*https://raw.githubusercontent.com/golangci/golangci-lint/master/install.sh | sh -s -- .* (?<currentValue>.*?)\\n" ], "depNameTemplate": "github.com/golangci/golangci-lint", "datasourceTemplate": "go" } ] } 
```

This then allows defining a somewhat opinionated good starting point for teams with a `default.json`:

```
{  "$schema": "https://docs.renovatebot.com/renovate-schema.json", "extends": [ "local>your-org-here/renovate-config:base", "config:best-practices" ] } 
```

Then, teams only need to set the following in their repos' `renovate.json`, and they'll have the benefits of onboarding with a lot of the hard work done with starting:

```
{  "$schema": "https://docs.renovatebot.com/renovate-schema.json", "extends": [ "local>your-org-here/renovate-config" ] } 
```

And that's it 👏

## Good defaults

Linking back to the comment about having to hand-craft `dependabot.yml`s, the great thing about Renovate is that there's some great defaults, and that it can autodetect the ecosystems your repo uses, and appropriately raises PRs.

This ease of onboarding is truly excellent, and you can even have [an onboarding PR](https://docs.renovatebot.com/getting-started/installing-onboarding/#repository-onboarding) raised to your repos to make it even simpler.

For instance, at Deliveroo, we made it so [there was a default set of configuration for all repos](https://www.jvt.me/posts/2023/01/30/renovate-global-defaults/) to give us an easy means to update Docker images, but then if teams wanted to manage everything, they could add a `renovate.json`, and it'd be more fully featured.

Additionally, Renovate comes with some great inbuilt configuration in the form of presets, including [a "best practices" guide](https://docs.renovatebot.com/upgrade-best-practices/) and associated preset, which makes it easier to keep on top of community best practices, without needing to bikeshed about what you think is best.

If you find that `config:best-practices` is a little too much, there's `config:recommended` as a starting point, and you can always downgrade or exclude rules you'd prefer not to follow. Or if you *really* want to control things `config:base` is the minimum you should pull in.

## Grouping

As suggested above, it's possible to tune how different packages get updated, where you can [group multiple updates into a single PR](https://docs.renovatebot.com/noise-reduction/#package-grouping).

For instance, let's say that you use 9 different AWS services in your application, and instead of receiving 9 PRs every time there are updates across the SDKs, you want a single one. In this case, you could craft the following Renovate configuration:

```
{  "$schema": "https://docs.renovatebot.com/renovate-schema.json", "packageRules": [ { "groupName": "aws-sdk-go", "matchPackagePatterns": [  "^github.com/aws/aws-sdk-go-v2" ] } ] } 
```

What's great about this is that you can also bundle things like all patch updates into a single PR:

```
{  "packageRules": [ { "matchPackagePatterns": [ "*" ], "matchUpdateTypes": [ "patch" ], "groupName": "all patch dependencies", "groupSlug": "all-patch" } ] } 
```

This is something that's [recently](https://github.blog/2023-08-24-a-faster-way-to-manage-version-updates-with-dependabot/) been added to Dependabot, which is great, cause Renovate's had it for years 😝

## Use for one-off bumps

Because Renovate is an Open Source tool you can run on the command-line, it means you can *also* get the ability to use [Renovate for one-off executions](https://www.jvt.me/posts/2022/12/12/renovate-one-off/), for instance to get everyone in your organisation to a minimum version of a given dependency, or just to do an infrequently performed set of updates.

Something I always refer back to is the fact that Renovate has a *tonne* of supported package managers, package ecosystems and versioning tools.

Renovate's [bot comparison guide docs](https://docs.renovatebot.com/bot-comparison/) has a good example which links out to the differences between:

As Dependabot is more focussed on being used on GitHub's platforms (citation needed?) support for things like CircleCI, GitLab CI or other competitors' tooling doesn't seem to be available.

## Adding support for additional ecosystems

One of the things you'll be used to having from your dependency update tool of choice is your standard package manager support, where it'll update a `go.mod` or a `pom.xml` in your repositories.

But one thing that's quite important to understand is that there are *many more* dependencies in your project than those installed in your package manager. Something I find great about Renovate is that, as well as managing `Dockerfile`s, `build.gradle`s, `.gitlab-ci.yml`, etc, it will *also* manage things your `.ruby-version` or configuration for the ASDF version manager.

But what about some of the non-standard, or organisation-specific means for tracking versions?

For instance, installing the `golangci-lint` linting tool recommends using `curl | sh`:

```
$(GOBIN)/golangci-lint:  curl -sSfL https://raw.githubusercontent.com/golangci/golangci-lint/master/install.sh | sh -s -- -b $(GOBIN) v1.57.2 
```

And it's common for `Dockerfile`s to have a definition of the version of dependencies:

```
ARG MONGODB_VERSION=6.0.4 
```

Or what if you have a custom deployment configuration like:

```
# this is an (imaginary) company specific deployment configuration file application:  lb: image:  uk-tooling/load-balancer@v3.0.0 
```

It's very unlikely that you have these supported by other tools out-of-the-box, and Renovate is the same.

But Renovate *does* give you the ability to manage these versions yourself.

Using Renovate's [custom managers](https://docs.renovatebot.com/modules/manager/regex/) we can craft a configuration file such as:

```
{  "$schema": "https://docs.renovatebot.com/renovate-schema.json", "regexManagers": [ { "fileMatch": [ "^Makefile$" ], "matchStrings": [ "curl .*https://raw.githubusercontent.com/golangci/golangci-lint/master/install.sh | sh -s -- .* (?<currentValue>.*?)\\n" ], "depNameTemplate": "github.com/golangci/golangci-lint", "datasourceTemplate": "go" } ] } 
```

And this will allow us to manage the `golangci-lint` installation.

Ideally, this sort of configuration may make it upstream so Renovate can do it out-of-the-box, but as we can see from the above example, there may be things that are organisation- or repo-specific, and so having it upstream'd doesn't make sense.

## Because you can do more with it

And you can do even more with it - I've mentioned you can use it for one-off updates on the command-line, but for instance I've also written a tool [renovate-graph](https://gitlab.com/tanna.dev/renovate-graph) which takes the detected dependency data from Renovate and gives you a JSON blob you can consume.

This underpins a large swathe of the power behind [dependency-management-data](https://dmd.tanna.dev) and can be used for other means, such as [converting Renovate data exports to a Software Bill of Materials (SBOM)](https://www.jvt.me/posts/2023/11/03/renovate-to-sbom/).

## Dependency Dashboard

One thing I love about Renovate is that you can enable the [Dependency Dashboard](https://docs.renovatebot.com/key-concepts/dashboard/), which gives you an overview of the detected dependencies, open PRs, as well s anything that may be waiting, or has failed to update.

This is a hugely useful insight into the at-a-glance how far behind are we on updates, giving a view of whether you maybe want to spend a bit more time focussing on updates, or looking at ways to cut through the noise.

When you merge a PR into your default branch, Renovate will rebase open PRs, so they're easier to review, and are guaranteed to run against the latest changes. However, if you're not getting to your updates as often as changes are going in, you may have a tonne of PRs constantly building, which is a waste of energy and CI minutes.

Instead, you can use the Dependency Dashboard for its ability to i.e. [require major bumps be gated behind a manual approval](https://docs.renovatebot.com/key-concepts/dashboard/#require-approval-for-major-updates), as it's likely you'll need some human interaction for that PR, and can then only raise it when you're actually ready to deal with it.

## Open Source

A very important factor to me is that Renovate is Open Source, and open to community contributions.

Although it's probably best to be split into another article, I love the AGPL3, and it's a great way of making sure that anyone hosting Renovate as a platform makes sure that their users get the access to the source code.

Alternatively, Snyk is proprietary, and ~~Dependabot is source available (or at a stretch "open source" not "Open Source") with [a license](https://github.com/dependabot/dependabot-core?tab=readme-ov-file#license) that doesn't even appear on the SPDX license list~~. **Update 2024-05-19**: Dependabot is now MIT-licensed 👏

Renovate is also brilliantly set up as a community project, where they're shipping hundreds of PRs a month, alongside managing the community really well. I've seen a few things change in the last few years I've been more actively contributing, and it's a really well run project and an indication of something I'd love to be able to replicate at some point!

It also helps that [Mend](https://mend.io), the company behind Renovate, invests a fair bit of time and money into development of the project, as well as their commercial offerings on top of it, which continue to make the project sustainable and remain Free and Open.

## Great documentation

Following on from the excellent community and maintainer contributions alike, there's also some really excellent work on a technical writing and documentation point of view, which makes a lot of tasks straightforward to solve.

If there's something a little more complex or custom than the docs can offer, it's usually something that can be answered by the community in a GitHub Discussion, and likely could turn into a docs improvement, if necessary.

There's even a great [comparison table between Renovate and other dependency update bots](https://docs.renovatebot.com/bot-comparison/), as well as [tips on how best to update your projects](https://docs.renovatebot.com/upgrade-best-practices/).

I've recently discovered a few pages and features that I wasn't aware, just by going through the docs.

## Overall

In summary, there's just so much that makes Renovate a greater choice than any of the alternatives, and I'm sure I could talk about more things that make it great.

Whether you self-host it for maximum control (such as being able to access internal artifact registries), or run the hosted app for ease of operations, or just run it from the command-line once in a while, it can be hugely useful to your experience as an engineer.

I hope you'll check it out!