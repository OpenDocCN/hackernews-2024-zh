<!--yml

category: 未分类

date: 2024-05-27 13:38:51

-->

# 三种处理用户输入的方式

> 来源：[https://dubroy.com/blog/three-ways-of-handling-user-input/](https://dubroy.com/blog/three-ways-of-handling-user-input/)

2022年1月5日

> 尽管在将图形输出到计算机屏幕的模型和包方面取得了重要进展，但从鼠标、键盘和其他输入设备接收输入的方式几乎没有变化。即使被广泛认为不足以满足需求，新的图形标准仍在使用一个十五年前的模型。

— *布拉德·迈尔斯，[处理输入的新模型](https://www.cs.cmu.edu/~amulet/papers/p289-myers-TOIS-new-model.pdf)，1990年。*

现在已经是2022年，情况依旧如此：处理用户输入的主要方式仍然是基于事件和 — 以某种形式 — 回调。它们以稍微不同的形式出现（[addEventListener](https://developer.mozilla.org/en-US/docs/Web/API/EventTarget/addEventListener)，[responder](https://developer.apple.com/documentation/appkit/nsresponder) [对象](https://developer.apple.com/documentation/uikit/uiresponder)，等等），但核心思想是相同的。

40多年后它仍然强大，因此基于事件和回调的方法显然有其优点。但在我工作过的几乎每个代码库中，最复杂、最难修改的逻辑都在事件处理代码中。处理用户输入 — 借用[劳伦斯·特拉特的一句话](https://tratt.net/laurie/blog/entries/parsing_the_solved_problem_that_isnt.html) — 是一个解决了但*并未*解决的问题。

为了帮助我理解事件和回调模型的利弊，并尝试一些其他方法，我决定用三种不同的风格写一个简单的示例。

## 一个玩具问题

目标是实现一个正方形，你可以拖放或点击。代码应该区分这两种手势：点击不应仅被视为一个没有拖动的放置。最后，在拖动时，按下Escape键应该中止拖动并将对象重置回其原始位置。

这里有一个交互式演示。这里至少有两个bug — 看看你能找到它们吗！ 😁

## 第一种方法：事件侦听器

作为基线，让我们从基于事件侦听器的标准方法开始。

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

有更好/更清晰的实现方式，但我认为这个作为一个代表性示例已经很不错了。请注意，我使用`pointerdown`而不是`mousedown`，以便我可以利用[指针捕获](https://developer.mozilla.org/en-US/docs/Web/API/Element/setPointerCapture#overview_of_pointer_capture)。

仔细审视它，首先显眼的是所有事件处理程序都使用的共享状态：

+   `didDrag` 用于跟踪是否发生了任何*pointermove*事件，因此我们可以区分拖动和点击

+   `dragOrigin` 记录了拖拽手势开始的位置，并且也充当了检查鼠标当前是否按下的一种方式。

+   `origPos` 保存了目标对象的原始位置，这样如果拖拽被取消，我们就可以重置它。

我对基于回调的方法有一点不喜欢的地方是控制流大多是隐式的。你可以在代码的许多部分“跳入”控制，并且处理程序的顺序纯粹由事件到达的顺序决定。

你能找出 `pointerup` 处理程序中导致我上面提到的问题的问题吗？

<details><summary>*给我看答案！*</summary> `pointerup` 处理程序的主体应该通过 `if (dragOrigin) { ... }` 进行保护，就像 `pointermove` 处理程序一样。否则 (1) 从正方形外开始但在其上结束的拖拽会被视为点击；(2) 在取消拖动后，下一个 `pointerup` 事件将状态设置为 *dropped*。</details>

## 方法 #2：轮询

处理用户输入的另一种方法是 *轮询*，即定期检查硬件状态。最初的Smalltalk MVC就使用了这种方法，在游戏编程中仍然被广泛使用。^(我想我从来没有编写过使用轮询处理用户输入的代码，所以我至少想试试。)

下面是与上述相同示例的基于轮询的版本。我省略了处理更新 `mouse` 和 `keyboard` 变量的帮助代码。

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

在概念上，轮询是一个非常简单的方法，但它会给处理代码增加相当多的额外复杂性。首先，我需要 `didHandlePointerDown`，这样它只在按下鼠标按钮时才会反应一次，而不是每帧都会反应。我还需要使用 `rectContains` 来进行自己的命中测试。这是轮询的一个缺点 —— 你需要为一些东西实现额外的记录，而在事件驱动模型中你是免费获得这些的。

当然，轮询的主要缺点是你总是需要检查状态变化。这会消耗CPU周期，如果检查频率不够高，可能会错过一些用户输入。这就是为什么在游戏编程之外它不再常见的原因，因为通常你已经有一个活跃的渲染循环。

正面看，与事件监听器方法相比，控制流更为显式。例如，处理取消的代码总是会在同一帧中的任何其他处理程序之后运行。我们仍然需要注意，不要在没有对应的 `pointerdown` 的情况下处理 `pointerup`，因为它们总是在不同的帧中处理。

这里还有一点很好的地方，你基本上可以无限制地访问当前输入设备的状态，而在基于回调的模型中，要做像在`keydown`处理程序中查看当前鼠标位置这样的事情就不那么方便了。DOM事件通过在MouseEvent实例中包含一些键盘状态（例如[`altKey`](https://developer.mozilla.org/en-US/docs/Web/API/MouseEvent/altKey)，[`ctrlKey`](https://developer.mozilla.org/en-US/docs/Web/API/MouseEvent/ctrlKey)等）来解决这一限制，但这只解决了最常见的情况。

使用轮询，不仅可以更灵活地*检查输入状态的位置*，还可以*检查其发生的时间*。使用事件时，控制权有所倒置：事件分发机制决定每个组件何时（以及是否）观察状态更改。这可能有所限制，但对于组合性而言，却是一个巨大的优势，因为它支持父子组件之间的松耦合协调（例如事件委托）。我不知道如何在基于轮询的模型中实现类似的东西。

### 混合模型

在阅读本文初稿后，有几个人提到基于事件的方法，感觉与轮询方法有些类似。例如，你可以有一个单一的回调函数来处理所有事件，而不是每个事件都有一个单独的回调函数。

这使你可以享受事件的所有优势，同时使控制流程更加明确（就像轮询方法一样），并且更容易在不同路径之间共享状态和代码。

## 方法三：面向进程

我想尝试的第三种方法灵感来自[Squeak：与鼠标交流的语言](https://swtch.com/~rsc/thread/squeak.pdf)，作者是Luca Cardelli和Rob Pike。核心思想是并发的、通信的进程是模拟用户交互的一个很好的方法。请注意，这里的*进程*并不是指操作系统级别的进程；它指的是轻量级的、顺序控制的线程，类似于[goroutine](https://go.dev/tour/concurrency/1)或[Ruby fiber](https://ruby-doc.org/core-2.5.0/Fiber.html)。

要在JS中尝试这些想法，我构建了一个名为Abro.js的小型库，^(它在`async`和`await`之上提供了一些稍高级的抽象。结果看起来更像[Esterel语言](http://www-sop.inria.fr/meije/esterel/esterel-eng.html)，而不是Squeak。（Esterel也是Cardelli和Pike的灵感来源。))

这是一个基于Abro的工作实现：

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

你也可以尝试这个版本：

在深入讨论之前，让我解释一下Abro的基础：

+   每个进程（或*纤程*）是一个异步函数，可以阻塞，直到从*EventSource*接收到事件为止。例如，控制将在1️⃣ [↩](#code-pointerdown)处阻塞，直到发生`pointerdown`事件。

+   与普通的异步函数不同，Abro纤程可以从外部清晰地终止。我通过跟踪每个纤程所阻塞的Promise来实现这一点，因此可以通过拒绝该Promise来终止它。

+   `abro.or`（见 2️⃣ [↩](#code-abro-or)）启动多个纤程，一旦其中一个完成，其他纤程就会终止。它基于[Céu](http://www.ceu-lang.org/)中的`par/or`结构，这是受Esterel启发的另一种语言。

+   从外部终止纤程的能力使得编写没有显式退出条件的循环成为可能（见 3️⃣ [↩](#code-infinite-loop)），只要它们在循环内的某个地方阻塞即可。

如果你感兴趣，你可以查看[GitHub上的Abro](https://github.com/pdubroy/handling-user-input/blob/main/abro.js)，但请注意——它目前仍在积极开发中。

这种方法的第一件需要注意的事情是不同状态之间的清晰、明确的序列。循环的第一行在`pointerdown`事件上阻塞；直到控制流过该点后，其他处理程序甚至都不活跃^。

我真正喜欢的另一件事是由`or`结构和中止单个控制线程的能力启用的关注点分离。仔细看看`handleDrag` —— 它只是处理`pointermove`事件的无限循环。它不需要关注取消操作，当`handleEscape`任务完成时，取消操作会自动发生。此外，由于这是基于事件的，我们仍然可以利用事件冒泡和容器及其后代之间的其他形式协调。

总的来说，我发现面向过程的方法相当引人入胜。它似乎保留了事件驱动模型的所有优势（例如与轮询相比），同时改进了回调函数存在的一些问题。我绝对有兴趣在一些更现实的用例中尝试它。

我发现通过几种不同的方法解决一个玩具问题非常有用。比较事件监听器和轮询使我更容易看到每种风格的利弊。事件（和事件分发）的优势对我来说现在明显多了，我意识到轮询并不像听起来那么简单。

当然，我在这里展示的三种风格只是一个起点，还有许多其他方法我没有涉及到。有两种方法特别值得一提：

+   [状态图](https://statecharts.dev/)与我在这里探讨过的面向过程模型密切相关。[原始状态图论文](https://dubroy.com/refs/Statecharts_a_visual_formalism_for_complex_systems.pdf)引用了Esterel，称：“它的动机与我们自己的关注非常相似，而且其中许多结果决策与状态图中的那些决策非常相似。” 那里有足够的内容需要解读，所以我决定将其留给后续文章。

+   [函数式响应式编程](https://stackoverflow.com/questions/1028250/what-is-functional-reactive-programming)（FRP）及其相关方法，如 RxJS。我对FRP的经验不多，但确实应该试一试。也许是未来文章的另一个话题！

查看 [第二部分](https://dubroy.com/blog/handling-user-input-with-structured-concurrency/)，在那里我深入探讨了 Abro.js 的实现细节，并更深入地探索了面向过程的方法。

💬 *想要留下反馈？[给我发电子邮件](https://dubroy.com/blog/about/#contact) 或者 [在 Twitter 上回复](https://twitter.com/dubroy/status/1478728490807611393)。*

*感谢 Mariano Guerra、Sarah GHP 和 Kevin Lynagh 对我的草稿提供反馈。*
