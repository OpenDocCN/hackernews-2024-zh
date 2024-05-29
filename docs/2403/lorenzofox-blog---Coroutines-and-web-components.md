<!--yml
category: 未分类
date: 2024-05-27 14:43:21
-->

# lorenzofox blog | Coroutines and web components

> 来源：[https://lorenzofox.dev/posts/component-as-infinite-loop/](https://lorenzofox.dev/posts/component-as-infinite-loop/)

<main id="main">

In the [previous article](/posts/coroutine) we learned what coroutines are and saw some patterns they can help implement. In this article, we will see how coroutines can be used to model web components in a different way, and why you might like it.

## 

Among other things, coroutines have a few properties that we will use in this short essay:

*   They are primarily **functions** and can benefit from the whole functional arsenal of Javascript (composition, higher order function, delegation, etc.).
*   They are **stateful**.
*   You can inject pretty much any kind of data when they are paused. For example, an infinite loop within the body of the routine can be considered as a public API function.
*   You cannot, by design, call the `next` function concurrently.

## 

Consider the following generator:

```
function* someComponent({$host}) {
    while (true) {
        const {content = ''} = yield;
        $host.textContent = content;
    }
}
```

It takes a `$host` DOM element and has a rendering loop. You can wrap this generator with a function that produces a `render` function:

```
const createComponent = (generator) => ({$host}) => {
    const gen = generator({$host});
    gen.next(); 
    return (state = {}) => gen.next(state);
};

const HelloWorldComponent = createComponent(function* ({$host}) {
    while (true) {
        const {name = ''} = yield;
        $host.textContent = `hello ${name}`;
    }
});

const render = HelloWorldComponent({
    $host: div
});

render({name: 'Laurent'});
render({name: 'Bernadette'});
```

## 

For now, the rendering loop is a piece of imperative code, but it can use any rendering library you want (react and so on). The first point above says that functions (and therefore coroutines) are very versatile in Javascript. We could easily go back to a known paradigm if we wanted to. For example, we use [lit-html](./todo) to have a declarative view instead of a bunch of imperative code:

```
import {render, html} from 'lit-element';

const HelloWorldComponent = createComponent(function* ({$host}) {
    while (true) {
        const {name=''} = yield
        const template = html`<p>hello ${name}</p>`;
        render($host, template);
    }
});
```

you can draw the template into a function:

```
import {html} from 'lit-element';

const template = ({name = ''} = {}) => html`<p>hello ${name}</p>`;
```

And compose with a new higher order function:

```
import {render} from 'lit-element';

const withView = (templateFn) => function* ({$host}) {
    while (true) {
        render($host, templateFn(yield));
    }
};

const HelloWorldComponent = createComponent(withView(template));
```

All right, we are on familiar ground: our component is now a simple function of the state

```
({name}) => html`<p>hello ${name}</p>`; 
```

## 

Having an infinite rendering loop to model our component can actually be more interesting than it seems at first: you can have a state in the closure of that loop.

If we first modify the higher-level `createComponent` function a little to bind the `render` function to the host element:

```
const createComponent = (generator) => ({$host}) => {
    const gen = generator({$host});
    gen.next();
    $host.render = (state = {}) => gen.next(state);
    return $host;
};
```

We can now make the component trigger its own rendering:

```
const CountClick = createComponent(function *({$host}){
   let clickCount = 0;

   $host.addEventListener('click', () => {
       clickCount+=1;
       $host.render();
   });

   while(true) {
       $host.textContent = `I have been clicked ${clickCount} time(s)`
       yield;
   }
});
```

In frameworks like React, where you only have access to the equivalent of what is inside the loop, you rely on the framework extension points (the hooks in the case of React) to build this sort of mechanism, and have very little control over rendering scheduling.

## 

The component embeds its view and some logic at the same time. Again, we can easily decouple them so that we can reuse either the view or the logic: All we need to do is take advantage of the third property of coroutines mentioned in the introduction, and a simple delegation mechanism inherent to generators: `yield*`.

```
const countClickable = (view) => function *({$host}) {
    let clickCount = 0;

    $host.addEventListener('click', () => {
        clickCount+=1;
        $host.render({count: clickCount});
    });

    yield* view({$host}); 
}
```

This type of mixin is responsible for holding the state and triggering the rendering of any *view*. Rendering is left to the view thanks to **delegation**, while the state is passed whenever the view coroutine is paused and requires a new render:

```
const CountClick = createComponent(countClickable(function* ({$host}) {
    while (true) {
        const {count = 0} = yield;
        $host.textContent = `I have been clicked ${count} time(s)`;
    }
}));
```

Neat ! You can now use the “clickable” behaviour independently, on different views. In the same way, you can plug the view into a different controller logic, as long as it passes the expected data interface (`{ count: number | string}`): note how the data comes from the `yield` assignation.

We will see more patterns like this in future articles.

## 

So far we have designed our component to be a function of the host. We can go further and ensure that the rendering routine is actually private to the host, so that the rendering code is encapsulated inside along with any potential behaviour enhancements (the `countClickable` mixin for example), while both remain reusable.

Let’s look at another way of modelling [custom elements](https://developer.mozilla.org/en-US/docs/Web/API/Web_components/Using_custom_elements). To enhance your HTML document, you can teach the browser new ones using its registry and the `define` method.

```
customElements.define('hello-world', class extends HTMLElement {
    connectedCallback() {
        this.textContent = `hello ${this.getAttribute('name')}`
    }
}) 
```

And then use the `hello-world` tag in the markup like any other regular HTML tag.

```
<hello-world name="Laurent"></hello-world>
```

Instead of using a class that extends the `HTMLElement` class (or any other valid built-in element class), we want the second argument to be a generator function. This means our custom `define` would need to turn the generator into a class.

```
const define = (tag, gen) => {
    customElements.define(tag, class extends HTMLElement {
        #loop;

        constructor() {
            super();
            this.#loop = gen.bind(this)({
                $host: this
            });
            this.render = this.render.bind(this);
            this.#loop.next();
        }

        connectedCallback() {
            this.render();
        }

        render(state = {}) {
            this.#loop.next(state);
        }

    });
};

define('hello-world', function* ({$host}) {
    while(true) {
        yield;
        $host.textContent = `hello ${$host.getAttribute('name')}`
    }
});
```

Using a class expression, we create the custom element class on the fly. The `#loop` rendering routine is instantiated inside the constructor and advanced to its first `yield` point. Note that we pass the host as a parameter to the routine, although the routine is specifically bound to the host so that we could just use `this` inside the generator to refer to the host. This is a personal preference as I find the use of `this` in Javascript very error-prone.

When the `connectedCallback` is called (this happens when the component is mounted into the DOM). We call `next` again, which in our previous example corresponds to the first iteration of the loop. Then, whenever the component needs to be rendered (when `render` is called) again, we continue the loop.

This is very interesting because we are able to match the different component lifecycles to a location within the generator function:

```
function* comp({$host}) {

    console.log('I am being instantiated');

    yield;

    console.log('Iam being mounted');
    $host.textContent = 'I have just been mounted';

    yield;

    while (true) {
        console.log('I am being rendered');
        $host.textContent = `hello ${$host.getAttribute('name')}`;
        yield 'I have been rendered';
    }
}
```

Yet, one important lifecycle remains to be implemented. When the component is unmounted, the `disconnectedCallbak` of the class definition is normally called, allowing us to run cleanup code and avoid memory leaks for example.

In the generator we can force the exit of the loop into a `finally` clause. This is as simple as calling the loop’s `return` function instead of the usual `next`.

Altogether:

```
const define = (tag, gen) => {
    customElements.define(tag, class extends HTMLElement {
        #loop;

        constructor() {
            super();
            this.#loop = gen.bind(this)({
                $host: this
            });
            this.render = this.render.bind(this);
            this.#loop.next();
        }

        connectedCallback() {
            this.render();
        }

        disconnectedCallback() {
            this.#loop.return();
        }

        render(state = {}) {
            this.#loop.next(state);
        }

    });
};

function* comp({$host}) {

    console.log('I am being instantiated');

    yield;

    console.log('Iam being mounted');
    $host.textContent = 'I have just been mounted'

    yield;

    try {
        while (true) {
            console.log('I am being rendered');
            $host.textContent = `hello ${$host.getAttribute('name')}`;
            yield;
        }
    } finally {
        console.log('cleanup here !!!');
    }
}
```

This “linear” representation of the component and its lifetime makes things easier to reason about: there are no surprises when a callback or a hook is called, everything is read from top to bottom!

## 

Before we conclude, we can illustrate the fourth point mentioned in the introduction: if you try to advance a generator function while it is already advancing, you will get an error. In the component world, this means that concurrent rendering is impossible by design!

This code:

```
function* comp({$host}) {
    while (true) {
        console.log('I am being rendered');

        $host.render(); 

        $host.textContent = `hello ${$host.getAttribute('name')}`;
        yield;
    }
}
```

will trigger an error `Uncaught TypeError: already executing generator`.

## 

We have seen throughout this article that the functional nature of a generator combined with its intrinsic properties can be useful to build a very flexible and simple abstraction of UI component, with the ability to split behaviour and view into reusable bits, to maintain internal state or to have at reach all component lifecycles in the same place.

In [the next article](/posts/reactive-attributes), we will see how we can further improve and optimise our generator-to-class conversion.

</main>