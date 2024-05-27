<!--yml
category: 未分类
date: 2024-05-27 13:22:49
-->

# Introduce BPF trampoline [LWN.net]

> 来源：[https://lwn.net/Articles/804937/](https://lwn.net/Articles/804937/)

| **From**: |   | Alexei Starovoitov <ast-AT-kernel.org> |
| **To**: |   | <davem-AT-davemloft.net> |
| **Subject**: |   | [PATCH v4 bpf-next 00/20] Introduce BPF trampoline |
| **Date**: |   | Thu, 14 Nov 2019 10:57:00 -0800 |
| **Message-ID**: |   | <20191114185720.1641606-1-ast@kernel.org> |
| **Cc**: |   | <daniel-AT-iogearbox.net>, <x86-AT-kernel.org>, <netdev-AT-vger.kernel.org>, <bpf-AT-vger.kernel.org>, <kernel-team-AT-fb.com> |
| **Archive-link**: |   | [Article](https://lwn.net/ml/netdev/20191114185720.1641606-1-ast@kernel.org/) |

```
Introduce BPF trampoline that works as a bridge between kernel functions, BPF
programs and other BPF programs.

The first use case is fentry/fexit BPF programs that are roughly equivalent to
kprobe/kretprobe. Unlike k[ret]probe there is practically zero overhead to call
a set of BPF programs before or after kernel function.

The second use case is heavily influenced by pain points in XDP development.
BPF trampoline allows attaching similar fentry/fexit BPF program to any
networking BPF program. It's now possible to see packets on input and output of
any XDP, TC, lwt, cgroup programs without disturbing them. This greatly helps
BPF-based network troubleshooting.

The third use case of BPF trampoline will be explored in the follow up patches.
The BPF trampoline will be used to dynamicly link BPF programs. It's more
generic mechanism than array and link list of programs used in tracing,
networking, cgroups. In many cases it can be used as a replacement for
bpf_tail_call-based program chaining. See [1] for long term design discussion.

v3->v4:
- Included Peter's
  "86/alternatives: Teach text_poke_bp() to emulate instructions" as a first patch.
  If it changes between now and merge window, I'll rebease to newer version.
  The patch is necessary to do s/text_poke/text_poke_bp/ in patch 3 to fix the race.
- In patch 4 fixed bpf_trampoline creation race spotted by Andrii.
- Added patch 15 that annotates prog->kern bpf context types. It made patches 16
  and 17 cleaner and more generic.
- Addressed Andrii's feedback in other patches.

v2->v3:
- Addressed Song's and Andrii's comments
- Fixed few minor bugs discovered while testing
- Added one more libbpf patch

v1->v2:
- Addressed Andrii's comments
- Added more test for fentry/fexit to kernel functions. Including stress test
  for maximum number of progs per trampoline.
- Fixed a race btf_resolve_helper_id()
- Added a patch to compare BTF types of functions arguments with actual types.
- Added support for attaching BPF program to another BPF program via trampoline
- Converted to use text_poke() API. That's the only viable mechanism to
  implement BPF-to-BPF attach. BPF-to-kernel attach can be refactored to use
  register_ftrace_direct() whenever it's available.

[1]
https://lore.kernel.org/bpf/20191112025112.bhzmrrh2pr76ss...

Alexei Starovoitov (19):
  bpf: refactor x86 JIT into helpers
  bpf: Add bpf_arch_text_poke() helper
  bpf: Introduce BPF trampoline
  libbpf: Introduce btf__find_by_name_kind()
  libbpf: Add support to attach to fentry/fexit tracing progs
  selftest/bpf: Simple test for fentry/fexit
  bpf: Add kernel test functions for fentry testing
  selftests/bpf: Add test for BPF trampoline
  selftests/bpf: Add fexit tests for BPF trampoline
  selftests/bpf: Add combined fentry/fexit test
  selftests/bpf: Add stress test for maximum number of progs
  bpf: Reserve space for BPF trampoline in BPF programs
  bpf: Fix race in btf_resolve_helper_id()
  bpf: Annotate context types
  bpf: Compare BTF types of functions arguments with actual types
  bpf: Support attaching tracing BPF program to other BPF programs
  libbpf: Add support for attaching BPF programs to other BPF programs
  selftests/bpf: Extend test_pkt_access test
  selftests/bpf: Add a test for attaching BPF prog to another BPF prog
    and subprog

Peter Zijlstra (1):
  x86/alternatives: Teach text_poke_bp() to emulate instructions

 arch/x86/include/asm/text-patching.h          |  24 +-
 arch/x86/kernel/alternative.c                 | 132 ++++--
 arch/x86/kernel/jump_label.c                  |   9 +-
 arch/x86/kernel/kprobes/opt.c                 |  11 +-
 arch/x86/net/bpf_jit_comp.c                   | 424 +++++++++++++++---
 include/linux/bpf.h                           | 138 +++++-
 include/linux/bpf_types.h                     |  78 ++--
 include/linux/bpf_verifier.h                  |   1 +
 include/linux/btf.h                           |   1 +
 include/uapi/linux/bpf.h                      |   3 +
 kernel/bpf/Makefile                           |   1 +
 kernel/bpf/btf.c                              | 363 ++++++++++++++-
 kernel/bpf/core.c                             |   9 +
 kernel/bpf/syscall.c                          |  77 +++-
 kernel/bpf/trampoline.c                       | 253 +++++++++++
 kernel/bpf/verifier.c                         | 131 +++++-
 net/bpf/test_run.c                            |  41 ++
 net/core/filter.c                             |  12 +-
 tools/include/uapi/linux/bpf.h                |   3 +
 tools/lib/bpf/bpf.c                           |   8 +-
 tools/lib/bpf/bpf.h                           |   5 +-
 tools/lib/bpf/bpf_helpers.h                   |  13 +
 tools/lib/bpf/btf.c                           |  22 +
 tools/lib/bpf/btf.h                           |   2 +
 tools/lib/bpf/libbpf.c                        | 154 +++++--
 tools/lib/bpf/libbpf.h                        |   7 +-
 tools/lib/bpf/libbpf.map                      |   3 +
 .../selftests/bpf/prog_tests/fentry_fexit.c   |  90 ++++
 .../selftests/bpf/prog_tests/fentry_test.c    |  64 +++
 .../selftests/bpf/prog_tests/fexit_bpf2bpf.c  |  76 ++++
 .../selftests/bpf/prog_tests/fexit_stress.c   |  76 ++++
 .../selftests/bpf/prog_tests/fexit_test.c     |  64 +++
 .../selftests/bpf/prog_tests/kfree_skb.c      |  39 +-
 .../testing/selftests/bpf/progs/fentry_test.c |  90 ++++
 .../selftests/bpf/progs/fexit_bpf2bpf.c       |  91 ++++
 .../testing/selftests/bpf/progs/fexit_test.c  |  98 ++++
 tools/testing/selftests/bpf/progs/kfree_skb.c |  52 +++
 .../selftests/bpf/progs/test_pkt_access.c     |  38 +-
 38 files changed, 2484 insertions(+), 219 deletions(-)
 create mode 100644 kernel/bpf/trampoline.c
 create mode 100644 tools/testing/selftests/bpf/prog_tests/fentry_fexit.c
 create mode 100644 tools/testing/selftests/bpf/prog_tests/fentry_test.c
 create mode 100644 tools/testing/selftests/bpf/prog_tests/fexit_bpf2bpf.c
 create mode 100644 tools/testing/selftests/bpf/prog_tests/fexit_stress.c
 create mode 100644 tools/testing/selftests/bpf/prog_tests/fexit_test.c
 create mode 100644 tools/testing/selftests/bpf/progs/fentry_test.c
 create mode 100644 tools/testing/selftests/bpf/progs/fexit_bpf2bpf.c
 create mode 100644 tools/testing/selftests/bpf/progs/fexit_test.c

-- 
2.23.0

```