<!--yml
category: 未分类
date: 2024-05-27 13:28:43
-->

# Changelog-Driven Releases

> 来源：[https://mathieularose.com/changelog-driven-releases](https://mathieularose.com/changelog-driven-releases)

<main>

<hgroup>

# Changelog-Driven Releases

April 2024

</hgroup>

Ever released an untested version of your open-source project or forgot to update the version number? We've all been there. Managing releases for open-source projects can feel like a tedious, error-prone process, especially when done with a mix of manual and automated steps. This article introduces an approach that turns your changelog into the ultimate source of truth for releases, making your project's releases fully automated

## Changelog as the Release Source of Truth

Imagine a changelog that serves not just as a history, but also functions as a source of truth for releases. This is precisely the approach I adopted for [utt](https://github.com/larose/utt), an open source project I maintain. The changelog is kept in reverse chronological order, with the most recent version at the top. Here's a glimpse:

```
## 1.30 (2024-01-17)

  * Add 'per-task' CSV report type

## 1.29 (2021-01-16)

  * Show total time in summary section
  * Remove redundant "time" label from summary section
... 
```

### Unreleased Changes

New changes are added to the changelog in utt with a special `(unreleased)` marker. This allows grouping multiple changes before publishing them together.

Here's how it works:

```
## (unreleased)

  * Fix bug X
  * Enhance feature Y 
```

### Publishing a Release

When ready to publish, I simply replace `(unreleased)` with the desired version number and date:

```
## 1.31 (2024-04-20)

  * Fix bug X
  * Enhance feature Y 
```

Merging this change into the master branch triggers the automated release process in utt, including running automated tests.

## Implementation

While the specific implementation in utt uses Python and GitHub Actions (see [utt's workflow](https://github.com/larose/utt/blob/e31c7340cd4c151b7a5bfa004b676ca72149978f/.github/workflows/publish.yml)), the concepts are adaptable to most programming languages and CI/CD tools.

The core workflow involves a script that parses the changelog on push events to the master branch. If the latest entry is not `(unreleased)`, the script automatically updates the version number in relevant project files (e.g., pyproject.toml, version.txt).

Finally, if the version is not `(unreleased)`, the script attempts to publish the new version to PyPI, if it's not already on PyPI. If the version already exists, the script skips the publish step to prevent attempting the same version again.

</main>