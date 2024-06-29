<!--yml

分类：未分类

日期：2024-05-27 13:04:55

-->

# 如何在Go中编写测试 :: 非常好的软件，不是病毒

> 来源：[https://blog.verygoodsoftwarenotvirus.ru/posts/testing-in-go/](https://blog.verygoodsoftwarenotvirus.ru/posts/testing-in-go/)

# 如何在Go中编写单元测试[⌗](#how-i-write-unit-tests-in-go)

在Go中，我最喜欢的功能之一是，与许多流行语言不同，它自带了自己的测试框架，即[testing](https://pkg.go.dev/testing)包。假设我们有一个在名为`numbers.go`的文件中的微不足道的函数：

```
package numbers

func addNumbers(numbers ...int) int {
	sum := 0

	for _, number := range numbers {
		sum += number
	}

	return sum
} 
```

测试此函数的惯用方法是创建一个名为`numbers_test.go`的第二个文件，并使用上述`testing`包对其进行测试：

```
package numbers

import (
	"testing"
)

func Test_addNumbers(t *testing.T) {
	if addNumbers(3, 7) != 10 {
		t.FailNow()
	}
} 
```

然后我可以运行`go test`并查看测试的输出：

```
ok      github.com/example/package/numbers 
```

或者，如果测试失败（即我期望的结果是11），我会得到一个合适的错误报告：

```
--- FAIL: Test_addNumbers (0.00s)
FAIL
FAIL    github.com/example/package/numbers    0.255s
FAIL 
```

总体而言，这方法运行得相当不错。当我需要找到特定文件的测试时，通常可以假设它们位于相应的 `_test.go` 文件中。`testing` 框架内置于标准库中，对这段代码的处理正是你所期待的。

### 我们拥有技术[⌗](#we-have-the-technology)

但这并不是我实际喜欢编写Go单元测试的方式。当事情出错时，我使用的技术和库可以使Go中的单元测试更有效率和易于理解。

### 测试命令[⌗](#the-test-command)

您可以在给定目录中运行`go test`并查看测试的输出，但这会留下许多功能未使用的可能性。我通常使用的完整命令是：

`go test -cover -shuffle=on -race -vet=all -failfast <$RELEVANT_PACKAGE(S)>`

这些标志各自的作用：

### 表驱动测试[⌗](#table-driven-tests)

在Go的潮流中，我要说一些可能有争议的话：我发现表驱动测试很少有用。对于像我们的`addNumbers`函数这样的简单输入输出，我可能会真的去做，因为这样做起来很简单。这可能是这样的：

```
package numbers

import (
	"testing"
)

type addNumbersTestCase struct {
  inputs []int
  expectedOutput int
}

func Test_addNumbers_table(t *testing.T) {
	testCases := []addNumbersTestCase{
		{
			inputs:         []int{4, 4},
			expectedOutput: 8,
		},
		{
			inputs:         []int{10, 0},
			expectedOutput: 10,
		},
		{
			inputs:         []int{-1, 1},
			expectedOutput: 0,
		},
	}

	for _, testCase := range testCases {
		actual := addNumbers(testCase.inputs...)
		if testCase.expectedOutput != actual {
			t.FailNow()
		}
	}
} 
```

尽管如此，很容易陷入一种情况，即你的表驱动测试需要变得非常复杂和棘手才能达到完全覆盖。

Goland允许您使用默认模板为函数生成测试，该模板试图遵循表驱动测试的传统。这是一个在旁边项目中生成的测试函数示例。该方法从数据库获取一个webhook对象，并接受3个参数 - 上下文、webhook ID 和 accountID：

```
package whatever

func TestQuerier_GetWebhook(t *testing.T) {
	type fields struct {
		tracer                  tracing.Tracer
		logger                  logging.Logger
		secretGenerator         random.Generator
		oauth2ClientTokenEncDec encryption.EncryptorDecryptor
		generatedQuerier        generated.Querier
		timeFunc                func() time.Time
		config                  *dbconfig.Config
		db                      *sql.DB
		migrateOnce             sync.Once
	}
	type args struct {
		ctx         context.Context
		webhookID   string
		accountID string
	}
	tests := []struct {
		name    string
		fields  fields
		args    args
		want    *types.Webhook
		wantErr assert.ErrorAssertionFunc
	}{
		// TODO: Add test cases.  }
	for _, tt := range tests {
		t.Run(tt.name, func(t *testing.T) {
			q := &Querier{
				tracer:                  tt.fields.tracer,
				logger:                  tt.fields.logger,
				secretGenerator:         tt.fields.secretGenerator,
				oauth2ClientTokenEncDec: tt.fields.oauth2ClientTokenEncDec,
				generatedQuerier:        tt.fields.generatedQuerier,
				timeFunc:                tt.fields.timeFunc,
				config:                  tt.fields.config,
				db:                      tt.fields.db,
				migrateOnce:             tt.fields.migrateOnce,
			}
			got, err := q.GetWebhook(tt.args.ctx, tt.args.webhookID, tt.args.accountID)
			if !tt.wantErr(t, err, fmt.Sprintf("GetWebhook(%v, %v, %v)", tt.args.ctx, tt.args.webhookID, tt.args.accountID)) {
				return
			}
			assert.Equalf(t, tt.want, got, "GetWebhook(%v, %v, %v)", tt.args.ctx, tt.args.webhookID, tt.args.accountID)
		})
	}
} 
```

我觉得这有点混乱，甚至还没有包括测试用例。我更愿意将查询器构建为仅在测试中可用的构造函数，该构造函数为所有这些值设置合理的默认值。

这并不是为了贬低Goland（我是一个快乐的付费用户，已经多年了），而是为了说明即使是一个非常复杂的函数，也可能会显著降低表驱动测试的可读性。

### 并行[⌗](#parallel)

Go 测试可以并行运行，通过在顶层将测试声明为并行来实现。这个行为默认开启，并且只能通过在调用 `go test` 时设置 `-parallel=1` 标志来禁用（在调试不稳定测试时很有用）。

我的观点是，除非绝对不能并行运行，否则测试应该是并行的。即使不能并行，我也更愿意修复不能并行运行的原因，而不是强制顺序运行它们。

通常，在并行运行测试时，即使没有使用竞态检测器（你应该使用），也可以更早地发现并发错误，并且最终会导致你在编写代码时养成编写并发安全代码的习惯。因此，通常情况下，我编写测试时的第一行就是并行声明：

```
package numbers

import (
	"testing"
)

func Test_addNumbers(t *testing.T) {
	t.Parallel()

	expected := 10
	actual := addNumbers(3, 7)

	if expected != actual {
		t.FailNow()
	}
} 
```

### 子测试[⌗](#subtests)

Go 的测试库中的子测试（Subtests）是一个特性，允许你为特定情况编写更小的测试。这些测试让你能够获得行为驱动测试风格的一些好处，而无需调用外部依赖。即使只关心一个测试用例，我通常也会编写子测试，因为这样做几乎没有额外开销，并且在被测试函数变得更复杂时，可以轻松地测试新的情况。

当我编写子测试时，我通常会将顶层的 `*testing.T` 对象命名为大写的 `T`，并且仅在子测试中使用传统的小写 `t`。这样子测试逻辑看起来总是像 Go 的习惯性测试，而且不会混淆变量是哪个或者哪个变量会遮蔽哪个变量。对于大 `*T` 和小 `*T` 变量都需要调用 `.Parallel()`。

这里是一个演示，调整我们之前为 `addNumbers` 函数编写的示例：

```
package numbers

import (
	"testing"
)

func Test_addNumbers(T *testing.T) {
	T.Parallel()

	T.Run("standard", func(t *testing.T) {
		t.Parallel()

		expected := 10
		actual := addNumbers(3, 7)

		if expected != actual {
			t.FailNow()
		}
	})
} 
```

现在，如果我决定添加另一个测试用例，只需复制子测试块，更改名称，然后使测试与描述匹配即可：

```
package numbers

import (
	"testing"
)

func Test_addNumbers(T *testing.T) {
	T.Parallel()

	T.Run("standard", func(t *testing.T) {
		t.Parallel()

		expected := 10
		actual := addNumbers(3, 7)

		if expected != actual {
			t.FailNow()
		}
	})

	T.Run("with many numbers", func(t *testing.T) {
		t.Parallel()

		expected := 15
		actual := addNumbers(1, 2, 3, 4, 5)

		if expected != actual {
			t.FailNow()
		}
	})

	T.Run("with negative numbers", func(t *testing.T) {
		t.Parallel()

		expected := 3
		actual := addNumbers(4, -1)

		if expected != actual {
			t.FailNow()
		}
	})
} 
```

此外，如果有与所有子测试相关的测试固件或常量，它们可以存在于调用`.Parallel()`和第一个声明的子测试之间的空间中。

### 库[⌗](#libraries)

虽然标准库的测试包非常出色，但社区对 Go 生态系统的贡献使得测试变得更加完善。

### 伪造值[⌗](#fake-values)

通常，当我需要在测试中与一个对象交互时，我希望对于该对象尽可能地具有不可预测的值，这样在编写测试时就不会依赖于这些约定。

比如说，如果我需要一个 `*User` 对象用于测试，我希望不知道它的用户名可能是什么，而不是一个常见的静态值，可能会从一个地方复制/粘贴到另一个地方，这样做如果被无意中更改会导致许多测试失败。

我喜欢使用 [brianvoe/gofakeit](https://pkg.go.dev/github.com/brianvoe/gofakeit/v7)。它提供了大量合理的函数来构建不同类型的虚假实例，我从未遇到过问题。这是一个构建虚假用户的示例：

```
package whatever

import (
	"github.com/example/project/internal/authorization"
	"github.com/example/project/internal/pkg/pointer"
	"github.com/example/project/pkg/types"

	fake "github.com/brianvoe/gofakeit/v7"
)

func buildFakeTime() time.Time {
	return fake.Date().Add(0).Truncate(time.Second).UTC()
}

func BuildFakeUser() *types.User {
	fakeDate := buildFakeTime()

	return &types.User{
		ID:                        identifiers.New(),
		FirstName:                 fake.FirstName(),
		LastName:                  fake.LastName(),
		EmailAddress:              fake.Email(),
		Username:                  fmt.Sprintf("%s_%d_%s", fake.Username(), fake.Uint8(), fake.Username()),
		Birthday:                  pointer.To(buildFakeTime()),
		AccountStatus:             string(types.UnverifiedHouseholdStatus),
		TwoFactorSecret:           base32.StdEncoding.EncodeToString([]byte(fake.Password(false, true, true, false, false, 32))),
		TwoFactorSecretVerifiedAt: &fakeDate,
		ServiceRole:               authorization.ServiceUserRole.String(),
		CreatedAt:                 buildFakeTime(),
	}
} 
```

### Testify[⌗](#testify)

我非常喜欢 [testify](https://github.com/stretchr/testify) 中的`(assert|require|mock)`库。在我们的示例测试中，使用 testify 看起来像这样：

```
package numbers

import (
	"testing"

	"github.com/stretchr/testify/assert"
)

func Test_addNumbers(T *testing.T) {
	T.Parallel()

	T.Run("standard", func(t *testing.T) {
		t.Parallel()

		expected := 10
		actual := addNumbers(3, 7)

		assert.Equal(t, expected, actual)
	})
} 
```

我认为`assert.Equal`对于新接触代码库的用户来说，其功能是相当明显的。`testify`可能是我唯一全力支持并入标准库的库。

如果你希望测试在失败时停止使用 `require` 也可以。这通常适用于测试的依赖项，例如，如果我需要创建一个文件并将其传递给一个函数，我可能应该`require`在创建时没有错误发生：

```
package whatever

import (
	"os"
	"testing"

	"github.com/stretchr/testify/require"
)

func TestSomeFunction(T *testing.T) {
	T.Parallel()

	T.Run("standard", func(t *testing.T) {
		t.Parallel()

		f, err := os.Create("some_path.txt")
		require.NoError(t, err)
		require.NotNil(t, f)
	})
} 
```

`mock`也可以用于定义结构的模拟实现。假设我有以下接口：

```
package something

type UserRetriever interface {
	GetUser(ctx context.Context, userID string) (*User, error)
} 
```

为这个接口实现一个模拟只是一个简单的事情：

```
package something

import (
	"github.com/stretchr/testify/mock"
)

type mockUserRetriever struct {
	mock.Mock
}

func (m *mockUserRetriever) GetUser(ctx context.Context, userID string) (*User, error) {
	returnValues := m.Called(ctx, userID)

	return returnValues.Get(0).(*User), returnValues.Error(1)
} 
```

在测试中使用上述模拟看起来是这样的：

```
package something

import (
	"testing"

	"github.com/stretchr/testify/mock"
)

func TestUserRetrieval(T *testing.T) {
	T.Parallel()

	T.Run("standard", func(t *testing.T) {
		t.Parallel()

		expected := &User{ID: "something"}

		mur := &mockUserRetriever{}
		mur.On("GetUser", mock.Anything, expected.ID).Return(expected, nil)

		// pass our mock to whatever uses it and test it's functionality  mock.AssertExpectationsForObjects(t, mur)
	})
} 
```

我不会撒谎，我对`mock.Mock`确定哪些变量以及何时以何种顺序返回并不深感印象深刻，当事情出错时（例如，你复制并粘贴了顺序错误或者返回值的索引错误时），它非常难以理清，但总体而言，`testify/mock`做了它需要做的事情。

通过编写类型匹配器，你可以避免在期望调用中放置`mock.Anything`。对于`context.Context`，它看起来是这样的：

```
mock.MatchedBy(func(context.Context) bool { return true }) 
```

还有`testify/suite`，我*曾经*使用过，但不推荐。当你需要大量公共前提条件用于预先设置测试时，它很有用，但我仍然认为更合理的做法是将这些前提条件作为一个通用函数的产物，而不是编写非标准的测试代码。

### Testcontainers[⌗](#testcontainers)

这是一个我最近才开始使用的很好的工具，效果非常好。通常需要编写与数据库、k/v 存储或内存中缓存进行交互的代码，但在有了 [testcontainers](https://testcontainers.com/) 之前，你要么必须编写一套集成测试来使用该代码来验证其是否按预期工作，要么在生产环境中进行盲目测试。使用 testcontainers，我可以启动临时容器来运行指定的软件，并让我的代码与之交互以验证功能。

在实践中，它并不完美。这些测试通常在我的机器上运行得很好，并且在 Github Actions 中也是如此，但你必须记住，你要求你的计算机在自身内部创建一个虚拟计算机，在该虚拟计算机上运行一个软件，并处理所有与其接口相关的网络花絮，因此存在大量机会出现问题和错误。

有时候测试容器的调用会失败，其原因可以总结为：计算机很难。由于它们可能比标准测试更容易出错，我通常不为这些写子测试。相反，我倾向于编写一个使用容器的单个实例来测试一堆相关函数的大函数。

这里有一个测试前述 Webhook 检索函数的示例：

```
package whatever

import (
	"context"
	"database/sql"
	"fmt"
	"testing"

	"github.com/example/project/pkg/types"
	"github.com/example/project/pkg/types/converters"
	"github.com/example/project/pkg/types/fakes"

	"github.com/stretchr/testify/assert"
	"github.com/stretchr/testify/require"
	"gopkg.in/matryer/try.v1"
)

func buildDatabaseClientForTest(t *testing.T, ctx context.Context) (*Querier, *postgres.PostgresContainer) {
	t.Helper()

	dbUsername := fmt.Sprintf("%d", hashStringToNumber(t.Name()))
	testcontainers.Logger = log.New(io.Discard, "", log.LstdFlags)

	var container *postgres.PostgresContainer
	err := try.Do(func(attempt int) (bool, error) {
		var containerErr error
		container, containerErr = postgres.RunContainer(
			ctx,
			testcontainers.WithImage(defaultImage),
			postgres.WithDatabase(splitReverseConcat(dbUsername)),
			postgres.WithUsername(dbUsername),
			postgres.WithPassword(reverseString(dbUsername)),
			testcontainers.WithWaitStrategyAndDeadline(2*time.Minute, wait.ForLog("database system is ready to accept connections").WithOccurrence(2)),
		)

		return attempt < 5, containerErr
	})
	require.NoError(t, err)
	require.NotNil(t, container)

	connStr, err := container.ConnectionString(ctx, "sslmode=disable")
	require.NoError(t, err)

	dbc, err := ProvideDatabaseClient(ctx, logging.NewNoopLogger(), tracing.NewNoopTracerProvider(), &config.Config{ConnectionDetails: connStr, RunMigrations: true, OAuth2TokenEncryptionKey: "blahblahblahblahblahblahblahblah"})
	require.NoError(t, err)
	require.NotNil(t, dbc)

	return dbc, container
}

func createWebhookForTest(t *testing.T, ctx context.Context, exampleWebhook *types.Webhook, dbc *Querier) *types.Webhook {
	t.Helper()

	// create  if exampleWebhook == nil {
		exampleWebhook = fakes.BuildFakeWebhook()
	}
	dbInput := converters.ConvertWebhookToWebhookDatabaseCreationInput(exampleWebhook)

	created, err := dbc.CreateWebhook(ctx, dbInput)
	assert.NoError(t, err)
	require.NotNil(t, created)

	assert.Equal(t, exampleWebhook, created)

	webhook, err := dbc.GetWebhook(ctx, created.ID, created.BelongsToAccount)
	assert.NoError(t, err)
	assert.Equal(t, webhook, exampleWebhook)

	return created
}

func TestQuerier_Integration_Webhooks(t *testing.T) {
	ctx := context.Background()
	dbc, container := buildDatabaseClientForTest(t, ctx)

	databaseURI, err := container.ConnectionString(ctx)
	require.NoError(t, err)
	require.NotEmpty(t, databaseURI)

	defer func(t *testing.T) {
		t.Helper()
		assert.NoError(t, container.Terminate(ctx))
	}(t)

	user := createUserForTest(t, ctx, nil, dbc)
	accountID, err := dbc.GetDefaultAccountIDForUser(ctx, user.ID)
	require.NoError(t, err)
	require.NotEmpty(t, accountID)

	exampleWebhook := fakes.BuildFakeWebhook()
	exampleWebhook.BelongsToAccount = accountID
	createdWebhooks := []*types.Webhook{}

	// create  createdWebhooks = append(createdWebhooks, createWebhookForTest(t, ctx, exampleWebhook, dbc))

	// other tests would go here  // delete  for _, webhook := range createdWebhooks {
		assert.NoError(t, dbc.ArchiveWebhook(ctx, webhook.ID, accountID))

		var exists bool
		exists, err = dbc.WebhookExists(ctx, webhook.ID, accountID)
		assert.NoError(t, err)
		assert.False(t, exists)

		var y *types.Webhook
		y, err = dbc.GetWebhook(ctx, webhook.ID, accountID)
		assert.Nil(t, y)
		assert.Error(t, err)
		assert.ErrorIs(t, err, sql.ErrNoRows)
	}
} 
```

(我承认这段代码缺少一些函数定义，但它是为了让你有个概念，而不是复制/粘贴)

### 失败案例[⌗](#failure-cases)

Go 标准库中的一些函数会返回错误，你可能会将这些错误暴露给更高级别的调用站点，但为了测试这些错误是否被捕获并返回，这里有一些我设法在特定情况下造成失败的方式。

### JSON 编码[⌗](#json-encoding)

在 Go 代码中，将给定结构的实例转换为 JSON 是一种非常常见的操作。如果源结构不可表示，此函数可以返回错误，但实际上在 Go 中很难使结构不可表示。你基本上必须是故意的。

我实现这一点的方式是使用 [`json.Number`](https://pkg.go.dev/encoding/json#Number)，这是一个代表数字的字符串别名，并给它一个非数字值。这是它的样子：

```
type BreakableStruct struct {
		Thing json.Number
}

func TestRenderToJSON(T *testing.T) {
	T.Parallel()

	T.Run("with invalid structure", func(t *testing.T) {
		t.Parallel()

		x := &BreakableStruct{Thing: "stuff"}
		actual, err := json.Marshal(x)

		assert.Nil(t, actual)
		assert.Error(t, err)
	})
} 
```

### URL 解析[⌗](#url-parsing)

在 Go 中解析 URL [会返回一个错误](https://pkg.go.dev/net/url#Parse)，但试图强制这个错误发生对我来说非常令人困惑。我不得不深入到[`net/url`](https://pkg.go.dev/net/url) 标准库的实际测试案例中。这里有一些你可能认为会从 `url.Parse` 返回错误的输入，但实际上并不会：

+   一个空字符串

+   一个路径中带有双点的网址（即`https://google.com/../../var/data`）

+   只是一个方案（即`https://`）

+   一个带有表情符号的网址（即`https://😘.🔥`）

+   一个带有无效方案的网址（即`farts://`）

所有这些实际上都没有返回错误。我能够强制它发生的唯一方法是使用：

```
fmt.Sprintf(`%s://`, string(byte(127))) 
```

由于某种原因，这会导致 `url.Parse` 失败。

### 结论[⌗](#conclusion)

在许多其他语言中，你不仅需要评估测试库，还需要以符合该库期望的方式编写你的测试。Gophers 幸运地拥有一个彻底合适的解决方案，甚至还有一个积极的生态系统，让深入测试变得轻而易举。
