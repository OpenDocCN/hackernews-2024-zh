<!--yml

分类：未分类

日期：2024-05-27 14:31:24

-->

# 推荐使用表驱动测试 | Dave Cheney

> 来源：[https://dave.cheney.net/2019/05/07/prefer-table-driven-tests](https://dave.cheney.net/2019/05/07/prefer-table-driven-tests)

我非常喜欢测试，特别是[unit testing](https://dave.cheney.net/2019/04/03/absolute-unit-test)和正确进行的TDD（[当然](https://www.youtube.com/watch?v=EZ05e7EMOLM)）。围绕 Go 项目发展起来的一种实践是表驱动测试的概念。本文探讨了编写表驱动测试的如何和为什么。

假设我们有一个拆分字符串的函数：

```
// Split slices s into all substrings separated by sep and
// returns a slice of the substrings between those separators.
func Split(s, sep string) []string {
    var result []string
    i := strings.Index(s, sep)
    for i > -1 {
        result = append(result, s[:i])
        s = s[i+len(sep):]
        i = strings.Index(s, sep)
    }
    return append(result, s)
}
```

在 Go 中，单元测试只是常规的 Go 函数（带有一些规则），因此我们为这个函数编写一个单元测试，从同一目录中的一个文件开始，具有相同的包名称，`strings`。

```
package split

import (
    "reflect"
    "testing"
)

func TestSplit(t *testing.T) {
    got := Split("a/b/c", "/")
    want := []string{"a", "b", "c"}
    if !reflect.DeepEqual(want, got) {
         t.Fatalf("expected: %v, got: %v", want, got)
    }
}
```

测试只是带有几个规则的常规 Go 函数：

1.  测试函数的名称必须以`Test`开头。

1.  测试函数必须接受一个类型为`*testing.T`的参数。`*testing.T`是由测试包本身注入的类型，用于提供打印、跳过和失败测试的方法。

在我们的测试中，我们使用一些输入调用`Split`，然后将其与我们预期的结果进行比较。

## 代码覆盖率

下一个问题是，这个包的覆盖率是多少？幸运的是，Go 工具有一个内置的分支覆盖率。我们可以像这样调用它：

```
% go test -coverprofile=c.out
PASS
coverage: 100.0% of statements
ok      split   0.010s
```

这告诉我们，我们有100%的分支覆盖率，这并不奇怪，因为这段代码只有一个分支。

如果我们想深入了解覆盖率报告，go 工具有几个选项可以打印覆盖率报告。我们可以使用`go tool cover -func`来按函数分解覆盖率：

```
% go tool cover -func=c.out
split/split.go:8:       Split          100.0%
total:                  (statements)   100.0%
```

这个包里只有一个函数，所以可能不那么令人兴奋，但我确信你会找到更刺激的包进行测试。

### 在那上面喷一些 .bashrc

对我来说，这对命令非常有用，我有一个shell别名，可以在一个命令中运行测试覆盖率和报告：

```
cover () {
    local t=$(mktemp -t cover)
    go test $COVERFLAGS -coverprofile=$t $@ \
        && go tool cover -func=$t \
        && unlink $t
}
```

## 超过100%的覆盖率

所以，我们编写了一个测试案例，获得了100%的覆盖率，但这并不是故事的终结。我们有了良好的分支覆盖率，但可能需要测试一些边界条件。例如，如果我们尝试在逗号上进行拆分会发生什么？

```
func TestSplitWrongSep(t *testing.T) {
    got := Split("a/b/c", ",")
    want := []string{"a/b/c"}
    if !reflect.DeepEqual(want, got) {
        t.Fatalf("expected: %v, got: %v", want, got)
    }
}
```

或者，如果源字符串中没有分隔符会发生什么？

```
func TestSplitNoSep(t *testing.T) {
    got := Split("abc", "/")
    want := []string{"abc"}
    if !reflect.DeepEqual(want, got) {
        t.Fatalf("expected: %v, got: %v", want, got)
    }
}
```

我们正在构建一组测试案例，以测试边界条件。这是很好的。

## 引入基于表驱动的测试

然而，我们的测试中存在大量重复。对于每个测试用例，只有输入、预期输出和测试用例名称会发生变化。其他所有内容都是样板文件。我们希望设置所有输入和预期输出并将它们填充到单个测试框架中。这是引入表驱动测试的绝佳时机。

```
func TestSplit(t *testing.T) {
    type test struct {
        input string
        sep   string
        want  []string
    }

    tests := []test{
        {input: "a/b/c", sep: "/", want: []string{"a", "b", "c"}},
        {input: "a/b/c", sep: ",", want: []string{"a/b/c"}},
        {input: "abc", sep: "/", want: []string{"abc"}},
    }

    for _, tc := range tests {
        got := Split(tc.input, tc.sep)
        if !reflect.DeepEqual(tc.want, got) {
            t.Fatalf("expected: %v, got: %v", tc.want, got)
        }
    }
}
```

我们声明了一个结构来保存我们的测试输入和预期输出。这就是我们的表。`tests`结构通常是本地声明，因为我们希望在此包中的其他测试中重用这个名称。

实际上，我们甚至不需要给类型命名，我们可以使用匿名结构文字来减少样板代码，就像这样：

```
func TestSplit(t *testing.T) {
    tests := []struct {
        input string
        sep   string
        want  []string
    }{
        {input: "a/b/c", sep: "/", want: []string{"a", "b", "c"}},
        {input: "a/b/c", sep: ",", want: []string{"a/b/c"}},
        {input: "abc", sep: "/", want: []string{"abc"}},
    } 

    for _, tc := range tests {
        got := Split(tc.input, tc.sep)
        if !reflect.DeepEqual(tc.want, got) {
            t.Fatalf("expected: %v, got: %v", tc.want, got)
        }
    }
}
```

现在，添加一个新的测试就是一个直接的事情；只需在`tests`结构中添加另一行。例如，如果我们的输入字符串有一个尾部分隔符会发生什么？

```
{input: "a/b/c", sep: "/", want: []string{"a", "b", "c"}},
{input: "a/b/c", sep: ",", want: []string{"a/b/c"}},
{input: "abc", sep: "/", want: []string{"abc"}},
{input: "a/b/c/", sep: "/", want: []string{"a", "b", "c"}}, // trailing sep
```

但是，当我们运行`go test`时，我们得到了这个结果

```
% go test
--- FAIL: TestSplit (0.00s)
    split_test.go:24: expected: [a b c], got: [a b c ]

```

除了测试失败之外，还有几个问题需要讨论。

第一种方法是将每个测试从函数重写为表中的一行，我们失去了失败测试的名称。我们在测试文件中添加了一条注释来说明这种情况，但我们无法在`go test`输出中访问该注释。

有几种方法可以解决这个问题。在Go代码库中，您会看到各种风格的混合使用，因为人们继续尝试这种形式。

## 枚举测试案例

由于测试案例存储在一个切片中，我们可以在失败消息中打印出测试案例的索引：

```
func TestSplit(t *testing.T) {
    tests := []struct {
        input string
        sep   string
        want  []string
    }{
        {input: "a/b/c", sep: "/", want: []string{"a", "b", "c"}},
        {input: "a/b/c", sep: ",", want: []string{"a/b/c"}},
        {input: "abc", sep: "/", want: []string{"abc"}},
        {input: "a/b/c/", sep: "/", want: []string{"a", "b", "c"}},
    }

    for i, tc := range tests {
        got := Split(tc.input, tc.sep)
        if !reflect.DeepEqual(tc.want, got) {
            t.Fatalf("test %d: expected: %v, got: %v", i+1, tc.want, got)
        }
    }
}
```

现在当我们运行`go test`时，我们得到了这个结果

```
% go test
--- FAIL: TestSplit (0.00s)
    split_test.go:24: test 4: expected: [a b c], got: [a b c ]

```

这样稍微好一点了。现在我们知道第四个测试失败了，尽管我们必须进行一些调整，因为切片索引和范围迭代是从零开始的。这要求测试案例之间保持一致性；如果有些使用零基础报告，而其他人使用一基础报告，这会令人困惑。而且，如果测试案例列表很长，可能会难以计算括号以确切找出哪个装置构成第四个测试案例。

## 给您的测试案例命名

另一种常见的模式是在测试装置中包含一个名称字段。

```
func TestSplit(t *testing.T) {
    tests := []struct {
 name  string
        input string
        sep   string
        want  []string
    }{
        {name: "simple", input: "a/b/c", sep: "/", want: []string{"a", "b", "c"}},
        {name: "wrong sep", input: "a/b/c", sep: ",", want: []string{"a/b/c"}},
        {name: "no sep", input: "abc", sep: "/", want: []string{"abc"}},
        {name: "trailing sep", input: "a/b/c/", sep: "/", want: []string{"a", "b", "c"}},
    }

    for _, tc := range tests {
        got := Split(tc.input, tc.sep)
        if !reflect.DeepEqual(tc.want, got) {
            t.Fatalf("%s: expected: %v, got: %v", tc.name, tc.want, got)
        }
    }
}
```

现在当测试失败时，我们有一个描述性名称来说明测试在做什么。我们不再需要尝试从输出中找出这些信息，同时现在我们有一个可以搜索的字符串。

```
% go test
--- FAIL: TestSplit (0.00s)
    split_test.go:25: trailing sep: expected: [a b c], got: [a b c ]

```

我们可以使用地图文字语法进一步简化这个过程：

```
func TestSplit(t *testing.T) {
    tests := map[string]struct {
        input string
        sep   string
        want  []string
    }{ 
        "simple":       {input: "a/b/c", sep: "/", want: []string{"a", "b", "c"}}, 
        "wrong sep":    {input: "a/b/c", sep: ",", want: []string{"a/b/c"}},
        "no sep":       {input: "abc", sep: "/", want: []string{"abc"}},
        "trailing sep": {input: "a/b/c/", sep: "/", want: []string{"a", "b", "c"}},
    }

    for name, tc := range tests {
        got := Split(tc.input, tc.sep)
        if !reflect.DeepEqual(tc.want, got) {
            t.Fatalf("%s: expected: %v, got: %v", name, tc.want, got)
        }
    }
}
```

使用地图文字语法定义我们的测试案例，不是作为结构体切片，而是作为测试名称到测试装置的地图。使用地图的一个附加好处是可能提高我们测试的实用性。

地图迭代顺序是*未定义*的。这意味着每次运行`go test`时，我们的测试可能会按不同的顺序运行。

这对于发现在语句顺序运行时测试通过，但在其他情况下不通过的情况非常有用。如果发现这种情况发生，您可能有一些全局状态被一个测试改变，而随后的测试依赖于该修改。

## 引入子测试

在我们修复失败的测试之前，还有几个其他问题需要在我们的表驱动测试框架中解决。

第一个问题是当一个测试案例失败时，我们调用了`t.Fatalf`。这意味着在第一个失败的测试案例后，我们停止了其他测试案例的测试。由于测试案例按未定义的顺序运行，如果有测试失败，知道是唯一的失败还是第一个失败将是很好的。

如果我们费力地编写每个测试用例作为其自己的函数，测试包会为我们完成这项工作，但这样做相当冗长。好消息是，自从Go 1.7以来，已经添加了一个新功能，使我们可以轻松地为表驱动测试做到这一点。它们被称为[sub tests](https://blog.golang.org/subtests)。

```
func TestSplit(t *testing.T) {
    tests := map[string]struct {
        input string
        sep   string
        want  []string
    }{
        "simple":       {input: "a/b/c", sep: "/", want: []string{"a", "b", "c"}},
        "wrong sep":    {input: "a/b/c", sep: ",", want: []string{"a/b/c"}},
        "no sep":       {input: "abc", sep: "/", want: []string{"abc"}},
        "trailing sep": {input: "a/b/c/", sep: "/", want: []string{"a", "b", "c"}},
    }

    for name, tc := range tests {
        t.Run(name, func(t *testing.T) {
            got := Split(tc.input, tc.sep)
            if !reflect.DeepEqual(tc.want, got) {
                t.Fatalf("expected: %v, got: %v", tc.want, got)
            }
        })
    }
}
```

现在，每个子测试都有一个名称，所以在任何测试运行中，该名称都会自动打印出来。

```
% go test
--- FAIL: TestSplit (0.00s)
    --- FAIL: TestSplit/trailing_sep (0.00s)
        split_test.go:25: expected: [a b c], got: [a b c ]

```

每个子测试都是其自己的匿名函数，因此我们可以使用`t.Fatalf`、`t.Skipf`以及所有其他`testing.T`助手，同时保持表驱动测试的紧凑性。

### 可以直接执行各个单独的子测试用例。

因为子测试有一个名称，所以可以使用`go test -run`标志按名称运行选择的子测试。

```
% go test -run=.*/trailing -v
=== RUN   TestSplit
=== RUN   TestSplit/trailing_sep
--- FAIL: TestSplit (0.00s)
    --- FAIL: TestSplit/trailing_sep (0.00s)
        split_test.go:25: expected: [a b c], got: [a b c ]

```

## 比较我们得到的与我们想要的结果：

现在我们准备修复测试用例。让我们看看错误信息。

```
--- FAIL: TestSplit (0.00s)
    --- FAIL: TestSplit/trailing_sep (0.00s)
        split_test.go:25: expected: [a b c], got: [a b c ]
```

你能发现问题吗？显然切片是不同的，这是`reflect.DeepEqual`感到困惑的原因。但是找到实际的差异并不容易，你必须注意到在`c`后面有额外的空格。在这个简单的例子中，这看起来很简单，但当你比较两个复杂的深度嵌套的gRPC结构时，情况却并非如此。

如果我们转而使用`%#v`语法查看该值作为Go（ish）声明，我们可以改善输出：

```
got := Split(tc.input, tc.sep)
if !reflect.DeepEqual(tc.want, got) {
    t.Fatalf("expected: %#v, got: %#v", tc.want, got)
}
```

现在当我们运行我们的测试时，很明显问题在于切片中多了一个空白元素。

```
% go test
--- FAIL: TestSplit (0.00s)
    --- FAIL: TestSplit/trailing_sep (0.00s)
        split_test.go:25: expected: []string{"a", "b", "c"}, got: []string{"a", "b", "c", ""}

```

但在我们修复测试失败之前，我想更详细地讨论选择正确的方式来呈现测试失败。我们的`Split`函数很简单，它接受一个基本字符串并返回一个字符串切片，但如果它与结构体一起工作，或者更糟，与结构体的指针一起工作会怎么样？

这里有一个`%#v`效果不佳的例子：

```
func main() {
    type T struct {
        I int
    }
    x := []*T{{1}, {2}, {3}}
    y := []*T{{1}, {2}, {4}}
    fmt.Printf("%v %v\n", x, y)
    fmt.Printf("%#v %#v\n", x, y)
}
```

第一个`fmt.Printf`打印了令人无助但是预期的地址切片; `[0xc000096000 0xc000096008 0xc000096010] [0xc000096018 0xc000096020 0xc000096028]`。然而我们的`%#v`版本并没有更好，打印了一个将地址切片转换为`*main.T`的切片; `[]*main.T{(*main.T)(0xc000096000), (*main.T)(0xc000096008), (*main.T)(0xc000096010)} []*main.T{(*main.T)(0xc000096018), (*main.T)(0xc000096020), (*main.T)(0xc000096028)}`

由于使用任何`fmt.Printf`动词的限制，我想介绍来自Google的[go-cmp](https://github.com/google/go-cmp)库。

cmp库的目标是专门用于比较两个值。这类似于`reflect.DeepEqual`，但它具有更多的功能。当然，使用cmp包，你可以编写：

```
func main() {
    type T struct {
        I int
    }
    x := []*T{{1}, {2}, {3}}
    y := []*T{{1}, {2}, {4}}
    fmt.Println(cmp.Equal(x, y)) // false
}
```

但是对于我们的测试函数来说，更有用的是`cmp.Diff`函数，它将递归地生成描述两个值之间差异的文本描述。

```
func main() {
    type T struct {
        I int
    }
    x := []*T{{1}, {2}, {3}}
    y := []*T{{1}, {2}, {4}}
    diff := cmp.Diff(x, y)
    fmt.Printf(diff)
}
```

而实际上产生了：

```
% go run
{[]*main.T}[2].I:
         -: 3
         +: 4
```

告诉我们在`T`结构体切片的第2个元素处，`I`字段预期值为3，但实际上是4。

把所有这些都放在一起，我们有了我们的表驱动的go-cmp测试。

```
func TestSplit(t *testing.T) {
    tests := map[string]struct {
        input string
        sep   string
        want  []string
    }{
        "simple":       {input: "a/b/c", sep: "/", want: []string{"a", "b", "c"}},
        "wrong sep":    {input: "a/b/c", sep: ",", want: []string{"a/b/c"}},
        "no sep":       {input: "abc", sep: "/", want: []string{"abc"}},
        "trailing sep": {input: "a/b/c/", sep: "/", want: []string{"a", "b", "c"}},
    }

    for name, tc := range tests {
        t.Run(name, func(t *testing.T) {
            got := Split(tc.input, tc.sep)
            diff := cmp.Diff(tc.want, got)
            if diff != "" {
                t.Fatalf(diff)
            }
        })
    }
}
```

运行后我们得到：

```
% go test
--- FAIL: TestSplit (0.00s)
    --- FAIL: TestSplit/trailing_sep (0.00s)
        split_test.go:27: {[]string}[?->3]:
                -: <non-existent>
                +: ""
FAIL
exit status 1
FAIL    split   0.006s
```

使用 `cmp.Diff` 我们的测试工具不仅仅告诉我们得到的和我们想要的不同。我们的测试表明字符串的长度不同，夹具中的第三个索引不应存在，但实际输出为空字符串，即“”。从这里开始修复测试失败是直截了当的。
