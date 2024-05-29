<!--yml
category: 未分类
date: 2024-05-27 14:55:19
-->

# Erlang/OTP 27.0 Release Candidate 1 - Erlang/OTP

> 来源：[https://www.erlang.org/news/167](https://www.erlang.org/news/167)

## OTP 27.0-rc1 [#](#otp-270-rc1)

Erlang/OTP 27.0-rc1 is the first release candidate of three before the OTP 27.0 release.

The intention with this release is to get feedback from our users. All feedback is welcome, even if it is only to say that it works for you. We encourage users to try it out and give us feedback either by creating an issue at [https://github.com/erlang/otp/issues](https://github.com/erlang/otp/issues) or by posting to [Erlang Forums](https://erlangforums.com/).

All artifacts for the release can be downloaded from the [Erlang/OTP Github](https://github.com/erlang/otp/releases/tag/OTP-27.0-rc1) release and you can view the new documentation at [https://erlang.org/documentation/doc-15.0-rc1/doc](https://erlang.org/documentation/doc-15.0-rc1/doc/). You can also install the latest release using [kerl](https://github.com/kerl/kerl) like this:

```
kerl build 27.0-rc1 27.0-rc1. 
```

Erlang/OTP 27 is a new major release with new features, improvements as well as a few incompatibilities. Some of the new features are highlighted below.

Many thanks to all contributors!

# Highlights [#](#highlights)

## Documentation [#](#documentation)

[EEP-59](https://www.erlang.org/eeps/eep-0059) has been implemented. Documentation attributes in source files can now be used to document functions, types, callbacks, and modules.

The entire Erlang/OTP documentation is now using the new documentation system.

## New language features [#](#new-language-features)

*   Triple-Quoted Strings has been implemented as per [EEP 64](https://www.erlang.org/eeps/eep-0064) to allow a string to encompass a complete paragraph.

*   Adjacent string literals without intervening white space is now a syntax error, to avoid possible confusion with triple-quoted strings.

*   Sigils on string literals (both ordinary and triple-quoted) have been implemented as per [EEP 66](https://www.erlang.org/eeps/eep-0066). For example, `~"Björn"` or `~b"Björn"` are now equivalent to `<<"Björn"/utf8>>`.

## Compiler and JIT improvements [#](#compiler-and-jit-improvements)

*   The compiler will now merge consecutive updates of the same record.

*   Safe destructive update of tuples has been implemented in the compiler and runtime system. This allows the VM to update tuples in-place when it is safe to do so, thus improving performance by doing less copying but also by producing less garbage.

*   The `maybe` expression is now enabled by default, eliminating the need for enabling the `maybe_expr` feature.

*   Native coverage support has been implemented in the JIT. It will automatically be used by the `cover` tool to reduce the execution overhead when running cover-compiled code. There are also new APIs to support native coverage without using the `cover` tool.

*   The compiler will now raise a warning when updating record/map literals to catch a common mistake. For example, the compiler will now emit a warning for `#r{a=1}#r{b=2}`.

## ERTS [#](#erts)

*   The `erl` command now supports the `-S` flag, which is similar to the `-run` flag, but with some of the rough edges filed off.

*   By default, escripts will now be compiled instead of interpreted. That means that the `compiler` application must be installed.

*   The default process limit has been raised to `1048576` processes.

*   The `erlang:system_monitor/2` functionality is now able to monitor long message queues in the system.

*   The obsolete and undocumented support for opening a port to an external resource by passing an atom (or a string) as first argument to `open_port()`, implemented by the vanilla driver, has been removed. This feature has been scheduled for removal in OTP 27 since the release of OTP 26.

*   The `pid` field has been removed from `erlang:fun_info/1,2`.

*   Multiple trace sessions are now supported.

## STDLIB [#](#stdlib)

*   Several new functions that accept funs have been added to module `timer`.

*   The functions `is_equal/2`, `map/2`, and `filtermap/2` have been added to the modules `sets`, `ordsets`, and `gb_sets`.

*   There are new efficient `ets` traversal functions with guaranteed atomicity. For example, `ets:next/2` followed by `ets:lookup/2` can now be replaced with `ets:next_lookup/1`.

*   The new function `ets:update_element/4` is similar to `ets:update_element/3`, but takes a default tuple as the fourth argument, which will be inserted if no previous record with that key exists.

*   `binary:replace/3,4` now supports using a fun for supplying the replacement binary.

*   The new function `proc_lib:set_label/1` can be used to add a descriptive term to any process that does not have a registered name. The name will be shown by tools such as `c:i/0` and `observer`, and it will be included in crash reports produced by processes using `gen_server`, `gen_statem`, `gen_event`, and `gen_fsm`.

*   Added functions to retrieve the next higher or lower key/element from `gb_trees` and `gb_sets`, as well as returning iterators that start at given keys/elements.

## common_test [#](#common_test)

*   Calls to `ct:capture_start/0` and `ct:capture_stop/0` are now synchronous to ensure that all output is captured.

*   The default CSS will now include a basic dark mode handling if it is preferred by the browser.

## crypto [#](#crypto)

*   The functions `crypto_dyn_iv_init/3` and `crypto_dyn_iv_update/3` that were marked as deprecated in Erlang/OTP 25 have been removed.

## dialyzer [#](#dialyzer)

*   The `--gui` option for Dialyzer has been removed.

## ssl [#](#ssl)

*   The `ssl` client can negotiate and handle certificate status request (OCSP stapling support on the client side).

*   There is a new tool `tprof`, which combines the functionality of `eprof` and `cprof` under one interface. It also adds heap profiling.

## xmerl [#](#xmerl)

*   As an alternative to `xmerl_xml`, a new export module `xmerl_xml_indent` that provides out-of-the box indented output has been added.

For more details about new features and potential incompatibilities see the [README](https://erlang.org/download/otp_src_27.0-rc1.readme).