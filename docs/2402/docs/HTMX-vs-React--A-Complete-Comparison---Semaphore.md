<!--yml
category: 未分类
date: 2024-05-27 15:04:30
-->

# HTMX vs React: A Complete Comparison - Semaphore

> 来源：[https://semaphoreci.com/blog/htmx-react](https://semaphoreci.com/blog/htmx-react)

The ultimate goal of HTMX is to provide modern browser interactivity directly within HTML, without the need for JavaScript. Although relatively new, with its initial release in late 2020, this frontend library has quickly caught the attention of the IT web community.

With 2nd place in the [2023 JavaScript Rising Stars](https://risingstars.js.org/2023/en#section-framework) “Front-end Frameworks” category (right behind React), a spot in the GitHub Accelerator, and over 20k stars on GitHub, HTMX is rapidly gaining popularity. Why is there so much excitement around it? Is it here to dethrone React? Let’s find it out!

In this HTMX vs React guide, you will discover why we came to HTMX, what it is, what features it offers, and how it compares to React in terms of performance, community, functionality, and more.

## HTMX vs React in Short

Take a look at the summary table below to jump right into the HTMX vs React comparison:

|  | **HTMX** | **React** |
| --- | --- | --- |
| Developed by | Big Sky Software | Meta |
| Open source | ✅ | ✅ |
| GitHub stars | 29k+ | 218k+ |
| Weight | 2.9 kB | 6.4 kB |
| Syntax | Based on HTML, with custom attributes | Based on JSX, an extended version of JavaScript |
| Goal | Add modern interactivity features directly in HTML | Provide a component-based, full-featured UI JavaScript library |
| Learning curve | Gentle | Steep |
| Features | AJAX requesting and some other minor features | Composability, one-way data binding, state management, hooks, and many others |
| Performance | Great | Good, especially on large-scale apps or complex web applications |
| Integration | Embeddable into existing HTML pages | Embeddable into existing HTML pages, but mainly used on JavaScript-based projects |
| Community | Small, but growing | The largest on the market |
| Ecosystem | Small | Very rich |

## How We Got to React: From jQuery to Modern Web Frameworks

In the early days of web development, developers relied on jQuery to deal with AJAX requests, DOM manipulation, and event handling. Over time, online applications evolved and became more modern, structured, and scalable. This is when frameworks and libraries such as Angular, React, and Vue made the difference.

React introduced a component-based architecture, changing web development forever. Its declarative approach to UI and one-way data flow simplifies web development, promoting reusability and maintainability. These aspects have made React the go-to solution for building dynamic, responsive, and interactive web applications.

## How We Got to HTMX: From Web Frameworks to a More Modern HTML

While web frameworks like React, Vue, and Angular are great for building structured web applications, their complexity can represent a huge overhead for developers seeking simplicity. This is where HTMX comes into play!

HTMX is a lightweight solution for modern interactivity as in React, but with simple integration and no overhead as in jQuery. It extends HTML with custom attributes, enabling AJAX requests without the need for JavaScript code. The idea behind HTMX is to keep things simple, allowing developers to wander into the magic of the Web without abandoning familiar HTML.

HTMX serves as a streamlined and flexible alternative in a universe dominated by more complex frontend frameworks. Learn more in the section below.

## HTMX: A New, Modern Approach to Interactivity

[HTMX](https://htmx.org/) is a lightweight, dependency-free, extendable JavaScript frontend library to access modern browser features directly from HTML. In detail, it allows you to deal with [AJAX requests](https://developer.mozilla.org/en-US/docs/Glossary/AJAX), [CSS Transitions](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Transitions/Using_CSS_transitions), [WebSockets](https://developer.mozilla.org/en-US/docs/Web/API/WebSockets_API), and [Server Sent Events](https://developer.mozilla.org/en-US/docs/Web/API/Server-sent_events/Using_server-sent_events) directly in HTML code.

The library gives you access to most of those features just by setting special HTML attributes, without having to write a single line of JavaScript. This way, HTMX brings HTML to the next level, making it a full-fledged hypertext.

Let’s now see what the library has to offer through some HTMX examples.

### AJAX Request Triggers

The main concept behind HTMX is the ability to send AJAX requests directly from HTML. That is possible thanks to the following attributes:

*   [`hx-get`](https://htmx.org/attributes/hx-get/): Issues a `GET` request to the given URL.
*   [`hx-post`](https://htmx.org/attributes/hx-post/): Issues a `POST` request to the given URL.
*   [`hx-put`](https://htmx.org/attributes/hx-put/): Issues a `PUT` request to the given URL.
*   [`hx-patch`](https://htmx.org/attributes/hx-patch/): Issues a `PATCH` request to the given URL.
*   [`hx-delete`](https://htmx.org/attributes/hx-delete/): Issues a `DELETE` request to the given URL.

When the HTML element with one of those HTMX attributes is triggered, an AJAX request of the specified type to the given URL will be made. By default, elements are triggered by the [“natural” event](https://htmx.org/docs/#triggers) (e.g., `change` for `<input>`, `<textarea>`, and `<select>`, `submit` for `<form>`, `click` for everything else). You can customize this behavior with the attribute [`hx-trigger`](https://htmx.org/attributes/hx-trigger/).

Now, consider the HTMX example below:

```
<div hx-get="/users">
    Show Users
</div>
```

This tells the browser:

> “When a user clicks on the `<div>`, send a `GET` request to the `/users` endpoint and render the response into the `<div>`“

Note that for this mechanism to work, the `/users` endpoint should return raw HTML.

### Query Parameters and Body Data

The way HTMX sets query parameters and body data depends on the type of HTTP request:

*   **`GET` requests**: By default, `hx-get` does not automatically include any query parameters in the AJAX request. To set query parameters, specify them in the URL passed to `hx-get`. Otherwise, override the HTMX default behavior with the [`hx-params`](https://htmx.org/attributes/hx-params/)attribute.
*   **Non-`GET` requests**: When an element is a `<form>`, the body of the AJAX request will include the values of all its inputs, using their `name` attribute as the parameter name. When it is not a `<form>`, the body will involve the values of all inputs in the nearest enclosing `<form>`. Otherwise, if the element has a `value` attribute, that will be used in the body. To add the values of other elements to the body, use the [`hx-include`](https://htmx.org/attributes/hx-include/) attribute with a CSS selector of all the elements whose values you want to include in the body of the request. Then, you can employ the `hx-params` attribute to filter out some body parameters. You can also write a custom [`htmx:configRequest`](https://htmx.org/events/#htmx:configRequest) event handler to programmatically modify the body definition logic.

### AJAX Result Handling

As mentioned earlier, HTMX replaces the inner HTML of the element that triggered the AJAX request with the HTML content returned by the server. You can customize this behavior using the [`hx-swap`](https://htmx.org/attributes/hx-swap/) and [`hx-target`](https://htmx.org/attributes/hx-target/) attributes:

*   `hx-swap` defines what to do with the HTML returned by the server, accepting one of the following auto-explicative values: `innerHTML` (default), `outerHTML`, `beforebegin`, `afterbegin`, `beforeend`, `afterend`, `delete`, `none`.
*   `hx-target` accepts a CSS selector and instructs HTMX to apply the swap logic to the selected element.

Now, take a look at the HTMX snippet below:

```
<button
  hx-post="/tasks"
  hx-swap="afterend"
  hx-target=".todo-list"
>
  Add task
</button>
```

This tells the browser:

> “When a user clicks the `<button>` node, perform a `POST` request to the `/tasks` endpoint and append the HTML returned by the server to the `.todo-list` element”

Awesome! You just explored the basics of HTMX and the fundamentals of how it works. Keep in mind that these are just some of the features supported by the library. For more information, [explore the documentation](https://htmx.org/docs/).

## HTMX vs React: Comparing the Two Web Technologies

Now that you know what HTMX is and how it works, let’s look at how it compares against React, the reigning king of frontend web development libraries. This section will explore the essential aspects to consider when determining which is better between HTMX and React.

Time to dig into the HTMX vs React comparison!

### Approach

*   **HTMX:** It extends HTML, providing the ability to interact with the server directly in markup. It prioritizes simplicity, conciseness, and readability:

```
<div hx-get="/hello-world">
    Click me!
</div> 
```

*   **React:** A full-featured JavaScript library for building user interfaces based on reusable components written in [JSX](https://react.dev/learn/writing-markup-with-jsx):

```
import React, { useState } from "react"

export default const HelloWorldComponent = () => {
  const [responseData, setResponseData] = useState(null)

  const handleClick = () => {
    fetch("/hello-world")
      .then(response => response.text())
      .then(data => {
        setResponseData(data)
      })
  }

  return (
    <div onClick={handleClick}>
      {responseData ? <>{responseData}</> : "Click me!"}
    </div>
  )
} 
```

### Learning Curve

*   **HTMX**: With its HTML-based syntax and approach, HTMX offers a smooth learning curve. Developers already familiar with traditional web development can master it in a few days, while newcomers can start using it from day zero.
*   **React**: Because of its unique approach to web development, React has a steep learning curve. Before building your first React application, you need to understand the concepts of SPA ([Single Page Application](https://en.wikipedia.org/wiki/Single-page_application)), Virtual DOM, JSX, state management, props, re-renders, and more. This may overwhelm some beginners.

### Features

*   **HTMX**: The core concept behind the library can be summarized as enabling AJAX calls in HTML without the need for JavaScript code. While other cool features could be mentioned, that pretty much sums up what HTMX has to offer.
*   **React:** Some of the features that have made React so popular are its component-based architecture based on code reuse, JSX syntax for easy UI development, robust state management, [hooks](https://react.dev/reference/react/hooks), support for both client- and server-side rendering, efficient Virtual DOM, CSS-in-JS support, and more.

### Performance

*   **HTMX**: Its lightweight, dependency-free nature means that web pages that rely on HTMX will have fast initial page loading and reduced client-side processing. In general, HTMX performs well when it comes to applications with simple interactions.
*   **React**: SPA applications built in React usually contain a lot of JavaScript. That results in higher network utilization and client-side rendering times. However, the virtual DOM and efficient reconciliation algorithm allow React to swiftly update the UI, making it suitable for large-scale applications.

### Integration

*   **HTMX**: It can be embedded in any HTML web page. HTMX integrates natively with backend technologies that can return raw HTML content, such as Node.js, Django, Laravel, Spring Boot, Flask, and others.
*   **React**: As a frontend library, not a framework, it is technically possible to integrate it into any existing sites. At the same time, integrating React may require additional configuration, especially in frontend projects not built around JavaScript.

Note that HTMX and React can coexist in the same project. This means that you can have a web page that uses both React and HTMX in different sections of the page, and even React components that rely on HTMX attributes.

### Use Cases

*   **HTMX**: Best suited for projects that require simple, modern, and dynamic interactivity. HTMX is a lightweight and efficient option when you do not need all the advanced features of a full frontend framework. It is also ideal for backend developers who want to serve interactive HTML pages without having to write dedicated client-side JavaScript logic.
*   **React**: ****Best suited for developing single-page applications and complex web applications that need to provide a rich user experience and/or deal with complex state. It is also a great option for large teams who want to reuse UI components across multiple projects.

Migrating from React to HTMX is possible and can lead to a [67% smaller codebase](https://htmx.org/essays/a-real-world-react-to-htmx-port/). However, that is recommended only when you do not need all the features that make React so popular, such as advanced state management.

*   **HTMX**: With the first release in late 2020, you cannot expect HTMX to be as popular as React. So, you will not find many guides, tutorials, and walkthrough videos about it. Nevertheless, the project has already reached more than [29k starts on GitHub](https://github.com/bigskysoftware/htmx) and there is a lot of buzz around it.
*   **React**: With millions of developers worldwide and over [218k stars on GitHub](https://github.com/facebook/react), React is the undisputed heavyweight champion of web development libraries. According to a [Statista survey](https://www.statista.com/statistics/1124699/worldwide-developer-survey-most-used-frameworks-web/), React is by far the most used frontend web library, with a market share of over 40%. No wonder, there are hundreds of thousands of tutorials, articles, and videos dedicated to React.

### Ecosystem

*   **HTMX**: While the library is extensible, the project is relatively new and there are not many HTMX libraries and utilities available. As of this writing, the [`htmx` tag on npm](https://www.npmjs.com/search?q=keywords:htmx) counts only 35 packages.
*   **React**: The [`react`](https://www.npmjs.com/search?q=keywords%3Areact) tag on npm alone counts over 6,000 libraries. This is just one of the React-related tags, and you can find dozens of thousands of other libraries compatible with it.

## Which Frontend Library Should You Choose Between HTMX and React?

As always when comparing two technologies, there is no real winner in absolute terms. HTMX and React are both excellent frontend web development libraries, and the choice of one over the other depends on the requirements and goals of your project.

When creating web applications that require state management, offer complex functionality, and need reusable components, then React is a more suitable option. When building sites with simple interactivity and no particular advanced features, HTMX might be a better solution.

To help you make an informed decision between HTMX and React, let’s look at the pros and cons of both libraries!

### HTMX: Pros and Cons

**👍 Pros**:

*   Simple and intuitive HTML-based syntax.
*   AJAX requests and DOM updates with just a couple of HTML attributes.
*   Dynamic interactivity directly in HTML with no JavaScript.
*   Easily integrates into existing HTML web pages.
*   Lightweight library that weighs only a few kB.

**👎 Cons**:

*   Needs backend UI endpoints that return raw HTML and are therefore more tied with the frontend.
*   Still relatively new.

### React: Pros and Cons

**👍 Pros**:

*   Structuring UI with reusable components written in JSX.
*   Complex state management and support for many other useful features.
*   The most used frontend web library in the world.
*   Developed and maintained by Meta.
*   Unopinionated on the backend.

**👎 Cons**:

*   Not so easy to learn and master.
*   Difficult to integrate into non-JavaScript-based projects.

## Conclusion

In this article HTMX vs React, you learned what HTMX is, how it works, and how it competes against React. HTMX enables modern HTML interactivity without the complexities introduced by full-fledged web frameworks. Although its future is bright, HTMX is not here to replace React. To better understand where HTMX shines, take a look at the list of [HTMX examples](https://htmx.org/examples/) from the official site.

Quite the opposite, the two libraries can coexist and target different use cases. As learned here, React is ideal for web applications with a rich user experience and complex functionality, while HTMX is better for web pages with simpler interactivity needs.