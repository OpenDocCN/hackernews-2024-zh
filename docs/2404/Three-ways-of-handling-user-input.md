<!--yml
category: 未分类
date: 2024-05-27 13:38:51
-->

# Three ways of handling user input

> 来源：[https://dubroy.com/blog/three-ways-of-handling-user-input/](https://dubroy.com/blog/three-ways-of-handling-user-input/)

January 5, 2022

> Although there has been important progress in models and packages for the output of graphics to computer screens, there has been little change in the way that input from the mouse, keyboard, and other input devices is handled. New graphics standards are still using a fifteen-year-old model even though it is widely accepted as inadequate.

— *Brad Myers, [A New Model for Handling Input](https://www.cs.cmu.edu/~amulet/papers/p289-myers-TOIS-new-model.pdf), 1990.*

It’s 2022 and things are pretty much the same: the dominant way of handling user input is still based on events and — in some form or another — callbacks. They come in slightly different forms ([addEventListener](https://developer.mozilla.org/en-US/docs/Web/API/EventTarget/addEventListener), [responder](https://developer.apple.com/documentation/appkit/nsresponder) [objects](https://developer.apple.com/documentation/uikit/uiresponder), etc.) but the core idea is the same.

It’s still going strong more than 40 years later, so the event-and-callback-based approach clearly has something going for it. But in nearly every code base I’ve worked in, the hairiest, most difficult-to-modify logic is in the event handling code. Handling user input is — to borrow [a phrase from Laurence Tratt](https://tratt.net/laurie/blog/entries/parsing_the_solved_problem_that_isnt.html) — a solved problem that *isn’t*.

To help me understand the pros and cons of the events-and-callbacks model and to experiment with some other approaches, I decided to write up a simple example in a three different styles.

## A toy problem

The goal is to implement a square that you can either drag and drop, or click. The code should distinguish between the two gestures: a click shouldn’t just be treated as a drop with no drag. Finally, when you’re dragging, pressing escape should abort the drag and reset the object back to its original position.

Here’s an interactive demo. There are at least two bugs here — see if you can find them! 😁

## Approach #1: Event listeners

As a baseline, let’s start with the standard approach based on event listeners.

```
const draggable = document.getElementById("myObject");
const setStatus = (str) =>
  (document.getElementById("status").textContent = str);

let didDrag = false;
let dragOrigin;
let origPos;

draggable.addEventListener("pointerdown", (evt) => {
  evt.target.setPointerCapture(evt.pointerId);
  dragOrigin = { x: evt.clientX, y: evt.clientY };
  const { left, top } = draggable.getBoundingClientRect();
  origPos = { left, top };
  didDrag = false;
});
draggable.addEventListener("pointermove", (evt) => {
  if (dragOrigin) {
    didDrag = true;
    const deltaX = evt.clientX - dragOrigin.x;
    const deltaY = evt.clientY - dragOrigin.y;
    draggable.style.left = `${origPos.left + deltaX}px`;
    draggable.style.top = `${origPos.top + deltaY}px`;
    setStatus("dragging...");
  }
});
draggable.addEventListener("pointerup", (evt) => {
  dragOrigin = undefined;
  if (didDrag) {
    setStatus("Dropped!");
  } else {
    setStatus("Clicked!");
  }
});
window.addEventListener("keydown", (evt) => {
  if (evt.key === "Escape") {
    dragOrigin = undefined;
    draggable.style.left = `${origPos.left}px`;
    draggable.style.top = `${origPos.top}px`;
    setStatus("Cancelled!");
  }
});
```

There are slightly nicer/cleaner ways to implement this, but I think this is pretty good as a representative example. Note that I’m using the `pointerdown` rather than `mousedown` so that I can take advantage of [pointer capture](https://developer.mozilla.org/en-US/docs/Web/API/Element/setPointerCapture#overview_of_pointer_capture).

Taking a look at it with a critical eye, the first thing that sticks out is the shared state that’s used by all of the event handlers:

*   `didDrag` keeps track of whether any *pointermove* event occurred, so we can distinguish between a drag and a click
*   `dragOrigin` records the position where the drag gesture started, and doubles as a way of checking whether the mouse is currently down.
*   `origPos` holds the original position of the target object, so we can reset it if the drag is canceled.

One thing I dislike a bit about the callback-based approach is how the control flow is mostly implicit. You can “jump into” control at many parts of the code, and the order of the handlers is purely determined by the order that the events arrive.

Can you spot the problem with the `pointerup` handler, that causes the bugs that I mentioned above?

<details><summary>*Show me the answer!*</summary> The body of `pointerup` handler should be guarded with `if (dragOrigin) { ... }`, just like the `pointermove` handler. Otherwise (1) a drag that starts *outside* the square, but ends over it, is treated as a click; and (2) after cancelling a drag, the next `pointerup` event sets the status to *dropped*.</details>

## Approach #2: Polling

Another way to handle user input is *polling*, or periodically checking the state of the hardware. The original Smalltalk MVC used this approach, and it’s still used in game programming. ^(I don’t think I’ve ever written any code that used polling for handling user input, so I wanted to at least give it a try.)

Below is a polling-based version of the same example from above. I’ve left out the helper code that takes care of updating the `mouse` and `keyboard` variables using the standard DOM events.

```
let didDrag = false;
let didHandlePointerDown = false;
let dragOrigin;
let origPos;

const rectContains = ({ top, right, bottom, left }, x, y) =>
  left <= x && x <= right && top <= y && y <= bottom;

(function handleFrame() {
  if (mouse.buttons[0].pressed) {
    const { clientX, clientY } = mouse.location;
    const draggableRect = draggable.getBoundingClientRect();

    if (
      !didHandlePointerDown &&
      rectContains(draggableRect, clientX, clientY)
    ) {
      // Handle pointerdown
      dragOrigin = { x: clientX, y: clientY };
      origPos = {
        left: draggableRect.left,
        top: draggableRect.top,
      };
      didDrag = false;
    }

    // Ensure that we only act on pointerdown action in the
    // first frame that we detect that the button is pressed.
    didHandlePointerDown = true;

    if (
      dragOrigin &&
      (clientX !== dragOrigin.x || clientY !== dragOrigin.y)
    ) {
      // Handle pointermove
      didDrag = true;
      const deltaX = clientX - dragOrigin.x;
      const deltaY = clientY - dragOrigin.y;
      draggable.style.left = `${origPos.left + deltaX}px`;
      draggable.style.top = `${origPos.top + deltaY}px`;
      setStatus("dragging...");
    }
  } else if (dragOrigin) {
    // Handle pointerup
    dragOrigin = undefined;
    if (didDrag) {
      setStatus("Dropped!");
    } else {
      setStatus("Clicked!");
    }
  } else {
    didHandlePointerDown = false;
  }

  if (dragOrigin && keyboard.keys["Escape"].pressed) {
    // Handle keypress
    dragOrigin = undefined;
    draggable.style.left = `${origPos.left}px`;
    draggable.style.top = `${origPos.top}px`;
    setStatus("Cancelled!");
  }

  requestAnimationFrame(handleFrame);
})();
```

Conceptually, polling is a pretty simple approach, but it imposes a fair bit of extra complexity on the handling code. For one thing, I needed `didHandlePointerDown` so that it only reacts once when the mouse button is pressed, rather than on every frame. I also needed to do my own hit testing with `rectContains`. This is one downside of polling — you need to implement extra bookkeeping for some things that you get for free in an event-based model.

Of course, the major drawback of polling is that you always need to be checking for state changes. This consumes CPU cycles, and if it’s not frequent enough, you can miss some user input. That’s why it’s not seen much anymore outside game programming, where you typically have an active rendering loop anyways.

On the plus side, the control flow is more explicit when compared to the event listener approach. For example, the code that handles cancellation will always run after any other other handler *in the same frame*. We do still need to be careful not process a `pointerup` without a corresponding `pointerdown`, as those would always be handled in separate frames.

Another thing that’s nice here is that you basically have unrestricted access to the current state of the input devices, whereas in the callback-based model, it’s less convenient to do something like looking at the current mouse position inside the `keydown` handler. DOM events hack around this limitation by including some keyboard state in the MouseEvent instances ([`altKey`](https://developer.mozilla.org/en-US/docs/Web/API/MouseEvent/altKey),[`ctrlKey`](https://developer.mozilla.org/en-US/docs/Web/API/MouseEvent/ctrlKey), etc.), but that only addresses the most common cases.

With polling, not only do you have more flexibility in *where* you examine the input state, but also *when* you examine it. With events, there’s an inversion of control: the event dispatch mechanism determines when (and if) each component observes the state change. This can be limiting, but it’s also a huge benefit for composability, as it enables loosely-coupled coordination between parent and child components (e.g. event delegation). I don’t know how you could achieve the something similar in polling-based model.

### Hybrid models

After reading the first draft of this post, a few people mentioned event-based approaches that feel somewhat similar to the polling approach. For example, you could have a single callback that handles all the events, rather than a separate callback for each event.

This gives you all the advantages of events, while making the control flow a bit more explicit (as with the polling approach), and making it easier to share state and code between the different paths.

## Approach #3: Process-oriented

The third approach I wanted to try is inspired by [Squeak: a Language for Communicating with Mice](https://swtch.com/~rsc/thread/squeak.pdf), by Luca Cardelli and Rob Pike. The core idea is that concurrent, communicating processes are a good way to model user interaction. Note that *process* here doesn’t mean an OS-level process; it means a lightweight, sequential thread of control, like a [goroutine](https://go.dev/tour/concurrency/1) or a [Ruby fiber](https://ruby-doc.org/core-2.5.0/Fiber.html).

To experiment with these ideas in JS, I built a tiny library called Abro.js ^(which provides some slightly higher-level abstractions on top of `async` and `await`. The result looks a lot more like the [Esterel language](http://www-sop.inria.fr/meije/esterel/esterel-eng.html) than Squeak. (Esterel was also an inspiration for Cardelli and Pike.))

Here’s a working implementation based on Abro:

```
const draggable = document.getElementById("myObject");
const setStatus = (str) =>
  (document.getElementById("status").textContent = str);

const events = new abro.EventSource(draggable);
const windowEvents = new abro.EventSource(window);

abro.loop(async () => {
  // Control will block on this line until pointerdown happens.
  const downEvent = await events.pointerdown; // 👈 1️⃣
  downEvent.target.setPointerCapture(downEvent.pointerId);

  const { left, top } = draggable.getBoundingClientRect();
  const origPos = { left, top };

  let didDrag = false;

  // `abro.or` starts multiple fibers, and completes as soon as *one*
  // of them completes (the others are terminated).
  await abro.or( // 👈 2️⃣
    async function handleDrag() {
      // An infinite loop? Abro will terminate this fiber when either
      // `handlePointerUp` or `handleEscape` is done.
      while (true) { // 👈 3️⃣
        const { clientX, clientY } = await events.pointermove;
        didDrag = true;
        const deltaX = clientX - downEvent.x;
        const deltaY = clientY - downEvent.y;
        draggable.style.left = `${origPos.left + deltaX}px`;
        draggable.style.top = `${origPos.top + deltaY}px`;
        setStatus("dragging...");
      }
    },
    async function handlePointerUp() {
      await events.pointerup;
      if (didDrag) {
        setStatus("Dropped!");
      } else {
        setStatus("Clicked!");
      }
    },
    async function handleEscape() {
      let key;
      while (key !== "Escape") {
        ({ key } = await windowEvents.keydown);
      }
      draggable.style.left = `${origPos.left}px`;
      draggable.style.top = `${origPos.top}px`;
      setStatus("Cancelled!");
    }
  );
});
```

You can also try this version out:

Before getting into the details, let me explain the basics of Abro:

*   Each process (or *fiber*) is an async function that can block until an event arrives from an *EventSource*. For example, control will block at 1️⃣ [↩](#code-pointerdown) until a `pointerdown` occurs.
*   Unlike regular async functions, Abro fibers can be cleanly terminated from the outside. I do this by keeping track of the promise that each fiber is blocked on, so it can be terminated by rejecting that promise.
*   `abro.or` (see 2️⃣ [↩](#code-abro-or)) starts multiple fibers, and as soon as one of them completes, the others are terminated. It’s based on the `par/or` construct in [Céu](http://www.ceu-lang.org/), another Esterel-inspired language.
*   The ability to terminate fibers from the outside makes it possible to write loops with no explicit exit condition (see 3️⃣ [↩](#code-infinite-loop)), as long as they block somewhere within the loop.

If you’re interested, you can take a look at [Abro on GitHub](https://github.com/pdubroy/handling-user-input/blob/main/abro.js), but be warned — it’s very much a work in progress.

This first thing to notice with this approach is the clear, explicit sequencing between the different states. The first line of the loop blocks on a `pointerdown` event; the other handlers aren’t even active until control proceeds past that point^.

The other thing I really like is the separation of concerns that’s enabled by the `or` construct along with the ability to abort individual threads of control. Take a close look at `handleDrag` — it’s just an infinite loop that processes `pointermove` events. It doesn’t need to be concerned with cancellation, that happens automatically when the `handleEscape` task completes. Also, since this is built on top of events, we can still take advantage of event bubbling and other forms of coordination between a container and its descendants.

Overall, I find the process-oriented approach to be pretty compelling. It seems to retain all the advantages of the event-driven model (e.g., in comparison to polling), while improving on some of the issues that callbacks have. I’m definitely interested to try it out in some more realistic use cases.

I found it really useful to work through a toy problem with a few different approaches. Comparing event listeners to polling made it much easier for me to see the pros and cons of each style. The advantages of events (and event dispatch) are much more obvious to me now, and I realized that polling isn’t really as simple as it sounds.

Of course, the three styles I’ve presented here are just a starting point, and there are many other approaches that I haven’t touched on. There are two in particular that I want to briefly mention:

*   [Statecharts](https://statecharts.dev/) are closely related to the process-oriented model I’ve explored here. The [original statecharts paper](https://dubroy.com/refs/Statecharts_a_visual_formalism_for_complex_systems.pdf) references Esterel, saying: *“It is motivated by concerns very similar to our own, and many of the resulting decisions taken there are strikingly similar to those present in statecharts.”* There’s enough to unpack there that I decided to leave it for a follow-up post.
*   [Functional reactive programming](https://stackoverflow.com/questions/1028250/what-is-functional-reactive-programming) (FRP), and related approaches like RxJS. I don’t have much experience with FRP, but I really should give it a try. Maybe another topic for a future post!

Check out [Part II](https://dubroy.com/blog/handling-user-input-with-structured-concurrency/), where I dig into the implementation details of Abro.js, and explore the process-oriented approach a bit more deeply.

💬 *Want to leave feedback? [Send me an email](https://dubroy.com/blog/about/#contact) or [respond on on Twitter](https://twitter.com/dubroy/status/1478728490807611393).*

*Thanks to Mariano Guerra, Sarah GHP, and Kevin Lynagh for providing feedback on my drafts.*