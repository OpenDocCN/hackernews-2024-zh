<!--yml

类别：未分类

日期：2024-05-27 14:31:03

-->

# 2024 年每个 JavaScript 开发人员都应该知道的 10 个面试问题 | 作者：Eric Elliott | JavaScript 场景 | Medium

> 来源：[https://medium.com/javascript-scene/10-interview-questions-every-javascript-developer-should-know-in-2024-c1044bcb0dfb](https://medium.com/javascript-scene/10-interview-questions-every-javascript-developer-should-know-in-2024-c1044bcb0dfb)

# **2024 年每个 JavaScript 开发人员都应该知道的 10 个面试问题**

JavaScript 的世界发生了巨大变化，面试趋势多年来也发生了很大变化。本指南介绍了 2024 年每个 JavaScript 开发人员都应该知道答案的 10 个基本问题。它涵盖了从闭包到 TDD 的各种主题，为您提供了处理现代 JavaScript 挑战所需的知识和信心。

作为一名招聘经理，我定期在真实的技术面试中使用所有这些问题。

当工程师们不知道答案时，我不会自动拒绝他们。相反，我会教他们概念，并了解他们的倾听、学习和应对面试问题不知道答案的紧张情况的能力。

一个好的面试官正在寻找渴望学习和推进自己的理解和职业发展的人。如果我在招聘一个经验较少的职位，候选人没有回答上述所有问题，但表现出了良好的学习能力，他们仍然有可能得到这份工作！

# 1\. 什么是闭包？

闭包允许您从内部函数访问外部函数的作用域。当函数嵌套时，内部函数可以访问外部函数作用域中声明的变量，即使外部函数已经返回：

```
const createSecret = (secret) => {
  return {
    getSecret: () => secret,
    setSecret: (newSecret) => {
      secret = newSecret;
    },
  };
};

const mySecret = createSecret("My secret");
console.log(mySecret.getSecret()); // My secret

mySecret.setSecret("My new secret");
console.log(mySecret.getSecret()); // My new secret
```

闭包变量是对外部作用域变量的活动引用，而不是副本。这意味着如果您更改外部作用域变量，将在闭包变量中反映这些更改，反之亦然，这意味着在同一外部函数中声明的其他函数将可以访问这些更改。

闭包的常见用例包括：

+   数据隐私

+   柯里化和部分应用（经常用于改进函数组合，例如对 Express 中间件或[React 高阶组件](/javascript-scene/why-every-react-developer-should-learn-function-composition-23f41d4db3b1)进行参数化）

+   与事件处理程序和回调共享数据

**数据隐私**

封装是面向对象编程的一个重要特性。它允许您隐藏类的实现细节，不让外界知道。JavaScript 中的闭包允许您为对象声明私有变量：

```
// Data privacy
const createCounter = () => {
  let count = 0;
  return {
    increment: () => ++count,
    decrement: () => --count,
    getCount: () => count,
  };
};
```

**柯里化函数和部分应用：**

```
 // A curried function takes multiple arguments one at a time.
const add = (a) => (b) => a + b;

// A partial application is a function that has been applied to some,
// but not yet all of its arguments.
const increment = add(1); // partial application

increment(2); // 3
```

# 2\. 什么是纯函数？

纯函数在函数式编程中很重要。纯函数是可预测的，这使它们比不纯的函数更容易理解、调试和测试。纯函数遵循两个规则：

1.  **确定性** — 给定相同的输入，纯函数将始终返回相同的输出。

1.  **无副作用** — 副作用是指除了返回值以外，在被调用函数之外可观察到的任何应用程序状态更改。

**非确定性函数的例子**

非确定性函数包括依赖于的函数：

+   一个随机数生成器。

+   一个可以改变状态的全局变量。

+   可以改变状态的参数。

+   当前系统时间。

**副作用的例子**

+   修改任何外部变量或对象属性（例如全局变量，或者父函数作用域链中的变量）。

+   记录到控制台。

+   写入到屏幕、文件或网络。

+   抛出错误。相反，函数应返回指示错误的结果。

+   触发任何外部进程。

在 Redux 中，所有的 reducer 必须是纯函数。如果不是，应用程序的状态将是不可预测的，像时间旅行调试之类的功能也将无法工作。reducer 函数中的不纯性也可能导致难以追踪的错误，包括陈旧的 React 组件状态。

# 3\. 什么是函数组合？

函数组合是将两个或多个函数组合成一个新函数或执行某些计算的过程：`(f ∘ g)(x) = f(g(x))`（`f` 组合 `g` 的 `x` 等于 `f` 的 `g` 的 `x`）。

```
const compose = (f, g) => (x) => f(g(x));

const g = (num) => num + 1;
const f = (num) => num * 2;

const h = compose(f, g);

h(20); // 42
```

[React 开发人员可以通过函数组合清理大型组件树](/javascript-scene/why-every-react-developer-should-learn-function-composition-23f41d4db3b1)。你可以将组件组合在一起，而不是嵌套它们，以创建一个新的高阶组件，可以通过传递给它的任何组件增强附加功能。

# 4\. 什么是函数式编程？

函数式编程是一种以纯函数作为主要组合单位的编程范式。组合在软件开发中非常重要，以至于几乎所有的编程范式都是根据它们使用的组合单位命名的：

+   面向对象编程使用对象作为组合的单位。

+   过程式编程使用过程作为组合的单位。

+   函数式编程使用函数作为组合的单位。

函数式编程是一种声明式的编程范式，这意味着程序是根据它们所做的事情编写的，而不是它们是如何做到的。这使得函数式程序比命令式程序更容易理解、调试和测试。它们也往往更加简洁，这减少了代码复杂性，并使其更易于维护。

函数式编程的其他关键方面包括：

+   **不可变性** — 不可变数据结构比可变数据结构更容易理解。

+   **高阶函数** — 接受其他函数作为参数或返回函数作为其结果。

+   **避免共享可变状态** — 共享可变状态使得程序难以理解、调试和测试。它也使得很难推断程序的正确性。

由于纯函数易于测试，函数式编程还倾向于产生更好的测试覆盖率和更少的错误。

# 5\. 什么是 Promise？

JavaScript 中的 Promise 是一个表示异步操作的最终完成或失败的对象。它充当一个值的占位符，最初是未知的，通常是因为其值的计算尚未完成。

Promise 的关键特性：

**有状态：** Promise 处于以下三种状态之一：

+   **待定：** 初始状态，既不被完成也不被拒绝。

+   **已完成：** 操作成功完成。

+   **已拒绝：** 操作失败。

**不可变：** 一旦 Promise 被完成或拒绝，其状态就不能改变。它变得不可变，永久地保存其结果。这使得 Promise 在异步流程控制中可靠。

**链式调用：** Promise 可以链式调用，意味着一个 Promise 的输出可以作为另一个 Promise 的输入。这是通过 `.then()` 来处理成功或 `.catch()` 来处理失败完成的，允许进行优雅且可读的顺序异步操作。链式调用是函数组合的异步等价。

```
const promise = new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve("Success!");
    // You could also reject with a new error on failure.
  }, 1000);
});

promise
  .then((value) => {
    console.log(value); // Success!
  })
  .catch((error) => {
    console.log(error);
  });
```

在 JavaScript 中，您可以将 promises 和返回 promise 的函数视为同步使用，使用 async/await 语法。这使得异步代码更易于阅读和理解。

```
const processData = async () => {
  try {
    const data = await fetchData(); // Waits until the Promise is resolved
    console.log("Processed:", data); // Process and display the data
  } catch (error) {
    console.error("Error:", error); // Handle any errors
  }
};
```

# 6\. 什么是 TypeScript？

TypeScript 是 JavaScript 的一个超集，由 Microsoft 开发和维护。它在近年来已经显著增长，并且有可能如果你是一名 JavaScript 工程师，您最终将需要使用 TypeScript。它为 JavaScript 添加了静态类型，这是一种动态类型语言。静态类型帮助开发人员在开发过程的早期捕获错误，提高代码质量和可维护性。

**TypeScript 的主要特性：**

**静态类型：** 为您的变量和函数参数定义类型，以确保代码的一致性。

**增强的 IDE 支持：** 集成开发环境（IDE）可以提供更好的自动完成、导航和重构功能，使开发过程更加高效。

**编译：** TypeScript 代码被转译为 JavaScript，使其与任何浏览器或 JavaScript 环境兼容。在此过程中，捕获类型错误，使代码更加健壮。

**接口：** 接口允许您指定对象和函数必须满足的抽象约定。

**与 JavaScript 的兼容性：** TypeScript 与现有的 JavaScript 代码高度兼容。JavaScript 代码可以逐渐迁移到 TypeScript，使得对现有项目的过渡更加平稳。

```
interface User {
  id: number;
  name: string;
}

type GetUser = (userId: number) => User;

const getUser: GetUser = (userId) => {
  // Fetch user data from a database or API
  return {
    id: userId,
    name: "John Doe",
  };
};
```

防止错误的最佳方法是代码审查、TDD 和诸如 ESLint 等的 lint 工具。TypeScript 不是这些实践的替代品，因为类型正确性不能保证程序正确性。即使在应用了所有其他质量措施之后，TypeScript 有时也会捕获错误。但它的主要好处是通过 IDE 支持提供的改进的开发人员体验。

# 7\. 什么是 Web 组件？

Web 组件是一组 Web 平台 API，允许您创建新的自定义、可重用、封装的 HTML 标签，用于网页和 Web 应用程序中。它们使用诸如 HTML、CSS 和 JavaScript 之类的开放 Web 技术构建。它们是浏览器的一部分，不需要外部库或框架。

Web 组件在具有许多可能使用不同框架的工程师的大型团队中特别有用。它们允许您创建可在任何框架中或完全无框架中使用的可重用组件。例如，Adobe 的 Spectrum 设计系统使用 Web 组件构建，并与流行的框架如 React 无缝集成。

Web 组件已经存在很长时间，但最近在大型组织中变得越来越受欢迎。它们受到所有主要浏览器的支持，并且是 W3C 标准。

```
<!-- Defining a simple Web Component -->
<script>
  // Define a class that extends HTMLElement
  class SimpleGreeting extends HTMLElement {
    // Define a constructor that attaches a shadow root
    constructor() {
      super();
      const shadowRoot = this.attachShadow({ mode: "open" });
      // Use a template literal for the shadow root's innerHTML
      shadowRoot.innerHTML = `
        <style>
          /* Style the web component using a style tag */
          p {
            font-family: Arial, sans-serif;
            color: var(--color, black); /* Use a CSS variable for the color */
          }
        </style>
        <!-- The <slot> element is a placeholder for user-provided content. -->
        <!-- If no content is provided, it displays its own default content. -->
        <p><slot>Hello, Web Components!</slot></p>
      `;
    }

    // Define a static getter for the observed attributes
    static get observedAttributes() {
      return ["color"]; // Observe the color attribute
    }

    // Define a callback for when an attribute changes
    attributeChangedCallback(name, oldValue, newValue) {
      // Update the CSS variable when the color attribute changes
      if (name === "color") {
        this.style.setProperty("--color", newValue);
      }
    }
  }

  // Register the custom element with a tag name
  customElements.define("simple-greeting", SimpleGreeting);
</script>

<!-- Using the Web Component -->
<!-- Pass a custom greeting message using the slot -->
<simple-greeting>Hello, reader!</simple-greeting>
<!-- Pass a custom color using the attribute -->
<simple-greeting color="blue">Hello, World!</simple-greeting>
```

# 8\. 什么是 React Hook？

Hooks 是一种让您在不编写类的情况下使用状态和其他 React 功能的函数。通过调用函数而不是编写类方法，Hooks 允许您使用状态、上下文、引用和组件生命周期事件。函数的额外灵活性使您能够更好地组织代码，将相关功能组合在单个钩子调用中，并通过在单独的函数调用中实现将无关的功能分离。Hooks 提供了一种强大而富有表现力的方式来在组件内部组合逻辑。

重要的 React Hooks

+   `useState` - 允许您向函数组件添加状态。状态变量在重新渲染之间保留。

+   `useEffect` - 允许您在函数组件中执行副作用。它将 `componentDidMount`、`componentDidUpdate` 和 `componentWillUnmount` 的功能组合成单个函数调用，减少了所需的代码，并创建了比类组件更好的代码组织。

+   `useContext` - 允许您在函数组件中消耗上下文。

+   `useRef` - 允许您创建一个在组件生命周期内持续存在的可变引用。

+   **自定义 Hooks** — 用于封装可重用的逻辑。这使得跨不同组件共享逻辑变得容易。

**Hooks 规则：** Hooks 必须在 React 函数的顶层使用（不能在循环、条件或嵌套函数内部使用），且仅限于 React 函数组件或自定义 Hooks 中使用。

钩子解决了类组件的一些常见痛点，比如需要在构造函数中绑定方法，以及需要将功能拆分为多个生命周期方法。它们还使得更容易在组件之间共享逻辑，并且在不改变组件层次结构的情况下重用有状态逻辑。

# 9\. 如何在React中创建一个点击计数器？

你可以使用`useState`钩子在React中创建一个点击计数器，如下所示：

```
import React, { useState } from "react";

const ClickCounter = () => {
  const [count, setCount] = useState(0); // Initialize count to 0

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount((count) => count + 1)}>Click me</button>
    </div>
  );
};

export default ClickCounter;
```

注意，将一个函数传递给`setCount`是在你从现有状态中派生新值时的最佳实践，以确保你总是使用最新的状态。

# 10\. 什么是测试驱动开发（TDD）？

测试驱动开发（TDD）是一种软件开发方法，其中测试在实际代码之前编写。它围绕着一个短小、重复的开发周期，旨在确保代码符合指定的需求，并且没有错误。TDD可以在提高代码质量、减少错误和提高开发人员生产力方面发挥重要作用。

开发团队生产力的一个重要衡量标准是部署频率。持续交付的主要障碍之一是对变更的恐惧。TDD通过确保代码始终处于可部署状态来减少这种恐惧。这使得更容易部署新功能和错误修复，从而增加部署频率。

先测试有许多优点，比后测试要好：

+   **更好的代码覆盖率：**先编写测试时，测试更有可能覆盖所有边缘情况。

+   **改进的API设计：**在编写代码之前，测试会迫使你考虑API设计，这有助于避免将实现细节泄漏到API中。

+   **更少的错误：**先测试有助于你在开发过程中更早地捕捉错误，这样更容易修复。

+   **更好的代码质量：**先测试会迫使你编写模块化、低耦合的代码，这样更容易维护和重用。

最后一点是TDD我最喜欢的特性，它教会了我大部分关于编写模块化、整洁架构的代码的知识。

TDD的关键步骤：

1.  **编写一个测试：**这个测试最初会失败，因为相应的功能尚不存在。

1.  **编写实现：**只需编写足够使测试通过的代码。

1.  **自信地重构：**一旦测试通过，就可以自信地进行重构。重构是重构现有代码而不改变其外部行为的过程。它的目的是清理代码，提高可读性，减少复杂性。有了测试，如果出错，测试会立即警示你。

**重复：**每个功能需求都要重复这个周期，逐渐构建软件，同时确保所有测试都能通过。

**挑战**

+   **学习曲线：** TDD 是一种需要花费相当时间来发展的技能和纪律。进行了 6 个月的 TDD 后，你可能仍然觉得 TDD 很困难，并且影响了生产力。然而，进行了 2 年的 TDD 后，你可能会发现它已经变得轻车熟路，并且比以往任何时候都更加高效。

+   **耗时：** 为每个小功能编写测试最初可能会感到耗时，尽管通常在长期内会以减少错误和更容易的维护来得到回报。我经常告诉人们，“如果你认为自己没有时间做 TDD，那么你*真的*没有时间跳过 TDD。”

# 结论

准备好在面试环境中回答这些问题，肯定会帮助你脱颖而出。它会帮助你成为一名更优秀的 JavaScript 开发者，这将有助于你在新角色中蓬勃发展。

# **接下来的步骤**

提升职业生涯的最快方式是一对一的导师指导。考虑到这一点，我共同创建了一个平台，该平台将工程师和工程领导与资深导师配对，他们将每周通过视频与您会面。主题包括*JavaScript，TypeScript，React，TDD，* [*AI 驱动开发*](/javascript-scene/the-art-of-effortless-programming-3e1860abe1d3)*和工程领导力*。立即加入 [DevAnywhere.io](https://devanywhere.io)。

更愿意自学有关函数式编程和 JavaScript 等主题吗？请查看 [EricElliottJS.com](https://ericelliottjs.com) 或购买我的书，[电子书](https://leanpub.com/composingsoftware) 或 [纸质书](https://amzn.to/3H2xqsQ)。
