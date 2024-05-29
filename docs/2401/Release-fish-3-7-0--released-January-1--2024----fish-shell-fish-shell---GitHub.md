<!--yml
category: 未分类
date: 2024-05-27 14:24:26
-->

# Release fish 3.7.0 (released January 1, 2024) · fish-shell/fish-shell · GitHub

> 来源：[https://github.com/fish-shell/fish-shell/releases/tag/3.7.0](https://github.com/fish-shell/fish-shell/releases/tag/3.7.0)

This release of fish includes a number of improvements over fish 3.6.4, detailed below. Although work continues on the porting of fish internals to the Rust programming language, that work is not included in this release. fish 3.7.0 and any future releases in the 3.7 series remain C++ programs.

## Notable improvements and fixes

*   Improvements to the history pager, including:
    *   The history pager will now also attempt subsequence matches ([#9476](https://github.com/fish-shell/fish-shell/issues/9476)), so you can find a command line like `git log 3.6.1..Integration_3.7.0` by searching for `gitInt`.
    *   Opening the history pager will now fill the search field with a search string if you’re already in a search ([#10005](https://github.com/fish-shell/fish-shell/issues/10005)). This makes it nicer to search something with `↑` and then later decide to switch to the full pager.
    *   Closing the history pager with enter will now copy the search text to the commandline if there was no match, so you can continue editing the command you tried to find right away ([#9934](https://github.com/fish-shell/fish-shell/issues/9934)).
*   Performance improvements for command completions and globbing, where supported by the operating system, especially on slow filesystems such as NFS ([#9891](https://github.com/fish-shell/fish-shell/issues/9891), [#9931](https://github.com/fish-shell/fish-shell/issues/9931), [#10032](https://github.com/fish-shell/fish-shell/issues/10032), [#10052](https://github.com/fish-shell/fish-shell/issues/10052)).
*   fish can now be configured to wait a specified amount of time for a multi-key sequence to be completed, instead of waiting indefinitely. For example, this makes binding `kj` to switching modes in vi mode possible.
    The timeout can be set via the new `fish_sequence_key_delay_ms` variable ([#7401](https://github.com/fish-shell/fish-shell/issues/7401)), and may be set by default in future versions.

## Deprecations and removed features

*   `LS_COLORS` is no longer set automatically by the `ls` function ([#10080](https://github.com/fish-shell/fish-shell/issues/10080)). Users
    that set `.dircolors` should manually import it using other means. Typically this would be `set -gx LS_COLORS (dircolors -c .dircolors | string split ' ')[3]`

## Scripting improvements

*   Running `exit` with a negative number no longer crashes fish ([#9659](https://github.com/fish-shell/fish-shell/issues/9659)).
*   `fish --command` will now return a non-zero status if parsing failed ([#9888](https://github.com/fish-shell/fish-shell/issues/9888)).
*   The `jobs` builtin will now escape the commands it prints ([#9808](https://github.com/fish-shell/fish-shell/issues/9808)).
*   `string repeat` no longer overflows if the count is a multiple of the chunk size ([#9900](https://github.com/fish-shell/fish-shell/issues/9900)).
*   The `builtin` builtin will now properly error out with invalid arguments instead of doing nothing and returning true ([#9942](https://github.com/fish-shell/fish-shell/issues/9942)).
*   `command time` in a pipeline is allowed again, as is `command and` and `command or` ([#9985](https://github.com/fish-shell/fish-shell/issues/9985)).
*   `exec` will now also apply variable overrides, so `FOO=bar exec` will now set `$FOO` correctly ([#9995](https://github.com/fish-shell/fish-shell/issues/9995)).
*   `umask` will now handle empty symbolic modes correctly, like `umask u=,g=rwx,o=` ([#10177](https://github.com/fish-shell/fish-shell/issues/10177)).
*   Improved error messages for errors occurring in command substitutions ([#10054](https://github.com/fish-shell/fish-shell/issues/10054)).

## Interactive improvements

*   `read` no longer enables bracketed paste so it doesn’t stay enabled in combined commandlines like `mysql -p(read --silent)` ([#8285](https://github.com/fish-shell/fish-shell/issues/8285)).
*   Vi mode now uses `fish_cursor_external` to set the cursor shape for external commands ([#4656](https://github.com/fish-shell/fish-shell/issues/4656)).
*   Opening the history search in vi mode switches to insert mode correctly ([#10141](https://github.com/fish-shell/fish-shell/issues/10141)).
*   Vi mode cursor shaping is now enabled in iTerm2 ([#9698](https://github.com/fish-shell/fish-shell/issues/9698)).
*   Working directory reporting is enabled for iTerm2 ([#9955](https://github.com/fish-shell/fish-shell/issues/9955)).
*   Completing commands as root includes commands not owned by root, fixing a regression introduced in fish 3.2.0 ([#9699](https://github.com/fish-shell/fish-shell/issues/9699)).
*   Selection uses `fish_color_selection` for the foreground and background colors, as intended, rather than just the background ([#9717](https://github.com/fish-shell/fish-shell/issues/9717)).
*   The completion pager will no longer sometimes skip the last entry when moving through a long list ([#9833](https://github.com/fish-shell/fish-shell/issues/9833)).
*   The interactive `history delete` interface now allows specifying index ranges like “1..5” ([#9736](https://github.com/fish-shell/fish-shell/issues/9736)), and `history delete --exact` now properly saves the history ([#10066](https://github.com/fish-shell/fish-shell/issues/10066)).
*   Command completion will now call the stock `manpath` on macOS, instead of a potential Homebrew version. This prevents awkward error messages ([#9817](https://github.com/fish-shell/fish-shell/issues/9817)).
*   A new bind function `history-pager-delete`, bound to `Shift` + `Delete` by default, will delete the currently-selected history pager item from history ([#9454](https://github.com/fish-shell/fish-shell/issues/9454)).
*   `fish_key_reader` will now use printable characters as-is, so pressing “ö” no longer leads to it telling you to bind `\\u00F6` ([#9986](https://github.com/fish-shell/fish-shell/issues/9986)).
*   `open` can be used to launch terminal programs again, as an `xdg-open` bug has been fixed and a workaround has been removed ([#10045](https://github.com/fish-shell/fish-shell/issues/10045)).
*   The `repaint-mode` binding will now only move the cursor if there is repainting to be done. This fixes `Alt` combination bindings in vi mode ([#7910](https://github.com/fish-shell/fish-shell/issues/7910)).
*   A new `clear-screen` bind function is used for `Ctrl` + `l` by default. This clears the screen and repaints the existing prompt at first,
    so it eliminates visible flicker unless the terminal is very slow ([#10044](https://github.com/fish-shell/fish-shell/issues/10044)).
*   The `alias` convenience function has better support for commands with unusual characters, like `+` ([#8720](https://github.com/fish-shell/fish-shell/issues/8720)).
*   A longstanding issue where items in the pager would sometimes display without proper formatting has been fixed ([#9617](https://github.com/fish-shell/fish-shell/issues/9617)).
*   The `Alt` + `l` binding, which lists the directory of the token under the cursor, correctly expands tilde (`~`) to the home directory ([#9954](https://github.com/fish-shell/fish-shell/issues/9954)).
*   Various fish utilities that use an external pager will now try a selection of common pagers if the `PAGER` environment variable is not set, or write the output to the screen without a pager if there is not one available ([#10074](https://github.com/fish-shell/fish-shell/issues/10074)).
*   Command-specific tab completions may now offer results whose first character is a period. For example, it is now possible to tab-complete `git add` for files with leading periods. The default file completions hide these files, unless the token itself has a leading period ([#3707](https://github.com/fish-shell/fish-shell/issues/3707)).

### Improved prompts

*   The default theme now only uses named colors, so it will track the terminal’s palette ([#9913](https://github.com/fish-shell/fish-shell/issues/9913)).
*   The Dracula theme has now been synced with upstream ([#9807](https://github.com/fish-shell/fish-shell/issues/9807)); use `fish_config` to re-apply it to pick up the changes.
*   `fish_vcs_prompt` now also supports fossil ([#9497](https://github.com/fish-shell/fish-shell/issues/9497)).
*   Prompts which display the working directory using the `prompt_pwd` function correctly display directories beginning with dashes ([#10169](https://github.com/fish-shell/fish-shell/issues/10169)).

### Completions

*   Added completions for:
*   The `zfs` completions no longer print errors about setting a read-only variable ([#9705](https://github.com/fish-shell/fish-shell/issues/9705)).
*   The `kitty` completions have been removed in favor of keeping them upstream ([#9750](https://github.com/fish-shell/fish-shell/issues/9750)).
*   `git` completions now support aliases that reference other aliases ([#9992](https://github.com/fish-shell/fish-shell/issues/9992)).
*   The `gw` and `gradlew` completions are loaded properly ([#10127](https://github.com/fish-shell/fish-shell/issues/10127)).
*   Improvements to many other completions.
*   Improvements to the manual page completion generator ([#9787](https://github.com/fish-shell/fish-shell/issues/9787), [#9814](https://github.com/fish-shell/fish-shell/issues/9814), [#9961](https://github.com/fish-shell/fish-shell/issues/9961)).

## Other improvements

*   Improvements and corrections to the documentation.
*   The Web-based configuration now uses a more readable style when printed, such as for a keybinding reference ([#9828](https://github.com/fish-shell/fish-shell/issues/9828)).
*   Updates to the German translations ([#9824](https://github.com/fish-shell/fish-shell/issues/9824)).
*   The colors of the Nord theme better match their official style ([#10168](https://github.com/fish-shell/fish-shell/issues/10168)).

## For distributors

*   The licensing information for some of the derived code distributed with fish was incomplete. Though the license information was present in the source distribution, it was not present in the documentation. This has been corrected ([#10162](https://github.com/fish-shell/fish-shell/issues/10162)).
*   The CMake configure step will now also look for libterminfo as an alternative name for libtinfo, as used in NetBSD curses ([#9794](https://github.com/fish-shell/fish-shell/issues/9794)).

* * *

*Download links: To download the source code for fish, we suggest the file named "fish-3.7.0.tar.xz". The file downloaded from "Source code (tar.gz)" will not build correctly. The SHA-256 sum of this file is `df1b7378b714f0690b285ed9e4e58afe270ac98dbc9ca5839589c1afcca33ab1`. A GPG signature from David Adam (key ID `0x7A67D962D88A709A`) is available as "fish-3.7.0.tar.xz.asc".*