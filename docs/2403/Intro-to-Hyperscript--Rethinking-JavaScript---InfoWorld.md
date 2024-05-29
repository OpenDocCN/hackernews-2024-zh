<!--yml
category: 未分类
date: 2024-05-27 15:03:43
-->

# Intro to Hyperscript: Rethinking JavaScript | InfoWorld

> 来源：[https://www.infoworld.com/article/3708109/intro-to-hyperscript-rethinking-javascript.html](https://www.infoworld.com/article/3708109/intro-to-hyperscript-rethinking-javascript.html)

Some of us remember [HyperCard](https://hypercard.org/), an interesting branch in the evolutionary tree of programming languages. If you’re of a certain age, you might even have learned programming with HyperCard. Hyperscript is a newer technology that takes some of the benefits of HyperCard, especially its simplicity and English-like syntax, and applies it to the browser environment. It's a kind of JavaScript replacement that can work standalone or in tandem with [HTMX](https://www.infoworld.com/article/3706951/htmx-dynamic-html-without-the-javascript.html) to simplify common scripting needs on the [JavaScript front end](https://www.infoworld.com/article/3661810/reactive-javascript-the-evolution-of-front-end-architecture.html).

## An example worth a thousand words

Before we get into too much discussion, let’s look at a [Hyperscript example](https://hyperscript.org/#be-async-with-no-extra-code) that communicates the spirit of the thing:

```
 <div _="init fetch https://stuff as json then put result into me">Using fetch() API...</div> 
```

It’s pretty obvious what this code does. The underscore property is the special one the denotes a Hyperscript. In JavaScript, this same functionality might look something like this:

```
 <div id="myDiv" onload="async function() { 
  const response = await fetch('https://stuff', { 
    headers: { Accept: 'application/json', }
  }); 
  const data = await response.json(); 
  myDiv.innerHTML = JSON.stringify(data); 
}">
</div> 
```

In English, it says: "When the div element is loaded, send an async request to 'https://stuff' and put the results inside the div.”  Hopefully you can see already that Hyperscript feels almost like a behavioral extension of HTML. 

## What is Hyperscript?

Hyperscript is a kind of simplified, more English-like JavaScript. As pointed out to me by [Sacha Greif](https://www.infoworld.com/article/3688770/state-of-javascript-a-conversation-with-sacha-greif.html), you can also think about it as a domain-specific language, or DSL. In essence, Hyperscript boils down JavaScript to a syntax devoted explicitly to common and recurring needs in building front-end UIs. It brings a bunch of conventions to bear to make this coding more concise.

Hyperscript is a sibling of [HTMX](https://www.infoworld.com/article/3706951/htmx-dynamic-html-without-the-javascript.html) and springs from the mind of the same developer, Carson Gross. Both projects reflect Gross's devotion to simplicity and his attempts to relentlessly apply it to big, active problem spaces. In HTMX, we see a more capable HTML that could sweep away much of the complexity that has evolved over the last decade, as developers have embraced the front-end paradigm of reactive frameworks + JSON + [REST-like APIs](https://www.infoworld.com/article/3706031/what-is-rest-the-de-facto-web-architecture-standard.html). In Hyperscript, we see an alternative to the apparently never-ending expansion of JavaScript's language sophistication. It's a tantalizing proposition.

## Tackling front-end complexity

There is a definite sense of overwhelm facing front-end developers who code in the trenches. Who wouldn’t want to replace boilerplate JavaScript with an expressive language that was easy to remember? Something you could just type out from memory, without referencing anything, to perform everyday, basic coding.

Let's consider an example. In the following Hyperscript snippet, the canonical [button-click-counter example](https://hyperscript.org/docs/#control-flow) becomes:

```
 <button _="on click increment :x
            if :x <= 3
              put :x into the next <output/>
            else
              put '3 is the max...' into the next <output/>
            end">
Click Me
</button>
<output>--</output> 
```

Here's the same example in React:

```
 import React from "react";

const Counter = () => {
  const [x, setX] = React.useState(0);

  const handleClick = () => {
    setX((prevX) => {
      if (prevX <= 3) {
        return prevX + 1;
      } else {
        return 3;
      }
    });
  };

  return (
    <div>
      <button onClick={handleClick}>Click Me</button>
      <output>{x}</output>
    </div>
  );
};

export default Counter; 
```

Of course, shorter doesn’t necessarily mean simpler. In this case, though, the self-descriptive obviousness of Hyperscript shines in comparison to React. Now, some might say that React is more complex because it's more powerful; it is a language that makes complex things possible. But in this case, we are just looking at the most common approach to JavaScript front-end development (which is React) next to Hyperscript. We're looking at the everyday activities that *can* be simplified—and should be.

Hyperscript is intended to replace JavaScript. Or maybe *elaboration* is a better word. Hyperscript's creator, Caron Gross, notes that it's a “speculative” project. Nonetheless, it is well thought-out, capable, and ambitious. One can definitely picture enterprise-scale applications using Hyperscript.

Probably the biggest barrier for Hyperscript is developers' collective familiarity and attachment to JavaScript. Sure, it can be obfuscating and tricky at times, but it’s *my* obfuscated and tricky language. If Hyperscript does catch on, it’ll probably be in some combination with JavaScript for most projects.

## Asynchronous events in Hyperscript

Let's take a look at [how Hyperscript handles events](https://hyperscript.org/docs/#sending-events):

```
 <button _="on click send foo to the next <output/>">Send Foo</button>
<button _="on click trigger bar on the next <output/>">Send Bar</button>
<output _="on foo put 'I got a foo event!' into me
         on bar put 'I got a bar event!' into me">
No Events Yet...
</output> 
```

[Reactive programming](https://www.infoworld.com/article/3701889/what-is-reactive-programming-programming-with-event-streams.html) is an important plotline in the history of programming, and Hyperscript embraces it fully. In this example, we can see how Hyperscript approaches asynchronous events. The eventing system is quite powerful, including a range of reactive-style functions like filtering, event message objects, queuing, and more.

You can also see that the phrase “the next `<output/>`” is able to refer to another element in the DOM, specifically, the next `<output/>` element, and send the event to it. That's a very terse and obvious way to do something that would otherwise be fairly verbose and clunky, or would at least require some reactive wiring. This is one example of how Hyperscript does away with the separation of concerns by design.

## Looping, conditionals, and logging

Looping can be finicky in some front-end contexts (like [JSX for React](https://www.infoworld.com/article/3307858/tutorial-programming-in-react-for-beginners.html)). Here's a [looping example in Hyperscript](https://hyperscript.org/docs/#loops):

```
 for x in [1, 2, 3] index i
  log i, "is", x
end 
```

This example also gives us a look at how Hyperscript handles logging. It’s pretty straightforward, using comma-separated values.

We’ve already seen how Hyperscript handles the `if` command with an `else`. Notice that it ends with the `End` keyword (unless you are at the end of the script anyway, as happens on an element property):

```
 if :x <= 3
  put :x into the next <output/>
else
  put '3 is the max...' into the next <output/>
End 
```

Hyperscript also supports an `unless` modifier that can reference things like CSS properties:

```
 <button _="on click toggle .bordered on #second-button">
Toggle Next Border
</button>
<button id="second-button"
        _="on click toggle .red unless I match .bordered">
Toggle My Background
</button> 
```

This code chunk makes the second button change its color, unless it has the `.bordered` class. That's some pretty concise handling of an otherwise unwieldy bit of JavaScript with CSS and HTML.

## Locality of behavior

One of the principles that is pounded into new programmers is the so-called separation of concerns (SoC). Most of the time, this principle holds good. By implementing SoC, we get decoupled components, and that makes for more resilient systems. There is a counter-current to consider, however, which Gross calls [locality of behavior](https://htmx.org/essays/locality-of-behaviour/). He's embedded this idea in Hyperscript, and you'll also find it in other projects, like [Tailwind](https://www.infoworld.com/article/3622288/tailwind-css-learn-the-joys-of-functional-responsive-css.html).

The idea here is that separating concerns can actually make for systems that are harder to follow. Separation of concerns on the front end usually means putting your markup (the view) in one place, your JavaScript (behavior) in another, and your CSS (presentation) somewhere else. The design benefit of doing all this turns out to be dubious at best. SoC usually comes to the rescue in more architectural situations. In projects founded on HTML, CSS, and JavaScript, though, it’s often a pain to have to jump between contexts and always keep the thread of your tasks in mind. It's actually one of the things that is appealing about JSX and the motivator behind the [Styled Components framework](https://www.infoworld.com/article/3678950/intro-to-css-in-js-generating-css-from-javascript.html).

### Violating separation of concerns

Hyperscript takes much of the grunt work you would extract into JavaScript and wraps it inside Hyperscript syntax, so you can just inline fairly elaborate functionality directly into the markup. This makes it easy to keep things together and makes them more self-documenting.

On the downside, there could be a problem if you need to make changes to behavior that go beyond the built-in syntax. If you need to do something out of the box, in other words, you will find that your strongly coupled components are relying on the Hyperscript engine itself, which you will have to modify.

I haven’t used Hyperscript in a large, real-world project, so I can’t speak to that directly. There is, however, [support for extensibility](https://hyperscript.org/docs/#extending) in the language. 

It's also possible to run JavaScript and Hyperscript [side-by-side](https://hyperscript.org/#enhance-existing-code), so that gives you latitude for gradual enhancement. It also raises the question, for me, of how running these languages in parallel would look in a React, Svelte, or Vue project.

## Conclusion

When I started out with Hyperscript, I was extremely skeptical—maybe even a little superior. I’ve been gradually charmed by Hypserscript’s feel, however. Do I think it’ll supplant JavaScript anytime soon?  No, I don’t. But I can envision a world where HTMX and Hyperscript have greatly simplified the JavaScript front end. 

There’s something about this push toward a simpler programming paradigm that I’d call *applied common sense*. By questioning assumptions and starting over with everything we’ve learned, maybe we can build better tools. 

One thing we know is that complexity kills creativity and productivity in software. As engineers, we are forever at risk of building in an abstraction that is the final straw of complexity—the one that kills our project.

Sometimes we get into technology because we *just like it*; the tool takes on a life of its own. The tooling, like React, becomes the product. That’s alright in some scenarios, but at other times, a simpler alternative like Hypserscript could be better.

At the very least, Hyperscript has fresh ideas to contribute. And like HTMX, this language could help improve the overall experience of developing on the web.

Copyright © 2023 IDG Communications, Inc.