<!--yml
category: 未分类
date: 2024-05-27 14:32:09
-->

# Not all automated testing tools support Shadow DOM in web components - Manuel Matuzovic

> 来源：[https://matuzo.at/blog/2024/automated-testing-tools-and-web-components](https://matuzo.at/blog/2024/automated-testing-tools-and-web-components)

# Not all automated testing tools support Shadow DOM in web components

posted on 02.01.2024

There isn't much more to say; it's all in the title.

Many automated testing tools are a collection of JavaScript functions you run on a page. Most of those rely on querying the DOM. If a tool doesn't consider shadow trees, it only catches accessibility errors in the Light DOM, which may give you a wrong sense of safety and potentially affect your users.

That's one more reason [not to rely on automated testing only](/blog/building-the-most-inaccessible-site-possible-with-a-perfect-lighthouse-score/).

That doesn't mean you shouldn't use automated testing if your site contains web components. You just have to ensure that the tool you're using supports them. A good way of doing that is creating a simple component that contains some accessibility bugs caused by nodes attached to the component's Shadow DOM, like the following component.

```
<main>
  <h1 id="h1">Testing a11y bugs in web components</h1>

  <a href="#el">Jump to elem in shadow</a> <!-- Broken skip link -->

  <some-bugs></some-bugs>
</main>

<script>
class SomeBugs extends HTMLElement {
  constructor() {
    super();
    this.attachShadow({mode: 'open'});

    this.shadowRoot.innerHTML = `
    <style>:host { color: #efefef }</style> <!-- Low contrast. -->
    <h4 id="el">Heading</h4> <!-- Skipped heading. -->
    <img src="#test"> <!-- Missing alt. -->

    <button aria-labelledby="h1"></button> <!-- Broken ARIA reference/missing accessible name. -->
    <input type="text" id="input"> <!-- Missing label. -->
      `
  }
}

customElements.define('some-bugs', SomeBugs);
</script>
```

CodePen: [Test web component containing accessibility bugs](https://codepen.io/matuzo/pen/yLwYNYY?editors=1010)

If you add this component to a page, testing tools should find at least five issues. I picked five popular tools: [axe DevTools](https://www.deque.com/axe/), [Google Lighthouse](https://developer.chrome.com/docs/lighthouse/overview/), [ARC Toolkit](https://www.tpgi.com/arc-platform/arc-toolkit/), [WAVE](https://wave.webaim.org/extension/), and [IBM Equal Access (IBM EAAC)](https://www.ibm.com/able/toolkit/verify/automated/), and ran them against the component.

axe DevTools and Lighthouse, both based on axe-core, found all five issues.
WAVE found no problems in Shadow DOM, but reported a broken skip link in Light DOM. WAVE and ARC are the only tools that check for broken anchor links. Arguably, because it’s not an obvious accessibility issue or WCAG violation, but according to the Web Aim 1 Million report, [one out of every six skip links](https://webaim.org/projects/million/#skip) on tested sites that contained skip links were broken. They should check whether targets of anchor links exist.
IBM EA found all issues except the skipped heading, which can be considered a bad practice and not a violation, which IBM EA doesn't report anyway, regardless of the type of DOM.
ARC Toolkit found all issues except the low contrast text color of the heading.

Comparison of issues reported in different testing tools

| Bug | axe | WAVE | ARC | IBM EAAC |
| --- | --- | --- | --- | --- |
| Detected issues | 5 | 1 | 5 | 4 |
| Broken label/input | yes | no | yes | yes |
| Broken ARIA | yes | no | yes | yes |
| Missing alt | yes | no | yes | yes |
| Skipped heading | yes | no | yes | no |
| Low contrast | yes | no | no | yes |
| Broken anchor link | no* | yes | Yes | no* |

*not a direct accessibility concern, but should be checked.

When I published this post, ARC didn't support Shadow DOM, but it does since early May 2024 with version 5.7.0\. I'm not aware if testing Shadow DOM is on WAVE's roadmap, but in the meantime, I'd recommend not using the tool for sites that contain web components or using another tool to double-check.

The same applies to HTML validators. The official [W3C validator](https://validator.w3.org/), for example, doesn't support Shadow DOM.

## Update 04.01.24

Added info about ARC's plans to support Shadow DOM in early 2024.

## Update 08.01.24

Added clarification about the broken skip link bug.

## Update 17.05.24

Starting with version 5.7.0, [Arc Toolkit now supports Shadow DOM](https://www.tpgi.com/arc-toolkit-5-7-0/).