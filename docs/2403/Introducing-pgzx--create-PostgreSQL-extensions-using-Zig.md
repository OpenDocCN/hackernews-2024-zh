<!--yml

类别：未分类

日期：2024-05-29 12:33:11

-->

# 介绍 pgzx：使用 Zig 创建 PostgreSQL 扩展

> 来源：[https://xata.io/blog/introducing-pgzx](https://xata.io/blog/introducing-pgzx)

作为 [Xata 发布周](https://xata.io/launch-week-unleash-the-elephant) 的一部分，我们推出了 [pgzx](https://github.com/xataio/pgzx)，这是一个用 Zig 编写的开源框架，用于开发 PostgreSQL 扩展。它提供一组实用工具（例如错误处理、内存分配器、包装器），以及简化与 Postgres 代码库集成的构建和开发环境。

[Zig](https://ziglang.org/) 在其网站上被描述为用于维护**健壮**、**优化**和**可重用**软件的通用编程语言和工具链。Zig 是一种新语言（在 1.0 版之前），但它在系统编程社区中已经越来越受欢迎。可以将其视为 "现代 C"，提供更安全的内存管理、编译时代码执行（comptime）和丰富的标准库。我们将在下面的 Postgres 扩展的上下文中展示其中一些特性。

使用 Zig 进行 Postgres 开发的一个主要原因是它可以与 C 代码互操作。Zig 支持 C ABI，与 C 指针和数据类型（包括空终止字符串）一起工作，甚至可以将 C 头文件翻译为 Zig 代码。虽然 Zig 自动将 C 宏翻译为 Zig 代码的功能还不完善，但仍然非常有帮助。这使得 Zig 成为处理大型 C 代码库（比如 Postgres）的优秀选择。

因为 Zig 可以调用任何 C 函数并将 C 宏转换为内联 Zig 函数，因此您可以按照创建 C 扩展的相同步骤在 Zig 中编写 Postgres 扩展，而无需任何额外的工具支持。然而，像 pgzx 这样的框架通过提供开发环境、一组 Postgres API 的实用工具和包装器、常见的错误处理等显著简化了这一过程。

pgzx 目前有 2 个示例扩展，供您参考。[char_count_zig](https://github.com/xataio/pgzx/tree/main/examples/char_count_zig) 是一个最小的扩展，而 [pg_audit_zig](https://github.com/xataio/pgzx/tree/main/examples/pgaudit_zig) 则更为复杂，展示了 pgzx 的更多特性。

让我们来看看 `char_count_zig`，它略微超过了 Postgres 扩展的 "Hello, World!"。它添加了一个函数，用于计算字符串中字符出现的次数。这受到了 [此教程](https://www.highgo.ca/2019/10/01/a-guide-to-create-user-defined-extension-modules-to-postgres/) 的启发，该教程展示了如何在 plpqsql 和 C 中实现这一功能。

```
select char_count_zig('hi hii', 'i');
 char_count_zig
----------------
 3
(1 row) 
```

这是用 Zig 和 C 编写的 `char_count` 扩展。

用 Zig 编写的 char_count

```
const std = @import("std");
const pgzx = @import("pgzx");
 comptime {
 pgzx.PG_MODULE_MAGIC();
 pgzx.PG_FUNCTION_V1("char_count_zig", char_count_zig);
}
 fn char_count_zig(input_text: []const u8, target_char: []const u8) !u32 {
 if (target_char.len > 1) {
 return pgzx.elog.Error(@src(), "Target char is more than one byte", .{});
 }
 pgzx.elog.Info(@src(), "input_text: {s}\n", .{input_text});
 pgzx.elog.Info(@src(), "target_char: {s}\n", .{target_char});
 pgzx.elog.Info(@src(), "Target char len: {}\n", .{target_char.len});
 var count: u32 = 0;
 for (input_text) |char| {
 if (char == target_char[0]) {
 count += 1;
 }
 }
 return count;
} 
```

用 C 编写的 char_count

```
#include "postgres.h"
#include "fmgr.h"
#include "utils/builtins.h"
 PG_MODULE_MAGIC;
 PG_FUNCTION_INFO_V1(char_count_c);
 Datum
char_count_c(PG_FUNCTION_ARGS)
{
 int charCount = 0;
 int i = 0;
 text * inputText = PG_GETARG_TEXT_PP(0);
 text * targetChar = PG_GETARG_TEXT_PP(1);
 int inputText_sz = VARSIZE(inputText)-VARHDRSZ;
 int targetChar_sz = VARSIZE(targetChar)-VARHDRSZ;
 char * cp_inputText = NULL;
 char * cp_targetChar = NULL;
 if (targetChar_sz > 1 )
 {
 elog(ERROR, "arg1 must be 1 char long");
 }
 cp_inputText = (char *) palloc(inputText_sz + 1);
 cp_targetChar = (char *) palloc(targetChar_sz + 1);
 memcpy(cp_inputText, VARDATA(inputText), inputText_sz);
 memcpy(cp_targetChar, VARDATA(targetChar), targetChar_sz);
 elog(INFO, "arg0 length is %d, value %s", (int)strlen(cp_inputText), cp_inputText);
 elog(INFO, "arg1 length is %d, value %s", (int)strlen(cp_targetChar), cp_targetChar);
 while ( i < strlen(cp_inputText) )
 {
 if( cp_inputText[i] == cp_targetChar[0] )
 charCount++;
 i++;
 }
 pfree(cp_inputText);
 pfree(cp_targetChar);
 PG_RETURN_INT32(charCount);
} 
```

尽管它们很相似，但 Zig 版本实际上更加简洁。部分原因是 Zig 更具表达性，但也因为 pgzx 处理了更多的工作。

注册的 SQL 函数接收序列化的参数，并且需要一些代码来对它们进行反序列化。在 C 语言中，你可以通过 `G_GETARG_*` 宏来实现，但是在 pgzx 中，你可以直接接收已经反序列化的普通参数。如何实现的呢？通过一个 `comptime` 函数，在编译时生成必要的样板代码。如果你感兴趣，可以查看 [pgCall](https://github.com/xataio/pgzx/blob/9825dde752ee4ace2cbc594332eba100292aafd5/src/pgzx/fmgr.zig#L77-L102) 函数，它是 Zig `comptime` 执行力量的一个很好的例子。

`PG_MODULE_MAGIC` 和 `PG_FUNCTION_INFO_V1` 函数是 `comptime` 使用的第二个例子。它们导出了必要的符号，以便 Postgres 可以识别这个扩展并将函数注册为 SQL 函数。在这种情况下，`comptime` 的作用几乎与对应的 C 宏相似。

如果你仔细观察上面的代码，你可能已经注意到它包含一个 bug。它检查 `target_char` 不应该超过 1 个字符，但却没有检查是否有 0 个字符。稍后，代码访问 `target_char[0]`，因此如果字符串是空字符串，将会出现越界访问错误。我们故意留下这个 bug，让你看看当扩展中出现这种 bug 时会发生什么。

你可以使用这个 SQL 触发 bug：

```
select char_count_zig('hi hii', ''); 
```

它的回应是：

```
server closed the connection unexpectedly
 This probably means the server terminated abnormally
 before or while processing the request.
The connection to the server was lost. Attempting reset: Failed. 
```

在 C 代码中，这种类型的 bug 可能会触发段错误，甚至是安全漏洞。如果你尝试使用 `char_count_zig` 扩展来执行此操作，Postgres 进程仍然会崩溃（不是整个服务器，只是处理连接的进程），但如果你查看日志，你会看到类似这样的错误消息：

```
thread 70501513 panic: index out of bounds: index 0, len 0
/Users/tsg/src/xataio/pgzx/examples/char_count_zig/src/main.zig:21:32: 0x103aaedff in char_count_zig (char_count_zig)
 if (char == target_char[0]) {
 ^
/Users/tsg/src/xataio/pgzx/src/pgzx/fmgr.zig:95:5: 0x103aaf20f in call (char_count_zig)
 const value = @call(.no_async, impl, callArgs) catch |e| elog.throwAsPostgresError(src, e);
 ^
???:?:?: 0x10316045b in _ExecInterpExpr (???)
???:?:?: 0x10315fbef in _ExecInterpExprStillValid (???)
???:?:?: 0x10326ceef in _evaluate_expr (???)
???:?:?: 0x10326da67 in _simplify_function (???)
???:?:?: 0x10326bacf in _eval_const_expressions_mutator (???) 
```

它确切地指出错误发生的位置！这是因为 Zig 根据[构建模式](https://ziglang.org/documentation/master/#Build-Mode)具有运行时检查。例如，`ReleaseSafe` 模式为更多的安全检查牺牲了一些性能。

注意，这个堆栈跟踪之所以这么有效，是因为错误出现在 Zig 代码中。在构建 Postgres 扩展时，你经常需要调用 Postgres 的 API，如果使用不正确，仍然会导致段错误。

Postgres 使用[分配器区域](https://en.wikipedia.org/wiki/Region-based_memory_management)来管理内存。在 Postgres 源代码中，这些区域被称为[内存上下文](https://github.com/postgres/postgres/blob/master/src/backend/utils/mmgr/README)。在一个“上下文”中分配的内存可以一次性释放（例如，当查询执行完成时），这显著简化了内存管理，因为你只需要跟踪上下文，而不是单个分配。上下文还是层次化的，因此你可以创建一个是另一个上下文子级的上下文，当父上下文被释放时，所有子上下文也会被释放。这使得内存泄漏变得不太可能。

另一个内存上下文的优点是它们改善了[内存监控](https://www.postgresql.org/docs/current/view-pg-backend-memory-contexts.html)，因为上下文有名称，您可以看到每个上下文使用了多少内存。这对于调试大内存使用非常有用。

使用arena/context分配器的这种模型恰好非常适合Zig。一个原因是Zig的约定是，所有分配内存的函数/对象都接收一个分配器作为参数。这使得分配更加[明确](https://notes.eatonphil.com/2024-03-15-zig-rust-and-other-languages.html)，同时也使得可以使用自定义分配器变得容易。pgzx定义了包装Postgres内存上下文的自定义分配器，并使其可用于Zig代码。

下面是一个示例，创建一个新的上下文作为当前上下文的子上下文，并获取其分配器：

```
var memctx = try pgzx.mem.createAllocSetContext("zig_context", .{ .parent = pg.CurrentMemoryContext });
const allocator = memctx.allocator(); 
```

在更复杂的扩展中，您很可能还需要了解另一个Postgres API，即错误处理。Postgres通过`setjmp/longjmp`在C中实现了"异常"，并提供了一组宏来抛出和捕获它们（[PG_TRY/PG_CATCH](https://github.com/postgres/postgres/blob/master/src/include/utils/elog.h#L318)）。

问题在于长跳跃可能会绕过Zig控制流，例如`errdefer`块可能不会被执行。这意味着，如果您的扩展调用Postgres API，并且这些API可能会引发错误，则长跳跃可能会跳过您的`defer`和`errdefer`块！

幸运的是，pgzx在此处提供帮助。它提供了一组函数，允许您捕获Postgres异常并将其转换为Zig错误。例如：

```
var errctx = pgzx.err.Context.init();
defer errctx.deinit();
if (errctx.pg_try()) {
 // Call Postgres C functions.
} else {
 return errctx.errorValue();
} 
```

pgzx提供了一个基于[Nix flakes](https://nixos.wiki/wiki/Flakes)的开发环境，用于开发扩展以及pgzx本身。它还提供了一个项目模板，您可以在新的存储库中使用它来设置这个环境。要使用它，[安装Nix](https://github.com/DeterminateSystems/nix-installer)，然后运行：

```
mkdir my_extension
cd my_extension
nix flake init -t github:xataio/pgzx 
```

然后加载nix shell：

开发环境包括用于在开发环境中重新定位Postgres二进制文件、启动服务器等的命令。该模板还带有一个最小的扩展和一个带有几个常见任务的`build.zig`文件。查看模板的[README](https://github.com/xataio/pgzx/tree/main/nix/templates/init)了解如何从这一点开始构建扩展。

Postgres扩展通常通过称为`pg_regress`的工具进行测试。pgzx也支持这一点，只需调用`zig build pg_regress`。

但我们也希望进行单元测试。这有点棘手，因为测试需要在**Postgres实例的上下文中**编译和运行。否则，它们将无法与Postgres的API进行交互。

为了解决这个问题，pgzx 注册了一个自定义的 `run_tests` 函数。这个函数可以从 SQL 中调用（`SELECT run_tests();`），它将运行单元测试。测试套件是一个 Zig 结构，其中的函数以 `test` 开头。要注册一个测试套件，你通常会像这样做：

```
comptime {
 pgzx.testing.registerTests(@import("build_options").testfn, .{Tests});
} 
```

[registerTests](https://github.com/xataio/pgzx/blob/9825dde752ee4ace2cbc594332eba100292aafd5/src/pgzx/testing.zig#L39) 函数是 `comptime` 使用的另一个示例。它遍历结构体的所有字段，并在 SQL 中调用 `run_tests()` 函数时生成调用以运行测试。

这个演示覆盖了 pgzx 提供的一些更有趣的功能，但还有更多：Postgres 数据结构的包装器（[lists](https://xataio.github.io/pgzx/#A;pgzx:list)，哈希表），[SPI](https://xataio.github.io/pgzx/#A;pgzx:spi)，[共享内存访问](https://xataio.github.io/pgzx/#A;pgzx:shmem)，[连接管理](https://xataio.github.io/pgzx/#A;pgzx:pq)等等。

在 Xata，我们一直在进行一个新的 Postgres 项目，目前还没有一个确切的名称，所以让我们称之为“Xata 对分布式 Postgres 的看法”。目前还处于非常早期阶段，但我们很快将其开源，并计划[公开构建](https://mailchi.mp/xata/2zoy27tx2e)。它将与 Citus 有些相似，但在使用和开发体验上会有一些不同的选择，这些选择基于我们过去几年运行 Xata 的经验。

对于这个项目，我们考虑了三种潜在的实现方向：

1.  就像 Vitess 对 MySQL 所做的外部代理一样。

1.  就像 Citus 对 Postgres 的扩展一样。

1.  就像 Greenplum 对 Postgres 的分支一样。

我们对第一个选项有一些经验，因为这就是我们当前的 Xata 代理工作的方式，更多详细信息请查看这篇[博客文章](https://xata.io/blog/serverless-postgres-platform)，因此我们几乎偏向于它。我们知道我们可以使用 Postgres 线协议进行通信，解析传入的查询并深入理解它们，还可以通过 2PC 创建分布式事务。然而，我们也知道利用现有的 Postgres 代码会开放更多选择，并避免重新实现一些非常困难的部分。考虑到项目的长期愿景，并且我们不想维护一个分支，我们决定选择第二个选项。

我们决策树的下一个节点是**编程语言**。我们考虑的主要选项是 C、Rust 和 Zig。虽然每种选项都有其利弊，我们可能会在未来的博客文章中详细讨论，但我们决定选择 Zig。Zig 的主要优势在于：

+   它几乎允许我们直接调用 Postgres 的 API，因此我们拥有使用 C 语言相同的强大能力。

+   它比 C 语言提供了更多的内存安全性，更富表现力，我们认为更有趣。

+   它与 Postgres 代码库非常契合，例如在内存管理和字符串处理方面。

在我们开始工作时，我们意识到Rust的[pgrx](https://github.com/pgcentralfoundation/pgrx)需要在Zig中也有一个等效的，因此我们启动了pgzx。

尽管Zig和pgzx可能不是每个Postgres扩展的最佳选择，但我们认为对于我们的项目以及其他一些项目来说，这是一个合理的选择。

就目前而言，pgzx还很新，应被视为“alpha”版。然而，如果你想构建一个Postgres扩展，并且想使用Zig，使用pgzx要比不使用它容易得多。已覆盖功能的状态在[README](https://github.com/xataio/pgzx)中。如果需要帮助或想要贡献，请加入我们的[Xata Discord](https://xata.io/discord)。

此外，如果这篇博文激发了你对Zig的兴趣，并且你想试一试，为什么不开发一个Postgres扩展呢？告诉我们，我们将把它包含在示例列表中！

想要了解这个项目的发展历程？听听那些参与其中的人说说，还可以观看一个快速演示看看它是如何运作的。在这里查看我们最新的见面会：

想要关注分布式Postgres项目pgzx，或者其他来自Xata的开源项目吗？我们专门设置了一个邮件通讯，可以在[这里](https://mailchi.mp/xata/2zoy27tx2e)订阅。
