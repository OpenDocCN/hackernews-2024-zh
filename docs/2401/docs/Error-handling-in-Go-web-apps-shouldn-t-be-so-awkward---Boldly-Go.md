<!--yml
category: 未分类
date: 2024-05-27 14:40:30
-->

# Error handling in Go web apps shouldn't be so awkward - Boldly Go

> 来源：[https://boldlygo.tech/posts/2024-01-08-error-handling/](https://boldlygo.tech/posts/2024-01-08-error-handling/)

*Updated 2024-01-10 to include [Domain-specific errors](https://boldlygo.tech/posts/2024-01-08-error-handling/#domain-specific-error-codes) and [limitations](https://boldlygo.tech/posts/2024-01-08-error-handling/#other-limitations) sections.*

* * *

In this post I’m going to describe an error-handling pattern I’ve found to be fairly elegant when writing REST, gRPC, or other services in Go. I have three goals in writing this post:

1.  To explain the pattern I’ve implemented for a few clients, so that others developing on the same codebase will understand it.
2.  To give others a pattern they may wish to implement in their own applications.
3.  To solicit feedback. Is there a better pattern out there I haven’t seen yet? Are there tweaks I can make to this pattern to make it better?

For simplicty, the examples I’ll be discussing will be part of a REST API, using simple HTTP status codes. But the same principles can be used for gRPC services as well, or even arbitrary error codes for a CLI tool. Or even all of the above simultaneously, which I may expand on in a later post.

## The problem

Before I explain the pattern I use, let me explain what it replaces, so that we might understand the problems it’s meant to solve.

Let’s look at a simple HTTP handler, using the standard library’s [HandlerFunc](https://pkg.go.dev/net/http#HandlerFunc) pattern, which simply fetches a widget record from the database, and serves it to the client as JSON

```
func (s *Service) GetWidget(w http.ResponseWriter, r *http.Request) {  if err := r.ParseForm(); err != nil {  http.Error(w, err.Error(), http.StatusBadRequest)  return  }  id, err := strconv.Atoi(r.Form.Get("widget_id"))  if err != nil {  http.Error(w, err.Error(), http.StatusBadRequest)  return  }  widget, err := s.db.GetWidget(id)  if err != nil {  if errors.Is(err, sql.ErrNoRows) {  http.Error(w, err.Error(), http.StatusNotFound)  return  }  http.Error(w, err.Error(), http.StatusInternalServerError)  return  }  widgetJSON, err := json.Marshal(widget)  if err != nil {  http.Error(w, err.Error(), http.StatusInternalServerError)  return  }  w.Header.Set("Content-Type", "application/json")  w.Write(widgetJSON) } 
```

While this should be a more or less realistic example, it’s intentionally over-simplified from what I typically find in production services. In particular, I’ve never seen [`http.Error`](https://pkg.go.dev/net/http#Error) used in a real service. Much more likely, you’ll have a custom error format you want to send back. Possibly utilizing a JSON error response, with some additional error context or internal error codes, etc. Or maybe you want to render the error as HTML. In any case, I’ll assume your app replaces the `http.Error()` call with something more sophisticated. Which likely means your code is even more annoying and repetitive than what I’ve shown above.

That aside, let me call out a few specific problems I see with the above code:

*   The error handling is repetitive, and non-idiomatic. Go is (in)famous for its `if err != nil { return err }` idiom. Yet we can’t even use *that* here, because the [HandlerFunc](https://pkg.go.dev/net/http#HandlerFunc) signature doesn’t return an error. Instead, for every error, we must (a) serve the error, and separately (b) return.
*   We must explicitly handle the HTTP status for every error case. If you have dozens or hundreds of handlers (and you probably do), this quickly becomes repetitive, and error-prone. There’s no DRY here. In a single handler like this, maybe it’s not a big deal. But it would be nice if we had some sort of default HTTP status code for an error—probably 500 / Internal Server Error.
*   This handler has to concern itself with database internals. In particular, it checks whether we received a [`sql.ErrNoRows`](https://pkg.go.dev/database/sql#pkg-variables) error. The HTTP handler should be completely database agnostic, so this detail should not need to be exposed here. This is some ugly tight-coupling we can get rid of.

## What if, instead…

What if, instead:

*   for every error, we could simply `return err`, and the right thing would happen? The error would be rendered to the proper format, and sent to the user?
*   the magic that renders the error would know the proper HTTP status to set, too? 400 for invalid input, 404 for not found, 401 for unauthorized access, etc?
*   the data store, whether an SQL database, or MongoDB, or the filesystem, would just tell us “this error means not found”, and that could be automatically converted to a 404 instead of the handler knowing the implementation details?

The pattern I’m about to describe gives us all of these things. Not only that, but it enables a number of other emergant patterns which are quite powerful as well. I’ll mention some of them at the end, and may write more extensively about some of them later on (let me know if that would interest you).

## Idiomatic error handling

The three behaviors I’ve described that we want all depend on two things, the first of which is “idiomatic error handling”. We need to be able to simply `return err` in our handlers. Unfortunately, the standard libray doesn’t give us this. But some third-party frameworks do. The most popular one I’m familiar with is [labstack echo](https://echo.labstack.com/), whose [`HandlerFunc`](https://pkg.go.dev/github.com/labstack/echo/v4#HandlerFunc) looks like this:

```
type HandlerFunc func(c Context) error 
```

But I don’t believe you should need to adopt a heavy framework like Echo just to get handy error-handling primitives. So you can do this yourself. Here’s a simple adapter function pattern you can use:

```
// customHandler converts an error-returning handler to a standard http.HandlerFunc. func customHandler(f func(http.ResponseWriter, *http.Request) error) http.HandlerFunc {  return func(w http.ResponseWriter, r *http.Request) {  err := f(w, r)  if err != nil {  http.Error(w, err.Error(), http.StatusInternalServerError) // Wait, alwyas 500? More on that later  }  } } 
```

With such an adapter function, our earlier handler gets simplified to:

```
func (s *Service) GetWidget(w http.ResponseWriter, r *http.Request) error {  if err := r.ParseForm(); err != nil {  return err  }  id, err := strconv.Atoi(r.Form.Get("widget_id"))  if err != nil {  return err  }  widget, err := s.db.GetWidget(id)  if err != nil {  return err  }  widgetJSON, err := json.Marshal(widget)  if err != nil {  return err  }  w.Header.Set("Content-Type", "application/json")  w.Write(widgetJSON) } 
```

Of course this requires that we actually *use* the adapter function when setting up our routes:

```
mux.Handle("/widget", customHandler(s.GetWidget)) 
```

Of course we’ve also actually *broken* this endpoint. It now treats *all* errors as internal server errors. So we’ll address that next.

## But first, an experiment I’m working on

But before we do, I want to call out an experimental library I’m working on, with the hope it may eventually become an official proposal to the standard library (although I think it’s a long shot it would be accepted) to extend the definition of the [`http.HandlerFunc`](https://pkg.go.dev/net/http#HandlerFunc) type to include an optional error return value. The library is [gitlab.com/flimzy/httpe](https://pkg.go.dev/gitlab.com/flimzy/httpe), and it adds -`WithError` variants to `http.Handler`, `http.HandlerFunc`, `ServeHTTP`, and related middlewares. It’s based on work I’ve been using for years with clients, but now living in its own stand-alone library for easy inclusion, if you wish.

If you choose to use this library, the new version of the handler remains unchanged, but in places of calling `customHandler`, you could do:

```
import "gitlab.com/flimzy/httpe"   mux.Handle("/widget", httpe.ToHandler(s.GetWidget)) 
```

The main advantage to the `httpe` library over your own custom handler is that it provides support for middleware adapters, and inter-mixing standard and error-enabled handlers, with some behind-the-scenes error propagation. But that’s beyond the scope of this post.

## How to handle different HTTP statuses

The second thing these improvements depends on is some way to specify an HTTP status. We’ve observed that while this new handler pattern makes error handling *simpler*, it also breaks it, by treating all errors as 500 / Internal Server Error (or whatever arbitrary status you set in your `customHandler` function). Let’s address that now.

### Errors are interfaces

Recall that in Go, the `error` type is an interface type, define as:

```
type error interface {  Error() string } 
```

This is powerful, because it means we can create our own custom error types. And what’s more, for our purposes, we can *extend* the error type to include other methods.

We want to take advantage of both of these capabilities to create a custom error type that includes an HTTP status, and add a method to expose that status. Here’s the simple custom type we’ll be using:

```
type statusError struct {  error  status int } 
```

Now already this is a “complete” error type. It already satisfies the `error` interface by virtue of embedding the `error` type (so its methods are promoted to our type’s methods). And it includes a status code. But we need a couple more pieces to make this complete. First, let’s add an `Unwrap` method, to allow [`errors.Unwrap`](https://pkg.go.dev/errors#Unwrap) and the related [`errors.Is`](https://pkg.go.dev/errors#Is) and [`errors.As`](https://pkg.go.dev/errors#As), etc, to work properly:

```
func (e statusError) Unwrap() error {  return e.error } 
```

And now we also want to add a method to expose the included status. Strictly speaking, this isn’t *necessary*. You *can* get at the status code by type-converting an error back to the `statusError` type with a type assertion, or with the use of `errors.Is` or `errors.As`. But it’s a bit cubersome, and requires exporting the field (unless your entire application is in a single package–I sure hope that’s not the case!) Further, by exposing the status via an interface method, we have the freedom to use multiple implementations of our custom error type, which is something I virtually always do. So let’s add that detail:

```
func (e statusError) HTTPStatus() int {  return e.status } 
```

Now, you could name your method whatever you want. I’ve settled on `HTTPStatus`, after initially using simply `Status()`, because it’s less ambiguous, but still short enough not to be annoying. You can just as eaisly use any other method (or multiple methods). For example, maybe you want `JSONRPCStatus()` if you’re building a JSON-RPC service. Or if you’re building a gRPC service, there’s already an interface defined for you: `GRPCStatus() *status.Status`.

## Using our custom error type

Now that we have our `statusError` type, let’s incorporate it into our handler, to un-break our status code handling:

```
func (s *Service) GetWidget(w http.ResponseWriter, r *http.Request) error {  if err := r.ParseForm(); err != nil {  return statusError{error: err, status: http.StatusBadRequest}  }  id, err := strconv.Atoi(r.Form.Get("widget_id"))  if err != nil {  return statusError{error: err, status: http.StatusBadRequest}  }  widget, err := s.db.GetWidget(id)  if err != nil {  return statusError{error: err, status: http.StatusInternalServerError}  }  widgetJSON, err := json.Marshal(widget)  if err != nil {  return statusError{error: err, status: http.StatusInternalServerError}  }  w.Header.Set("Content-Type", "application/json")  w.Write(widgetJSON) } 
```

So now we’ve (mostly) unbroken our status codes. The one exception is the database call. We treat all errors as status 500, when we should treat a missing widget as 404\. The solution here is to make our data access layer aware of these new error types:

```
func (db *DB) GetWidget(id int) (*Widget, error) {  widget, err := db.Get(/* ... */)  if errors.Is(err, sql.ErrNoRows) {  return nil, statusError{error: err, status: http.StatusNotFound}  }  if err != nil {  return nil, statusError{error: err, status: http.StatusInternalServerError}  }  return widget, nil } 
```

One last thing: We need to update our `customHandler` to understand this new error type:

```
// customHandler converts an error-returning handler to a standard http.HandlerFunc. func customHandler(f func(http.ResponseWriter, *http.Request) error) http.HandlerFunc {  return func(w http.ResponseWriter, r *http.Request) {  err := f(w, r)  if err != nil {  var status int  var statusErr interface {  error  HTTPStatus() int  }  if errors.As(err, &statusErr) {  status = statusErr.HTTPStatus()  }  http.Error(w, err.Error(), status)  }  } } 
```

Okay… so now we’re back to a fully-functional widget handler. And we’ve also decoupled our databaes logic from our HTTP handler. So that’s a win.

## Further improvments

But our handler is still kind of ugly, with a bunch of repeated `customErrror{}` structs. We also have both our handler and our data access layer depending on a concrete `statusError` type. Which isn’t even exported, which implies that our data access layer and handler are in the same package. Ick. We really don’t want that. So let’s move our custom error type to its own package. And we’ll add a convenient and descriptive constructor function, as well.

```
package apperr // Use a descriptive name  type statusError struct {  error  status int }   func (e statusError) Unwrap() error   { return e.error } func (e statusError) HTTPStatus() int { return e.status }   func WithHTTPStatus(err error, status int) error {  return statusError{  error: err,  status: int,  } } 
```

Now our handler can be updated to the slightly more readable:

```
func (s *Service) GetWidget(w http.ResponseWriter, r *http.Request) error {  if err := r.ParseForm(); err != nil {  return apperr.WithStatus(err, http.StatusBadRequest)  }  id, err := strconv.Atoi(r.Form.Get("widget_id"))  if err != nil {  return apperr.WithStatus(err, http.StatusBadRequest)  }  widget, err := s.db.GetWidget(id)  if err != nil {  return err // No call to apperr.WithStatus here, as we trust the db has already set the appropriate status code for us  }  widgetJSON, err := json.Marshal(widget)  if err != nil {  return apperr.WithStatus(err, http.StatusBadRequest)  }  w.Header.Set("Content-Type", "application/json")  w.Write(widgetJSON) } 
```

## Setting a default status

We can do one other big improvement to this setup: Setting a default status.

In fact, you may have noticed that our improved `customHandler` func *has no default status*. This means that if we pass it an error that doesn’t include an HTTP status, it will try to serve an HTTP response with status of `0`. Probably not ideal.

Let’s solve this problem by adding a helper function to our `apperr` package, which can also be used from other places:

```
package apperr   // HTTPStatus returns the HTTP status included in err. If err is nil, this // function returns 0\. If err is non-nil, and does not include an HTTP status, // a default value of [net/http.StatusInternalServerError] is returned. func HTTPStatus(err error) int {  if err == nil {  return 0  }  var statusErr interface {  error  HTTPStatus() int  }  if errors.As(err, &statusErr) {  return statusErr.HTTPStatus()  }  return http.StatusInternalServerError } 
```

With this new function in our pocket, our custom handler can be simplified and improved:

```
// customHandler converts an error-returning handler to a standard http.HandlerFunc. func customHandler(f func(http.ResponseWriter, *http.Request) error) http.HandlerFunc {  return func(w http.ResponseWriter, r *http.Request) {  err := f(w, r)  if err != nil {  http.Error(w, err.Error(), apperr.HTTPStatus(err))  }  } } 
```

And our handler can also be improved by omitting the call to `apierr.WithStatus` when we want the default status of 500 / Internal Server Error.

## Further improvements

This is really only the beginning of what using an app-wide standard error extension and matching handlers can provide.

The next areas I usually improve on apps where I implement this pattern is to add a couple standard middlewares:

### Logging middleware

A middleware to log, with an associated error, if relevant, is a great addition. And it eliminates the “need” to log an error every time it happens–just pass it to your caller, and let the middleware log it for you. Here’s a simplified example, using the function signatures in [gitlab.com/flimzy/httpe](https://pkg.go.dev/gitlab.com/flimzy/httpe):

```
func loggingMiddleware(logger *slog.Logger) func(next httpe.HandlerWithError) httpe.HandlerWithError {  return httpe.HandlerWithErrorFunc(func (w http.ResponseWriter, r *http.Request) error {  err := next.ServeHTTPWithError(w, r)  status := http.StatusOK  if err != nil {  status = apperr.HTTPStatus(err)  logger.Error("request served with error", "status", status, "request", r, "error", err)  } else {  logger.Info("request served", "status", status, "request", r)  }  }) } 
```

Keep in mind that a handler can still call `w.WriteHeader` with a status distinct from that contained in `err` (or even in the absence of an error). So a robust implementation will check for that as well.

### Error-serving middleware

An improvement over the `customHandler` function shown above, is to move the error serving into a middleware. This does require the `httpe` package, or a similar implemenatation, that can work with middlewares.

```
func serveErrors() func(next httpe.HandlerWithError) httpe.HandlerWithError {  return httpe.HandlerWithErrorFunc(func (w http.ResponseWriter, r *http.Request) error {  err := next.ServeHTTPWithError(w, r)  if err != nil {  http.Error(w, err.Error(), apperr.Status(err))  }  }) } 
```

### Panc-recovery middleware

Most web apps have this (or they should!), but a version that works with error-returning handlers can be nice, as it just has to convert panics to errors, rather than serving them directly:

```
func serveErrors() func(next httpe.HandlerWithError) httpe.HandlerWithError {  return httpe.HandlerWithErrorFunc(func (w http.ResponseWriter, r *http.Request) (err error) {  defer func() {  if r := recover(); r != nil {  switch t := r.(type) {  case error:  err = t  default:  err = fmt.Errorf("%v")  }  }  }()  return next.ServeHTTPWithError(w, r)  }) } 
```

## Domain-specific error codes

What’s better than having your entire application set HTTP statuses on errors all over the place, is to define your own domain-specific error codes. For a small, web-only app, maybe HTTP status codes are sufficient, but in most real-world apps, they aren’t. Everything else about this pattern can still be used with your own domain-specific error codes. Simply make your custom error types also return the appropriate HTTP (or JSON-RPC or gRPC or whatever…) codes as well. After such a change, our above database method might look more like:

```
func (db *DB) GetWidget(id int) (*Widget, error) {  widget, err := db.Get(/* ... */)  if errors.Is(err, sql.ErrNoRows) {  return nil, apperror.ErrWidgetNotFound  }  if err != nil {  return nil, err  }  return widget, nil } 
```

Then the various callers can do their own error inspection, as appropriate:

```
internalCode := apperror.Code(err) // The internal error code  httpStatus := apperror.HTTPStatus(err) // The HTTP status 
```

I leave the exact implementation of `apperror.ErrWidgetNotFound` and the associated functions as an exercise for the reader.

## Caveats

This approach doesn’t come without some drawbacks. It’s worth calling a few of them out.

1.  It’s non-standard. Obviously. There’s a cost in verbosity to be paid to convert from an error-returning handler to a standard handler. Although it seems many people are happy to pay this cost, in the form of adopting a heavy framework that offers this benefit (along with others, of course).

2.  There are now *two* ways to send responses to the client. This annoys me. But I have yet to find any way around it. And in practice, it doesn’t seem to be a big problem, but it does require keeping it in mind. You can set the HTTP status of a resonse *either* by calling `w.WriteHeader()`, *or* by returning an error. Each response can have only a single status, obviously. And if you call `w.WriteHeader()`, that one generally takes precident (unless you’ve implemented your own `http.ResponseWriter` with different behavior).

3.  It makes certain behaviors implicit. Or at least it can. As an example, the shown `apperr.HTTPStatus()` function returns a default status for errors that don’t contain a status. While I believe this makes good sense, and is a benefit, it is a bit “magical”, and may surprise someone who’s not familiar with the pattern at play. It can also be confusing to see `apperr.WithStatus(err, http.StatusNotFound)` the first time. While it should be apparent upon plain reading that it’s including an HTTP status with an error, it’s not apparent what other code consumes that status, or how its used. Of course, the purpose of this post is to help solve this drawback.

## Other limitations

This is by no means a one-size-fits-all solution. A couple of obvious limitations I have run into on various applications:

*   It doesn’t provide any ergonomic way to specify a non-200, non-error status (such as 201). For this, you still must fall back on `w.WriteHeader()`.
*   Some applications would be better served with a function signature such as `func(*http.Request) (any, error)`, such that a response (likely to be renderd as JSON) is the first return argument.

## What’s next?

I mentioned at the outset that this pattern tends to lend itself to additional improvements. Let me just mention a few, without going into detail here. If you’d like a longer explanation on any of these, [let me know](https://boldlygo.tech/contact).

*   Include additional metadata in errors. Stack traces are an obvious one, which is nicely provided by [`github.com/pkg/errors`](https://pkg.go.dev/github.com/pkg/errors), for example. Expand your logging middleware to extract the stack trace and include it in logs.
*   Hide error messages for certain HTTP statuses. I typically write my error-serving middleware to return simply `Internal Server Error` to the client any time there’s a 500 status, to avoid the risk of potentially reporting sensitive information, which can happen in some unsanitized errors. HTTP statuses 401 and 403 are also good candidates for this.
*   Similar to obscuring certain errors, maybe you want to expose a user-friendly version of an error message to the users of your app, while logging the nitty gritty details that the error originally included. Add a `Public() string` method to such errors, and send the `Public()` version to your users, and the `Error()` version to your logs.
*   HTTP statuses not detailed enough for you? Maybe you need to distinguish between `widget not found` and `user not found`? You can create your own internal error status/codes, for use internally, which resolve to a common HTTP status.
*   Look for ways to DRY up your code. For example, from the handler example, consider moving the call to `r.ParseForm` and `strconv.Atoi` to a common function—or use a validation library such as [github.com/go-playground/validator/v10](https://pkg.go.dev/github.com/go-playground/validator/v10) in place of `strconv` calls—which returns an error with 400 status. Then your handler can just pass that error through.

What other use cases have you seen, or can you think of? [I’d love to hear from you](https://boldlygo.tech/contact).

## Is there a better way?

I don’t know.

I’m always on the lookout for a better way to do things.

If you know of a better pattern, or even small ways I can improve on this pattern, please, let me know! I’d love to learn from you!