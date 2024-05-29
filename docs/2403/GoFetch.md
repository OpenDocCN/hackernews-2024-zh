<!--yml
category: 未分类
date: 2024-05-29 12:33:07
-->

# GoFetch

> 来源：[https://gofetch.fail](https://gofetch.fail)

Constant-time programming is a paradigm that aims to harden code against side-channel attacks by ensuring that all operations take the same amount of time, regardless of their operands. In particular, constant-time code cannot contain secret-dependent branches, loops, or other control structures. Moreover, as the CPU caches different addresses with attacker-observable latency, constant-time code cannot mix data and addresses in any way and prohibits the use of secret-dependent memory accesses or array indices.

We show that even if a victim correctly separates data from addresses by following the constant-time paradigm, the DMP will generate secret-dependent memory access on the victim's behalf, resulting in variable-time code susceptible to our key-extraction attacks.