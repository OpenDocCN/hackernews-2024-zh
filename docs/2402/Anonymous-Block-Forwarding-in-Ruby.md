<!--yml
category: 未分类
date: 2024-05-27 14:54:10
-->

# Anonymous Block Forwarding in Ruby

> 来源：[https://www.writesoftwarewell.com/anonymous-block-forwarding-in-ruby/](https://www.writesoftwarewell.com/anonymous-block-forwarding-in-ruby/)

You probably have used (or at least heard about) the argument forwarding feature in Ruby, which was added in Ruby 2.7, that lets you forward all the arguments passed to a method, as they are, to another method, using the three dots `...` syntax.

**This includes all positional, named, and even block arguments.**

```
def execute(action="", **options)
  p action   # get
  p options  # {:url=>"/test", :values=>[201, 202]}

  yield "fake response"
end

def perform(...)
  # execute receives all the arguments 
  # received by the perform function
  execute(...)
end

perform action="get", url: "/test", values: [201, 202] do |result|
  p result  # "fake response"
end
```

**Did you know that there's a similar shorthand to forward a block anonymously?** I learned this very recently (yesterday!), and I love it.

Notice the `perform` method in the following example:

```
def execute(&blk)
  yield "response 1"
  blk.call "response 2"
end

def perform(&)
  # execute receives the same block
  # received by the perform function
  execute(&)
end

perform do |result|
  p result  
end

# Output:
# "response 1"
# "response 2"
```

Earlier, you'd have done something like:

```
def perform(&blk)
  execute(&blk)
end
```

But starting in Ruby 3.1, you can do this:

```
def perform(&)
  execute(&)
end
```

Pretty cool, right? Also looks pretty nice, in my opinion. What do you think?

If you like this syntax, and would like to enforce it via Rubocop ([Rails 8 ships with a default Rubocop linter out of box](https://github.com/rails/rubocop-rails-omakase?ref=writesoftwarewell.com)), you can use the following Rule:

```
Naming/BlockForwarding:
  EnforcedStyle: anonymous
```

On the other hand, if you prefer the older explicit version, enforce it with this rule:

```
Naming/BlockForwarding:
  EnforcedStyle: explicit
```

P.S. For an in-depth explanation, I suggest reading [this post](https://zverok.space/blog/2023-11-24-syntax-sugar4-argument-forwarding.html?ref=writesoftwarewell.com) from zverok. It goes into much more depth.

* * *

That's a wrap. I hope you found this article helpful and you learned something new.

As always, if you have any questions or feedback, didn't understand something, or found a mistake, please leave a comment below or [send me an email](mailto:akshay.khot@hey.com?ref=akshays-blog). I reply to all emails I get from developers, and I look forward to hearing from you.

If you'd like to receive future articles directly in your email, please [subscribe to my blog](#/portal/signup). If you're already a subscriber, thank you.

Subscribe to get the new posts via email. Your email address is never sold or shared.