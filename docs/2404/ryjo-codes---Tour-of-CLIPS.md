<!--yml
category: 未分类
date: 2024-05-27 13:40:28
-->

# ryjo.codes - Tour of CLIPS

> 来源：[https://ryjo.codes/tour-of-clips.html](https://ryjo.codes/tour-of-clips.html)

### 0\. Introduction

Over the last year and a half or so, I've been learning about the [CLIPS](https://www.clipsrules.net/) programming language. [A team of developers at NASA](https://clipsrules.net/AboutCLIPS.html) created CLIPS in the mid-'80s. It was released into the public domain in the mid-'90s, and it has been actively developed by one of the original team members [Gary Riley](http://www.clipsrules.net/AboutTheDeveloper.html) ever since.

CLIPS has this air of mystery about it. There's no npm-like service that centralizes common CLIPS "libraries." A lot of writing about the language appears, on the surface, academic or higher-level. To top it off, the [Rete algorithm](https://en.wikipedia.org/wiki/Rete_algorithm) central to CLIPS can be rather intimidating to wrap one's head around.

Don't let these things deter you; CLIPS is well worth learning. Its Rules-based approach is a good primer to learning neural networks and AI. With this tutorial, you'll take your first steps into Rules Engines and Expert Systems.

First, we'll talk about why you should consider using CLIPS. At the heart of CLIPS is the [Rete algorithm](https://en.wikipedia.org/wiki/Rete_algorithm). This algorithm does a few very nice things for us. For one, it determines the order in which Rules should run. This allows us to focus on **what the domain business logic is** of our program rather than **how the software runs**. Second, the Rete network "caches" common calculations our software will make. Those who have written an in-app cache understand the benefits of this feature.

Another huge win for CLIPS is the excellent documentation and support provided by the lead developer [Gary Riley](http://www.clipsrules.net/AboutTheDeveloper.html). Take a look at the various guides of all levels linked in the [documentation section](https://clipsrules.net/#documentation) of the CLIPS homepage. Also, Mr. Riley is very active in various forums providing support for CLIPS. Once again, look at the main CLIPS website to find links to the [various avenues of support](https://clipsrules.net/#support).

Finally, CLIPS is still actively developed; its latest release 6.40 was published in May of 2021\. Pretty incredible!

#### 0.a This Tutorial

This tutorial is greatly inspired by the fantastic [A Tour of Go](https://go.dev/tour/welcome/1). Run the examples presented in this tutorial by clicking the `(batch*)` button under the code. The code in the box is editable, so tweak the code to test your curiosity. The output from the code will display in the box below the buttons. Navigate between the previous and next Chapters by clicking the links at the bottom of the screen. The left link (←) will navigate you to the previous chapter while the right link (→) will advance to the next chapter. Between these two links will show the name of the current Chapter. Click that link to reveal a list of all Chapters in this book. Clicking one of these Chapters should navigate you to the appropriate Chapter.

```
; Welcome to the tutorial! This is known as a "comment."
; It is meant to help developers
; understand a particular line of code better.
;
; Click the (batch*) button to run this code sample.
; You will see CLIPS code written in this area of the webpage
; throughout this tour.
;
; This is a WASM compilation of CLIPS run completely in your browser;
; no server involved!
(println "Hello, world")
```

### 1\. Saying Hello

The classic "first program" is the "Hello, World" program. This will show you how to output some text from your program. Click the `(batch*)` button under the code on the right hand side. You should see text appear in the box beneath the buttons.

With CLIPS, there are a few ways we can print text to the screen. Using `(print)` will print words out, while `(println)` will print words out **with a new line at the end**. In the code sample to the right, you can see us using `(print)` to print the word "Hello". We then use `(println)` to print ", World!" and a new line character. Things printed after this text will appear on the next line.

`(printout t)` is like `(print)` and `(println)`, but it allows you to send text to a given "router." In this case, we send the text to the router `t` which is what `(print)` and `(println)` send text to by default. We also use the text `crlf` as the last argument to `(printout`. We do this so that our text "breaks" at the end of the line; the next thing we print will appear beneath this current line. `crlf` stands for "carriage return line feed," which means "move the cursor to the beginning of the next line."

`(format t)` allows you to print a sentence structure that contains "format flags." The `%s` and `%n` are format flags that specify a string and newline respectively. The next argument (in this case "fun") will be substituted in place of the `%s`. `%n` will be replaced with a new line character. To see more of these "format flags," check out the [Basic Programming Guide](https://clipsrules.sourceforge.io/documentation/v640/bpg.pdf) -- specifically the section entitled **12.4.6 Formatted Printing** -- to learn more.

```
(print Hello)
(println ", World!")
(printout t "Welcome to this tour of CLIPS" crlf)
(format t "I hope that you have %s!%n" fun)
```

### 2\. Learning the Rules

Rules are an important part of understanding how CLIPS works. Rules consist of an "antecedent" and a "consequent." The antecedent is also known as the "if portion" or the "left-hand side." This part consists of the conditions that must be satisfied for the Rule to run.

The consequent is also known as the "then portion" or the "right-hand side." This part describes what will happen when the Rule is run.

These two pieces are separated by a `=>`. The antecedent is before this symbol while the consequent is after it.

In this example, we define a Rule called `will-run-no-matter-what`. This particular Rule has nothing on the left-hand side so it will always run. The right-hand side specifies that a `(println)` will run.

Finally after the Rule is defined, there is a `(run)`. In order for this to work, click the `(batch*)` button.

```
(defrule will-run-no-matter-what
      =>
      (println "Hello from a rule, world!"))
(run)
```

### 3\. Just the Facts

In addition to Rules in CLIPS, we have "Facts." A Fact represents "data" in CLIPS. In order to put data into CLIPS, we `(assert)`. Once we have `(assert)`ed Facts, they are said to be "in working memory."

In the code to the right, we `(assert)` one `(name` Fact with `ryjo` in its second field.

We then print a single line of text with our old friend `(println)`.

`(facts)` is a new one. We use this to print out the Facts stored in Working Memory. In this case, we see 1 fact printed with an index `1` as denoted by `f-1`.

Finally, we'll `(retract` this Fact that we just asserted. We can reference this Fact with its index `1`.

Note: If you click `(batch*)` multiple times in a row, an error will appear. This is because we must `(reset)` or `(clear)` the environment to reset the ids assigned to Facts in Working Memory. If we do not, the Fact that we assert will have an id of `2`, not `1`. If you click `(reset)` or `(clear)` and then click `(batch*)` again, the code will behave as expected.

**Bonus**: if you are returning to this Chapter after completing [Chapter 5](#5), you may see a few more than 1 Fact returned. That's because we do not `(clear)` at the top of the code to first "clean up" the environment. You'll learn more about `(clear)` in [Chapter 18](#18).

```
(assert (name ryjo))
(println "The current facts: ")
(facts)
(retract 1)
(println "After retracting: ")
(facts)
```

### 4\. Running the Engine

The "Facts" stored in "Working Memory" can be used to satisfy the LHS (left-hand side) of a rule. Remember: when the LHS of a Rule is satisfied, the Rule's RHS (right-hand side) will run.

In the code to the right, we create a Rule using `(defrule)` called `greet`. This Rule will "activate" when a Fact that matches the form `(name ?)` is entered into Working Memory. Once we `(run)` the rules engine, the Right Hand side of this Rule will run if a `name` Fact exists in the system with a second field.

Next, we `(assert)` a `name` Fact with the value `ryjo` in its second field. Once we do this, the `greet` rule is "activated;" when we `(run)` our Rules Engine next, this Rule's RHS will be executed.

**Bonus**: if you are returning to this Chapter after completing [Chapter 5](#5), you may see a few more than 1 greeting. That's because we do not `(clear)` at the top of the code to first "clean up" the environment. You'll learn more about `(clear)` in [Chapter 18](#18).

```
(defrule greet
      (name ?name)
      =>
      (println "Hello " ?name "!"))
(assert (name ryjo))
(run)
```

### 5\. Negative, Ghostrider

Sometimes you want to activate a Rule when there is a Fact in Working Memory that does not match a certain pattern. The tilde (`~`) character is used to negate a given pattern. It's like saying "a Fact that does not have this here."

Note that we are not able to use the non-matching names in the RHS of this Rule like we did in [Chapter 4](#4). We'll cover how to use these names in [Chapter 6](#6).

First, we `(clear)` our environment of all previously defined Rules and Facts from the CLIPS Environment. We then `(assert` two `(name`s: `Zach` and `Zethus`. Finally, our Rule `non-ryjos-only` will "activate" when a Fact matching the pattern `(name ~ryjo)` enters working memory. This pattern might be read in English as "a Fact that starts with `name` and does not have `ryjo` in its second field." Finally, the RHS of this Rule prints the text "Hello, not ryjo!"

**Bonus:** After you `(batch*)` this code, try navigating to [Chapter 4](#4); you'll see that the previously defined `greet` Rule runs for these two new names. That's because the examples in this tutorial use the same CLIPS Environment. In this example, we use `(clear)` at the top which removes everything defined in previous examples. You will see this convention more in following chapters.

```
(clear)

(assert (name Zach) (name Zethus))

(defrule non-ryjos-only
  (name ~ryjo)
  =>
  (println "Hello, not ryjo!"))

(run)
```

### 6\. Capturing the Name

Now we'll combine two previously presented concepts using a `&`. First, we'll capture the value in the first field with `?n` ([Chapter 4](#4)). Next, we'll specify "not ryjo" with `~` ([Chapter 5](#5)). The `&` symbol acts as an "and." We can read the pattern `(name ?n&~ryjo)` as "a value and it is not ryjo."

Additionally, we `(assert)` three facts. Only two of them will "activate" the `non-ryjos-only` rule we define right below it. Something neat to make note of: we only need to call `(assert)` once in order to put three new Facts into Working Memory.

We define a second rule `ryjo` here to demonstrate the "opposite" of the `non-ryjos-only` rule. This will activate when a Fact enters Working Memory that looks exactly like `(name ryjo)`.

Note how the Rules in this example don't execute in the order in which they were written. The reason for this is **the order in which Rules are run is determined algorithmically**. Read more about how CLIPS determines the order in which Rules are run in section **5.2 Basic Cycle Of Rule Execution** of the [Basic Programming Guide](https://clipsrules.sourceforge.io/documentation/v640/bpg.pdf) . We can use `(salience` to explicitly declare the order in which Rules run. We'll discuss this in [Chapter 15](#15).

```
(clear)

(assert
  (name Gabe)
  (name ryjo)
  (name John))

(defrule non-ryjos-only
  (name ?n&~ryjo)
  =>
  (println "Hello, " ?n "!"))

(defrule ryjo
  (name ryjo)
  =>
  (println "Oh hey there"))

(run)
```

### 7\. And/Or

We can make use of "Connective Constraints" to better refine the LHS of our Rules. For example, in the Rule `ors` in the code editor, we match on a Fact whose second field matches either `Gabe` or `Peter`. This can be read as "A `name` Fact whose first field we store the variable `?name` and matches either `Gabe` or `Peter`."

In the `and-nots` Rule, we store the second field of a `name` Fact in the variable `?name`. This time, we say "the value in the first field cannot match `ryjo` and it cannot match `Zach` and it cannot match `Zethus`."

```
(clear)

(assert
  (name Gabe)
  (name ryjo)
  (name John)
  (name Zach)
  (name Zethus)
  (name Peter))

(defrule ors
  (name ?name&Gabe|Peter)
  =>
  (println "Greetings, " ?name))

(defrule and-nots
  (name ?name&~ryjo&~Zach&~Zethus)
  =>
  (println "Hi there, " ?name))

(run)
```

### 8\. Predicate Constraints

Predicate Constraints sound fancy, but they're pretty straight forward. Basically they allow you to further specify what value a field holds. For example, with our current knowledge, we can specify *exact* matches for values such as `(name ?n&ryjo)`. Predicate Conditions allow us to perform a function on the value to determine if it matches.

In our code example for this chapter, we write a basic Rule that describes a rule that allows entry into a bar. In the United States of America, we require a person to be 21 years of age or older. If they are under 21 years of age, they are not allowed into a bar.

Note the familiar form of `?variable&`. We use the `and` symbol here (`&`) to signify that `?variable` must match certain criteria. This allows us to read this as "the age of a person and it is greater than or equal to 21" in the `allow-entry-to-bar` Rule.

Also note that we make use of 2 different Facts in Working Memory to activate our rules: the `name` and `age` Facts. Also note that in the LHS of our Rules, we specify that the first field of these Facts must match by using `?person` in both. Since `?person` is used in both of these conditions, we say that this Rule is "activated" when these Facts have the same value in their first field.

For example, the `(name ryjo)` Fact and the `(age ryjo 32)` Fact will activate the `allow-entry-to-bar` Rule together. Both the value in the first field match for these Facts *and* the number 32 is greater than or equal to 21\. The Facts `(name ryjo)` and `(age someone 20)` will not activate the `no-entry-to-bar` Rule. The number `20` is indeed less than 21, but these two Facts will not activate the Rule together. However, the 2 Facts `(name someone)` and `(age someone 20)` will. Not only is `20` less than `21` as noted in the Predicate Condition `(>= ?a 21)`, but the Symbol `someone` matches in both Facts.

```
(clear)

(defrule allow-entry-to-bar
  (name ?person)
  (age ?person ?a&:(>= ?a 21))
  =>
  (println ?person " is over 21 years old and can enter the bar!"))

(defrule no-entry-to-bar
  (name ?person)
  (age ?person ?a&:(< ?a 21))
  =>
  (println ?person " is not allowed in (under 21)."))

(assert
  (name ryjo) (age ryjo 32)
  (name someone) (age someone 20))

(run)
```

### 9\. Test Conditional

This example looks like the previous section. Instead of using `&:`, we define our constraints using the `(test` Conditional Element.

The output for this example should look very similar to last example's.

```
(clear)

(defrule allow-entry-to-bar
  (name ?person)
  (age ?person ?a)
  (test (>= ?a 21))
  =>
  (println ?person " is over 21 years old and can enter the bar!"))

(defrule no-entry-to-bar
  (name ?person)
  (age ?person ?a)
  (test (< ?a 21))
  =>
  (println ?person " is not allowed in (under 21)."))

(assert
  (name ryjo) (age ryjo 32)
  (name someone) (age someone 20))

(run)
```

### 10\. One forall

Sometimes you want to know if all Facts in Working Memory match a given pattern. `(forall` provides this check. In our example, `(forall` will match if every `(name` Fact in working memory can be matched with an `(age` Fact (and vice versa).

In our example, the Fact `(name another)` does not have an `(age` Fact with a matching value `?n`. Therefore, our Engine tells us that the "Roster has been declined."

```
(clear)

(assert
  (name ryjo) (age ryjo 32)
  (name someone) (age someone 20)
  (name another))

(defrule accept-roster
  (forall (name ?n) (age ?n ?))
  =>
  (println "Roster has been accepted"))

(defrule decline-roster
  (not (forall (name ?n) (age ?n ?)))
  =>
  (println "Roster has been declined"))

(run)
```

### 11\. Aliens exists

This is a bit of a longer example, but it showcases a few things combined together. `(exists` will match once if a Fact in Working Memory matches the pattern within. It will not run again while the condition continues to match. In our example, we match for "any fact that starts with `(age` and is followed with two values." When we use `?`, it doesn't matter what these values are; it only matters that they are there at all.

Try modifying our `any-age-reported` Rule to capture the `?name` and `(println ?name)` it in the RHS of the Rule.

We also use `(forall` to determine when every user has reported their age.

It may seem strange to give up control of our code's order of execution. Remember, this lets us focus on our Business Logic rather than Control Flow of our domain. Make sure to review the section **Performance Considerations** in the [Basic Programming Guide](https://clipsrules.sourceforge.io/documentation/v640/bpg.pdf) to learn how to write code in harmony with this approach.

```
(clear)

(defrule any-age-reported
  (exists (age ? ?))
  =>
  (println "Users have begun to report their age"))

(defrule no-ages-reported
  (not (age ? ?))
  =>
  (println "No user has reported their age yet..."))

(defrule reported-age
  (name ?n)
  (age ?n ?a)
  =>
  (format t "%s has reported their age: %d%n" ?n ?a))

(defrule has-not-reported-age
  (name ?n)
  (not (age ?n ?))
  =>
  (println ?n " has not yet reported their age"))

(defrule all-ages-reported
  (forall (name ?n) (age ?n ?))
  =>
  (println "Everyone in the system has reported their age"))

(assert
  (name ryjo)
  (name someone)
  (name another))

(println "First run results:")
(println "------------------")
(run)
(println " ")

(assert (age ryjo 32))

(println "Second run results:")
(println "-------------------")
(run)
(println " ")

(assert (age someone 20)
        (age another 52))

(println "Third run results:")
(println "------------------")
(run)
```

### 12\. Do you read(line) me?

Sometime you want to take input from the user rather than "hard-code" their name, age, and other details. We can take input from the user using `(readline)`.

In this example, we take input from the user, then we tell them what they entered. Note: when you run this example, a prompt will pop up on your screen. Enter your text into the field, then click the "OK" button.

Similarly, `(read)` will take the input from a user, but it'll only take a single word. Change `(readline)` to `(read)` in the code editor. If you enter "foo bar 123" into the text box and then click the "OK" button, it'll only echo out "foo."

```
(clear)

; replace (readline) with (read) and note the differences
(println "You entered: " (readline))

(run)
```

### 13\. Say the secret word

**Be warned:** note that there is a line commented out in this example. If you uncomment this line, the pop-up prompt will continue to show until you enter the word `secret`.

The reason that this loop occurs is because we `(retract` the `(word` Fact which we captured as `?f` in the LHS of the `user-did-not-say-secret` Rule. This triggers the pattern `(not (word ?))` in the `get-word-from-user` which is the absence of a `(word` Fact with one field in Working Memory.

This example is a little more complex, but it provides a good example of a way in which you can "validate" user input. This is not the *only* way, mind you. It is, however, a good example of how the "main loop" of a program can be made using only Rules and Facts.

```
(clear)

(defrule get-word-from-user
  ;uncommenting this line will cause a prompt to popup
  ;until you enter "secret" in the textbox ;)
  ;(not (word ?))
  =>
  (print "Enter your word: ")
  (bind ?word (read))
  (println ?word)
  (assert (word ?word)))

(defrule user-said-secret
  (word secret)
  =>
  (println "You said the secret word! Great job"))

(defrule user-did-not-say-secret
  ?f 
  (retract ?f)
  (println "Try again..."))

(run)
```

### 14\. So many words

The `(readline)` function will capture user input and return a string. If the user's input has any spaces between words, we can use `(explode$` to break the input up into a "multifield value." This lets us define patterns in our Rules that will match on individual words entered by the user.

Take a look at the Rule `first-word-one`. This Rule will activate when the first word entered by the user is "**one**." Similarly, the Rule `last-word-last` will activate when the last word entered is "**last**."

The `anywhere` Rule will trigger if "**anywhere**" is entered anywhere in the prompt. The `$?` will match for any number of words. For example, we see in the `counter` Rule the pattern `(exploded-words $?words)`. This means "match any number of values and reference them as a "multifield value" `?words`. Just like `?`, `$?` allows us to say "match on any number of fields" without referencing it in the RHS.

Speaking of the `counter` Rule: take a look at `(length$ ?words)`. As you may have guessed, `(length$` will return the number of values in the multifield value passed to it. In this case, it returns the number of words (as referenced by `?words`) entered by the user in the prompt.

```
(clear)

(defrule get-words-from-user
  =>
  (assert (words (readline))))

(defrule catcher
  (words ?words)
  =>
  (println "You said: " ?words)
  (assert (exploded-words (explode$ ?words))))

(defrule counter
  (exploded-words $?words)
  =>
  (println "That's " (length$ ?words) " words total."))

(defrule silence
  (exploded-words)
  =>
  (println "Ah, the strong silent type..."))

(defrule first-word-one
  (exploded-words one $?words)
  =>
  (println "Your first word was 'one'"))

(defrule last-word-last
  (exploded-words $?words last)
  =>
  (println "Your last word was 'last'"))

(defrule anywhere
  (exploded-words $? anywhere $?)
  =>
  (println "At some point, you typed 'anywhere'"))

(run)
```

### 15\. Prominently salient

By using `(declare (salience` in our Rule, we can specify the "priority" of a Rule. This allows us to declare that **some Rules must always run before others**. We "hard-code" the execution order for our rules, and the Rete algorithm builds around it.

Rules have a salience of `0` by default. Rules with a salience greater than `0` will execute before them. Similarly, Rules with a lower salience will execute after them. According to the [Basic Programming Guide](https://clipsrules.sourceforge.io/documentation/v640/bpg.pdf), this feature should be used sparingly:

> Despite the large number of possible values, with good design there's rarely a need for more than five salience values in a simple program and ten salience values in a complex program.

For the most part, a Rules Engine should be order independent. This allows CLIPS to do the "heavy lifting" of determining when Rules run. This is subtle, but rather powerful. With Rules, you do not "pass" values into a "function" that runs and returns a value. Instead, the algorithm "activates" Rules based on "matched" Facts in Working Memory. This decouples the data stored in your system (Facts) from the business logic (Rules).

This decoupling can help express the domain you're modeling with less references to the "control flow" of the software. The more transparent the software, the closer your system brings the user to the "truth" of your domain.

```
(clear)

(defrule third
  (declare (salience 0))
  ?f 
  (println "Last!")
  (retract ?f))

(defrule never-runs
  (declare (salience -1))
  (my-fact)
  =>
  (println ":("))

(defrule first
  (declare (salience 2))
  (my-fact)
  =>
  (println "First!"))

(defrule second
  (declare (salience 1))
  (my-fact)
  =>
  (println "Second!"))

(assert (my-fact))

(run)
```

### 16\. What's your function?

Functions allow you to reference chunks of code that you write repeatedly throughout your program. If we were writing a book, we may find it easier to "wrap" text that has been spoken in quotes as well as a reference to the speaker. We do the same for shouting.

We can also store difficult-to-remember formulas as Functions. Personally, I have trouble remembering the formula for converting fahrenheit to celsius. With the help of `deffunction`, I can store this calculation and reference it easily.

Take note of the `(*`, `(-` and `(/` functions. These are basic math operations: multiplication, subtraction and division. In CLIPS, we write the math problem `1 + 2` as `(+ 1 2)`. That's because CLIPS uses a "LISP-like" syntax.

```
(clear)

(deffunction say (?words)
  (println "You say, \"" ?words ".\""))

(say "Hello, world")

(deffunction shout (?words)
  (println "\"" ?words "!\" you shout."))

(shout "Hello, world")'

(deffunction fahrenheit-to-celsius (?f)
  (* (- ?f 32) (/ 5 9) ))

(println "50°F is " (fahrenheit-to-celsius 50) "°C")
```

### 17\. Templatize me

Templates are used to define a formal structure for Facts. By using `(deftemplate person` in our example, we describe the structure of a `(person` Fact. We wrote Facts as lists of values in previous Chapters. We would describe these Facts as having ordered "fields." `(person Sally "Hi")` would describe a `(person` Fact with the values `Sally` in its second field and `"Hi"` in its third field.

With Templates, we are no longer bound to storing people's information in a particular order. Instead, we store values in named "Slots." For example, we can choose to define our Fact as `(person (greeting "Hi") (name Sally))`, greeting first. We can also define "default" values for these named slots and omit them in asserted Facts.

Using Templates gives us some flexibility in how we define our Facts as well as how we write the matching patterns in the LHS of our Rules. Take a look at the Rule `people-greet-each-other`. Our second pattern matches "a person whos name is not `?name1`." We do not need to include a reference to this Fact's `(greeting` Slot in this Rule, so we can omit it.

You'll also notice we define a default value for `(greeting` in the Template's definition. There are many other ways to describe a Slot besides `default`. Take a look at the [Basic Programming Guide](https://clipsrules.sourceforge.io/documentation/v640/bpg.pdf) in the section titled "Deftemplate Construct" to learn what these are.

```
(clear)

(deftemplate person
  (slot name)
  (slot greeting (default Hello)))

(assert
  (person (name Greg))
  (person (name Sally) (greeting Hi))
  (person (name Sam) (greeting Howdy)))

(deffunction greet (?name1 ?name2 ?greeting)
  (println ?name1 " says \"" ?greeting "\" to " ?name2 "."))

(defrule people-greet-each-other
  (person (name ?name1) (greeting ?greeting1))
  (person (name ?name2&~?name1))
  =>
  (greet ?name1 ?name2 ?greeting1))

(run)
```

### 18\. Clearly resetting

Let's talk about `(clear)` and `(reset)`. We're familiar with `(clear)` which makes our environment like new again. No more Rules, no more Facts.

`(reset)` does a little less. It retracts all Facts in your system and Rule activations while leaving the Rules themselves defined. In this way, you can use `(reset)` to return your Engine to a known default "state" before running it again with new input.

What happens if we need some Facts asserted to return to the "default" state? Introducing `(deffacts`. When you use `(reset)`, the Facts defined in your `(deffacts` will be asserted. Consider the example in this chapter and its output. On the first run, we `(assert` 2 more fruit Facts than in the second. As a result, we see two more people eat their favorite fruit.

```
(clear)

(deffacts fruits
  (fruit apple 1)
  (fruit banana 1)
  (fruit orange 1))

(deftemplate person
  (slot name)
  (slot favorite-fruit))

(deffacts people
  (person (name Tom) (favorite-fruit apple))
  (person (name Jenna) (favorite-fruit orange))
  (person (name Rian) (favorite-fruit apple)))

(defrule people-eat-favorite-fruits
  (person (name ?name) (favorite-fruit ?fruit))
  ?f 
  (println ?name " eats " ?fruit)
  (retract ?f))

(defrule persons-favorite-fruit-missing
  (person (name ?name) (favorite-fruit ?fruit))
  (not (fruit ?fruit ?))
  =>
  (format t "There's no more of %s's favorite fruit %s!%n" ?name ?fruit))

(reset)

(assert (fruit apple 2) (fruit orange 2))

(println "Run 1:")
(println "------")
(run)

(println " ")
(reset)

(println "Run 2:")
(println "------")
(run)
```

### 19\. Who watches the statistics?

```
(clear)
(watch statistics)

(defrule fibonacci
  ?fact  ?i 1))
  =>
  (print ?fib " ")
  (retract ?fact)
  (assert (fib (+ ?fib ?lastfib) ?fib (- ?i 1))))

(defrule last-fibonacci-number
  ?fact 
  (println ?fib)
  (retract ?fact))

(assert (fib 1 0 10))

(run)
(unwatch statistics)
```

### 20\. Rules: Activate!

```
(clear)
(watch activations)

(defrule b-exists-and-a-less-than-1
   (b ?b)
   (a ?a&:(< ?a 1))
   =>
   (println "there is b = " ?b ", AND a " ?a " is less than 1"))

(defrule a-less-than-1
   (a ?a&:(< ?a 1))
   =>
   (println "a " ?a " is less than 1"))

(println "  Step 1  ")
(println "==========")
(println "--- assert")
(assert (a 2))
(println "------ run")
(run)
(println " ")

(println "  Step 2  ")
(println "==========")
(println "--- assert")
(assert (a 0))
(println "------ run")
(run)
(println " ")

(println "  Step 3  ")
(println "==========")
(println "--- assert")
(assert (b foo) (a -1) (a 3))
(println "------ run")
(run)
(unwatch activations)
```

### 21\. Watching functions

```
(clear)
(watch deffunctions)

(deffunction less-than-1 (?a)
   (println "calculating whether " ?a " is less than 1...")
   (< ?a 1))

(defrule b-exists-and-a-less-than-1
   (b ?b)
   (a ?a&:(less-than-1 ?a))
   =>
   (println "there is b = " ?b ", AND a " ?a " is less than 1"))

(defrule a-less-than-1
   (a ?a&:(less-than-1 ?a))
   =>
   (println "a " ?a " is less than 1"))

(println "  Step 1  ")
(println "==========")
(println "--- assert")
(assert (a 2))
(println "------ run")
(run)
(println " ")

(println "  Step 2  ")
(println "==========")
(println "--- assert")
(assert (a 0))
(println "------ run")
(run)
(println " ")

(println "  Step 3  ")
(println "==========")
(println "--- assert")
(assert (b foo) (a -1) (a 3))
(println "------ run")
(run)
(unwatch deffunctions)
```

### 22\. ???

That's it for now! Thanks for sticking with the tutorial up to this point. I'll be adding to this over time, so make sure to bookmark this page and check back :).

This entire page is written from scratch when possible. No frameworks, just good ol' fashioned procedural Javascript, CSS and HTML. I took a lot of inspiration from the [Tour of Go](https://go.dev/tour/welcome/1). I found it super helpful to not need to install anything when learning Go years back. I used [emscripten](https://emscripten.org/) to compile [CLIPS](https://www.clipsrules.net/) into a WASM binary. I wrote the code editor + syntax highlighter from scratch with code snippets pulled from [Stackoverflow](https://stackoverflow.com/). I'll admit I'm a bit proud of that :).

My goal with this tutorial is to spread the word about CLIPS. I'd like to see it used in more everyday applications because it's a very powerful way of expressing complex business logic with less lines of code.

```
(clear)
(run)
```