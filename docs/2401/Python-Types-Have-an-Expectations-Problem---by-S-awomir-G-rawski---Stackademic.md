<!--yml
category: 未分类
date: 2024-05-27 15:18:59
-->

# Python Types Have an Expectations Problem | by Sławomir Górawski | Stackademic

> 来源：[https://medium.com/@sgorawski/python-types-have-an-expectations-problem-ea71a8645ce8](https://medium.com/@sgorawski/python-types-have-an-expectations-problem-ea71a8645ce8)

# Python Types Have an Expectations Problem

Photo by [Hitesh Choudhary](https://unsplash.com/@hiteshchoudhary?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com/?utm_source=medium&utm_medium=referral)

# Background

In the last ~10 or so years, many of the popular scripting languages gained optional static typing: JavaScript (via TypeScript), PHP, Python, even Ruby has something along the lines as far as I know.

It’s become quite widespread in each of these languages’ communities, to the point of being considered a best practice for applications, and definitely for libraries.

It’s easy to see how type annotations are useful — let’s take this function that doesn’t have them:

```
def check_permission(user, perm, obj):
    ...
```

It may seem pretty clear what it does — it checks whether `user` has permission `perm` on `obj`.

However, if we wanted to use it somewhere, some questions immediately spring to mind:

1.  Is `user` an instance of the application’s User model? (I guess.)
2.  Is `perm` an instance of some Permission model or just some representation, like a string? That one’s a bit harder, probably depends on the conventions of the framework used and the specific project.
3.  Is `obj` an instance of any model, or maybe the model *type*? What if the permission checked is broader: should I pass null or is there a different function for that somewhere?
4.  Does the function return true/false if the user has / doesn’t have the permission? Or maybe it doesn’t return anything and throws an exception if the check is negative?

Compare that to the typed version:

```
def check_permission(user: User, perm: str, obj: BaseModel | None) -> bool:
    ...
```

It’s a bit longer, but it answers all our questions. (We’re not *absolutely* sure what the code will do, but we can reasonably guess.) Also, it can catch bugs and helps the IDEs with autocomplete.

So, if the type annotations are so useful, why do I claim that in Python they have an “expectations problem”?

Well, let’s compare how other languages are doing it.

# Comparison

## JavaScript

JavaScript doesn’t have any typing on its own, you have to use TypeScript for that. It’s a very popular choice, so it’s shouldn’t be hard to start with.

We can have something like this in the source code:

```
function checkPermission(user: User, perm: string, obj?: BaseModel) {
	console.log(user, perm, obj);
}

checkPermission(1, 2, 3);
```

Now, TypeScript is not gonna run in the browser (or Node.js) like this, we have to compile it:

```
$ node_modules/.bin/tsc example.ts
error TS2345: Argument of type 'number' is not assignable to parameter of type 'User'.
```

It’s not gonna let us. And so if we succeed in compiling and running it later we can be sure that the type mismatch was fixed and the program type checks.

## PHP

Now, let’s do something similar in PHP:

```
<?php
function checkPermission(User $user, string $perm, ?BaseModel $obj) {
    var_dump($user, $perm, $obj);
}

checkPermission(1, 2, 3);
```

There is no compile step in PHP. However, if we run this code, it will crash at runtime:

```
$ php -f example.php
Uncaught TypeError: checkPermission(): Argument #1 ($user) must be of type User, int given
```

Yeah, maybe throwing exceptions is not as good as catching that earlier, but at least we can be sure that the function code won’t be executed if the types are wrong.

## Python

Enter Python:

```
def check_permission(user: User, perm: str, obj: BaseModel | None):
    print(user, perm, obj)

check_permission(1, 2, 3)
```

```
$ python3 example.py
1 2 3
```

We ran this and the type annotations did nothing! It called the function with the arguments of the wrong types, as if we didn’t do anything to prevent that.

Now, to someone familiar with Python it’ll be very obvious — the type hints don’t work at runtime, they are just there for a type checker program, like MyPy, that you have to run separately.

But after what we saw with the other languages it may not be obvious at all.

In all my examples, I wrote the code using the most popular static typing solution of the given language, then I did the simplest thing I could to get the code running in my application. In case of TypeScript, I had to compile it to JS to run the code and it caught the error then. In PHP, I “just” ran it and it caught the type mismatch at runtime. In Python, I also could just run it — there was no compile step needed — and then my type hints at runtime did nothing!

So if I have the source code in a given language and a running application, that’s what I can know from that:

Now, the last answer for Python would be different if I knew that someone ran a correctly configured type checker on the codebase before running the code — but how can I know that? I’d have to set up some CI with the type checking step and make it impossible to deploy the code that doesn’t pass it, maybe. If I don’t have that kind of infrastructure, there’s not much I can do.

Python type hints are a core part of the language, they even have standard library modules (`typing`), and yet they don’t do anything when used in that language without some external tooling.

That, to me, is a bit of an expectations mismatch.

# Conclusions

Now, if the Python type annotations actually did look like what they are — information for external tools, not a “working” part of the language — it would probably be clearer. Like comments. And there actually *is* an official syntax for that!

```
def check_permission(user, perm, obj):
    # type: (User, str, BaseModel | None) -> bool
    return False
```

It works with MyPy, but it’s quite niche at this point (was intended for compatibility with old Python versions), doesn’t work too well with tools like PyCharm and probably can’t express everything that the annotations can.

A shame, because to me it communicates expectations better — it’s a comment, so it’s an information for an external tool and it doesn’t work at runtime. All expected.

# Stackademic 🎓

Thank you for reading until the end. Before you go: