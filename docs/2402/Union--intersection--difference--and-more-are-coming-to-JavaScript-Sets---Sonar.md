<!--yml
category: 未分类
date: 2024-05-27 15:00:03
-->

# Union, intersection, difference, and more are coming to JavaScript Sets | Sonar

> 来源：[https://www.sonarsource.com/blog/union-intersection-difference-javascript-sets/](https://www.sonarsource.com/blog/union-intersection-difference-javascript-sets/)

The [JavaScript `Set`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Set "JavaScript Set") was introduced to the language in the ES2015 spec, but it has always seemed incomplete. That's about to change.

Sets are collections of values where each value may only appear once. In the ES2015 version of the `Set`, the available functionality revolved around creating, adding to, removing from, and checking the membership of a `Set`. If you wanted to operate on or compare more than one set, you had to write your own functions. Thankfully, [TC39](https://tc39.es/ "TC39")—the committee established to work on the ECMAScript spec—and the browsers have been working on this. We are now seeing functions like `union`, `intersection` and `difference` in JavaScript implementations.

Before we look at the new functionality, let's recap what JavaScript Sets can do now and then we'll jump into the [new Set functions](https://www.sonarsource.com/blog/union-intersection-difference-javascript-sets/#what-are-the-new-set-functions "new Set functions") and the [JavaScript engines that support them](https://www.sonarsource.com/blog/union-intersection-difference-javascript-sets/#support "JavaScript engines that support them") below.

## What do ES2015 JavaScript Sets do?[](/blog/union-intersection-difference-javascript-sets/#what-do-es2015-javascript-sets-do "What do ES2015 JavaScript Sets do?")

Let's take a look at what the ES2015 version of the JavaScript `Set` can do. It's easiest to do so with some examples.

You can [construct a `Set`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Set/Set "construct a Set") without any arguments, which gives you an empty `Set`. Or, you can provide an iterable, like an array, to initialise the `Set`.

```
const languages = new Set(["JavaScript", "TypeScript", "HTML", "JavaScript"]);
```

`Set`s can only contain unique values, so the `Set` above has three members. You can check this with the [`size` property](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Set/size "size property").

```
languages.size;

// => 3
```

You can add more elements to the `Set` with the [`add` function](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Set/add "add function"). Adding an element that is already in the `Set` doesn't do anything.

```
languages.add("JavaScript");

languages.add("CSS");

languages.size;

// => 4
```

You can remove elements from the `Set` with [`delete`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Set/delete "delete").

```
languages.delete("TypeScript");

languages.size;

// => 3
```

You can check if an element is a member of the `Set` with the [`has` function](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Set/has "has function"). One of the benefits of a `Set` is that this check can be done in constant time (**O(1)**), whereas the time to check if an element is in an `Array` varies by the length of the `Array` (**O(n)**). Using `Set`s for tasks like this is a [clean way to write intentional efficient code](https://www.sonarsource.com/blog/what-is-clean-code/#example-4 "clean way to write intentional efficient code").

```
languages.has("JavaScript");

// => true

languages.has("TypeScript");

// => false
```

You can [loop through a `Set`'s elements using `forEach`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Set/forEach "loop through a Set's elements using forEach") or a `for...of` loop. Elements are sorted in the order they were added to the `Set`.

```
languages.forEach(element => console.log(element));

// "JavaScript"

// "HTML"

// "CSS"
```

You can also get an iterator from the `Set` using the [`keys`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Set/keys "keys") and [`values`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Set/values "values") functions ([which are actually equivalent](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Set/keys#using_keys "which are actually equivalent")) as well as the [`entries`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Set/entries "entries") function.

Finally, you can empty a Set with the [`clear`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Set/clear "clear") function.

```
languages.clear();

languages.size;

// => 0
```

That's a good reminder of what you can do with the ES2015 spec version of a `Set`: 

*   `Set`s provide methods to deal with a collection of unique values
*   It is efficient to add elements to a `Set` and to test for their presence in the `Set`
*   Converting an `Array` or other iterable to a `Set` is an easy way to filter out duplicates

This implementation misses out on operations between `Set`s, though. You might want to create a `Set` that contains all the items from two other `Set`s (a union of two `Set`s), find out what two `Set`s have in common (intersection), or find out what isn't present in one Set that is in another (difference). Until recently, you would have had to provide your own functions.

## What are the new Set functions?[](/blog/union-intersection-difference-javascript-sets/#what-are-the-new-set-functions "What are the new Set functions?")

The `Set` methods proposal adds the following methods to `Set` instances: `union`, `intersection`, `difference`, `symmetricDifference`, `isSubsetOf`, `isSupersetOf`, and `isDisjointFrom`.

Some of these methods are akin to some SQL joins, which we will use to illustrate the results alongside the code. Let's see some examples of what each function does.

You can try out any of the code examples below in Chrome 122+ or Safari 17+.

### Set.prototype.union(other)[](/blog/union-intersection-difference-javascript-sets/#setprototypeunionother "Set.prototype.union(other)")

A union of sets is a set that contains all the elements present in either set.

```
const frontEndLanguages = new Set(["JavaScript", "HTML", "CSS"]);

const backEndLanguages = new Set(["Python", "Java", "JavaScript"]);

const allLanguages = frontEndLanguages.union(backEndLanguages);

// => Set {"JavaScript", "HTML", "CSS", "Python", "Java"}
```

In this example, all the languages from the first two sets are in the third set. As with other methods that add elements to the `Set`, duplicates are removed.

This is the equivalent of a SQL `FULL OUTER JOIN` between two tables.

### Set.prototype.intersection(other)[](/blog/union-intersection-difference-javascript-sets/#setprototypeintersectionother "Set.prototype.intersection(other)")

An intersection is a set that contains all the elements that are present within both sets. 

```
const frontEndLanguages = new Set(["JavaScript", "HTML", "CSS"]);

const backEndLanguages = new Set(["Python", "Java", "JavaScript"]);

const frontAndBackEnd = frontEndLanguages.intersection(backEndLanguages);

// => Set {"JavaScript"} 
```

"JavaScript" is the only element present in both the sets here.

An intersection is like an `INNER JOIN`.

### Set.prototype.difference(other)[](/blog/union-intersection-difference-javascript-sets/#setprototypedifferenceother "Set.prototype.difference(other)")

The difference between the set you are working with and another set is all the elements present in the first set and not present in the second set.

```
const frontEndLanguages = new Set(["JavaScript", "HTML", "CSS"]);

const backEndLanguages = new Set(["Python", "Java", "JavaScript"]);

const onlyFrontEnd = frontEndLanguages.difference(backEndLanguages);

// => Set {"HTML", "CSS"} 

const onlyBackEnd = backEndLanguages.difference(frontEndLanguages);

// => Set {"Python", "Java"}
```

In finding the difference between sets, it matters which set you call the function on and which is the argument. In the example above, removing the back-end languages from the front-end languages results in "JavaScript" being removed and returning "HTML" and "CSS" in the resultant set. Whereas removing the front-end languages from the back-end languages still results in "JavaScript" being removed, and returns "Python" and "Java".

A difference is like performing a `LEFT JOIN`.

### Set.prototype.symmetricDifference(other)[](/blog/union-intersection-difference-javascript-sets/#setprototypesymmetricdifferenceother "Set.prototype.symmetricDifference(other)")

The symmetric difference between two sets is a set that contains all the elements that are in one of the two sets, but not both.

```
const frontEndLanguages = new Set(["JavaScript", "HTML", "CSS"]);

const backEndLanguages = new Set(["Python", "Java", "JavaScript"]);

const onlyFrontEnd = frontEndLanguages.symmetricDifference(backEndLanguages);

// => Set {"HTML", "CSS", "Python", "Java"} 

const onlyBackEnd = backEndLanguages.symmetricDifference(frontEndLanguages);

// => Set {"Python", "Java", "HTML", "CSS"}
```

In this case, the elements in the resultant sets are the same, but note that the order is different. Set order is determined by the order the elements are added to the set and the set on which the function is performed will have its elements added first.

A symmetric difference is like a FULL OUTER JOIN excluding any elements that are in both tables.

### Set.prototype.isSubsetOf(other)[](/blog/union-intersection-difference-javascript-sets/#setprototypeissubsetofother "Set.prototype.isSubsetOf(other)")

A set is a subset of another set if all the elements in the first set appear in the second set.

```
const frontEndLanguages = new Set(["JavaScript", "HTML", "CSS"]);

const declarativeLanguages = new Set(["HTML", "CSS"]);

declarativeLanguages.isSubsetOf(frontEndLanguages);

// => true

frontEndLanguages.isSubsetOf(declarativeLanguages);

// => false
```

A set is also a subset of itself.

```
frontEndLanguages.isSubsetOf(frontEndLanguages);

// => true
```

### Set.prototype.isSupersetOf(other)[](/blog/union-intersection-difference-javascript-sets/#setprototypeissupersetofother "Set.prototype.isSupersetOf(other)")

A set is a superset of another set if all the elements in the second set appear in the first set. It is the opposite relationship of being a subset.

```
const frontEndLanguages = new Set(["JavaScript", "HTML", "CSS"]);

const declarativeLanguages = new Set(["HTML", "CSS"]);

declarativeLanguages.isSupersetOf(frontEndLanguages);

// => false

frontEndLanguages.isSupersetOf(declarativeLanguages);

// => true
```

A set is also a superset of itself.

```
frontEndLanguages.isSupersetOf(frontEndLanguages);

// => true
```

### Set.prototype.isDisjointFrom(other)[](/blog/union-intersection-difference-javascript-sets/#setprototypeisdisjointfromother "Set.prototype.isDisjointFrom(other)")

Finally, a set is disjoint from another set if they have no elements in common.

```
const frontEndLanguages = new Set(["JavaScript", "HTML", "CSS"]);

const interpretedLanguages = new Set(["JavaScript", "Ruby", "Python"]);

const compiledLanguages = new Set(["Java", "C++", "TypeScript"]);

interpretedLanguages.isDisjointFrom(compiledLanguages);

// => true

frontEndLanguages.isDisjointFrom(interpretedLanguages);

// => false
```

The interpreted languages and compiled languages in these sets do not overlap, so the sets are disjoint. The front-end languages and the interpreted languages do overlap with the element "JavaScript", so they are not disjoint.

## Support[](/blog/union-intersection-difference-javascript-sets/#support "Support")

As of writing this, [the proposal](https://github.com/tc39/proposal-set-methods "the proposal") stands at stage 3 in [TC39's process](https://tc39.es/process-document/ "TC39's process") and Safari 17 (released in September 2023) and Chrome 122 (February 2024) have shipped implementations of these methods. Edge follows Chrome closely and Firefox Nightly has support behind a flag, I would expect both of these browsers to ship support soon too.

[Bun](https://bun.sh/ "Bun") also uses Safari's [JavaScriptCore engine](https://docs.webkit.org/Deep%20Dive/JSC/JavaScriptCore.html "JavaScriptCore engine") and thus already supports these new functions. Support in Chrome means that it has been added to the [V8 JavaScript engine](https://v8.dev/ "V8 JavaScript engine") and will be adopted by [Node.js](https://nodejs.org/ "Node.js") soon.

Hopefully, this means the proposal will graduate to stage 4 of the process, perhaps even in time to join the ES2024 spec before it is finalised.

### Polyfills[](/blog/union-intersection-difference-javascript-sets/#polyfills "Polyfills")

While you need older JavaScript engine support, there are polyfills you can use to upgrade to spec-compliant implementations of these functions. They are available in [core-js](https://github.com/zloirock/core-js#new-set-methods "core-js") or as individual packages per function in the [es-shims project](https://github.com/es-shims "es-shims project") (for example, the [set.prototype.union package](https://www.npmjs.com/package/set.prototype.union "set.prototype.union package") can be used for union functionality).

If you've written your own implementation of any of these functions, I would recommend first upgrading to the polyfills, before phasing them out as the support becomes more widespread.

## Sets no longer feel incomplete[](/blog/union-intersection-difference-javascript-sets/#sets-no-longer-feel-incomplete "Sets no longer feel incomplete")

The JavaScript `Set` has long been incomplete but these 7 new functions round out the implementation nicely. Building functionality like this into the language means we have to rely less on dependencies or our own implementations and can [focus](https://www.sonarsource.com/blog/what-is-clean-code/#intentional "focus") on the problems we are trying to solve.

This is just one of a [full pipeline of stage 3 proposals](https://github.com/tc39/proposals?tab=readme-ov-file "full pipeline of stage 3 proposals") before TC39 right now. Check out the list to see what else might be coming to JavaScript next. I've got my eye on [Temporal](https://github.com/tc39/proposal-temporal "Temporal") and [Decorators](https://github.com/tc39/proposal-decorators "Decorators"), both of which could change the way we write major parts of our JavaScript.