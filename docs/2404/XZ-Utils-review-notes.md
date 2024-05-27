<!--yml
category: 未分类
date: 2024-05-27 13:12:43
-->

# XZ Utils review notes

> 来源：[https://tukaani.org/xz-backdoor/review.html](https://tukaani.org/xz-backdoor/review.html)

### 2023-01

* * *

OK. Also, the VS project files aren’t present anymore.

* * *

* * *

* * *

* * *

* * *

He wanted to use liblzma’s internal headers in some tests and eventually I gave in. This required making those particular internal headers standalone in sense that they don’t include other internal headers.

* * *

The second commit was combined from my edits squashed into a single commit. The commit message was clearly edited by him, collecting the messages from my separate commits. For example, the way the bullet points are formatted doesn’t match the way I would have done them.

* * *

* * *

* * *

* * *

* * *

* * *

* * *

* * *

* * *

* * *

Likely OK. The Doxyfile was updated later again anyway.

* * *

* * *

The intent is good, fixing a very minor issue in a rarely-seen corner case. It had waited some time on a branch until I gave the permission to merge it. This introduced a regression that was fixed in [2023-11](#_2023_11).

* * *

The fix is good but commit message goes beyond overthinking. The doc simply was incorrect and that’s it.

* * *

### 2023-02

OK. These are the first commits that update the API docs.

* * *

* * *

* * *

* * *

* * *

OK (documentation updates).

* * *

OK (more documentation updates).

* * *

* * *

OK (even more documentation updates).

* * *

OK (documentation updates continue still).

The return value updates were good to do:

*   `lzma_properties_encode` should only return `LZMA_OK` or `LZMA_PROG_ERROR`. `LZMA_OPTIONS_ERROR` shouldn’t be possible although it was listed before as a possible value too.

*   `lzma_filter_flags_decode` can return also `LZMA_DATA_ERROR`.

* * *

* * *

### 2023-03

* * *

This series has intermediate steps that I didn’t like. The end result is good though (the last two commits are by me).

This series is identical to the two following commits in `v5.4`:

* * *

With the tweaks that were made later, this is OK enough.

* * *

I found and still find these useless additions to the API. He kept insisting on them so eventually I gave in. There’s nothing technically wrong, I just think they don’t improve readability.

* * *

`tuktest.h` had made these obsolete.

* * *

This is one of the supposedly suspicous commits due to the timezone. In reality Jia put it in my name by common agreement because I had done a significant portion of it.

* * *

OK. These will look different after the 5.6.1 release because pre-generated Doxygen output won’t be included in distribution tarballs anymore (not only to reduce the number of generated files but also to simplify the license questions of the source tarball).

* * *

* * *

* * *

* * *

* * *

* * *

* * *

* * *

* * *

* * *

### 2023-06

This is from [PR #51](https://github.com/tukaani-project/xz/pull/51). The commit message seems slightly wrong as it speaks about `ZLIB::ZLIB` but otherwise everything is fine.

* * *

Some genuine fixes were needed later but because ifunc support is now gone from the package, these commits aren’t relevant anymore.

* * *

These are my commits that Jia merged. Since ifunc support is now gone from the `master` branch, these commits aren’t relevant anymore.

* * *

* * *

* * *

OK. An inline function could have been better than a macro.

* * *

* * *

### 2023-07

OK. I think the CI setup with various build configs helped to spot this.

* * *

* * *

* * *

`--powerpc` had been abbreviated to `--power` by accident in the old version but it still worked correctly because `getopt_long` accepts unambiguous abbreviations.

* * *

OK. `str_to_filter` was renamed to more correct plural `str_to_filters` two commits later.

This commit begins the series of commits that add the `--filters1` …​ `--filters9` options.

* * *

* * *

It’s OK as an intermediate step. The next commits improve things.

* * *

* * *

* * *

I restored the `{ }` to one `if` statement. I like to keep the two if-statements in the nested form instead of using short-circuiting (`&&`) to make the function call conditional.

* * *

This does what it is supposed to do but it needed cleaning up:

*   `filters_memusage_max()` sets the static array `filter_memusages` but it’s better to make that array a local variable.

*   The second half is more complex than it needs to be. Splitting the dictionary size scaling into three steps makes it hard to read.

    *   `orig_dict_size` should be `uint32_t`, not `uint64_t`.

    *   The `while (count > 0)` loop and the need for the `count` variable are an odd way to do this. It adjusts the chains in parallel instead of handling one chain at a time.

    *   `if (filt_mem_usage < memory_limit)` should have had `<=`. In practice it didn’t matter though as the values are in bytes.

I edited this code quite a bit in 2024-05, including the fixes and cleanups for the things listed above. It might still be a bit heavy to read but it’s not due to the 2023 commits.

* * *

OK. Using multiple filter chains with `--flush-timeout` isn’t a likely use case but this isn’t much extra code either.

* * *

However, it’s better to collect the required information in `parse_block_list()` because it avoids the need to loop through the `opt_block_list` array. I did that change in 2024-05, thus not a lot remains from this commit.

This is the last commit to the `--filters1` …​ `--filters9` series.

* * *

* * *

* * *

* * *

* * *

* * *

OK. As the commit message says, it only reorders lines. The following can be helpful:

```
git log -p f5dc172a402f^! | sed 's/^-/+/' | grep ^+ \
    | sort | uniq -u
```

* * *

* * *

* * *

* * *

* * *

| Tip | Never tell how many things you are going to list. |

* * *

OK. We had been asked how to cross-compile on one machine but then run the tests on the target machine, so it was good to document it.

* * *

* * *

* * *

OK but it’s gone from the `master` now.

* * *

* * *

OK. It makes `test_files.sh` use exit status 77 (skipped test) when the feature isn’t configured instead of exit status 0 (passed test).

* * *

* * *

The special cases on Windows were researched and commented well but the end result is more complex than it needs to be. Instead of creating the symlinks in the build directory and then installing (copying) them, it’s simpler to create the symlinks at install time directly in the target directory. This was simplified in [CMake: Simplify symlink creation and install translated man pages.](https://github.com/tukaani-project/xz/commit/67610c245ba6c68cf65991693bab9312b7dc987b) which itself is a messy commit though as it did multiple things.

* * *

* * *

### 2023-09

* * *

* * *

These are OK, including getopt.m4 which was simplified a lot. xz doesn’t need support for all corner cases of getopt_long so checking for those corner cases isn’t needed either.

The GNU getopt_long isn’t used on GNU/Linux, *BSDs, Solaris, or MinGW-w64.

* * *

To verify these commits, the following can be helpful:

```
git diff ce162db07f03^..9fb5de41f2fb \
    | sed "/^-/{s/\`/'/g;s/^-/+/}" \
    | grep ^+ | sort | uniq -u
```

* * *

### 2023-10

It matches the file in Gnulib.

* * *

The final version is fine.

* * *

This attribute is obviously scary but it is unfortunately required with this version of the x86 SIMD code. The code makes aligned 16-byte reads which may read up to 15 bytes before the beginning or past the end of the buffer if the buffer is misaligned. The unneeded bytes are then ignored. It cannot cross page boundaries and thus cannot cause access violations.

The correctness is easy to review because memory is read only in a few places. If you do this, I suggest looking at the latest code in the `master` branch as that is the code that is actually in use now.

**However**, it also trips memory sanitizer which is a different thing than address sanitizer. Instead of adding another attribute to disable it, this code will be changed so that such attributes aren’t needed. Such a change won’t be backported to 5.4.x though because the current code does work correctly.

* * *

* * *

* * *

* * *

### 2023-11

* * *

It’s simplest to review this as a whole. It was a bit hilarious series of commits in the `master` branch as I had spotted a regression that was introduced in [xz: Refactor duplicated check for custom suffix when using --format=raw](https://github.com/tukaani-project/xz/commit/cc5aa9ab138beeecaee5a1e81197591893ee9ca0). We discussed it and gave ideas about the fix and then it turned out to be a bit wrong and needing another fix until he thought it’s better to finish it another day. Then the test script was added as well.

* * *

These are good although I clarified a comment in the next commit. It fixed a very old bug in Windows build of xz.

* * *

Ifunc detection was causing issues with musl. This seems fine but ifunc support was removed in [liblzma: Remove ifunc support.](https://github.com/tukaani-project/xz/commit/689ae2427342a2ea1206eb5ca08301baf410e7e0) so this doesn’t matter anymore anyway.

* * *

It really is just space change, no dots.

* * *

### 2023-12

These are from [PR #73](https://github.com/tukaani-project/xz/pull/73) and have been merged correctly with cleanups to commit messages and white space usage.

* * *

OK. Needed one minor comment fix.

* * *

* * *

* * *

OK. `options` definitely must not be `NULL`, and in practice it never is because it would be a bug in the application too.

* * *

OK, just like the commit message says.

* * *

The bug isn’t as serious as the commit message makes it sound as no one has a reason to call `lzma_filters_update` on a LZMA1 encoder, a function that is very rarely used anyway.

* * *

The order of the macros in the `#if` directive is different from src/xz/sandbox.h but the directive is correct still.

The Capsicum code is slightly simpler and stricter than in xz because xzdec needs to do less. Silently running without sandbox when kernel support is missing (`ENOSYS`) is an intentional feature and commonly accepted practice: without this, on such kernels xz wouldn’t run at all.

* * *

* * *

This moves code around and edits it a little. Comparing the moved sections shows it’s fine including the small edits. No suspicious typos (like mispelling `SANDBOX_COMPILE_DEFINITION`) are apparent.

* * *

### 2024-01

The `riscv.c` file in this commit was almost solely written by me and matched the file I had outside of the Git tree. Jia did some minor work on it like adding macros to avoid repeated code and a few comment improvements. Those I reviewed and edited further.

The rest of the commit was by Jia. Those changes are good.

* * *

* * *

This is the first commit preparing for the backdoor.

* * *

It’s very similar to the ARM64 code below the new code.

* * *

* * *

This is fine although now that the backdoor has been removed, this commit alone doesn’t do anything. But I left it there so that it’s ready *if* proper files along with a generator program are added.

* * *

I had mentioned the dictionary size and that gave a good excuse to update something in the backdoor code.

* * *

* * *

There is some churn in these commits so it’s simplest to review only the final outcome.

* * *

### 2024-02

* * *

* * *

These are good and were created at my request. They are big but it’s hard to split them into smaller pieces. The original versions of these are from 2023-04-24. I made small edits but it was agreed that I would commit these in his name.

Since these commits are complex, it’s understandable if they look scary. The new code does input buffer bounds checking only once per loop iteration, checking that there is at least 21 bytes available. At first this may look insufficient because, in the worst case, `rc_normalize` may be executed up to 48 times (22 bit model + 26 direct) per loop iteration. Each `rc_normalize` macro will read one input byte if needed. However, the condition to read a new byte cannot be true everytime. This has been verified both with math and experimentally. Comments in the code about this could be improved.

The small edits I made included fixing an off-by-one error with the loop condition. The commit originally had 20 but I changed it to 21 because the final normalization after the end of payload marker (EOPM) was included in the non-resumable path. The next commit by me ([liblzma: LZMA decoder improvements.](https://github.com/tukaani-project/xz/commit/e0c0ee475c0800c08291ae45e0d66aa00d5ce604)) made the non-resumable variant jump to the safe code (`goto eopm`) instead and thus 21 was changed to 20.

XZ Embedded has had somewhat comparable code with the per-loop 20-byte check since the beginning (2009) and it’s been in the Linux kernel since 2010.

As most of the core compression code in XZ Utils, the original idea for this detail comes from LZMA SDK. It’s just fairly different-looking code.

* * *

Modified build-to-host.m4 had the backdoor trigger. Ignoring the file here is correct because it is added among several other .m4 files when running `./autogen.sh` or `autoreconf -fi`.

* * *

* * *

* * *

I didn’t like the extra macro that had been added solely for this test. The last commit does more than reverting a single commit but the change is OK.

* * *

This test assumes that the encoder output doesn’t change, that is, the test will fail if the encoder is modified too much. If the encoder is modified, updating the test to match isn’t much extra to do.

* * *

Note that tests/test_files.sh uses globs to pick the files. So just adding files means that a decompression test will be done with them.

* * *

We discussed this and I agreed to it. This way man page translations didn’t need to go via translators in the last days of the release rush to fix a typo in the English source text.

* * *

The symbol name was supposed to be `XZ_5.6` not `XZ_5.6.0`. It makes no difference in practice as it is merely a string.

* * *

OK. He liked to run codespell while I didn’t.

* * *

He fixed it because I noticed it. Clearly the test file had been written some time ago already.

* * *

* * *

* * *

The need to add this fix had been discussed.

* * *

This was discussed in length with multiple people. The commit matches what was decided.

* * *

### 2024-03

This is [a real bug](https://gcc.gnu.org/bugzilla/show_bug.cgi?id=114115#c12) but the bug number in the commit message lacks one digit.

* * *

It’s one of the backdoor commits.

* * *

* * *

This is a continuation for [xz: Add missing RISC-V on the filter list in the man page](https://github.com/tukaani-project/xz/commit/eee579fff50099ba163c12305e81a4bd42b7dd53). Almost always it’s not good to touch the translations without upstream approval. In this case I felt mixed as I wondered if the date translations would be correct but didn’t want to stop it so that the translation could be in the 5.6.1 release that was going to be made the same day.

When translations are sent to the translators, they will remake these changes anyway (they only pick the original English text from the tarball).

* * *

I got the impression that a lot of speculation happened around this commit.

I had asked Jia to simplify the GitHub PR and Issue templates already, and simplifying `SECURITY.md` seemed reasonable too. People reporting such bugs don’t need detailed handholding.

I felt 21–30 days would be appropriate but Jia wanted to keep 90. With backdoor like “bugs”, fast full disclosure is the only correct way though.