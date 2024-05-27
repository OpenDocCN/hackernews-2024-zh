<!--yml
category: 未分类
date: 2024-05-27 13:10:34
-->

# Metaprogramming in Ruby: It's All About the Self

> 来源：[https://yehudakatz.com/2009/11/15/metaprogramming-in-ruby-its-all-about-the-self/](https://yehudakatz.com/2009/11/15/metaprogramming-in-ruby-its-all-about-the-self/)

After writing my last post on Rails plugin idioms, I realized that Ruby metaprogramming, at its core, is actually quite simple.

It comes down to the fact that all Ruby code is executed code--there is no separate compile or runtime phase. In Ruby, every line of code is executed against a particular `self`. Consider the following five snippets:

```
class Person
  def self.species
    "Homo Sapien"
  end
end

class Person
  class << self
    def species
      "Homo Sapien"
    end
  end
end

class << Person
  def species
    "Homo Sapien"
  end
end

Person.instance_eval do
  def species
    "Homo Sapien"
  end
end

def Person.species
  "Homo Sapien"
end 
```

All five of these snippets define a `Person.species` that returns `Homo Sapien`. Now consider another set of snippets:

```
class Person
  def name
    "Matz"
  end
end

Person.class_eval do
  def name
    "Matz"
  end
end 
```

These snippets all define a method called `name` on the Person class. So `Person.new.name` will return "Matz". For those familiar with Ruby, this isn't news. When learning about metaprogramming, each of these snippets is presented in isolation: another mechanism for getting methods where they "belong". In fact, however, there is a single unified reason that all of these snippets work the way they do.

First, it is important to understand how Ruby's metaclass works. When you first learn Ruby, you learn about the concept of the class, and that each object in Ruby has one:

```
class Person
end

Person.class #=> Class

class Class
  def loud_name
    "#{name.upcase}!"
  end
end

Person.loud_name #=> "PERSON!" 
```

`Person` is an instance of `Class`, so any methods added to `Class` are available on `Person` as well. What they don't tell you, however, is that each object in Ruby also has its own **metaclass**, a `Class` that can have methods, but is only attached to the object itself.

```
matz = Object.new
def matz.speak
  "Place your burden to machine's shoulders"
end 
```

What's going on here is that we're adding the `speak` method to `matz`'s **metaclass**, and the `matz` object inherits from its **metaclass** and then `Object`. The reason this is somewhat less clear than ideal is that the metaclass is invisible in Ruby:

```
matz = Object.new
def matz.speak
  "Place your burden to machine's shoulders"
end

matz.class #=> Object 
```

In fact, `matz`'s "class" is its invisible metaclass. We can even get access to the metaclass:

```
metaclass = class << matz; self; end
metaclass.instance_methods.grep(/speak/) #=> ["speak"] 
```

At this point in other articles on this topic, you're probably struggling to keep all of the details in your head; it seems as though there are so many rules. And what's this `class << matz` thing anyway?

It turns out that all of these weird rules collapse down into a single concept: control over the `self` in a given part of the code. Let's go back and take a look at some of the snippets we looked at earlier:

```
class Person
  def name
    "Matz"
  end

  self.name #=> "Person"
end 
```

Here, we are adding the `name` method to the `Person` class. Once we say `class Person`, the `self` until the end of the block is the `Person` class itself.

```
Person.class_eval do
  def name
    "Matz"
  end

  self.name #=> "Person"
end 
```

Here, we're doing exactly the same thing: adding the `name` method to instances of the Person class. In this case, `class_eval` is setting the `self` to `Person` until the end of the block. This is all perfectly straight forward when dealing with classes, but it's equally straight forward when dealing with metaclasses:

```
def Person.species
  "Homo Sapien"
end

Person.name #=> "Person" 
```

As in the `matz` example earlier, we are defining the `species` method on `Person`'s metaclass. We have not manipulated `self`, but you can see using `def` with an object attaches the method to the object's metaclass.

```
class Person
  def self.species
    "Homo Sapien"
  end

  self.name #=> "Person"
end 
```

Here, we have opened the `Person` class, setting the `self` to `Person` for the duration of the block, as in the example above. However, we are defining a method on `Person`'s metaclass here, since we're defining the method on an object (`self`). Also, you can see that `self.name` while inside the person class is identical to `Person.name` while outside it.

```
class << Person
  def species
    "Homo Sapien"
  end

  self.name #=> ""
end 
```

Ruby provides a syntax for accessing an object's metaclass directly. By doing `class << Person`, we are setting `self` to `Person`'s metaclass for the duration of the block. As a result, the `species` method is added to `Person`'s metaclass, rather than the class itself.

```
class Person
  class << self
    def species
      "Homo Sapien"
    end

    self.name #=> ""
  end
end 
```

Here, we combine several of the techniques. First, we open `Person`, making `self` equal to the `Person` class. Next, we do `class << self`, making `self` equal to `Person`'s metaclass. When we then define the `species` method, it is defined on `Person`'s metaclass.

```
Person.instance_eval do
  def species
    "Homo Sapien"
  end

  self.name #=> "Person"
end 
```

The last case, `instance_eval`, actually does something interesting. It breaks apart the `self` into the `self` that is used to execute methods and the `self` that is used when new methods are defined. When `instance_eval` is used, new methods are defined on the **metaclass**, but the `self` is the object itself.

In some of these cases, the multiple ways to achieve the same thing arise naturally out of Ruby's semantics. After this explanation, it should be clear that `def Person.species`, `class << Person; def species`, and `class Person; class << self; def species` aren't three ways to achieve the same thing **by design**, but that they arise out of Ruby's flexibility with regard to specifying what `self` is at any given point in your program.

On the other hand, `class_eval` is slightly different. Because it take a block, rather than act as a keyword, it captures the local variables surrounding it. This can provide powerful DSL capabilities, in addition to controlling the `self` used in a code block. But other than that, they are exactly identical to the other constructs used here.

Finally, `instance_eval` breaks apart the `self` into two parts, while also giving you access to local variables defined outside of it.

In the following table, *defines a new scope* means that code inside the block does **not** have access to local variables outside of the block.

| mechanism | method resolution | method definition | new scope? |
| --- | --- | --- | --- |
| class Person | Person | same | yes |
| class << Person | Person's metaclass | same | yes |
| Person.class_eval | Person | same | no |
| Person.instance_eval | Person | Person's metaclass | no |

Also note that `class_eval` is only available to `Modules` (note that Class inherits from Module) and is an alias for `module_eval`. Additionally, `instance_exec`, which was added to Ruby in 1.8.7, works exactly like `instance_eval`, except that it also allows you to send variables into the block.

**UPDATE:** Thank you to Yugui of the Ruby core team for [correcting the original post](http://yugui.jp/articles/846?ref=yehudakatz.com), which ignored the fact that `self` is broken into two in the case of `instance_eval`.