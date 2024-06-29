<!--yml

category: 未分类

日期：2024-05-27 14:41:22

-->

# Go 1.22 发行说明 - Go 编程语言

> 来源：[https://go.dev/doc/go1.22](https://go.dev/doc/go1.22)

# Go 1.22 发行说明

## Go 1.22 简介

最新的 Go 发行版，版本 1.22，在[Go 1.21](/doc/go1.21)之后六个月内发布。其大部分变化都在工具链、运行时和库的实现中。如往常一样，此版本保持了 Go 1 的[兼容性承诺](/doc/go1compat)。我们预计几乎所有的 Go 程序都将继续像以前一样编译和运行。

## 语言的变更

Go 1.22 对“for”循环进行了两个更改。

+   在 Go 1.22 中，“for”循环声明的变量以前是在每次迭代时创建一次并更新。现在每次循环迭代都会创建新变量，以避免意外共享错误。提议中描述的[过渡支持工具](/wiki/LoopvarExperiment#my-test-fails-with-the-change-how-can-i-debug-it)在 Go 1.21 中的工作方式保持不变。

+   现在，“For”循环可以在整数上进行遍历。例如，[示例](/play/p/ky02zZxgk_r?v=gotip)：

    ```
    package main

    import "fmt"

    func main() {
      for i := range 10 {
        fmt.Println(10 - i)
      }
      fmt.Println("go1.22 has lift-off!")
    } 
    ```

    参见[详情](/ref/spec#For_range)规范。

Go 1.22 包含我们考虑在未来版本中引入的语言更改的预览：[range-over-function iterators](/wiki/RangefuncExperiment)。使用 `GOEXPERIMENT=rangefunc` 可启用此功能。

### Go 命令

[工作区](/ref/mod#workspaces)中的命令现在可以使用包含工作区依赖项的 `vendor` 目录。该目录由 [`go` `work` `vendor`](/pkg/cmd/go#hdr-Make_vendored_copy_of_dependencies) 创建，并在设置 `-mod` 标志为 `vendor` 时由构建命令使用，这是工作区存在 `vendor` 目录时的默认设置。

请注意，工作区 `vendor` 目录的内容与单个模块的内容不同：如果工作区根目录也包含工作区中一个模块，则其 `vendor` 目录可以包含工作区或模块的依赖项，但不能同时包含两者。

`go` `get` 不再支持在传统的 `GOPATH` 模式之外使用模块。其他构建命令，如 `go` `build` 和 `go` `test`，将继续无限期地支持传统 `GOPATH` 程序。

`go` `mod` `init` 不再尝试从其他供应工具（如 `Gopkg.lock`）的配置文件中导入模块要求。

`go` `test` `-cover` 现在为没有自己的测试文件的覆盖包打印覆盖摘要。在 Go 1.22 之前，对这样的包运行 `go` `test` `-cover` 会报告

`? mymod/mypack [无测试文件]`

现在，Go 1.22 中包中的函数被视为未覆盖：

`mymod/mypack 覆盖率：0.0% 的语句`

注意，如果一个包根本没有可执行代码，我们无法报告有意义的覆盖百分比；对于这样的包，`go` 工具将继续报告没有测试文件。

现在，调用链接器的 `go` 构建命令如果将使用外部（C）链接器但未启用 cgo，将会报错。（Go 运行时需要 cgo 支持以确保其与任何通过 C 链接器添加的附加库兼容。）

### 跟踪

`trace` 工具的 Web 用户界面在支持新跟踪器的工作中已经轻微刷新，解决了几个问题，并提高了各个子页面的可读性。Web 用户界面现在支持以线程导向视图探索跟踪。跟踪查看器还显示所有系统调用的完整持续时间。

这些改进仅适用于使用 Go 1.22 或更新版本构建的程序生成的跟踪。未来的版本将把其中一些改进应用到使用旧版本 Go 生成的跟踪中。

### `vet` 

#### 循环变量的引用

`vet` 工具的行为已更改，以匹配 Go 1.22 中循环变量的新语义（参见上文）。在分析需要 Go 1.22 或更新版本的文件（由其 go.mod 文件或每个文件的构建约束决定）时，`vet` 不再报告来自函数文字的可能会超出循环迭代的循环变量引用。在 Go 1.22 中，循环变量每次迭代时都会重新创建，因此这样的引用不再有可能在更新循环变量后使用它们。

#### 追加后缺少值的新警告

`vet` 工具现在会报告对 [`append`](/pkg/builtin/#append) 的调用，这些调用未传递要追加到切片的值，例如 `slice = append(slice)`。这种语句没有效果，经验表明这几乎总是一个错误。

#### 延迟 `time.Since` 的新警告

`vet` 工具现在会报告在 `defer` 语句中对 [`time.Since(t)`](/pkg/time/#Since) 的非延迟调用。这相当于在 `defer` 语句之前调用 `time.Now().Sub(t)`，而不是在延迟函数被调用时。几乎所有情况下，正确的代码都需要延迟 `time.Since` 的调用。例如：

```
t := time.Now()
defer log.Println(time.Since(t)) // non-deferred call to time.Since
tmp := time.Since(t); defer log.Println(tmp) // equivalent to the previous defer

defer func() {
  log.Println(time.Since(t)) // a correctly deferred call to time.Since
}() 
```

#### 在 `log/slog` 调用中的键值对不匹配时会有新的警告

`vet` 工具现在会报告在结构化日志包 [`log/slog`](/pkg/log/slog) 中对函数和方法调用的无效参数，这些调用接受交替的键/值对。它会报告那些在键位置的参数既不是 `string` 也不是 `slog.Attr`，以及缺少值的最后一个键的调用。

## 运行时

现在运行时将基于类型的垃圾收集元数据保持在每个堆对象附近，从而提高了 Go 程序的 CPU 性能（延迟或吞吐量）约 1-3%。这一变更还通过去重冗余元数据，将大多数 Go 程序的内存开销减少约 1%。由于此变更调整了内存分配器的大小类边界，一些对象可能会被提升到更大的大小类，因此某些程序可能会看到较小的改进。

这种更改的结果是，一些对象的地址，以前始终对齐到 16 字节（或更高）边界，现在只会对齐到 8 字节边界。一些使用需要内存地址超过 8 字节对齐的汇编指令的程序，并依赖于内存分配器先前的对齐行为的程序可能会中断，但我们预计这样的程序很少。此类程序可以使用 `GOEXPERIMENT=noallocheaders` 构建，以恢复旧的元数据布局并恢复先前的对齐行为，但包的所有者应更新其汇编代码以避免对齐假设，因为此解决方法将在将来的发布中删除。

在 `windows/amd64 port` 上，使用 `-buildmode=c-archive` 或 `-buildmode=c-shared` 构建的 Go 库的程序现在可以使用 `SetUnhandledExceptionFilter` Win32 函数来捕获 Go 运行时未处理的异常。请注意，这在 `windows/386` 端口上已经支持。

## 编译器

[Profile-guided Optimization (PGO)](/doc/pgo) 构建现在可以比以前更高比例地去虚拟化调用。来自一组代表性 Go 程序的大多数程序现在在运行时启用 PGO 后看到了 2 到 14% 的性能提升。

编译器现在交错进行去虚拟化和内联，因此接口方法调用得到了更好的优化。

Go 1.22 还包括对编译器内联阶段增强实现的预览，该实现使用启发式方法增强了在被视为“重要”的调用站点（例如在循环中）的内联性，并阻止了在被视为“不重要”的调用站点（例如在 panic 路径上）的内联。使用 `GOEXPERIMENT=newinliner` 构建启用新的调用站点启发式方法；详见 [issue #61502](/issue/61502) 获取更多信息并提供反馈。

## 链接器

链接器的 `-s` 和 `-w` 标志现在在所有平台上的行为更加一致。`-w` 标志抑制 DWARF 调试信息的生成。`-s` 标志抑制符号表的生成。`-s` 标志也意味着 `-w` 标志，可以通过 `-w=0` 来否定。也就是说，`-s` `-w=0` 将生成一个同时生成 DWARF 调试信息但不生成符号表的二进制文件。

在 ELF 平台上，`-B` 链接器标志现在接受一种特殊形式：使用 `-B` `gobuildid`，链接器将生成一个从 Go 构建 ID 派生的 GNU 构建 ID（ELF `NT_GNU_BUILD_ID` 注释）。

在 Windows 上，当使用 `-linkmode=internal` 构建时，链接器现在通过将 `.pdata` 和 `.xdata` 部分复制到最终二进制文件中来保留来自 C 对象文件的 SEH 信息。这有助于使用本地工具（如 WinDbg）调试和分析二进制文件。请注意，直到现在，未能遵守 C 函数的 SEH 异常处理程序，因此此更改可能导致某些程序的行为不同。`-linkmode=external` 不受此更改影响，因为外部链接器已经保留 SEH 信息。

## Bootstrap

如 [Go 1.20 发布说明](/doc/go1.20#bootstrap) 中所述，Go 1.22 现在要求使用 Go 1.20 或更高版本的最终点发行版进行引导。我们预计 Go 1.24 将要求使用 Go 1.22 或更高版本的最终点发行版进行引导。

## 核心库

### 新的 math/rand/v2 包

Go 1.22 中包含了标准库中的第一个“v2”包，[`math/rand/v2`](/pkg/math/rand/v2/)。与 [`math/rand`](/pkg/math/rand/) 相比的变化在 [提案 #61716](/issue/61716) 中有详细说明。最重要的变化包括：

+   `math/rand` 中已弃用的 `Read` 方法未被 `math/rand/v2` 继续使用。（在 `math/rand` 中仍可用。）绝大多数对 `Read` 的调用应改为使用 [`crypto/rand` 的 `Read`](/pkg/crypto/rand/#Read)。否则，可以使用 `Uint64` 方法构造自定义的 `Read`。

+   顶级函数访问的全局生成器被无条件随机种子化。因为 API 保证不固定结果序列，所以现在可以进行像每个线程随机生成器状态这样的优化。

+   [`Source`](/pkg/math/rand/v2/#Source) 接口现在只有一个 `Uint64` 方法；不再有 `Source64` 接口。

+   许多方法现在使用更快的算法，这在 `math/rand` 中不可能采用，因为它们改变了输出流。

+   `math/rand` 中的 `Intn`、`Int31`、`Int31n`、`Int63` 和 `Int64n` 顶级函数和方法在 `math/rand/v2` 中以更符合习惯的方式命名：`IntN`、`Int32`、`Int32N`、`Int64` 和 `Int64N`。还新增了顶级函数和方法 `Uint32`、`Uint32N`、`Uint64`、`Uint64N` 和 `UintN`。

+   新的通用函数 [`N`](/pkg/math/rand/v2/#N) 类似于 [`Int64N`](/pkg/math/rand/v2/#Int64N) 或 [`Uint64N`](/pkg/math/rand/v2/#Uint64N)，但适用于任何整数类型。例如，从 0 到 5 分钟的随机持续时间是 `rand.N(5*time.Minute)`。

+   [`math/rand` 的 `Source`](/pkg/math/rand/#Source) 提供的 Mitchell & Reeds LFSR 生成器已被两个更现代的伪随机生成器源替代： [`ChaCha8`](/pkg/math/rand/v2/#ChaCha8) 和 [`PCG`](/pkg/math/rand/v2/#PCG)。ChaCha8 是一个新的、密码学上强大的随机数生成器，在效率上与 PCG 类似。ChaCha8 是 `math/rand/v2` 中顶级函数使用的算法。自 Go 1.22 起，`math/rand` 的顶级函数（未显式种子化时）和 Go 运行时也使用 ChaCha8 进行随机化。

我们计划在未来发布的 Go 1.23 中包含一个 API 迁移工具。

### 新的 go/version 包

新的 [`go/version`](/pkg/go/version/) 包实现了用于验证和比较 Go 版本字符串的功能。

### 改进的路由模式

标准库中的 HTTP 路由现在更加表达丰富。 [`net/http.ServeMux`](/pkg/net/http#ServeMux) 使用的模式已经改进，可以接受方法和通配符。

使用方法注册处理程序，如 `"POST /items/create"`，限制调用处理程序的请求必须使用给定方法。具有方法的模式优先于没有方法但匹配的模式。作为特例，使用 `"GET"` 注册处理程序也同时注册为 `"HEAD"`。

模式中的通配符，如 `/items/{id}`，匹配 URL 路径的段。可以通过调用 [`Request.PathValue`](/pkg/net/http#Request.PathValue) 方法访问实际段值。以“…”结尾的通配符，如 `/files/{path...}`，必须出现在模式的末尾，并匹配所有剩余的段。

以“/”结尾的模式与其作为前缀的所有路径匹配，这是一贯的。要匹配包括尾部斜杠的确切模式，请以 `{$}` 结尾，例如 `/exact/match/{$}`。

如果两个模式在它们匹配的请求中重叠，则更具体的模式优先。如果两者都不更具体，则模式冲突。此规则推广了原始的优先规则，并保持了模式注册顺序不重要的特性。

此更改以小幅度方式破坏了向后兼容性，一些明显的方式是——带有“{”和“}”的模式行为不同——还有一些不太明显的方式是——已改进了转义路径的处理。这一更改由名为 `httpmuxgo121` 的 [`GODEBUG`](/doc/godebug) 字段控制。设置 `httpmuxgo121=1` 可恢复旧行为。

### 库中的轻微更改

一如既往，库中还有各种轻微更改和更新，这些更改是考虑到了 Go 1 [兼容性承诺](/doc/go1compat)。此外，还有各种未在此处列举的性能改进。

[archive/tar](/pkg/archive/tar/)

新方法 [`Writer.AddFS`](/pkg/archive/tar#Writer.AddFS) 将 [`fs.FS`](/pkg/io/fs#FS) 中的所有文件添加到存档中。

[archive/zip](/pkg/archive/zip/)

新方法 [`Writer.AddFS`](/pkg/archive/zip#Writer.AddFS) 将 [`fs.FS`](/pkg/io/fs#FS) 中的所有文件添加到存档中。

[bufio](/pkg/bufio/)

当 [`SplitFunc`](/pkg/bufio#SplitFunc) 返回带有 `nil` 令牌的 [`ErrFinalToken`](/pkg/bufio#ErrFinalToken) 时，[`Scanner`](/pkg/bufio#Scanner) 现在会立即停止。以前，它会在停止前报告一个最终的空令牌，这通常是不需要的。希望报告最终空令牌的调用者可以通过返回 `[]byte{}` 而不是 `nil` 来实现。

[cmp](/pkg/cmp/)

新函数 `Or` 返回不是零值的值序列中的第一个。

[crypto/tls](/pkg/crypto/tls/)

[`ConnectionState.ExportKeyingMaterial`](/pkg/crypto/tls#ConnectionState.ExportKeyingMaterial) 现在只有在使用 TLS 1.3 或服务器和客户端均支持 `extended_master_secret` 扩展时才返回错误。自 Go 1.20 版起，`crypto/tls` 就支持此扩展。可以通过设置 `tlsunsafeekm=1` 的 `GODEBUG` 设置来禁用此功能。

默认情况下，如果未通过[`config.MinimumVersion`](/pkg/crypto/tls#Config.MinimumVersion)指定，`crypto/tls`服务器现在提供的最低版本是TLS 1.2，与`crypto/tls`客户端的行为一致。可以通过设置`GODEBUG`中的`tls10server=1`来恢复此行为。

在预TLS 1.3握手期间，客户端和服务器默认不再提供不支持ECDHE的密码套件。可以通过设置`GODEBUG`中的`tlsrsakex=1`来恢复此行为。

[crypto/x509](/pkg/crypto/x509/)

新的[`CertPool.AddCertWithConstraint`](/pkg/crypto/x509#CertPool.AddCertWithConstraint)方法可以用于向根证书添加自定义约束，在链路建立期间应用这些约束。

在Android上，根证书现在将从`/data/misc/keychain/certs-added`以及`/system/etc/security/cacerts`加载。

新类型[`OID`](/pkg/crypto/x509#OID)支持具有大于31位的单独组件的ASN.1对象标识符。`Certificate`结构中新增了使用此类型的字段[`Policies`](/pkg/crypto/x509#Certificate.Policies)，在解析过程中现在会填充该字段。任何无法用[`asn1.ObjectIdentifier`](/pkg/encoding/asn1#ObjectIdentifier)表示的OID都将出现在`Policies`中，而不会出现在旧的`PolicyIdentifiers`字段中。调用[`CreateCertificate`](/pkg/crypto/x509#CreateCertificate)时，将忽略`Policies`字段，并从`PolicyIdentifiers`字段中获取策略。通过设置`GODEBUG`中的`x509usepolicies=1`可以反转此行为，从`Policies`字段中填充证书策略，并忽略`PolicyIdentifiers`字段。我们可能会在Go 1.23中更改`x509usepolicies`的默认值，使`Policies`成为默认的编组字段。

[database/sql](/pkg/database/sql/)

新的[`Null[T]`](/pkg/database/sql/#Null)类型提供了一种方式来扫描可空列的任何列类型。

[debug/elf](/pkg/debug/elf/)

为MIPS64系统定义了常量`R_MIPS_PC32`。

针对龙芯系统，额外定义了`R_LARCH_*`常量。

[encoding](/pkg/encoding/)

在[`encoding/base32`](/pkg/encoding/base32)、[`encoding/base64`](/pkg/encoding/base64)和[`encoding/hex`](/pkg/encoding/hex)包中，新增了`AppendEncode`和`AppendDecode`方法，简化了对字节切片的编码和解码，同时处理了字节切片缓冲区管理。

现在，[`base32.Encoding.WithPadding`](/pkg/encoding/base32#Encoding.WithPadding)和[`base64.Encoding.WithPadding`](/pkg/encoding/base64#Encoding.WithPadding)方法如果`padding`参数为负值且不是`NoPadding`，将会抛出panic异常。

[encoding/json](/pkg/encoding/json/)

现在，编组和编码功能将`'\b'`和`'\f'`字符转义为`\b`和`\f`，而不再转义为`\u0008`和`\u000c`。

[go/ast](/pkg/go/ast/)

与 [语法标识符解析](https://pkg.go.dev/go/ast#Object) 相关的以下声明现已被 [弃用](/issue/52463)：`Ident.Obj`、`Object`、`Scope`、`File.Scope`、`File.Unresolved`、`Importer`、`Package`、`NewPackage`。一般情况下，没有类型信息的情况下无法准确解析标识符。例如，在 `T{K: ""}` 中的标识符 `K` 可能是地图类型 T 的局部变量名称，也可能是结构类型 T 的字段名称。新程序应使用 [go/types](/pkg/go/types) 包来解析标识符；详见 [`Object`](https://pkg.go.dev/go/types#Object)、[`Info.Uses`](https://pkg.go.dev/go/types#Info.Uses) 和 [`Info.Defs`](https://pkg.go.dev/go/types#Info.Defs)。

新的 [`ast.Unparen`](https://pkg.go.dev/go/ast#Unparen) 函数从表达式中移除任何外围的 [括号](https://pkg.go.dev/go/ast#ParenExpr)。

[go/types](/pkg/go/types/)

新的 [`Alias`](/pkg/go/types#Alias) 类型表示类型别名。以前，类型别名没有明确表示，因此对类型别名的引用相当于拼写出别名的类型，并且别名的名称丢失。新的表示保留了中间的 `Alias`。这样做可以改善错误报告（可以报告类型别名的名称），并且允许更好地处理涉及类型别名的循环类型声明。在将来的发布版本中，`Alias` 类型还将携带 [类型参数信息](/issue/46477)。新的函数 [`Unalias`](/pkg/go/types#Unalias) 返回由 `Alias` 类型（或任何其他 [`Type`](/pkg/go/types#Type)）表示的实际类型。

由于 `Alias` 类型可能会破坏不知道如何检查它们的现有类型开关，此功能由名为 `GODEBUG` 的字段控制，名为 `gotypesalias`。当 `gotypesalias=0` 时，所有行为与以前相同，并且永远不会创建 `Alias` 类型。当 `gotypesalias=1` 时，将创建 `Alias` 类型，并且客户端必须预期它们。默认值为 `gotypesalias=0`。在将来的发布版本中，默认值将更改为 `gotypesalias=1`。*[`go/types`](/pkg/go/types) 的客户端被敦促尽快调整他们的代码以适应 `gotypesalias=1`，以尽早消除问题。*

现在，[`Info`](/pkg/go/types#Info) 结构导出了 [`FileVersions`](/pkg/go/types#Info.FileVersions) 映射，提供每个文件的 Go 版本信息。

新的辅助方法 [`PkgNameOf`](/pkg/go/types#Info.PkgNameOf) 返回给定导入声明的本地包名称。

[`SizesFor`](/pkg/go/types#SizesFor) 的实现已经调整为在 `SizesFor` 的编译器参数为 `"gc"` 时计算与编译器相同的类型大小。类型检查器使用的默认 [`Sizes`](/pkg/go/types#Sizes) 实现现在是 `types.SizesFor("gc", "amd64")`。

表示函数体的词法环境块（[`Scope`](/pkg/go/types#Scope)）的起始位置（[`Pos`](/pkg/go/types#Scope.Pos)）已更改：它以前从函数体的左大括号开始，现在从函数的`func`标记开始。

[html/template](/pkg/html/template/)

JavaScript模板文字现在可以包含Go模板动作，并且解析包含此类动作的模板将不再返回`ErrJSTemplate`。同样，GODEBUG设置`jstmpllitinterp`不再起作用。

[io](/pkg/io/)

新的[`SectionReader.Outer`](/pkg/io#SectionReader.Outer)方法返回[`NewSectionReader`](/pkg/io#NewSectionReader)传递的[`ReaderAt`](/pkg/io#ReaderAt)、偏移量和大小。

[log/slog](/pkg/log/slog/)

新的[`SetLogLoggerLevel`](/pkg/log/slog#SetLogLoggerLevel)函数控制`slog`和`log`包之间的桥接级别。它设置对顶级`slog`日志函数的调用的最小级别，并设置通过`slog`传递到`log.Logger`的调用的级别。

[math/big](/pkg/math/big/)

新方法[`Rat.FloatPrec`](/pkg/math/big#Rat.FloatPrec)计算将有理数准确表示为浮点数所需的小数位数，以及首先是否可能进行准确的十进制表示。

[net](/pkg/net/)

当[`io.Copy`](/pkg/io#Copy)从`TCPConn`复制到`UnixConn`时，如果可能，它现在将使用Linux的`splice(2)`系统调用，使用新方法[`TCPConn.WriteTo`](/pkg/net#TCPConn.WriteTo)。

在使用“-tags=netgo”编译时，Go DNS解析器现在会在执行DNS查询之前，在Windows主机文件中搜索匹配的名称，该文件位于`%SystemRoot%\System32\drivers\etc\hosts`。

[net/http](/pkg/net/http/)

新函数[`ServeFileFS`](/pkg/net/http#ServeFileFS)、[`FileServerFS`](/pkg/net/http#FileServerFS)和[`NewFileTransportFS`](/pkg/net/http#NewFileTransportFS)是现有`ServeFile`、`FileServer`和`NewFileTransport`的版本，它们在`fs.FS`上操作。

HTTP服务器和客户端现在拒绝包含无效空`Content-Length`头的请求和响应。可以通过设置[`GODEBUG`](/doc/godebug)字段`httplaxcontentlength=1`来恢复先前的行为。

新方法[`Request.PathValue`](/pkg/net/http#Request.PathValue)从请求中返回路径通配符值，新方法[`Request.SetPathValue`](/pkg/net/http#Request.SetPathValue)在请求上设置路径通配符值。

[net/http/cgi](/pkg/net/http/cgi/)

在执行CGI进程时，`PATH_INFO`变量现在始终设置为空字符串或以`/`字符开头的值，如RFC 3875所需。以前，一些[`Handler.Root`](/pkg/net/http/cgi#Handler.Root)和请求URL的组合可能会违反此要求。

[net/netip](/pkg/net/netip/)

新的[`AddrPort.Compare`](/pkg/net/netip#AddrPort.Compare)方法比较两个`AddrPort`。

[os](/pkg/os/)

在 Windows 上，[`Stat`](/pkg/os#Stat) 函数现在遵循系统中连接到另一个命名实体的所有重解析点。以前只会跟随 `IO_REPARSE_TAG_SYMLINK` 和 `IO_REPARSE_TAG_MOUNT_POINT` 重解析点。

在 Windows 上，将 [`O_SYNC`](/pkg/os#O_SYNC) 传递给 [`OpenFile`](/pkg/os#OpenFile) 现在会导致写操作直接写入磁盘，相当于在 Unix 平台上的 `O_SYNC`。

在 Windows 上，[`ReadDir`](/pkg/os#ReadDir)，[`File.ReadDir`](/pkg/os#File.ReadDir)，[`File.Readdir`](/pkg/os#File.Readdir) 和 [`File.Readdirnames`](/pkg/os#File.Readdirnames) 函数现在批量读取目录条目以减少系统调用次数，提高性能达到 30%。

当 [`io.Copy`](/pkg/io#Copy) 从 `File` 复制到 `net.UnixConn` 时，如果可能，现在将使用 Linux 的 `sendfile(2)` 系统调用，使用新的方法 [`File.WriteTo`](/pkg/os#File.WriteTo)。

[os/exec](/pkg/os/exec/)

在 Windows 上，[`LookPath`](/pkg/os/exec#LookPath) 现在忽略 `%PATH%` 中的空条目，并且如果找不到可执行文件扩展名以解析一个非常明确的名称，则返回 `ErrNotFound`（而不是 `ErrNotExist`）。

在 Windows 上，如果可执行文件的路径已经是绝对路径并且具有可执行文件扩展名，则 [`Command`](/pkg/os/exec#Command) 和 [`Cmd.Start`](/pkg/os/exec#Cmd.Start) 不再调用 `LookPath`。此外，`Cmd.Start` 不再将解析后的扩展名写回 [`Path`](/pkg/os/exec#Cmd.Path) 字段，因此可以安全地同时调用 `Start` 方法和 `String` 方法。

[reflect](/pkg/reflect/)

[`Value.IsZero`](/pkg/reflect/#Value.IsZero) 方法现在对于浮点数或复数的负零会返回 true，如果结构体的一个空白字段（即命名为 `_` 的字段）某种方式具有非零值，也会返回 true。这些变更使得 `IsZero` 方法与使用语言 `==` 运算符比较值是否为零的行为保持一致。

[`PtrTo`](/pkg/reflect/#PtrTo) 函数已废弃，推荐使用 [`PointerTo`](/pkg/reflect/#PointerTo)。

新的函数 [`TypeFor`](/pkg/reflect/#TypeFor) 返回表示类型参数 T 的 [`Type`](/pkg/reflect/#Type)。以前，要获取类型的 `reflect.Type` 值，必须使用 `reflect.TypeOf((*T)(nil)).Elem()`。现在可以写成 `reflect.TypeFor[T]()`。

[runtime/metrics](/pkg/runtime/metrics/)

四个新的直方图度量 `/sched/pauses/stopping/gc:seconds`，`/sched/pauses/stopping/other:seconds`，`/sched/pauses/total/gc:seconds` 和 `/sched/pauses/total/other:seconds` 提供关于停止世界暂停的额外详细信息。"stopping" 度量报告从决定停止世界到所有 goroutine 停止的时间。"total" 度量报告从决定停止世界到再次启动的时间。

度量 `/gc/pauses:seconds` 已废弃，因为它等效于新的 `/sched/pauses/total/gc:seconds` 度量。

`/sync/mutex/wait/total:seconds` 现在包括对运行时内部锁的争用，除了 [`sync.Mutex`](/pkg/sync#Mutex) 和 [`sync.RWMutex`](/pkg/sync#RWMutex)。

[runtime/pprof](/pkg/runtime/pprof/)

互斥锁性能分析现在根据在互斥锁上阻塞的 goroutine 数量来衡量争用程度。这提供了一个更准确的表示，在 Go 程序中互斥锁作为瓶颈的程度。例如，如果有 100 个 goroutine 在互斥锁上阻塞了 10 毫秒，互斥锁性能分析现在将记录 1 秒的延迟，而不是 10 毫秒的延迟。

互斥锁性能分析现在还包括对运行时内部锁的争用，除了 [`sync.Mutex`](/pkg/sync#Mutex) 和 [`sync.RWMutex`](/pkg/sync#RWMutex)。运行时内部锁的争用始终报告为 `runtime._LostContendedRuntimeLock`。将来的版本将在这些情况下添加完整的堆栈跟踪。

在 Darwin 平台上的 CPU 性能分析现在包含进程的内存映射，这使得 pprof 工具中的反汇编视图得以启用。

[runtime/trace](/pkg/runtime/trace/)

此版本中完全改写了执行追踪器，解决了几个长期存在的问题，并为执行追踪的新用例铺平了道路。

执行追踪现在在大多数平台上使用操作系统的时钟（Windows 除外），因此可以将其与低级组件生成的追踪进行关联。执行追踪不再依赖平台时钟的可靠性来生成正确的追踪。执行追踪现在会动态地进行分区，并因此可以以流的方式处理。执行追踪现在包含所有系统调用的完整持续时间。执行追踪现在包含有关 goroutine 执行的操作系统线程信息。启动和停止执行追踪的延迟影响已经显著减少。执行追踪现在可以在垃圾收集标记阶段期间开始或结束。

为了让 Go 开发者能够利用这些改进，一个实验性的追踪读取包已经在 [golang.org/x/exp/trace](/pkg/golang.org/x/exp/trace) 上提供。请注意，目前这个包只能处理使用 Go 1.22 构建的程序生成的追踪数据。请尝试使用该包，并在 [对应的提议问题](/issue/62627) 上提供反馈。

如果您在新的执行追踪器实现中遇到任何问题，您可以通过使用 `GOEXPERIMENT=noexectracer2` 编译您的 Go 程序来切换回旧的实现。如果您这样做，请提交一个问题报告，否则这个选项将在将来的版本中被移除。

[切片](/pkg/slices/)

新函数 `Concat` 用于连接多个切片。

缩小切片大小的函数（`Delete`、`DeleteFunc`、`Compact`、`CompactFunc` 和 `Replace`）现在会将新长度和旧长度之间的元素置零。

如果 `Insert` 的参数 `i` 超出范围，现在会始终引发 panic。之前在这种情况下，如果没有要插入的元素，它不会引发 panic。

[syscall](/pkg/syscall/)

`syscall` 包自 Go 1.4 起已被冻结，并在 Go 1.11 中标记为废弃，导致许多编辑器对该包的任何使用发出警告。然而，一些非废弃功能需要使用 `syscall` 包，例如[`os/exec.Cmd.SysProcAttr`](/pkg/os/exec#Cmd)字段。为了避免对这类代码的不必要投诉，`syscall` 包不再标记为废弃。该包保持冻结状态，大部分新功能不再添加，新代码鼓励使用[`golang.org/x/sys/unix`](/pkg/golang.org/x/sys/unix)或[`golang.org/x/sys/windows`](/pkg/golang.org/x/sys/windows)。

在 Linux 上，新的[`SysProcAttr.PidFD`](/pkg/syscall#SysProcAttr)字段允许在通过[`StartProcess`](/pkg/syscall#StartProcess)或[`os/exec`](/pkg/os/exec)启动子进程时获取 PID FD。

在 Windows 上，将[`O_SYNC`](/pkg/syscall#O_SYNC)传递给[`Open`](/pkg/syscall#Open)现在会导致写操作直接写入磁盘，相当于 Unix 平台上的 `O_SYNC`。

[testing/slogtest](/pkg/testing/slogtest/)

新的[`Run`](/pkg/testing/slogtest#Run)函数使用子测试运行测试用例，提供更精细的控制。

## 端口

### Darwin

在 macOS 的 64 位 x86 架构上（`darwin/amd64` 端口），Go 工具链现在默认生成位置无关可执行文件（PIE）。可以通过指定 `-buildmode=exe` 构建标志生成非 PIE 二进制文件。在基于 64 位 ARM 的 macOS（`darwin/arm64` 端口），Go 工具链已经默认生成 PIE。

Go 1.22 是最后一个支持运行在 macOS 10.15 Catalina 上的版本。Go 1.23 将要求 macOS 11 Big Sur 或更高版本。

### Arm

`GOARM` 环境变量现在允许选择使用软件或硬件浮点。以前，`GOARM` 的有效值为 `5`、`6` 或 `7`。现在可以在这些值后面选择 `,softfloat` 或 `,hardfloat` 来选择浮点实现。

这个新选项在版本 `5` 默认为 `softfloat`，版本 `6` 和 `7` 默认为 `hardfloat`。

### Loong64

`loong64` 端口现在支持使用寄存器传递函数参数和结果。

`linux/loong64` 端口现在支持地址检测器、内存检测器、新式链接器重定位和 `plugin` 构建模式。

### OpenBSD

Go 1.22 在 OpenBSD 上添加了一个实验性端口，支持大端 64 位 PowerPC (`openbsd/ppc64`)。
