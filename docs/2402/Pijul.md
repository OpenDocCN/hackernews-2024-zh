<!--yml
category: 未分类
date: 2024-05-27 15:03:54
-->

# Pijul

> 来源：[https://pijul.org/](https://pijul.org/)

Pijul is a free and open source (GPL2) **distributed version control system**. Its distinctive feature is to be based on a **theory of patches**, while still being fast and scalable. This makes it easy to learn and use, without any compromise on power or features.

#### Why?

##### Commutation

In Pijul, independent changes can be applied in any order without changing the result or the version's identifier. This makes Pijul significantly simpler than workflows using `git rebase` or `hg transplant`. Pijul has a branch-like feature called "channels", but these are not as important as in other systems. For example, so-called *feature branches* are often just changes in Pijul. Keeping your history clean is the default.

##### Merge correctness

Pijul guarantees a number of strong properties on merges. The most important one is that the **order** between lines is always preserved. This is unlike 3-way merge, which may sometimes shuffle lines around. When the order is unknown (for example in the case of concurrent edits), this is a *conflict*, which contrasts with systems with "automatic" or "no conflicts" merges.

##### First-class conflicts

In Pijul, conflicts are not modelled as a "failure to merge", but rather as the standard case. Specifically, conflicts happen between two changes, and are solved by one change. The resolution change solves the conflict between the same two changes, no matter if other changes have been made concurrently. Once solved, conflicts never come back.

##### Partial clones

Commutation makes it possible to clone only a small subset of a repository: indeed, one can only apply the changes related to that subset. Working on a partial clone produces changes that can readily be sent to the large repository.

#### Where to find it?