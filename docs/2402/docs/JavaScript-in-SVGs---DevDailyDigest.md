<!--yml
category: 未分类
date: 2024-05-27 14:40:06
-->

# JavaScript in SVGs - DevDailyDigest

> 来源：[https://www.devdailydigest.tech/javascript-in-svgs/](https://www.devdailydigest.tech/javascript-in-svgs/)

Recently, I stumbled upon a hidden gem in web development – the fusion of **JavaScript** and **Scalable Vector Graphics (SVG)**. Here’s a glimpse into the magic of JavaScript in SVG.

Yes, Its a SVG, which fetches crypto coin prices from an API and updates the SVG in real-time. Might not work sometimes, as the API is rate-limited. But you get the idea.

Here’s another one, The Chuck Norris Joke Generator.

Click on the SVG for another joke, if you want it before time runs out. To be honest, The jokes are not that good, but the SVG is cool.

## The Encounter with CDATA

You might have noticed the `<![CDATA[...]]>` tags in the script if you opened the dev-tools for inspecting.

This is CDATA (Character Data) – a way to include blocks of text, like JavaScript code, within XML documents without the need for escaping special characters. It ensures that the content is not misinterpreted as XML, making it perfect for embedding JavaScript in SVG.

Its important to note that it is not made for HTML, but for XML. Then why is it used in SVG? The answer is simple – SVG is an XML-based format.

In standalone SVG you need namespaces defined using `xmlns="http://www.w3.org/2000/svg"` and there you need CDATA to prevent various characters in javascript e.g. `<` being interpreted as the beginning of a tag and `&`. If you are embedding SVG in HTML which has javascript, you don’t need CDATA. Otherwise if so it is a standalone SVG, being opened in a browser, you need CDATA just to be safe.

```
<script type="text/ecmascript">
  <![CDATA[
    // Your JavaScript code here
  ]]>
</script>
```

## Why JavaScript in SVG?

I’m not sure what use-cases will this full fill, which can’t be done with simple javascript and HTML with SVG. But it’s a cool thing to know and experiment with.

This could be useful in creating dynamic graphics, like charts, graphs. It can also be used to make interactive SVGs, responsive to user actions, fetching external data, and even animating.

These are just a few things that comes to my mind for the potential of JavaScript in SVG. Probably theres a lot more to it, which I’m not aware of. Including some security concerns??

Here’s some more information on [MDN web docs](https://developer.mozilla.org/en-US/docs/Web/SVG/Element/script).