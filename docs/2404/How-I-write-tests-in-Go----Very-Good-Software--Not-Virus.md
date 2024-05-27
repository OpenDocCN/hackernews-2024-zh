<!--yml
category: 未分类
date: 2024-05-27 13:04:55
-->

# How I write tests in Go :: Very Good Software, Not Virus

> 来源：[https://blog.verygoodsoftwarenotvirus.ru/posts/testing-in-go/](https://blog.verygoodsoftwarenotvirus.ru/posts/testing-in-go/)

# How I write unit tests in Go[⌗](#how-i-write-unit-tests-in-go)

One of my favorite features of Go is that unlike many popular languages, it comes with it’s own testing framework, the [testing](https://pkg.go.dev/testing) package. Let’s say we have this trivial function in a file called `numbers.go`:

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

The idiomatic way to test this function is to create a second file called `numbers_test.go` and test it using the aforementioned `testing` package:

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

Then I can run `go test` and see the output of the test:

```
ok      github.com/example/package/numbers 
```

Alternatively, if the test fails (i.e. if I expect the result to be 11 instead), I get an appropriate error report:

```
--- FAIL: Test_addNumbers (0.00s)
FAIL
FAIL    github.com/example/package/numbers    0.255s
FAIL 
```

This works pretty well, overall. When I need to find the tests for a given file, I can usually assume they’re in its corresponding `_test.go` file. The `testing` framework is built into the standard library, and does exactly what you’d expect with this code.

### We have the technology[⌗](#we-have-the-technology)

This isn’t how I actually like to write Go unit tests, however. There are techniques and libraries I use to make unit testing in Go more effective to write and understand when things go (heh) awry.

### The test command[⌗](#the-test-command)

You can run `go test` in a given directory and see the output of your test, but that leaves a lot of functionality on the table that can and will be useful. The full command I typically use is:

`go test -cover -shuffle=on -race -vet=all -failfast <$RELEVANT_PACKAGE(S)>`

Here’s what each of these flags do:

### Table-driven tests[⌗](#table-driven-tests)

I’m going to say something sort of controversial in the Go zeitgeist: I find table tests very rarely useful. For something like our `addNumbers` function, I might actually do it, because the inputs and outputs are simple. Here’s what that might look like:

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

That said, it’s pretty easy to end up in a situation where your table tests need to be quite complicated and gnarly to get full coverage.

Goland lets you generate tests for functions using a default template that attempts to adhere to the tradition of table-driven tests. Here’s what the generated test function looks like for a method in a side project. This method fetches a webhook object from the database, and accepts 3 total parameters — a context, a webhook ID, and an accountID:

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

I think this is kind of a mess, and it doesn’t even include test cases yet. I’d rather have the querier built by an only-available-to-tests constructor that puts sensible defaults for all these values.

The point of this is not to dunk on Goland (I am a happily paying customer and have been for many years), it’s just to illustrate how even a non-trivial function can really blow up the readability of a table-driven test.

### Parallel[⌗](#parallel)

Go tests can be run in parallel, by declaring the test as parallel in the top level. This behavior is on by default, and can only be disabled by setting the `-parallel=1` flag when invoking `go test` (which can be useful when debugging flaky tests).

I’m of the opinion that tests should be parallel unless they absolutely cannot be, and even then, I’d rather fix the flaw behind why they can’t be run in parallel than force them to be run sequentially.

Running your tests in parallel generally allows you to suss out concurrency bugs earlier, even if you’re not using the race detector (which you should), and will eventually lead to you developing concurrency-safe habits when writing code in the first place. So generally, when I write a test, the very first line is the parallel declaration:

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

### Subtests[⌗](#subtests)

Subtests are a feature of Go’s testing library that allow you to have smaller tests for specific circumstances. These allow you to get some of the benefits of a behavior-driven testing style, without having to invoke an external dependency. I usually write subtests even when there’s only one test case I care about, because it incurs no real overhead to do so, and allows you to easily test a new case in the event the tested function gets more complex.

When I write subtests, I usually name the top-level `*testing.T` object a capitalized `T`, and use the conventional lower-cased `t` for subtests only. This way, the subtest logic always looks like an idiomatic Go test, and there’s no confusion of what variable is which or what shadows what. Calls to `.Parallel()` will have to be made for both the big and little `*T` variables.

Here’s a demonstration that adapts our earlier example for the `addNumbers` function:

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

Now if I decide to add another test case, it’s simply a matter of copying the subtest block, changing the name, and then making the test match that description:

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

Additionally, if there are test fixtures or constants that are relevant to all subtests, they can live in the space between the big-T call to `.Parallel()` and the first declared subtest.

### Libraries[⌗](#libraries)

While the standard library testing package is pretty great, there have been some great contributions by the community to the ecosystem that make testing in Go even better.

### Fake Values[⌗](#fake-values)

Typically, when I need to interact with an object in a test, I want to have unpredictable values to the extent possible for that object, so that I’m discouraged from relying on those conventions when writing tests.

So for instance, if I need a `*User` object for a test, I want to have no idea what it’s username might be, as opposed to a common static value that I might copy/paste from place to place, which would make many tests fail if it was inadvertently changed.

I like to use [brianvoe/gofakeit](https://pkg.go.dev/github.com/brianvoe/gofakeit/v7) for this. It comes with a ton of sensible functions with which to build fake instances of different types, and I’ve never had a problem with it. Here’s what that fake user builder might look like:

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

I’m a huge fan of the `(assert|require|mock)` libraries in [testify](https://github.com/stretchr/testify). In our example test, using testify would look like:

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

I think `assert.Equal` is fairly obvious in its function for users who are new to a code base. `testify` is probably the only library I can think of that I would wholeheartedly support being absorbed into the standard library.

You can use `require` instead if you want the test to stop in the event of a failure. This is usually good for dependencies of a test, i.e. if I need to create a file and pass it to a function, I should probably `require` that no errors happen when creating it:

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

`mock` is also useful for defining mock implementations of structures. Say I have the following interface:

```
package something

type UserRetriever interface {
	GetUser(ctx context.Context, userID string) (*User, error)
} 
```

Implementing a mock for this interface becomes a matter of:

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

Using the above mock in a test looks like this:

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

I won’t lie, I’m not deeply impressed with the way that `mock.Mock` determines which variables to return in what order, and when things go wrong (i.e. you copy and paste something in the wrong order or have the wrong indices on your return values) it’s VERY hard to suss out, but by and large, `testify/mock` does an incredible job of what it needs to do.

You can avoid having to put `mock.Anything` in an expectation call by writing a type matcher. For `context.Context`, it would look like this:

```
mock.MatchedBy(func(context.Context) bool { return true }) 
```

There’s also `testify/suite`, which I *have* used, but wouldn’t recommend. It’s useful when you have a ton of common prerequisites for tests that need to be spun up ahead of time, but I still think it makes more sense to just make those prerequisites the product of a common function, instead of writing non-standard test code.

### Testcontainers[⌗](#testcontainers)

This is a neat tool I’ve only recently started using, but to great effect. It’s very common to have to write code that interfaces with a database, or a k/v store, or an in-memory cache, but before [testcontainers](https://testcontainers.com/), you either had to write a suite of integration tests that used that code to verify it did what you wanted, or otherwise fly blind in production. With testcontainers, I can spin up ephemeral containers that run a given piece of software and have my code interact with that to verify functionality.

In practice, it’s not perfect. These tests generally run great on my machine, and in Github Actions, but you must remember that you’re asking your computer to make a fake computer within itself, spin up a piece of software on that fake computer, and handle all the networking shenanigans that go along with interfacing with it, so there’s plenty of opportunity for mishaps and errors to occur.

Sometimes a testcontainer invocation will just fail, and the reason behind it can be summarized up as: computers are hard. Since they can be a big more error prone than standard tests, I don’t typically write subtests for these. Instead, I tend to write one big function that uses a single instance of the container to test a bunch of related functions.

Here’s an example of the test for that aforementioned webhook retrieval function:

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

(I acknowledge that this code is missing some function definitions, but it’s meant for you to get an idea, not copy/paste)

### Failure Cases[⌗](#failure-cases)

Some functions in the Go standard library return errors that you might surface to higher-level call sites, but in order to test that those errors are caught and returned, Here are some ways I manage to achieve failures for very specific circumstances in Go.

### JSON Encoding[⌗](#json-encoding)

It’s a pretty common operation in Go code to take an instance of a given struct and render it as JSON. This function is capable of returning an error in the event the source struct is unrepresentable, but it’s actually pretty hard to make a struct unrepresentable in Go. You basically have to do it on purpose.

The way I achieve this is by using [`json.Number`](https://pkg.go.dev/encoding/json#Number), a string alias meant to represent numbers, and giving it a non-numerical value. Here’s what that looks like:

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

### URL Parsing[⌗](#url-parsing)

Parsing a URL in Go [returns an error](https://pkg.go.dev/net/url#Parse), but trying to force this error to occur was very confusing to me. I had to dive into the actual test cases in the standard library for [`net/url`](https://pkg.go.dev/net/url). Here are some inputs you might think return an error from `url.Parse`, but actually don’t:

*   an empty string
*   a URL with double dots in the path (i.e. `https://google.com/../../var/data`)
*   just a scheme (i.e. `https://`)
*   a URL with emojis in it (i.e. `https://😘.🔥`)
*   a URL with an invalid scheme (i.e. `farts://`)

All of these actually don’t return an error. The only way I was able to force it to occur by using:

```
fmt.Sprintf(`%s://`, string(byte(127))) 
```

For whatever reason, this causes `url.Parse` to fail.

### Conclusion[⌗](#conclusion)

In many other languages, you have to not only evaluate testing libraries, but also write your tests in a style that complies with that library’s expectations. Gophers are blessed to have a thoroughly adequate solution out-of-the-box, and even further blessed to have an active ecosystem where folks are making in-depth testing a walk in the park.