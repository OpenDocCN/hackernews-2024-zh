<!--yml
category: 未分类
date: 2024-05-27 13:29:13
-->

# Borrow checking, RC, GC, and the Eleven (!) Other Memory Safety Approaches

> 来源：[https://verdagon.dev/grimoire/grimoire](https://verdagon.dev/grimoire/grimoire)

**13: Constraint references** is a blend of reference counting and single ownership (in the C++ sense, unrelated to borrow checking). In this approach, every object has a single owner, doesn't necessarily need to be on the heap, and has a counter for all references to it. When we try to destroy the object, we just assert that there are no other references to this object.

This is used surprisingly often. Some [game developers](https://discord.com/channels/402956206529970177/402956206529970180/451828861861101569) have been using this for a long time, and it can be used as the memory safety model for an entire language like in [Gel](https://web.archive.org/web/20220111001720/https://researcher.watson.ibm.com/researcher/files/us-bacon/Dingle07Ownership.pdf). It supports a lot more patterns than borrow checking (intrusive data structures, graphs, observers, back-references, dependency references, callbacks, delegates, many forms of RAII, etc).

However, this checking is at run-time. Halting in release mode is often undesirable, so this technique shines the most when it's very targeted or when we can fall back to a different strategy in release mode.