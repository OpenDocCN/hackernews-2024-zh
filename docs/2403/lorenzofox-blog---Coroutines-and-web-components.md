<!--yml

类别：未分类

日期：2024-05-27 14:43:21

-->

# lorenzofox 博客 | 协程和 web 组件

> 来源：[https://lorenzofox.dev/posts/component-as-infinite-loop/](https://lorenzofox.dev/posts/component-as-infinite-loop/)

<main id="main">

在[上一篇文章](/posts/coroutine)中，我们学习了什么是协程，并看到它们可以帮助实现的一些模式。在本文中，我们将看到协程如何以不同的方式对 web 组件进行建模，以及为什么你可能会喜欢它。

协程除了其他属性外，还有一些我们将在本篇短文中使用的属性：

+   它们主要是**函数**，可以受益于 JavaScript 的整个函数库（组合、高阶函数、委托等）。

+   它们是**有状态的**。

+   当它们被暂停时，你可以注入几乎任何类型的数据。例如，例程体内的无限循环可以被视为公共 API 函数。

+   按设计，你无法同时调用`next`函数。

考虑以下生成器：

```
function* someComponent({$host}) {
    while (true) {
        const {content = ''} = yield;
        $host.textContent = content;
    }
}
```

它接受一个`$host` DOM 元素并具有渲染循环。你可以用一个生成器包装这个函数，产生一个`render`函数：

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

目前，渲染循环是一段命令式代码，但你可以使用任何你想要的渲染库（例如 React 等）。上面的第一个点说函数（因此协程）在 JavaScript 中非常多才多艺。如果我们希望，我们可以很容易地回到一个已知的范例。例如，我们使用[lit-html](./todo)来获得一个声明式视图，而不是一堆命令式代码：

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

你可以将模板绘制成一个函数：

```
import {html} from 'lit-element';

const template = ({name = ''} = {}) => html`<p>hello ${name}</p>`;
```

并与一个新的高阶函数组合：

```
import {render} from 'lit-element';

const withView = (templateFn) => function* ({$host}) {
    while (true) {
        render($host, templateFn(yield));
    }
};

const HelloWorldComponent = createComponent(withView(template));
```

好的，我们现在处于熟悉的领域：我们的组件现在是状态的一个简单函数。

```
({name}) => html`<p>hello ${name}</p>`; 
```

在我们的组件中建模无限渲染循环实际上比一开始看起来更有趣：你可以在该循环的闭包中拥有一个状态。

如果我们首先稍微修改更高级别的`createComponent`函数，将`render`函数绑定到宿主元素：

```
const createComponent = (generator) => ({$host}) => {
    const gen = generator({$host});
    gen.next();
    $host.render = (state = {}) => gen.next(state);
    return $host;
};
```

现在我们可以使组件触发其自身的渲染：

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

在像 React 这样的框架中，你只能访问循环内部的等价物，你依赖于框架的扩展点（例如 React 中的 hooks）来构建这种机制，并且对于渲染调度几乎没有控制权。

组件同时嵌入其视图和一些逻辑。同样，我们可以轻松解耦它们，以便可以重用视图或逻辑：我们只需要利用介绍中提到的协程的第三个属性，以及生成器固有的简单委托机制：`yield*`。

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

此类 mixin 负责保存状态并触发任何*视图*的渲染。渲染交由**委托**处理，而状态则在视图协程暂停时传递，并需要进行新的渲染：

```
const CountClick = createComponent(countClickable(function* ({$host}) {
    while (true) {
        const {count = 0} = yield;
        $host.textContent = `I have been clicked ${count} time(s)`;
    }
}));
```

很棒！现在你可以独立使用“可点击”行为，应用在不同的视图上。同样，您可以将视图插入到不同的控制器逻辑中，只要它通过预期的数据接口（`{ count: number | string}`）即可：请注意数据是如何从`yield`赋值中获取的。

在未来的文章中，我们将会看到更多类似的模式。

到目前为止，我们已经设计了我们的组件作为宿主的一个函数。我们可以进一步确保渲染程序实际上是私有于宿主，以便渲染代码与任何潜在的行为增强（例如`countClickable`混合）一起封装在内，同时两者保持可重用。

让我们看看另一种[自定义元素](https://developer.mozilla.org/en-US/docs/Web/API/Web_components/Using_custom_elements)建模的方式。为了增强您的HTML文档，您可以使用其注册表和`define`方法来向浏览器教授新元素。

```
customElements.define('hello-world', class extends HTMLElement {
    connectedCallback() {
        this.textContent = `hello ${this.getAttribute('name')}`
    }
}) 
```

然后在标记中像使用任何其他常规HTML标签一样使用`hello-world`标签。

```
<hello-world name="Laurent"></hello-world>
```

我们不再使用扩展`HTMLElement`类（或任何其他有效的内置元素类）的类。我们希望第二个参数是一个生成器函数。这意味着我们的自定义`define`需要将生成器转换为一个类。

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

使用类表达式，我们可以动态创建自定义元素类。`#loop`渲染程序在构造函数内实例化，并进入其第一个`yield`点。请注意，我们将宿主作为参数传递给程序，尽管程序专门绑定到宿主，因此我们可以在生成器内部使用`this`来引用宿主。这是一种个人偏好，因为我发现在JavaScript中使用`this`非常容易出错。

当调用`connectedCallback`时（当组件被挂载到DOM中时）。我们再次调用`next`，这在我们之前的示例中对应于循环的第一次迭代。然后，每当需要重新渲染组件（调用`render`时），我们继续循环。

这非常有趣，因为我们能够将不同组件生命周期匹配到生成器函数的某个位置：

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

然而，还有一个重要的生命周期需要实现。当组件卸载时，通常会调用类定义的`disconnectedCallback`，允许我们运行清理代码，例如避免内存泄漏。

在生成器中，我们可以强制退出循环到一个`finally`子句中。这就像调用循环的`return`函数而不是通常的`next`一样简单。

总体来说：

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

这种关于组件及其生命周期的“线性”表示使得理解变得更加容易：当回调或钩子被调用时，没有什么会让人感到意外，一切都是从头到尾顺序阅读的！

在我们结束之前，我们可以说明介绍中提到的第四点：如果尝试在生成器函数正在前进时推进它，将会出现错误。在组件世界中，这意味着并发渲染在设计上是不可能的！

此代码：

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

将触发错误 `Uncaught TypeError: already executing generator`。

我们在整篇文章中看到，生成器的函数性质与其固有属性的结合可以用来构建非常灵活和简单的UI组件抽象，具有将行为和视图拆分为可重用部分的能力，维护内部状态或在同一位置方便地获取所有组件生命周期的能力。

在[下一篇文章](/posts/reactive-attributes)中，我们将看到如何进一步改进和优化我们的生成器到类的转换。

</main>
