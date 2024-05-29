<!--yml
category: 未分类
date: 2024-05-27 14:56:53
-->

# GraphQL and the beads on a string - by Lucca Siaudzionis

> 来源：[https://blog.luccasiau.com/p/graphql-and-the-beads-on-a-string](https://blog.luccasiau.com/p/graphql-and-the-beads-on-a-string)

> “Your database, backend and the user are all beads on a string.”
> 
> Dax ([thdxr](https://twitter.com/thdxr))

Basically:

But normally you’ll be hosting your backend very closely to your database, making this other piece of art a bit more accurate:

* * *

Loading a typical web page requires different pieces of information — for example, you may be loading the logged-in user’s profile on the corner, the main content in the center, and other little details around the page. In a typical REST API design, this will often translate to several separate requests, looking like:

One of the main benefits of GraphQL is that easily enables the client to make a single request specifying all those pieces of data, and the backend stitches them together. The browser would make a single request asking for the user’s profile, the main content, and the rest — and the result looks way better:

Which brings some simplicity to the client and potentially reduces load time, improving your users’ experience.

* * *

While it’s true that, in the RESTful case, you can make those requests asynchronously, you would on the other hand add significant complexity to the client. That means dealing with independent request failures and all the dependencies between these separate calls.

You could also define a single REST endpoint that contains all the information to load the page and mimic the GraphQL behavior. But this would just shift the burden of dealing with complexity from the client to the backend. And while that may be viable for simple applications, it would not be in more complex ones.

In contrast, GraphQL’s design fits like a glove for this case. There is no need to design solutions to reduce latency, as it was designed specifically to address such cases.

* * *

This is just one of the many reasons I appreciate GraphQL. Some other advantages are the very, very easy use of GraphQL subscriptions, and how it makes the API contract between frontend and backend explicitly define (some people don’t like this one though, but I’m a huge fan).

Admittedly, GraphQL is *quite* different and feels weird at first. It may not seem worth spending the effort to “relearn” a new way to do core things. But one day I decided to just try it with Apollo and was surprised and how easy it was. So, if you ever feel a bit curious, just give it a shot.

[Share](https://blog.luccasiau.com/p/graphql-and-the-beads-on-a-string?utm_source=substack&utm_medium=email&utm_content=share&action=share)